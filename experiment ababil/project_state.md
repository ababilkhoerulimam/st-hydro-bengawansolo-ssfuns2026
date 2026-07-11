```markdown
# PROJECT STATE
# Update ini setiap kali ada handoff atau ganti akun Claude.
# Saat ganti akun: paste AGENTS.md dulu, lalu paste file ini, lalu ketik "lanjutkan dari [stage]"

---

## META

```
Competition / Project : Prediksi Tinggi Muka Air (TMA) — BBWS Bengawan Solo
                        Sebelas Maret Statistics Data Science 2026
Kaggle URL            : https://www.kaggle.com/competitions/sebelas-maret-statistics-data-science-2026/data
Deadline              : 25 Juli 2026 pukul 23:59 WIB
Metric                : RMSE (Root Mean Squared Error) — semakin kecil semakin baik
                        (Micro-average per baris, BUKAN per-pos)
Submission budget     : 28 total | Kuota harian: 3 sub/hari (periode 11–25 Juli)
Submissions used today: 1 / 3
Submissions remaining : 27
Last updated          : 11 Juli 2026, malam
Updated by            : Ababil (Stage 9 — Diagnostik & Ablasi selesai)
```

---

## CURRENT STATUS

```
Active phase          : FASE 9 MODELING (DIAGNOSTIK SELESAI) → MENUJU SLOT 2
Last completed stage  : Ababil — Eksperimen Ablasi & Validasi v2 (240 hari)
Next action           : Finalisasi Slot 2 berdasarkan temuan terbaru:
                        1. DROP nino_34 dari feature set (terbukti overfit)
                        2. Perhatikan pos outlier (Gunungsari, Wonogiri Dam)
                        3. Gunakan per-horizon-bucket model (0-30, 31-90, 91-241 hari)
                        Vierico — B1 Problem Brief (masih pending)
                        Jeremy — E9 finalisasi dokumentasi
Blocker (if any)      : NONE — semua diagnostic clear, hipotesis teridentifikasi
```

---

## DATASET

### File & Path

```
Semua file relatif terhadap root project (sejajar dengan train.csv)

train.csv                                     — TMA historis (target)
test.csv                                      — data prediksi
sample_submission.csv                         — format submisi
data_pendukung/data_lingkungan.csv            — fitur eksogen per jam
data_pendukung/koordinat_pos.csv              — koordinat 30 pos
data_pendukung/HydroRIVERS_v10_au_shp/       — shapefile sungai
agents/AGENTS_ababil.md                       — role Ababil
agents/AGENTS_jeremy.md                       — role Jeremy
agents/AGENTS_vierico.md                      — role Vierico
project_state.md                              — file ini
train_fe.parquet / train_fe.csv               — hasil FE pipeline (training)
test_fe.parquet / test_fe.csv                 — hasil FE pipeline (test)
exp1.ipynb                                    — notebook eksperimen pertama
```

### Ukuran & Struktur

```
train.csv             : 84.396 rows × 3 cols
test.csv              : 21.780 rows × 1 col
data_lingkungan.csv   : 888.480 rows × 27 cols
koordinat_pos.csv     : 30 rows × 3 cols
Periode train         : 2023-01-01 06:00:00 → 2025-09-18 18:00:00
Periode test          : 2025-09-19 06:00:00 → 2026-05-18 18:00:00
Gap train→test        : 12 jam (nyambung langsung)
Frekuensi TMA         : 3x sehari (06:00, 12:00, 18:00)
Frekuensi env         : per jam (00:00–23:00)
Jumlah pos            : 30 pos pemantauan DAS Bengawan Solo
Test period mencakup  : 724 hari hujan, 243 hari kemarau
                        (transisi kemarau→hujan penuh: Sep-Okt 2025 kemarau,
                         Nov 2025-Apr 2026 hujan PENUH, Mei 2026 awal kemarau)
```

### Kolom train_fe.parquet / test_fe.parquet

```
# Identitas
datetime                datetime    Waktu observasi
nama_pos                str         Nama pos pemantauan (30 nilai unik)
tma_mdpl                float64     TARGET — Tinggi Muka Air (train only)

# Temporal Features (4)
bulan                   int64       1–12
day_of_year             int64       1–365
hour                    int64       6, 12, 18
days_since_last_valid_tma float64   Train: 0 | Test: 0.5–241 hari

