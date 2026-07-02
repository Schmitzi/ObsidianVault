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

- *Slope (m)*: The slope indicates how much the dependent variable changes for every one-unit increase in the independent variable. For example, if the slope is 5, then y increases by 5 units for every 1-unit increase in x.

- *Intercept (b)*: The intercept represents the predicted value of y when x = 0. It’s the point where the line crosses the y-axis.

