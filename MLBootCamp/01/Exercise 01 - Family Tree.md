Class inheritance: a parent class defines shared state and behaviour, child classes
extend it. This is the standard Python OOP pattern and the first time `super()` comes
up.

- `super().__init__(...)` calls the parent class's `__init__` — always do this first in a child's `__init__`
- `__dict__` is the instance's attribute dictionary — every attribute you set on `self` ends up there
- `__doc__` is the class's docstring, accessible as a string at runtime
- Child classes inherit all parent methods; they can override them or add new ones

## The Parent Class

```python
class GotCharacter:
    def __init__(self, first_name=None, is_alive=True):
        self.first_name = first_name
        self.is_alive = is_alive
```

Nothing special here — just stores the two attributes shared by all characters.

## A Child Class

`super().__init__()` passes the relevant arguments up to `GotCharacter.__init__`
so it can set `self.first_name` and `self.is_alive`. The child then sets its own
additional attributes:

```python
class Stark(GotCharacter):
    """A class representing the Stark family. Or when bad things happen to good people."""

    def __init__(self, first_name=None, is_alive=True):
        super().__init__(first_name=first_name, is_alive=is_alive)
        self.family_name = "Stark"
        self.house_words = "Winter is Coming"

    def print_house_words(self):
        print(self.house_words)

    def die(self):
        self.is_alive = False
```

After `__init__` runs, `arya.__dict__` will be:

```python
{'first_name': 'Arya', 'is_alive': True, 'family_name': 'Stark', 'house_words': 'Winter is Coming'}
```

`__dict__` is just a regular dict — you can inspect it, iterate it, and add to it
dynamically (though usually you shouldn't outside of `__init__`).

## Why super() and Not GotCharacter.__init__()

Calling `GotCharacter.__init__(self, ...)` directly works but breaks if you ever use
multiple inheritance. `super()` follows the MRO (Method Resolution Order) correctly
and is the idiomatic way to call parent methods in Python.

## Class Docstrings

The docstring goes immediately after the `class` line, triple-quoted. It becomes the
class's `__doc__` attribute and is accessible on both the class and its instances:

```python
print(Stark.__doc__)      # "A class representing the Stark family. ..."
arya = Stark("Arya")
print(arya.__doc__)       # same string
```

This is why the exercise checks `arya.__doc__` — the docstring is on the class but
instances inherit it.

## isinstance() for Type Checking

To check if a variable is an instance of a class or any of its subclasses:

```python
arya = Stark("Arya")
isinstance(arya, Stark)         # True
isinstance(arya, GotCharacter)  # True — because Stark inherits from GotCharacter
isinstance(arya, str)           # False
```

This is important for `add_recipe` in ex00 and comes up constantly in OOP exercises.
