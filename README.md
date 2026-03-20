# FinancialData-Multi-Agents
# 📈 Financial Data Multi-Agent System

### AI-Powered Portfolio Analysis for FAANG + NVIDIA Stocks

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.8+-blue?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/Jupyter-Notebook-orange?style=for-the-badge&logo=jupyter&logoColor=white" />
  <img src="https://img.shields.io/badge/Plotly-Interactive_Charts-3F4F75?style=for-the-badge&logo=plotly&logoColor=white" />
  <img src="https://img.shields.io/badge/OpenAI-API_Ready-412991?style=for-the-badge&logo=openai&logoColor=white" />
  <img src="https://img.shields.io/badge/Yahoo_Finance-Data_Source-720e9e?style=for-the-badge" />
  <img src="https://img.shields.io/badge/License-MIT-success?style=for-the-badge" />
</p>

<p align="center">
  <img src="assets/chart1_allocation.png" width="80%" alt="Portfolio Allocation Preview"/>
</p>

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Key Features](#-key-features)
- [System Architecture](#-system-architecture)
- [AI Agents](#-ai-agents)
- [Visualizations](#-visualizations)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Usage Guide](#-usage-guide)
- [Sample Output](#-sample-output)
- [Pipeline Flow](#-pipeline-flow)
- [Future Enhancements](#-future-enhancements)
- [Contributing](#-contributing)
- [License](#-license)
- [Author](#-author)

---

## 🔍 Project Overview

The **Financial Data Multi-Agent System** is a production-grade, end-to-end quantitative investment analysis pipeline built entirely in Python. It combines live financial data ingestion, mathematical portfolio optimization via Modern Portfolio Theory (MPT), AI-driven trading decision engines, and a comprehensive suite of **10 interactive visualizations** — all orchestrated through a clean **multi-agent architecture**.

The system targets six of the world's most-watched technology stocks:

> **AAPL &nbsp;·&nbsp; AMZN &nbsp;·&nbsp; GOOGL &nbsp;·&nbsp; META &nbsp;·&nbsp; NFLX &nbsp;·&nbsp; NVDA** &nbsp;*(FAANG + NVIDIA)*

Data is sourced live from **Yahoo Finance** over a rolling **5-year daily window**, enabling both long-term trend analysis and short-term tactical trade planning. All output charts are fully interactive (Plotly) and can be exported as standalone HTML files.

---

## ✨ Key Features

- **Multi-Agent Architecture** — Three specialized AI agents with clearly separated responsibilities: portfolio optimizer, trading signal engine, and report compiler — all independently swappable
- **Markowitz MPT Optimization** — Mathematically optimal portfolio weights using convex optimization (`cvxpy`) with strict diversification bounds (5%–40% per stock)
- **Three-Tier Decision Engine** — Progressively evolves from simple rule-based signals → historical entry dates → full 180-day holding period simulation
- **Optimal Entry & Hold Simulation** — Identifies the historically best entry date per stock, then simulates every possible holding window (1–180 days) to find the maximum cumulative return period
- **Monthly Seasonality Analysis** — Average return patterns computed across 5 years per ticker to identify the strongest and weakest calendar months
- **10 Interactive Visualizations** — Covering portfolio allocation, risk-return analysis, time series decomposition (STL), correlation networks, drawdown analytics, and seasonality distributions
- **Secure API Key Handling** — Python `getpass()` ensures the OpenAI API key is never hardcoded or visible in notebook output
- **HTML Chart Exports** — Key dashboards exported as shareable `.html` files for presentations and stakeholder reporting

---

## 🏗️ System Architecture

The entire pipeline is structured around three specialized agents with clean input/output contracts, making each one independently testable and replaceable:

```
┌──────────────────────────────────────────────────────────────────────┐
│                        DATA INGESTION LAYER                          │
│                   get_yahoo_data()  ·  yfinance API                  │
│     5Y daily OHLCV  ·  Returns  ·  Volatility  ·  Seasonality       │
└───────────────────────────────┬──────────────────────────────────────┘
                                 │
               ┌─────────────────┼──────────────────┐
               ▼                 ▼                  ▼
   ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────┐
   │   MPT AGENT      │  │ DECISION ENGINE  │  │  DASHBOARD AGENT     │
   │                  │  │                  │  │                      │
   │ Markowitz Optim. │  │  V1: Rule-Based  │  │  Report Compiler     │
   │ cvxpy + SCS      │  │  V2: + Dates     │  │  Risk Summarizer     │
   │ 5%–40% bounds    │  │  V3: + Hold Opt. │  │  Quarterly Tactics   │
   └────────┬─────────┘  └────────┬─────────┘  └──────────┬───────────┘
            │                     │                        │
            └─────────────────────┼────────────────────────┘
                                  ▼
                  ┌───────────────────────────────┐
                  │   10 PLOTLY VISUALIZATIONS    │
                  │   + OpenAI LLM Narrative Layer │
                  │   + HTML Export Files          │
                  └───────────────────────────────┘
```

---

## 🤖 AI Agents

### Agent 1 — MPT Portfolio Optimizer · `mpt_agent_diversified()`

The MPT Agent is the mathematical backbone of the system. It implements **Markowitz mean-variance optimization** via the `cvxpy` convex optimization library. For each run, it computes a regularized covariance matrix from recent returns and solves the following optimization problem:

```
Maximize:   μᵀw  −  γ · wᵀΣw

Where:
  μ   =  vector of mean returns per ticker
  Σ   =  return covariance matrix  (+1e-6 regularization diagonal)
  γ   =  risk-aversion parameter   (default: 0.5)
  w   =  portfolio weight vector   (the decision variable)

Constraints:
  Σ wᵢ = 1.0          (fully invested)
  wᵢ ≥ 0.05           (minimum 5% per stock)
  wᵢ ≤ 0.40           (maximum 40% per stock)
  Solver: SCS via ecos
```

**Output:** A dictionary of optimized allocation weights, e.g.:
```python
{"AAPL": 0.05, "AMZN": 0.40, "GOOGL": 0.05, "META": 0.05, "NFLX": 0.40, "NVDA": 0.05}
```

---

### Agent 2 — Decision Engine · Three Progressive Versions

The Decision Engine generates a **Buy / Sell / Hold** signal with a confidence score and capital allocation recommendation for each stock. It evolved across three versions of increasing analytical depth:

#### V1 — `decision_engine_stable()` · Rule-Based Signals
Evaluates each stock on trailing 1-year return and annualized volatility:

| Condition | Signal | Confidence Formula |
|---|---|---|
| 1Y Return > 10% AND Vol < 35% | **Buy** | `min(0.9, return × 2)` |
| 1Y Return < -5% | **Sell** | `min(0.8, abs(return) × 2)` |
| Otherwise | **Hold** | `0.5` (flat) |

Confidence scores are proportionally scaled across all tickers to produce the suggested capital allocation.

#### V2 — `decision_engine_dates()` · + Historical Entry Dates
Adds a time-aware entry strategy on top of V1. Scans the recent price window to identify the single day with the highest historical return as the optimal buy entry point, and pairs it with a fixed **30-day holding period** — transforming directional signals into concrete dated trade plans.

#### V3 — `decision_engine_optimal_hold()` · + Holding Period Simulation
The most sophisticated version. After identifying the best entry date (same as V2), it **simulates every holding period from 1 to 180 days** and selects whichever duration produced the highest cumulative return. Each stock receives a unique, data-driven holding window — making this the closest to a quantitative trading strategy engine.

```python
# Core holding period optimization
for h in range(1, min(max_hold_days, len(df) - best_idx)):
    cum_return = df[price_col].iloc[best_idx:best_idx+h].pct_change().add(1).prod() - 1
    if cum_return > max_cum_return:
        max_cum_return = cum_return
        optimal_hold  = h
```

---

### Agent 3 — Dashboard Agent · `dashboard_agent_enhanced()` / `dashboard_agent_dates()`

The orchestration and reporting layer. Consumes outputs from both agents above and synthesizes them into a structured investment report with four sections:

| Section | Content |
|---|---|
| **Portfolio Allocation** | MPT-optimized weights per ticker |
| **AI Decision Recommendations** | Action · Confidence · AI vs MPT allocation · Volatility · Best entry month |
| **Risk Summary** | Flags tickers where annualized volatility exceeds 25% |
| **Quarterly Tactical Recommendations** | One-liner per stock: action + sizing + confidence + seasonality context |

---

### Agent 4 — OpenAI LLM Agent *(Wired In — Ready to Activate)*

The project initializes an OpenAI client at the top of the notebook using Python's `getpass()` for secure, non-hardcoded key entry. This agent is architecturally designed to receive the Dashboard Agent's markdown report and generate polished, natural-language investment commentary — completing the system's transformation into a fully automated AI research analyst.

```python
from getpass import getpass
os.environ["OPENAI_API_KEY"] = getpass("Enter your OpenAI API key: ")
client = OpenAI()
# → Pass dashboard report to GPT-4 for narrative generation
```

---

## 📊 Visualizations

The project delivers **10 fully interactive Plotly charts** covering every dimension of portfolio analysis. Below are the actual output visualizations:

---

### Chart 1 — Portfolio Allocation: AI Agent vs MPT Optimizer

A grouped bar chart comparing what the AI Decision Engine recommends versus what the mathematical MPT optimizer calculates as the optimal weight for each ticker. Immediately reveals where the two systems agree and where they diverge.

<p align="center">
  <img src="assets/chart1_allocation.png" width="90%" alt="Portfolio Allocation Chart"/>
</p>

---

### Chart 2 — Confidence vs Volatility

A bubble scatter where X = annualized volatility, Y = AI confidence score, and bubble size = AI-suggested allocation. Color-coded by Buy/Hold/Sell action. Shows whether the system is assigning higher confidence to lower-risk stocks as expected.

<p align="center">
  <img src="assets/chart2_confidence_volatility.png" width="88%" alt="Confidence vs Volatility Scatter"/>
</p>

---

### Chart 3 — Best Entry Dates & Optimal Holding Period Timeline

A Gantt-style horizontal bar chart showing each stock's historically optimal entry date and holding window. Makes the Decision Engine V3's output immediately actionable — a concrete trade calendar with personalized hold durations per ticker.

<p align="center">
  <img src="assets/chart3_timeline.png" width="88%" alt="Holding Period Gantt Timeline"/>
</p>

---

### Chart 4 — Monthly Seasonality Heatmap

A 6 × 12 heatmap (tickers × calendar months) using a RdYlGn colorscale centered at zero. Each cell shows the average daily return for that stock in that month over 5 years. Instantly reveals the strongest and weakest seasonal entry windows per ticker, directly supporting the Decision Engine's seasonality-based logic.

<p align="center">
  <img src="assets/chart4_heatmap.png" width="95%" alt="Monthly Seasonality Heatmap"/>
</p>

---

### Chart 5 — Multi-Metric Radar Chart

A spider/polar chart that normalizes all six stocks across four dimensions — 1-Year Return, 5-Year Return, Low Volatility (inverted), and AI Confidence — onto a 0–100 scale for fair comparison. Stocks with the largest filled polygon area are the strongest all-around performers across every dimension simultaneously.

<p align="center">
  <img src="assets/chart5_radar.png" width="72%" alt="Multi-Metric Radar Chart"/>
</p>

---

### Chart 6 — Risk-Return Bubble Chart with MPT Weights

The classic quantitative finance risk-return plane. X = annualized volatility, Y = 1-year return, bubble size = MPT portfolio weight. Reference lines divide the chart into four quadrants. Stocks in the **top-left** (high return, low risk, large bubble) represent the ideal intersection of all three agents' signals.

<p align="center">
  <img src="assets/chart6_bubble.png" width="90%" alt="Risk-Return Bubble Chart"/>
</p>

---

### Chart 7 — Rolling 90-Day Annualized Volatility (5-Year Timeline)

Tracks each stock's risk profile evolution over the full 5-year window using a 90-day rolling standard deviation (annualized via `× √252`). A red dotted line marks the 35% threshold — exactly the volatility ceiling used in the Decision Engine's Buy condition. Includes an interactive range slider and 6M/1Y/3Y/5Y zoom buttons.

<p align="center">
  <img src="assets/chart7_rolling_vol.png" width="95%" alt="Rolling 90-Day Volatility"/>
</p>

---

### Chart 8 — Underwater Drawdown Chart with Recovery Bands

A six-panel stacked chart tracking each stock's percentage decline from its rolling all-time high. Features filled area shading, a red dotted -20% bear market threshold line, red-shaded recovery bands during periods when drawdown exceeded -20%, and an annotated arrow pointing to the maximum drawdown point. Shared X-axis enables cross-stock timing comparison.

<p align="center">
  <img src="assets/chart8_drawdown.png" width="90%" alt="Underwater Drawdown Chart"/>
</p>

---

### Chart 9 — Stock Correlation Network Graph

A force-directed network graph where stocks are nodes and edges connect pairs with return correlation above the 0.55 threshold. Edge width and opacity scale with correlation strength. Node size reflects the number of strong connections (degree). Stocks with few or no edges provide the most genuine diversification — a direct visual diagnostic for the MPT Agent's covariance logic.

<p align="center">
  <img src="assets/chart9_network.png" width="78%" alt="Stock Correlation Network"/>
</p>

---

### Chart 10 — STL Decomposition: Trend / Seasonality / Residual

Uses `statsmodels` STL (Seasonal-Trend decomposition via LOESS) on weekly-resampled price data to decompose each stock into three components across a 6 × 3 subplot grid. The **Trend** column shows long-term price momentum, the **Seasonality** column (period=52 weeks) validates the monthly patterns driving the Decision Engine, and the **Residual** bar chart highlights structural anomalies — earnings shocks, macro events — as unexplained noise.

<p align="center">
  <img src="assets/chart10_stl.png" width="95%" alt="STL Decomposition Grid"/>
</p>

---

## 🛠️ Tech Stack

| Library | Version | Purpose |
|---|---|---|
| `yfinance` | Latest | Real-time OHLCV price data from Yahoo Finance |
| `pandas` | ≥ 1.5 | Data manipulation and time series operations |
| `numpy` | ≥ 1.23 | Numerical computing and matrix operations |
| `cvxpy` | ≥ 1.3 | Convex optimization for Markowitz MPT |
| `ecos` | Latest | Solver backend for cvxpy (SCS mode) |
| `plotly` | ≥ 5.0 | All 10 interactive visualizations |
| `statsmodels` | ≥ 0.14 | STL time series decomposition |
| `networkx` | ≥ 3.0 | Correlation network graph construction |
| `openai` | ≥ 1.0 | LLM client for narrative commentary |
| `nest_asyncio` | Latest | Async compatibility within Jupyter |
| `getpass` | stdlib | Secure API key entry (no hardcoding) |

---

## 📁 Project Structure

```
financial-data-multi-agents/
│
├── FinancialData_Multi_Agents.ipynb     # Main Jupyter notebook (33 cells)
│
├── assets/                              # Visualization output images
│   ├── chart1_allocation.png
│   ├── chart2_confidence_volatility.png
│   ├── chart3_timeline.png
│   ├── chart4_heatmap.png
│   ├── chart5_radar.png
│   ├── chart6_bubble.png
│   ├── chart7_rolling_vol.png
│   ├── chart8_drawdown.png
│   ├── chart9_network.png
│   └── chart10_stl.png
│
├── outputs/                             # Auto-generated HTML chart exports
│   ├── portfolio_allocation.html
│   ├── confidence_vs_volatility.html
│   └── holding_period_timeline.html
│
└── README.md                            # This file
```

---

## 🚀 Getting Started

### Prerequisites

- Python **3.8** or higher
- Jupyter Notebook or JupyterLab
- An OpenAI API key *(optional — only needed for the LLM narrative layer)*
- Active internet connection *(for live Yahoo Finance data fetch)*

### Installation

**1. Clone the repository**
```bash
git clone https://github.com/dhrumil231/financial-data-multi-agents.git
cd financial-data-multi-agents
```

**2. Create and activate a virtual environment** *(recommended)*
```bash
python -m venv venv
source venv/bin/activate          # macOS / Linux
venv\Scripts\activate             # Windows
```

**3. Install all dependencies**
```bash
pip install openai yfinance pandas numpy cvxpy ecos plotly statsmodels networkx nest_asyncio
```

**4. Launch Jupyter**
```bash
jupyter notebook FinancialData_Multi_Agents.ipynb
```

---

## ▶️ Usage Guide

### Step 1 — Secure API Key Entry *(optional)*
When Cell 4 runs, enter your OpenAI key at the prompt — it is never printed or stored:
```
Enter your OpenAI API key: ████████████████
```

### Step 2 — Fetch Live Market Data
```python
data = get_yahoo_data()
# Fetches 5Y daily prices for AAPL, AMZN, GOOGL, META, NFLX, NVDA
```

### Step 3 — Run the Three Agents
```python
# Agent 1: MPT Portfolio Optimization
mpt_weights = mpt_agent_diversified(data)

# Agent 2: Decision Engine — choose one version
decisions = decision_engine_stable(data, mpt_weights)          # V1: rule-based
decisions = decision_engine_dates(data, mpt_weights)           # V2: + entry dates
decisions = decision_engine_optimal_hold(data, mpt_weights)    # V3: + hold simulation

# Agent 3: Dashboard Report
report = dashboard_agent_enhanced(data, mpt_weights, decisions)
print(report)
```

### Step 4 — Generate All Visualizations
Run cells 17 → 33 sequentially. All charts render inline in the notebook; the first three are also auto-exported as HTML:
```bash
portfolio_allocation.html
confidence_vs_volatility.html
holding_period_timeline.html
```

### Customization Parameters

| Parameter | Location | Default | Description |
|---|---|---|---|
| `TICKERS` | Cell 7 | FAANG+NVDA | Swap any valid Yahoo Finance ticker symbols |
| `period` | Cell 7 | `"5y"` | Data lookback window: `"1y"` / `"2y"` / `"5y"` / `"10y"` |
| `gamma` | MPT Agent | `0.5` | Risk-aversion coefficient — higher = more conservative |
| `min_weight` | MPT Agent | `0.05` | Minimum allocation per stock (5%) |
| `max_weight` | MPT Agent | `0.40` | Maximum allocation per stock (40%) |
| `max_hold_days` | V3 Engine | `180` | Max days simulated in holding period optimization |
| `CORR_THRESHOLD` | Cell 29 | `0.5` | Minimum correlation to draw an edge in the network graph |

---

## 📋 Sample Output

### Decision Engine Report (V3)

```
# Portfolio Dashboard

## Suggested Portfolio Allocation (MPT-based, diversified)
- AAPL:   5.0%
- AMZN:  40.0%
- GOOGL:  5.0%
- META:   5.0%
- NFLX:  40.0%
- NVDA:   5.0%

## AI Decision Engine Recommendations

### AAPL
- Action: Buy          | Confidence: 32.6%
- AI Allocation: 10%   | MPT Allocation: 5%
- Volatility: 27.48%   | Best entry: Feb 24 | Hold: 1 day

### GOOGL
- Action: Buy          | Confidence: 90.0%
- AI Allocation: 28%   | MPT Allocation: 5%
- Volatility: 30.67%   | Best entry: Mar 09 | Hold: 6 days

### NVDA
- Action: Hold         | Confidence: 50.0%
- AI Allocation: 16%   | MPT Allocation: 5%
- Volatility: 51.66%   | Best entry: Mar 02 | Hold: 7 days

## Risk Summary
High volatility stocks (>25% annualized): AMZN, META, NFLX, NVDA
```

### Full Results Table

| Ticker | Action | Confidence | AI Alloc | MPT Alloc | Volatility | Best Entry | Hold |
|--------|--------|-----------|----------|-----------|------------|------------|------|
| AAPL   | Buy    | 32.6%     | 10%      | 5%        | 27.5%      | Feb 24     | 1d   |
| AMZN   | Hold   | 50.0%     | 16%      | 40%       | 35.2%      | Mar 04     | 1d   |
| GOOGL  | Buy    | 90.0%     | 28%      | 5%        | 30.7%      | Mar 09     | 6d   |
| META   | Hold   | 50.0%     | 16%      | 5%        | 43.5%      | Mar 16     | 1d   |
| NFLX   | Hold   | 50.0%     | 16%      | 40%       | 43.0%      | Feb 27     | 4d   |
| NVDA   | Hold   | 50.0%     | 16%      | 5%        | 51.7%      | Mar 02     | 7d   |

---

## 🔄 Pipeline Flow

```
Yahoo Finance API  →  5Y Daily OHLCV Data
          │
          ▼
  get_yahoo_data()
  ├── recent_prices      (last 20 closing prices)
  ├── monthly_seasonality (avg return by month, 1–12)
  ├── one_year_return    (trailing 252-day return)
  ├── five_year_return   (full period total return)
  └── volatility         (annualized std dev: σ × √252)
          │
          ├──────────────────────────────────────┐
          ▼                                      ▼
  mpt_agent_diversified()         decision_engine_optimal_hold()
  (Markowitz MPT weights)         (Buy/Sell/Hold + entry date +
                                   optimal holding period)
          │                                      │
          └──────────────────┬───────────────────┘
                             ▼
               dashboard_agent_enhanced()
               (Structured markdown report)
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
         10 Plotly      HTML Exports    OpenAI LLM
         Charts         (3 files)       (optional)
```

---

## 🔮 Future Enhancements

- **Activate LLM Narrative Layer** — Connect the OpenAI client to auto-generate professional research commentary from the Dashboard Agent's report output
- **Backtesting Engine** — Validate all three Decision Engine versions against historical prices to measure Sharpe ratio, CAGR, win rate, and maximum drawdown
- **Streamlit Web App** — Convert the notebook into a live web dashboard with real-time ticker selection, date range sliders, and auto-refresh on market open
- **Live Trade Alerts** — Schedule daily pipeline runs with email or Slack push notifications when Buy signals are triggered
- **Sentiment Integration** — Augment Decision Engine signals with real-time news sentiment scores via NewsAPI + FinBERT for a hybrid quant + NLP signal
- **Expanded Universe** — Extend beyond FAANG+NVDA to sector ETFs, international ADRs, or a fully customizable watchlist
- **Portfolio Drift Tracker** — Add a rebalancing module that monitors live weight drift from MPT targets and auto-generates rebalance trade recommendations

---

## 🤝 Contributing

Contributions are welcome! Here's how to get involved:

1. **Fork** the repository
2. **Create** your feature branch
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Commit** your changes with a clear message
   ```bash
   git commit -m "Add: description of what you built"
   ```
4. **Push** to your branch
   ```bash
   git push origin feature/your-feature-name
   ```
5. **Open a Pull Request** — describe what changed and why

Please follow existing code style conventions and add inline comments for any new agent logic or visualization cells.

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for full details.

---

## 👤 Author

**Dhrumil Shah and Pritika Pravin**

MS Engineering Management · Syracuse University (Dec 2025)
Senior Business Analyst · Angel One | Fintech & Capital Markets


University of Washington | Bachelor of Arts - BA, Geography with Data Science
Backend Software Engineer | Agentic AI

[![GitHub](https://img.shields.io/badge/GitHub-dhrumil231-181717?style=flat&logo=github&logoColor=white)](https://github.com/dhrumil231)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/dhrumil-shah-101853215/)

---

> ⚠️ **Disclaimer:** This project is built for educational and portfolio demonstration purposes only. Nothing in this repository constitutes financial advice or investment recommendations. Always perform your own due diligence before making any investment decisions.
