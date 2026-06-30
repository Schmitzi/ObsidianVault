Two classes that reference each other: `Recipe` holds structured data with validation
in `__init__`, `Book` stores recipes and operates on them. The main Python concepts
are `__init__` validation patterns, `__str__` for human-readable output, and
`datetime` for timestamps.

- `__init__` is where you validate and reject bad data — raise or exit if constraints aren't met
- `__str__` is what `print()` and `str()` call on your object; return a formatted string from it
- `datetime.now()` gives the current local time as a `datetime` object
- Only `description` can be empty — everything else must be present and correctly typed

## Recipe Class

The validation in `__init__` needs to check types and value ranges. Exit on failure
after printing the error, since the exercise says "exit properly":

```python
from datetime import datetime
import sys

class Recipe:
    def __init__(self, name, cooking_lvl, cooking_time, ingredients,
                 description="", recipe_type=""):
        if not isinstance(name, str) or not name:
            print("Error: name must be a non-empty string")
            sys.exit(1)
        if not isinstance(cooking_lvl, int) or not (1 <= cooking_lvl <= 5):
            print("Error: cooking_lvl must be int between 1 and 5")
            sys.exit(1)
        if not isinstance(cooking_time, int) or cooking_time < 0:
            print("Error: cooking_time must be a non-negative int")
            sys.exit(1)
        if not isinstance(ingredients, list) or len(ingredients) == 0:
            print("Error: ingredients must be a non-empty list")
            sys.exit(1)
        if recipe_type not in ("starter", "lunch", "dessert"):
            print("Error: recipe_type must be 'starter', 'lunch', or 'dessert'")
            sys.exit(1)

        self.name = name
        self.cooking_lvl = cooking_lvl
        self.cooking_time = cooking_time
        self.ingredients = ingredients
        self.description = description
        self.recipe_type = recipe_type
```

## __str__

`__str__` must return a string — it cannot print directly. Build the string and return it:

```python
def __str__(self):
    """Returns the string to print with the recipe's info"""
    txt = (
        f"Recipe: {self.name}\n"
        f"Cooking level: {self.cooking_lvl}/5\n"
        f"Cooking time: {self.cooking_time} minutes\n"
        f"Ingredients: {', '.join(self.ingredients)}\n"
        f"Type: {self.recipe_type}\n"
        f"Description: {self.description if self.description else 'N/A'}"
    )
    return txt
```

When you call `print(recipe)`, Python calls `recipe.__str__()` and prints the result.
When you call `str(recipe)`, same thing. If you don't define `__repr__`, the REPL
falls back to the default object representation — define both if you want consistent
output everywhere.

## Book Class

`last_update` changes whenever a recipe is added. `recipes_list` maps meal types
to lists of `Recipe` objects:

```python
class Book:
    def __init__(self, name):
        self.name = name
        self.creation_date = datetime.now()
        self.last_update = datetime.now()
        self.recipes_list = {"starter": [], "lunch": [], "dessert": []}
```

## Book Methods

```python
def get_recipe_by_name(self, name):
    """Prints a recipe with the name `name` and returns the instance"""
    for recipes in self.recipes_list.values():
        for recipe in recipes:
            if recipe.name == name:
                print(recipe)
                return recipe
    print(f"Recipe '{name}' not found.")
    return None

def get_recipes_by_types(self, recipe_type):
    """Gets all recipe names for a given recipe_type"""
    if recipe_type not in self.recipes_list:
        return []
    return [recipe.name for recipe in self.recipes_list[recipe_type]]

def add_recipe(self, recipe):
    """Adds a recipe to the book and updates last_update"""
    if not isinstance(recipe, Recipe):
        raise TypeError("argument must be a Recipe instance")
    self.recipes_list[recipe.recipe_type].append(recipe)
    self.last_update = datetime.now()
```

## datetime

`datetime.now()` returns a datetime object representing the current moment.
You can print it directly — it has a readable `__str__`. For formatted output:

```python
from datetime import datetime

now = datetime.now()
print(now)                          # 2024-01-15 14:32:07.123456
print(now.strftime("%Y-%m-%d"))     # 2024-01-15
```

The `strftime` format codes are the same as in C's `strftime` — `%Y` for 4-digit
year, `%m` for month, `%d` for day, `%H:%M:%S` for time.
