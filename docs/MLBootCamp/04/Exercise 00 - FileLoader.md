A simple class that loads a CSV dataset into a pandas DataFrame and displays rows
from it. The whole module leans on this class — every subsequent exercise imports it.

- `pd.read_csv(path)` reads a CSV file and returns a DataFrame
- `df.shape` returns a tuple `(rows, columns)` — shape[0] is rows, shape[1] is columns
- `df.head(n)` returns the first n rows; `df.tail(n)` returns the last n rows

## load

Opens the file, prints the dimensions, and returns the DataFrame:

```python
def load(self, path):
    try:
        df = pd.read_csv(path, low_memory=False)
        print(f"Loading dataset of dimensions {df.shape[0]} x {df.shape[1]}")
        return df
    except Exception:
        return None
```

`low_memory=False` tells pandas not to guess column dtypes mid-file — useful for large
datasets where a column might look like integers in the first chunk but contain strings
later. Without it you can get dtype warnings on big files like `athlete_events.csv`.

The whole method is wrapped in `try/except` so that a wrong path, a non-CSV file, or
any other bad input returns `None` instead of crashing.

## display

Prints the first or last n rows depending on the sign of n:

```python
def display(self, df, n):
    try:
        if not isinstance(df, pd.DataFrame) or not isinstance(n, int):
            return
        if n > 0:
            print(df.head(n))
        elif n < 0:
            print(df.tail(-n))
    except Exception:
        return
```

Two things worth noting:

`n == 0` is silently ignored — neither head nor tail is called. The subject doesn't
specify this case, so returning without printing is the safe choice.

`df.tail(-n)`: when n is negative (e.g. `-5`), `-n` is positive (`5`), so
`df.tail(5)` gives the last 5 rows. You negate it to flip the sign back.

## What is a DataFrame?

A DataFrame is pandas' core data structure — a table with labelled rows and columns.
Each column is a Series (a 1D array with an index). You'll interact with DataFrames
constantly from here on:

```python
df['Column']          # select one column → Series
df[['A', 'B']]        # select multiple columns → DataFrame
df[df['Age'] > 25]    # filter rows by condition → DataFrame
```

The index (row labels) starts at 0 by default and is preserved through filtering,
which means after a filter the index numbers won't be contiguous — you might see
rows 0, 7, 42, etc. That's expected.
