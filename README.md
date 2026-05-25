# Stock Return Predictor

A GitHub-ready Python project for predicting next-day stock returns with engineered technical features, a Random Forest model, leakage-aware time-series validation, long/short backtesting, and Monte Carlo resampling.

> Educational project only. This is not financial advice and should not be traded live without much deeper validation, slippage modeling, risk controls, and out-of-sample testing.

## What this project does

- Downloads historical adjusted OHLCV data with `yfinance`
- Engineers features:
  - lagged returns
  - moving-average ratios
  - RSI
  - rolling volatility
  - volume change
- Predicts next-day close-to-close returns
- Uses a time-series train/test split with no shuffling
- Backtests a long/short strategy against buy-and-hold
- Includes transaction costs
- Evaluates robustness with block-bootstrap Monte Carlo resampling
- Saves metrics, time series, plots, Monte Carlo samples, and model artifacts

## Project structure

```text
stock-return-predictor/
├── README.md
├── requirements.txt
├── pyproject.toml
├── run.py
├── src/stock_predictor/
│   ├── __init__.py
│   ├── backtest.py
│   ├── cli.py
│   ├── data.py
│   ├── features.py
│   ├── model.py
│   └── plotting.py
├── tests/
│   ├── test_backtest.py
│   └── test_features.py
└── reports/figures/
```

## Setup

```bash
git clone <your-repo-url>
cd stock-return-predictor
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -e .
```

Or:

```bash
pip install -r requirements.txt
```

## Run the baseline model

```bash
python run.py --ticker SPY --start 2010-01-01
```

Example with custom settings:

```bash
python run.py \
  --ticker AAPL \
  --start 2012-01-01 \
  --test-size 0.25 \
  --threshold 0.0005 \
  --transaction-cost-bps 2 \
  --n-sims 5000 \
  --block-size 20
```

## Outputs

The command saves:

```text
reports/prediction_metrics.csv
reports/backtest_stats.csv
reports/backtest_timeseries.csv
reports/monte_carlo_samples.csv
reports/monte_carlo_summary.csv
reports/figures/equity_curve.png
reports/figures/mc_excess_return_distribution.png
models/<TICKER>_random_forest.joblib
```

## Methodology notes

The model predicts `target_next_return`, which is the next day's close-to-close percentage return. Features are computed using only data available up to the current close. The train/test split is chronological, not randomized.

The trading rule is intentionally simple:

- predicted return > threshold: long
- predicted return < -threshold: short
- otherwise: flat

Transaction costs are charged on turnover. The Monte Carlo evaluation uses block bootstrap resampling of paired strategy and benchmark returns to estimate distributions of outcomes, preserving some short-run serial dependence.

## Next improvements

- Add walk-forward cross-validation
- Add purged/embargoed validation for overlapping labels
- Add LSTM or temporal convolution model
- Add market/sector features rather than single-asset features only
- Tune thresholds by train-only validation
- Add volatility targeting and position sizing
- Add short-borrow and realistic slippage assumptions
- Add experiment tracking with MLflow or Weights & Biases

## References

- Gu, Kelly, and Xiu, *Empirical Asset Pricing via Machine Learning*, Review of Financial Studies, 2020.
- López de Prado, *The 10 Reasons Most Machine Learning Funds Fail*, 2018.
- Stefan Jansen, *Machine Learning for Trading* GitHub repository.
- QuantStart, *Forecasting Financial Time Series — Part 1*.
