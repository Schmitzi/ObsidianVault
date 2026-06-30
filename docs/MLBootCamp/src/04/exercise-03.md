# Exercise 03 - HowManyMedals

A function that returns a nested dictionary: for each year an athlete won medals,
how many of each type (Gold, Silver, Bronze) did they win?

- `df[df['Name'] == name]` filters to one athlete
- `df[df['Medal'].notna()]` keeps only rows where a medal was awarded
- `groupby` + `value_counts` or manual iteration over years

## how_many_medals

```python
def how_many_medals(df, name):
    athlete = df[(df['Name'] == name) & (df['Medal'].notna())]
    result = {}
    for year in athlete['Year'].unique():
        year_df = athlete[athlete['Year'] == year]
        result[year] = {
            'G': int((year_df['Medal'] == 'Gold').sum()),
            'S': int((year_df['Medal'] == 'Silver').sum()),
            'B': int((year_df['Medal'] == 'Bronze').sum()),
        }
    return result
```

## Why filter by notna() first

The Medal column contains `NaN` for every event where no medal was awarded. If you
don't filter these out before counting, your sums still work (NaN != 'Gold' is True),
but your year loop would include years where the athlete participated but won nothing.
The expected output shows 1998 as `{'G': 0, 'S': 0, 'B': 0}` for Kjetil Aamodt —
so actually the subject *does* include medal-less years. Filter by athlete name only,
not by medal, and let the counts naturally come out as 0.

```python
athlete = df[df['Name'] == name]   # all years, including medal-less ones
```

## The Medal column values

The three values in the dataset are exactly `'Gold'`, `'Silver'`, `'Bronze'` — full
words, capitalised. The output keys are `'G'`, `'S'`, `'B'` — you map between them
explicitly.

## int() cast

`(year_df['Medal'] == 'Gold').sum()` returns a numpy integer. Wrapping in `int()`
gives a plain Python int, which prints more cleanly. Not required but worth doing.
