# PROJECT STATE
# Update ini setiap kali ada handoff atau ganti akun Claude.
# Saat ganti akun: paste AGENTS.md dulu, lalu paste file ini, lalu ketik "lanjutkan dari [stage]"

---

## META
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

```
Competition / Project : Prediksi Tinggi Muka Air (TMA) — BBWS Bengawan Solo
                        Sebelas Maret Statistics Data Science 2026
                        70% Private, 30% public (leaderboard)
Kaggle URL            : https://www.kaggle.com/competitions/sebelas-maret-statistics-data-science-2026/data
Deadline              : 25 Juli 2026 pukul 23:59 WIB
Metric                : RMSE (Root Mean Squared Error) — semakin kecil semakin baik
Submission budget     : 37 total | Kuota harian: 3 sub/hari (periode 11–25 Juli)
Submissions used today: 0 / 3
Last updated          : 13 Juli 2026 (post-EDA 7-11, Stage 3 Step 1-2)
Updated by            : Ababil (Stage 3 — Technical EDA in progress)
```

---

## CURRENT STATUS

```
Active phase          : FASE 2 TECHNICAL EDA (Stage 3)
Last completed stage  : Jeremy E8 + Ababil EDA 7-11 ✓
Current stage         : Ababil Stage 3 (Technical EDA) — Step 2 DONE, Step 3 pending
Next action           : Ababil → Stage 3 Step 3 (Leakage Candidate Verification)
                        Proceed dengan Multicollinearity confirmation (sudah ada dari EDA 4)
Blocker (if any)      : NONE — Stage 3 running smoothly
```

---

## DATASET

[Dataset section tetap sama seperti sebelumnya]

---

## EXPLORATION REPORT STATUS (Jeremy)

```
Jeremy stage saat ini : E8 DONE — Exploration Report SENT TO ABABIL
Post-FE Report (E9)   : BELUM — standby menunggu delegasi dari Ababil
EDA TAMBAHAN (7-11)   : DONE ✓ (13 Jul 2026 oleh Ababil)
```

### Key Findings dari Jeremy (E1–E8) + EDA Tambahan 7–11

