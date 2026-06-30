Reimplementing Python's three classic higher-order functions to understand what they
actually do under the hood. All three operate on functions and iterables — the key
insight is that functions are first-class objects in Python and can be passed around
like any other value.

- `map` applies a function to every element of an iterable and returns results lazily
- `filter` keeps only elements for which the function returns truthy
- `reduce` collapses an iterable into a single value by applying a function cumulatively
- All three should return generator objects (lazy), not lists — the examples show `<generator object ...>`

## ft_map

Apply `function_to_apply` to each element and yield the result. Using `yield` makes
it a generator that produces values on demand, matching the behaviour of Python's
built-in `map`:

```python
def ft_map(function_to_apply, iterable):
    """Map the function to all elements of the iterable."""
    try:
        for item in iterable:
            yield function_to_apply(item)
    except TypeError as e:
        raise TypeError(str(e))
```

Calling `ft_map(lambda x: x + 1, [1, 2, 3])` returns a generator object immediately.
Only when you iterate it (via `list()`, a `for` loop, etc.) does it actually run.

## ft_filter

Yield only the elements for which `function_to_apply` returns a truthy value:

```python
def ft_filter(function_to_apply, iterable):
    """Filter the result of function apply to all elements of the iterable."""
    try:
        for item in iterable:
            if function_to_apply(item):
                yield item
    except TypeError as e:
        raise TypeError(str(e))
```

`not (dum % 2)` is True when `dum % 2 == 0` (even numbers) — the lambda in the
example keeps even numbers by testing that the remainder is not truthy (i.e. is zero).

## ft_reduce

Accumulate: start with the first element as the running value, then apply the function
to `(running_value, next_element)` for each subsequent element:

```python
def ft_reduce(function_to_apply, iterable):
    """Apply function of two arguments cumulatively."""
    try:
        it = iter(iterable)
        result = next(it)   # first element becomes the initial accumulator
        for item in it:
            result = function_to_apply(result, item)
        return result
    except StopIteration:
        raise TypeError("ft_reduce() of empty sequence with no initial value")
    except TypeError as e:
        raise TypeError(str(e))
```

`iter()` converts the iterable to an iterator. `next(it)` pulls the first element
without consuming the rest. This is why `reduce` on an empty sequence raises — there's
nothing to seed the accumulator with.

## Lambda Functions

Lambdas are anonymous single-expression functions. The syntax is `lambda args: expression`:

```python
lambda x: x + 1          # same as def f(x): return x + 1
lambda u, v: u + v        # two args, used in ft_reduce example
lambda dum: not (dum % 2) # True for even numbers
```

Lambdas can only contain a single expression — no statements, no assignments, no
multi-line bodies. Use a regular `def` when the logic is more than one expression.

## Why Generators and Not Lists

The built-in `map` and `filter` in Python 3 are lazy — they don't compute anything
until iterated. The examples show a `<generator object ft_map at 0x...>` when called
without `list()`. Implementing with `yield` matches this behaviour exactly.
