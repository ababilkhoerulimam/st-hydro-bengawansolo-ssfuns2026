PROJECT STATE
Update ini setiap kali ada handoff atau ganti akun Claude.
Saat ganti akun: paste AGENTS.md dulu, lalu paste file ini, lalu ketik "lanjutkan dari [stage]"
META
struktur folder:
SSDS-UNS-2026/
├── agents/
│   ├── AGENTS_ababil.md
│   ├── AGENTS_jeremy.md
│   └── AGENTS_vierico.md
├── data_pendukung/
│   ├── HydroRIVERS_v10_au_shp/
│   │   ├── HydroRIVERS_v10_au.dbf
│   │   ├── HydroRIVERS_v10_au.prj
│   │   ├── HydroRIVERS_v10_au.sbn
│   │   ├── HydroRIVERS_v10_au.sbx
│   │   ├── HydroRIVERS_v10_au.shp
│   │   └── HydroRIVERS_v10_au.shx
│   ├── data_lingkungan.csv
│   ├── HydroRIVERS_TechDoc_v10.pdf
│   └── koordinat_pos.csv
├── .gitignore
├── edassds.ipynb
├── LICENSE
├── projectstate.md
├── README.md
├── sample_submission.csv
├── test.csv
├── tma_research_advisory.md
└── train.csv
Competition / Project : Prediksi Tinggi Muka Air (TMA) — BBWS Bengawan Solo
                        Sebelas Maret Statistics Data Science 2026
                        70% Private, 30% public (leaderboard)
Kaggle URL            : https://www.kaggle.com/competitions/sebelas-maret-statistics-data-science-2026/data
Deadline              : 25 Juli 2026 pukul 23:59 WIB
Metric                : RMSE (Root Mean Squared Error) — semakin kecil semakin baik
Submission budget     : 37 total | Kuota harian: 3 sub/hari (periode 11–25 Juli)
Submissions used today: 0 / 3
Last updated          : 14 Juli 2026 (post-Stage 9 Baseline exp001)
Updated by            : Ababil (Stage 9 complete, awaiting LB score)
CURRENT STATUS
Active phase          : FASE 4 IMPLEMENTATION & EXPERIMENTATION (Stage 9-10)
Last completed stage  : Stage 9 (Baseline Training exp001) ✓
Current stage         : Stage 10 (Iteration & Optimization) — pending
Next action           : Ababil → Submit exp001 to Kaggle, evaluate LB score, 
                        proceed to exp002 (log1p transform) or exp003 (Tier 2 features).
Blocker (if any)      : NONE — Baseline established, ready for iteration.
DATASET
[Dataset section tetap sama seperti sebelumnya]
EXPLORATION REPORT STATUS (Jeremy)
Jeremy stage saat ini : E8 DONE — Exploration Report SENT TO ABABIL
Post-FE Report (E9)   : BELUM — standby menunggu delegasi dari Ababil
EDA TAMBAHAN (7-11)   : DONE ✓ (13 Jul 2026 oleh Ababil)
Key Findings dari Jeremy (E1–E8) + EDA Tambahan 7–11
[Semua FINDING 0–22 dari E1-E11 & Stage 3 tetap sama]
Hipotesis Final E7 (ranked by expected impact) — UPDATED
[H1-H7 tetap sama]
Risk Flags dari Jeremy — UPDATED
[Flags 0-17 tetap sama]
BUSINESS BRIEF STATUS (Vierico)
Checkpoint B1 (Problem Brief)  : BELUM
Checkpoint B2 (EDA Commentary) : BELUM
Checkpoint B3 (Strategy Review): BELUM
Checkpoint B4 (Error Cost)     : BELUM
Checkpoint B5 (Explainability) : BELUM
Checkpoint B6 (Exec Summary)   : BELUM
Active veto dari Vierico:
- NONE
Business constraints yang sudah dikonfirmasi:
- [belum tersedia — tunggu B1]
FEATURE SET (Ababil) — FINALIZED POST-STAGE 8
Ababil stage saat ini : STAGE 10 (Iteration & Optimization)
FE status             : LOCKED & IMPLEMENTED
Leakage check         : COMPLETE (All groups passed Rule 12 Taxonomy)
Vierico FE review     : PENDING (proceeding with risk)
Final Dataset Shapes  : Train (84395, 39) | Test (21780, 36)

