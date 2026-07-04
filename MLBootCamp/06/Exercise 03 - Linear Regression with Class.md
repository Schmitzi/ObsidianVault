## Objective

Package everything from ex00–ex02 into a single reusable class, `MyLinearRegression`, mirroring the shape of `sklearn.linear_model.LinearRegression`. No new math — but a lot of design decisions about how state (`thetas`, `alpha`, `max_iter`) should live on `self` instead of being passed around as loose function arguments.

## The Skeleton

```python
class MyLinearRegression():
    def __init__(self, thetas, alpha=0.001, max_iter=1000):
        self.alpha = alpha
        self.max_iter = max_iter
        self.thetas = thetas
```

Required methods: `fit_(self, x, y)`, `predict_(self, x)`, `loss_elem_(self, y, y_hat)`, `loss_(self, y, y_hat)`.

## Why This Matters

This is the shift from "a pile of functions that happen to work together" to "an object that owns its own state." It sounds cosmetic, but it exposes real bugs that pure functions never would — because now something can mutate `self.thetas` between calls, and every method needs to defend itself against that, not just against bad arguments at the door.

## Real Bugs Hit While Building This (and the fixes)

### 1. Signature mismatch — methods still expected `theta`/`alpha`/`max_iter` as params

The first draft kept `fit_(self, x, y, theta, alpha, max_iter)` and `predict_(self, x, theta)` — but the subject's own test calls `lr2.fit_(x, y)` and `lr1.predict_(x)`, with no theta argument at all. **Fix:** drop the parameters entirely and read `self.thetas` / `self.alpha` / `self.max_iter` instead.

### 2. `fit_` computed a new theta but never stored it

```python
# WRONG — computes locally, self.thetas never updates
for _ in range(max_iter):
    theta = theta - alpha * gradient(x, y, theta)
return theta

# RIGHT — mutates self.thetas each iteration
for _ in range(self.max_iter):
    self.thetas = self.thetas - self.alpha * self.gradient(x, y, self.thetas)
return np.array(self.thetas)
```

The test checks `lr2.thetas` _after_ calling `fit_` without capturing the return value — so `fit_` has to mutate `self.thetas` in place, not just return a value.

### 3. `__init__`-only validation doesn't protect against later mutation

Adding a shape check inside `__init__` (`if thetas.shape != (2,1): raise ValueError`) only guards construction time. Nothing stops `lr.thetas = "garbage"` afterward, and `__init__` never runs again to catch it. **Lesson:** validation needs to live in every method that touches `self.thetas`, not just the constructor.

### 4. Validation _ordering_ bug — `len()`/`.ndim` before `isinstance`

```python
# WRONG — crashes if self.thetas is an int (ints have no len())
if len(x) == 0 or len(self.thetas) == 0:
    return None
if not isinstance(self.thetas, np.ndarray):
    return None

# RIGHT — type check first, always
if not isinstance(self.thetas, np.ndarray):
    return None
if len(x) == 0 or len(self.thetas) == 0:
    return None
```

**Standing rule going forward:** `isinstance` → `.ndim`/`.shape` → `len()`. Never ask "how big is this?" before confirming "is this even the right type?"

### 5. Checking the wrong variable

```python
def gradient(self, x, y, theta):
    ...
    if not isinstance(self.thetas, np.ndarray):   # WRONG — checks self.thetas
        return None
    if theta.ndim != 2 ...                          # but uses the theta *parameter*
```

`gradient` takes `theta` as its own parameter — the check should validate `theta`, not `self.thetas`. They happen to be the same object when called from inside `fit_`, which is exactly why this bug hid successfully until tested directly with a mismatched `theta`.

### 6. `mse_` needed to be `@staticmethod`

The subject calls it as `MyLR.mse_(Yscore, Y_model1)` — on the **class**, with only two arguments, no instance. A regular method `def mse_(self, y, y_hat)` would try to slot `Yscore` into `self`. Since `mse_` never touches `self.thetas`/`self.alpha`, it qualifies as a `@staticmethod`. `add_intercept` got the same treatment later for the same reason (doesn't touch instance state).

### 7. `loss_` vs `mse_` — not the same formula

- `loss_` = **half** mean squared error: `sum(J_elem) / (2*m)`
- `mse_` = **plain** mean squared error: `sum(J_elem) / m`

Both exist deliberately, for different call sites later in the module — don't try to collapse them into one function.

## Common Mistakes (recap)

- `and`/`or` operator precedence: `and` binds tighter than `or`, so `a and b and c or d` parses as `(a and b and c) or d` — if `d` alone can be true, the whole `and`-chain is bypassed. Always parenthesize explicitly when combining.
- `isinstance` before `.ndim`/`.shape`/`len()` — every time, no exceptions.
- When a method takes a parameter with the same "meaning" as an instance attribute (e.g. `theta` vs `self.thetas`), double check every validation line is checking the _parameter_, not the attribute, unless that's genuinely intended.