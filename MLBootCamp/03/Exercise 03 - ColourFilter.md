
---
*ATTENTION*

*NO, "Colour" is not incorrectly spelt.*

---

Six image filters, each implemented with a constrained set of NumPy tools. The
constraints are deliberate — they force you to understand *why* each approach works
rather than just calling a general-purpose function for everything.

- An RGB image array has shape `(H, W, 3)` — the last axis holds [R, G, B]
- Broadcasting: operations on arrays of different shapes work when shapes are
  compatible (e.g. multiplying an `(H, W, 3)` array by a scalar affects all values)
- `np.copy(array)` makes a deep copy so the original is not modified
- `np.zeros(shape)` creates an all-zero array; `np.dstack([a, b, c])` stacks arrays
  along a new third axis

## invert

Subtracts every channel value from 1.0 (since pixel values are floats in [0, 1]).
Black becomes white, white becomes black, and every colour becomes its complement:

```python
def invert(self, array):
    result = array.copy()
    return 1 - result
```

The constraint allows only `.copy`, `+`, `-`, `=`. The `1 - array` expression works
because NumPy broadcasts the scalar `1` across the entire array.

## to_green

Zeroes out the red and blue channels by multiplying the array by `[0, 1, 0]`. NumPy
broadcasts `[0, 1, 0]` along the last axis — R×0, G×1, B×0:

```python
def to_green(self, array):
    return array.copy() * [0, 1, 0]
```

Only `.copy` and `*` are allowed — the multiplication by a 3-element list is the key
insight. No indexing or assignment needed.

## to_blue

Builds a new array where only the blue channel is kept. The approach: create zero
arrays for R and G channels matching the image size, then stack them with the original
blue channel:

```python
def to_blue(self, array):
    zeros = np.zeros(array.shape[:2])   # shape (H, W)
    blue  = array.copy()[:, :, 2]      # shape (H, W)
    return np.dstack([zeros, zeros, blue])
```

`array.shape[:2]` gives `(H, W)` — slicing the shape tuple to drop the channel count.
`np.dstack` stacks three `(H, W)` arrays into a single `(H, W, 3)` array.

## to_red

The subject allows only `.copy`, `.to_green`, `.to_blue`, `-`, `+`. This is a hint:
build red by subtracting the green and blue contributions from the original:

```python
def to_red(self, array):
    return array.copy() - self.to_green(array) - self.to_blue(array)
```

This works because `to_green` zeroes R and B (leaving only G), and `to_blue` zeroes R
and G (leaving only B). Subtracting both from the original leaves only R.

## to_celluloid

Cel-shading posterises the image: continuous pixel values are snapped to a small number
of discrete levels, producing flat bands of colour like a cartoon.

The approach: define threshold boundaries, then for each pixel assign it the value of
whichever band it falls into:

```python
def to_celluloid(self, array):
    result = array.copy()
    thresholds = np.linspace(0, 1, 5)   # [0.0, 0.25, 0.5, 0.75, 1.0]
    for i in range(len(thresholds) - 1):
        mask = (result > thresholds[i]) & (result <= thresholds[i + 1])
        result[mask] = thresholds[i + 1]
    return result
```

`np.linspace(0, 1, 5)` gives 5 evenly spaced values — the boundaries of 4 shade bands.
The boolean mask selects pixels in each band and assigns them the band's ceiling value.
The subject requires at least four thresholds (i.e. at least 4 shade bands).

## to_grayscale

Two modes, selected by the `filter` argument:

**Mean (`'m'` or `'mean'`)**: average the three channels at each pixel. The result is a
single scalar per pixel, repeated 3 times to keep the `(H, W, 3)` shape:

```python
gray = array.sum(axis=2) / 3         # shape (H, W)
gray = gray.reshape(*array.shape[:2], 1)
gray = np.broadcast_to(gray, array.shape)
return gray.astype(array.dtype)
```

`sum(axis=2)` collapses the channel axis. `reshape(..., 1)` re-adds a size-1 channel
axis so `broadcast_to` can expand it back to 3 channels.

**Weighted (`'w'` or `'weight'`)**: each channel gets its own weight. The standard
perceptual weights (if not supplied) are roughly R=0.299, G=0.587, B=0.114, but the
method should accept `r_weight`, `g_weight`, `b_weight` via `**kwargs`:

```python
weights = np.array([kwargs.get('r_weight', 0.299),
                    kwargs.get('g_weight', 0.587),
                    kwargs.get('b_weight', 0.114)])
gray = (array * weights).sum(axis=2)
# then reshape and broadcast as above
```

Multiplying `(H, W, 3)` by a `(3,)` array broadcasts along the last axis —
each channel is scaled by its weight before summing.
