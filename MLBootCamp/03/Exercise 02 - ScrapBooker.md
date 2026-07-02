Four array manipulation operations, all implemented via NumPy slicing. No loops
required — every method is expressible as a slice or a stack. The coordinate system
used throughout: `(row, col)`, origin at top-left, zero-indexed.

- NumPy slicing: `array[start:stop]` excludes `stop`, `array[::n]` takes every n-th element
- `np.delete(array, indices, axis)` removes rows or columns by index
- `np.concatenate([a, b, ...], axis)` joins arrays along an axis
- `np.tile(array, reps)` repeats an array a given number of times along each axis

## crop

Cuts a rectangular region out of the array. `dim` is `(height, width)` of the desired
output; `position` is the `(row, col)` of the top-left corner.

```python
def crop(self, array, dim, position=(0,0)):
    r, c = position
    h, w = dim
    # guard: the requested region must fit inside the array
    if r + h > array.shape[0] or c + w > array.shape[1]:
        return None
    return array[r:r+h, c:c+w]
```

The slice `array[r:r+h, c:c+w]` reads as: rows from `r` up to (not including) `r+h`,
columns from `c` up to (not including) `c+w`.

If the combination of position and dim goes outside the array bounds, return `None`.
Also return `None` for invalid inputs (negative dims, wrong types, etc.).

## thin

Deletes every n-th element along an axis. `axis=0` deletes columns (vertical lines);
`axis=1` deletes rows (horizontal lines).

The indices to delete are `n-1, 2n-1, 3n-1, ...` — i.e. every n-th index starting
from index `n-1` (the first one to delete). Using `np.arange`:

```python
def thin(self, array, n, axis):
    if axis == 0:
        size = array.shape[1]   # number of columns
    elif axis == 1:
        size = array.shape[0]   # number of rows
    else:
        return None
    indices = np.arange(n - 1, size, n)
    return np.delete(array, indices, axis=axis)
```

Watch the axis naming carefully: `axis=0` in `np.delete` removes *rows*, and `axis=1`
removes *columns* — but the subject says `axis=0` means vertical (column deletion).
The subject's axis convention is the opposite of NumPy's for `np.delete`. You need to
flip: when the subject says `axis=0`, call `np.delete(..., axis=1)`.

## juxtapose

Places `n` copies of the array side by side along an axis. `np.concatenate` with a
list of `n` copies does this cleanly:

```python
def juxtapose(self, array, n, axis):
    if n <= 0 or axis not in (0, 1):
        return None
    return np.concatenate([array] * n, axis=axis)
```

`np.tile(array, reps)` is an alternative — `reps` is a tuple specifying repetitions
per axis. For `axis=1` with `n=3`: `np.tile(array, (1, 3))`.

## mosaic

Creates a grid of copies: `dim[0]` repetitions vertically, `dim[1]` horizontally.
`np.tile` is the natural fit here since it handles both axes at once:

```python
def mosaic(self, array, dim):
    if not isinstance(dim, tuple) or len(dim) != 2:
        return None
    return np.tile(array, dim)
```

`np.tile(array, (2, 3))` produces a grid with 2 rows of copies and 3 columns of
copies — i.e. the image tiled 2× vertically and 3× horizontally.

## Return None on Bad Input

All four methods must return `None` rather than raise exceptions. A safe pattern is to
wrap the body in a `try/except` and return `None` in the `except` block, *in addition
to* explicit guards for the cases you can anticipate:

```python
try:
    # main logic
except Exception:
    return None
```
