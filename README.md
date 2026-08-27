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

## Notebooks

| Notebook | Approach |
|---|---|
| [`Renang Data_NNLS Bootsrap.ipynb`](Renang%20Data_NNLS%20Bootsrap.ipynb) | NNLS Global Ensemble + Selective Guardrail Clipping |
| [`Renang Data_Residual LGBM.ipynb`](Renang%20Data_Residual%20LGBM.ipynb) | NNLS Ensemble + Residual LightGBM Stacker |

Both notebooks share the same data ingestion and EDA pipeline; the modelling
head is the key difference.

## Data Sources

> **Note:** Large data files are excluded from this repository via `.gitignore`.
> Download them separately and place them at the paths below.

| File | Path | Source |
|---|---|---|
| Training observations | `train.csv` | Competition page |
| Test observations | `test.csv` | Competition page |
| Sample submission | `sample_submission.csv` | Competition page |
| Environmental features | `data_pendukung/data_lingkungan.csv` | Competition page |
| Station coordinates | `data_pendukung/koordinat_pos.csv` | Competition page |
| HydroRIVERS shapefile (AU) | `data_pendukung/HydroRIVERS_v10_au_shp/` | [HydroSHEDS](https://www.hydrosheds.org/) |
| HydroRIVERS technical doc | `data_pendukung/HydroRIVERS_TechDoc_v10.pdf` | [HydroSHEDS](https://www.hydrosheds.org/) |

Expected working-directory structure:

```
ssds-uns-2026/
├── train.csv
├── test.csv
├── sample_submission.csv
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

## Modelling Overview

### Common Pipeline

1. **Data Ingestion** - Load `train.csv`, `test.csv`, `data_lingkungan.csv`,
   `koordinat_pos.csv`; parse ISO datetimes; enforce chronological order.
2. **Structural Diagnostics** - Validate 6-hourly observation grid completeness
   per station; flag missing timestamps.
3. **EDA & Anomaly Audit** - Per-station TMA statistics; environmental feature
   missingness; train/test distribution drift (KS test).
4. **Feature Engineering** - Temporal features, lag/rolling statistics,
   geospatial features from HydroRIVERS, environmental covariates.
5. **Base Model Training** - Multiple base regressors trained per station
   (or globally with station embeddings).
6. **NNLS Ensemble** - Non-Negative Least Squares optimisation of base-model
   weights to minimise global RMSE.

### NNLS Bootstrap (`Renang Data_NNLS Bootsrap.ipynb`)

Extends the NNLS ensemble with **bootstrapped weight estimation** and
**Selective Guardrail Clipping** post-processing to suppress physically
implausible predictions.

### Residual LGBM Stacker (`Renang Data_Residual LGBM.ipynb`)

Adds a **LightGBM residual correction** layer on top of the NNLS ensemble:
the LGBM model learns to predict the residual between the ensemble output and
ground truth, with the corrected prediction clipped to physically valid ranges.

## Documentation

- [`docs/Renang Data_Pakta Integritas.pdf`](docs/Renang%20Data_Pakta%20Integritas.pdf) - Competition integrity pledge (Pakta Integritas)

## License

See [LICENSE](LICENSE).
