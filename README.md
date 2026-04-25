# Basket Pricing Under Log-Normal Models

This repository contains code, notes, and numerical experiments for pricing basket and spread options under multi-asset log-normal models. The project studies analytical approximation methods for European basket options whose payoffs depend on weighted combinations of correlated assets. The main objective is to develop pricing formulas that are mathematically transparent, computationally efficient, and accurate enough for practical use in commodity markets and other multi-asset settings.

The repository is based on the working paper **“On Basket Pricing Problems Under Log-Normal Models: Reviews and New Insights”** by Steve Tchoneteck, Sophia Zorek, Hasanjan Sayit, and Frederi Viens. The paper develops a unified probability-based framework for basket pricing, derives two complementary representations for standard baskets, and compares two analytical approximations against Monte Carlo benchmarks using soybean crush spread data.

---

## Motivation

Basket and spread options are important in practice because many real financial exposures depend on several correlated assets rather than one single underlying price. Typical examples include crack spreads in energy markets, crush spreads in agricultural markets, and multi-asset structured products. These derivatives are useful for hedging relative price movements and managing complex risk exposures, but unlike the single-asset Black–Scholes model, they usually do not admit simple closed-form pricing formulas.

The main mathematical challenge comes from the exercise boundary. Once a payoff involves several assets with positive and negative weights, valuation depends on sums and differences of correlated log-normal random variables. This makes exact pricing difficult, especially when fast calculations are needed for repeated pricing, calibration, or risk management. This project focuses on approximation methods that keep the pricing problem analytically manageable while still staying close to Monte Carlo benchmark prices.

---

## Main Contributions

This project develops and implements a basket pricing framework with the following main contributions:

- expresses basket option prices as linear combinations of probabilities of Gaussian events
- derives two useful representations for standard basket options:
  - an exercise boundary integral representation
  - a representation in terms of sums of correlated log-normal random variables
- implements two fast analytical approximation methods:
  - a three-moment shifted log-normal approximation
  - an averaging/projection Black–Scholes-style approximation
- compares both analytical methods against Monte Carlo simulation
- applies the framework to soybean crush spread options using historical data

A central practical result is that the averaging/projection method performs very well in the empirical study and remains close to Monte Carlo prices while being much faster to compute.

---

## Repository Structure

A suggested repository structure is:

```text
.
├── README.md
├── paper/
│   └── Arthur_Carmona_Durrleman.pdf
├── R/
│   ├── data_preparation.R
│   ├── monte_carlo_pricing.R
│   ├── moment_matching.R
│   ├── avg_bs_method.R
│   ├── strike_grid_analysis.R
│   ├── maturity_analysis.R
│   └── plots_tables.R
├── data/
│   └── README.md
├── figures/
├── results/
└── references/
```

### Folder description

- `paper/` contains the working paper with the theory, derivations, and numerical study
- `R/` contains scripts for calibration, pricing, simulations, tables, and plots
- `data/` contains input data or notes explaining how the data should be organized
- `figures/` contains exported charts for strike comparisons, convergence analysis, and maturity sensitivity
- `results/` contains numerical outputs such as price tables, error summaries, and comparison metrics
- `references/` can contain supporting notes or bibliography files if needed

---

## Mathematical Background

We consider a basket option written on `n` assets whose dynamics follow correlated geometric Brownian motions under the risk-neutral measure:

```math
dS_i(t) = r S_i(t)dt + \sigma_i S_i(t)dW_i(t), \qquad i=1,\dots,n,
```

where `r` is the risk-free rate, `\sigma_i` is the volatility of asset `i`, and the Brownian motions are correlated. The basket call payoff at maturity `T` is

```math
C(T) = e^{-rT}\, \mathbb{E}\left(\sum_{i=1}^n \omega_i S_i(T) - K\right)^+,
```

where `\omega_i` are the basket weights and `K` is the strike.

