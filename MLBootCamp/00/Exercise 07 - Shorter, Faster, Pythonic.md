Filtering words by length using a list comprehension. The constraint that `filter`
is forbidden is a nudge toward writing the list comprehension manually, which is the
idiomatic Python way anyway.

- A list comprehension builds a new list by applying an expression to each item in an iterable, optionally filtered by a condition
- `string.punctuation` gives you all punctuation characters to strip
- `str.translate()` is the fastest way to remove multiple characters from a string

## List Comprehension Syntax

The general form:

```python
[expression for item in iterable if condition]
```

The `if condition` part is optional. This is equivalent to a for loop that appends
to a list, but more readable and faster:

```python
# These are equivalent
result = [x * 2 for x in range(10) if x % 2 == 0]

result = []
for x in range(10):
    if x % 2 == 0:
        result.append(x * 2)
```

## Stripping Punctuation from Words

The exercise wants punctuation removed from each word before counting its length.
`str.translate()` with a deletion table is the cleanest approach:

```python
import string

# Build a translation table that maps every punctuation character to None (delete it)
table = str.maketrans('', '', string.punctuation)

word = "Hello,"
clean = word.translate(table)   # "Hello"
```

`str.maketrans(from, to, delete)` builds the mapping. When `delete` is provided,
those characters are removed entirely.

## The Full Solution

```python
import sys
import string

def main():
    assert len(sys.argv) == 3, "ERROR"
    assert isinstance(sys.argv[1], str), "ERROR"
    try:
        n = int(sys.argv[2])
    except ValueError:
        print("ERROR")
        return

    table = str.maketrans('', '', string.punctuation)
    words = [
        word.translate(table)
        for word in sys.argv[1].split()
        if len(word.translate(table)) > n
    ]
    print(words)
```

The condition and the transformation use `word.translate(table)` twice — you could
also do a nested comprehension or use a walrus operator (`:=`, Python 3.8+) to
avoid the double call, but since the module requires 3.7, just call it twice or
pre-process the list in two steps.

## Argument Validation

The tricky part is detecting that `sys.argv[1]` is the string and `sys.argv[2]` is
the integer — if the user swaps them, `int(sys.argv[2])` fails and you print ERROR.
But if they pass two strings like `Hello World` without quotes, `sys.argv` gets three
elements (script + Hello + World) and the argument count check catches it.

```python
if len(sys.argv) != 3:
    print("ERROR")
    sys.exit()
try:
    n = int(sys.argv[2])
except ValueError:
    print("ERROR")
    sys.exit()
```
