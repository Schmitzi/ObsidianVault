The introduction to `yield` and generators. The exercise wants a function that wraps
an iterable, yields each element one at a time, and prints a progress bar to the
terminal on each iteration.

- `yield` turns a function into a generator — it pauses execution and returns a value, resuming from the same point on the next call
- A generator is itself an iterable, so it works in `for` loops just like a list
- `\r` moves the cursor to the beginning of the current line without a newline, allowing you to overwrite the same line repeatedly

## How yield Works

A normal function runs to completion and returns once. A generator function contains
`yield` and returns a generator object. Each time the caller iterates it, execution
resumes from where it left off:

```python
def count_to_three():
    yield 1
    yield 2
    yield 3

for n in count_to_three():
    print(n)   # prints 1, 2, 3
```

For `ft_progress`, you need to wrap an existing iterable — yield each element in
order while also printing progress:

```python
def ft_progress(lst):
    total = len(lst)
    for i, item in enumerate(lst):
        # print progress bar here
        yield item
```

The caller's `for elem in ft_progress(listy)` gets each `item` in sequence. The
progress printing happens as a side effect of the iteration.

## Computing ETA and Elapsed Time

Use `time.time()` to get the current timestamp in seconds as a float:

```python
import time

start = time.time()

# inside the loop:
elapsed = time.time() - start
if i > 0:
    rate = elapsed / i          # seconds per item
    eta = rate * (total - i)    # seconds remaining
else:
    eta = 0.0
```

## The Progress Bar Format

The expected output looks like:

```
ETA: 8.67s [ 23%][=====>          ] 233/1000 | elapsed time 2.33s
```

The filled portion of the bar scales with percentage. A bar width of 20 characters
is a reasonable default:

```python
bar_width = 20
filled = int(bar_width * i / total)
bar = '=' * filled + '>' + ' ' * (bar_width - filled - 1)
line = f"ETA: {eta:.2f}s [{percent:3d}%][{bar}] {i}/{total} | elapsed time {elapsed:.2f}s"
```

## Overwriting the Same Line

Print to stdout with `\r` at the start and no newline at the end:

```python
import sys

print(f"\r{line}", end='', flush=True)
```

`end=''` suppresses the newline. `flush=True` forces Python to actually write the
output immediately rather than buffering it — essential for real-time progress
display. The calling code does `print()` after the loop to move to a new line.

## Full Generator

```python
import time
import sys

def ft_progress(lst):
    total = len(lst)
    start = time.time()
    for i, item in enumerate(lst):
        elapsed = time.time() - start
        percent = int(100 * i / total) if total > 0 else 0
        eta = (elapsed / i) * (total - i) if i > 0 else 0.0
        bar_width = 20
        filled = int(bar_width * i / total) if total > 0 else 0
        bar = '=' * filled + '>' + ' ' * (bar_width - filled - 1)
        line = (f"ETA: {eta:.2f}s [{percent:3d}%]"
                f"[{bar}] {i}/{total} | elapsed time {elapsed:.2f}s")
        print(f"\r{line}", end='', flush=True)
        yield item
```

## Why Not tqdm

`tqdm` does exactly this, just with more polish. Looking at how to build it from
scratch is what teaches you generators and terminal control sequences. After this,
`tqdm` makes perfect sense.
