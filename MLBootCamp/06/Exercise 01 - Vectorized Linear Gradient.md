## Objective

Rewrite `simple_gradient` from Exercise 00 **without any for-loop**, using linear algebra instead. Same result, but ready to scale to large datasets.

## The Linear Algebra Trick: X′

Concatenate a column of 1's to the left of `x`:

```
X′ = [ 1   x⁽¹⁾ ]
     [ ⋮    ⋮   ]
     [ 1   x⁽ᵐ⁾ ]
```

This turns `hθ(x) = θ0 + θ1·x` into a single matrix multiplication: `hθ(x) = X′·θ`. That's exactly what `add_intercept(x)` builds.

## The Formula

```
∇(J) = (1/m) · X′ᵀ · (X′θ − y)
```

- `X′` has shape `(m, 2)`
- `X′ᵀ` has shape `(2, m)`
- `X′θ − y` has shape `(m, 1)` (the vector of prediction errors)
- Final result: shape `(2, 1)` — matches ∇(J)0 and ∇(J)1 in one shot

## Implementation Pattern

```python
def gradient(x, y, theta):
    X_prime = add_intercept(x)              # (m, 2)
    return (1/len(x)) * X_prime.T @ (X_prime @ theta - y)
```

## Why This Matters

This is the first real taste of **vectorization**: replacing an explicit loop with matrix operations that NumPy executes in optimized C code. The iterative version from ex00 does the exact same math, but the vectorized version is what actually scales to datasets with thousands or millions of rows — the for-loop version would grind to a halt.

## Common Mistakes

- **Forgetting to call `add_intercept`** before multiplying by `theta` — `x` alone is shape `(m, 1)`, but `theta` is `(2, 1)`; the shapes don't align for the dot product until `x` becomes `X′` of shape `(m, 2)`.
- Getting the transpose backwards — `X′ᵀ(X′θ − y)` vs `(X′θ − y)X′ᵀ` — shape errors here are the fastest way to catch it (matrix multiply will just refuse to run).
- Not reshaping `y` to `(m, 1)` before the subtraction — a shape `(m,)` array will silently broadcast in ways that don't match the intended math.