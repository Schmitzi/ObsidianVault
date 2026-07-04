The same prediction as ex02, but implemented using the linear algebra trick instead
of a loop. The formula `ŷ = X'θ` replaces the per-example computation with a single
matrix multiplication.

## The Trick

Prepend a column of 1s to x to get X' (shape `m×2`), then multiply by θ (shape `2×1`):

```
ŷ = X'θ = [[1, x⁽¹⁾], ..., [1, x⁽ᵐ⁾]] · [[θ₀], [θ₁]]
         = [[θ₀ + θ₁x⁽¹⁾], ..., [θ₀ + θ₁x⁽ᵐ⁾]]
```

This produces the same result as the loop in ex02, but NumPy executes it in optimised
C code — much faster on large datasets.

## Implementation

```python
x_prime = add_intercept(x)
return np.dot(x_prime, theta).astype(float)
```

## Shape Difference from ex02

In ex02, `theta` was 1D — shape `(2,)`. In ex04, `theta` is 2D — shape `(2, 1)`.
This is required for the matrix multiplication to work: `(m×2) @ (2×1) = (m×1)`.

The output is also 2D — shape `(m, 1)` — rather than the 1D output from ex02.