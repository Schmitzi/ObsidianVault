The same half-MSE loss as ex06, but computed without any loops using the dot product formula. Both produce identical results — this version is faster on large datasets.

## The Formula

```
J(θ) = 1/(2m) * (ŷ - y) · (ŷ - y)
```

The dot product of a vector with itself is the sum of squared elements — equivalent to `Σ(ŷ⁽ⁱ⁾ - y⁽ⁱ⁾)²` without writing the sum explicitly.

## Implementation

```python
diff = y_hat - y
return float(np.dot(diff, diff) / (2 * len(y)))
```

## Common Mistake: Dotting the Wrong Things

```python
np.dot(y_hat, y)        # Wrong — cross-multiplies predictions with targets
np.dot(diff, diff)      # Right — squares the errors
```

The error vector must be dotted with itself, not with the original arrays.

## Shape Note

This version takes 1D arrays (shape `(m,)`) unlike ex06 which expected 2D `(m, 1)`. The dot product of two 1D arrays of the same length returns a scalar directly.