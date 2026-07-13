Oke, ini full rewrite `project_state.md` berdasarkan semua informasi dari Cell 0–37:

---

# PROJECT STATE

## META

```
Competition       : Sebelas Maret Statistics Data Science 2026
Host              : Sebelas Maret Statistics Fair
Kaggle URL        : [link Kaggle]
Deadline          : 13 days to go (dari 2026-07-13)
Metric            : RMSE (Root Mean Squared Error) — CONFIRMED dari competition page
Submission budget : 36 remaining / total unknown
Submissions used  : 0 / 3 today
Last updated      : 2026-07-13
Updated by        : Jeremy
```

---

## CURRENT STATUS

```
Active phase          : FASE 1 EDA — hampir selesai
Last completed stage  : Cell 37 (env merge ke train/test, missingness check)
Next action           : Cell 38+ — feature engineering (lag tma, upstream features, reindex)
Blocker               : NONE
```

---

## DATASET

```
Train file    : train.csv — 84,395 rows × 4 cols (post Cell 8: 1 outlier removed)
Test file     : test.csv — 21,780 rows × 1 col (id = "datetime - nama_pos")
Env file      : data_pendukung/data_lingkungan.csv — 888,480 rows × 28 cols (hourly)
Coords file   : data_pendukung/koordinat_pos.csv — 30 stations, lat/lon
Hydro file    : data_pendukung/HydroRIVERS_v10_au_shp/HydroRIVERS_v10_au.shp — 836,472 segments
Target column : tma_mdpl (water level, meters above sea level)
Task type     : Regression — RMSE metric
Time-based    : YES — 6-hourly (jam 06, 12, 18 WIB)
Train cutoff  : TRAIN_TEST_CUTOFF = 2025-09-19
Test range    : 2025-09-19 06:00 — 2026-05-18 18:00 (726 obs per station, NO GAPS)
Train range   : 2023-01-01 — 2025-09-18 18:00
```

---

## NOTEBOOK STRUCTURE

### Cell 0 — Library Imports
Libraries: numpy, pandas, itertools.combinations, matplotlib.pyplot, scipy.stats (ks_2samp, ttest_ind), sklearn (Ridge, Pipeline, StandardScaler, mean_squared_error), geopandas.
Config: `pd.set_option("display.width", 120)`

---

### Cell 1 — Global Configuration
```
TRAIN_PATH         : train.csv
TEST_PATH          : test.csv
ENV_PATH           : data_pendukung/data_lingkungan.csv
COORDS_PATH        : data_pendukung/koordinat_pos.csv
SAMPLE_SUB_PATH    : sample_submission.csv
TARGET_HOURS       : [6, 12, 18]
TRAIN_TEST_CUTOFF  : pd.Timestamp("2025-09-19")
```

---

### Cell 2 — Load Raw Datasets
Variables created: `df` (84,396×4), `test` (21,780×1), `env` (888,480×28).
Transformations: `datetime_parsed` added via `pd.to_datetime`.

---

### Cell 3 — Structural Inspection
Findings: train 0 missing, 30 stations, range 2023-01-01–2025-09-18 18:00. Env missing: soil_moisture×4, pressure×2, rmm/mjo×6 (720 rows each), nino_34 (12,960 rows). No duplicate (datetime, nama_pos) pairs.

---

### Cell 4 — Missing Timestamps per Station
Function: `missing_timestamps(sub, hours)` — detects missing 6-hourly observations.
Findings: Gunungsari 80 missing, Floodway Bridge C 599 missing (Jun–Dec 2023 + Feb 2025), Bojonegoro 288 missing. Station row counts: 1,126–2,976.

---

### Cell 5 — February 2025 Outage Confirmation
Findings: All 30 stations drop to 9–11 observations (expected 84) in Feb 2025. Daily count collapses near-zero. Systemic sensor/logging outage confirmed.

---

### Cell 6 — Target Distribution dan Per-Station Stats
Findings: Global mean 56.48, std 46.77, skew 0.343, kurtosis -1.236. Per-station mean range: 1.12–143.56. Invalid readings (tma_mdpl ≤ 0): 5 rows — **PENDING investigation**.

