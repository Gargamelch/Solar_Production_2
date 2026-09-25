---
title: "Solar Energy Production Dashboard"
emoji: ⚡
colorFrom: yellow
colorTo: red
sdk: docker
app_file: app.py
pinned: false
license: gpl-3.0
short_description: Solar Energy Production analysis and prediction
---
# ☀️ Solar France — Solar Energy Production Dashboard

An interactive dashboard for analyzing and predicting solar energy production across French regions, built with Streamlit and deployed on Hugging Face Spaces.

[![Hugging Face](https://img.shields.io/badge/🤗%20Hugging%20Face-Solar__Production-yellow)](https://gargamelch-solar-production.hf.space/)
[![GitHub](https://img.shields.io/badge/GitHub-Solar__Production__2-181717?style=flat&logo=github)](https://github.com/Gargamelch/Solar_Production_2)
[![Python](https://img.shields.io/badge/Python-3.13-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/downloads/release/python-3130/)
[![License](https://img.shields.io/badge/License-GPL--3.0-blue)](https://www.gnu.org/licenses/gpl-3.0.html)
![Version](https://img.shields.io/badge/version-1.3.0-E67E22)

---

## 📋 Table of Contents
- [Overview](#-overview)
- [Features](#-features)
- [Project Structure](#-project-structure)
- [Data Sources](#%EF%B8%8F-data-sources)
- [Data Pipeline](#%EF%B8%8F-data-pipeline)
- [Dataset Description](#-dataset-description)
- [Installation](#-installation)
- [Docker](#-docker)
- [Machine Learning](#-machine-learning)
- [Notes & Limitations](#-notes--limitations)

---

## 🔍 Overview

This project combines meteorological data from **Météo-France**, electricity production data from **RTE** (Réseau de Transport d'Électricité) and installed solar capacity from **SDES** to analyze and predict solar energy production across the 12 metropolitan French regions (Corsica excluded) from **January 2020 to January 2026**.

The app includes two pages:
- **Dashboard**: historical production analysis across 6 tabs (overview, regional map, production, solar radiation, installations, raw data)
- **Predictions**: next-day (J+1) solar production forecast per m² of solar panel, compared with the actual value when available

---

## ✨ Features

**Dashboard**
- 📊 **Overview**: total production, daily average, growth between full years, average installed capacity, yearly and monthly production trends
- 🗺️ **Regions**: choropleth map of production by region with a ranking table
- ⚡ **Production**: daily production statistics (mean, median, max), distribution of daily production, monthly capacity factor
- 🌤️ **Solar Radiations**: correlation between visible radiation and production per installed capacity, monthly and seasonal comparison
- 🔌 **Installations**: number of installations and installed capacity over time, with growth
- 🗃️ **Data**: dataset preview, download link and sources
- 🔍 Filters by year range and region

**Predictions**
- 🤖 Next-day visible radiation prediction converted into solar production per m²
- 🌡️ Daily weather KPIs for the selected day (temperature, rainfall, wind, radiation)
- 🎯 Gauge comparing predicted vs actual production for a user-defined panel surface (10 to 10,000 m²)
- 🔮 Forecast-only mode for the day after the last available data

**Deployment**
- 📦 Dockerized and deployed on Hugging Face Spaces
- ☁️ Datasets loaded remotely from Hugging Face and cached for one week

---

## 📁 Project Structure

```
📁 Solar_Production_2
├── 🐳 Dockerfile
├── 📄 requirements.txt
├── 🗒️ app.py                          ← Navigation entry point
├── 🗒️ utils.py                        ← Constants, data loading & caching, SVG helpers
├── 📁 pages
│   ├── 🗒️ Dashboard.py                ← Production analysis dashboard
│   └── 🗒️ Predictions.py              ← J+1 prediction page
├── 📁 static                          ← Logos and SVG icons
├── 📁 .streamlit                      ← Streamlit configuration
├── 📓 RTE_concat.ipynb                ← Aggregates raw RTE files per year
├── 📓 solar_production_v35.ipynb      ← Data preparation, EDA and machine learning
└── 📁 data                            ← Raw data (not needed to run the app)
    ├── 📁 coordinates
    │   ├── 📄 departements-20180101.shp
    │   └── 📄 ...
    ├── 📁 Meteo-France
    │   └── 📄 QUOT_SIM2_previous-2020-202605.csv
    ├── 📁 RTE
    │   ├── 📁 Regions
    │   │   ├── 📄 eCO2mix_RTE_*_Annuel-Definitif_*.xls
    │   │   ├── 📄 eCO2mix_RTE_*_En-cours-Consolide.xls
    │   │   └── 📄 ...
    │   ├── 📄 rte_regions_2013.csv ... rte_regions_2024.csv
    │   └── 📄 rte_regions_en_cours.csv
    └── 📁 solar_installations
        └── 📄 statistiques_sdes_2026_t1_*.xlsx
```

> The app does not read the local `data/` folder: the two final datasets (`solar_production.csv` and `solar_prod_predictions.csv`) are produced by the notebook and hosted on Hugging Face.

---

## 🗃️ Data Sources

| Source | Description | Format | Coverage used |
|--------|-------------|--------|---------------|
| ⚡ [RTE eCO2mix](https://www.rte-france.com/donnees-publications/eco2mix-donnees-temps-reel/telecharger-indicateurs) | Solar electricity production by region (half-hourly) | `.xls` (tab-separated) | 2020–2026 |
| 🌤️ [Météo-France SIM2](https://www.data.gouv.fr/datasets/donnees-changement-climatique-sim-quotidienne) | Daily meteorological data on an 8 km grid | `.csv` | 2020–2026 |
| ☀️ [SDES](https://www.statistiques.developpement-durable.gouv.fr/tableau-de-bord-solaire-photovoltaique-premier-trimestre-2026) | Quarterly number of installations and installed capacity by region | `.xlsx` | 2020–2026 |
| 🗺️ [OpenStreetMap / data.gouv.fr](https://www.data.gouv.fr/datasets/contours-des-departements-francais-issus-d-openstreetmap) | French département boundaries (shapefile) | `.shp` | - |
| 🗺️ [france-geojson](https://github.com/gregoiredavid/france-geojson) | Simplified region boundaries for the map | `.geojson` | - |

---

## ⚙️ Data Pipeline

1. **RTE_concat.ipynb**: merges the 12 regional RTE files into one CSV per year (2013–2024) plus the current, not yet final, data (`en_cours`).
2. **solar_production_v35.ipynb**:
   - Loads RTE data, converts half-hourly MW values into daily **TWh** per region
   - Loads SDES installations (quarterly) and forward-fills them to a daily frequency
   - Converts the Météo-France grid coordinates (Lambert II étendu, EPSG:27572) to GPS (EPSG:4326) and assigns each grid point to a département, then to a region, using a spatial join (nearest département for points on borders or coasts)
   - Aggregates Météo-France data per day and region (mean of all grid points)
   - Merges the three sources and computes `production_per_capacity`
   - Exports `solar_production.csv` (used by the Dashboard)
   - Runs the EDA, trains and compares the models, and exports `solar_prod_predictions.csv` (used by the Predictions page)

Date range: **2020-01-01 → 2026-01-31**, 12 regions, **26,676 rows**.

---

## 📊 Dataset Description

### `solar_production.csv` (Dashboard)

Daily data per region:

| Column | Description | Unit | Precision | Source |
|--------|-------------|------|-----------|--------|
| `date` | Measurement date | YYYY-MM-DD | - | 🌤️ Meteo-France |
| `region` | French administrative region | - | - | 🔮 Engineered |
| `snowfall` | Solid precipitation (daily cumul 06UTC-06UTC) | mm | 1/10 | 🌤️ Meteo-France |
| `rainfall` | Liquid precipitation (daily cumul 06UTC-06UTC) | mm | 1/10 | 🌤️ Meteo-France |
| `daily_avg_temp` | Average daily temperature (00UTC-00UTC) | °C | 1/10 | 🌤️ Meteo-France |
| `daily_avg_wind_speed` | Average daily wind speed (00UTC-00UTC) | m/s | 1/10 | 🌤️ Meteo-France |
| `daily_avg_specific_humidity` | Specific humidity (00UTC-00UTC) | g/kg | - | 🌤️ Meteo-France |
| `atmospheric_radiation` | Atmospheric radiation (daily cumul 00UTC-00UTC) | J/cm² | - | 🌤️ Meteo-France |
| `visible_radiation` | Visible radiation (daily cumul 00UTC-00UTC) | J/cm² | - | 🌤️ Meteo-France |
| `daily_avg_relative_humidity` | Relative humidity (00UTC-00UTC) | % | - | 🌤️ Meteo-France |
| `total_evapotranspiration` | Total evapotranspiration (daily cumul 06UTC-06UTC) | mm | 1/10 | 🌤️ Meteo-France |
| `potential_evapotranspiration` | Potential evapotranspiration - Penman-Monteith FAO-56 | mm | 1/10 | 🌤️ Meteo-France |
| `effective_rainfall` | Effective rainfall (daily cumul 06UTC-06UTC) | mm | 1/10 | 🌤️ Meteo-France |
| `daily_avg_soil_moisture_index` | Soil moisture index (06UTC-06UTC) | % | - | 🌤️ Meteo-France |
| `soil_drought_index_10d` | 10-day integrated soil drought index | - | - | 🌤️ Meteo-France |
| `drainage` | Drainage (daily cumul 06UTC-06UTC) | mm | 1/10 | 🌤️ Meteo-France |
| `runoff` | Runoff (daily cumul 06UTC-06UTC) | mm | 1/10 | 🌤️ Meteo-France |
| `daily_avg_snow_water` | Snow water equivalent - daily average (06UTC-06UTC) | mm | 1/10 | 🌤️ Meteo-France |
| `snow_water_equivalent_6h` | Snow water equivalent at 06UTC | mm | 1/10 | 🌤️ Meteo-France |
| `daily_avg_snow_depth` | Snow depth - daily average (06UTC-06UTC) | m | - | 🌤️ Meteo-France |
| `snow_depth_6h` | Snow depth at 06UTC | m | - | 🌤️ Meteo-France |
| `max_snow_depth` | Maximum hourly snow depth during the day | m | - | 🌤️ Meteo-France |
| `daily_avg_snow_cover_fraction` | Snow cover fraction - daily average (06UTC-06UTC) | % | - | 🌤️ Meteo-France |
| `snowmelt_runoff` | Runoff at the base of the snowpack (daily cumul 06UTC-06UTC) | mm | - | 🌤️ Meteo-France |
| `root_liquid_water` | Liquid water content in root layer at 06UTC | m³/m³ | - | 🌤️ Meteo-France |
| `root_frozen_water` | Frozen water content in root layer at 06UTC | m³/m³ | - | 🌤️ Meteo-France |
| `min_temp` | Minimum temperature over 24h (18UTC-18UTC) | °C | 1/10 | 🌤️ Meteo-France |
| `max_temp` | Maximum temperature over 24h (06UTC-06UTC) | °C | 1/10 | 🌤️ Meteo-France |
| `solar` | Solar power production (sum of half-hourly values) | MW | - | ⚡ RTE |
| `TWh` | Daily solar energy production | TWh | - | ⚡ RTE |
| `installation_number` | Number of connected solar installations | - | - | ☀️ SDES |
| `capacity_power` | Installed solar capacity | MW | - | ☀️ SDES |
| `production_per_capacity` | Daily production / (installed capacity × 24 h): capacity factor | ratio | - | 🔮 Engineered |

### `solar_prod_predictions.csv` (Predictions)

All the columns above, plus:

| Column | Description | Unit | Source |
|--------|-------------|------|--------|
| **Target** |
| `target` | Next day visible radiation (J+1) | J/cm² | 🔮 Engineered |
| **Lag features** |
| `radiation_J1` / `J2` / `J3` | Visible radiation 1, 2 and 3 days ago | J/cm² | 🔮 Engineered |
| **Rolling features** (previous 7 days, current day excluded) |
| `radiation_7d_mean` | 7-day rolling mean of visible radiation | J/cm² | 🔮 Engineered |
| `temp_7d_mean` | 7-day rolling mean of temperature | °C | 🔮 Engineered |
| `humidity_7d_mean` | 7-day rolling mean of relative humidity | % | 🔮 Engineered |
| `rainfall_7d_sum` | 7-day rolling sum of rainfall | mm | 🔮 Engineered |
| **Seasonality encoding** |
| `day_sin` / `day_cos` | Sine / cosine encoding of the day of year | - | 🔮 Engineered |
| **Radiation prediction** |
| `visible_radiation J+1 predit` | Predicted next day visible radiation | J/cm² | 🔮 Engineered |
| `radiation_J+1 reel` | Actual next day visible radiation | J/cm² | 🔮 Engineered |
| `ecart prediction` | Prediction error (predicted − actual) | J/cm² | 🔮 Engineered |
| **Solar production** |
| `Kwh/m2` | Visible radiation converted to energy (J/cm² ÷ 360) | kWh/m² | 🔮 Engineered |
| `production_solaire/m2(en Kwh)` | Estimated production per m² at 20% panel efficiency | kWh/m² | 🔮 Engineered |
| `Kwh/m2 prédit J+1` | Predicted next day radiation energy | kWh/m² | 🔮 Engineered |
| `production_solaire/m2(en Kwh) J+1 predit` | Predicted next day production per m² at 20% efficiency | kWh/m² | 🔮 Engineered |

> On a given row (day D), the `J+1` columns refer to day D+1. The Predictions page therefore reads the prediction from the previous day's row.

---

## 🚀 Installation

### Prerequisites

- Python 3.13+ (3.12 minimum: the app uses nested quotes in f-strings)
- pip

### Local setup

```bash
# Clone the repository
git clone https://github.com/Gargamelch/Solar_Production_2.git
cd Solar_Production_2

# Create and activate a virtual environment
python3.13 -m venv .venv
source .venv/bin/activate  # Linux/Mac
# or
.venv\Scripts\activate     # Windows

# Install dependencies
pip install -r requirements.txt

# Run the app
streamlit run app.py
```

An internet connection is needed: the datasets and the region GeoJSON are downloaded at startup, then cached.

---

## 🐳 Docker
### Create a Dockerfile to run the app locally:
```dockerfile
FROM anaconda/miniconda:26.3.2

# Update system
RUN apt-get update -y
RUN apt-get install nano unzip curl -y

# Install dependencies
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

# Set working directory
WORKDIR /app

# Copy needed local files
COPY app.py /app/app.py
COPY utils.py /app/utils.py
COPY pages/ /app/pages/
COPY .streamlit/ /app/.streamlit/
COPY static/ /app/static/
COPY requirements.txt /app/requirements.txt

CMD ["python", "-m", "streamlit", "run", "app.py", \
     "--server.port=8501", \
     "--server.address=0.0.0.0", \
     "--server.headless=true"]
```

### Then run:
```bash
# Build the image
docker build -t solar-app .

# Run locally
docker run -p 8501:8501 solar-app
```

Open [http://localhost:8501](http://localhost:8501) in your browser.

The production image is deployed on **Hugging Face Spaces** on port `7860`.

---

## 🤖 Machine Learning

### Objective

Predict the **next-day visible radiation (J+1)** for each region, then convert it into **solar production per m²**:

```
Predicted radiation (J/cm²) → ÷ 360 → kWh/m² → × 20% panel efficiency → production per m²
```

### Features used

All regions are trained together in a single model, with the region one-hot encoded (35 numeric + 1 categorical feature, 46 after encoding).

| Category | Features |
|----------|----------|
| Radiation | `visible_radiation`, `atmospheric_radiation`, `radiation_J1`, `radiation_J2`, `radiation_J3`, `radiation_7d_mean` |
| Seasonality | `day_sin`, `day_cos` |
| Weather (current day) | temperatures, humidity, wind, rainfall, evapotranspiration, snow, soil and runoff variables |
| Weather (last 7 days) | `temp_7d_mean`, `humidity_7d_mean`, `rainfall_7d_sum` |
| Location | `region` (one-hot encoded) |

Same-day production columns (`solar`, `TWh`, `installation_number`, `capacity_power`, `production_per_capacity`) are excluded.

### Feature selection

- **Lags**: ACF and PACF of the national radiation series show a strong lag-1 effect. 3, 5, 10 and 30 lags were compared: gains beyond 3 lags are within the fold-to-fold variation, so **3 lags** were kept.
- **Yearly lag (J-365)**: tested and **not kept**. It brings no improvement: seasonality is already encoded by `day_sin`/`day_cos`, last year's weather on a given day has no predictive value, and the lag removes a full year of training data.

### Pipeline

```
Numeric features → StandardScaler ┐
                                  ├→ Ridge (alpha = 200) → clip at 0 → Predicted J+1 radiation
Region → OneHotEncoder ───────────┘
```

Predictions are clipped at 0, since radiation can never be negative.

### Validation strategy

> ⚠️ Data is sorted chronologically, with no random shuffling, so the model is never trained on the future.

- **Hold-out**: train before 2024-01-01 (17,448 rows), test from 2024-01-01 (9,132 rows)
- **Hyperparameter tuning**: `GridSearchCV` with a 5-fold `TimeSeriesSplit` on the training set
- **Final evaluation**: 5-fold `TimeSeriesSplit` on the full dataset, preprocessing refitted on each fold

| Fold | Train | Test |
|------|-------|------|
| 1 | 2020-01 → 2021-01 | 2021-01 → 2022-01 |
| 2 | 2020-01 → 2022-01 | 2022-01 → 2023-01 |
| 3 | 2020-01 → 2023-01 | 2023-01 → 2024-01 |
| 4 | 2020-01 → 2024-01 | 2024-01 → 2025-01 |
| 5 | 2020-01 → 2025-01 | 2025-01 → 2026-01 |

### Model comparison

Hold-out test set (2024-01 → 2026-01) and mean over the 5 cross-validation folds (training set):

| Model | Test R² | Test MAE | Test RMSE | CV R² (mean) | CV RMSE (mean) |
|-------|---------|----------|-----------|--------------|----------------|
| Naive baseline (J+1 = today) | 0.708 | 316.5 | 436.5 | 0.657 | - |
| Linear Regression, raw features | 0.761 | 300.9 | 395.2 | 0.675 | 406.5 |
| Linear Regression + lags | 0.778 | 287.5 | 380.5 | 0.706 | 389.7 |
| Linear Regression, full features | 0.792 | 279.4 | 368.5 | 0.726 | 377.2 |
| Lasso (alpha = 2) | 0.788 | 281.5 | 371.7 | 0.742 | 368.5 |
| **Ridge (alpha = 200)** | **0.789** | **282.0** | **371.4** | **0.742** | **369.0** |
| Random Forest | 0.800 | 273.8 | 361.6 | 0.740 | 369.1 |
| XGBoost | 0.801 | 272.1 | 360.5 | 0.748 | 363.7 |

*MAE and RMSE in J/cm².*

### Why Ridge?

The top models are within one fold standard deviation of each other, so none is clearly better. Ridge was chosen because it is:
- **Stable** across folds, with no overfitting (unlike Random Forest: train R² 0.87 vs test 0.80)
- **Robust to multicollinearity**: many weather variables are strongly correlated (e.g. `rainfall` and `effective_rainfall`), which makes plain linear regression coefficients unstable
- **Simple and interpretable**, and fast enough to retrain easily

### Final model evaluation

Ridge (alpha = 200), 5-fold `TimeSeriesSplit` on the full dataset:

| | R² | MAE (J/cm²) | RMSE (J/cm²) |
|-|----|-------------|--------------|
| Naive baseline | 0.701 | 318.8 | 439.0 |
| Train | 0.786 | 291.3 | 382.0 |
| Test | 0.779 | 288.3 | 377.3 |

> ✅ Test performance is close to train: no significant overfitting. The model reduces RMSE by about 14% compared with the naive baseline.

Most important variables: `visible_radiation`, `day_cos`, `daily_avg_temp`, `atmospheric_radiation`, `potential_evapotranspiration` and the region.

---

## 📝 Notes & Limitations

- Corsica is excluded because it is not covered by RTE's regional data.
- Installed capacity data is quarterly and forward-filled to a daily frequency.
- Météo-France grid points are assigned to regions through a spatial join with OpenStreetMap département boundaries.
- Per-m² production is a **theoretical estimate** assuming **20% panel efficiency**. It does not account for panel orientation, shading, temperature or equipment age.
- The model predicts the next day from **observed** weather data. A real operational forecast would use weather forecasts, as SIM2 data is published with a delay.
- The exported predictions come from the model trained on data before 2024: predictions for 2020–2023 are **in-sample** and will look better than genuine forecasts. Predictions from 2024 onward are out-of-sample.
- The Predictions page allows one day after the last available data (prediction only) and a second day with neither data nor prediction.
