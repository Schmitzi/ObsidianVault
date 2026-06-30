# Exercise 04 - Working with Lists

Two ways to iterate over paired data: `zip` to walk two sequences in lockstep, and
`enumerate` to walk one sequence with an index. Both produce the same result here
— the exercise is about knowing both tools exist.

- `zip(a, b)` produces pairs `(a[0], b[0])`, `(a[1], b[1])`, ... stopping at the shorter sequence
- `enumerate(lst)` produces pairs `(0, lst[0])`, `(1, lst[1])`, ... — index plus item
- Static methods belong to the class but don't receive `self` or `cls`; use `@staticmethod`
- `zip` stops silently at the shorter list — so it won't naturally detect length mismatches; check manually

## zip

```python
words = ["Le", "Lorem", "Ipsum", "est", "simple"]
coefs = [1.0, 2.0, 1.0, 4.0, 0.5]

for coef, word in zip(coefs, words):
    print(coef, word)
# 1.0 Le
# 2.0 Lorem
# ...
```

`zip` is lazy in Python 3 — it returns a zip object, not a list. It produces pairs
on demand as you iterate. The critical behaviour: if the sequences have different
lengths, `zip` stops at the shortest without raising an error. This is why you must
check lengths manually before computing.

## enumerate

```python
for i, word in enumerate(words):
    print(i, coefs[i], word)
```

`enumerate` gives you the index so you can access the parallel list by position.
It's less elegant than `zip` for this specific use case but demonstrates the pattern.

## The Evaluator Class

```python
class Evaluator:

    @staticmethod
    def zip_evaluate(coefs, words):
        if len(coefs) != len(words):
            return -1
        return sum(coef * len(word) for coef, word in zip(coefs, words))

    @staticmethod
    def enumerate_evaluate(coefs, words):
        if len(coefs) != len(words):
            return -1
        return sum(coefs[i] * len(words[i]) for i in range(len(words)))
```

Both functions compute `Σ coef[i] * len(word[i])` — a weighted sum of word lengths.

## @staticmethod vs @classmethod

`@staticmethod` — no implicit first argument. The method doesn't need access to
the instance or the class. Used when the function logically belongs to the class
but doesn't need class-level state.

`@classmethod` — receives `cls` as the first argument (the class itself, not an
instance). Used for alternative constructors or methods that need to create
instances of the class.

```python
class Foo:
    @staticmethod
    def static_method(x):
        return x * 2          # no self, no cls

    @classmethod
    def class_method(cls, x):
        return cls()          # can create instances of cls
```

Call either without instantiating: `Evaluator.zip_evaluate(coefs, words)`.

## Gotcha: zip Length Mismatch

```python
coefs = [1.0, 2.0]
words = ["hello", "world", "extra"]

list(zip(coefs, words))   # [('1.0', 'hello'), ('2.0', 'world')]  — 'extra' silently dropped
```

The exercise returns `-1` on mismatch, so pre-check `len(coefs) != len(words)` before
zipping.
