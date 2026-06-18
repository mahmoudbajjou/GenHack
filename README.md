<div align="center">

# GenHack — ERA5-Land Bias Correction & Urban Heat Island Analysis

**4-week data science hackathon · Team 31 (sbyyaat) · Ecole Polytechnique**

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat-square&logo=jupyter&logoColor=white)](https://jupyter.org/)
[![ERA5](https://img.shields.io/badge/Data-ERA5--Land%20(ECMWF)-003580?style=flat-square)](https://cds.climate.copernicus.eu/)
[![Sentinel-2](https://img.shields.io/badge/Data-Sentinel--2%20NDVI-4CAF50?style=flat-square)](https://sentinel.esa.int/)
[![ECA&D](https://img.shields.io/badge/Data-ECA%26D%20Stations-E65100?style=flat-square)](https://www.ecad.eu/)

<br/>

> Quantifying and correcting the systematic bias between ERA5-Land reanalysis temperatures and ground weather stations in Madrid — with a focus on the **Urban Heat Island (UHI) effect** and its link to vegetation cover (NDVI).

</div>

---

## Table of Contents

- [Overview](#overview)
- [Data Sources](#data-sources)
- [Project Structure](#project-structure)
- [Methodology](#methodology)
- [Key Findings](#key-findings)
- [Correction Models](#correction-models)
- [Results](#results)
- [Installation](#installation)
- [Team](#team)

---

## Overview

ERA5-Land is a gridded reanalysis dataset produced by ECMWF. Despite its global coverage, it cannot fully capture local thermal effects — particularly the **Urban Heat Island (UHI)**, where dense urban areas retain significantly more heat than surrounding rural zones.

This project investigates this bias over Madrid (Spain) using 21 ground-truth weather stations and quarterly Sentinel-2 NDVI maps, then proposes four correction models ranging from simple linear regression to hybrid neural networks.

### Highlights

| | |
|---|---|
| **Study area** | Madrid, Spain (Comunidad de Madrid) |
| **Period** | 2020 – 2025 |
| **Ground stations** | 21 stations in Madrid · 900 stations across Spain |
| **Reanalysis grid** | ERA5-Land · 0.1° · daily max temperature |
| **Vegetation data** | Sentinel-2 NDVI · quarterly composites |
| **Best model MAE** | ~1.65°C (Neural Network on Madrid stations) |

---

## Data Sources

| Source | Description | Format | Resolution |
|---|---|---|---|
| **ERA5-Land** (ECMWF/Copernicus) | Daily max 2m temperature, precipitation, wind | NetCDF | 0.1° grid, daily |
| **Sentinel-2 NDVI** | Normalized Difference Vegetation Index | GeoTIFF | Quarterly composites |
| **ECA&D blend_tx** | Daily maximum temperature from European stations | CSV (per STAID) | Daily, 8568 European stations |
| **GADM v4.1** | Administrative boundaries (Europe) | GeoPackage | Sub-municipality level |

### Expected data directory layout

```
data/
├── gadm_410_europe.gpkg
├── derived-era5-land-daily-statistics/
│   ├── 2020_2m_temperature_daily_maximum.nc
│   ├── 2021_2m_temperature_daily_maximum.nc
│   └── ...
├── sentinel2_ndvi/
│   ├── ndvi_2019-12-01_2020-03-01.tif
│   └── ...
└── ECA_blend_tx/
    ├── stations.txt
    ├── TX_STAID000230.txt
    └── ...
```

---

## Project Structure

```
GenHack/
├── final-read-era5-netcdf_v2.ipynb     ← Data loading tutorial (ERA5, NDVI, ECA&D, GADM)
├── week2-team31-sbyyaat.ipynb          ← Week 2: bias analysis & UHI/NDVI correlation
├── week3-team31-sbyyaat.ipynb          ← Week 3: 4 correction models
├── weekfinal-team31-sbyyaat.ipynb      ← Final: synthesis & comparison
├── Team31-slides-week1.pdf             ← Week 1 presentation
├── Team31-slides-week2.pdf             ← Week 2 presentation
└── Team31-slides-week3.pdf             ← Week 3 presentation
```

---

## Methodology

### Pipeline

```
ERA5-Land NetCDF          Sentinel-2 NDVI          ECA&D Stations
(daily max temp)          (quarterly GeoTIFF)       (CSV per station)
       │                        │                         │
       ▼                        ▼                         ▼
  Spatial selection       Crop to city bbox          Parse + filter
  (nearest-neighbor)      Convert int8 → float       valid records only
       │                        │                         │
       └────────────────────────┴──────────────┬──────────┘
                                               ▼
                                   Bias = Station_TX − ERA5_TX
                                               │
                              ┌────────────────┼────────────────┐
                              ▼                ▼                ▼
                         Per-station      Seasonal         NDVI vs bias
                          analysis        breakdown        correlation
                              │
                              ▼
                     4 Correction Models
              (OLS · Random Forest · Neural Net · SAX-NN)
```

### Key preprocessing steps

- **ERA5**: extract nearest grid point per station (`xr.DataArray.sel(..., method='nearest')`), convert Kelvin → °C
- **NDVI**: `rasterio.mask` crop to city polygon, int8 [0–254] → float [-1, 1] scale
- **ECA&D**: parse DMS coordinates, filter `Q_TX == 0` (valid records only), convert to °C (`TX / 10`)
- **Temporal alignment**: daily ERA5 ↔ daily stations; quarterly NDVI reindexed with forward-fill

---

## Key Findings

### 1. ERA5-Land has a systematic positive bias in urban Madrid

Most stations in the Madrid basin record temperatures **warmer** than ERA5 predicts — confirming the UHI effect. Bias, MSE, and correlation per station:

| Metric | Value (mean across stations) |
|---|---|
| Bias (Station − ERA5) | **+1.3 to +1.6°C** (urban), negative (rural) |
| Correlation (ERA5 vs Station) | **0.987** |
| MSE | ~2.3°C² |

Three stations located in rural/mountain areas (Somosierra, Robledo de Chavela, Rozas de Puerto Real) show **negative bias** — ERA5 overestimates their temperature, consistent with the absence of an urban heat source.

### 2. The UHI effect is a summer phenomenon

| Season | Bias (°C) | MSE |
|---|---|---|
| Summer | **1.64** | 5.52 |
| Autumn | 1.29 | 5.02 |
| Spring | 1.34 | 4.95 |
| Winter | 1.38 | 5.77 |

The bias is highest in summer. Restricting the NDVI correlation analysis to summer dramatically strengthens the signal.

### 3. Strong negative correlation between NDVI and bias in summer

> *Less vegetation → more urban heating → larger ERA5 underestimation*

| Station example | Summer corr(bias, NDVI) |
|---|---|
| Station 3947 | **−0.988** |
| Station 27230 | **−0.968** |
| Station 27231 | **−0.948** |
| All stations (mean) | **−0.65** |

Altitude shows negligible influence on the discrepancy — confirming the UHI, not topography, as the primary driver.

---

## Correction Models

Four models were developed to correct ERA5-Land temperatures using station observations:

### Model 1 — OLS Linear Regression

**Features**: NDVI, Longitude, Latitude, Season (dummy variables)  
**Approach**: Fit a mean bias per (station, season) group, map back to daily predictions.

```python
model = sm.OLS(Y, X).fit()
Station_TX_corrected = ERA5_TX + predicted_mean_bias
```

### Model 2 — Random Forest

**Features**: `ERA5_TX`, `NDVI`, `Month_Sin`, `Month_Cos`  
**Training data**: 1,143,198 daily records from **900 Spanish stations** (2020–2023)  
**Validation**: leave-one-station-out cross-validation

```python
rf = RandomForestRegressor(n_estimators=100, max_depth=12)
rf.fit(X_train, y_train)
```

**Feature importances**: ERA5_TX dominates (~70%), followed by seasonality (Month_Sin/Cos), then NDVI.

### Model 3 — Neural Network (MLP, Keras)

**Architecture**: Dense(64) → BatchNorm → Dropout(0.1) → Dense(64) → BatchNorm → Dense(32) → Dense(1)  
**Training**: 1,143,198 points · batch_size=4096 · EarlyStopping · ReduceLROnPlateau  
**Final MAE on test set**: **1.74°C**  
**MAE on Madrid stations**: **1.65°C** (Bias ≈ −0.004°C, near-zero)

### Model 4 — SAX + Hybrid Neural Network

Introduces **Symbolic Aggregate approXimation (SAX)** to encode the 7-day temperature trend pattern as a categorical feature, fed into an Embedding layer.

**Architecture**:
```
Input (ERA5_TX) ─→ Dense(32)  ─┐
                                ├─→ Concat → Dense(64) → BN → Dropout → Dense(32) → Dense(16) → Dense(1)
Input (SAX code) → Embedding(5) ─┘
```
**Total parameters**: 5,502 (very lightweight)

---

## Results

### Correction model comparison on Madrid stations

| Model | MAE (°C) | Bias (°C) | Notes |
|---|---|---|---|
| ERA5 raw (no correction) | ~1.8 | +1.4 | Baseline |
| OLS Linear Regression | ~1.6 | ~0.0 | Interpretable, season-aware |
| Random Forest | ~1.5 | ~0.0 | Best spatial generalization |
| Neural Network (MLP) | **1.65** | −0.004 | Near-zero bias |
| SAX + Hybrid NN | ~1.7 | ~0.0 | Lightweight, trend-aware |

All four models significantly reduce the systematic bias. The Random Forest generalizes best across unseen stations (leave-one-out CV), while the MLP achieves the lowest bias.

---

## Installation

```bash
git clone https://github.com/mahmoud-bajjou/genhack-team31.git
cd genhack-team31
```

```bash
pip install numpy pandas geopandas xarray rioxarray rasterio matplotlib seaborn scipy
pip install scikit-learn statsmodels tensorflow tqdm
```

> **Data**: ERA5-Land files require a free account on the [Copernicus Climate Data Store](https://cds.climate.copernicus.eu/). ECA&D station data is freely available at [ecad.eu](https://www.ecad.eu/dailydata/predefinedseries.php). NDVI composites are available via [Sentinel Hub](https://www.sentinel-hub.com/).

### Recommended execution order

1. `final-read-era5-netcdf_v2.ipynb` — understand the data format
2. `week2-team31-sbyyaat.ipynb` — exploratory analysis & UHI findings
3. `week3-team31-sbyyaat.ipynb` — correction models
4. `weekfinal-team31-sbyyaat.ipynb` — final synthesis

---

## Team

**Team 31 — sbyyaat**  
Hackathon GenHack · Ecole Polytechnique

**Mahmoud Bajjou** — Télécom Paris, Data Science & AI  
📧 mahmoudbajjou5@gmail.com  
🔗 [GitHub](https://github.com/mahmoudbajjou)

---

<div align="center">

*GenHack 4-week Hackathon · Ecole Polytechnique · 2025*

</div>
