## Objective

Implement `minmax(x)`: rescale a dataset so all values fall within `[0, 1]`.

## The Formula

```
x′⁽ⁱ⁾ = (x⁽ⁱ⁾ − min(x)) / (max(x) − min(x))
```

Unlike z-score, this one has a hard, guaranteed range: the smallest value in the dataset always maps to exactly `0`, the largest always maps to exactly `1`, and everything else falls proportionally in between.

## Implementation Pattern

```python
def minmax(x):
    if not isinstance(x, np.ndarray):
        return None
    x = x.flatten()
    if len(x) == 0:
        return None
    sigma = []
    for i in x:
        sigma_pre = i - min(x)
        sigma.append(sigma_pre / (max(x) - min(x)))
    return np.array(sigma, dtype=float)
```

No need for `TinyStatistician` here — Python's built-in `min()`/`max()` are enough; this exercise doesn't require reimplementing anything the way `mean`/`std` did.

## Real Bugs Hit While Building This

### 1. Same forgot-to-append bug as `zscore`

```python
# WRONG
sigma = []
for i in x:
    sigma_pre = i - min(x)
    sigma = sigma_pre / (max(x) - min(x))   # overwrites, doesn't build a list
return np.array(sigma, dtype=float)

# RIGHT
sigma.append(sigma_pre / (max(x) - min(x)))
```

Symptom: `minmax(X)` printed `[0.]` — a single value — instead of an array of 7. Recognizing this immediately once flagged, since it's the identical pattern already hit and fixed in `zscore` a few exercises earlier. Worth internalizing as a genuine recurring blind spot: **a loop that computes something per-iteration needs an explicit `.append()` (or equivalent) to actually retain each iteration's result** — plain assignment (`=`) inside a loop only ever keeps the last one.

### 2. Same 2D-input shape issue as `zscore`

`minmax(X)` on a `(7,1)`-shaped input returned a nested `(7,1)` result, not the flat `(7,)` shape the subject's expected output shows. Same root cause and same fix as `zscore`: flatten `x` immediately after the type check, before the loop runs, so every iteration variable `i` is guaranteed to be a plain scalar regardless of input shape.

```python
if not isinstance(x, np.ndarray):
    return None
x = x.flatten()
if len(x) == 0:
    return None
```

### 3. Redundant `x is None` check

```python
if not isinstance(x, np.ndarray) or x is None:
```

If `x` really is `None`, `isinstance(None, np.ndarray)` is already `False`, so the first clause alone already returns `None` before the second clause (`x is None`) could ever matter. The `or x is None` never actually does anything — `isinstance` alone fully subsumes it. Not wrong, just dead code; worth trimming for clarity, or consciously keeping only if there's a stylistic reason to be extra-explicit.

## Why This Matters

Min-max and z-score solve the same underlying problem (features on wildly different scales slowing or breaking gradient descent) but with different guarantees. Min-max gives a **hard bound** (`[0,1]`) — useful when you specifically need values confined to a known range (e.g. feeding into something that expects normalized inputs). Z-score gives no hard bound but is generally more robust to outliers, since a single extreme value doesn't compress the _entire_ rest of the dataset's range the way it would under min-max (one huge outlier as the max will squash every other value close to 0).

## Common Mistakes (recap)

- The forgot-to-`.append()` bug is now confirmed as a _pattern_, not a one-off — worth double-checking any future loop that "computes X and stores it" for whether the store step is actually inside the loop and actually accumulating.
- Flatten input shape defensively, every time, in any function that loops element-by-element over `x` — this is the second exercise in a row where a `(m,1)` column vector produced a differently-shaped (and initially wrong-looking) result than a flat `(m,)` array with identical values.
- Watch for redundant guard clauses that look like they're doing something but are fully subsumed by an earlier check — harmless, but worth trimming during review.