---

### Cell 7 — Skewness Visualization (Pre-cleaning)
Plot: horizontal barh per station. Superseded by Cell 36 post-cleaning version.

---

### Cell 8 — Floodway Bridge C Outlier Removal
Spike 47.0 at 2025-06-15 18:00 — single-point sensor error confirmed, removed. `df` → 84,395 rows. Max post-removal: 8.00.

---

### Cell 9 — Environmental Missing Value Patterns
Findings: soil_moisture×4, pressure×2, rmm/mjo: missing only 2026-05-18 (720 rows). nino_34: missing entire May 2026 (12,960 rows). solar_radiation = -999 sentinel (100,080 rows, Dec 2025–May 2026). rainfall_mm == rainfall_openmeteo_mm always (redundant).

---

### Cell 10 — Environmental Feature Range dan Sanity Checks
Findings: rainfall_mm 71.4% zero. All range checks PASS. Pressure diff (MSL - surface): 0.2–17.0 hPa — valid elevation proxy.

---

### Cell 11 — MJO dan Nino34 Granularity
Findings: MJO daily (constant within day), nino_34 monthly, both identical across all 30 stations. Global indices, not station-specific.
Variable created: `env["date_only"]`.

---

### Cell 12 — Global vs Within-Station Correlation (Simpson's Paradox)
Variables: `env_matched`, `merged` (84,395×31, 0 dropped), `global_corr`, `within_pos_corr`.
Key findings (global → within-station):
- surface_pressure_hpa: -0.947 → -0.223 (elevation confound)
- soil_moisture_7_28cm: +0.124 → +0.493
- soil_moisture_0_7cm: +0.106 → +0.458
- soil_moisture_28_100cm: +0.127 → +0.427
- rainfall_max_24h_mm: -0.025 → +0.318
- dew_point_c: -0.122 → +0.308
- mjo_active: +0.048 (weakest)

---

### Cell 13 — Feature Redundancy Check
High correlations: soil_0_7 ↔ soil_7_28 (0.913), humidity ↔ temperature (-0.884), soil_7_28 ↔ soil_28_100 (0.822), mjo_amplitude ↔ mjo_active (0.732). Deep layer (100_255cm) most independent.

---

### Cell 14 — Autocorrelation dan Observation Spacing
Two clusters:
- **High AC (lag1 > 0.6):** Wonogiri Dam (0.998), Colo Weir (0.987), Karanggeneng (0.981), Boboh Kali Lamong (0.978), Cepu (0.976), Sumberrejo (0.974), Babat (0.964), Bengkelolor (0.955), Serenan (0.953), Floodway Bridge C (0.948), Gunungsari (0.904), Brangkal (0.895), Sekayu (0.878), Badegan (0.851), Ketonggo (0.815), Lorog (0.813), Ngadipiro (0.804), Ngrembang (0.723), Arjowinangun (0.624), Kali Pepe Tugu Boto (0.611).
- **Low AC (lag1 < 0.25):** Kali Pepe PTPN (0.009), Jarum (0.024), Kali Anyar (0.036), Peren (0.047), Karangnongko (0.078), Napel (0.094), Bojonegoro (0.133), Kajangan (0.225), Kedungupit (0.233).

Largest gap: Ngadipiro 612h (Feb 2025). Shift(1) without reindexing = corrupt lags.

---

### Cell 15 — Seasonal dan Yearly Patterns
Variables: `df["month"]`, `df["year"]`, `df["tma_normalized"]`.
Peak wet: Feb (+1.06 normalized). Nadir dry: Oct (-1.09). Feb 2025 near zero = outage artifact. Notable trends: Wonogiri Dam +0.79, Boboh Kali Lamong +0.68, Kali Anyar -1.64.

---

