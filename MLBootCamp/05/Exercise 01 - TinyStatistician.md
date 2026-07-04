A class implementing basic descriptive statistics from scratch — no NumPy statistical
functions allowed. The point is to translate mathematical formulas directly into code
and understand what each metric actually computes.

## Mean

Sum all elements and divide by the count. Simple, but the foundation everything else
builds on:

```
x̄ = Σxᵢ / m
```

## Median

Sort the list, then pick the middle element. For even-length lists, average the two
middle elements:

```python
lst = sorted(x)
n = len(lst)
if n % 2 == 0:
    return float((lst[n // 2 - 1] + lst[n // 2]) / 2)
else:
    return float(lst[n // 2])
```

## Percentile

The percentile formula uses linear interpolation between the two surrounding elements:

```python
pos = p / 100 * (len(lst) - 1)
integer_part = int(pos)
frac = pos - integer_part
return lst[integer_part] + frac * (lst[integer_part + 1] - lst[integer_part])
```

The `p / 100 * (n - 1)` maps the percentile to an exact position in the sorted list.
The fractional part controls how far between the two surrounding values the result sits.

## Variance and Standard Deviation

The subject formula uses `m - 1` in the denominator (sample variance), but the provided
example outputs match population variance (denominator `m`). The example outputs are
authoritative — divide by `len(x)`:

```python
m = self.mean(x)
res = sum((xi - m) ** 2 for xi in x)
return float(res / len(x))
```

Standard deviation is just the square root of variance. Since `math.sqrt` is not
available without importing, implement Newton's method:

```python
def sqrt(self, n):
    guess = n / 2.0
    while True:
        better = (guess + n / guess) / 2.0
        if abs(better - guess) < 1e-10:
            break
        guess = better
    return guess
```

## Note on the Subject Inconsistency

The subject states the sample variance formula (divides by `m - 1`) but the example
output matches population variance (divides by `m`). The example outputs were used
as ground truth. Worth noting for peer evaluation.