# Exercise 02 - The Odd, the Even and the Zero

Checking odd/even/zero with proper input validation. The main concept here is using
`assert` for error handling — a Python idiom that raises `AssertionError` with a
message when the condition is false.

- `assert` is the idiomatic way to validate preconditions in Python
- `int()` raises `ValueError` when it can't convert — catch that to detect non-integers
- Zero is a separate case from even, so check it first

## Using assert for Validation

`assert condition, "message"` raises `AssertionError: message` if the condition is
False. The subject's expected output shows `AssertionError:` prefixed output, which
is exactly what Python prints when an assertion fails and nothing catches it.

```python
assert len(sys.argv) == 2, "more than one argument is provided"
```

This is cleaner than manually raising exceptions for simple validation.

## Detecting Non-Integer Input

`int()` is all you need here. If it raises a `ValueError`, the input wasn't an integer:

```python
try:
    n = int(sys.argv[1])
except ValueError:
    assert False, "argument is not an integer"
```

Or more directly:

```python
try:
    n = int(sys.argv[1])
except ValueError:
    raise AssertionError("argument is not an integer")
```

## The Logic

Zero has to come before even, because 0 % 2 == 0 — it would match the even branch
if you checked even first and you want a distinct output for zero.

```python
if n == 0:
    print("I'm Zero.")
elif n % 2 == 0:
    print("I'm Even.")
else:
    print("I'm Odd.")
```

The modulo operator `%` in Python always returns a non-negative result when the
divisor is positive, unlike C where it can be negative for negative dividends. So
`-3 % 2 == 1` in Python, meaning `-3` correctly reports as odd.

## Full Structure

```python
import sys

if len(sys.argv) == 1:
    pass  # or print usage
else:
    assert len(sys.argv) == 2, "more than one argument is provided"
    try:
        n = int(sys.argv[1])
    except ValueError:
        raise AssertionError("argument is not an integer")
    if n == 0:
        print("I'm Zero.")
    elif n % 2 == 0:
        print("I'm Even.")
    else:
        print("I'm Odd.")
```