# Pos Identity & Static (6)
latitude                float64     Koordinat lintang
longitude               float64     Koordinat bujur
tma_mean_pos            float64     Mean TMA per pos (per-fold di CV)
tma_std_pos             float64     Std TMA per pos (per-fold di CV)
ac_lag1_pos             float64     Autocorrelation lag-1 per pos (per-fold di CV)

# Exogenous — Langsung dari data_lingkungan (19)
soil_moisture_0_7cm     float64     Kelembapan tanah 0–7cm (m³/m³)
soil_moisture_7_28cm    float64     Kelembapan tanah 7–28cm (m³/m³) ← corr +0.241
soil_moisture_28_100cm  float64     Kelembapan tanah 28–100cm (m³/m³)
rainfall_mm             float64     Curah hujan (mm)
rainfall_max_24h_mm     float64     Rolling max 24h (backward-only) ✅
humidity_pct            float64     Kelembapan udara (%)
dew_point_c             float64     Titik embun (°C)
cloud_cover_pct         float64     Tutupan awan (%)
temperature_c           float64     Suhu udara 2m (°C)
wind_speed_kmh          float64     Kecepatan angin 10m (km/h)
wind_direction_deg      float64     Arah angin (°)
pressure_msl_hpa        float64     Tekanan MSL (hPa)
rmm1                    float64     Komponen RMM1 MJO
rmm2                    float64     Komponen RMM2 MJO
mjo_amplitude           float64     Amplitudo MJO
mjo_phase               float64     Fase MJO (1–8)
nino_34                 float64     Indeks ENSO Nino 3.4 (°C) ⚠️ AKAN DI-DROP

# Engineered Rolling Features (3)
rolling_rain_48h        float64     Rolling sum 48h per pos (backward-only)
rolling_rain_72h        float64     Rolling sum 72h per pos (backward-only)
rolling_rain_7d         float64     Rolling sum 7d per pos (backward-only)

# Horizon Features (2)
horizon_days            float64     Test: 0–241 | Train: 0
horizon_bucket          str         Test: near/mid/far | Train: train
```

### 30 Pos Pemantauan (sorted by mean TMA desc) — SUDAH DIVALIDASI

```
                          AC-lag1  mean TMA   obs    catatan
Ngadipiro                  0.804   143.6    2901
Ngrembang                  0.723   140.1    2901
Wonogiri Dam               0.998   132.3    2901   ⚠️ RMSE 2.30 di Slot 1 (padahal AC tertinggi)
Badegan                    0.851   122.4    2901
Colo Weir                  0.988   107.8    2903
Kali Pepe - Tugu Boto      0.611    94.8    2900
Peren                      0.047    91.3    2902   spike-prone
Jarum                      0.024    90.7    2864   spike-prone
Sekayu                     0.878    87.3    2901
Kali Anyar - Kreteg Abang  0.036    86.5    2901   spike-prone
Serenan                    0.953    86.3    2901
Kali Pepe - PTPN           0.009    82.5    2901   spike-prone (AC terendah)
Jurug                      0.382    78.8    2903
Kedungupit                 0.233    64.9    2901
Kajangan                   0.225    51.0    2901
Ketonggo                   0.815    37.5    2901
Napel                      0.094    34.2    2901   spike-prone
Karangnongko               0.078    22.7    2901   spike-prone
Cepu                       0.976    17.5    2893
Brangkal                   0.895    13.0    2901
Lorog                      0.813    12.9    2901
Gunungsari                 0.904    10.0    1126   ⚠️⚠️ data sangat sedikit (39% normal), RMSE 3.17
Bengkelolor                0.955     9.8    2901
Bojonegoro - Kali Kethek   0.133     9.1    2688   data sedikit + AC rendah + spike
Sumberrejo                 0.974     7.5    2878
Babat                      0.964     6.3    2838
Boboh Kali Lamong          0.978     4.1    2902
Floodway Bridge C          0.553     3.9    2377   gap 163 hari di training
Karanggeneng               0.981     2.2    2903
Arjowinangun - Pacitan     0.624     1.1    2903
```

---

## EXPLORATION REPORT STATUS (Jeremy)

```
Jeremy stage saat ini : E8 DONE — Exploration Report SENT TO ABABIL (11 Juli 2026)
Post-FE Report (E9)   : SUDAH DIJALANKAN — plotting 31 features selesai
                        Insight digunakan dalam Stage 8 finalisasi
