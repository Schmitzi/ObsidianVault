# Exercise 04 - Elementary

Basic arithmetic operations with proper error handling for division by zero. The main
things to learn here are Python's division behaviour (which differs from C/Rust) and
using try/except to handle runtime errors gracefully instead of crashing.

- Python has two division operators: `/` for float division, `//` for integer (floor) division
- Division and modulo by zero raise `ZeroDivisionError` at runtime — catch it with try/except
- The output format for non-terminating decimals uses `...` not actual rounding

## Python Division vs C Division

In Python 3, `/` always returns a float, even for two integers:

```python
10 / 3    # 3.3333333333333335  (float)
10 // 3   # 3                   (floor division, rounds toward -inf)
10 % 3    # 1                   (remainder)
```

This is different from C where `int / int` gives int. The exercise output shows
`3.3333...` which means you need to detect non-terminating results and format them
accordingly — or just check if the result is a whole number:

```python
quotient = A / B
if quotient == int(quotient):
    print(f"Quotient:\t{int(quotient)}")
else:
    # show as many decimals as needed, truncate with ...
    formatted = f"{quotient:.4f}".rstrip('0').rstrip('.')
    print(f"Quotient:\t{formatted}...")
```

## Handling Division by Zero

`ZeroDivisionError` is raised by both `/` and `%` when the right operand is zero.
Wrap them in try/except rather than pre-checking `B == 0` manually:

```python
try:
    print(f"Quotient:\t{A / B}")
except ZeroDivisionError:
    print("Quotient:\tERROR (division by zero)")

try:
    print(f"Remainder:\t{A % B}")
except ZeroDivisionError:
    print("Remainder:\tERROR (modulo by zero)")
```

## Input Validation

The exercise expects `AssertionError` for bad argument counts and non-integer inputs:

```python
import sys

args = sys.argv[1:]

if len(args) == 0:
    print("Usage: python operations.py <number1> <number2>")
    sys.exit(0)

assert len(args) == 2, "too many arguments" if len(args) > 2 else "not enough arguments"

try:
    A, B = int(args[0]), int(args[1])
except ValueError:
    raise AssertionError("only integers")
```

Note the subject says "no bonus for handling decimals or scientific notation" — `int()`
is all you need. Anything that fails `int()` conversion gets the error message.

## Tab Alignment

Looking at the expected output, the labels are tab-separated from the values to keep
them aligned:

```
Sum:        13
Difference: 7
```

Use `\t` or an f-string with a tab character to match this formatting.