### Cell 16 — Rainfall-to-TMA Lag Structure
Optimal lag (mean across stations): rainfall_mm lag 1 (6h, corr 0.1293), rainfall_max_24h_mm lag 1 (6h, corr 0.3285), soil_moisture_7_28cm lag 0 (corr 0.4930), soil_moisture_0_7cm lag 1 (corr 0.4606).

Station-specific optimal lag for rainfall_max_24h_mm:
- Lag 0: Badegan, Jarum, Lorog, Kali Pepe Tugu Boto
- Lag 1 (6h): Jurug, Ketonggo, Gunungsari, Arjowinangun, Ngadipiro, Peren, Serenan, Sekayu
- Lag 2 (12h): Kali Anyar, Kajangan, Bengkelolor, Brangkal, Kedungupit
- Lag 3–4 (18–24h): Napel, Babat, Karangnongko, Cepu, Boboh Kali Lamong, Colo Weir, Kali Pepe PTPN, Sumberrejo
- Lag 6–8 (36–48h): Karanggeneng, Ngrembang, Floodway Bridge C, Bojonegoro
- Lag 10 (60h): Wonogiri Dam (reservoir damping)

---

### Cell 17 — Cross-Station TMA Correlation dan Upstream-Downstream Lag
Top correlated pairs: Jurug ↔ Serenan (0.946), Karanggeneng ↔ Sumberrejo (0.936), Babat ↔ Sumberrejo (0.895), Cepu ↔ Sumberrejo (0.880), Bengkelolor ↔ Boboh Kali Lamong (0.874).
Cross-station lag: Cepu → Sumberrejo lag 2 (0.927), Cepu → Karanggeneng lag 3 (0.914).

---

### Cell 18 — Geographic Distances dan Elevation Proxy
Function: `haversine()` → km. Variable: `geo`, `dist_df` (435 pairs).
Pattern: South = upstream, higher elevation, longer optimal lag. North = downstream, low elevation, high AC.
Closest pairs: Kali Anyar ↔ Kali Pepe PTPN (1.48 km), Kali Anyar ↔ Jurug (3.33 km).

---

### Cell 19 — Spike vs Sustained Flood Classification
Function: `run_length_encoding(bool_series)`. Threshold: q99 per station.
- Kali Anyar: **100% isolated** — sensor noise, NOT real floods
- Peren: 88.9% isolated, max run 72h (genuine prolonged exists)
- Napel, Kedungupit: 12–14% isolated — **genuine sustained floods** (36–54h)
- Bojonegoro: 30% isolated, max run 36h

---

### Cell 20 — TMA Level Shift Around Feb-2025 Gap
19/30 stations have |jump| > 1 mdpl. Largest: Ketonggo +5.90, Floodway Bridge C +5.22, Napel +5.10, Sumberrejo +5.03. Genuine seasonal transition, not artifact. Lag features crossing gap produce anomalous deltas.

---

### Cell 21 — Floodway Bridge C Pre vs Post-2023 Outage
Mean nearly identical (3.837 vs 3.760), t-test p=0.149 (not significant). KS-test D=0.185, p=5e-12 — distribution shape changed. Possible sensor recalibration during 7-month gap.

---

### Cell 22 — Wonogiri Dam Reservoir Operation
95.81% steps |delta| ≤ 0.5 mdpl. Big steps occur at mean rainfall 0.30 mm vs small steps 2.30 mm — **level inversely correlated with rainfall**. Confirmed manual operator control. Rainfall features irrelevant for this station.

---

### Cell 23 — Train vs Test Distribution Drift (KS Test)
env_train: 714,240 rows. env_test: 174,240 rows.
Strong drift (KS > 0.1): nino_34 (0.497, El Niño→La Niña), soil_moisture_7_28cm (0.299), soil_moisture_0_7cm (0.269), dew_point_c (0.219), soil_moisture_28_100cm (0.222), rainfall_max_24h_mm (0.199), rmm2 (0.179), cloud_cover_pct (0.175), pressure_msl_hpa (0.168), mjo_phase (0.161), wind_direction_deg (0.119), rainfall_mm (0.116), humidity_pct (0.110), soil_moisture_100_255cm (0.127).

---

