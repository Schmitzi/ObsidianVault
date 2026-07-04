## Objective

Understand and manipulate the notion of **gradient** in machine learning by writing `simple_gradient(x, y, theta)`: a function that computes the gradient of the loss function with a plain for-loop, one partial derivative per θ parameter.

## The Formula

For a univariate hypothesis `hθ(x) = θ0 + θ1·x`:

```
∇(J)0 = (1/m) Σ (hθ(x⁽ⁱ⁾) − y⁽ⁱ⁾)
∇(J)1 = (1/m) Σ (hθ(x⁽ⁱ⁾) − y⁽ⁱ⁾) · x⁽ⁱ⁾
```

- `∇(J)` is the gradient vector, shape `2 × 1`
- `x`, `y` are vectors of dimension `m` (number of training examples)
- `hθ(x⁽ⁱ⁾)` is the model's prediction ŷ⁽ⁱ⁾ for the i-th example

## Implementation Pattern

```python
def simple_gradient(x, y, theta):
    m = len(x)
    grad0 = sum(theta[0] + theta[1]*x[i] - y[i] for i in range(m)) / m
    grad1 = sum((theta[0] + theta[1]*x[i] - y[i]) * x[i] for i in range(m)) / m
    return np.array([[grad0], [grad1]])
```

## Why This Matters

The gradient tells the training loop **which direction and how far** to nudge each θ parameter to reduce the loss. It's the mechanical heart of gradient descent — before this exercise, θ0/θ1 were static guesses; after it, they become _adjustable_ based on how wrong the current model is.

## Common Mistakes

- **Shape mismatches**: `x`/`y` must be reshaped to `(m, 1)` column vectors before use — a bare 1D array of shape `(m,)` will silently behave differently in downstream matrix operations later in the module.
- **Forgetting the `1/m` normalization** — without it, the gradient magnitude scales with dataset size, which breaks a fixed learning rate `alpha` across datasets of different sizes.
- Mixing up which partial derivative gets multiplied by `x⁽ⁱ⁾` (∇(J)1) versus which doesn't (∇(J)0) — easy to transpose if copying the formula too quickly.