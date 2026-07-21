The aim of this project is to introduce you to the basic concepts behind machine learning.

For this project, I created a program that predicts the price of a car by using a `linear function` trained with a `gradient descent` algorithm.

Of course, I have picked Rust, what else.

## Goal

I need to implement a simple linear regression with a single feature; in this case, the mileage of a car.

To do so requires two programs:

### Predictor

This program will be used to predict the price of a car for a given mileage. When the program is launched, it needs to prompt for a mileage, then give back the estimated price for that mileage. The program should use the following hypothesis to predict the price:

$$ estimatedPrice(mileage) = \Theta_{0} + (\Theta_{1} * mileage) $$

Before running the training program, θ₀ and θ₁ will be set to 0.

### Trainer

This second program will be used to train the model. It will read the dataset file and perform a linear regression on the data.

Once linear regression has completed, you will save the variables θ₀ and θ₁ for use in the first program.

These are our formulas: $$ tmp\Theta_{0} = learningRate * \frac{1}{m} \sum_{i=0}^{m - 1}(estimatedPrice(mileage[i]) - price[i]) $$

$$ tmp\Theta_{1} = learningRate * \frac{1}{m} \sum_{i=0}^{m - 1}(estimatedPrice(mileage[i]) - price[i]) * mileage[i] $$

The `estimatePrice` here is the same as in the first program, but here is uses the temporary, most recently computed θ₀ and θ₁. These are both updated together.

## Bonus

All three bonus items from the subject are implemented:

- Plot the data into a graph to see the distribution.
- Plot the line resulting from the linear regression into the same graph.
- A third program that calculates the precision of the algorithm.

The `plot` binary renders the raw `(mileage, price)` data as a scatter, plus the trained regression line, to `regression.png`.

The `precision` binary computes the R² (coefficient of determination) of the trained model against the training data. On the provided `data.csv`, the model achieves an R² of roughly **0.73** — meaning mileage alone explains about 73% of the variance in price, which lines up with the visible spread of points around the regression line in `regression.png`.

## Build & Run

This is a single Cargo package with four binaries:

```bash
cargo build
cargo run --bin train      # trains the model, reads data.csv, writes thetas
cargo run --bin predict    # prompts for a mileage, prints an estimated price
cargo run --bin plot       # renders data.csv and the trained line to regression.png
cargo run --bin precision  # prints the R² of the trained model against data.csv
```

`train` must be run at least once before `predict`, `plot`, or `precision` will produce a meaningful result — before training, θ₀ and θ₁ default to 0.

## Data format

`train` expects a CSV file at `data.csv` with a header row, mileage in the first column and price in the second:

```
km,price
240000,3650
139800,3800
...
```

## Design decisions

- **Normalization.** Mileage and price are on very different scales (hundreds of thousands vs. thousands), which causes gradient descent to diverge on raw values. `train` applies min-max normalization to both columns before training, then converts the resulting θ₀/θ₁ back into real-world scale before saving — so `predict` can take a raw mileage as input with no knowledge of normalization ever having happened.
    
- **Convergence, not a fixed iteration count.** Rather than always running a hardcoded number of iterations, `train` loops until the per-iteration change in θ₀ and θ₁ both drop below a small threshold (`EPSILON`), with a hard iteration cap as a safety net in case a different dataset or learning rate never converges cleanly.
    
- **No cheating libraries.** The `csv` crate is used only to parse rows from the input file — it plays no part in computing the regression itself. The `plotters` crate only draws pixels to a PNG — it doesn't fit anything. All gradient descent logic, normalization, and the R² calculation are hand-written.