### Cell 24 — Seasonal Confounding Check
Test overrepresents wet months (Oct–Apr +3 to +6.6%), entirely missing Jun–Aug (-9.4%). Most drift = seasonal imbalance. Exception: nino_34 ENSO shift is genuine multi-year climate change.

---

### Cell 25 — Missing Value Classification (MCAR/MAR/MNAR)
- soil_moisture×4, pressure×2, rmm/mjo×6: MCAR — only 2026-05-18, safe forward-fill
- nino_34: MCAR — May 2026 not yet available, safe impute with April 2026 value
- solar_radiation_mj_m2: **MNAR** — 0% missing in train, 57.4% missing in test. DROP or encode -999 as flag

---

### Cell 26 — Per-Station Ridge Baseline vs Naive Mean
Features: rainfall_max_24h_mm, humidity_pct, dew_point_c, soil_moisture_7_28cm, soil_moisture_0_7cm, cloud_cover_pct, temperature_c, hour, month. 80/20 chronological split.
Mean naive RMSE: 1.0195 | Mean ridge RMSE: 0.8724 | Mean improvement: 6.9%.
Stations where ridge worse than naive: Kali Pepe PTPN (-192%), Kali Anyar (-70%), Gunungsari (-45%), Colo Weir (-18%), Ngrembang (-8.7%).
**Conclusion: lag tma features are essential — env alone insufficient.**

---

### Cell 27 — Baseline Residuals by Month and Hour
March = worst month (MAE 0.71–0.76, peak flood). Sep = best (MAE 0.42–0.45, dry). Hour 18 consistently slightly worse (afternoon convective rainfall harder to predict).

---

### Cell 28 — Load HydroRIVERS Shapefile
Variable: `rivers` (836,472 × 15 cols, EPSG:4326). Key columns: HYRIV_ID, NEXT_DOWN, MAIN_RIV, ORD_FLOW, ORD_STRA, DIS_AV_CMS, UPLAND_SKM, DIST_DN_KM, LENGTH_KM.

---

### Cell 29 — Column Reference and Station Coverage
UPLAND_SKM and DIST_DN_KM = best downstream position indicators. All 30 stations within shapefile extent.

---

### Cell 30 — Snap Stations to Nearest River Segment
Variable: `river_pos` (30 rows). All stations within 1.25 km of nearest segment. UPLAND_SKM ordering consistent with optimal rainfall lag ordering (Cell 16) — confirms river routing time as mechanism.

Station groups by MAIN_RIV:
- 50387677: main Bengawan Solo stem (largest group)
- 50392385: Kali Lamong (Boboh Kali Lamong, Bengkelolor)
- 50419297: Pacitan system (Gunungsari, Arjowinangun)
- 50419937: Lorog (isolated)

---

### Cell 31 — Topological vs Empirical Ordering Validation
UPLAND_SKM ordering confirmed consistent with: mean TMA elevation, empirical optimal lag, cross-station correlation structure. 9/10 top correlated pairs share same MAIN_RIV. Upstream-downstream direction correct for directional pairs.

Notable anomaly: Floodway Bridge C — UPLAND_SKM 8.1 km² (small) but DIST_DN_KM 61.9 km (also small). Artificial canal, not main river — ordering not applicable.

Full UPLAND_SKM ordering (upstream → downstream):
```
Colo Weir (3.6) → Kali Pepe PTPN (7.0) → Floodway Bridge C (8.1*) → Jurug (11.3)
→ Ketonggo (21.5) → Kali Pepe Tugu Boto (24.9) → Peren (30.8) → Sumberrejo (31.1*)
→ Lorog (47.6) → Ngrembang (138.8) → Badegan (276.7) → Ngadipiro (377.5)
→ Bengkelolor (382.4) → Kali Anyar (426.7) → Gunungsari (593.4)
→ Boboh Kali Lamong (617.2) → Jarum (638.1) → Arjowinangun (642.9)
→ Sekayu (800.9) → Brangkal (801.6) → Wonogiri Dam (1344.7)
→ Serenan (2477.8) → Kedungupit (4585.4) → Kajangan (5492.4)
→ Napel (9838.8) → Karangnongko (10032.6) → Cepu (10926.8)
→ Bojonegoro (12778.1) → Babat (14342.2) → Karanggeneng (15087.4)
*anomalous positions
```

