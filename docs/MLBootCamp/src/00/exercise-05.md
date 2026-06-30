# Exercise 05 - The Right Format

Five small formatting exercises covering the main ways Python formats strings. The
interesting part isn't the string formatting syntax itself but the width/precision/
alignment specifiers that let you control output without manual padding logic.

- f-strings (`f"..."`) are the modern way — readable and fast
- Format specifiers go inside `{}` after a colon: `{value:specifier}`
- Zero-padding, alignment, decimal precision, and scientific notation all have dedicated specifiers

## Format Specifier Syntax

The general form inside `{}` is:

```
{value:[[fill]align][sign][width][.precision][type]}
```

The most useful pieces:

| Specifier | Meaning | Example |
|---|---|---|
| `02d` | zero-pad integer to width 2 | `f"{9:02d}"` → `"09"` |
| `>42` | right-align in field of width 42 | `f"{'hi':>10}"` → `"        hi"` |
| `<42` | left-align in field of width 42 | `f"{'hi':<10}"` → `"hi        "` |
| `.2f` | float with 2 decimal places | `f"{3.14159:.2f}"` → `"3.14"` |
| `.2e` | scientific notation, 2 decimal places | `f"{10000:.2e}"` → `"1.00e+04"` |

## kata00 — Tuple of Integers

Unpack the tuple into the format string using `*` to expand:

```python
kata = (19, 42, 21)
print(f"The {len(kata)} numbers are: {', '.join(str(n) for n in kata)}")
```

Or use `.format()` with unpacking:

```python
print("The {} numbers are: {}, {}, {}".format(len(kata), *kata))
```

## kata01 — Dictionary Iteration

Iterating a dict gives keys. Access values with `dict[key]`:

```python
for lang, creator in kata.items():
    print(f"{lang} was created by {creator}")
```

`.items()` returns key-value pairs as tuples — the cleanest way to loop over both.

## kata02 — Zero-Padded Date/Time

The output `09/25/2019 03:30` needs zero-padding on month, day, hour, minute:

```python
kata = (2019, 9, 25, 3, 30)
print(f"{kata[1]:02d}/{kata[2]:02d}/{kata[0]} {kata[3]:02d}:{kata[4]:02d}", end='')
```

The `wc -c` check tells you the exact character count expected (16 chars + newline = 17).
Don't add a trailing newline if the count needs to match — or use `end=''` and let
the shell add one. Check with `| cat -e` to see where the line ends.

## kata03 — Right-Aligned with Dashes

Fill character comes before the alignment direction. To fill with `-` and right-align
in 42 characters:

```python
kata = "The right format"
print(f"{kata:->42}", end='')
```

`->42` means: fill with `-`, align right, total width 42. The `%` in `cat -e` output
marks the end of a line without a trailing newline — use `end=''`.

## kata04 — Mixed Number Formatting

```python
kata = (0, 4, 132.42222, 10000, 12345.67)
print(f"module_{kata[0]:02d}, ex_{kata[1]:02d} : {kata[2]:.2f}, {kata[3]:.2e}, {kata[4]:.2e}")
```

`1.00e+04` is `10000` in scientific notation with 2 decimal places. Python's `:.2e`
produces lowercase `e` with a sign and zero-padded exponent: `1.00e+04`. The `cut -c 10,18`
check verifies that character 10 is `,` and character 18 is `:`, so the spacing has
to be exact.
