# Exercise 00 - NumPyCreator

The goal here is to get comfortable with the most common ways to *create* NumPy arrays
from scratch or from existing Python data structures. Every method in `NumPyCreator` is
essentially a thin wrapper around a single NumPy function — the exercise is about
discovering which function maps to which input type.

- `np.array()` accepts lists, tuples, and nested versions of both
- `np.full(shape, value)` fills an array of a given shape with a constant
- `np.random.rand(shape)` fills with uniform random floats in [0, 1)
- `np.eye(n)` returns the n×n identity matrix
- All creation functions accept an optional `dtype=` argument

## Input Type Strictness

Several methods are strict about what they accept — passing the wrong container type
should return `None`, not raise an exception.

`from_list` only accepts `list`. If you pass a tuple of tuples, it returns `None`:

```python
npc.from_list(((1,2),(3,4)))   # None — it's a tuple, not a list
```

`from_tuple` only accepts `tuple`. A list returns `None`:

```python
npc.from_tuple(["a", "b", "c"])   # None — it's a list, not a tuple
```

`from_iterable` is the relaxed version — it accepts anything iterable (ranges, generators,
sets, etc.) and returns a flat 1D array of all elements.

The cleanest way to enforce this is with `isinstance`:

```python
if not isinstance(lst, list):
    return None
```

## Ragged Arrays and dtype Promotion

When you pass a nested list where the inner lists have different lengths, `np.array` cannot
build a rectangular array. Since NumPy 1.24+ this raises a `ValueError` by default — your
method should catch it and return `None`.

When inner lists mix types (integers and strings), NumPy promotes everything to the most
general dtype it can. A list mixing ints and strings becomes an array of strings:

```python
npc.from_list([[1,2,3],['a','b','c'],[6,4,7]])
# array([['1','2','3'], ['a','b','c'], ['6','4','7']], dtype='<U21')
```

The `<U21` dtype means Unicode strings of up to 21 characters. NumPy chose the width
automatically.

## from_shape and the Default Value

`from_shape` takes a shape tuple and a fill value that defaults to `0`:

```python
def from_shape(self, shape, value=0):
    return np.full(shape, value)
```

`np.zeros(shape)` would also work for the default case, but `np.full` handles both
the default and the custom-value case in one call.

## identity

`np.eye(n)` produces a square matrix with 1s on the diagonal and 0s elsewhere.
The values are floats by default (`1.`, `0.`), which is the expected output.

```python
npc.identity(4)
# array([[1., 0., 0., 0.],
#        [0., 1., 0., 0.],
#        [0., 0., 1., 0.],
#        [0., 0., 0., 1.]])
```

## The dtype Bonus

All NumPy creation functions accept `dtype=` as a keyword argument. Adding it as an
optional parameter to each method is just a matter of forwarding it:

```python
def from_list(self, lst, dtype=None):
    if not isinstance(lst, list):
        return None
    return np.array(lst, dtype=dtype)
```

When `dtype=None`, NumPy infers the type automatically — same as the non-bonus behaviour.
