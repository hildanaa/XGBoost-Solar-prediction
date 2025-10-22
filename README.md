## Solar Energy Prediction — AMS 2013–2014 Contest

Forecasting daily **solar energy output** for the **ACME** station using historical weather data, PCA components, and Numerical Weather Prediction (NWP) variables.  
This project replicates a real-world long-horizon forecasting challenge: predicting five years (2008–2012) of missing solar output based on past data (1994–2007).

## Project Overview

This repository contains an end-to-end data science workflow:
1. **Exploratory Data Analysis (EDA)** — identifying patterns, trends, and seasonality.  
2. **Data Cleaning** — missing value handling, outlier detection, and winsorization.  
3. **Feature Engineering** — incorporating cyclical time encoding (month/day/year), PCA, and NWP predictors.  
4. **Model Development** — comparing baseline and advanced regression models.  
5. **Model Selection** — choosing the most stable and generalizable model for long-horizon forecasting.

## ⚙️ Data Sources

- `solar_dataset.csv` — Daily solar production for 98 stations (1994–2007 real values).  
- `station_info.csv` — Station metadata (latitude, longitude, elevation).  
- `additional_variables.csv` — NWP features (V1–V100) representing weather forecasts.

Each row corresponds to one day, with PCA/NWP weather features capturing atmospheric and climatic conditions.

## key Steps Implemented

### 1. Data Preparation
- Parsed `Date` to datetime format and sorted chronologically.
- Merged solar, PCA, and NWP datasets on `Date`.
- Identified feature groups:  
  - **Station outputs:** 98 target columns (e.g., ACME)  
  - **PCA components:** `PC1–PC357`  
  - **NWP variables:** `V1–V100`

###  2. Missingness & Outlier Analysis
- Confirmed all station data (including ACME) missing post-2007-12-31.  
- Applied **season-aware winsorization** — capping ACME values at monthly 1st/99th percentiles.  
  *→ Stabilizes training and reduces MAPE distortion.*

###  3. Exploratory Data Analysis (EDA)
- **Time-Series Trends:** Seasonal peaks mid-year; consistent yearly patterns.  
- **STL Decomposition:** Extracted trend, seasonal, and residual components.  
- **Autocorrelation (ACF):** Strong short-term and annual lags.  
- **Seasonality Plots:** Monthly/Yearly mean ± std, boxplots, calendar heatmaps.

###  4. Geographic Analysis
- Station map (latitude vs longitude) colored by mean solar output.  
- Solar production vs **elevation** and **latitude** scatter/regression plots.  
  *→ Confirms physical geographic trends.*

### 5. Feature Correlation
- Correlation heatmaps across PCA and NWP predictors.  
- Top correlated features with ACME: **PC1, PC2, VINI**.  
- Deseasonalized ACME correlation confirmed genuine meteorological relationships.

### 6. Feature Engineering
- **Cyclical encoding:** month/day/day-of-year as sine/cosine pairs to preserve continuity.  
- **Chronological splitting:**  
  - Train (1994–2005)  
  - Validation (2006–2007)  
  - Test (2008–2012 — missing target)  
  *→ Prevents data leakage across time.*

### 7. Modeling
Tested multiple regression algorithms:
| Model | Key Finding |
|--------|--------------|
| Linear Regression | Stable baseline |
| Ridge & Lasso | Best among linear models (MAE ≈ 2.62M) |
| Random Forest | Underfit, higher MAE |
| **XGBoost** | Best generalization (MAE ↓ from 2.59M → 2.56M) |
| **LightGBM** | Lowest validation MAE (2.39M) but worsened on full retraining (2.49M) |

> 🏁 **Final Model: XGBoost** — chosen for stronger generalization to unseen data (2008–2012).

###  8. Final Prediction
- Refit tuned XGBoost on 1994–2007 data.  
- Predicted ACME solar output for 2008–2012.  
- Output saved as `acme_predictions.csv`.

---

## Results Summary

| Metric | Validation (2006–2007) | Full Data (1994–2012) |
|--------|-------------------------|------------------------|
| **LightGBM MAE** | 2.39M | 2.49M |
| **XGBoost MAE** | 2.59M | **2.56M** |
| **Filtered MAPE** | ≈ 25% | — |

**Interpretation:**  
LightGBM slightly overfit the validation window.  
XGBoost improved with more data → better long-horizon generalization → chosen as the final submission model.

##  Tools & Libraries
- **Python 3.10+**
- **Pandas**, **NumPy**, **Matplotlib**, **Seaborn**
- **Scikit-learn** — preprocessing, scaling, Ridge/Lasso
- **XGBoost**, **LightGBM**
- **Statsmodels** — STL decomposition
- **ReportLab** — for generating summary PDF

## Key Insights
- Solar output shows **strong annual periodicity** and mild long-term drift.
- PCA features (PC1/PC2) capture global weather variability effectively.
- Deseasonalized analysis revealed **true meteorological drivers** beyond time-based seasonality.
- XGBoost maintained stability when trained on the full 14-year dataset, making it ideal for **unseen-year forecasting**.

