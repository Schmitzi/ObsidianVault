# Exercise 04 - SpatioTemporalData

A class wrapping the Olympics dataset that answers two directional queries: given a
location, when were games held there? Given a year, where were the games held?

- `df[df['Col'] == value]` filters rows by value
- `series.unique()` returns an array of deduplicated values
- `array.tolist()` converts a NumPy array to a plain Python list

## when and where — the same pattern, mirrored

Both methods follow identical steps, just with the filter column and the extraction
column swapped:

```python
def when(self, location):
    rows = self.df[self.df['City'] == location]
    return rows['Year'].unique().tolist()

def where(self, date):
    rows = self.df[self.df['Year'] == date]
    return rows['City'].unique().tolist()
```

`when`: filter by City → extract Year column → deduplicate → list  
`where`: filter by Year → extract City column → deduplicate → list

## Why unique() is necessary

The dataset has one row per athlete per event. The 1896 Athens games had 380 rows,
all with `City == 'Athina'`. Without `.unique()`, you'd return a list of 380 identical
strings. `.unique()` reduces that to `['Athina']`.

## The chain: filter → column → unique → tolist

Each step produces a different type:

```
self.df[self.df['City'] == location]   → DataFrame  (many rows, all columns)
rows['Year']                           → Series     (one column, many rows)
rows['Year'].unique()                  → np.ndarray (deduplicated values)
rows['Year'].unique().tolist()         → list       (plain Python list)
```

A common mistake is calling `.tolist()` on the DataFrame directly — DataFrames don't
have `.tolist()`. You need to select the column first to get a Series or array, then
convert.

## Column selection syntax

```python
rows['Year']        # ✅ selects the 'Year' column by name → Series
rows[rows['Year']]  # ❌ passes a Series of year values as column names → KeyError
```

The second form looks plausible but tries to use the *values* of the Year column as
column *names* to look up — which fails because `2004`, `1906`, etc. aren't column
names in the DataFrame.