TIER 1 FEATURES (16 features, used in exp001):
   tma_lag_1, tma_lag_2, tma_lag_3     — per pos, masked di gap (NaN: 0.11%-0.19%)
   soil_moisture_7_28cm                — within-pos corr +0.492
   soil_moisture_0_7cm                 — within-pos corr +0.457
   rainfall_max_24h_mm                 — backward confirmed
   dew_point_c                         — within-pos corr +0.308
   humidity_pct                        — within-pos corr +0.252
   pos_encoded                         — target encoding nama_pos (fit from train only)
   month, hour                         — calendar features
   doy_sin, doy_cos                    — cyclical day of year
   month_sin, month_cos                — cyclical month
   pressure_detrended                  — per pos detrend (fit from train only)

TIER 2 FEATURES (9 features, for exp003 additive):
   soil_moisture_28_100cm              — redundant but monitored
   cloud_cover_pct                     — proxy solar
   wind_direction_deg                  -- topografi lokal
   mjo_amplitude                       — broadcast signal
   nino_34                             — forward-filled (phase shift warning)
   rolling_rain_48h, rolling_rain_72h  — aggregated from hourly rainfall
   dis_av_cms, ord_flow                — HydroRIVERS static topology

Outlier Resolution:
   🔴 Floodway Bridge C 47.0 mdpl (15 Jun 2025 18:00) — CONFIRMED SENSOR ERROR.
   Removed 1 row. Max TMA for Floodway is now ~4.7 mdpl.

Features yang di-drop (final):
- rainfall_openmeteo_mm (duplikat)
- solar_radiation_mj_m2 (60% test missing, better proxy available)
- built_surface_m2, landcover_* (statis, redundan)
- mjo_active (redundan to mjo_amplitude)
- temperature_c (redundan ke humidity, corr -0.884)
- soil_moisture_100_255cm (corr 0.140, decoupled)

STAGE 7 STATUS — Validation Design (Ababil)
   Status: ✅ COMPLETE
   Locked Strategy: Purged TimeSeriesSplit (expanding window)
   Parameters: 5 folds, purge 12 steps (72 hours), seed 42
   Split unit: Unique datetime (all 30 pos in same fold)
   Metrics: RMSE mean ± std across 5 folds + per-pos breakdown
   LB Risks identified: nino_34 phase shift, seasonal mismatch fold 5, exposure bias.

STAGE 8 STATUS — Feature Engineering & Outlier Resolution (Ababil)
   Status: ✅ COMPLETE
   Leakage Taxonomy: ALL PASS (Lag, Rolling, Calendar, Pressure, Target Enc, HydroRIVERS)
   Inference Structure: Autoregressive per timestep, clip to train range per pos.
   Outlier: Floodway Bridge C 47.0 mdpl removed (sensor error confirmed via rainfall check).
   Final Shapes: Train (84395, 39), Test (21780, 36).

STAGE 9 STATUS — Baseline Training (Ababil)
   Status: ✅ COMPLETE (exp001 executed)
   Model: Global LightGBM (Tier 1 features only)
   Inference: Autoregressive loop, seed lag from last 3 train values per pos, 
              clip predictions to [min_train, max_train] per pos to mitigate exposure bias.
   CV Results exp001:
     Fold 1 (2023-06 to 2023-11): RMSE 3.2344 | Best iter: 88
     Fold 2 (2023-11 to 2024-04): RMSE 2.6409 | Best iter: 100
     Fold 3 (2024-04 to 2024-10): RMSE 1.0298 | Best iter: 99
     Fold 4 (2024-10 to 2025-04): RMSE 1.0304 | Best iter: 146
     Fold 5 (2025-04 to 2025-09): RMSE 0.8810 | Best iter: 118
     Mean RMSE: 1.7633 | Std RMSE: 0.9786 | OOF RMSE: 1.9928
   Feature Importance (Top 5 gain):
     1. pos_encoded (5.74e8) — dominant identity signal
     2. tma_lag_1 (9.89e7) — strong autoregressive signal
     3. tma_lag_2 (5.79e7)
     4. tma_lag_3 (9.30e6)
     5. soil_moisture_0_7cm (1.73e5) — strongest exogenous
   Submission Sanity Check:
     Shape: (21780, 2) | NaN: 0 | Negative: 0
     Mean: 55.91 | Std: 45.51 | Min: 1.06 | Max: 143.27
   Observation: Fold 1 & 2 CV RMSE significantly higher than Fold 3-5. 
                Indicates model struggles with earlier temporal periods or specific seasonal regimes.