```

### Key Findings dari Jeremy — SUDAH DIVALIDASI DAN DIADOPSI

```
FINDING 1 — TMA adalah 30 distribusi terpisah → CONFIRMED | F05, F07, F08
FINDING 2 — Dua karakter pos (stabil vs spike) → CONFIRMED | F09, F08
FINDING 3 — Soil moisture lapisan dangkal = prediktor terbaik → CONFIRMED | F10–F12
FINDING 4 — rainfall_mm == rainfall_openmeteo_mm (duplikat) → CONFIRMED | DROP
FINDING 5 — Akumulasi hujan > intensitas sesaat → CONFIRMED | F27–F29
FINDING 6 — Gap sistemik Feb–Mar 2025 → CONFIRMED | Masking protokol
FINDING 7 — Floodway Bridge C gap 163 hari → CONFIRMED | F05 + global model
FINDING 8 — Gunungsari hanya 1.126 obs → CONFIRMED | F05 + global model
FINDING 9 — Outlier ekstrem = event banjir nyata → CONFIRMED | JANGAN clip
FINDING 10 — 5 nilai anomali kecil di train → CONFIRMED | Impute lag-1
```

---

## BUSINESS BRIEF STATUS (Vierico)

```
Checkpoint B1 (Problem Brief)  : BELUM — akan dimulai setelah baseline Slot 1-2
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

## FEATURE SET (Ababil)

```
Ababil stage saat ini : Stage 9 — FE FINAL (dengan koreksi dari ablasi)
FE status             : ✅ DONE (31 fitur → akan berkurang 1 = 30 fitur)
Leakage check         : ✅ COMPLETE — semua feature clear
Vierico FE review     : BELUM (akan dilakukan di B2)
```

### Keputusan Preprocessing Final — TIDAK BERUBAH

**DROP (tidak dipakai):**
| Feature | Alasan |
| :--- | :--- |
| `solar_radiation_mj_m2` | 57.5% missing test (sentinel -999), MNAR struktural |
| `rainfall_openmeteo_mm` | 100% duplikat `rainfall_mm` |
| `surface_pressure_hpa` | Proxy elevasi, korelasi spurious -0.947 global |
| `soil_moisture_100_255cm` | Korelasi target lemah, redundant |
| `mjo_active` | Binary derivasi dari `mjo_amplitude` |
| `built_surface_m2` | Statis per pos, sudah terwakili `nama_pos` |
| `landcover_class` / `landcover_name` | Statis per pos, sudah terwakili `nama_pos` |

**KEEP + forward-fill:**
| Feature | Treatment |
| :--- | :--- |
| `nino_34` | Forward-fill untuk 7.45% missing (Mei 2026) → **AKAN DI-DROP setelah ablasi** |
| `pressure_msl_hpa` | Forward-fill per pos untuk 0.41% missing |
| `soil_moisture_0_7cm` | Forward-fill per pos untuk 0.41% missing |
| `soil_moisture_7_28cm` | Forward-fill per pos untuk 0.41% missing |
| `soil_moisture_28_100cm` | Forward-fill per pos untuk 0.41% missing |
| `rmm1`, `rmm2`, `mjo_phase`, `mjo_amplitude` | Forward-fill per pos untuk 0.41% missing |

### Perubahan dari Ablasi — AKAN DITERAPKAN DI SLOT 2

| Feature | Keputusan Baru | Alasan |
| :--- | :--- | :--- |
| `nino_34` | **DROP dari feature set** | Ablasi: tanpa nino_34 OOF turun 0.0344 (1.3052 → 1.2709). Indikasi model overfit ke pola El Niño di training yang tidak general ke rezim La Niña di test (KS drift 0.508). |

### Final Feature Set — SLOT 2 (30 fitur)

**DROP nino_34 dari daftar sebelumnya.** Semua fitur lain tetap.

```
FEATURES_SLOT2 = [
    # Temporal
    "bulan", "day_of_year", "hour", "days_since_last_valid_tma",
    # Identity
    "nama_pos", "latitude", "longitude",
    "tma_mean_pos", "tma_std_pos", "ac_lag1_pos",
    # Exogenous (tanpa nino_34)
    "soil_moisture_0_7cm", "soil_moisture_7_28cm", "soil_moisture_28_100cm",
    "rainfall_mm", "rainfall_max_24h_mm",
    "humidity_pct", "dew_point_c", "cloud_cover_pct",
    "temperature_c", "wind_speed_kmh", "wind_direction_deg",
    "pressure_msl_hpa",
    "rmm1", "rmm2", "mjo_amplitude", "mjo_phase",
    # Engineered Rolling
    "rolling_rain_48h", "rolling_rain_72h", "rolling_rain_7d",
    # Horizon
    "horizon_days", "horizon_bucket",
]
```