---

### Cell 32 — Validate sample_submission.csv
Shape: 21,780 × 2. Columns: `id` (str), `tma_mdpl` (int64, placeholder 0). ID format: `"YYYY-MM-DD HH:MM:SS - nama_pos"`. Row count, order, and station names match test.csv exactly.

---

### Cell 33 — Test Period Exploration
Test range: 2025-09-19 06:00 — 2026-05-18 18:00. All 30 stations: exactly 726 observations each, **zero gaps** in test. Test period = wet season dominant (Sep–May), Jun–Aug entirely absent. Test rainfall mean 0.367 mm/h vs train 0.256 mm/h — higher due to wet season overrepresentation.

Total rainfall per station (train vs test — note: train ~32 months, test ~8 months):
- Lowest test rainfall: Badegan (1642 mm), Kajangan (1680 mm), Sekayu (1731 mm)
- Highest test rainfall: Jurug (2452 mm), Serenan (2402 mm), Jarum (2402 mm)

---

### Cell 34 — Rainfall × Soil Moisture Interaction
Method: stratify each station by median soil_moisture_7_28cm, compute rainfall-TMA corr separately for wet vs dry soil. `diff = corr_wet - corr_dry`.

Two clusters:
- **Positive interaction (diff > 0.2) — rainfall more effective on wet soil:** Peren (+0.326), Kedungupit (+0.315), Floodway Bridge C (+0.276), Karangnongko (+0.250), Kajangan (+0.241), Karanggeneng (+0.234), Sumberrejo (+0.234). → Feature `rainfall × soil_moisture` beneficial
- **Negative interaction (diff < -0.1) — rainfall more correlated on dry soil:** Napel (-0.256), Jarum (-0.194), Jurug (-0.192), Sekayu (-0.178), Serenan (-0.128), Brangkal (-0.088). → These stations dominated by upstream flow, not local rainfall; interaction feature may add noise
- Wonogiri Dam: negative corr in both conditions (-0.113 wet, -0.173 dry) — manual operation confirmed

---

### Cell 35 — String Matching Validation
All 30 stations match exactly across train, env, coords, and test. Zero mismatches, zero whitespace/case issues. Merge operations safe with raw `nama_pos` strings.

---

### Cell 36 — Post-cleaning Skewness Re-check
Floodway Bridge C: 13.29 → 1.03 after Cell 8 removal — effective. All other 29 stations unchanged.

Post-cleaning skewness by station (sorted ascending):
```
Jurug: -22.5 | Kali Pepe PTPN: -14.1 | Colo Weir: -3.2 | Ketonggo: -1.8
Wonogiri Dam: -0.5 | Bengkelolor: 0.5 | Boboh Kali Lamong: 0.8
Karanggeneng: 0.9 | Floodway Bridge C: 1.0 | Badegan: 1.2 | Babat: 1.2
Cepu: 1.4 | Gunungsari: 1.4 | Arjowinangun: 1.6 | Sumberrejo: 1.6
Lorog: 1.8 | Serenan: 1.9 | Brangkal: 2.3 | Ngadipiro: 2.3
Sekayu: 2.5 | Kali Pepe Tugu Boto: 3.9 | Ngrembang: 5.4
Karangnongko: 26.8 | Bojonegoro: 29.1 | Jarum: 32.9
Peren: 34.3 | Kedungupit: 34.6 | Kajangan: 35.2
Napel: 46.4 | Kali Anyar: 51.5
```

High positive skew stations — two types:
- Kali Anyar (51.5): sensor noise — winsorize/mask justified
- Napel, Kajangan, Kedungupit, Peren, Jarum, Bojonegoro, Karangnongko: genuine sustained floods — use robust loss (MAE/Huber) instead of winsorize