```
[Semua FINDING 0–15 dari E1-E8 tetap sama, ditambahkan:]

FINDING 16 — [BARU, EDA 7] Koordinat spasial: dua cluster geografis jelas
  Cluster Barat (Solo Hulu, lon ~110.7-111.0): Jarum, Serenan, Peren, Kali Pepe,
  Kali Anyar, Jurug, Colo Weir, Wonogiri, Ngadipiro, Ngrembang, Badegan
  → mean TMA tinggi (78–143 mdpl), zona pegunungan/hulu
  Cluster Timur (Bengawan Hilir, lon ~111.5-112.6): Karanggeneng, Babat,
  Sumberrejo, Bengkelolor, Boboh Kali Lamong, Bojonegoro, Cepu, Brangkal
  → mean TMA rendah (2–22 mdpl), zona dataran/hilir
  Lag optimal konsisten dgn jarak geografis: semakin jauh, lag semakin besar
  (Cepu→Sumberrejo 47km lag-12h; Cepu→Karanggeneng 87km lag-18h)

FINDING 17 — [BARU, EDA 8] HydroRIVERS validasi urutan hulu-hilir secara hidrologi
  dis_av_cms (debit rata-rata) & ord_flow (stream order) confirm posisi dalam
  jaringan sungai: Colo Weir (0.12) paling hulu → Karanggeneng (442) paling hilir
  Wonogiri Dam dis_av_cms hanya 47.6 (di hulu) — masuk akal karena waduk
  Fitur dis_av_cms & ord_flow lebih informatif dari koordinat mentah

FINDING 18 — [BARU, EDA 9] MAE baseline per bulan & per jam sangat terstruktur
  Seasonal MAE: Mar (0.735) paling tinggi → Sep (0.439) terendah
  Per jam: 18:00 paling susah (0.630) karena puncak hujan konvektif sore
  Wonogiri Dam MAE meledak Jun-Agustus (3.5-4.5) → bukan step-function,
  butuh lag lebih panjang untuk menangkap dinamika lambat bendungan
  Kali Anyar: MAE tinggi justru di kemarau (Jun-Sep) — anomali perlu investigasi

FINDING 19 — [BARU, EDA 10] Floodway Bridge C: TIDAK ada level shift permanen
  T-test mean sebelum-sesudah gap: p=0.46 (tidak signifikan)
  KS-test: p≈0 tapi karena distribusi sesudah lebih lebar (std 0.93→1.50)
  Outlier ekstrem: TMA 47.0 mdpl (15 Juni 2025) dengan jump 42.3 mdpl
  → Kemungkinan: banjir nyata atau sensor error — PERLU investigasi manual

FINDING 20 — [BARU, EDA 11] Wonogiri Dam: BUKAN operasional bendungan agresif
  Lompatan >1 mdpl: hanya 3 dari 1.934 steps (0.16%) → sangat jarang
  Lompatan >2 mdpl: ZERO
  3 lompatan besar terjadi saat rainfall rendah (0.30mm vs rata-rata 2.30mm)
  → Indikasi operasional, tapi frekuensi negligible untuk model
  Penyebab MAE tinggi Jun-Agustus: kemungkinan karena rezim lambat (AC=0.998),
  model linear tidak cukup → butuh fitur lag panjang
  Tren tahunan: 2023 (131.99) → 2024 (131.20) → 2025 (134.64) ✓ CONFIRMED

FINDING 21 — [BARU, EDA 7-11] Train-test seasonal distribution shift EXPECTED
  Train over-represent Juni-Agustus (kemarau, ~9.4% each)
  Test over-represent Oktober-Maret (hujan, ~12-13% each)
  Seasonal drift di: soil_moisture, rainfall, humidity, dew_point, cloud_cover
  → Rendah risiko, model akan lihat bulan melalui fitur interaksi month×features
  SATU-SATUNYA drift berbahaya: nino_34 (KS=0.497)
  Train: +0.36 (El Niño), Test: -0.37 (La Niña) → phase shift ENSO nyata

FINDING 22 — [BARU, Stage 3] Missing value pattern: MCAR bukan MNAR
  soil_moisture/pressure/MJO (720, 2026-05-18): MCAR — cutoff artifact 1 hari
  → Forward-fill dari 2026-05-17, low risk
  nino_34 (12.960, seluruh Mei 2026): MCAR — data belum tersedia, bukan missing
  → Forward-fill dari April 2026, acceptable (ENSO index lambat)
  solar_radiation (-999, ~60% test): MNAR — structural mismatch train-test
  → REKOMENDASI: DROP (bukan impute), proxy dengan cloud_cover_pct
```

### Hipotesis Final E7 (ranked by expected impact) — UPDATED

```
H1 [TINGGI] Model per-pos (atau nama_pos sbg strong identifier) akan jauh
            kalahkan model global naif
            Evidence: skala TMA 1.1-143.6 mdpl, skew -22.5 s.d +51.5,
            AC lag-1 0.009-0.998, Simpson's paradox terbukti di E4
            Risk: overfitting kalau per-pos model tanpa cukup data (Gunungsari)
            Status: CONFIRMED — koordinat & HydroRIVERS bisa enhance per-pos features

H2 [TINGGI] Soil moisture 0-28cm = prediktor eksogen terkuat
            Evidence: within-pos corr 0.42-0.49, konsisten scr fisik &
            konsisten dgn rainfall_max_24h (corr 0.55-0.63 ke rainfall)
            Suggested: prioritaskan 0-7cm & 7-28cm, layer 28-100cm redundan
            Status: CONFIRMED, evidence kuat

H3 [TINGGI] Gap Feb 2025 (global) akan corrupt lag/rolling di SEMUA pos
            jika tidak di-mask
            Evidence: outage 4-28 Feb confirmed semua pos, lag-1 naif
            "melompat" 25 hari di baris 1 Maret (time-delta check)
            Status: CONFIRMED — wajib masking eksplisit di pipeline

H4 [SEDANG-TINGGI] solar_radiation lebih merugikan drpd membantu
            Evidence: -999 merata semua pos, persis 31 Des 2025-18 Mei
            2026 (139 hari = ~60% periode test), train 100% valid
            Suggested: pertimbangkan drop atau proxy cloud_cover_pct
            Status: CONFIRMED, REKOMENDASI DIPERKUAT → DROP

H5 [SEDANG] surface_pressure = proxy elevasi, bukan sinyal cuaca murni
            Evidence: global corr -0.947 vs within-pos -0.223 (revisi),
            urutan pos by pressure-diff = urutan pos by mean TMA
            Suggested: detrend per pos atau drop, andalkan nama_pos
            Status: CONFIRMED — revisi dr H4 lama (was -0.033, now -0.223)

H6 [SEDANG] Wonogiri Dam (& mungkin pos dekat bendungan) py dinamika
            non-hidrologi-alami (intervensi manusia)
            Evidence: kenaikan TMA 2023→2025 terbesar (+0.79), nama "Dam"
            Investigasi EDA 11: operasional bendungan TIDAK agresif (0.16% lompatan >1m)
            tapi dinamika lambat butuh lag panjang, MAE tinggi Jun-Agustus
            Status: KUALIFIKASI — bukan operasional dramatis, tapi rezim lambat berbeda

H7 [RENDAH-SEDANG] MJO/ENSO informasi rendah di level baris, mgkn
            berguna sbg modulator musiman
            Evidence: resolusi asli harian/bulanan, broadcast identik
            ke 30 pos, within-pos corr lemah (0.05-0.13)
            PENTING: nino_34 phase shift train→test (El Niño→La Niña)
            → Feature importance akan berubah, perlu monitoring
            Status: CONFIRMED broadcast pattern + PHASE SHIFT WARNING
```

