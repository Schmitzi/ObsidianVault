Four additional metrics for evaluating regression models: MSE, RMSE, MAE, and R². Each captures something slightly different about how wrong the predictions are.

## MSE — Mean Squared Error

```
MSE = 1/m * Σ(ŷ⁽ⁱ⁾ - y⁽ⁱ⁾)²
```

The loss function from ex06 divided by 2. Squaring the errors penalises large mistakes more heavily than small ones. The `/2` in ex06 was a calculus convenience — MSE uses `/m` only.

## RMSE — Root Mean Squared Error

```
RMSE = √MSE
```

Taking the square root brings the metric back to the same units as the original data, making it easier to interpret. An RMSE of 5 means predictions are off by about 5 units on average.

## MAE — Mean Absolute Error

```
MAE = 1/m * Σ|ŷ⁽ⁱ⁾ - y⁽ⁱ⁾|
```

Uses absolute value instead of squaring. Less sensitive to outliers than MSE — a single very wrong prediction does not dominate the metric.

## R² Score

```
R² = 1 - Σ(ŷ⁽ⁱ⁾ - y⁽ⁱ⁾)² / Σ(y⁽ⁱ⁾ - ȳ)²
```

Measures how much of the variance in y the model explains. R² = 1 means perfect predictions. R² = 0 means the model is no better than just predicting the mean. Negative R² means the model is actively worse than predicting the mean.

The denominator uses `ȳ` (the mean of y), not `ŷ`:

```python
y_mean = np.mean(y)
ss_res = np.sum((y_hat - y) ** 2)
ss_tot = np.sum((y - y_mean) ** 2)
return float(1 - ss_res / ss_tot)
```

## Choosing a Metric

- Use MSE or RMSE when large errors are especially bad (they get squared)
- Use MAE when outliers should not dominate the evaluation
- Use R² when you want to know how well the model explains the data relative to a baseline