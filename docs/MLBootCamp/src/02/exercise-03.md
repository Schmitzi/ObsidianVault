# Exercise 03 - CsvReader (Context Manager)

A context manager is the mechanism behind `with` statements. Implementing one as
a class means defining `__enter__` and `__exit__` — Python calls these automatically
when entering and leaving the `with` block, guaranteeing cleanup even if an exception
is raised.

- `__enter__` runs when entering the `with` block; its return value is bound to `as file`
- `__exit__` runs when leaving the block for any reason — including exceptions
- If `__exit__` returns a falsy value, any exception propagates; return `True` to suppress it
- A corrupted CSV (inconsistent row lengths) should make the class return `None` from `__enter__`

## The Context Manager Protocol

```python
with SomeContextManager() as resource:
    # __enter__ was called; resource = __enter__'s return value
    do_stuff(resource)
# __exit__ was called here, whether or not an exception occurred
```

The three arguments to `__exit__` are the exception type, value, and traceback —
all `None` if no exception occurred. Return `True` to suppress an exception, `False`
(or `None`) to let it propagate.

## CsvReader Implementation

```python
class CsvReader:
    def __init__(self, filename=None, sep=',', header=False,
                 skip_top=0, skip_bottom=0):
        self.filename = filename
        self.sep = sep
        self.header = header
        self.skip_top = skip_top
        self.skip_bottom = skip_bottom
        self.file = None
        self._data = None
        self._header = None

    def __enter__(self):
        try:
            self.file = open(self.filename, 'r')
        except (FileNotFoundError, TypeError):
            return None

        lines = self.file.read().splitlines()

        # Split every line
        rows = [line.split(self.sep) for line in lines if line]

        if not rows:
            return self

        # Extract header if requested (before skip_top)
        if self.header:
            self._header = rows[0]
            rows = rows[1:]

        # Apply skip_top and skip_bottom
        if self.skip_bottom > 0:
            rows = rows[self.skip_top:-self.skip_bottom]
        else:
            rows = rows[self.skip_top:]

        # Validate: all rows must have the same length
        if rows:
            expected_len = len(rows[0])
            if any(len(row) != expected_len for row in rows):
                return None
            # Also check header length matches if we have one
            if self._header and len(self._header) != expected_len:
                return None

        self._data = rows
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        return False   # don't suppress exceptions

    def getdata(self):
        """Retrieves the data/records from skip_top to skip_bottom."""
        return self._data

    def getheader(self):
        """Retrieves the header from the csv file."""
        return self._header
```

## The Corruption Check

Two kinds of corruption to detect: rows with different numbers of fields (inconsistent
widths), and mismatch between the number of header fields and data fields. The
cleanest check uses a set of lengths — if all rows are the same width, the set has
exactly one element:

```python
lengths = {len(row) for row in rows}
if len(lengths) > 1:
    return None   # rows have inconsistent widths
```

## Why Return None From __enter__

The calling code checks `if file == None`. When `__enter__` returns `None`, the
`as file` variable is bound to `None`. The caller can then check and handle it.
Returning `self` from `__enter__` is the normal case — it's what makes `file.getdata()`
work.

## The file not found Case

`open()` raises `FileNotFoundError` when the file doesn't exist. Catch it in
`__enter__` and return `None`. The file handle `self.file` is only set if `open()`
succeeded, so `__exit__` needs to guard: `if self.file: self.file.close()`.

## Comparison: Class vs Generator Context Manager

The class approach (`__enter__`/`__exit__`) is the standard for stateful resources
like file handles. For simpler cases, `contextlib.contextmanager` lets you write
a generator with `yield` instead:

```python
from contextlib import contextmanager

@contextmanager
def open_file(path):
    f = open(path)
    try:
        yield f
    finally:
        f.close()
```

Both work — the class approach is more explicit about what runs when.
