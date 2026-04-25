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
3. Put the data file `soy_crush_20y.csv` in the project root, or adjust the path inside the R Markdown / R script.
4. Install the required R packages.
5. Run the main R Markdown or R script that compares all pricing methods and generates the tables and figures.

### Main R workflow included in this project

The main R code loads soybean crush spread data, estimates annualized volatilities and correlation from recent log-returns, and then compares four methods:

- **MC**: Monte Carlo benchmark
- **Avg_BS**: Section 5.1.2 averaging / projection method
- **MM_Spread**: optional 3-moment matching applied directly to the spread \(Y = S_2 - S_1\)
- **MM_511_Formula55**: Section 5.1.1 moment-based approximation using \(\psi\), \(\eta\), \(\Theta\), and Cardano-style shifted lognormal fitting

The code also:
- builds the strike-grid comparison table
- performs Monte Carlo convergence analysis
- compares prices across maturities
- produces the strike-profile comparison
- saves output tables as CSV files

### R code block

```r
###############################################################################
# Complete Comparison:
#   - MC (benchmark)                          [OK]
#   - Avg_BS  = Section 5.1.2 (your code)     [OK]
#   - MM_Spread = your 3-moment match of Y=S2-S1 (kept)
#   - MM_511_Formula55 = Section 5.1.1 / Formula (55) using ψ,η,Θ + Cardano fit
###############################################################################

suppressPackageStartupMessages({
  if (!requireNamespace("data.table", quietly = TRUE)) install.packages("data.table")
  if (!requireNamespace("MASS",       quietly = TRUE)) install.packages("MASS")
  if (!requireNamespace("statmod",    quietly = TRUE)) install.packages("statmod")
  if (!requireNamespace("ggplot2",    quietly = TRUE)) install.packages("ggplot2")
  if (!requireNamespace("reshape2",   quietly = TRUE)) install.packages("reshape2")
})
library(data.table)
library(MASS)
library(statmod)
library(ggplot2)
library(reshape2)

csv_path <- "soy_crush_20y.csv"
DT <- fread(csv_path)
setnames(DT, tolower(names(DT)))
stopifnot(all(c("date","s1","s2") %in% names(DT)))
DT <- DT[, .(date = as.IDate(date), S1 = as.numeric(s1), S2 = as.numeric(s2))]
DT <- DT[is.finite(S1) & is.finite(S2)]
setorder(DT, date)

ann   <- 252
Tdays <- 30
T     <- Tdays / ann
r     <- 0

S1_0 <- DT$S1[nrow(DT)]
S2_0 <- DT$S2[nrow(DT)]
ATM  <- max(S2_0 - S1_0, 1e-10)

ret <- DT[, .(lS1 = c(NA, diff(log(S1))), lS2 = c(NA, diff(log(S2))))]
ret <- ret[complete.cases(ret)]
sub <- tail(ret, min(252L, nrow(ret)))
covm <- cov(sub)
vols <- sqrt(diag(covm))
rho  <- covm[1,2] / (vols[1]*vols[2] + 1e-16)
sigma1 <- vols[1] * sqrt(ann)
sigma2 <- vols[2] * sqrt(ann)

mc_spread_call <- function(S1_0, S2_0, sigma1, sigma2, rho, T, K, r=0,
                           n_paths=4e5, seed=123) {
  set.seed(seed)
  Z1 <- rnorm(n_paths)
  Z2 <- rnorm(n_paths)
  Z2c <- rho * Z1 + sqrt(max(1 - rho^2, 0)) * Z2
  S1T <- S1_0 * exp((r - 0.5 * sigma1^2) * T + sigma1 * sqrt(T) * Z1)
  S2T <- S2_0 * exp((r - 0.5 * sigma2^2) * T + sigma2 * sqrt(T) * Z2c)
  payoff <- pmax(S2T - S1T - K, 0)
  mean(payoff) * exp(-r * T)
}

fit_shifted_lognormal_closedform <- function(mu, var, mu3) {
  mu  <- as.numeric(mu)[1]
  var <- as.numeric(var)[1]
  mu3 <- as.numeric(mu3)[1]
  if (!is.finite(var) || var <= 0) var <- 1e-12
  if (!is.finite(mu3)) mu3 <- 0

  csgn <- if (mu3 >= 0) 1 else -1
  gamma <- mu3 / (var^(3/2) + 1e-18)
  gabs  <- abs(gamma)

  if (!is.finite(gabs) || gabs < 1e-10) {
    x <- 1 + 1e-12
  } else {
    A <- 1 + 0.5*gabs^2
    B <- gabs * sqrt(1 + 0.25*gabs^2)
    x <- (A + B)^(1/3) + (A - B)^(1/3) - 1
    if (!is.finite(x) || x <= 1) x <- 1 + 1e-12
  }

  s <- sqrt(log(x))
  m <- 0.5 * log(var / (x*(x-1)) + 1e-300)
  tau <- csgn*mu - sqrt(var) / sqrt(x - 1 + 1e-18)
  list(c = csgn, s = s, m = m, tau = tau)
}

prob_W_leq_1 <- function(csgn, s, m, tau) {
  if (!is.finite(s) || s <= 0) return(as.numeric(tau <= 1))
  if (csgn > 0) {
    if (tau >= 1) return(0)
    return(pnorm((log(1 - tau) - m) / s))
  } else {
    rhs <- -1 - tau
    if (rhs <= 0) return(0)
    return(pnorm((log(rhs) - m) / s))
  }
}

build_eta_theta <- function(sigma1, sigma2, rho, T) {
  eta1 <- 0
  eta2 <- sqrt(pmax(sigma2^2*T + sigma1^2*T - 2*rho*sigma1*sigma2*T, 0))
  denom <- sqrt(pmax(sigma2^2 + sigma1^2 - 2*rho*sigma1*sigma2, 1e-18))
  theta12 <- (sigma2*rho - sigma1) / denom
  Theta <- matrix(c(1, theta12, theta12, 1), 2, 2)
  list(eta = c(eta1, eta2), Theta = Theta)
}

build_psi_spread <- function(S1_0, S2_0, sigma1, sigma2, rho, K, T, r=0) {
  kdisc <- K * exp(-r*T)
  s_plus  <- S2_0
  s_minus <- S1_0

  s11 <- s_plus  * exp(sigma2^2 * T)
  s12 <- s_minus * exp(rho*sigma1*sigma2*T)
  s21 <- s_plus  * exp(rho*sigma1*sigma2*T)
  s22 <- s_minus * exp(sigma1^2 * T)
  s31 <- s_plus
  s32 <- s_minus
  ref <- sigma1

  Psi <- rbind(
    c((kdisc / s11) * exp(ref^2 * T), (s12 / s11) * exp(ref^2 * T - rho*ref*sigma2*T)),
    c((kdisc / s21) * exp(ref^2 * T), (s22 / s21) * exp(ref^2 * T - rho*ref*sigma2*T)),
    c((kdisc / s31) * exp(ref^2 * T), (s32 / s31) * exp(ref^2 * T - rho*ref*sigma2*T))
  )

  list(Psi = Psi, s_plus = s_plus, s_minus = s_minus, kdisc = kdisc)
}

price_avg_bs <- function(S1_0, S2_0, sigma1, sigma2, rho, K, T, r=0) {
  et <- build_eta_theta(sigma1, sigma2, rho, T)
  ps <- build_psi_spread(S1_0, S2_0, sigma1, sigma2, rho, K, T, r)
  cvec <- rowSums(ps$Psi)
  Pvec <- numeric(3)
  for (i in 1:3) {
    hi <- ps$Psi[i, ] / cvec[i]
    denom <- sqrt(pmax(as.numeric(hi %*% et$Theta %*% hi), 1e-18))
    di <- pmax(sum(hi * et$eta) / denom, 1e-12)
    Pvec[i] <- pnorm(log(1/cvec[i])/di + 0.5*di)
  }
  ps$s_plus * Pvec[1] - ps$s_minus * Pvec[2] - ps$kdisc * Pvec[3]
}

moments_Zi_from_psi_eta_theta <- function(psi_row, eta, Theta) {
  n <- length(psi_row)
  M1 <- sum(psi_row)
  M2 <- 0
  M3 <- 0
  for (j in 1:n) for (k in 1:n)
    M2 <- M2 + psi_row[j]*psi_row[k] * exp(eta[j]*eta[k]*Theta[j,k])
  for (a in 1:n) for (b in 1:n) for (c in 1:n) {
    expo <- eta[a]*eta[b]*Theta[a,b] + eta[a]*eta[c]*Theta[a,c] + eta[b]*eta[c]*Theta[b,c]
    M3 <- M3 + psi_row[a]*psi_row[b]*psi_row[c] * exp(expo)
  }
  list(mu = M1, var = max(M2 - M1^2, 1e-12), mu3 = M3 - 3*M1*M2 + 2*M1^3)
}

price_formula55_511 <- function(S1_0, S2_0, sigma1, sigma2, rho, K, T, r=0) {
  et <- build_eta_theta(sigma1, sigma2, rho, T)
  ps <- build_psi_spread(S1_0, S2_0, sigma1, sigma2, rho, K, T, r)
  PZi <- numeric(3)
  for (i in 1:3) {
    mom <- moments_Zi_from_psi_eta_theta(ps$Psi[i,], et$eta, et$Theta)
    par <- fit_shifted_lognormal_closedform(mom$mu, mom$var, mom$mu3)
    PZi[i] <- prob_W_leq_1(par$c, par$s, par$m, par$tau)
  }
  ps$s_plus * PZi[1] - ps$s_minus * PZi[2] - ps$kdisc * PZi[3]
}

strikes <- c(0.8*ATM, ATM, 1.2*ATM)
comparison <- data.table(
  K = strikes,
  MC = sapply(seq_along(strikes), function(i)
    mc_spread_call(S1_0, S2_0, sigma1, sigma2, rho, T, strikes[i], r, n_paths=4e5, seed=100+i)),
  Avg_BS_512 = sapply(strikes, function(K)
    price_avg_bs(S1_0, S2_0, sigma1, sigma2, rho, K, T, r)),
  MM_511_Formula55 = sapply(strikes, function(K)
    price_formula55_511(S1_0, S2_0, sigma1, sigma2, rho, K, T, r))
)

print(comparison)
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
