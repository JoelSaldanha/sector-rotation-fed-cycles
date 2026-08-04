> Originally built as group coursework for LSE DS105, March to May 2026. Republished under my personal account after grade release, with permission from co-authors Hugo Whyte and Parthiv Chakadath. Awarded First Class.

# Sector Rotation & Federal Reserve Rate Cycles

**Live website:** joelsaldanha.github.io/sector-rotation-fed-cycles

Python data pipeline analysing US equity sector rotation across Federal Reserve rate cycles | FRED & Alpha Vantage APIs | SQLite | GitHub Pages

**Research Question:** How do US equity sector returns vary across Federal Reserve rate hiking and cutting cycles, and which sectors consistently rotate into outperformance as monetary policy shifts?

We analyse whether the direction of the Federal Funds Rate can explain cross-sectional differences in US equity sector performance, using the 10 S&P 500 SPDR sector ETFs (XLK, XLF, XLE, XLV, XLU, XLY, XLP, XLI, XLB, XLRE).

## Headline Finding

Cyclical sectors (Technology, Industrials, Financials, Consumer Discretionary) earned higher annualised returns when the Fed was hiking than when it was cutting. Defensives (Health Care, Consumer Staples) and Real Estate showed the opposite pattern. The finding holds across a 26-year sample (2000–2026), but with an important caveat: every cutting cycle in the data coincides with a US recession (dot-com bust, GFC, COVID), so cutting-cycle returns reflect crisis dynamics rather than normal monetary easing.

## Team

| Member | Role | Notebook |
|--------|------|----------|
| Hugo Whyte | Data collection, FRED & Alpha Vantage API, and README | NB01, README |
| Parthiv Chakadath | Cleaning, transformation & SQLite database | NB02 |
| Joel Saldanha | Analysis, visualisations & GitHub Pages website | NB03 |

## Data Sources

| Series | Source | ID |
|--------|--------|----|
| Federal Funds Rate | FRED | `FEDFUNDS` |
| CPI Inflation | FRED | `CPIAUCSL` |
| 10Y–2Y Yield Spread | FRED | `T10Y2Y` |
| Sector ETF monthly prices | Alpha Vantage | `TIME_SERIES_MONTHLY_ADJUSTED` |

All data is collected programmatically via Python's `requests` library. API keys are stored in `.env` (never hardcoded).

## Pipeline

**NB01 — Collection**
- `requests.get()` to FRED and Alpha Vantage APIs
- `time.sleep()` for rate limiting
- Raw responses saved as JSON to `data/raw/` before any transformation

**NB02 — Cleaning & Storage**
- `pd.json_normalize()` to flatten FRED responses
- `pd.DataFrame.from_dict()` to flatten Alpha Vantage time series
- Date conventions reconciled (FRED: first of month; Alpha Vantage: last trading day) via monthly period conversion
- Rate cycle labelled with `np.select()` on a rolling 3-month mean of `fed_funds.diff()`
- `pd.merge()` joins macro and sector data on date
- Written to SQLite with explicit `CREATE TABLE`, primary and foreign keys, and `PRAGMA foreign_keys = ON`

**NB03 — Analysis**
- Reads from SQLite via `pd.read_sql()` with an `INNER JOIN`
- `.groupby().agg()` computes mean, median, std, and n per sector × cycle
- `.pivot_table()` builds the sector × cycle comparison matrix
- Three visualisations produced with Seaborn and Matplotlib, saved to `docs/figures/`

## Database Schema

Two normalised tables linked by a foreign key on `date`:

```
macro_indicators  — date (PK), fed_funds, cpi, yield_spread, rate_cycle
sector_returns    — id (PK), date (FK), sector, monthly_return
```

Rather than repeating macro context across all ten sector rows, we store it once per month — the normalisation decision is documented in NB02.

## Visualisations

Three interactive Plotly charts on the [live website](https://lse-ds105.github.io/group-project-git-it-done/):

1. **Bar chart** — annualised return per sector grouped by hiking vs cutting cycle, sorted left-to-right by hiking performance. Neutral regime excluded (41 months, clustered in late-cycle bull runs; see NB03 Decision 5).
2. **Heatmap** — full sector × cycle matrix with conditional formatting. Includes the neutral column and the XLRE small-sample caveat.
3. **Fed Funds timeline** — Federal Funds Rate plotted over time with shaded regime periods. Shows that every cutting cycle in the sample (2001, 2008–09, 2020) coincides with a recession.

Static PNG fallbacks for each chart are in `docs/figures/` for users with JavaScript disabled.

## Limitations

Four caveats apply to the analysis. Each is documented at the point in NB03 where it affects the output, not collected in a footer.

1. **Cutting cycles are crisis-driven.** The three cutting episodes in the sample (2001, 2008–09, 2020) all coincide with US recessions. Cutting-cycle returns here reflect crisis dynamics rather than normal monetary easing. A sample including a non-recessionary cut would likely look different.
2. **XLRE has a short history.** Real Estate (XLRE) launched in October 2015 and has 124 observations versus 315 for every other sector. Its cutting-cycle figure reflects the 2020 COVID episode alone and should be treated as one event rather than a pattern.
3. **Neutral sample is small.** The neutral regime covers only 41 months across 10 sectors, clustered in calm late-cycle periods. Neutral cells in the heatmap are included for completeness but shouldn't be read as a reliable regime estimate.
4. **Rate-cycle labelling is noisy during flat-rate periods.** The rolling 3-month mean rule produces label flips during periods when the Fed held rates flat (2010–2015, 2021–2022), because the pure sign classification has no magnitude threshold. The per-cycle averages don't shift materially (each regime averages over 100+ months), but the issue is documented in [Issue #9](https://github.com/lse-ds105/group-project-git-it-done/issues/9) as a candidate future improvement.

## Reproduction

```bash
# 1. Clone the repository
git clone https://github.com/lse-ds105/group-project-git-it-done.git
cd group-project-git-it-done

# 2. Create a .env file in the repo root with your API keys
# FRED does not require a key for public series
# Alpha Vantage free tier requires one key
echo "ALPHAVANTAGE_API_KEY=your_key_here" > .env

# 3. Run notebooks in order
# NB01 writes raw JSON to data/raw/
# NB02 reads from data/raw/ and writes sector_rotation.db to data/processed/
# NB03 reads from sector_rotation.db and writes figures to docs/figures/
jupyter notebook notebooks/NB01_collect.ipynb
jupyter notebook notebooks/NB02-Data-Transformation.ipynb
jupyter notebook notebooks/NB03_analyse.ipynb
```

**Known data gaps:** October 2025 CPI is null (BLS publication gap following the US federal shutdown). April 2026 macro row was dropped (FRED publication lag). Both are documented in NB02 Decision 6 and handled by the pipeline without manual intervention.

## Repository Structure

```
repo/
├── .env                        # API keys (not tracked)
├── .gitignore
├── data/
│   ├── raw/                    # JSON responses from FRED and Alpha Vantage
│   └── processed/
│       ├── sector_rotation.db  # SQLite database (NB02 output)
│       └── analysis/           # cycle_returns.csv, sector_cycle_pivot.csv (NB03 output)
├── docs/
│   ├── index.html              # GitHub Pages custom HTML website
│   ├── style.css               # Website styling
│   └── figures/                # PNG fallbacks for noscript
├── notebooks/
│   ├── NB01_collect.ipynb
│   ├── NB02-Data-Transformation.ipynb
│   └── NB03_analyse.ipynb
├── reflections/                # one .md per team member
├── requirements.txt            # Pinned Python dependencies for reproduction
└── README.md
```
  
