# Euro Area Housing & Monetary Policy — Panel Data Analysis

Panel study of whether lower mortgage rates predict higher house price growth across nine euro-area countries, **2003–2023**.

**Research question:** Does a reduction in mortgage rates predict higher house price growth?

**Main finding (Phase A, Model 4A):** A one percentage-point increase in the mortgage rate is associated with **2.2 pp lower year-over-year house price growth** (p = 0.004), conditional on country and quarter fixed effects and macro controls.

---

## Sample

| | |
|---|---|
| **Countries** | Germany, France, Italy, Spain, Netherlands, Belgium, Austria, Portugal, Finland |
| **Period** | 2003Q2 – 2023Q4 |
| **Observations** | 747 country–quarter pairs (720 in YoY regressions) |

---

## Data sources

| Variable | Source | Series |
|----------|--------|--------|
| House Price Index | FRED (BIS) | `Q{CC}N628BIS` — Index 2010 = 100 |
| Mortgage Rate | ECB SDW | MIR — new business, dwellings, floating ≤ 10 yr |
| ECB Policy Rate | FRED | `ECBDFR` |
| GDP Growth | Eurostat | `namq_10_gdp` |
| Unemployment | FRED (OECD) | `LRHUTTTT{CC}Q156S` |
| Building Permits | Eurostat | `sts_cobp_q` |

> **Note:** Use `N628BIS` (index levels), not `N368BIS` (annual % growth). The latter are pre-computed growth rates and must not be passed through `pct_change()`.

---

## Setup

### 1. Clone and create a virtual environment

```bash
git clone https://github.com/bhavleenkaur-econ/Pannel-Regression.git
cd Pannel-Regression
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 2. FRED API key (free)

Register at [fred.stlouisfed.org/docs/api/api_key.html](https://fred.stlouisfed.org/docs/api/api_key.html), then:

```bash
export FRED_API_KEY='your_key_here'
```

Or set the key in **Section 0** of the notebook.

---

## Running the analysis

### Main notebook (data + Phase A regressions)

```bash
jupyter notebook euro_area_housing_panel.ipynb
```

Execute all cells, or run headless:

```bash
MPLBACKEND=Agg jupyter nbconvert \
  --to notebook \
  --execute euro_area_housing_panel.ipynb \
  --inplace \
  --ExecutePreprocessor.timeout=600
