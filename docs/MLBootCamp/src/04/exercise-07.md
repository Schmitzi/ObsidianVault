# Exercise 07 - Komparator

A class that compares how a numerical variable is distributed across different
subpopulations defined by a categorical variable. All three methods follow the same
pattern: group by category, plot the numerical variable, show once.

- `sns.boxplot(x=cat, y=num, data=df)` — one box per category, auto-positioned
- `sns.kdeplot(data=df, x=num, hue=cat)` — one density curve per category
- `sns.histplot(data=df, x=num, hue=cat)` — one histogram per category, overlapping

## The pattern — x, y, and hue

All three seaborn functions take the same arguments:

- `x` or `y` — the numerical variable being measured
- `hue` (or `x`/`y` for boxplot) — the categorical variable that defines subpopulations
- `data` — the full DataFrame

Seaborn handles the grouping, colouring, and positioning internally. No loop needed.

```python
def compare_box_plots(self, categorical_var, numerical_var):
    sns.boxplot(x=categorical_var, y=numerical_var, data=self.df)
    mpl.show()

def density(self, categorical_var, numerical_var):
    sns.kdeplot(data=self.df, x=numerical_var, hue=categorical_var)
    mpl.show()

def compare_histograms(self, categorical_var, numerical_var):
    sns.histplot(data=self.df, x=numerical_var, hue=categorical_var)
    mpl.show()
```

## Why MyPlotLib is "optional" — and why that makes sense

The subject lists `MyPlotLib.py` as an optional file to turn in. At first glance it
seems natural to reuse it here — you already built the plotting logic, why not use it?

The problem is that `MyPlotLib` is designed for a different task. Its methods loop
over a list of features and plot each one independently on the same figure. `Komparator`
needs something different: plot one numerical variable grouped by a categorical variable,
with seaborn handling the subpopulation separation automatically.

Trying to force `MyPlotLib` into `Komparator` leads to positioning issues — each
`box_plot` call draws at position 0, so subpopulations overlap instead of sitting
side by side. The `show=False` parameter trick gets you partway there, but you'd still
need to pass position indices through `MyPlotLib`, which breaks the abstraction.

The cleanest solution is to call seaborn directly in `Komparator` and use `MyPlotLib`
only for the `mpl.show()` call — or just call `plt.show()` directly and skip it
entirely. `MyPlotLib` is a good abstraction for ex06's use case; it's simply not the
right tool here.

## Reusing MyPlotLib as a plt replacement

One clean pattern that does work: create a module-level instance at the bottom of
`MyPlotLib.py`:

```python
mpl = MyPlotLib()
```

Then import it in `Komparator`:

```python
from MyPlotLib import mpl
```

This lets you call `mpl.show()` instead of `plt.show()` everywhere, keeping all
matplotlib/seaborn calls behind your own interface — even if `Komparator` does its
own drawing via seaborn directly.
