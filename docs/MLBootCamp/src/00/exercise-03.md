# Exercise 03 - Functional File

Two things in one exercise: writing a function with a docstring that uses Python's
built-in string classification methods, and then using `__name__ == "__main__"` to
make the same file work both as an importable module and as a standalone script.

- Python strings have built-in methods for character classification — no need to check ASCII ranges
- Docstrings are the standard way to document functions; accessible at runtime via `.__doc__`
- `__name__ == "__main__"` is the standard Python pattern for dual-mode files

## String Classification Methods

Python's `str` has methods that check character categories directly:

```python
'A'.isupper()   # True
'a'.islower()   # True
'!'.isspace()   # False
' '.isspace()   # True
```

For punctuation, the `string` module gives you a reference string of all punctuation
characters, which you can use with `in`:

```python
import string

'!' in string.punctuation   # True
'a' in string.punctuation   # False
```

`string.punctuation` is just the string `!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~`.

## Counting Characters

`str.isprintable()` returns True for any character that isn't a control character.
All the characters the exercise cares about (upper, lower, punctuation, space) are
printable.

```python
def text_analyzer(text=None):
    """
    This function counts the number of upper characters, lower characters,
    punctuation and spaces in a given text.
    """
    if text is None:
        text = input("What is the text to analyze?\n>> ")
    assert isinstance(text, str), "argument is not a string"

    printable = sum(1 for c in text if c.isprintable())
    upper = sum(1 for c in text if c.isupper())
    lower = sum(1 for c in text if c.islower())
    punct = sum(1 for c in text if c in string.punctuation)
    spaces = sum(1 for c in text if c == ' ')

    print(f"The text contains {printable} printable character(s):")
    print(f"- {upper} upper letter(s)")
    print(f"- {lower} lower letter(s)")
    print(f"- {punct} punctuation mark(s)")
    print(f"- {spaces} space(s)")
```

The generator expression inside `sum()` is the idiomatic Python way to count
elements matching a condition — equivalent to filtering and counting but in one pass.

## The __name__ == "__main__" Pattern

When Python imports a file, `__name__` is set to the module's name. When Python
runs a file directly from the command line, `__name__` is set to `"__main__"`. This
lets you put script-specific code in a block that only runs when the file is executed
directly, not when it's imported:

```python
if __name__ == "__main__":
    assert len(sys.argv) <= 2, "too many arguments"
    if len(sys.argv) == 2:
        text_analyzer(sys.argv[1])
    else:
        text_analyzer()
```

This is the standard Python pattern for any file that should work both ways. You'll
use it in almost every module going forward.

## Accessing the Docstring

The docstring of a function is stored in its `__doc__` attribute:

```python
print(text_analyzer.__doc__)
```

This is why docstrings are triple-quoted strings placed immediately after the `def`
line rather than just comments — they're actual string objects attached to the
function object at runtime.
