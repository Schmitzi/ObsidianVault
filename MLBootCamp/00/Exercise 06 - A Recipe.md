A nested dictionary structure with a menu-driven REPL. The data structure part is
straightforward; the interesting challenge is writing a loop that keeps running,
handles bad input gracefully, and terminates cleanly.

- A dict whose values are also dicts is called a nested dictionary — the natural fit for structured records
- `input()` blocks until the user hits Enter, returning the string they typed
- The REPL loop pattern: while True + handle input + break on quit

## The Data Structure

Each recipe is itself a dict. The outer dict maps recipe names to recipe dicts:

```python
cookbook = {
    "Sandwich": {
        "ingredients": ["ham", "bread", "cheese", "tomatoes"],
        "meal": "lunch",
        "prep_time": 10
    },
    "Cake": {
        "ingredients": ["flour", "sugar", "eggs"],
        "meal": "dessert",
        "prep_time": 60
    },
    "Salad": {
        "ingredients": ["avocado", "arugula", "tomatoes", "spinach"],
        "meal": "lunch",
        "prep_time": 15
    }
}
```

Accessing nested values: `cookbook["Cake"]["prep_time"]` gives `60`.

## The Helper Functions

```python
def print_recipe_names(cookbook):
    for name in cookbook:
        print(name)

def print_recipe(cookbook, name):
    if name not in cookbook:
        print(f"Recipe '{name}' not found.")
        return
    r = cookbook[name]
    print(f"Recipe for {name}:")
    print(f"  Ingredients list: {r['ingredients']}")
    print(f"  To be eaten for {r['meal']}.")
    print(f"  Takes {r['prep_time']} minutes of cooking.")

def delete_recipe(cookbook, name):
    if name in cookbook:
        del cookbook[name]

def add_recipe(cookbook):
    name = input(">>> Enter a name:\n")
    ingredients = []
    print(">>> Enter ingredients:")
    while True:
        item = input()
        if item == "":
            break
        ingredients.append(item)
    meal = input(">>> Enter a meal type:\n")
    prep_time = int(input(">>> Enter a preparation time:\n"))
    cookbook[name] = {"ingredients": ingredients, "meal": meal, "prep_time": prep_time}
```

## The REPL Loop

The key requirement is that the program cannot crash on bad input — wrap the menu
dispatch in a try/except or just check the value explicitly:

```python
def main():
    print("Welcome to the Python Cookbook !")
    while True:
        print("\nList of available options:")
        print("  1: Add a recipe")
        print("  2: Delete a recipe")
        print("  3: Print a recipe")
        print("  4: Print the cookbook")
        print("  5: Quit")
        choice = input("\nPlease select an option:\n>> ")
        if choice == "1":
            add_recipe(cookbook)
        elif choice == "2":
            name = input("Please enter a recipe name to delete:\n>> ")
            delete_recipe(cookbook, name)
        elif choice == "3":
            name = input("Please enter a recipe name to get its details:\n>> ")
            print_recipe(cookbook, name)
        elif choice == "4":
            print_recipe_names(cookbook)
        elif choice == "5":
            print("Cookbook closed. Goodbye !")
            break
        else:
            print("Sorry, this option does not exist.")
```

The `else` branch at the bottom handles any garbage input without crashing. The
loop continues until the user picks 5, at which point `break` exits the `while True`.

## Gotcha: Mutable Default Arguments

Don't do `def add_recipe(cookbook={})` — mutable default arguments in Python are
created once and shared across all calls. Always pass the dict explicitly.