MODEL CONFIGURATION (LightGBM)
   objective          : regression
   metric             : rmse
   learning_rate      : 0.05
   num_leaves         : 127
   min_child_samples  : 20
   feature_fraction   : 0.8
   bagging_fraction   : 0.8
   bagging_freq       : 1
   lambda_l1          : 0.1
   lambda_l2          : 0.1
   verbose            : -1
   n_jobs             : -1
   seed               : 42
   num_boost_round    : 2000
   early_stopping     : 50 rounds

EXPERIMENT LOG SUMMARY
Anchor model (Slot 1) : exp001 (Global LightGBM, Tier 1, Autoregressive, Clip)
Anchor model (Slot 2) : [belum ada]
Best CV so far        : exp001 | OOF RMSE 1.9928 | Mean CV 1.7633
Best LB so far        : [belum submit]
VALIDATION STRATEGY (locked at Stage 7, executed Stage 9)
CV method     : Purged TimeSeriesSplit (expanding window)
n_folds       : 5
purge         : 12 steps (72 hours)
Seed          : 42
Split unit    : Unique datetime (bukan row index)
Locked        : YES
⚠️ Gap 4-28 Feb 2025 TIDAK BOLEH jadi fold boundary (jatuh di Fold 3 val, acceptable)
⚠️ Target encoding & pressure detrend WAJIB fit dari train fold only
⚠️ Expect CV lebih optimistic dari LB karena exposure bias & nino_34 phase shift
PENDING ACTIONS — UPDATED
[HIGH] Ababil   — Submit exp001 to Kaggle, record Public LB score.
[HIGH] Ababil   — Stage 10: Train exp002 (Tier 1 + log1p transform target) to address high-skew pos.
[HIGH] Ababil   — Train exp003 (Tier 1 + Tier 2 features) to test additive value.
[MED]  Ababil   — Investigate Fold 1 & 2 high RMSE (3.23, 2.64) vs Fold 3-5 (~1.0).
[MED]  Ababil   — Investigasi Kali Anyar-Kreteg Abang: MAE tinggi di kemarau (anomali)
[MED]  Vierico  — Review proposed feature set & exp001 results.
[MED]  Vierico  — Assess business validity: HydroRIVERS features, cross-lag features
[LOW]  Jeremy   — Stage E9: Post-FE Business Plotting (delegasi ketika FE done)
CONTEXT RESET PROTOCOL
[Tetap sama seperti sebelumnya]
CATATAN BEBAS — UPDATED
12 Juli 2026 (Jeremy) : E8 Exploration Report SENT.
13 Juli 2026 (Ababil) : EDA Tambahan 7-11 complete.
13 Juli 2026 (Ababil) : Stage 3 (Technical EDA) Step 1-2 COMPLETE.
14 Juli 2026 (Ababil) : Stage 3 Step 3-4 COMPLETE. Locked: Autoregressive lag, Detrend pressure.
14 Juli 2026 (Ababil) : Stage 4 (Competitive Research) COMPLETE. Anchor: Global LightGBM + nama_pos.
14 Juli 2026 (Ababil) : Stage 5 (Strategy Discussion) COMPLETE. Purged TSS, log1p, Budget 4/20/12.
14 Juli 2026 (Ababil) : Stage 6 (Hypothesis Generation) COMPLETE. Generated H1-H10.
14 Juli 2026 (Ababil) : Stage 7 (Validation Design) COMPLETE. 
                        Locked Purged TimeSeriesSplit (5 folds, purge 12, seed 42).
14 Juli 2026 (Ababil) : Stage 8 (Feature Engineering) COMPLETE. 
                        All leakage checks passed. 
                        Floodway Bridge C outlier 47.0 mdpl confirmed sensor error & removed.
                        Final shapes: Train (84395, 39), Test (21780, 36).
14 Juli 2026 (Ababil) : Stage 9 (Baseline Training) COMPLETE.
                        exp001: Global LightGBM, Tier 1 features, Autoregressive inference.
                        CV Mean RMSE: 1.7633 | OOF RMSE: 1.9928.
                        Top features: pos_encoded, tma_lag_1/2/3, soil_moisture_0_7cm.
                        Submission sanity check passed (0 NaN, 0 Negative).
                        Next: Submit to Kaggle, proceed to exp002 (log1p) & exp003 (Tier 2).