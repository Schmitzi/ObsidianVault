## Objective

Apply `MyLinearRegression` to a real (tiny) dataset — `are_blue_pills_magics.csv` — compute MSE for given models, and produce two plots:

- **Figure VI.1**: actual data + hypothesis line + residuals
- **Figure VI.2**: loss curves `J(θ)` vs θ1, for several fixed θ0 values

## File Organization Decision

`my_linear_regression.py` stays lean (just the class, matching the ex03 turn-in). Everything ex04-specific — `FileLoader`, `MyPlotLib`, and (by choice) a duplicate copy of `MyLinearRegression` for convenience — lives in a separate `tools.py`. Trade-off accepted deliberately: two copies of the class to keep in sync, but it keeps the ex03 deliverable clean since this practice code isn't graded.

## Step 1 — MSE Sanity Check

```python
linear_model1 = MyLR(np.array([[89.0], [-8]]))
linear_model2 = MyLR(np.array([[89.0], [-6]]))
```

- `linear_model1` → MSE **57.6**
- `linear_model2` → MSE **232.2**

**Key realization:** "lower loss is better" is a _within-comparison_ statement — it's only meaningful when comparing models scored on the _same data_ with the _same loss function_. It doesn't generalize to comparing train vs. test loss, or models with very different numbers of parameters (that's the overfitting conversation, for later modules).

Neither `linear_model1` nor `linear_model2` was ever actually _trained_ — both are hand-picked example thetas from the subject, purely to test `mse_`. Running real `fit_()` on the same data found a genuinely better model: **MSE 36.3**, visibly tighter residuals in the plot. This is the difference between "an arbitrary guess with low-ish loss" and "gradient descent actually searching for the best fit."

## Step 2 — Figure VI.1 (`plot_with_loss`)

```python
@staticmethod
def plot_with_loss(x, y, theta):
    if x.ndim == 1 and y.ndim == 1 and theta.ndim == 2 and theta.shape == (2, 1):
        c = MyLinearRegression.add_intercept(x)
        y_hat = np.dot(c, theta).astype(float)
        ...
```

### Bug: operator precedence in the guard clause

```python
# WRONG — the `or` at the end bypasses the whole and-chain
if x.ndim == 1 and y.ndim == 1 and theta.ndim == 2 or theta.shape == (2, 1):

# RIGHT — every condition genuinely required
if x.ndim == 1 and y.ndim == 1 and theta.ndim == 2 and theta.shape == (2, 1):
```

Same root cause as the ex03 precedence bug — worth treating as a personal recurring blind spot, not a one-off typo.

### Bug: constructing a throwaway instance just to borrow a method

```python
# WRONG — builds an instance with garbage thetas just to call add_intercept
mlr = MyLinearRegression((0, 0))
c = mlr.add_intercept(x)

# RIGHT — add_intercept doesn't touch self, so make it @staticmethod
c = MyLinearRegression.add_intercept(x)
```

### Bug: passing the wrong object as theta

```python
# WRONG — passes the whole model instance, not its .thetas array
MyPL.plot_with_loss(Xpill, Yscore, linear_model2)

# RIGHT
MyPL.plot_with_loss(Xpill.flatten(), Yscore.flatten(), linear_model2.thetas)
```

Also note the `.flatten()` — `Xpill`/`Yscore` are `(m,1)` 2D per the class's convention everywhere else, but `plot_with_loss`'s guard explicitly wants 1D. Decision made: flatten at the call site rather than loosen the guard, to keep the plotting function's contract simple.

## Step 3 — Figure VI.2 (`plot_loss_curves`)

Sweep θ1 across a range for each of several fixed θ0 values, plotting one curve per θ0:

```python
@staticmethod
def plot_loss_curves(x, y, thetas, values, spread=3):
    theta0_center = thetas[0, 0]
    theta1_center = thetas[1, 0]
    theta0_values = [theta0_center + offset for offset in values]
    theta1_range = np.linspace(theta1_center - spread, theta1_center + spread, 100)
    model = MyLinearRegression(np.array([[0.0], [0.0]]))
    for t0 in theta0_values:
        losses = [model.loss_(y, model.predict_(x))
                  for t1 in theta1_range
                  if (model.thetas := np.array([[t0], [t1]])) is not None]
        plt.plot(theta1_range, losses, label=f'J(θ0={t0:.1f}, θ1)')
```

### Key realization: the "tightness" of the curves is a parameter choice, not a bug

A first attempt spread θ0 too widely (`±25` from the fitted center), producing loss values up to ~730 — much higher than the subject's illustrative figure (~140 max). **This wasn't a math error** — verified by computing real loss values by hand at each (θ0, θ1) combination. Narrowing the θ0 offsets (`±6` instead of `±25`) and the θ1 spread brought the plot's scale in line with the subject's example. The subject's figure uses arbitrary placeholder θ0 labels (`c0`...`c5`) — there is no "correct" numeric answer to match, only a reasonable illustrative spread centered on the actual fitted θ0.

## Why This Whole Exercise Matters

This is the first time loss stops being an abstract number and becomes something _visible_: a tighter-fitting line, a lower curve, smaller residual gaps. Seeing `fit_()` beat both hand-picked example thetas — with a real, lower MSE and a visibly better-fitting plot — is the moment gradient descent stops being "a formula I typed" and becomes "a search that actually works."

## Common Mistakes (recap)

- Same `and`/`or` precedence trap as ex03 — always parenthesize explicit boolean chains rather than relying on default precedence.
- Don't construct throwaway instances with invalid state just to reach a method that could be `@staticmethod` instead.
- Double-check _which object_ you're passing (a model instance vs. its `.thetas` array) — Python won't catch this until the attribute access fails downstream.
- Don't try to visually match illustrative subject figures pixel-for-pixel — check the _shape_ and _relative_ behavior of the curves, not exact axis ranges, unless the subject gives you exact numbers to reproduce.