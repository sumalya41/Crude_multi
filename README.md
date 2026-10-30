# 📊 Nowcast Lab: Transparent Commodity & Macro Nowcasts

**Nowcast Lab** is a rigorous, transparent, and fully automated Python framework for building publication-aware nowcasts and indices for commodities (Crude Oil) and macroeconomic indicators (Inflation, Labour, Market Conditions). 

Built with a strict **"baselines first, walk-forward, zero look-ahead bias"** philosophy, it transforms free, public data into institutional-grade research dashboards and automated analytical commentary.

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