The paper distinguishes between two important basket types:

- **general basket**: weights may be positive or negative without restriction
- **standard basket**: exactly one weight is positive and the others are negative

This distinction matters because the paper develops its main analytical results for standard basket options and then discusses how some non-standard cases can be transformed into that setting.

---

## Pricing Framework

### 1. Probability Reformulation

A first key step in the paper is to rewrite the basket option price as a linear combination of probabilities of Gaussian events. This reformulation separates the structural coefficients of the basket from the probability terms that need to be approximated.

This point of view is useful because it makes the pricing problem easier to analyze and gives a common framework for different approximation methods.

### 2. Exercise Boundary Representation

For standard basket options, the paper derives an exercise boundary integral representation. This connects the problem to the classical literature that approximates the boundary through Taylor expansion. The representation is mathematically elegant and clarifies how the geometry of the exercise region enters the pricing formula.

At the same time, this method has practical limits:
- it still requires multidimensional integration
- computational cost grows with dimension
- accuracy depends on how well the boundary is approximated

### 3. Sum-of-Lognormals Representation

A second major representation expresses the standard basket price in terms of distribution functions of sums of positively weighted correlated log-normal random variables. This is important because it connects basket pricing to the broader literature on approximating sums of lognormals.

This representation is the starting point for the two fast analytical methods implemented in the project.

---

## Methods

## Monte Carlo Benchmark

Monte Carlo simulation is used as the benchmark pricing method. Under the risk-neutral measure, correlated normal draws are simulated, terminal asset prices are generated, and discounted payoffs are averaged. This gives a flexible and accurate benchmark, but it is computationally more expensive than analytical approximations.

Monte Carlo is especially useful here because it provides a reference for checking how close the approximation methods are across strikes and maturities.

## Method 1: Three-Moment Matching

The first analytical method approximates each sum of correlated log-normal random variables by a shifted log-normal random variable. The parameters of the shifted log-normal distribution are chosen by matching the first three moments.

This approach is attractive because:
- the required moments can be computed in closed form
- the approximation becomes fully analytical once the moments are known
- the method captures skewness better than a simple two-moment fit

However, the empirical results in the paper show that this method tends to overprice options, especially for out-of-the-money strikes and longer maturities.

## Method 2: Averaging / Projection Method (Avg-BS)

The second analytical method is the main practical contribution of the paper. It reduces the multidimensional pricing problem to a sequence of single-factor Black–Scholes-style calculations.

The idea is to:
1. normalize the coefficients in each probability term
2. interpret them as portfolio weights
3. project the correlated log-normal sum onto an effective single-factor log-normal variable
4. compute the approximate probability with a standard normal CDF

This method is simple, fast, and scales well with dimension. In the numerical study, it performs much better than the moment-matching method and stays very close to Monte Carlo prices.

---

## Data and Calibration

The empirical application in the paper focuses on the **soybean crush spread**, which represents the margin from processing soybeans into soybean meal and soybean oil. In the paper, the processed products are aggregated into a composite asset, so the pricing problem becomes a two-asset spread option.

The calibration uses:
- 15 years of daily settlement prices
- 3,776 daily observations from 2010 to 2024
- annualized volatility estimates
- correlation estimated from daily log-returns

The main calibration values used in the paper are:
- `\sigma_1 = 0.18` for soybeans
- `\sigma_2 = 0.20` for the soybean oil + meal composite
- `\rho = 0.80`
- risk-free rate `r = 0`

The paper studies 30-day spread call options at three strike levels:
- ITM: `0.8 × ATM`
- ATM
- OTM: `1.2 × ATM`

It also examines sensitivity across maturities `7`, `30`, `90`, and `180` days.

---

## Numerical Results

## Strike Grid Comparison

The strike grid analysis compares Monte Carlo, the averaging/projection method, and the three-moment matching method for 30-day options.