---

## EXPERIMENT LOG & DIAGNOSTIK

### Exp001 — Slot 1 (Global LightGBM Direct, tanpa lag TMA)

**Validasi v1 (4-fold walk-forward, val terpanjang 121 hari):**
- OOF RMSE: **1.2888**
- Public LB: **1.68063**
- Gap (LB – CV): **+0.3918**

**Validasi v2 (Fold 4 diperpanjang hingga 240 hari):**
- OOF RMSE: **1.3052** (hanya naik 0.0164 → gap tetap ~0.38)
- Breakdown horizon-bucket di Fold 4 (240 hari):

| Horizon Bucket | n_rows | RMSE   |
|----------------|--------|--------|
| 0–30 hari      | 1.266  | 1.3527 |
| 31–90 hari     | 4.671  | 1.1199 |
| 91–241 hari    | 13.462 | 1.2654 |

**Pola:** berbentuk U/lembah (bucket tengah terbaik, bucket dekat terburuk) — **bukan** monoton naik. Ini menggagalkan hipotesis "gap disebabkan oleh horizon panjang yang tidak pernah divalidasi".

### Ablasi & Analisis Kontribusi Error

**A. Eksklusi Pos Outlier (Gunungsari & Wonogiri Dam):**
- OOF RMSE (semua 30 pos) : 1.3052
- OOF RMSE (exclude 2 pos): **1.2709** (turun 0.0343)
- **Kesimpulan:** 2 pos outlier menyumbang error tidak proporsional. Perlu perhatian khusus di Slot 2.

**B. Top 5 kontributor SSE terhadap total (dari v2):**
| Pos | % SSE | RMSE |
|-----|-------|------|
| Ngadipiro | 7.68% | 101.75 |
| Ngrembang | 7.29% | 99.15 |
| Wonogiri Dam | 6.18% | 91.30 |
| Badegan | 5.07% | 82.68 |
| Karanggeneng | 3.91% | 72.58 |

**Top 5 pos menyumbang 30.1% dari total SSE** (lebih tinggi dari 16.7% jika merata). Error terkonsentrasi di pos-pos tertentu.

**C. Ablasi nino_34:**
- OOF RMSE dengan nino_34   : 1.3052
- OOF RMSE tanpa nino_34    : **1.2709**
- Delta (tanpa - dengan)    : **-0.0344**
- **Kesimpulan:** Tanpa nino_34, OOF JUSTru LEBIH BAIK secara lokal. Ini indikasi kuat model overfit ke pola nino_34 dari training yang tidak general ke rezim iklim test (La Niña, KS drift 0.508). **nino_34 akan DROP di Slot 2.**

### Ringkasan Gap CV↔LB — Hipotesis yang Sudah Dicoret

| Hipotesis | Status | Bukti |
|-----------|--------|-------|
| Horizon validasi terlalu pendek | ❌ Dicoret | Fold 4 240 hari hanya naik 0.0164 |
| Model belum pernah lihat transisi musim | ❌ Dicoret | 2x musim hujan penuh terlihat di training |
| Musim hujan puncak under-represented | ❌ Dicoret | Gap nyaris tidak membaik di fold v2 |
| nino_34 drift | ✅ TERKONFIRMASI | Ablasi bukti kuat: OOF turun 0.0344 tanpa nino_34 |

---

## VALIDATION STRATEGY (locked after Stage 7)

