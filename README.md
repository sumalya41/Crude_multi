# 📊 Nowcast Lab: Transparent Commodity & Macro Nowcasts

**Nowcast Lab** is a rigorous, transparent, quantitative research framework and fully automated Python framework for building publication-aware nowcasts and indices for commodities (Crude Oil) and macroeconomic indicators (Inflation, Labor, Market Conditions). 

Built with a strict **"baselines first, walk-forward tests, and automated look-ahead checks"** philosophy, this repository prioritizes methodological rigor over black-box machine learning. Every data point is joined using exact publication dates, ensuring that no future information leaks into historical backtests.

---

## 🌟 Key Features

* 🛢️ **Crude Pressure Index:** A transparent, equal-weighted z-score index for a 1-week WTI view, built on 4 *a priori* signed components (momentum, Brent-WTI premium, USD, volatility).
* 📈 **Macro Nowcasts:** Month-to-date (MTD) nowcasts for US CPI, Payroll Growth, and Unemployment using high-frequency proxies.
* 📊 **Market Indices:** Daily Market Conditions (stress) and weekly Labour Pulse indices, benchmarked against the Chicago Fed NFCI and S&P 500.
* 🛰️ **Alternative Data Engine:** Rigorous, publication-aware testing of logistics (PortWatch), positioning (CFTC), and fundamentals (EIA/JODI).
* 🛡️ **Automated Look-Ahead Guards:** A dedicated `pytest` suite that rebuilds the entire pipeline as of historical dates to mathematically guarantee zero data leakage.

---

## 🔬 Approach & Methodology

### 1. Publication-Aware Point-in-Time (PIT) Engine
Raw data is transformed into `(observation_date, publication_date, value)` tuples. The panel is built on the trading calendar, taking only the latest value whose `publication_date <= current_date`. 

### 2. Baselines First
Before testing complex models, the framework establishes naive baselines (e.g., Random Walk for daily returns, Seasonal Mean for monthly macro data). A model is only considered useful if it statistically beats the baseline.

### 3. Walk-Forward Validation
Models are refitted monthly. Training labels are strictly embargoed by the forecast horizon (e.g., a 5-day ahead prediction only trains on data where the 5-day return has already realized).

### 4. Multiple-Testing Corrections
Alternative data signals are evaluated using Rank IC and HAC t-statistics, with **Bonferroni corrections** applied to prevent false discoveries from data snooping.

---

## ⚖️ Assumptions

1. **Deterministic Release Lags:** The framework assumes specific, deterministic publication rules (e.g., EIA weekly data released Wed 10:30 ET is usable for the Thursday close).
2. **A Priori Economic Sign:** Component signs (e.g., strong USD is bearish for oil) are fixed *before* looking at backtest results to avoid data snooping.
3. **Trailing Stationarity:** A trailing 3-year window is sufficient to normalize cross-sectional signals into comparable z-scores.
4. **Free Data Proxies:** Public, keyless data sources are assumed to be accurate and timely enough for prototype signal extraction.

---

## ⚠️ Limitations

* **Statistical Insignificance:** The 1-week crude oil view frequently yields a Diebold-Mariano t-stat $< 2$, meaning it is not statistically distinguishable from a simple "no-change" random walk over the full out-of-sample period.
* **Macro Revision Bias:** Macro backtests currently use the *latest vintage* of revised data (e.g., final payroll numbers). This flatters the backtest; true production models require first-release vintages.
* **No Futures Curve Modeling:** The crude model predicts spot WTI returns. It ignores the futures curve structure (contango/backwardation), roll yield, and the cost of carry.
* **Geopolitical Shocks:** Geopolitical events are treated as visual context markers, not quantitatively modeled structural breaks.
* **Live-Only News:** The GDELT news sentiment indicator is restricted to the last ~3 months due to API limitations and cannot be formally backtested.

---

## 🚀 Path to Production

Before deploying this framework in a live trading or institutional environment, the following upgrades are required:

1. **Point-in-Time Revision Databases:** Integrate a paid provider (e.g., Bloomberg, Refinitiv, or FRED ALFRED) to train on *first-release* vintages of macro data.
2. **Transaction Costs & Roll Yield:** Build an execution simulator that translates spot predictions into futures roll strategies, accounting for bid-ask spreads and slippage.
3. **Robust Orchestration:** Migrate from local scripts to a production workflow orchestrator (e.g., Airflow, Prefect) with automated data-quality alerting.
4. **Regime-Switching Models:** Upgrade from linear ridge regression to Hidden Markov Models (HMM) to better capture non-linear volatility regimes.

