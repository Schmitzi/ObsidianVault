Introducing the `Loss Function`

Its hard to tell how good our model is by just looking at the plots. We can see the some regression lines fit better than others, but it would be great to find a way to measure.

To evaluate our model, we are going to use a `metric` called `the loss function` (sometimes called the `cost function`).

The loss function tells us how bad out model is performing, how much it *costs* to use it, how much information we *lose* when we use it. If the model is good, we won't lose that much; if its terrible, we will have a high loss!

The metric we choose will deeply impact the evaluation (and therefore also the training) of the model.

A frequent way to evaluate the performance of a regression model is to measure the distance between each predicted value `(ŷ⁽ⁱ⁾)` and the real value it tries to predict `(y⁽ⁱ⁾)`. The distances are then squared, and averaged to get one single metric, denoted `𝐽`:
<p align="center">
  <img src="imgs/07-formula.png" alt="formula"/>
  <img src="imgs/07-predicts.png" alt="predict"/>
</p>

## Step 6. Carry On, Regression

Because this is essentially the next steps after the last exercise, I'm just going to carry on from where we left off.

### 6. Minimizing the Error using Least Squares Method

To determine the best fit line, linear regression uses `Least Squares Method`, which minimizes the difference between actual and predicted values. These values are called `residuals`. The formula is the last part of our original formula

```
Residual = yᵢ - ŷᵢ
```

- *yᵢ*: the actual observed value
- *ŷᵢ*: the predicted value from the line for that xᵢ

The least squares method minimizes the sum of the squared residuals:

```
Σ(yᵢ - ŷᵢ)²
```

This method ensures that the best line represents the data where the sum of the squared differences between the predicted values and actual values is as small as possible.

### 7. Interpolation of the Best-Fit Line

- _Slope (m)_: The slope indicates how much the dependent variable changes for every one-unit increase in the independent variable. For example, if the slope is 5, then y increases by 5 units for every 1-unit increase in x.
    
- _Intercept (b)_: The intercept represents the predicted value of y when x = 0. It's the point where the line crosses the y-axis.
    

## Implementation

The loss is split into two functions — `loss_elem_` computes the squared error for each example individually, and `loss_` averages them into a single number.

### loss_elem_

```python
J_elem = (y_hat - y) ** 2
return np.array(J_elem, dtype=float)
```

Returns an array of shape `(m, 1)` — one squared error per training example. Useful for visualising which examples the model is getting most wrong.

### loss_

```python
J_elem = loss_elem_(y, y_hat)
return float(np.sum(J_elem) / (2 * len(y)))
```

Sums the squared errors and divides by `2m`. The `/2` is a calculus convenience — when you later differentiate the loss to compute the gradient, the 2 cancels with the exponent, simplifying the result. It does not change which model is better or worse, only the scale of the number.

## Relation to MSE

This loss function is `MSE / 2`:

```
J(θ) = MSE / 2 = 1/(2m) * Σ(ŷ⁽ⁱ⁾ - y⁽ⁱ⁾)²
MSE          = 1/m   * Σ(ŷ⁽ⁱ⁾ - y⁽ⁱ⁾)²
```

Both rank models identically — halving every score does not change the ordering.

## Shape Requirement

Both functions expect 2D arrays of shape `(m, 1)`, not flat 1D arrays. If `y` is 1D, reshape it before calling:

```python
loss_(y.reshape(-1, 1), y_hat)
```