```
CV method     : Walk-forward time series split, 4 folds, OOF RMSE
n_folds       : 4
Seed          : 42
Group column  : nama_pos (untuk monitoring per-pos, scoring micro)

FOLD DESIGN (v2 — final):
  Fold 1: Train → 2024-04-30 | Val: 2024-05-01 → 2024-07-31 (91 hari)
  Fold 2: Train → 2024-07-31 | Val: 2024-08-01 → 2024-10-31 (91 hari)
  Fold 3: Train → 2024-10-31 | Val: 2024-11-01 → 2025-02-28 (119 hari)
  Fold 4: Train → 2025-01-20 | Val: 2025-01-21 → 2025-09-18 (240 hari)

GAP PROTOCOL (exact, locked):
  GAP_START = 2025-02-03 18:00:00
  GAP_END   = 2025-03-01 06:00:00
  Durasi    = 25 hari 12 jam (exact)
  Masking: rolling features corrupt di sekitar gap sesuai window:
    - rolling_rain_48h: corrupt sampai 2025-03-03 06:00
    - rolling_rain_72h: corrupt sampai 2025-03-04 06:00
    - rolling_rain_7d:  corrupt sampai 2025-03-08 06:00

FEATURE COMPUTATION INSIDE CV:
  tma_mean_pos, tma_std_pos, ac_lag1_pos
  → Dihitung dari train fold only, diterapkan ke val fold
  → Test inference: dihitung dari full train.csv (aman)

⚠️ JANGAN random KFold — data bersifat temporal
⚠️ Gap Feb–Mar 2025 TIDAK BOLEH jadi fold boundary

CEK B — Scoring Mechanism (CONFIRMED dari halaman Kaggle):
  1. RMSE = micro-average per baris (semua baris test digabung).
     Rumus: RMSE = sqrt(1/n * sum((y_i - y_hat_i)^2))
     → BUKAN macro-average per pos.
  2. Public LB menggunakan ~30% test data.
  3. Private LB (final standings) menggunakan 70% sisanya.
  4. Implikasi: Fokus pada CV RMSE, JANGAN overfit ke public LB.
```

---

## PENDING ACTIONS

```
[HIGH] Ababil   — Implementasi Slot 2:
                   1. DROP nino_34 dari feature set
                   2. Per-horizon-bucket model (3 model terpisah)
                   3. Perhatikan pos outlier (Gunungsari, Wonogiri Dam)
[HIGH] Ababil   — Submit Slot 2, catat LB, bandingkan delta
[MED]  Ababil   — Evaluasi apakah Slot 3 (ensemble) diperlukan
[MED]  Vierico  — mulai B1 Problem Brief (dapat dilakukan paralel)
[LOW]  Jeremy   — finalisasi E9 dokumentasi
[LOW]  Ababil   — update state setelah setiap eksperimen
```

### Actions Already Completed

```
✅ Ababil — baca Exploration Report (E8)
✅ Ababil — Stage 3 (Data Quality & Preprocessing Final)
✅ Ababil — Stage 4 (Competitive Research)
✅ Ababil — Stage 5 (Strategy Discussion)
✅ Ababil — Stage 6 (Hypothesis Generation)
✅ Ababil — Stage 7 (Validation Design) — LOCKED
✅ Ababil — Stage 8 (FE Proposal & Pipeline) — DONE
✅ Ababil — verifikasi F14 (rainfall_max_24h_mm backward-only) — CONFIRMED
✅ Ababil — Stage 9 Exp001 (Slot 1) — SUBMITTED | LB: 1.68063
✅ Ababil — Validasi v2 (Fold 4 240 hari) — DONE | OOF: 1.3052
✅ Ababil — Ablasi nino_34 — DONE | Terbukti overfit, akan DROP
✅ Ababil — Breakdown horizon-bucket — DONE | Pola U-shape, bukan monoton
✅ Jeremy — E9 Post-FE Plotting — SUDAH DIJALANKAN
✅ Vierico — CEK B (scoring mechanism) — DONE via halaman Kaggle
```

---

## CONTEXT RESET PROTOCOL

Saat ganti akun Claude (limit habis), lakukan urutan ini:

**Untuk Ababil:**
1. Buka akun baru
2. Pesan 1: paste isi `agents/AGENTS_ababil.md`
3. Pesan 2: paste isi file ini (`project_state.md`)
4. Pesan 3: "Lanjutkan dari Stage 9 — Implementasi Slot 2. Semua context ada di atas. train_fe.csv dan test_fe.csv sudah ready. Keputusan: DROP nino_34, per-horizon-bucket model, perhatikan pos outlier."

**Untuk Jeremy:**
1. Buka akun baru
2. Pesan 1: paste isi `agents/AGENTS_jeremy.md`
3. Pesan 2: paste isi file ini (`project_state.md`)
4. Pesan 3: "Lanjutkan dari E9 — finalisasi dokumentasi Post-FE Report. Semua context ada di atas."

