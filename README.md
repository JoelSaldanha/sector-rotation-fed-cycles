[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/RENlL0QN)
# a-nearly-empty-repo

# sector-rotation-analysis
Python data pipeline analysing US equity sector rotation across Federal Reserve rate cycles | FRED &amp; Alpha Vantage APIs | SQLite | GitHub Pages

**Sector Rotation & Federal Reserve Rate Cycles**
Research Question

"How do US equity sector returns vary across Federal Reserve rate hiking and cutting cycles, and which sectors consistently rotate into outperformance as monetary policy shifts?"

We analyse whether monetary policy regime, as defined by the direction of the Federal Funds Rate, can explain cross-sectional differences in US equity sector performance, using the 10 S&P 500 SPDR sector ETFs (XLK, XLF, XLE, XLV, XLU, XLY, XLP, XLI, XLB, XLRE).

Team Members & Roles
Joel: Analysis, visualisations & GitHub Pages website
Hugo: Data collection, FRED & Alpha Vantage API notebooks
Parthiv: Data cleaning, transformation & SQLite database

Data Sources
Federal Funds Rate    FRED                Series FEDFUNDS
CPI Inflation         FRED                Series CPIAUCSL
10Y-2Y Yield Spread   FRED                Series T10Y2Y   
Sector ETF prices     Alpha Vantage       TIME_SERIES_MONTHLY

All data collected programmatically using Python's requests library. API keys stored in .env (never hardcoded).

**Pipeline Steps**
01_collect.ipynb
   └── requests.get() to FRED and Alpha Vantage
   └── time.sleep() for rate limiting
   └── Save raw JSON to data/raw/

02_clean_store.ipynb
   └── pd.json_normalize() to flatten responses
   └── np.select() to label rate cycle periods
   └── pd.merge() macro + sector data on date
   └── Write to SQLite database (data/processed/)

03_analyse.ipynb
   └── Read from SQLite database
   └── .groupby() + .agg() - mean returns per sector per cycle
   └── .pivot_table() - sector vs cycle comparison
   └── .melt() + Seaborn for visualisations

Database Schema
Two tables connected by foreign key on date:
macro_indicators - date (PK), fed_funds, cpi, yield_spread, rate_cycle
sector_returns - id (PK), date (FK), sector, monthly_return

Planned Visualisations (max 3)

Bar chart: average monthly return per sector, grouped by rate cycle (hiking / cutting / neutral)
Heatmap table: sectors × cycles with conditional formatting showing outperformers
Line chart: Fed Funds Rate over time with shaded cycle periods and sector overlays


Repository Structure
repo/
├── .env                  # API keys (not tracked)
├── .gitignore
├── data/
│   ├── raw/              # JSON from APIs
│   └── processed/        # SQLite database
├── docs/                 # GitHub Pages website
├── notebooks/
│   ├── 01_collect.ipynb
│   ├── 02_clean_store.ipynb
│   └── 03_analyse.ipynb
├── reflections/          # one .md per team member
└── README.md

Risk & Open Question
Risk: Alpha Vantage's free tier allows only 5 API calls per minute. With 10 sector ETFs this means our collection notebook must batch requests using time.sleep(15) between calls, and the full collection may need to run across multiple sessions.

Open question for feedback: We plan to define "hiking cycle" as any consecutive sequence of months where the Fed Funds Rate increases. Is this definition robust enough, or should we use NBER/Fed announcement dates instead?
