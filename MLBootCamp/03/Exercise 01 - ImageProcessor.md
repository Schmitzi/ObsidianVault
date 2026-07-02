The purpose of this exercise is to hide the loading/display plumbing behind a clean
interface so the next exercises can focus entirely on array manipulation. You build it
once here and import it everywhere else.

- Matplotlib's `plt.imread(path)` loads a PNG and returns a NumPy array of shape
  `(height, width, channels)` with `dtype=float32`, values in [0.0, 1.0]
- `plt.imshow(array)` renders an array as an image; `plt.show()` opens the window
- The array shape convention: `array.shape` → `(rows, cols, channels)` — rows is
  the height, cols is the width

## load

The method opens the file, prints the dimensions, and returns the array. The dimension
message format from the subject is `Loading image of dimensions W x H` — note it's
width first, then height (columns × rows):

```python
import matplotlib.pyplot as plt

def load(self, path):
    try:
        array = plt.imread(path)
        h, w = array.shape[0], array.shape[1]
        print(f"Loading image of dimensions {w} x {h}")
        return array
    except FileNotFoundError as e:
        print(f"Exception: FileNotFoundError -- strerror: {e.strerror}")
        return None
    except OSError as e:
        print(f"Exception: OSError -- strerror: {e.strerror}")
        return None
```

Two error cases are shown in the subject's examples:
- `FileNotFoundError` — path doesn't exist
- `OSError` — file exists but can't be read as an image (e.g. empty file)

`FileNotFoundError` is a subclass of `OSError`, so if you only catch `OSError` you'll
catch both — but you'd lose the ability to print different messages. Catch
`FileNotFoundError` first (more specific), then `OSError` (more general).

## display

`plt.imshow` does the heavy lifting. `plt.show()` blocks until the window is closed:

```python
def display(self, array):
    plt.imshow(array)
    plt.show()
```

No axis labels are required. The subject shows a plain image window — `plt.axis('off')`
is optional but cleaner.

## The Array Shape

After loading, `array.shape` is `(height, width, 3)` for an RGB image. The three
channels at index 2 are R, G, B in that order. You'll use this layout constantly in
the next exercises — knowing which axis is which matters:

```
array[row, col, channel]
       ↑     ↑      ↑
    height  width  R/G/B
```

Slicing `array[:, :, 0]` gives you all the red values; `array[:, :, 1]` is green;
`array[:, :, 2]` is blue.
