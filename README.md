# Quantitative Easing & Asset Valuation: A VAR/VECM Analysis

> Empirical analysis of how U.S. Federal Reserve balance sheet expansion (Quantitative Easing) affects equity market valuations through interest rate channels — implemented with rigorous econometric methodology in Python.

[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Methodology](https://img.shields.io/badge/methodology-VECM-green.svg)]()
[![Sample](https://img.shields.io/badge/sample-2007Q2--2021Q4-orange.svg)]()

---

## Table of Contents

- [Overview](#overview)
- [Research Question](#research-question)
- [Key Findings](#key-findings)
- [Data](#data)
- [Methodology](#methodology)
- [Repository Structure](#repository-structure)
- [Installation & Usage](#installation--usage)
- [Results Summary](#results-summary)
- [Limitations](#limitations)
- [References](#references)
- [Author](#author)

---

## Overview

This project investigates how Quantitative Easing (QE) — the unconventional monetary policy of large-scale asset purchases by the Federal Reserve — has affected U.S. equity market valuations over the period 2007 Q2 – 2021 Q4. Using a **Vector Error Correction Model (VECM)**, it quantifies both the short-run dynamics and long-run equilibrium relationships between the Fed balance sheet, 10-year Treasury yields, and the S&P 500 P/E ratio.

The methodology follows a strict **statistical decision tree**: I begin with a standard VAR model, then use formal econometric tests to determine whether VAR is appropriate, ultimately demonstrating that the data require a VECM specification due to cointegration.

---

## Research Question

> **To what extent does Federal Reserve balance sheet expansion (QE) explain variation in U.S. equity valuations, and through which transmission channels?**

Sub-questions:
1. Is QE a statistically significant driver of S&P 500 P/E movements?
2. Does QE operate primarily through direct effects, or indirectly via the yield channel?
3. Are these relationships stable across the four QE episodes (QE1-QE4)?

---

## Key Findings

| Finding | Value | Source |
|---|---|---|
| Cointegrating vectors | r = 1 | Johansen trace test, p < 0.05 |
| QE contribution to S&P P/E variance (8Q horizon) | **10.6%** | VECM FEVD |
| UST10 contribution to S&P P/E variance (8Q) | 5.9% | VECM FEVD |
| Speed of P/E adjustment to disequilibrium | α = −1.87 | VECM loading coefficient |
| Long-run equilibrium | Log(BS) + 0.685·UST10 − 0.001·P/E = 0 | Cointegrating vector β |

### Economic Interpretation

The Fed balance sheet (QE) and equity valuations share a **stable long-run relationship**. When the system deviates from this equilibrium, the P/E ratio adjusts most aggressively (~187% of the deviation per quarter), while the balance sheet itself is weakly exogenous — consistent with QE being a policy-driven, not market-driven, variable. The dominant transmission channel is **yield compression**: QE lowers the 10-year Treasury yield, which mechanically lifts the discount factor and raises equity valuations.

---

## Data

### Source Variables

| Variable | Symbol | Source | Frequency |
|---|---|---|---|
| Fed Total Assets (Balance Sheet) | `WALCL` | FRED | Quarterly |
| 10-Year U.S. Treasury Yield | `GS10` | FRED | Quarterly |
| S&P 500 Trailing P/E Ratio | — | Multpl.com | Quarterly |

### Sample

- **Period:** Q2 2007 – Q4 2021
- **Observations:** 59 quarters
- **Coverage:** All four QE rounds (QE1 2008, QE2 2010, QE3 2012, QE4 2020)

### Data File

```
data/
└── US_Fed_New_1_2.csv
```

The CSV contains: `Date`, `Balance sheet` (millions USD), `UST 10` (percent), `SP PE`.

---

## Methodology

The analysis follows a 13-phase econometric pipeline implementing best-practice methodology for non-stationary multivariate time series.

### Decision Tree

```
                   ┌─────────────────────┐
                   │  Data Validation    │
                   │  (T=59 quarters)    │
                   └──────────┬──────────┘
                              │
                   ┌──────────▼──────────┐
                   │  Stationarity Tests │
                   │  ADF + KPSS + PP    │
                   └──────────┬──────────┘
                              │
                       All series I(1)?
                              │
                           ┌──┴──┐
                          No     Yes
                           │      │
                  VAR in   │      │  Johansen Cointegration
                  levels   │      │  Test (find rank r)
                           │      │
                           │      ├── r=0 → VAR in differences
                           │      │
                           │      └── r≥1 → VECM (this project)
                           │             │
                           │      ┌──────▼──────┐
                           │      │ Estimate    │
                           │      │ VECM(p-1)   │
                           │      └──────┬──────┘
                           │             │
                           │      ┌──────▼──────┐
                           │      │ Diagnostics │
                           │      │ + IRF/FEVD  │
                           │      └──────┬──────┘
                           │             │
                           │      ┌──────▼──────┐
                           │      │ Robustness  │
                           │      │ Checks      │
                           │      └─────────────┘
```

### Statistical Tests Applied

| Phase | Test | Purpose |
|---|---|---|
| 1 | Data validation checks | Verify sample integrity |
| 3 | ADF, KPSS, Phillips-Perron | Establish I(d) order of each series |
| 5 | Johansen trace & max-eigenvalue | Detect cointegrating rank |
| 6 | AIC, BIC, HQIC | Select optimal lag order |
| 8 | Portmanteau (Ljung-Box) | Detect residual autocorrelation |
| 8 | Jarque-Bera | Test residual normality |
| 8 | Engle ARCH-LM | Detect conditional heteroscedasticity |
| 9 | Eigenvalues of companion matrix | Assess model stability |
| 12 | Sub-sample re-estimation | Robustness to time period |
| 12 | Alternative Cholesky ordering | Sensitivity to identification |

### Why VECM and Not VAR?

The original plan was to use a VAR model. However:

1. **All variables are I(1)** (confirmed by three stationarity tests) — VAR in levels would produce spurious regression
2. **Johansen test finds r = 1 cointegrating relationship** — VAR in first differences omits the long-run equilibrium
3. **VAR-in-differences Portmanteau test fails** (p = 0.04) — direct evidence of misspecification
4. **VECM Portmanteau test passes** (p = 0.35) — correctly specified model

The VECM is the statistically correct framework given the data properties.

### VECM Specification

$$
\Delta Y_t = \alpha \beta' Y_{t-1} + \Gamma_1 \Delta Y_{t-1} + \varepsilon_t
$$

where:
- $Y_t = [\log(BS_t), UST10_t, P/E_t]'$
- $\beta$ = cointegrating vector (long-run equilibrium)
- $\alpha$ = adjustment speeds (how each variable corrects toward equilibrium)
- $\Gamma_1$ = short-run dynamics matrix
- $\varepsilon_t \sim N(0, \Sigma)$

### Identification

Structural shocks are identified via **Cholesky decomposition** with ordering:

$$
\text{Balance Sheet} \rightarrow \text{UST 10Y} \rightarrow \text{S\&P P/E}
$$

This ordering reflects the economic causal chain: QE is policy-driven (most exogenous), yields respond to monetary policy, and equity valuations respond to both. Robustness is verified by re-estimating with an alternative ordering.

---

## Repository Structure

```
qe-asset-valuation-vecm/
│
├── README.md                          # This file
├── requirements.txt                   # Python dependencies
├── LICENSE                            # MIT license
│
├── data/
│   └── US_Fed_New_1_2.csv             # Source data
│
├── src/
│   └── best_practice_pipeline.py      # Main analysis script
│
├── notebooks/                         # (Optional) exploratory notebooks
│   └── exploration.ipynb
│
├── figures/                           # Output charts
│   ├── bp_fig1_decision_tree.png      # Methodology overview
│   ├── bp_fig2_series.png             # Raw series with QE periods
│   ├── bp_fig3_diagnostics.png        # Test results & stability
│   ├── bp_fig4_irf.png                # Impulse response functions
│   └── bp_fig5_fevd.png               # Variance decomposition
│
└── docs/
    ├── VAR_VECM_Step_by_Step_Guide.docx   # Detailed methodology guide
    └── methodology.md                      # Theory notes
```

---

## Installation & Usage

### Requirements

```bash
Python >= 3.9
numpy >= 1.20
pandas >= 1.3
scipy >= 1.7
matplotlib >= 3.4
```

### Setup

```bash
# Clone the repository
git clone https://github.com/<your-username>/qe-asset-valuation-vecm.git
cd qe-asset-valuation-vecm

# Create virtual environment
python -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Run the Analysis

```bash
python src/best_practice_pipeline.py
```

This will:
1. Load and validate the data
2. Run all three stationarity tests
3. Test for cointegration via Johansen
4. Estimate VAR(1) and run diagnostics
5. Estimate VECM(1) and run diagnostics
6. Compute IRFs with bootstrap confidence bands (500 draws)
7. Compute FEVD
8. Run robustness checks
9. Generate all figures in `figures/`

Expected runtime: ~30 seconds on a modern laptop.

---

## Results Summary

### Stationarity Test Results

| Series | ADF τ | KPSS | PP Z | Verdict |
|---|---|---|---|---|
| Log(BS) | −1.04 | 1.09 | −2.10 | I(1) |
| UST10 | −2.07 | 0.85 | −2.83 | I(1) |
| P/E | 0.99 | 0.91 | −3.31 | I(1) |
| Δ Log(BS) | −2.97* | 0.17 | −5.69** | I(0) |
| Δ UST10 | −4.15** | 0.12 | −6.18** | I(0) |
| Δ P/E | −3.96** | 0.09 | −5.53** | I(0) |

\* significant at 5%, ** significant at 1%

### Johansen Cointegration Test

| Null hypothesis | Trace statistic | 95% Critical Value | Decision |
|---|---|---|---|
| r = 0 | 38.59 | 29.80 | **Reject** |
| r ≤ 1 | 15.83 | 15.50 | **Reject** (marginal) |
| r ≤ 2 | 3.56 | 3.84 | Fail to reject |

Conclusion: **r = 1 cointegrating relationship**

### VECM Adjustment Speeds (α)

| Variable | α | Interpretation |
|---|---|---|
| Δ Log(BS) | −0.095 | Slow/weakly exogenous (policy-driven) |
| Δ UST10 | −0.394 | Moderate correction (~2-3 quarters) |
| Δ S&P P/E | −1.869 | Fast correction (over-shooting) |

### FEVD of S&P P/E (% of forecast variance)

| Horizon (Q) | BS Shock | UST10 Shock | Own |
|---|---|---|---|
| 1 | 1.6% | 3.6% | 94.8% |
| 4 | 12.3% | 2.9% | 84.8% |
| 8 | **10.6%** | 5.9% | 83.5% |
| 12 | 9.7% | 7.2% | 83.2% |
| 20 | 8.8% | 8.4% | 82.8% |

### Robustness Checks

- ✓ **Pre-COVID sub-sample:** cointegration still found (trace stat = 37.1 vs CV = 29.8)
- ✓ **Alternative Cholesky ordering:** BS contribution moves to 12.7% (consistent direction)
- ⚠ **COVID structural break:** Mean Δ Log(BS) increased from 3.1% to 9.3% per quarter

---

## Limitations

1. **Small sample size** (T = 59) limits statistical power and increases parameter uncertainty. Bootstrap confidence bands are correspondingly wide.

2. **COVID-19 represents a structural break** that the linear VECM does not explicitly model. A regime-switching or Markov-switching VAR could be a useful extension.

3. **The Cholesky identification scheme imposes a recursive causal ordering** that, while economically motivated, is not directly testable. Alternative identification strategies (sign restrictions, narrative restrictions, instrumental variables) would strengthen causal inference.

4. **The model captures domestic dynamics only** — global factors (ECB QE, BoJ QQE, dollar index) are omitted but likely relevant.

5. **P/E ratio is a backward-looking valuation metric** — forward-looking measures (Shiller CAPE, equity risk premium) might yield different results.

6. **Residuals exhibit non-normality and mild heteroscedasticity**, which is common for financial data but does not invalidate inference asymptotically. Bootstrap CIs are robust to this.

---

## Extensions & Future Work

- [ ] Markov-switching VECM to handle the COVID structural break formally
- [ ] Include credit spreads (BAA-AAA, high-yield) as additional channels
- [ ] Bayesian VAR with informative priors (Minnesota, sums-of-coefficients)
- [ ] Sign-restriction identification for structural shocks
- [ ] Time-varying parameter VAR (TVP-VAR) to capture evolving QE transmission
- [ ] Cross-country panel VAR (US, EU, UK, Japan)
- [ ] Higher-frequency analysis (monthly data, event studies around FOMC announcements)

---

## References

### Econometric Methodology
- Dickey, D. A., & Fuller, W. A. (1979). Distribution of the estimators for autoregressive time series with a unit root. *Journal of the American Statistical Association*, 74(366), 427-431.
- Johansen, S. (1991). Estimation and hypothesis testing of cointegration vectors in Gaussian vector autoregressive models. *Econometrica*, 59(6), 1551-1580.
- Kwiatkowski, D., Phillips, P. C. B., Schmidt, P., & Shin, Y. (1992). Testing the null hypothesis of stationarity against the alternative of a unit root. *Journal of Econometrics*, 54(1-3), 159-178.
- MacKinnon, J. G. (2010). Critical values for cointegration tests. Queen's Economics Department Working Paper.
- Lütkepohl, H. (2005). *New Introduction to Multiple Time Series Analysis*. Springer.

### QE Empirical Literature
- Gagnon, J., Raskin, M., Remache, J., & Sack, B. (2011). The financial market effects of the Federal Reserve's large-scale asset purchases. *International Journal of Central Banking*, 7(1), 3-43.
- Krishnamurthy, A., & Vissing-Jorgensen, A. (2011). The effects of Quantitative Easing on interest rates: Channels and implications for policy. *Brookings Papers on Economic Activity*, 2, 215-287.
- Joyce, M., Miles, D., Scott, A., & Vayanos, D. (2012). Quantitative Easing and unconventional monetary policy — an introduction. *The Economic Journal*, 122(564), F271-F288.

---

## Author

**[Your Name]**
[Your affiliation / programme]
[your.email@example.com]
[LinkedIn / Personal site]

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

Data sourced from FRED (Federal Reserve Economic Data) is in the public domain. S&P 500 P/E data from Multpl.com is used for non-commercial educational purposes.

---

## Acknowledgments

- Federal Reserve Bank of St. Louis (FRED) for balance sheet and Treasury yield data
- Robert Shiller's online dataset and Multpl.com for historical P/E series
- The Python scientific computing community (NumPy, SciPy, pandas, matplotlib)

---

*If you find this project useful, please consider giving it a ⭐ on GitHub!*