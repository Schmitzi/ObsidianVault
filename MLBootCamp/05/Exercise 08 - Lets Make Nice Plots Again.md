Extends the basic scatter + prediction line plot from ex05 by adding two things: the actual loss value in the title, and vertical dashed lines connecting each data point to its predicted value on the regression line.

## The Residual Lines

The dashed lines visualise what the loss function is measuring — the vertical distance between each real value and its prediction. The further the points are from the line, the longer the dashes, and the higher the loss.

```python
for xi, yi, yi_hat in zip(x, y, y_hat.flatten()):
    plt.plot([xi, xi], [yi, yi_hat], color='green', linestyle='--')
```

Each line is drawn between `(xi, yi)` and `(xi, yi_hat)` — same x coordinate, different y values.

## Shape Mismatch

`y_hat` from `predict_` is 2D — shape `(m, 1)`. Matplotlib's `plot` expects 1D arrays. Flattening at the call site with `.flatten()` avoids modifying y_hat itself.

Similarly, `loss_` expects 2D inputs. If `y` is 1D, reshape it before calling:

```python
loss = loss_(y.reshape(-1, 1), y_hat)
```

## Displaying the Loss

```python
plt.title(f'Linear Regression (Loss J = {loss})')
```

Note the closing `)` inside the string — the parenthesis in "Loss J = ..." needs to be closed or the f-string will produce a malformed title.