### Risk Flags dari Jeremy — UPDATED

```
[Flags 0-13 tetap sama, ditambahkan:]

🟡 FLAG 14 [MED, NEW] : Cluster geografis + HydroRIVERS topology
                     Dua cluster spasial jelas (Barat=hulu, Timur=hilir)
                     Lag optimal konsisten dgn jarak: validated by dis_av_cms
                     Koordinat + HydroRIVERS features (dis_av_cms, ord_flow)
                     bisa enhance model per-pos. Jangan gunakan lat/lon raw,
                     encode sebagai distance-to-mouth atau stream_order.

🟡 FLAG 15 [MED, NEW] : Floodway Bridge C anomali outlier 47.0 mdpl (15 Jun 2025)
                     Lompatan 42.3 mdpl dalam 6 jam → sensor error atau banjir
                     ekstrem. WAJIB investigasi manual sebelum training.
                     Kalau error: remove atau clip. Kalau banjir nyata: keep.

🟡 FLAG 16 [MED, NEW] : Seasonal drift adalah EXPECTED, bukan problem
                     Train over-represent kemarau, test over-represent hujan
                     Semua drift di atmospheric/soil fitur adalah seasonal
                     Low risk — model akan capture via month×feature interactions.

🔴 FLAG 17 [KRITIS, NEW] : nino_34 phase shift train→test (El Niño → La Niña)
                     Bukan seasonal drift, tapi structural difference dalam iklim
                     ENSO phase berbeda antar periode → feature importance shift
                     Imputation nino_34 Mei 2026 perlu hati-hati (phase consistency).
                     Wajib monitor model behavior di test apakah under/over-predict.
```

---

## BUSINESS BRIEF STATUS (Vierico)

```
Checkpoint B1 (Problem Brief)  : BELUM
Checkpoint B2 (EDA Commentary) : BELUM
Checkpoint B3 (Strategy Review): BELUM
Checkpoint B4 (Error Cost)     : BELUM
Checkpoint B5 (Explainability) : BELUM
Checkpoint B6 (Exec Summary)   : BELUM
```

**Active veto dari Vierico:**
```
- NONE
```

**Business constraints yang sudah dikonfirmasi:**
```
- [belum tersedia — tunggu B1]
```

---

## FEATURE SET (Ababil) — UPDATED POST-EDA 7-11

```
Ababil stage saat ini : STAGE 3 (Technical EDA) — Step 1-2 DONE, Step 3 pending
FE status             : BELUM (proposal tersedia, approval pending)
Leakage check          : IN PROGRESS (Step 3)
Vierico FE review     : BELUM
```

**Kandidat fitur (updated dengan EDA 7-11):**

