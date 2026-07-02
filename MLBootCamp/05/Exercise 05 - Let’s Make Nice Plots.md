The goal here is to implement a function to plot the data and the prediction line ( or regression line). We need to plot the data points (with their x and y values), and the prediction line that represents the hypothesis (hθ).

This means we are starting linear regression.

## What is linear regression?

Linear regression is a supervised learning algorithm that models the relationship between two variables using a `straight-line equation`.

```
y = mx + c
```

- *y*: Dependent variable ( the outcome we want to predict)
- *x*: Independent variable (the feature we use to make predictions)
- *m*: Slope of the line (how much y changes when x changes)
- *c*: Intercept (where the line crosses the Y-axis)

The goal of linear regression is to *find the best-fit line* that minimizes the difference between the actual and predicted values- This is done using the `Least Squares Method`, which minimizes the sum of squared differences between observed and predicted values.

## Why is Linear Regression Important

Linear Regression is widely used in various industries. Some practical applications include:

- *Stock Market Prediction*: Estimating future stock prices based on past trends
- *Sales Forecasting*: Predicting company revenue based on historical data.
- *Real Estate Pricing*: Determining house prices based on location, size, etc
- *Customer Analysis*: Understanding consumer behavior: purchasing patterns
- *Healthcare*: Predicting disease risk based on patient data

## Implementing Linear Regression in Numpy

### 1. First is adding our imports:

```py
import numpy as np
import matplotlib.pyplot as plt
```

### 2. Define the X and Y

```py
x = np.arange(1, 6)
y = np.array([3.74013816, 3.61473236, 4.57655287, 4.66793434, 5.95585554])
```

### 3. Compute Best-Fit Line

```py
c = add_intercept(x)
```

### 4. Predicting ŷ

```py
y_hat = np.dot(c, theta).astype(float)
```

### 5. Visualization

```py
plt.scatter(x, y, color='blue', label='Actual Data')
plt.plot(x, y_hat, color='red', label='Prediction line')
plt.xlabel('X (Independent Variable)')
plt.ylabel('Y (Dependent Variable)')
plt.title('Linear Regression with NumPy')
plt.legend()
plt.show()
```

## Output

![plot](imgs/06-plot.png)