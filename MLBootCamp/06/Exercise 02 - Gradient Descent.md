## Objective

Implement `fit_(x, y, theta, alpha, max_iter)`: the actual training loop that uses the gradient (from ex01) to iteratively improve θ.

## The Algorithm

```
repeat max_iter times:
    compute ∇(J)
    θ0 := θ0 − α·∇(J)0
    θ1 := θ1 − α·∇(J)1
```

`alpha` (the learning rate) controls step size — without it, subtracting the raw gradient would overshoot the minimum wildly.

## Implementation Pattern

```python
def fit_(x, y, theta, alpha, max_iter):
    for _ in range(max_iter):
        theta = theta - alpha * gradient(x, y, theta)
    return theta
```

## Why This Matters

This is where "the model learns" stops being an abstraction. Gradient descent doesn't solve for the optimal θ analytically — it _walks toward_ the minimum of the loss function, one small step at a time, guided purely by the local slope (the gradient) at its current position. Two knobs govern how well this walk goes:

- **Learning rate too large** → the walk overshoots the minimum repeatedly and diverges; θ can blow up to `nan`.
- **Learning rate too small (but enough iterations)** → converges correctly, just slowly. Wastes compute but doesn't break.

## Common Mistakes

- **Not iterating enough** — `max_iter` needs to be "sufficient for convergence," which for real datasets can mean hundreds of thousands of iterations at a small `alpha`. There's no way to know the right number except trial and error (or checking convergence numerically, which comes in later modules).
- **theta becomes `nan`** — the subject explicitly warns this means alpha is too large; it's a fast diagnostic, not a mysterious crash.
- Forgetting that `fit_` here **returns a new theta** rather than mutating anything in place — this becomes a real design decision once `fit_` moves into a class in Exercise 03 (see that exercise's notes for how this bit us).