```
PRIORITAS TINGGI:
  tma_mdpl lag-1, lag-2, lag-3            — per pos, WAJIB masking di gap 4-28 Feb 2025
                                             + gap lokal 2023 (Flags 5, 5b)
  soil_moisture_7_28cm                    — within-pos corr +0.492 (terkuat)
  soil_moisture_0_7cm                     — within-pos corr +0.457
  rainfall_max_24h_mm lag-1               — within-pos corr +0.318 (lag-1 optimal dari EDA 1)
  dew_point_c                             — within-pos corr +0.308
  humidity_pct                            — within-pos corr +0.252
  nama_pos                                — identitas pos (label/target encode) — WAJIB
  month, hour                             — calendar features (seasonal + diurnal pattern)
  tma_Cepu lag-2, tma_Karanggeneng lag-3 — cross-lag (validated EDA 2 + EDA 7 distance)

PRIORITAS SEDANG:
  soil_moisture_28_100cm                  — corr +0.425 tapi REDUNDAN (0.82 ke 7-28cm)
  soil_moisture_100_255cm                 — corr +0.140, decoupled dari rainfall jangka pendek
  cloud_cover_pct                         — proxy solar_radiation (valid penuh vs solar -999 60% test)
  wind_direction_deg                      — within-pos corr +0.137, efek topografi lokal
  mjo_amplitude                           — within-pos corr +0.133 (broadcast signal)
  rolling rainfall 48h, 72h               — perlu dibuat dari rainfall_mm
  dis_av_cms per pos (HydroRIVERS)        — proxy stream order / hulu-hilir (NEW EDA 8)
  ord_flow per pos (HydroRIVERS)          — stream order (NEW EDA 8)
  distance_to_mouth per pos               — encoded dari HydroRIVERS topology (NEW)
  lat_lon_encoded                         — cluster indicator: barat (hulu) vs timur (hilir) (NEW EDA 7)

PERLU TREATMENT KHUSUS SEBELUM PAKAI:
  surface_pressure_hpa                    — detrend per pos atau DROP
  solar_radiation_mj_m2                   — JANGAN pakai (60% test = -999)
  nino_34                                 — forward-fill Mei 2026 (TAPI: phase shift warning)
  mjo_active                              — redundan ke mjo_amplitude (corr 0.73)
  mjo_phase, rmm1, rmm2                   — sinyal lemah, coba sbg interaksi × month

DROP KANDIDAT:
  rainfall_openmeteo_mm                   — 100% duplikat rainfall_mm
  built_surface_m2                        — statis per pos (redundan with nama_pos)
  landcover_class / landcover_name        — statis per pos (redundan with nama_pos)
  solar_radiation_mj_m2                   — structural mismatch (NEW recommendation via Flag 2)
```

**Features yang sudah di-approve:**
| Feature | Type | Source | Leakage Check | Vierico | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| [belum ada] | | | | | |

**Features yang di-drop (recommended):**
```
- rainfall_openmeteo_mm (duplikat)
- solar_radiation_mj_m2 (60% test missing, better proxy available)
- built_surface_m2, landcover_* (statis, redundan)
- mjo_active (redundan to mjo_amplitude)
```

---

## STAGE 3 STATUS — Technical EDA (Ababil)

```
Stage 3 — STEP 1: Train vs Test Distribution Drift
  Status: ✅ COMPLETE
  Finding: 14/20 fitur drift parah, TAPI 13/14 adalah seasonal drift (EXPECTED)
  Hanya 1 drift berbahaya: nino_34 (KS=0.497, phase shift El Niño→La Niña)
  Rekomendasi: Proceed with monthly features untuk capture seasonal shift
  Risk: RENDAH (seasonal normal di time-series), TINGGI untuk nino_34

Stage 3 — STEP 2: Missing Value Pattern (MCAR/MAR/MNAR)
  Status: ✅ COMPLETE
  Finding: Semua missing adalah MCAR (cutoff artifacts atau future data)
  soil_moisture/pressure/MJO (720, 2026-05-18): Forward-fill dari -1 hari ✓
  nino_34 (12.960, Mei 2026): Forward-fill dari April ✓
  solar_radiation (-999, 60% test): JANGAN impute → DROP, proxy cloud_cover
  Rekomendasi: Forward-fill untuk soil/pressure/MJO, DROP solar_radiation

Stage 3 — STEP 3: Leakage Candidate Verification
  Status: PENDING (next step)
  Candidates to verify:
    - tma lag-1/2/3: temporal integrity (gap Feb 2025 masking)
    - cross-lag TMA hulu→hilir: no future leakage (lagged backward only)
    - nino_34 forward-fill: no information leakage (ENSO known ahead)
    - month/hour: no leakage (calendar deterministic)
    
Stage 3 — STEP 4: Multicollinearity Confirmation
  Status: PENDING (reuse dari EDA 4)
  High corr pairs (>0.7): soil_moisture layers, humidity-temperature, mjo_active-amplitude
  Rekomendasi: Keep strongest signal, drop redundant (already in FE candidate list)
```

