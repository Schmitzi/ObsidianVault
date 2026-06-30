Reversing a string and swapping its case. Simple on the surface, but a good
introduction to three Python idioms you'll use constantly: `sys.argv`, slice notation,
and string methods.

- `sys.argv` is how Python receives command-line arguments
- Slice notation `[::-1]` reverses any sequence without a loop
- `.swapcase()` does exactly what it says — no manual ord() arithmetic needed

## sys.argv

`sys.argv` is a list. `sys.argv[0]` is always the script name. Arguments start at
index 1.

```python
import sys

# running: python3 exec.py Hello World
# sys.argv = ['exec.py', 'Hello', 'World']

args = sys.argv[1:]  # everything after the script name
```

When multiple arguments are provided, the exercise wants them joined with a space
before processing — so join first, then reverse.

## Reversing a String

Python slicing syntax is `[start:stop:step]`. Setting step to `-1` walks backwards
through the entire sequence:

```python
s = "Hello World!"
print(s[::-1])   # !dlroW olleH
```

This works on any sequence — strings, lists, tuples. No need to implement a loop.

## Swapping Case

`.swapcase()` converts every uppercase letter to lowercase and vice versa. Non-letter
characters are left alone.

```python
s = "Hello World!"
print(s.swapcase())   # hELLO wORLD!
```

## Putting It Together

The key insight is that reversing and then swapping case gives the same result as
swapping and then reversing — order doesn't matter here, since `.swapcase()` is
character-level and doesn't depend on position.

```python
import sys

if len(sys.argv) > 1:
    text = ' '.join(sys.argv[1:])
    print(text[::-1].swapcase())
```

Note there's no `else` needed — if no arguments are provided, you just do nothing,
which Python does naturally when the `if` branch isn't taken.