**Untuk Vierico:**
1. Buka akun baru
2. Pesan 1: paste isi `agents/AGENTS_vierico.md`
3. Pesan 2: paste isi file ini (`project_state.md`)
4. Pesan 3: "Lanjutkan dari checkpoint B1. Semua context ada di atas."

**Jangan skip langkah ini.** Tanpa project_state, Claude mulai dari nol.

---

## CATATAN BEBAS

```
11 Juli 2026 (Tim)    : Project state diinisialisasi. Kompetisi baru mulai ~13 jam lalu.
11 Juli 2026 (Jeremy) : EDA E1–E8 selesai. Semua temuan, risk flags, dan hipotesis
                        terdokumentasi. Bola ada di tangan Ababil.
11 Juli 2026 (Ababil) : Stage 3–8 selesai dalam satu sprint. Semua diagnostic selesai.
                        Gap exact: 2025-02-03 18:00 → 2025-03-01 06:00.
                        Validation locked (4-fold walk-forward, OOF RMSE).
                        31 features approved, FE pipeline deployed, sanity check pass.
                        train_fe.parquet dan test_fe.parquet ready.
11 Juli 2026 (Ababil) : F14 rainfall_max_24h_mm diverifikasi — backward-only rolling,
                        aman dari future leakage.
11 Juli 2026 (Ababil) : Exp001 (Slot 1) disubmit — LB: 1.68063, CV: 1.2888, gap +0.3918.
11 Juli 2026 (Ababil) : Validasi v2 (Fold 4 240 hari) — OOF: 1.3052. Breakdown
                        horizon-bucket menunjukkan pola U-shape, bukan monoton naik.
                        Hipotesis "gap karena horizon panjang" dicoret.
11 Juli 2026 (Ababil) : Ablasi nino_34 — terbukti overfit (OOF turun 0.0344).
                        nino_34 akan DROP di Slot 2.
11 Juli 2026 (Ababil) : Kontribusi error top 5 pos = 30.1% dari total SSE.
                        Gunungsari & Wonogiri Dam adalah outlier utama.
11 Juli 2026 (Tim)    : Deadline malam ini. Paralel proceed: Ababil lanjut Slot 2
                        dengan keputusan: DROP nino_34, per-horizon-bucket model,
                        perhatikan pos outlier. Vierico B1 pending.
```

## META (update baris ini)

Submissions used today: 3 / 3
Submissions remaining : 25
Last updated          : 11 Juli 2026, malam (post-exp003)
Updated by             : Ababil — 3 submission hari ini selesai, evaluasi exp004 besok

---

## CURRENT STATUS (update)

Active phase          : FASE 9 MODELING → EVALUASI HASIL 3 SUBMISSION, PERSIAPAN EXP004
Last completed stage  : exp003 (Slot1/2 + tma_last_known) — SUBMITTED, LB TERBAIK SEJAUH INI
Next action            : 1. Fix tma_lag1 & tma_rolling_mean_7d (100% NaN di test, tidak berfungsi)
                          2. Re-run CV bersih: fitur existing + tma_last_known SAJA (drop lag features rusak)
                          3. Cek ulang nino_34 in/out DENGAN tma_last_known ikut serta
                          4. Submission budget baru tersedia besok (kuota harian reset)
Blocker (if any)       : NONE — submission budget hari ini habis, lanjut besok

---

## EXPERIMENT LOG & DIAGNOSTIK (tambahkan section baru)

### Exp001 — Slot 1 (Global LightGBM, dengan nino_34, tanpa lag TMA)
OOF RMSE   : 1.2949 (run awal) / 1.3052 (validasi v2, fold 4 240 hari)
Public LB  : 1.68063
Gap        : +0.3754 (vs OOF v2)

### Exp002 — Slot 2 (Global LightGBM, TANPA nino_34)
OOF RMSE   : 1.2709 (turun 0.0344 dari Slot 1 — awalnya dikira improvement)
Public LB  : 1.68894  ⚠️ LEBIH BURUK dari Slot 1, meskipun OOF lebih baik
Gap        : +0.4180 (lebih lebar dari Slot 1)
Kesimpulan : Ablasi nino_34 di CV TIDAK menerjemah ke LB. Bukti kuat CV lokal
             (walk-forward 4-fold) TIDAK predictive terhadap LB behavior untuk
             perubahan fitur ini. nino_34 kemungkinan bukan overfit murni —
             CV membaik saat di-drop, tapi LB memburuk. Hipotesis "drop nino_34
             = generalisasi lebih baik" DIBANTAH oleh LB.