---

### Cell 37 — Env Merge to Train/Test + Missingness Audit
Preprocessing applied:
- `nino_34` forward-filled per station (handle May 2026 MCAR gap)
- Dropped: `rainfall_openmeteo_mm`, `solar_radiation_mj_m2`, `built_surface_m2`, `landcover_class`, `landcover_name`, `mjo_active`
- Added: `rolling_rain_48h` (48-hour rolling sum), `rolling_rain_72h` (72-hour rolling sum) — computed on hourly env before filtering
- Filter to TARGET_HOURS before merge

ENV_FEATURES retained (21 cols): datetime, nama_pos, rainfall_mm, humidity_pct, wind_direction_deg, dew_point_c, cloud_cover_pct, temperature_c, wind_speed_kmh, rainfall_max_24h_mm, soil_moisture_0_7cm, soil_moisture_7_28cm, soil_moisture_28_100cm, soil_moisture_100_255cm, surface_pressure_hpa, pressure_msl_hpa, rmm1, rmm2, mjo_phase, mjo_amplitude, nino_34, rolling_rain_48h, rolling_rain_72h.

Post-merge missingness:
- Train: 0 missing — clean
- Test: 90 missing rows across 10 cols (soil_moisture×4, pressure×2, rmm1, rmm2, mjo_phase, mjo_amplitude) — all from 2026-05-18 only (3 rows × 30 stations). Average 0.2% per station. **0 stations with >20% missingness.** Handle with forward-fill.

---

## STATION TAXONOMY

| Type | Stations | Key Characteristics |
|---|---|---|
| Reservoir/Dam controlled | Wonogiri Dam, Ngadipiro, Ngrembang, Colo Weir, Badegan | Very high AC (>0.7), low std, operator-driven not rainfall |
| Stable downstream | Serenan, Bengkelolor, Cepu, Babat, Sumberrejo, Karanggeneng, Boboh Kali Lamong, Sekayu, Lorog, Brangkal, Ketonggo | High AC (>0.6), follows rainfall with lag 1–4 steps |
| Flashy flood | Kali Anyar, Jarum, Peren, Napel, Kajangan, Kedungupit, Karangnongko, Bojonegoro, Kali Pepe PTPN | Low AC (<0.25), high skew, extreme events, lag tma ineffective |
| Anomalous negative skew | Kali Pepe PTPN, Jurug | Extreme negative skew, Ridge worse than naive, peculiar dynamics — needs investigation |
| Sparse data | Gunungsari | Only 2024–2025, Ridge worse than naive (-45%) |
| Sensor noise | Kali Anyar - Kreteg Abang | 100% isolated extreme spikes (skew 51.5), winsorize justified |
| Distribution shift post-outage | Floodway Bridge C | 7-month 2023 gap, KS-confirmed distribution change, artificial canal |
| Manual operation | Wonogiri Dam | Level inversely correlated with rainfall — autoregressive only |

---

## RIVER SYSTEM STRUCTURE

Main system: MAIN_RIV 50387677 = **Bengawan Solo** (largest group, ~21 stations).

Upstream → downstream (Bengawan Solo stem, by UPLAND_SKM):
```
Colo Weir → Kali Pepe PTPN → Jurug → Ketonggo → Kali Pepe Tugu Boto
→ Peren → Ngrembang → Badegan → Ngadipiro → Kali Anyar → Jarum
→ Sekayu → Brangkal → Wonogiri Dam → Serenan → Kedungupit → Kajangan
→ Napel → Karangnongko → Cepu → Bojonegoro → Babat → Karanggeneng
```

Separate systems:
- Kali Lamong (MAIN_RIV 50392385): Bengkelolor → Boboh Kali Lamong
- Pacitan (MAIN_RIV 50419297): Gunungsari → Arjowinangun
- Isolated: Lorog (50419937), Sumberrejo, Floodway Bridge C, Ketonggo, Ngrembang, Ngadipiro, Badegan

---

## FEATURE ENGINEERING PRIORITIES

