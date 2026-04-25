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

