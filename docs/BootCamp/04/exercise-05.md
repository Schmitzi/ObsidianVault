# # Exercise 05 - HowManyMedalsByCountry

A function that returns a nested dictionary of medal counts per year for a given
country. The main challenge is deduplication — team sports award the same medal to
every player on the team, so you need to count those as one medal, not one per athlete.

- `df['Col'].isin(list)` returns a boolean Series — True where the value is in the list
- `~` negates a boolean Series
- `pd.concat([df1, df2])` stacks two DataFrames back together
- `drop_duplicates(subset=[...])` removes rows where all listed columns match

## The team sport problem

The dataset has one row per athlete per event. In a team sport like Ice Hockey, if
Finland wins bronze, every player on the squad gets a row with `Medal == 'Bronze'`.
Counting rows directly gives you 20+ bronze medals for one team event.

The fix is to deduplicate team sport rows on `['Year', 'Sport', 'Medal']` — that
collapses all players from the same team result into one row:

```python
individual = medals[~medals['Sport'].isin(team_sports)]
team = medals[medals['Sport'].isin(team_sports)].drop_duplicates(
    subset=['Year', 'Sport', 'Medal'])
medals = pd.concat([individual, team])
```

Do this *after* filtering by country and notna(), but *before* counting medals.

## drop_duplicates takes a list, not separate calls

A common mistake is calling drop_duplicates once per column:

```python
medals.drop_duplicates(subset=['Year'])   # ❌ keeps only one row per year total
medals.drop_duplicates(subset=['Sport'])  # ❌ keeps only one row per sport total
```

You want one call with all three columns together — a row is only dropped if all
three columns match another row simultaneously:

```python
drop_duplicates(subset=['Year', 'Sport', 'Medal'])  # ✅
```

## Pandas operations return new DataFrames

`drop_duplicates` and filtering don't modify in place — they return a new DataFrame.
You must assign the result:

```python
medals.drop_duplicates(subset=[...])        # ❌ result thrown away
medals = medals.drop_duplicates(subset=[...])  # ✅
```

## The team_sports list must match the dataset exactly

The subject provides a `team_sports` list, but the sport names must match what's
actually in the CSV. `'Hockey'` in the list won't catch `'Ice Hockey'` in the data —
you get a KeyError-style miss where those rows fall through to the individual sports
bucket and get counted per athlete.

Always sanity-check suspicious counts. Finland's 49 bronze in 1998 was the giveaway —
checking that year's data revealed `'Ice Hockey'` as the culprit.

## Structure

```python
def how_many_medals_by_country(df, country):
    ctry = df[df['Team'] == country]
    medals = ctry[ctry['Medal'].notna()]

    individual = medals[~medals['Sport'].isin(team_sports)]
    team = medals[medals['Sport'].isin(team_sports)].drop_duplicates(
        subset=['Year', 'Sport', 'Medal'])
    medals = pd.concat([individual, team])

    result = {}
    for year in medals['Year'].unique():
        year_medals = medals[medals['Year'] == year]
        result[year] = {
            'G': len(year_medals[year_medals['Medal'] == 'Gold']),
            'S': len(year_medals[year_medals['Medal'] == 'Silver']),
            'B': len(year_medals[year_medals['Medal'] == 'Bronze'])
        }

    for year in ctry['Year'].unique():
        if year not in result:
            result[year] = {'G': 0, 'S': 0, 'B': 0}

    return dict(sorted(result.items()))
```

The second loop adds years where the country participated but won no medals, with
all counts set to zero.
