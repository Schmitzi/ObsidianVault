# Exercise 02 - The Vector

Implementing a class with arithmetic operator overloading via Python's dunder methods.
The point is to make `v1 + v2`, `v1 * 5`, `5 * v1` etc. all work naturally — the
same way built-in types do.

- Dunder methods (double-underscore) are what Python calls when you use operators on objects
- `__add__` handles `a + b`; `__radd__` handles `b + a` when `b` doesn't know how to add `a`
- `__repr__` and `__str__` should be identical here — both show the vector's values
- Shape is `(1, n)` for row vectors, `(n, 1)` for column vectors

## The Data Layout

Row vector: one outer list, one inner list with all the floats — `[[1., 2., 3.]]`
Column vector: one outer list, each inner list has exactly one float — `[[1.], [2.], [3.]]`

The shape is derived from the values. Detecting which type you have:

```python
def __init__(self, values):
    if isinstance(values, int):
        self.values = [[float(i)] for i in range(values)]
    elif isinstance(values, tuple):
        a, b = values
        if a > b:
            raise ValueError("In Vector((a,b)), a must be <= b")
        self.values = [[float(i)] for i in range(a, b)]
    else:
        self.values = values

    # Determine shape
    if len(self.values) == 1:
        self.shape = (1, len(self.values[0]))   # row: (1, n)
    else:
        self.shape = (len(self.values), 1)      # column: (n, 1)
```

## The Dunder Methods

All arithmetic operations need to return a new `Vector`, not modify `self` in place.

```python
def __add__(self, other):
    if not isinstance(other, Vector) or self.shape != other.shape:
        raise ValueError("Vectors must have the same shape to add")
    if self.shape[0] == 1:  # row
        return Vector([[a + b for a, b in zip(self.values[0], other.values[0])]])
    else:  # column
        return Vector([[a[0] + b[0]] for a, b in zip(self.values, other.values)])

def __radd__(self, other):
    return self.__add__(other)

def __sub__(self, other):
    if not isinstance(other, Vector) or self.shape != other.shape:
        raise ValueError("Vectors must have the same shape to subtract")
    if self.shape[0] == 1:
        return Vector([[a - b for a, b in zip(self.values[0], other.values[0])]])
    else:
        return Vector([[a[0] - b[0]] for a, b in zip(self.values, other.values)])

def __rsub__(self, other):
    return self.__sub__(other)

def __mul__(self, scalar):
    if not isinstance(scalar, (int, float)):
        raise NotImplementedError("Multiplication only defined for scalar")
    if self.shape[0] == 1:
        return Vector([[x * scalar for x in self.values[0]]])
    else:
        return Vector([[x[0] * scalar] for x in self.values])

def __rmul__(self, scalar):
    return self.__mul__(scalar)

def __truediv__(self, scalar):
    if not isinstance(scalar, (int, float)):
        raise NotImplementedError("Division only defined for scalar")
    if scalar == 0:
        raise ZeroDivisionError("division by zero")
    return self.__mul__(1.0 / scalar)

def __rtruediv__(self, scalar):
    raise NotImplementedError("Division of a scalar by a Vector is not defined here.")
```

## Why __radd__ Exists

When Python evaluates `5 * v1`, it first tries `int.__mul__(5, v1)`. Since `int`
doesn't know about `Vector`, it returns `NotImplemented`. Python then tries the
reflected method `v1.__rmul__(5)`. Without `__rmul__`, `5 * v1` would fail even
if `v1 * 5` works. The `r` prefix means "right operand" — your object is on the
right side of the operator.

## .T() Transpose

Swapping row ↔ column means restructuring the values:

```python
def T(self):
    if self.shape[0] == 1:  # row → column
        return Vector([[x] for x in self.values[0]])
    else:  # column → row
        return Vector([[x[0] for x in self.values]])
```

## .dot() Product

Sum of element-wise products. Both vectors must have the same shape:

```python
def dot(self, other):
    if self.shape != other.shape:
        raise ValueError("Vectors must have the same shape for dot product")
    if self.shape[0] == 1:
        return sum(a * b for a, b in zip(self.values[0], other.values[0]))
    else:
        return sum(a[0] * b[0] for a, b in zip(self.values, other.values))
```

## __str__ and __repr__

Both should show the same output. `__repr__` is called in the REPL when you type
a variable name without `print`. Making them identical satisfies the requirement:

```python
def __str__(self):
    return f"Vector({self.values})"

def __repr__(self):
    return self.__str__()
```
