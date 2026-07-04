A utility function that prepends a column of 1s to a numpy array. It is used in
every subsequent exercise as the first step of the linear algebra prediction trick.

## Why 1s?

When you multiply a row `[1, x⁽ⁱ⁾]` by `[θ₀, θ₁]`, you get:

```
1 * θ₀ + x⁽ⁱ⁾ * θ₁ = θ₀ + θ₁x⁽ⁱ⁾
```

The 1 "carries" θ₀ through the matrix multiplication without needing to treat it
as a special case. This lets you compute the entire prediction vector in a single
`np.dot(X', theta)` call.

## Implementation

For a 1D input, each element becomes `[1, x[i]]`:

```python
rows = []
for i in range(len(x)):
    rows.append([1, x[i]])
return np.array(rows, dtype=float)
```

For a 2D input, each existing row gets a 1 inserted at position 0:

```python
for i in range(len(x)):
    row = list(x[i])
    row.insert(0, 1)
    rows.append(row)
return np.array(rows, dtype=float)
```

The branch on `x.ndim` determines which case applies.

## Output Shape

A 1D input of shape `(m,)` becomes `(m, 2)`. A 2D input of shape `(m, n)` becomes
`(m, n+1)`. The added column is always on the left.