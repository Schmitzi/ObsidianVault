A generator function that splits text and yields substrings one at a time, with
optional transformations. The interesting constraint is that `random.shuffle` and
`random.sample` are forbidden — so you need an alternative for the shuffle option.

- A generator function uses `yield` instead of `return`; calling it returns a generator object
- The caller drives execution: each `next()` call (or loop iteration) resumes the function until the next `yield`
- `random.shuffle` sorts in-place, `random.sample` picks without replacement — both are banned; use `sorted(..., key=lambda x: random.random())` instead
- Validate input before doing anything; `yield "ERROR"` exactly once and return

## The Function Structure

```python
import random

def generator(text, sep=" ", option=None):
    '''Splits the text according to sep value and yields the substrings.'''
    if not isinstance(text, str):
        yield "ERROR"
        return
    valid_options = (None, "shuffle", "unique", "ordered")
    if option not in valid_options:
        yield "ERROR"
        return

    words = text.split(sep)

    if option == "unique":
        seen = set()
        result = []
        for w in words:
            if w not in seen:
                seen.add(w)
                result.append(w)
        words = result
    elif option == "ordered":
        words = sorted(words)
    elif option == "shuffle":
        words = sorted(words, key=lambda x: random.random())

    for word in words:
        yield word
```

## The shuffle Workaround

`random.shuffle` mutates a list in-place; `random.sample` returns a new list. Both
are banned. The cleanest alternative is sorting with a random key:

```python
words = sorted(words, key=lambda x: random.random())
```

`random.random()` returns a float in [0, 1). Using it as the sort key assigns each
word a random priority, effectively shuffling the list. The result is not
cryptographically uniform but it's fine for this exercise.

You could also use `random.randint` to build a random permutation manually, but
the lambda approach is cleaner.

## Why yield "ERROR" and return

The exercise says the function should return "ERROR" one time. Since the function
is a generator, it can't `return "ERROR"` in the usual sense — calling `return` from
a generator raises `StopIteration`. The pattern is:

```python
yield "ERROR"
return   # stops the generator after yielding one value
```

Without the `return`, the function would continue after `yield "ERROR"` and yield
more values. The `return` terminates the generator immediately.

## Generator vs Returning a List

You could implement this by building the whole list and returning it, but that would
mean collecting everything in memory before the caller sees the first word. Generators
are lazy — they produce values on demand. For large texts this matters; for this
exercise it's mostly about learning the pattern.

## unique with set

Maintaining insertion order while deduplicating can't use `set` alone (sets are
unordered). The pattern above — a `seen` set for O(1) lookup plus a `result` list
for order — is the standard Python idiom:

```python
seen = set()
result = []
for w in words:
    if w not in seen:
        seen.add(w)
        result.append(w)
```

In Python 3.7+ dicts also preserve insertion order, so `dict.fromkeys(words).keys()`
works too, but the explicit pattern is clearer.
