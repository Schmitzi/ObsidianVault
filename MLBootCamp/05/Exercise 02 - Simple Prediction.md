The first implementation of the linear hypothesis — computing predictions from theta
and x using a plain Python loop, no linear algebra tricks yet.

## The Hypothesis

```
ŷ⁽ⁱ⁾ = θ₀ + θ₁x⁽ⁱ⁾    for i = 1, ..., m
```

θ₀ is the intercept (shifts the line up or down), θ₁ is the slope (controls how
steeply the line rises). Together they define a line, and the hypothesis applies that
line to each input value to produce a prediction.

## Why θ Notation?

The letters `a` and `b` work for two parameters. The θ notation with numeric indices
scales to any number of features — θ₀, θ₁, θ₂, ... θ₂₄₆₈ all follow the same pattern
and make it clear which parameter pairs with which feature.

## Implementation

```python
y_hat = []
for i in range(len(x)):
    y_hat.append(float(theta[0] + theta[1] * x[i]))
return np.array(y_hat)
```

## Reading the Examples

The subject examples are worth thinking through:

- `theta = [5, 0]` → `ŷ = 5 + 0 * x = 5` for all x — a flat horizontal line
- `theta = [0, 1]` → `ŷ = 0 + 1 * x = x` — the identity line
- `theta = [5, 3]` → `ŷ = 5 + 3x` — slope 3, intercept 5
- `theta = [-3, 1]` → `ŷ = -3 + x` — shifts the identity line down by 3