### Exp003 — Slot 1/2 + tma_last_known (+ tma_lag1, tma_rolling_mean_7d — RUSAK)
Fitur baru : tma_last_known (valid di test), tma_lag1 (100% NaN test — TIDAK
             FUNGSIONAL), tma_rolling_mean_7d (100% NaN test — TIDAK FUNGSIONAL)
Public LB  : 1.68000  ✅ TERBAIK dari 3 eksperimen
Catatan kritis :
  - tma_lag1 & tma_rolling_mean_7d dihitung dari histori TMA per pos, tapi di
    test set nilainya 100% NaN (test = forecast horizon 0-241 hari tanpa TMA
    aktual di antaranya). LightGBM treat NaN sebagai missing & fallback ke
    default split — fitur ini EFEKTIF TIDAK DIPAKAI saat inference test,
    meskipun mungkin berpengaruh signifikan saat training/CV (val fold masih
    py akses ke histori TMA asli → val-test MISMATCH, mirip leakage).
  - Improvement dari 1.68063/1.68894 ke 1.68000 kemungkinan besar berasal dari
    tma_last_known (yang VALID di test, missing=0), BUKAN dari tma_lag1/
    tma_rolling_mean_7d.
  - ⚠️ ACTION ITEM: hapus tma_lag1 & tma_rolling_mean_7d dari feature set exp
    berikutnya (tidak berguna, berpotensi menyesatkan OOF), ATAU didesain
    ulang total dengan strategi recursive/multi-step forecasting yang benar.

### Ringkasan 3 Submission Hari Ini

| Exp | Fitur vs Slot 1 | OOF RMSE | Public LB | Delta LB vs Slot 1 |
|-----|-----------------|----------|-----------|---------------------|
| 001 | baseline (dengan nino_34) | 1.3052 | 1.68063 | — |
| 002 | tanpa nino_34 | 1.2709 | 1.68894 | +0.00831 (memburuk) |
| 003 | + tma_last_known (+ 2 fitur rusak) | (belum dihitung ulang bersih) | **1.68000** | -0.00063 (membaik tipis) |

### Insight Kritis Baru — CV↔LB Tidak Reliable untuk Keputusan Fitur

CV walk-forward 4-fold yang dipakai sejauh ini TERBUKTI tidak predictive
terhadap arah perubahan LB untuk perubahan fitur skala kecil (nino_34 in/out).
Exp002 adalah bukti langsung: OOF membaik 0.0344, LB memburuk 0.00831.
Implikasi:
  - JANGAN ambil keputusan fitur hanya dari delta OOF kecil (<0.02-0.03)
    tanpa submission konfirmasi.
  - Prioritaskan submission budget untuk hipotesis dengan potensi impact
    besar (pos outlier handling, model architecture, ensemble) daripada
    fine-tuning fitur individual berdasarkan sinyal CV lemah.
  - Pertimbangkan: apakah split CV representatif terhadap public LB 30%
    sample? Belum diverifikasi — jadi PENDING ACTION.

---

## PENDING ACTIONS (tambahkan)

[HIGH] Ababil — Bersihkan feature set: hapus tma_lag1 & tma_rolling_mean_7d
                (100% NaN test, tidak fungsional) dari FEATURES_S2
[HIGH] Ababil — Re-run CV bersih: baseline + tma_last_known SAJA, cek ulang
                nino_34 in/out dengan tma_last_known sebagai kontrol
[MED]  Ababil — Investigasi apakah public LB (30% test) representatif
                terhadap CV fold desain saat ini (random per-baris atau
                stratified by time/pos?)
[MED]  Ababil — Ensemble check: sudah coba avg(Slot1, Slot2) di exp003?
                (perlu diklarifikasi apakah exp003 ini ensemble atau feature
                addition — dari kode terakhir, ini FEATURE ADDITION bukan
                ensemble avg)
[LOW]  Ababil — Submission budget baru tersedia besok — rencanakan 3 submisi
                besok dengan prioritas: (1) fitur bersih tma_last_known only,
                (2) ensemble avg(best models), (3) cadangan berdasarkan hasil
                (1) dan (2)