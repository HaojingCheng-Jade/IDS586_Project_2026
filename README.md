# IDS586_Project_2026

**Smart Beta Equity Portfolio Optimization**  
Final Project for IDS/Math 586 — Data Science and Decision Optimization in Banking and Finance  
Duke University · Spring 2026

---

## 📌 Project Overview

This project builds a complete **smart beta portfolio construction pipeline** that:

1. **Predicts** next-month stock excess returns using two estimation models (OLS and Random Forest)
2. **Selects** the top 6 highest-conviction stocks each month from a liquidity-filtered candidate pool
3. **Optimizes** portfolio weights via long-only Mean-Variance Optimization (MVO)
4. **Backtests** the full strategy out-of-sample under a rolling 36-month window framework

The central decision problem is: *At each monthly rebalancing date, how can we use historical Fama–French factor information to select stocks and allocate capital in a way that maximizes risk-adjusted return?*

---

## 📁 Repository Structure

```
IDS586_Project_2026/
│
├── Deliverables/
│   ├── Documentation.pdf      # Final written report
│   └── Presentation.pptx      # Final presentation slides
│
├── data_process/
│   └── Clean_2.ipynb          # Data cleaning, merging, feature engineering
│
├── Modeling/
│   ├── new_OLS.ipynb          # Rolling-window OLS regression
│   └── RandomForest.ipynb     # Rolling-window Random Forest regressor
│
├── MVO/
│   ├── new_MVO.ipynb          # Mean-variance optimization & backtesting
│   └── measurement.ipynb      # Portfolio performance metrics & evaluation
│
├── data/
│   ├── ols_top6_by_month_post2022.csv   # OLS monthly top-6 stock selections
│   ├── rf_top6_by_month_post2022.csv    # RF monthly top-6 stock selections
│   └── mvo_monthly_returns.csv          # Backtest monthly realized returns (2022–2025)
│
├── .gitignore
└── README.md
```

---

## 🗃️ Data Sources

| Dataset | Description |
|---|---|
| **CRSP Monthly Stock Data** | Stock-level panel: returns, prices, volume, shares outstanding (`PERMNO`, `MthCalDt`, `MthPrc`, `MthRet`, `MthVol`, `ShrOut`) |
| **Fama–French Five Factors** | Monthly macro/style factors: `Mkt-RF`, `SMB`, `HML`, `RMW`, `CMA`, `RF` |

> **Note:** Raw data files are not included in this repository due to size and licensing constraints. Access CRSP data via WRDS and Fama–French data via [Kenneth French's website](https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html).

---

## ⚙️ Methodology

### Phase 1 — Data Cleaning (`data_process/Clean_2.ipynb`)

- Convert `MthCalDt` to a `YYYYMM` integer key; align with FF5 factor dates
- Scale all factor columns and `RF` from percentage units (÷ 100)
- Clean negative CRSP prices: `MthPrc_Abs = abs(MthPrc)`
- Merge stock panel with factor data on the monthly date key
- Apply investability filters: price ≥ $5, volume > 0, ≥ 36 months of valid history
- Construct target label: `y_{i,t+1} = ExcessRet_{i,t+1} = MthRet_{i,t+1} − RF_{t+1}`
- Construct liquidity proxy: `DollarVol = MthPrc_Abs × MthVol`

### Phase 2 — Estimation Modeling

#### OLS (`Modeling/new_OLS.ipynb`)
Predicts next-month excess return via a five-factor linear regression:

$$\hat{y}_{i,t+1} = \hat{\alpha} + \hat{\beta}_1 (Mkt\text{-}RF)_t + \hat{\beta}_2 SMB_t + \hat{\beta}_3 HML_t + \hat{\beta}_4 RMW_t + \hat{\beta}_5 CMA_t$$

#### Random Forest (`Modeling/RandomForest.ipynb`)
Captures non-linear factor interactions via an ensemble of regression trees:

$$\hat{y}_{i,t+1} = \frac{1}{B} \sum_{b=1}^{B} T_b(X_t)$$

Both models use the same feature vector $X_t = [Mkt\text{-}RF,\ SMB,\ HML,\ RMW,\ CMA]_t$ and are re-estimated every month within a **36-month rolling window** to prevent look-ahead bias.

Output: `data/ols_top6_by_month_post2022.csv` (OLS) · `data/rf_top6_by_month_post2022.csv` (Random Forest)

### Phase 3 — Portfolio Optimization (`MVO/new_MVO.ipynb`) & Evaluation (`MVO/measurement.ipynb`)

At each rebalancing month $T$:

1. Rank all eligible stocks by `DollarVol`; retain the **top 100** as the candidate pool
2. Predict next-month excess returns for each candidate using both models
3. Select the **top 6 stocks** by predicted return
4. Estimate the 6×6 covariance matrix from the past 36 months of realized returns
5. Solve the **long-only Sharpe-maximizing MVO**:

$$\max_w \frac{w^\top \mu_T}{\sqrt{w^\top \Sigma_T w}} \quad \text{s.t.} \quad \sum w_i = 1,\ w_i \geq 0,\ w_i \leq 0.40$$

6. Record realized portfolio return in month $T+1$; compare against equal-weight benchmark

---

## 📊 Evaluation Metrics

**Prediction quality:**
- Root Mean Squared Error (RMSE)
- Mean Absolute Error (MAE)

**Portfolio performance:**
- Annualized Return
- Annualized Volatility
- Sharpe Ratio
- Maximum Drawdown

All metrics are computed **out-of-sample** on the post-2022 period.

---

## 🚀 How to Reproduce

### Requirements

```bash
pip install pandas numpy scikit-learn scipy matplotlib jupyter
```

### Run Order

```
1. data_process/Clean_2.ipynb       → generates cleaned panel dataset
2. Modeling/new_OLS.ipynb           → generates ols_top6_by_month_post2022.csv
3. Modeling/RandomForest.ipynb      → generates rf_top6_by_month_post2022.csv
4. MVO/new_MVO.ipynb                → runs MVO optimization, outputs mvo_monthly_returns.csv
5. MVO/measurement.ipynb            → computes performance metrics & generates evaluation plots
```

> Notebooks must be run in this order as each stage depends on outputs from the previous one.

---

## 👥 Team Members

<!-- Add team member names here -->
- Ginger Gao · Duke University
- Jade Cheng · Duke University
- Tea Tafaj · Duke University
- Wenyi Dai · Duke University

---

## 📎 References

- Fama, E. F., & French, K. R. (1993). Common risk factors in the returns on stocks and bonds. *Journal of Financial Economics*, 33(1), 3–56.
- Markowitz, H. (1952). Portfolio Selection. *The Journal of Finance*, 7(1), 77–91.
- CRSP Monthly Stock File — accessed via WRDS
- Kenneth French Data Library: https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html
