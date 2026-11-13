# Four signals on the US top 500, net of costs


We build four classic signals (mean-reversion, momentum, quality, value) on the 500 largest US
stocks from 2011 to 2024. Each one is simulated under rules shared by every team: quintile
long-short, 10% volatility target, execution the day after the signal, linear costs of 2, 5 or
10 bp. Results are reported **net of costs**. We then try to improve one signal: mean-reversion,
which predicts well but is too expensive to trade.

## Key results

Annualised Sharpe net of costs, each portfolio targeted at 10% volatility, 2011-2024.

| portfolio | net Sharpe 2 bp | 5 bp | 10 bp | turnover (×/yr) | break-even (bp) |
|---|---|---|---|---|---|
| mean-reversion (rebalanced daily) | −0.24 | −1.09 | −2.50 | 305 | 1.2 |
| momentum (monthly) | **0.39** | **0.35** | **0.29** | 13 | 33.4 |
| quality (monthly) | 0.22 | 0.19 | 0.14 | 10 | 24.7 |
| value (monthly) | −0.34 | −0.37 | −0.41 | 9 | −35.5 |
| 1/N blend | 0.12 | −0.38 | −1.21 | 145 | 2.7 |
| **improvement: mean-reversion rebalanced every 4 days** | **0.13** | −0.32 | −1.05 | 159 | 2.9 |

- **Momentum and quality** survive costs because they are slow: few trades, and a break-even well
  above 10 bp.
- **Value** loses money even before costs over this period: growth stocks dominated US large caps.
- **Mean-reversion** predicts, but its gross edge (1.2 bp per unit traded) is below the lowest
  cost. Rebalancing every 4 days halves turnover and makes the Sharpe positive at 2 bp, but not at
  5 bp. That frequency was chosen on the full sample, so the gain is fragile.

These figures come from running the pipeline on the MaQI platform. They are the outputs of §8 and
§9 of `02_strategy.ipynb` once executed.

## Repository structure

```
01_data.ipynb       data processing up to point-in-time → data/panel_clean.parquet
                    (loading and merging, universe and survivorship bias, point-in-time checks)
02_strategy.ipynb   data exploration, data loopholes, feature design, then the four signals,
                    simulator, results, improvement, robustness, monitoring, conclusion
requirements.txt
data/               (not versioned) the panel, built on the platform
```

Both notebooks are **self-contained**: all the code (S3 loading, features, signals, simulator,
monitoring) lives in their cells, with no import from a project package.

**Run order:** `01_data.ipynb`, then `02_strategy.ipynb`.

## Installation

```bash
pip install -r requirements.txt
```

Libraries: `pandas`, `numpy`, `scipy` (Spearman correlations), `matplotlib`, `pyarrow` (parquet),
`polars` (building the panel from S3), `jupyter`. Building the panel also uses `maqi_data`, the
MaQI platform library that provides S3 credentials. It is only available on the platform.

## Data

The data comes from **S&P Global Compustat** (snapshot `20260711`): daily prices, total-return
factors, quarterly fundamentals with their filing dates, and reference data (issue, exchange,
GICS sector). It is accessed through the MaQI platform (Wasabi S3 storage).

**It is not included in this repo.** The licence with the data provider forbids it from leaving
the platform. The `data/` folder is ignored by git, and the notebooks only display aggregates,
never an individual row. On the first run on the platform, `01_data.ipynb` rebuilds the panel
from S3 (about 20 seconds).

The construction choices are detailed in `01_data.ipynb`:

- point-in-time universe, re-selected at each year end, which avoids survivorship bias;
- fundamentals in force from their filing date.

The data loopholes and their fixes are detailed in `02_strategy.ipynb` (§4): split-adjusted
prices, total return including dividends and spin-offs, frozen prices, negative book equity, and
the issue-level versus company-level share count.

## Conclusion

On this panel, what matters is not the ability to predict but **transaction costs relative to the
gross edge**. The slow signals (momentum, quality) pass. The fast signal (mean-reversion)
predicts but does not cover its costs. Value fails on the merits over this decade.

One limitation weighs on every conclusion: over 14 years, the standard error of a Sharpe ratio is
about 0.27. Even the best result (momentum, 0.35) is therefore not statistically significant.

**Next steps, with six months:**

- validate out of sample (walk-forward, then 2025);
- trade mean-reversion more cheaply (no-trade band, volatility scaling, sector neutrality);
- weight the blend by break-even rather than 1/N;
- redefine value (sector-relative, intangible assets);
- add market impact and borrow costs to the cost model.

## Authors

Team **Ricardo**: Aymen, Gauthier. Hackathon 1, MaQI 2026-27.
