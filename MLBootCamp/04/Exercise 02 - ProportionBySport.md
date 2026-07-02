A function that answers: "what fraction of all female (or male) participants played
a given sport in a given year?" The tricky part is deduplication — an athlete who
competed in multiple events in the same sport should only be counted once.

- `df.drop_duplicates(subset=[...])` removes rows where all listed columns match
- `len(df)` or `df.shape[0]` counts rows

## proportion_by_sport

```python
def proportion_by_sport(df, year, sport, gender):
    year_gender = df[(df['Year'] == year) & (df['Sex'] == gender)]
    unique_athletes = year_gender.drop_duplicates(subset=['Name'])
    sport_athletes = unique_athletes[unique_athletes['Sport'] == sport]
    return len(sport_athletes) / len(unique_athletes)
```

## Why drop_duplicates matters

The dataset has one row per athlete per event. An athlete who competed in 3 swimming
events in the same year appears 3 times. If you count rows directly, you overcount
both the total and the sport-specific total — but not proportionally, so the ratio
would be wrong.

Dropping duplicates on `['Name']` first gives you one row per unique person. Do this
*before* filtering by sport, otherwise you'd lose people who appear in multiple sports.

The subject's hint says "drop duplicate sports people to count only unique ones —
beware to call the dropping function at the right moment." The right moment is after
filtering by year and gender, but before splitting by sport.

## The return value

The function returns a raw proportion (a float between 0 and 1), not a percentage.
`0.019` means roughly 1.9% — don't multiply by 100 unless you're displaying it.
