# Math and Stuff

Imagine you're standing at the top of a hill. You are blindfolded, and can only feel the ground underneath you. 

Now the goal is to get to the bottom of the hill. How do we do this? Well we feel where our feet are. Can we feel it sloping to one side? If we feel its flat, we need to move in one direction and feel (measure) the changes. Now we feel a slope to the left, and we can the proceed to head down the hill.

The distinction matters here for two reasons:

- *"The slope at the point where you're standing"* - This is the `derivative`. The derivative of a function at a point just tells you *how steep it is, and which way it's tilting, right there*.
- *"Flat ground"* - mathematically, that's where the `derivative equals zero`

So "setting the derivative to zero" in calculus is literally asking *"where is it completely flat"*. But flat can be at the top (the "most wrong" point) or at the bottom (the "least wrong point")

When we draw this we get a sloped valley called a `cost function` (sometimes "loss function") that measures "how wrong is the model"

That means *training a model = walking downhill on the cost function until you reach the valley bottom = the knob value where the wrongness is the least*

## Gradient Descent

The process of *walking downhill on a slope* - repeatedly checking the slope and stepping in the downhill direction is known as `Gradient Descent`. This is used for optimisation.

Now you might think that we could just check the whole map for the bottom but we can only fell the space under our feet

The degree of wrongness isn't just one single knob that decides how much to apply, its many knobs at once (in a *multivariate regression*, maybe dozens or hundreds of knobs - one per feature in the data).

Each knob in our model is another dimension:
- One knob -> 2D graph (knob, wrongness) - a curve, easy to eyeball
- Two knobs -> 3D graph (knob1, knob2, wrongness) - a surface, a landscape with hills and valley. Still **technically* visualisable.
- 100 knobs -> *101-dimensional* graph. You cannot draw this, picture it, or "just look at it". Its not a metaphorical hill anymore, its a hill in a space with more directions that human intuition can handle.

But our procedure works with any number of dimensions.
```
feel the slope -> feel the local slope -> take a small step downhill -> repeat
```

This is why Gradient Descent is important. It scales to dimensions we can't visualize.

The knobs I've mentioned will probably be known as `parameters` (often written as θ, theta), and the "slope in each direction" (one-per-knob) is called the `gradient` which is just the derivative but for a function with multiple inputs, one slope-reading per dimension. That's where the name comes from: descending following the gradient.

## Linear Regression

This is the first real algorithm.

The general shape of an equation for a line is like this:
```
y = mx +b
```

Where:

- x = the input data
- m = where the line crosses the y-axis
- b = the slope of the line 

In ML, this get renamed but has the same idea:

```
ŷ = θ₀ + θ₁x
```

Where:

- ŷ ("y-hat") = the models prediction
- x = the input data (a feature - like the "size of a house")
- θ₀ = ("theta zero") = same role as b - where the line crosses the y-axis
- θ₁ = ("theta one") = same role as m - the slope of the line

θ₀ and θ₁ are the *knobs* from the metaphor. Training a Linear REgression model means using Gradient Descent to fins the values of θ₀ and θ₁ that make the line fit the data as closely as possible. For example, the values that sit at the bottom of the `cost function` valley

Lets try this. Pretend you're predicting *house price (y)* from *house size in m² (x)* and you are told:

- θ₀ = 50,000
- θ₁ = 2,000

Using ŷ = θ₀ + θ₁x, what would the model predict for a house thats *80m²*

```
ŷ = 50,000 + 2,000 * 80

2,000 * 80 = 160,000

160,000 + 50,000 = 210,000

ŷ = 210,000
```

So the model pridicts an 80m
