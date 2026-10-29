# Kelly Position Sizing on AAPL: Educational Notebook

An educational Jupyter notebook that extends a coin-flip betting simulation to real AAPL daily returns. Its central question is whether bet size matters more than whether an edge exists, and how much estimation error in the edge threatens a Kelly-sized strategy.

> **Disclaimer:** This project is for education only. It is not investment advice and does not recommend trading AAPL or any other security. Past performance does not predict future returns.

---

## Table of Contents

- [Background](#background)
- [What the Notebook Does](#what-the-notebook-does)
- [Repository Contents](#repository-contents)
- [Requirements](#requirements)
- [Installation](#installation)
- [Running the Notebook](#running-the-notebook)
- [Configuration](#configuration)
- [Section-by-Section Guide](#section-by-section-guide)
- [Interpreting the Output](#interpreting-the-output)
- [Methodology Notes](#methodology-notes)
- [Limitations](#limitations)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Background

The Kelly criterion gives the bet fraction that maximizes the long-run growth rate of wealth. For a bet with win probability *p*, loss probability *q = 1 − p*, and payoff ratio *b* (average win divided by average loss in magnitude), the Kelly fraction is:

$$f^* = \frac{bp - q}{b}$$

For an even-money coin flip, *b = 1*, so this reduces to *f\* = p − q*. For a 55% coin, that is 10% of wealth per bet.

Two lessons motivate the project:

1. **Bet size dominates.** Given the same edge, the bet fraction determines whether wealth compounds or collapses. Overbetting is far more damaging than underbetting.
2. **The edge is estimated, not known.** Real Kelly sizing depends on a *p* and *b* taken from data. Errors in that estimate, not the absence of an edge, are the main practical risk.

---

## What the Notebook Does

The notebook runs in ten sections:

| # | Section | Purpose |
|---|---------|---------|
| 1 | Introduction and setup | Goal, disclaimer, imports, random seed |
| 2 | Download AAPL data | Pull 10 years of adjusted closes via yfinance |
| 3 | Data quality checks | Handle missing values, duplicate dates, non-positive prices |
| 4 | Returns and statistics | Simple and log returns, Sharpe ratio, fat-tail histogram |
| 5 | Estimate the edge | Win rate *p*, payoff ratio *b*, Kelly fraction *f\** |
| 6 | Estimation uncertainty | 2,000-sample bootstrap of win rate and Kelly fraction |
| 7 | Coin-flip baseline | Reproduce the 55% experiment across five bet sizes |
| 8 | AAPL fractional betting | Block bootstrap of real returns, fixed and Kelly-based sizing |
| 9 | Overbetting and error | True edge 5%, sized as 10%; growth curves and drawdowns |
| 10 | Conclusions and caveats | Findings, omitted effects, closing reminder |

---

## Repository Contents

```
.
├── README.md                      # This file
├── kelly_aapl_position_sizing.ipynb   # Main notebook (10 sections)
└── requirements.txt               # Python dependencies
```

`requirements.txt`:

```
numpy
pandas
matplotlib
scipy
yfinance
jupyter
```

---

## Requirements

- Python 3.9 or newer
- Internet access (the notebook downloads AAPL prices from Yahoo Finance at runtime)
- Roughly 200 MB of disk space for dependencies
- Runs on macOS, Linux, or Windows

No API keys or accounts are needed.

---

## Installation

Create and activate a virtual environment, then install the dependencies:

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

---

## Running the Notebook

Launch Jupyter from the repository folder:

```bash
jupyter notebook kelly_aapl_position_sizing.ipynb
```

Then choose **Kernel → Restart & Run All**. Cells must run in order in a single kernel. Running out of order will produce errors or stale values.

The full notebook takes a few minutes, mostly for the bootstrap and the 1,000-player simulations.

---

## Configuration

Key settings are defined in the setup cell (Section 1) and can be changed there:

| Variable | Default | Meaning |
|----------|---------|---------|
| `SEED` | `42` | Random seed for reproducibility |
| `TICKER` | `"AAPL"` | Stock symbol to download |
| `YEARS` | `10` | Length of the historical window |
| `RISK_FREE_RATE` | `0.0` | Risk-free rate used in the Sharpe ratio |
| `N_BOOT` | `2000` | Bootstrap resamples for Section 6 |
| `N_PLAYERS` / `N_FLIPS` | `1000` / `1000` | Simulation size for the coin-flip baseline |
| `BLOCK` | `10` | Block length (days) for block bootstrap |
| `TRUE_EDGE` / `SIZED_EDGE` | `0.05` / `0.10` | True and assumed edge in Section 9 |

Because `YEARS` and the ticker are configurable, you can point the analysis at other symbols. Results will change, and the conclusions in Section 10 may not apply to them.

---

## Section-by-Section Guide

**Section 2 (download).** Uses `auto_adjust=True`, so prices are dividend- and split-adjusted. If Yahoo Finance is unreachable or returns nothing, the notebook raises a `RuntimeError` with a clear message.

**Section 3 (data quality).** Missing prices are forward-filled, since a gap usually means no trade occurred. Duplicate dates keep the first occurrence. Non-positive prices are dropped because they make log returns undefined. Each action is printed.

**Section 5 (edge estimation).** Defines a win as a positive simple return. Note that the Kelly fraction here is a plug-in estimate from one historical sample.

**Section 6 (uncertainty).** Resamples daily returns with replacement and reports the 5th and 95th percentiles of the bootstrap Kelly fractions. A wide interval relative to the point estimate signals that the edge is poorly identified.

**Section 8 (AAPL simulation).** Uses 10-day block bootstrap to partially preserve autocorrelation and volatility clustering. Wealth is floored at zero so no player goes negative.

**Section 9 (overbetting).** Simulates a player who believes the edge is 10% when it is actually 5%. Plots the expected log growth function *g(f) = p·log(1+f) + q·log(1−f)* for both edges, with each Kelly peak marked.

---

## Interpreting the Output

- **Median final wealth** is more informative than the mean, since a few lucky players can skew the average.
- **Fraction below start** and **fraction lost over 99%** measure how often sizing leads to loss or near-ruin.
- **Bootstrap Kelly interval width** compares the uncertainty in *f\** to its point estimate. A wide interval means the sizing decision rests on weak evidence.
- **Log growth curves** show that the growth-maximizing bet is not the largest one. Past the Kelly peak, growth falls, and it turns negative at roughly twice the Kelly fraction.

Results are random-seed dependent in the simulations. The fixed seed makes them reproducible on your machine, but a different seed will give somewhat different numbers.

---

## Methodology Notes

- **Daily returns are not independent.** AAPL exhibits volatility clustering and regime shifts. The plug-in Kelly fraction assumes a stable edge, which the notebook tests rather than assumes.
- **Block bootstrap** keeps short-range dependence but assumes the past distribution holds across the simulated horizon.
- **Even-money approximation.** The coin-flip comparison uses *b = 1*. The AAPL simulation uses actual returns, so *b* is not fixed there.
- **Fixed-fraction betting** applies the same percentage of current wealth to each return, so position sizes scale with wealth.

---

## Limitations

The notebook does **not** model:

- Survivorship bias (it uses only AAPL, a company that survived and grew)
- Look-ahead bias in edge estimation
- Transaction costs, bid-ask spreads, and slippage
- Taxes and financing costs
- Regime change and structural breaks
- Leverage limits and margin calls

Because AAPL is a successful stock, the sample is biased toward favorable outcomes. Treat any positive result as a best case.

---

## Troubleshooting

| Problem | Likely cause | Fix |
|---------|--------------|-----|
| `RuntimeError: yfinance download failed` | No internet, or Yahoo Finance rate limit | Check connection; wait and rerun |
| `No data returned for AAPL` | Empty response, often a network block | Confirm access to finance.yahoo.com |
| Plots don't appear | Notebook not rendering inline | Add `%matplotlib inline` or use JupyterLab |
| Numbers differ from README | yfinance updated prices, or the window moved | Expected; the window ends at the latest trading day |
| Errors after editing a cell | Out-of-order execution | Restart kernel and Run All |

---

## Contributing

Contributions are welcome. Useful directions include:

- Adding transaction-cost and slippage models
- Testing other tickers or asset classes
- Adding a regime-switching model to the simulations
- Improving the block bootstrap (e.g., stationary bootstrap)

Please open an issue before large changes, and keep the educational framing and disclaimer intact.

---

## License

Specify a license before publishing (for example, MIT). Until one is added, all rights are reserved by the author.

---

## Acknowledgments

The coin-flip experiment that motivates this notebook follows the standard Kelly-criterion presentation. Market data is provided by Yahoo Finance via the `yfinance` package, which is not affiliated with Yahoo.
