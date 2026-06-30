A class with four plotting methods, each taking a DataFrame and a list of feature
names. The key insight is that seaborn and matplotlib share the same figure — you
can mix them freely and always close with `plt.show()`.

- `plt.hist(series)` plots a histogram of one column
- `sns.kdeplot(series)` plots a smooth density curve
- `sns.pairplot(dataframe)` plots a grid of scatter plots with histograms on the diagonal
- `plt.boxplot(series)` plots a box-and-whisker plot
- `plt.show()` opens the window — works regardless of whether you used matplotlib or seaborn to draw

## Features are numerical columns only

Histograms, density curves, and box plots only make sense for numerical data. In
`athlete_events.csv` the useful numerical features are `Age`, `Height`, `Weight`,
and `Year`. Don't pass string columns like `Team` or `Medal` — they'll either error
or produce meaningless plots.

## histogram and density — loop over features

Each feature gets its own call, all drawn on the same figure, then shown together:

```python
def histogram(self, data, features):
    for f in features:
        plt.hist(data[f])
    plt.show()

def density(self, data, features):
    for f in features:
        sns.kdeplot(data[f])
    plt.show()
```

`data[f]` selects a single column as a Series — that's what both functions expect.

## pair_plot — no loop, pass a DataFrame

`sns.pairplot` is different from the others. It takes a whole DataFrame and builds
the full grid internally — you don't loop:

```python
def pair_plot(self, data, features):
    sns.pairplot(data[features])
    plt.show()
```

`data[features]` with a list selects multiple columns and returns a DataFrame (not a
Series). Passing a Series to `sns.pairplot` raises a TypeError.

## box_plot — loop over features

```python
def box_plot(self, data, features):
    for f in features:
        plt.boxplot(data[f])
    plt.show()
```

## seaborn sits on top of matplotlib

Seaborn draws onto matplotlib's current figure. This means:
- `plt.show()` always works after seaborn calls
- The two libraries can be mixed in the same plot
- Backend issues (like the xcb/Wayland crash) affect both equally

## Backend issues on Linux/Wayland

If matplotlib crashes with `qt.qpa.plugin: Could not load the Qt platform plugin "xcb"`,
the fix is to use the TkAgg backend instead:

```bash
MPLBACKEND=TkAgg python MyPlotLib.py
```

This requires `tk` to be installed at the system level (`sudo pacman -S tk`), not
via pip — it's an OS-level dependency, not a Python one. 