```

This pulls data from FRED, ECB, and Eurostat, builds the panel, runs regressions, and saves outputs.

---

## Econometric approach

Two-way fixed-effects panel regression (country + quarter FE), standard errors clustered by country.

```
HPI_Growth_it = α_i + γ_t + β · MortgageRate_it + δ′X_it + ε_it
```

**Controls:** GDP growth, unemployment, building permits.

### Models

| Model | Description |
|-------|-------------|
| **1** | Baseline — QoQ HPI growth ~ mortgage rate + controls |
| **2** | Distributed lag — mortgage rates at t … t−4 |
| **3** | Sub-period — pre-QE (2003–2014) vs QE era (2015–2019) |
| **4A** | **Phase A primary** — YoY HPI growth ~ mortgage rate + controls |
| **4B** | Rate changes (Δrate) + one lag |
| **4C** | Mortgage rate × post-QE interaction |
| **4D** | Robustness — 2009–2019 subsample |

---

## Key results (Phase A)

| Model | Coefficient | p-value | Interpretation |
|-------|-------------|---------|----------------|
| **4A YoY baseline** | −2.22 | 0.004 | +1 pp mortgage rate → −2.2 pp YoY HPI growth |
| **4B Δ rate (t)** | +1.84 | 0.027 | Short-run rate change effects |
| **4B Δ rate (t−1)** | −1.82 | 0.021 | Offsetting lag; cumulative ≈ 0 |
| **4C pre-QE slope** | −2.85 | 0.001 | Strong pass-through before 2015 |
| **4C post-QE slope** | +0.56 | — | Link weakens at the effective lower bound |
| **4D 2009–2019** | −2.17 | 0.053 | Robust to excluding COVID period |

Baseline Model 1 (QoQ): mortgage coefficient **−0.73** (p = 0.006).

---

## Interpretation of results

All coefficients below are from the current panel (747 observations; BIS HPI index 2010 = 100). Estimates are **conditional associations**, not causal effects, unless noted otherwise. Country and quarter fixed effects absorb time-invariant country differences and common euro-area shocks.

### Answer to the research question

**Yes.** Across the full sample, lower mortgage rates are consistently associated with faster house price growth. The preferred specification (Model 4A) implies that a **1 pp increase in the mortgage rate** is linked to roughly **2.2 pp lower year-over-year house price growth** (coefficient = −2.22, p = 0.004, N = 720). Equivalently, a 100 bp cut in mortgage rates is associated with about **2 pp higher YoY house price growth**, holding GDP, unemployment, and permits fixed.

This magnitude is economically plausible and aligns with the negative raw correlations between mortgage rates and HPI growth (−0.24 QoQ, −0.26 YoY).

### Baseline models (quarter-over-quarter growth)

**Model 1** confirms the relationship at quarterly frequency: the mortgage coefficient is **−0.73** (p = 0.006)—a 1 pp higher rate corresponds to 0.73 pp slower *quarterly* growth. GDP growth is positive but only marginally significant (p ≈ 0.09); unemployment is negative and highly significant (−0.27, p < 0.001), consistent with weaker labour markets dampening house prices.

**Model 2** (distributed lags) shows the contemporaneous effect is strongest (−1.08, p = 0.009), while individual lags mostly offset each other. The **cumulative effect over four quarters** of a 1 pp rate *cut* is about **+0.52 pp** of quarterly house price growth—pass-through builds gradually rather than arriving in a single quarter.

**Model 3** (split sample, QoQ) compares pre-QE (2003–2014) and the QE era (2015–2019):

| Period | Mortgage coefficient |
|--------|---------------------|
| Pre-QE (2003–2014) | −1.01 |
| QE era (2015–2019) | −1.81 |

The link appears stronger during the QE period in this QoQ specification, though these are separate regressions rather than a formal interaction test (see Model 4C).

### Phase A — primary specification (Model 4A)

Model 4A is the **main result** for this project. YoY house price growth is less noisy than QoQ growth and better matches how housing cycles are typically assessed.

| Variable | Coefficient | p-value | Interpretation |
|----------|-------------|---------|----------------|
| Mortgage rate | −2.22 | 0.004 | Core finding — cost of borrowing and YoY house price growth move in opposite directions |
| GDP growth | +0.47 | 0.011 | Stronger activity supports house price growth |
| Unemployment | −1.22 | < 0.001 | Higher unemployment dampens house prices |
| Building permits | −0.004 | 0.28 | Supply proxy not significant in the full sample |

Within-country R² is 0.20, indicating mortgage rates and controls explain a meaningful share of within-country YoY variation after fixed effects.

### Rate levels vs. rate changes (Model 4B)

Model 4B replaces the mortgage *level* with quarterly *changes* (Δrate) and one lag:

- **Δ rate (t):** +1.84 (p = 0.027)
- **Δ rate (t−1):** −1.82 (p = 0.021)
- **Cumulative effect of a −1 pp cut (t + t−1):** ≈ −0.02 pp YoY (essentially zero)

**Interpretation:** House prices respond to the **level** of borrowing costs over time, not to each individual rate move. Contemporaneous and lagged rate changes largely cancel, which is consistent with gradual adjustment in housing markets.

### Structural break at the effective lower bound (Model 4C)

Model 4C adds a mortgage rate × post-QE interaction (post-QE = 1 from 2015Q1):

| Term | Coefficient | p-value |
|------|-------------|---------|
| Mortgage rate (pre-2015 slope) | −2.85 | 0.001 |
| Rate × post-QE | +3.41 | 0.079 |
| **Implied post-2015 slope** | **+0.56** | — |

Before the ECB Asset Purchase Programme, mortgage rates strongly predict YoY house price growth. After 2015, the implied slope is near zero—the pass-through link **weakens or disappears** when policy rates hit the effective lower bound and mortgage rate variation compresses. The interaction is marginally significant (p = 0.079), suggesting a structural shift rather than a precise break date.

### Robustness (Model 4D)

Re-estimating Model 4A on **2009–2019 only** (excluding the GFC onset and COVID-19) yields a coefficient of **−2.17** (p = 0.053, N = 396). The sign and magnitude match the full-sample result, indicating the main finding is not driven by crisis-period outliers. In this subsample, building permits become significant (−0.027, p = 0.027), suggesting supply conditions matter more in the pre-COVID decade.

### Summary table

| Question | Answer |
|----------|--------|
| Do lower mortgage rates predict higher house price growth? | **Yes**, statistically significant in Models 1 and 4A |
| How large is the effect? | ~**2 pp YoY** house price growth per **1 pp** mortgage rate (Model 4A) |
| Do rate *cuts* or rate *levels* matter more? | **Levels** (Model 4B: changes cancel out) |
| Did the relationship change after 2015? | **Likely yes** — strong pre-QE, weak post-QE (Model 4C) |
| Is the result robust to sample choice? | **Mostly yes** — similar coefficient in 2009–2019 window (Model 4D) |

### Caveats

- Results are **associations** within a fixed-effects framework; omitted variables (credit supply, expectations, macroprudential policy) may bias coefficients.
- Nine countries is a small panel; inference relies on within-country time variation.
- Post-2015 estimates are less reliable because mortgage rates had limited room to fall further.
- These results describe the **euro area**, not Canada directly—though the housing–rates mechanism is relevant to central-bank research more broadly.

---

## Project structure

```
├── euro_area_housing_panel.ipynb   # Main analysis notebook
├── euro_area_panel.csv             # Merged panel dataset
├── phase_a_results.csv             # Phase A model comparison
├── results_table.csv               # Baseline Models 1–2 summary
├── requirements.txt
├── chart1_hpi_by_country.png       # YoY HPI by country
├── chart2_mortgage_vs_hpi.png      # Mortgage rate vs HPI scatter
├── chart3_correlation.png          # Correlation heatmap
├── chart4_impulse_response.png     # Distributed-lag cumulative effect
└── chart5_subperiod.png            # Pre-QE vs QE era coefficients
```

---

## Data quality improvements

Several iterative fixes expanded the sample from **555 → 747** observations and corrected coefficient scaling:

1. **HPI series** — switched from `N368BIS` (growth rates) to `N628BIS` (index levels)
2. **Unemployment** — FRED OECD series (full 2003–2023) replaced Eurostat (2009+ only)
3. **Eurostat filters** — corrected GDP unit (`PD_PCH_PRE_EUR`) and permits codes
4. **Missing permits** — linear interpolation within country (7 gaps filled)
5. **Phase A specs** — YoY DV, rate-change model, QE interaction, robustness window

---

## Possible extensions

For causal identification beyond Phase A:

| Extension | Method |
|-----------|--------|
| Driscoll–Kraay SEs | Robust inference to cross-country correlation |
| IV-2SLS | ECB policy rate instruments mortgage rate |
| Local projections | Dynamic IRFs (Jordà 2005), horizons 0–8 quarters |

---

## Requirements

- Python 3.10+
- See `requirements.txt`: `pandas`, `numpy`, `linearmodels`, `matplotlib`, `seaborn`, `requests`, `eurostat`, `jupyter`

---

## Author

**Bhavleen Kaur** — [bhavleenkaur-econ](https://github.com/bhavleenkaur-econ)

---

## License

This project is for research and portfolio purposes. Data remain subject to the terms of FRED, ECB, and Eurostat.
