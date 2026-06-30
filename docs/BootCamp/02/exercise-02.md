# Exercise 02 - The Logger

Decorators: functions that wrap other functions to add behaviour before and/or after
the call. The `@log` decorator here times each method call and writes a formatted
line to `machine.log`.

- A decorator is a function that takes a function and returns a (usually modified) function
- `functools.wraps` preserves the original function's name and docstring on the wrapper
- `time.time()` gives seconds as a float — subtract start from end to get elapsed time
- `os.environ.get('USER')` reads the current username from environment variables

## Decorator Anatomy

The standard pattern: outer function receives the function to decorate, inner
`wrapper` adds behaviour around it, outer function returns the wrapper:

```python
import functools

def my_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # do something before
        result = func(*args, **kwargs)
        # do something after
        return result
    return wrapper
```

`@my_decorator` above a function definition is syntactic sugar for
`function = my_decorator(function)`. It runs at definition time, not call time.

## The log Decorator

```python
import time
import os
import functools

def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start

        # Convert function name: make_coffee → Make Coffee
        name = func.__name__.replace('_', ' ').title()
        user = os.environ.get('USER', 'unknown')

        # Format: elapsed in ms if < 1s, in s if >= 1s
        if elapsed < 1:
            time_str = f"{elapsed * 1000:.3f} ms"
        else:
            time_str = f"{elapsed:.3f} s"

        line = f"({user})Running: {name:<20}[ exec-time = {time_str} ]\n"

        with open("machine.log", "a") as f:
            f.write(line)

        return result
    return wrapper
```

## The Alignment Hint

The subject notes that the distance between `:` and `[` is 20 characters. The function
name field is left-aligned in a 20-character field — `{name:<20}` in an f-string
does this. `Start Machine` is 13 chars, padded to 20 with spaces; `Add Water` is 9
chars, padded to 20.

## Time Units

Looking at the log output: fast methods show milliseconds (`0.001 ms`), slow ones
show seconds (`2.499 s`). The threshold is 1 second:

```python
if elapsed < 1.0:
    time_str = f"{elapsed * 1000:.3f} ms"
else:
    time_str = f"{elapsed:.3f} s"
```

## functools.wraps

Without `@functools.wraps(func)`, the wrapper function's `__name__` would be
`"wrapper"` rather than the original function's name. This breaks logging, debugging,
and introspection. Always use it when writing decorators.

```python
@log
def make_coffee(self): ...

# Without @functools.wraps:
make_coffee.__name__   # "wrapper"

# With @functools.wraps:
make_coffee.__name__   # "make_coffee"
```

## Decorating Methods

`@log` works on methods (functions inside a class) the same way it works on plain
functions. The `self` argument is just the first positional arg — it gets captured
by `*args` in the wrapper and passed through to the real function call unchanged.
