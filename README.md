<div align="center">
  <img src="repo.png" alt="Logo" width="128" height="128">
  
  # st-hydro-bengawansolo-ssfuns2026
  **Spatio-Temporal Water Level (TMA) & Discharge Forecasting**
  
  <p align="center">
    <img src="https://img.shields.io/badge/Competition-SSDS_UNS_2026-blue?style=flat-square" alt="Competition">
    <img src="https://img.shields.io/badge/Language-Python_3-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python">
    <img src="https://img.shields.io/badge/Model-NNLS_Ensemble_&_LightGBM-4CAF50?style=flat-square" alt="Model">
    <img src="https://img.shields.io/badge/Status-Completed-success?style=flat-square" alt="Status">
  </p>
  
  <p align="center">
    Submission for the <b>SSDS UNS 2026</b> data science competition: predicting <b>Tinggi Muka Air (TMA)</b> (river water level, in m.dpl) and discharge for 30 monitoring stations across the <b>Bengawan Solo</b> watershed using spatio-temporal modeling and ensemble stacking.
  </p>
</div>

## Tech Stack

![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![NumPy](https://img.shields.io/badge/numpy-%23013243.svg?style=for-the-badge&logo=numpy&logoColor=white)
![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=for-the-badge&logo=pandas&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-%23F7931E.svg?style=for-the-badge&logo=scikit-learn&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-%23ffffff.svg?style=for-the-badge&logo=Matplotlib&logoColor=black)
![Git](https://img.shields.io/badge/git-%23F05033.svg?style=for-the-badge&logo=git&logoColor=white)

## System Architecture

```
+-----------------------------------------------------------------------------+
|                        1. DATA INGESTION & HYDROGRAPHY                      |
|  - 30 Monitoring Stations (6-hourly TMA grid)                                |
|  - Environmental covariates (rainfall, soil moisture, evaporation)          |
|  - HydroRIVERS topological routing graph & upstream catchment aggregation   |
+-----------------------------------------------------------------------------+
                                       |
                                       v
+-----------------------------------------------------------------------------+
|                     2. SPATIO-TEMPORAL FEATURE ENGINEERING                  |
|  - Autoregressive lags (t-6h, t-12h, t-24h, t-48h) & rolling statistics     |
|  - Spatial upstream-downstream travel time flow routing features            |
|  - Diurnal cyclical encodings & climate index calibration (El Nino index)   |
|  - Special station heuristics (Wonogiri Dam grid search, Gunungsari)        |
+-----------------------------------------------------------------------------+
                                       |
                                       v
+-----------------------------------------------------------------------------+
|                         3. TIER 1: BASE MODEL SUITE                         |
|  - Gradient Boosting Trees: XGBoost (MSE & Quantile losses), LightGBM,      |
|    CatBoost Regressors                                                      |
|  - Linear Baselines & Regularized Regressors                                |
|  - Two-Fold Chronological Temporal Cross-Validation                         |
+-----------------------------------------------------------------------------+
                                       |
                                       v
+-----------------------------------------------------------------------------+
|               4. TIER 2: NON-NEGATIVE LEAST SQUARES (NNLS) ENSEMBLE         |
|  - Solves min ||y - A w||^2 s.t. w >= 0, sum(w) = 1                         |
|  - Eliminates destructive collinearity across base models                   |
|  - Bootstrapped weight sampling for out-of-distribution stability           |
+-----------------------------------------------------------------------------+
                                       |
                                       v
+-----------------------------------------------------------------------------+
|                 5. TIER 3: RESIDUAL LIGHTGBM STACKER (EXP048)               |
|  - Residual target: e = y_true - y_pred_nnls                                |
|  - Second-level LightGBM learns systematic non-linear residual errors       |
|  - Final forecast: y_final = y_pred_nnls + e_pred                           |
+-----------------------------------------------------------------------------+
                                       |
                                       v
+-----------------------------------------------------------------------------+
|                   6. POST-PROCESSING: SELECTIVE GUARDRAIL                   |
|  - Empirical station-specific physical elevation boundary clipping          |
|  - [train_min - Delta_s, train_max + Delta_s] bounds to suppress anomalies  |
+-----------------------------------------------------------------------------+
```

## Modelling Methodology

### 1. Spatial Catchment & Topological Flow Routing

The Bengawan Solo watershed is modeled as a directed acyclic graph (DAG) derived from the HydroSHEDS HydroRIVERS vector network:

* **Station Snapping**: Monitoring station geographic coordinates (`koordinat_pos.csv`) are snapped to the nearest river reach in `HydroRIVERS_v10_au.shp`.
* **Upstream Catchment Aggregation**: Cumulative upstream drainage areas and travel times are computed across reach segments.
* **Flow Propagation Lags**: Upstream precipitation and water volume surges are propagated to downstream gauges based on physical hydrographic distance and estimated flow velocity.

### 2. Feature Space Design

The feature pipeline produces structured representations across four major categories:

* **Autoregressive & Momentum Features**: Station-specific historical water level lags ($t-6	ext{h}$, $t-12	ext{h}$, $t-18	ext{h}$, $t-24	ext{h}$, $t-48	ext{h}$), moving averages, rolling standard deviations, and exponential moving momentum.
* **Hydro-Meteorological Covariates**: Multi-depth soil moisture (`soil_moisture_0_7cm`, `soil_moisture_7_28cm`), evapotranspiration, rainfall volume, and upstream aggregated precipitation.
* **Temporal & Cyclical Encodings**: Hour-of-day and day-of-year cyclical trigonometric representations ($\sin / \cos$), monsoon seasonality indices, and El Nino Oceanic Nino Index (ONI) clipping.
* **Station Embeddings & Specific Calibrations**: Target-encoded station indicators, elevation differentials, and specialized parameters for controlled reservoirs (Wonogiri Dam release weights).

### 3. Multi-Tier Ensemble & Residual Stacking

The project evaluates two distinct ensemble architectures:

#### Approach A: NNLS Bootstrap Ensemble (`Renang Data_NNLS Bootsrap.ipynb`)
Combines predictions from diverse base regressors using constrained optimization:
$$\min_{\mathbf{w} \ge 0} \|\mathbf{y} - \mathbf{A}\mathbf{w}\|_2^2 \quad 	ext{subject to} \quad \sum_{i} w_i = 1$$
Where $\mathbf{A}$ is the matrix of out-of-fold base model predictions. Non-negativity constraints prevent destructive negative weighting caused by multicollinearity. Bootstrap resamplings of $\mathbf{A}$ are used to generate robust model weight distributions.

#### Approach B: Residual LightGBM Stacker (`Renang Data_Residual LGBM.ipynb`)
Extends the NNLS global ensemble by modeling residual prediction error:
$$e = y_{	ext{true}} - \hat{y}_{	ext{NNLS}}$$
A second-level LightGBM regressor is trained on the residual target using environmental covariates and interaction features. The final prediction combines the linear ensemble with the learned non-linear correction:
$$\hat{y}_{	ext{final}} = \hat{y}_{	ext{NNLS}} + \hat{e}_{	ext{LGBM}}$$

### 4. Post-Processing: Selective Guardrails

To protect against catastrophic extrapolation during multi-step inference, an empirical guardrail mechanism clips final forecasts to physical domain bounds:
$$\hat{y}_{	ext{clipped}} = 	ext{clip}\left(\hat{y}_{	ext{final}}, \;	ext{train\_min}_s - \Delta_s, \;	ext{train\_max}_s + \Delta_sight)$$
Where $\Delta_s$ is a station-specific safety margin proportional to historical seasonal variance.

## Notebooks

| Notebook | Focus | Key Method |
|---|---|---|
| [`Renang Data_NNLS Bootsrap.ipynb`](Renang%20Data_NNLS%20Bootsrap.ipynb) | Linear Ensemble Optimization | NNLS Global Ensemble + Bootstrapped Weights + Guardrail Clipping |
| [`Renang Data_Residual LGBM.ipynb`](Renang%20Data_Residual%20LGBM.ipynb) | Non-Linear Residual Stacking | NNLS Ensemble + Residual LightGBM Stacker + Guardrails |

## Data Sources

> **Note:** Large data files are excluded from this repository via `.gitignore`. Download them separately and place them at the paths below.

| File | Path | Source |
|---|---|---|
| Training observations | `train.csv` | Competition page |
| Test observations | `test.csv` | Competition page |
| Sample submission | `sample_submission.csv` | Competition page |
| Environmental features | `data_pendukung/data_lingkungan.csv` | Competition page |
| Station coordinates | `data_pendukung/koordinat_pos.csv` | Competition page |
| HydroRIVERS shapefile (AU) | `data_pendukung/HydroRIVERS_v10_au_shp/` | [HydroSHEDS](https://www.hydrosheds.org/) |
| HydroRIVERS technical doc | `data_pendukung/HydroRIVERS_TechDoc_v10.pdf` | [HydroSHEDS](https://www.hydrosheds.org/) |

Expected working directory structure:

```
st-hydro-bengawansolo-ssfuns2026/
├── train.csv
├── test.csv
├── sample_submission.csv
├── repo.png
├── requirements.txt
├── data_pendukung/
│   ├── data_lingkungan.csv
│   ├── koordinat_pos.csv
│   └── HydroRIVERS_v10_au_shp/
│       ├── HydroRIVERS_v10_au.shp
│       └── ...
├── Renang Data_NNLS Bootsrap.ipynb
├── Renang Data_Residual LGBM.ipynb
└── docs/
    └── Renang Data_Pakta Integritas.pdf
```

## Environment Setup

```bash
pip install -r requirements.txt
```

Notebooks were developed with **Python 3** and Jupyter.

## Documentation

- [`docs/Renang Data_Pakta Integritas.pdf`](docs/Renang%20Data_Pakta%20Integritas.pdf) - Competition integrity pledge (Pakta Integritas)

## License

See [LICENSE](LICENSE).