---

## EXPERIMENT LOG SUMMARY

```
Anchor model (Slot 1) : [belum ada]
Anchor model (Slot 2) : [belum ada]
Best CV so far        : [belum ada]
Best LB so far        : [belum ada]
```

---

## VALIDATION STRATEGY (locked after Stage 7)

```
CV method     : [belum ditentukan]
                Kandidat: TimeSeriesSplit — WAJIB respek temporal order
                          GroupKFold by nama_pos (jika per-pos model)
n_folds       : [belum ditentukan]
Seed          : [belum ditentukan]
Group column  : [belum ditentukan — kandidat: nama_pos]
Locked        : NO

⚠️ JANGAN random KFold — data bersifat temporal
⚠️ Gap 4-28 Feb 2025 (presisi, global semua pos) TIDAK BOLEH jadi fold boundary
⚠️ Gap lokal 2023 (Floodway Bridge C, Bojonegoro) perlu handling khusus
⚠️ nino_34 phase shift: split harus respect ENSO phase consistency (train=El Niño, test=La Niña)
```

---

## PENDING ACTIONS — UPDATED

```
[HIGH] Ababil   — Stage 3 Step 3: Leakage Candidate Verification (in progress)
[HIGH] Ababil   — Stage 3 Step 4: Multicollinearity Confirmation (pending)
[HIGH] Ababil   — FE approval gate: lock feature set after Vierico review
[HIGH] Ababil   — Investigasi manual Floodway Bridge C outlier 47.0 mdpl (15 Jun 2025)
[MED]  Ababil   — Decide: detrend surface_pressure per pos, atau drop entirely?
[MED]  Ababil   — Decide: keep nino_34 atau drop karena phase shift risk?
[MED]  Ababil   — Investigasi Kali Anyar-Kreteg Abang: MAE tinggi di kemarau (anomali)
[MED]  Ababil   — Investigasi Wonogiri Dam: MAE Jun-Agustus naik (lag panjang needed?)
[MED]  Vierico  — Review proposed feature set (post-Stage 3)
[MED]  Vierico  — Assess business validity: HydroRIVERS features, cross-lag features
[LOW]  Jeremy   — Stage E9: Post-FE Business Plotting (delegasi ketika FE done)
```

---

## CONTEXT RESET PROTOCOL

[Tetap sama seperti sebelumnya]

---

## CATATAN BEBAS — UPDATED

```
12 Juli 2026 (Jeremy) : E8 Exploration Report SENT.
13 Juli 2026 (Ababil) : EDA Tambahan 7-11 complete.
                        - EDA 7: Koordinat spasial, dua cluster jelas, lag vs jarak validated
                        - EDA 8: HydroRIVERS topology confirm urutan hulu-hilir
                        - EDA 9: MAE baseline per bulan/jam, Wonogiri MAE anomali Jun-Agustus
                        - EDA 10: Floodway Bridge C outlier 47.0 mdpl (manual investigation needed)
                        - EDA 11: Wonogiri Dam operasional TIDAK agresif (0.16% lompatan >1m)
                        Key: Seasonal drift EXPECTED, nino_34 phase shift CRITICAL
13 Juli 2026 (Ababil) : Stage 3 (Technical EDA) Step 1-2 COMPLETE.
                        - Step 1: 14/20 drift, tapi 13/14 seasonal (expected), 1 critical (nino_34)
                        - Step 2: All missing = MCAR, forward-fill strategy ready
                        Next: Step 3 (Leakage check) → Step 4 (Multicollinearity)
                        Then: FE lock → E9 delegation → Stage 5 Strategy Discussion
```