### Critical — Must Have
- **Reindex** setiap stasiun ke 6-hourly grid lengkap sebelum lag construction (Feb 2025 gap + station gaps)
- **Station identity** (one-hot atau embedding) — MANDATORY untuk avoid Simpson's Paradox
- **Lag tma_mdpl**: lag 1 (6h) universal. Lag 2–5 untuk high-AC stations
- **Month feature** — strong seasonal signal (Oct nadir, Feb peak)
- **Upstream station TMA** sebagai lagged feature: Cepu → Sumberrejo (lag 2), Cepu → Karanggeneng (lag 3), Jurug → Serenan (lag 1), Bengkelolor → Boboh Kali Lamong (lag 1)
- **rainfall_max_24h_mm** dengan station-specific optimal lag (range 0–10 steps, lihat Cell 16)
- **UPLAND_SKM** sebagai static station feature

### High Value
- soil_moisture_7_28cm (lag 0–1, within-station corr 0.493)
- soil_moisture_0_7cm (lag 1, corr 0.458)
- rolling_rain_48h, rolling_rain_72h (engineered in Cell 37)
- Binary rain flag (rainfall_mm > 0)
- Pressure differential (MSL - surface) sebagai static elevation feature
- dew_point_c (within-station corr 0.308)
- MJO phase (daily), nino_34 (monthly — note ENSO regime shift)
- DIST_DN_KM, DIS_AV_CMS sebagai static river position features
- MAIN_RIV group sebagai categorical
- **rainfall × soil_moisture interaction** — terutama untuk: Peren, Kedungupit, Floodway Bridge C, Karangnongko, Kajangan, Karanggeneng, Sumberrejo

### Drop / Handle
- rainfall_openmeteo_mm — identik dengan rainfall_mm (DROPPED Cell 37)
- mjo_active — redundant dengan mjo_amplitude (DROPPED Cell 37)
- solar_radiation_mj_m2 — MNAR, 57.4% missing di test (DROPPED Cell 37)
- built_surface_m2, landcover_class, landcover_name (DROPPED Cell 37)
- raw surface_pressure_hpa tanpa station context — spurious global correlation

### Special Handling per Station
- **Wonogiri Dam**: drop rainfall features, autoregressive only
- **Kali Anyar**: winsorize extreme values sebelum training (100% sensor noise)
- **Kali Pepe PTPN**: investigasi lebih dalam — peculiar dynamics, Ridge -192% vs naive
- **Jurug**: investigasi negative skew dynamics
- **Gunungsari**: short history (2024–2025 only), handle carefully
- **Flashy flood stations** (Napel, Kajangan, Kedungupit dll): robust loss (Huber/MAE), jangan winsorize karena extreme values are genuine

---

## VALIDATED HYPOTHESES

```
H1:  Feb 2025 systemic outage — CONFIRMED (Cell 5)
H2:  Floodway Bridge C spike 47.0 = sensor error — CONFIRMED, REMOVED (Cell 8)
H3:  Pressure differential (MSL - surface) = valid elevation proxy — CONFIRMED (Cell 10)
H4:  rainfall_mm == rainfall_openmeteo_mm — CONFIRMED (Cell 9)
H5:  MJO daily, nino_34 monthly, identical across all stations — CONFIRMED (Cell 11)
H6:  surface_pressure global corr = elevation confound (Simpson's Paradox) — CONFIRMED (Cell 12)
H7:  Kali Anyar extreme values = sensor noise — CONFIRMED (Cell 19, 100% isolated)
H8:  Feb 2025 level shift = genuine seasonal transition — CONFIRMED (Cell 20)
H9:  River routing time explains optimal lag per station — CONFIRMED (Cell 16 + 18 + 30)
H10: Cepu upstream of Sumberrejo (lag 2) and Karanggeneng (lag 3) — CONFIRMED (Cell 17 + 30 + 31)
H11: Floodway Bridge C post-2023 outage has different distribution — CONFIRMED (Cell 21, KS p=5e-12)
H12: Wonogiri Dam level driven by operator, not rainfall — CONFIRMED (Cell 22)
H13: Most train-test drift is seasonal confounding, not concept drift — CONFIRMED (Cell 24)
H14: nino_34 ENSO regime shift (El Niño train → La Niña test) is genuine — CONFIRMED (Cell 23)
H15: solar_radiation_mj_m2 is MNAR (57.4% missing in test) — CONFIRMED (Cell 25)
H16: UPLAND_SKM ordering consistent with optimal rainfall lag ordering — CONFIRMED (Cell 30 + 31)
H17: Env features without lag tma insufficient — CONFIRMED (Cell 26, +6.9% only)
H18: 9/10 top correlated station pairs share same MAIN_RIV — CONFIRMED (Cell 31)
H19: Test set has zero gaps (726 obs per station, all equal) — CONFIRMED (Cell 33)
H20: String matching exact across all files, no normalization needed — CONFIRMED (Cell 35)
H21: Floodway Bridge C outlier removal reduces skew from 13.29 → 1.03 — CONFIRMED (Cell 36)
H22: Rainfall more effective on wet soil (positive interaction) for downstream stations — CONFIRMED (Cell 34)
H23: Upstream-dominated stations (Napel, Jurug, Sekayu) show negative rainfall×soil interaction — CONFIRMED (Cell 34)
```

