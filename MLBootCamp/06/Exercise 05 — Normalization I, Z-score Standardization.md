## Objective

Implement `zscore(x)`: rescale a dataset so it has mean 0 and standard deviation 1.

## The Formula

```
x′⁽ⁱ⁾ = (x⁽ⁱ⁾ − μ) / σ
```

- `μ` = mean of `x`
- `σ` = standard deviation of `x`

## ⚠️ Subject Inconsistency — Formula vs. Worked Example

The subject's **written formula** for variance uses `m−1` in the denominator (sample variance). But the subject's **worked example numbers** only match variance computed with plain `m` in the denominator (population variance):

```python
X = np.array([0, 15, -9, 7, 12, 3, -21])
# ddof=1 (m-1, matches stated formula):  [-0.0798, 1.1173, -0.7981, ...]  <- does NOT match expected
# ddof=0 (m,   matches worked example):  [-0.0862, 1.2068, -0.8620, ...]  <- matches expected exactly
```

**Lesson:** when a subject's stated formula and its own worked example disagree, trust the worked example — that's what's actually being graded/checked against. Worth flagging in `TinyStatistician.var`: divide by `len(x)`, not `len(x) - 1`.

## Implementation Pattern

```python
def zscore(x):
    mu = ts.mean(x)
    sigma = ts.std(x)
    xi = []
    for i in x.flatten():
        xi.append((i - mu) / sigma)
    return np.array(xi)
```

## Real Bugs Hit While Building This

### 1. Forgot to `.append()` inside the loop

```python
# WRONG — overwrites sigma every iteration, only the LAST element survives
for i in x:
    mu = i - ts.mean(x)
    sigma = mu / ts.std(x)
return np.array(sigma)

# RIGHT
xi = []
for i in x:
    xi_pre = i - ts.mean(x)
    xi.append(xi_pre / ts.std(x))
return np.array(xi)
```

Symptom: output was a single float instead of an array of 7 values. Also a naming trap — calling the per-element deviation `mu`/`sigma` (names that conventionally mean "the dataset mean" / "the dataset std") made the bug harder to spot by eye.

### 2. Recomputing mean/std on every loop iteration

Not a correctness bug, but a performance one: calling `ts.mean(x)` and `ts.std(x)` fresh inside the loop is O(n²) work for something that only needs to be computed once. Better: hoist `mu = ts.mean(x)` and `sigma = ts.std(x)` outside the loop.

### 3. `TinyStatistician` methods broke on 2D input

```python
Y = np.array([2, 14, -13, 5, 12, 4, -19]).reshape((-1, 1))  # shape (7, 1)
```

Iterating `for i in x:` over a 2D array yields _rows_ (each a length-1 array), not scalars. This caused `ret += i` inside `mean()` to accumulate a numpy array instead of a plain number, which then crashed on in-place division (`ret /= len(x)`) with a dtype-casting error. **Fix**, applied consistently across `mean`, `median`, `quartile`, `percentile`, `var`:

```python
if not isinstance(x, (np.ndarray, list)):
    raise TypeError("wrong type")
x = np.array(x).flatten()      # unconditional — normalizes 1D, 2D, and list input alike
```

A near-miss first attempt only flattened when `x` was a `list`, guarded inside `if not isinstance(x, np.ndarray):` — which meant an already-2D `ndarray` (like `Y`) never got flattened at all, since it skipped that branch entirely. The fix needed to be **unconditional**, not nested inside a "was it not already an array" check.

### 4. Output shape depends on input shape (not caught until explicitly checked)

`zscore(X)` (1D input) returns shape `(7,)`. `zscore(Y)` (2D `(7,1)` input) returned shape `(7,1)` — nested — because each appended element was itself a 1-element array (row from the 2D iteration), not a scalar. Fixed by flattening `x` at the top of `zscore` before the loop, same pattern as inside `TinyStatistician`.

## Why This Matters

Z-score standardization doesn't just make numbers "nicer" — it's what makes gradient descent behave well once features have different natural scales (e.g. one feature in the 0–1 range, another in the tens of thousands). A single learning rate can't serve both scales well; normalizing first means one `alpha` works for the whole feature set. This becomes essential once multivariate regression (many features) shows up.

**Correction to a common intuition:** z-score does _not_ bound values to `[-1, 1]`. That's min-max normalization's job (ex06). Z-score just centers the mean at 0 and scales to unit variance — values commonly land around ±2 to ±3 for typical data, with no hard ceiling.

## Design Decision

`zscore` is kept as a **free-standing function**, not a method on `TinyStatistician` — `TinyStatistician`'s existing methods (`mean`, `median`, `var`, `std`, `quartile`, `percentile`) all reduce a dataset down to a single summary number. `zscore` does the opposite: it takes a dataset and returns a whole new dataset, transformed. Different enough in kind to warrant staying separate. Also deliberately _not_ grouped into a speculative "Preprocessor" class yet — better to wait until `minmax` (ex06) exists too before deciding on a shared abstraction, rather than guessing at structure early.

## Common Mistakes (recap)

- Loop that computes a value but never appends it to a list — silently returns only the last iteration's result. Watch for this specifically when a loop variable name looks like it "should" be building up a running result.
- Flatten defensively and **unconditionally** at the top of any function that loops over `x` element-by-element, regardless of whether the input arrived as list, 1D array, or 2D column vector.
- When a subject's formula and its own example disagree, verify against the example.