---


## ✨ Key Features

- 🕒 **Strictly Publication-Aware (Point-in-Time):** Every data point is joined using exact publication lag rules (e.g., EIA weekly data is only usable on Thursday; Fed H.10 FX on Tuesday). *No future data ever leaks into past predictions.*
- 📉 **Baselines First:** Complex models are only evaluated against simple, transparent baselines (seasonal, trailing 12m, random walk) using Diebold-Mariano t-tests.
- 🔄 **Walk-Forward Validation:** Monthly refits with embargoed labels to simulate real-time trading/research conditions.
- 📝 **Auto-Generated Commentary:** An LLM-style rule engine (`commentary.py`) reads the live metrics and writes a human-readable analysis for every single chart, updating dynamically with every run.
- 📺 **Interactive Dashboards:** Beautiful, standalone Plotly HTML dashboards for Crude, Macro, and Combined views.
- 💸 **100% Free Data Sources:** No paid APIs required. Uses FRED (keyless), GitHub datasets, IMF PortWatch, CFTC, JODI, and GDELT.

---

## 🏗️ Project Architecture

The core logic is encapsulated in the `nowcastlab` Python package:

| Module | Description |
| :--- | :--- |
| `config.py` | All tunable methodology choices: release rules, component signs, weights, and windows. |
| `data.py` | Fetches raw data, applies quality checks, and builds the **Point-in-Time (PIT)** daily panel. |
| `features.py` | Computes trailing z-scores and the **Crude Pressure Index** (Trend, Brent Premium, USD, Volatility). |
| `model.py` | Walk-forward engine (Random Walk, Ridge, OLS) with strict monthly refits and label embargoes. |
| `altdata.py` | Fetches alternative data (Tankers, EIA, CFTC, JODI, GDELT) and runs multi-horizon signal tests. |
| `macro.py` | Nowcasts for US CPI, Payrolls, Unemployment, plus Labour Pulse and Market Conditions indices. |
| `combined.py` | Multi-panel timeline, scoreboard, and correlation map merging crude, alt-data, and macro. |
| `dashboard.py` | Plotly-based interactive HTML dashboard generator. |
| `commentary.py` | Generates dynamic, data-driven Markdown/text analysis for every chart and metric. |
| `checks.py` | **Look-ahead guard:** Automatically rebuilds the pipeline as of past dates to prove no future leakage. |

---

## 📦 Data Sources (No API Keys Required)

| Asset Class | Source | Frequency | Release Rule Applied |
| :--- | :--- | :--- | :--- |
| **WTI / Brent** | GitHub `datasets/oil-prices` | Daily | Same-day close |
| **FX (USD Basket)** | GitHub `datasets/exchange-rates` | Weekly | Tuesday close (H.10 lag) |
| **Inflation (CPI)** | GitHub `datasets/cpi-us` | Monthly | Month-end + 17 days |
| **Macro (FRED)** | FRED Keyless CSV | Daily/Weekly | Varies (e.g., Claims + 5 days) |
| **Tanker Traffic** | IMF PortWatch | Daily | Obs + 10 days |
| **Positioning** | CFTC Commitments of Traders | Weekly | Tuesday data -> Monday close |
| **Global Balances** | JODI Oil World | Monthly | Month-end + 45 days |
| **News Sentiment** | GDELT Project | Daily | Live indicator only (not backtested) |

---

## 🚀 Installation & Usage

### 1. Kaggle (Recommended)
This notebook is optimized for Kaggle. 
1. Open the notebook in Kaggle.
2. Ensure **Internet** is turned ON in the right sidebar (Settings -> Internet).
3. Click **Run All**. The pipeline will automatically download data, run tests, and generate outputs.

### 2. Local Environment
```bash
# Clone the repository
git clone https://github.com/yourusername/nowcast-lab.git
cd nowcast-lab

# Install dependencies
pip install pandas numpy scipy plotly matplotlib kaleido

# Run the full crude oil pipeline
python run.py

# Run the alternative data experiment
python run_altdata.py

# Run the macro nowcasts
python run_macro.py
