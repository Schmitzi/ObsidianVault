## Exercise 05 - TinyStatistician

Implementing descriptive statistics from scratch using for-loops. The formulas are
given directly in the subject — the exercise is about translating mathematical notation
into Python accurately, not deriving the formulas.

- Mean: sum of all values divided by count
- Variance: mean of squared deviations from the mean
- Std: square root of variance
- Median and quartiles require sorting first, then index arithmetic

## Mean

The straightforward translation of `μ = Σxᵢ / m`:

```python
def mean(self, x):
    if len(x) == 0:
        return None
    total = 0.0
    for val in x:
        total += val
    return total / len(x)
```

The exercise says to use a for-loop explicitly — don't use `sum()`.

## Variance

Variance is the mean of squared deviations from the mean: `σ² = Σ(xᵢ - μ)² / m`.
Compute the mean first, then sum the squared differences:

```python
def var(self, x):
    if len(x) == 0:
        return None
    mu = self.mean(x)
    total = 0.0
    for val in x:
        total += (val - mu) ** 2
    return total / len(x)
```

## Standard Deviation

`σ = sqrt(σ²)` — just the square root of variance. Use `x ** 0.5` to avoid importing
math (though `import math; math.sqrt(v)` is fine too):

```python
def std(self, x):
    if len(x) == 0:
        return None
    v = self.var(x)
    return v ** 0.5
```

## Median

Sort the data, then find the middle element. For even-length lists, the median is
the mean of the two middle elements:

```python
def median(self, x):
    if len(x) == 0:
        return None
    sorted_x = sorted(x)
    m = len(sorted_x)
    mid = m // 2
    if m % 2 == 1:
        return float(sorted_x[mid])
    else:
        return (sorted_x[mid - 1] + sorted_x[mid]) / 2.0
```

For `[1, 42, 300, 10, 59]` → sorted: `[1, 10, 42, 59, 300]` → middle index 2 → `42.0`.

## Quartiles

Q1 is the median of the lower half, Q3 is the median of the upper half. Defining
"lower" and "upper" half with or without the median is a detail — the expected output
`[10.0, 59.0]` for `[1, 42, 300, 10, 59]` guides which convention to use.

Sorted: `[1, 10, 42, 59, 300]` → lower half `[1, 10]`, upper half `[59, 300]`

```python
def quartile(self, x):
    if len(x) == 0:
        return None
    sorted_x = sorted(x)
    m = len(sorted_x)
    mid = m // 2
    lower = sorted_x[:mid]
    if m % 2 == 1:
        upper = sorted_x[mid + 1:]
    else:
        upper = sorted_x[mid:]
    return [float(self.median(lower)), float(self.median(upper))]
```

For 5 elements: lower = `[1, 10]`, upper = `[59, 300]`. Median of `[1, 10]` = `5.5`...
but the expected output is `[10.0, 59.0]`. This suggests a different split: Q1 uses
index `m // 4` and Q3 uses index `3 * m // 4` on the sorted array directly:

```python
def quartile(self, x):
    if len(x) == 0:
        return None
    sorted_x = sorted(x)
    m = len(sorted_x)
    q1 = float(sorted_x[m // 4])
    q3 = float(sorted_x[3 * m // 4])
    return [q1, q3]
```

For `[1, 10, 42, 59, 300]` (m=5): `sorted_x[1]` = 10, `sorted_x[3]` = 59 → `[10.0, 59.0]`. ✓

## Note on Population vs Sample Statistics

The formulas here use population variance (`/ m`), not sample variance (`/ (m-1)`).
Population variance is used when you have the entire dataset, not a sample. NumPy's
`np.var()` defaults to population variance (`ddof=0`); `ddof=1` gives sample variance.
The exercise uses population variance — dividing by `m`, not `m-1`.