---

## VALIDATION STRATEGY

```
CV method     : BELUM DITENTUKAN
n_folds       : BELUM DITENTUKAN
Seed          : BELUM DITENTUKAN
Locked        : NO
Notes         : - Feb 2025 gap must be handled before validation design
                - Test set missing Jun–Aug entirely — CV folds must reflect seasonal distribution
                - ENSO regime shift means late train period closer to test distribution
                - Consider time-series CV with gap to prevent leakage
```

---

## EXPERIMENT LOG

```
Anchor model   : BELUM
Best CV RMSE   : BELUM
Best LB RMSE   : BELUM
Submissions    : 0 used, 36 remaining
```

---

## PENDING ACTIONS

```
[ ] Jeremy: Cell 38+ — feature engineering (reindex, lag tma, upstream features)
[ ] Jeremy: investigasi 5 invalid readings (tma_mdpl <= 0)
[ ] Jeremy: konfirmasi apakah rainfall_max_24h_mm backward atau forward-looking
[ ] Jeremy: investigasi Kali Pepe PTPN peculiar dynamics
[ ] Jeremy: investigasi Jurug negative skew dynamics
[ ] Jeremy: handle 90 missing test rows (2026-05-18) — forward-fill
[ ] Jeremy: winsorize strategy untuk Kali Anyar
[ ] Jeremy: desain validation strategy (time-series CV)
[ ] Ababil: BELUM mulai (menunggu Jeremy selesai EDA + FE)
[ ] Vierico: BELUM mulai
```

---

## CATATAN BEBAS

```
2026-07-13: Project State dibuat pertama kali dari Cell 0–10.
2026-07-13: Update Cell 11–20. Station taxonomy consolidated. River routing confirmed.
2026-07-13: Update Cell 21–30. HydroRIVERS integrated. Bengawan Solo system confirmed.
2026-07-13: Update Cell 31–37. Full rewrite.
             - Topological ordering validated against empirical pairs (Cell 31)
             - sample_submission format confirmed: id + tma_mdpl, all zeros placeholder (Cell 32)
             - Test period clean: 726 obs per station, no gaps (Cell 33)
             - Rainfall × soil moisture interaction identified: positive for 7 downstream stations,
               negative for upstream-dominated stations (Cell 34)
             - String matching confirmed exact across all files (Cell 35)
             - Post-cleaning skewness: only Floodway Bridge C changed (13.29 → 1.03) (Cell 36)
             - Env merged to train/test. 6 columns dropped. rolling_rain_48h/72h added.
               Train: 0 missing. Test: 90 missing (2026-05-18 only, 0.2% per station) (Cell 37)
             - Metric confirmed: RMSE (from competition page)
             - Submission budget: 36 remaining, 0/3 used today
```