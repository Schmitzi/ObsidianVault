The goal is to build `Matrix` and `Vector` classes from scratch without NumPy, implementing
all the standard matrix operations manually. This forces you to understand exactly what
happens under the hood when you later call `np.dot` or `np.transpose`.

## Initialisation

`Matrix` accepts two forms of input — either the data directly as a list of lists, or
a shape tuple to create a zero matrix:

```python
Matrix([[1.0, 2.0], [3.0, 4.0]])   # from data
Matrix((3, 3))                      # 3x3 zero matrix
```

The `__init__` must detect which case it's dealing with and set both `self.data` and
`self.shape` accordingly.

## The Aliased Row Trap

When building a zero matrix from a shape, the obvious shortcut creates a bug:

```python
self.data = [[0] * cols] * rows   # Wrong — all rows are the same object
```

Every row is a reference to the same list. Modifying `data[0][0]` changes every row.
The fix is to build each row independently:

```python
lst = []
for _ in range(rows):
    lst.append([0] * cols)
self.data = lst
```

## Transpose

The cleanest transpose uses `zip(*self.data)` — unpacking the rows and zipping them
together turns columns into rows:

```python
return type(self)([list(row) for row in zip(*self.data)])
```

`zip` returns tuples, so the `list()` conversion is necessary.

## Matrix Multiplication

Addition and subtraction are elementwise — zip over rows, zip over elements within
each row. Multiplication is different: each element of the result is the dot product
of a row from the left matrix with a column from the right matrix.

For a `(m×n)` matrix times a `(n×p)` matrix, the shape check is `self.shape[1] == m.shape[0]`
and the result is `(m×p)`:

```python
for row in self.data:
    new_row = []
    for j in range(m.shape[1]):
        col = [r[j] for r in m.data]
        new_row.append(sum(a * b for a, b in zip(row, col)))
    ret.append(new_row)
```

## Vector Inherits from Matrix

`Vector` subclasses `Matrix` and adds one validation: the data must be either a row
vector (one row, many columns) or a column vector (many rows, one column). Anything
else raises an error:

```python
if self.shape[0] != 1 and self.shape[1] != 1:
    raise ValueError("vector is invalid")
```

## Polymorphic Returns with type(self)

All operations return `type(self)(ret)` rather than `Matrix(ret)`. This means that
when a `Vector` calls `__add__`, the result is also a `Vector`, not a plain `Matrix`.
Without this, vector arithmetic would silently downgrade the return type.

## Circular Import

`Matrix` cannot import `Vector` and `Vector` cannot import `Matrix` — each imports
the other, creating a deadlock. The fix: since `Vector` is a subclass of `Matrix`,
it is already an instance of `Matrix`. The `isinstance(m, Matrix)` check in `__mul__`
catches both without needing to import `Vector` at all.