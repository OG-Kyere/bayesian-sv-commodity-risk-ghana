# Bayesian Latent Stochastic Volatility Models for Commodity Price Risk in West Africa

This repository contains the data pipeline, modelling code, backtests, and paper for a study of commodity price risk in cocoa, gold, and Brent crude oil—three markets with direct relevance to Ghana's economy.

The project compares Bayesian latent stochastic-volatility models with GARCH, EGARCH, Ornstein-Uhlenbeck, and Historical Simulation benchmarks for Value-at-Risk and Expected Shortfall forecasting.

The headline result is not that one model dominates everything. Student-t stochastic volatility passes every backtest for both cocoa and gold, while the leverage extension gives a smaller additional improvement for cocoa. Oil is more difficult: every specification tested, including the richest model, still fails at least one 99% VaR requirement. The paper gives the full comparison and interpretation.

## Repository structure

```text
paper/          Full paper (LaTeX source + compiled PDF), Elsevier elsarticle format
src/            Model estimation and backtesting code
data/raw/       Raw commodity price series (cocoa, gold, oil) plus validation series
checkpoints/    Saved rolling-window forecast results per model per commodity
outputs/        Backtest result tables and figures
notebooks/      Data cleaning, exploratory analysis, and stylised facts (Jupyter)
docs/           Data dictionary, cleaning policy, and supporting documentation
```

## Reproducing the analysis

Dependencies are listed in `requirements.txt`.

The main practical constraint is runtime. NUTS-based Bayesian SV estimation is slow: a full rolling backtest across the four SV variants for one commodity typically takes 10–25 hours on a normal laptop, depending on the regime-change periods inside the sample. GARCH, EGARCH, and OU are much faster and finish in under 30 minutes for all three commodities together.

To run only the benchmark models:

```bash
cd src
python production_runner.py --benchmark-only
```

To run the SV models for one commodity:

```bash
python production_runner.py --commodity gold --sv-only
```

The SV runner saves a checkpoint every 50 steps. If a long run is interrupted, it resumes from the most recent checkpoint instead of starting over.

## Data sources and limitations

Full provenance for every series, including the data-quality issues found during validation, is documented in `docs/data_dictionary.md`.

In brief:

- cocoa and gold use Yahoo Finance continuous futures, cross-validated against ICCO and Stooq respectively;
- oil uses FRED's Brent spot series as the primary benchmark;
- Brent is used rather than WTI because it is the more relevant international benchmark for West African oil exposure;
- Brent and WTI behaved as genuinely different risk processes during the April 2020 negative-price episode.

## Status of results

All 24 checkpoint files are present: three commodities × eight models, each with a complete rolling-window forecast series.

Each checkpoint was independently spot-checked before inclusion. Row counts match the expected out-of-sample step count exactly. OU non-convergence rates match the paper's stated 4.3% / 5.1% / 10.7% figures for cocoa / oil / gold respectively. A backtest recomputed directly from the raw cocoa SV-t-Leverage checkpoint also reproduces the paper's reported 36 violations at 99% VaR.

The compiled result tables in `outputs/tables/` are currently separated by commodity because they were generated from separate `--commodity X --sv-only` runs. A single combined 48-row file has not yet been generated in one pass, although it can be rebuilt directly from the 24 checkpoint files using the compilation step in `production_runner.py`.

One correction is worth documenting. An earlier draft had several incorrect cells in the summary table, including a case where the Kupiec and Christoffersen results were reversed for oil's best-performing model. Cross-checking the manuscript against the aggregated CSV files caught the problem. Keeping raw checkpoints, aggregate tables, and manuscript claims side by side is exactly what made that error traceable.

## Pre-committed robustness checks still outstanding

Six checks are stated in the paper but have not yet been run:

- rolling-window sensitivity at 750 / 1,000 / 1,250 days;
- exclusion of cocoa's three flagged 2024 rollover-illiquidity dates;
- cocoa using the ICCO price series instead of Yahoo;
- oil using WTI instead of Brent;
- prior sensitivity for the persistence parameter;
- GARCH order checks against (1,2) and (2,1).

These remain explicitly outstanding rather than being treated as completed.

## Author

Jonathan, independent researcher, Ghana.