Main findings:
- the **Avg-BS** method achieves very small pricing errors relative to Monte Carlo
- its mean relative error is about **2.28%**
- the **moment matching** method is less accurate, with mean relative error around **12.57%**
- moment matching systematically overprices, especially for OTM options

This is one of the main practical conclusions of the paper: the projection-based method is the stronger approximation in this setting.

## Monte Carlo Convergence

The paper also studies Monte Carlo convergence for the ATM 30-day option by increasing the number of simulation paths.

Main findings:
- Monte Carlo prices stabilize around `0.1704`
- convergence is already strong by around `100,000` paths
- analytical methods remain fixed because they are deterministic
- Avg-BS stays close to the converged Monte Carlo value
- moment matching remains noticeably above it

This confirms both the quality of the Monte Carlo benchmark and the accuracy advantage of Avg-BS over the moment-based method.

## Sensitivity to Maturity

The paper evaluates option prices for maturities of `7`, `30`, `90`, and `180` days.

Main findings:
- Avg-BS tracks Monte Carlo well across all maturities
- relative errors remain small and stable
- moment matching becomes less accurate as maturity increases
- the gap is especially visible for OTM options
- both methods perform better for ITM options than for OTM options

Overall, the maturity study shows that the projection method is robust across different option horizons.

---

## Why the Avg-BS Method Matters

A major message of the paper is that the averaging/projection method offers a strong balance between speed and accuracy.

Its advantages are:
- fully analytical once the effective parameters are computed
- requires only standard normal CDF evaluations
- avoids costly multidimensional integration
- scales naturally to higher-dimensional baskets
- remains close to Monte Carlo in the empirical application

Because of these features, the method is attractive for practical pricing, repeated valuation tasks, and real-time risk applications.

---

## How to Run the Code

1. Clone this repository.
2. Open the project in **RStudio**.
3. Put the data files in the `data/` folder or follow the instructions in `data/README.md`.
4. Install the required R packages.
5. Run the scripts in the following order:

```r
# Suggested workflow
source("R/data_preparation.R")
source("R/monte_carlo_pricing.R")
source("R/moment_matching.R")
source("R/avg_bs_method.R")
source("R/strike_grid_analysis.R")
source("R/maturity_analysis.R")
source("R/plots_tables.R")
```

6. Check the generated tables in `results/` and exported plots in `figures/`.

---

## Required R Packages

Depending on your final scripts, you may need packages such as:

```r
install.packages(c(
  "tidyverse",
  "data.table",
  "mvtnorm",
  "ggplot2",
  "Matrix"
))
```

If you use other packages for simulation, optimization, or table formatting, add them here.

---

## Expected Output

The code in this repository is expected to produce:

- Monte Carlo benchmark prices
- analytical prices from the three-moment matching method
- analytical prices from the Avg-BS method
- tables of absolute and relative pricing errors
- Monte Carlo convergence tables and plots
- maturity sensitivity tables and plots
- figures comparing the three methods across strikes

---

## Paper

The full methodology, derivations, and empirical validation are provided in the working paper:

**Arthur_Carmona_Durrleman.pdf**

This paper is the main reference for the repository and should be read for the full mathematical details.

---

## Authors

- Steve Tchoneteck
- Sophia Zorek
- Hasanjan Sayit
- Frederi Viens

---

## Citation

If you use this repository or build on the methods, please cite the working paper:

```text
Tchoneteck, S., Zorek, S., Sayit, H., and Viens, F.
On Basket Pricing Problems Under Log-Normal Models: Reviews and New Insights.
Working paper, 2026.
```

---

## Notes

This repository can be extended in several directions, including:

- higher-dimensional basket options
- additional empirical applications in energy or agricultural spreads
- further comparison with other analytical approximations
- calibration and hedging studies
- sensitivity analysis with respect to correlation and volatility structure

The current project emphasizes simple, fast, and interpretable approximations that remain close to benchmark Monte Carlo prices.
