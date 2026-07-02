A standalone function that finds the youngest male and female participants in the
Olympics for a given year. The core skill here is chaining filters in pandas.

- `df[condition]` filters rows where condition is True
- `&` chains multiple conditions (both must be True)
- `df['Col'].min()` returns the smallest value in a column

## youngest_fellah

```python
def youngest_fellah(df, year):
    year_df = df[df['Year'] == year]
    youngest_f = year_df[year_df['Sex'] == 'F']['Age'].min()
    youngest_m = year_df[year_df['Sex'] == 'M']['Age'].min()
    return {'f': youngest_f, 'm': youngest_m}
```

Break it down in two steps: first narrow to the right year, then within that filtered
DataFrame narrow to each gender and grab the minimum age.

You could write it as one chained filter instead:

```python
df[(df['Year'] == year) & (df['Sex'] == 'F')]['Age'].min()
```

The parentheses around each condition are required — `&` has higher precedence than
`==` in Python, so without them the expression is parsed wrong and you get a TypeError.

## Why .min() returns a float

Ages in `athlete_events.csv` are stored as floats (because the column contains NaN
values, and pandas can't store NaN in an integer column). So `.min()` returns something
like `13.0`, not `13`. That's correct — don't try to cast it.

## The key names are up to you

The subject says the dictionary keys must be "explicit and self-explanatory". The
example output uses `'f'` and `'m'`, which is fine. Longer names like
`'youngest_female'` and `'youngest_male'` are equally valid.
