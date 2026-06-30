# Exercise 01 - args and kwargs?

`*args` and `**kwargs` let a function accept an arbitrary number of positional and
keyword arguments. This exercise uses them to dynamically set attributes on an object
instance — and to detect collisions between positional auto-names and explicit kwargs.

- `*args` collects extra positional arguments as a tuple
- `**kwargs` collects extra keyword arguments as a dict
- Positional args get auto-named `var_0`, `var_1`, ... based on their index
- If a kwarg key conflicts with an auto-generated name, return `None`

## *args and **kwargs

```python
def example(*args, **kwargs):
    print(args)    # tuple of positional args
    print(kwargs)  # dict of keyword args

example(1, 2, 3, a=10, b=20)
# args   = (1, 2, 3)
# kwargs = {'a': 10, 'b': 20}
```

You can use any names — `*args` and `**kwargs` are just convention. What matters
is the `*` and `**` unpacking operators.

## The Collision Problem

Positional args at index `i` get the name `var_i`. If a kwarg explicitly uses a
name like `var_0`, and there's also a positional arg that would get named `var_0`,
that's a collision — the expected output shows `None` (returned by `doom_printer`
when it receives `None`).

Looking at the failing test case:
```python
obj = what_are_the_vars(42, a=10, var_0="world")
# 42 → would be named var_0
# var_0="world" → also wants var_0
# CONFLICT → return None
```

## Implementation

```python
class ObjectC(object):
    def __init__(self):
        pass

def what_are_the_vars(*args, **kwargs):
    obj = ObjectC()
    # Check for collisions first
    for i in range(len(args)):
        auto_name = f"var_{i}"
        if auto_name in kwargs:
            return None   # collision detected
    # Set positional args as var_0, var_1, ...
    for i, value in enumerate(args):
        setattr(obj, f"var_{i}", value)
    # Set keyword args directly
    for key, value in kwargs.items():
        setattr(obj, key, value)
    return obj
```

`setattr(obj, name, value)` is equivalent to `obj.name = value` but works when
the attribute name is a variable (string). You can't write `obj.f"var_{i}"` — the
dot notation only works with literal names.

## Why Modify the Instance, Not the Class

The exercise specifically says to modify the instance, NOT the class. Setting an
attribute on the class would affect all future instances:

```python
ObjectC.x = 10     # sets class attribute — all instances see it
obj.x = 10         # sets instance attribute — only this instance has it
```

`setattr(obj, 'x', 10)` targets the instance. `setattr(ObjectC, 'x', 10)` would
target the class. Always use the instance variable here.

## doom_printer

The provided `doom_printer` filters out dunder attributes with `if attr[0] != '_'`
and prints the rest sorted alphabetically (because `dir()` returns names in sorted
order). This is why the output comes out alphabetically — `a` before `hello` before
`var_0`, etc.
