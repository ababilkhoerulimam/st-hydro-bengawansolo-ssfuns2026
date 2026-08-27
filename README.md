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

The forecasting architecture is organized into an end-to-end multi-stage pipeline designed for spatio-temporal water level prediction:

1. **Data Ingestion and Hydrography**: Ingests 6-hourly observation grids across 30 monitoring stations alongside environmental covariates (rainfall, soil moisture, evaporation). The network is aligned with the HydroSHEDS HydroRIVERS hydrographic vector dataset to establish topological routing and spatial catchment boundaries.

2. **Spatio-Temporal Feature Engineering**: Constructs multi-scale temporal lags, rolling momentum statistics, and spatial upstream-downstream travel time flow delays. Incorporates diurnal solar cycles, monsoon seasonality, climate indices (El Nino ONI), and reservoir heuristics such as Wonogiri Dam release calibration.

3. **Tier 1 Base Model Suite**: Trains diverse gradient boosted decision tree regressors (XGBoost with MSE and Quantile loss formulations, LightGBM, CatBoost) alongside regularized linear baselines using two-fold chronological temporal cross-validation.

4. **Tier 2 Non-Negative Least Squares (NNLS) Ensemble**: Optimizes global ensemble weights subject to non-negativity constraints to prevent destructive interference and collinearity distortion across base model predictions. Uses bootstrap weight sampling for out-of-distribution stability.

5. **Tier 3 Residual LightGBM Stacker**: Models systematic non-linear residual errors between the ground truth and the linear NNLS ensemble output. A second-level LightGBM regressor learns residual corrections to produce the augmented forecast.

6. **Post-Processing Selective Guardrails**: Applies station-specific empirical boundary clipping derived from historical physical elevation limits to prevent unconstrained extrapolation during long-horizon test forecasting.

## Modelling Methodology

### 1. Spatial Catchment & Topological Flow Routing

The Bengawan Solo watershed is modeled as a directed acyclic graph (DAG) derived from the HydroSHEDS HydroRIVERS vector network:

* **Station Snapping**: Monitoring station geographic coordinates (`koordinat_pos.csv`) are snapped to the nearest river reach in `HydroRIVERS_v10_au.shp`.
* **Upstream Catchment Aggregation**: Cumulative upstream drainage areas and travel times are computed across reach segments.
* **Flow Propagation Lags**: Upstream precipitation and water volume surges are propagated to downstream gauges based on physical hydrographic distance and estimated flow velocity.

### 2. Feature Space Design

The feature pipeline produces structured representations across four major categories:

* **Autoregressive & Momentum Features**: Station-specific historical water level lags (t-6h, t-12h, t-18h, t-24h, t-48h), moving averages, rolling standard deviations, and exponential moving momentum.
* **Hydro-Meteorological Covariates**: Multi-depth soil moisture (`soil_moisture_0_7cm`, `soil_moisture_7_28cm`), evapotranspiration, rainfall volume, and upstream aggregated precipitation.
* **Temporal & Cyclical Encodings**: Hour-of-day and day-of-year cyclical trigonometric representations (sine and cosine encodings), monsoon seasonality indices, and El Nino Oceanic Nino Index (ONI) clipping.
* **Station Embeddings & Specific Calibrations**: Target-encoded station indicators, elevation differentials, and specialized parameters for controlled reservoirs (Wonogiri Dam release weights).

### 3. Multi-Tier Ensemble & Residual Stacking

The project evaluates two distinct ensemble architectures:

#### Approach A: NNLS Bootstrap Ensemble (`Renang Data_NNLS Bootsrap.ipynb`)

Combines predictions from diverse base regressors using constrained optimization:

$$
\min_{\mathbf{w} \ge 0} \|\mathbf{y} - \mathbf{A}\mathbf{w}\|_2^2 \quad \text{subject to} \quad \sum_{i} w_i = 1
$$

Where $\mathbf{A}$ is the matrix of out-of-fold base model predictions. Non-negativity constraints prevent destructive negative weighting caused by multicollinearity. Bootstrap resamplings of $\mathbf{A}$ are used to generate robust model weight distributions.

#### Approach B: Residual LightGBM Stacker (`Renang Data_Residual LGBM.ipynb`)

Extends the NNLS global ensemble by modeling residual prediction error:

$$
e = y_{\text{true}} - \hat{y}_{\text{NNLS}}
$$

A second-level LightGBM regressor is trained on the residual target using environmental covariates and interaction features. The final prediction combines the linear ensemble with the learned non-linear correction:

$$
\hat{y}_{\text{final}} = \hat{y}_{\text{NNLS}} + \hat{e}_{\text{LGBM}}
$$

### 4. Post-Processing: Selective Guardrails

To protect against catastrophic extrapolation during multi-step inference, an empirical guardrail mechanism clips final forecasts to physical domain bounds:

$$
\hat{y}_{\text{clipped}} = \text{clip}\left(\hat{y}_{\text{final}}, \; y_{\text{min}, s} - \Delta_s, \; y_{\text{max}, s} + \Delta_s\right)
$$

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
