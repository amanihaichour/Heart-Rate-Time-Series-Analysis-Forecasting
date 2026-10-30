# Heart Rate Time Series Analysis & Forecasting (ARIMA)

Using the Box-Jenkins methodology in **R**, I modeled and forecast the heart rate (HR) of a healthy individual recorded over 24 hours with a wrist-worn actigraph.

---

## Overview

Heart rate reflects circadian rhythm, physical activity, stress and autonomic nervous system regulation. Being able to forecast it a few minutes ahead is useful for patient monitoring, wearable devices and early detection of anomalies such as arrhythmias.

The goal here: find a simple ARIMA model that captures short-term HR dynamics, validate it properly, and test it on data it has never seen.

## Dataset

- **Source:** [MMASH – Multilevel Monitoring of Activity and Sleep in Healthy people](https://physionet.org/content/mmash/1.0.0/) (PhysioNet)
- **Signal:** heart rate in bpm, sampled every second (`Actigraph.csv`)
- **Preprocessing:**
  - kept day 1 only for a clean 24h window
  - removed physiologically implausible values (HR ≤ 30 or ≥ 220 bpm)
  - resampled to **per-minute averages** → **n = 830** observations
- **Train / test split:** first 815 observations for modelling, last **15 minutes withheld** for forecast evaluation

> The raw data is not included in this repo. Download `Actigraph.csv` from PhysioNet and place it next to the notebook.

## Methodology

1. **Variance stabilization** – variance of the two halves of the series compared (ratio = 1.12) → no log transform needed.
2. **Stationarity** – ADF test on raw HR (p = 0.051, non-stationary) and on the first difference (p < 0.01, stationary) → **d = 1**.
3. **Model specification** – ACF/PACF of the differenced series suggest a mixed ARMA structure.
4. **Model selection** – 6 candidate ARIMA models compared with AIC and BIC.
5. **Diagnostics** – residual plots, ACF/PACF of residuals, Q-Q plot, Ljung-Box, Shapiro-Wilk, runs test.
6. **Forecasting** – 15-step-ahead forecast with 95% intervals, compared with the withheld data.

## Results

**Selected model: ARIMA(1,1,1)**

| Candidate | AIC | BIC |
|---|---|---|
| **ARIMA(1,1,1)** | **5303.75** | **5317.85** |
| ARIMA(1,1,2) | 5305.72 | 5324.53 |
| ARIMA(2,1,1) | 5305.73 | 5324.54 |
| ARIMA(2,1,2) | 5307.50 | 5331.01 |
| ARIMA(0,1,1) | 5336.95 | 5346.35 |
| ARIMA(1,1,0) | 5367.30 | 5376.70 |

Estimated coefficients: AR(1) = 0.448 (s.e. 0.059), MA(1) = −0.807 (s.e. 0.040), σ² = 39.35.

**Residual diagnostics**

| Test | Result |
|---|---|
| Ljung-Box (lags 10 / 20 / 30) | p = 0.164 / 0.519 / 0.761 → no leftover autocorrelation |
| Runs test | p = 0.674 → residuals look random |
| Shapiro-Wilk | p ≈ 0 → heavy tails (likely bursts of physical activity) |

**Forecast accuracy (15 minutes ahead)**

| MAE | RMSE | MAPE |
|---|---|---|
| 4.60 bpm | 5.09 bpm | 5.22 % |

All 15 actual values fall inside the 95% prediction intervals.

## Limitations

- Forecasts quickly revert to a near-constant level (~94 bpm), so the model over-predicts when HR is trending down (e.g. entering a rest period).
- Residuals are not normal (heavy tails from activity bursts).
- Only one person and one day of data.
- A seasonal model (SARIMA) or one using activity/posture as exogenous variables would be a natural next step.

## Tech stack

R · Google Colab · `forecast` · `tseries` · `dplyr` · `ggplot2` · `lubridate` · `randtests`

## How to run

1. Download `Actigraph.csv` from the [MMASH dataset](https://physionet.org/content/mmash/1.0.0/).
2. Open `TSA_Heart_Rate_Variability_Analysis_and_forecasting.ipynb` in Google Colab (R runtime) or Jupyter with the R kernel.
3. Upload `Actigraph.csv` to the same folder.
4. Run all cells.

## Repository structure

```
.
├── TSA_Heart_Rate_Variability_Analysis_and_forecasting.ipynb
└── README.md
```
