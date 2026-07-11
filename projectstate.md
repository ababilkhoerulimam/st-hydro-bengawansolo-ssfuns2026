# PROJECT STATE
# Update ini setiap kali ada handoff atau ganti akun Claude.
# Saat ganti akun: paste AGENTS.md dulu, lalu paste file ini, lalu ketik "lanjutkan dari [stage]"

---

## META

```
Competition / Project : Prediksi Tinggi Muka Air (TMA) — BBWS Bengawan Solo
                        Sebelas Maret Statistics Data Science 2026
                        70% Private, 30% public ( leaderboard)
Kaggle URL            : https://www.kaggle.com/competitions/sebelas-maret-statistics-data-science-2026/data
Deadline              : 25 Juli 2026 pukul 23:59 WIB
Metric                : RMSE (Root Mean Squared Error) — semakin kecil semakin baik
Submission budget     : 25 total | Kuota harian: 3 sub/hari (periode 11–25 Juli)
Submissions used today: 0 / 3
Last updated          : 12 Juli 2026
Updated by            : Jeremy (post-EDA E1–E8)
```

---

## CURRENT STATUS

```
Active phase          : FASE 1 EDA → FASE 2 FE (transisi)
Last completed stage  : Jeremy E8 (Exploration Report — SENT TO ABABIL)
Next action           : Ababil mulai Stage 3 (Strategy) menggunakan Exploration Report
                        Jeremy standby untuk E9 atau delegasi dari Ababil
                        Vierico mulai B1 (Problem Brief)
Blocker (if any)      : NONE
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
```

### Kolom train.csv

```
datetime    str/datetime    Waktu observasi (format: YYYY-MM-DD HH:MM:SS)
nama_pos    str             Nama pos pemantauan (30 nilai unik)
tma_mdpl    float64         TARGET — Tinggi Muka Air (meter di atas permukaan laut)
```

### Kolom test.csv

```
id          str             Gabungan datetime + nama_pos
                            Format: "YYYY-MM-DD HH:MM:SS - Nama Pos"
                            Contoh: "2025-09-19 06:00:00 - Arjowinangun - Pacitan"
```

### Kolom data_lingkungan.csv (27 kolom)

```
# IDENTIFIER
datetime                 str/datetime    Waktu observasi per jam
nama_pos                 str             Nama pos (30 nilai unik, sama dengan train)

# CURAH HUJAN
rainfall_mm              float64         Curah hujan mm — IDENTIK dengan rainfall_openmeteo_mm (duplikat!)
rainfall_openmeteo_mm    float64         Curah hujan mm — IDENTIK dengan rainfall_mm (duplikat!) → DROP
rainfall_max_24h_mm      float64         Curah hujan maksimum rolling 24 jam (79.6% non-zero)

# ATMOSFER
humidity_pct             int64           Kelembapan udara (%) — range 16–100
wind_direction_deg       int64           Arah angin (°) — range 0–360
dew_point_c              float64         Titik embun (°C) — range 3.5–27.5
cloud_cover_pct          int64           Tutupan awan (%) — range 0–100
temperature_c            float64         Suhu udara 2m (°C) — range 17.5–38.8
wind_speed_kmh           float64         Kecepatan angin 10m (km/h) — range 0–39.2

# RADIASI
solar_radiation_mj_m2    float64         Radiasi surya (MJ/m²)
                                         ⚠️ SENTINEL -999 = missing
                                         Valid: Jan 2023 – 30 Des 2025
                                         -999: 31 Des 2025 – 18 Mei 2026
                                         Total -999: 100.080 baris (57.4% test period)
                                         → Replace -999 → NaN sebelum dipakai

# TANAH
soil_moisture_0_7cm      float64         Kelembapan tanah 0–7cm (m³/m³)
soil_moisture_7_28cm     float64         Kelembapan tanah 7–28cm (m³/m³)   ← within-pos corr terkuat +0.241
soil_moisture_28_100cm   float64         Kelembapan tanah 28–100cm (m³/m³)
soil_moisture_100_255cm  float64         Kelembapan tanah 100–255cm (m³/m³)
                                         Semua 4 kolom: missing 720 baris = cutoff artifact
                                         (30 pos × 24 jam di 2026-05-18, hari terakhir)

# TEKANAN
surface_pressure_hpa     float64         Tekanan permukaan (hPa)
                                         ⚠️ KORELASI SPURIOUS global -0.947 vs TMA
                                         Within-pos corr hanya -0.033 → proxy elevasi bukan kausal
                                         → Detrend per pos atau DROP
pressure_msl_hpa         float64         Tekanan MSL (hPa) — missing 720 baris (cutoff artifact)

# TUTUPAN LAHAN (STATIS PER POS — tidak berubah waktu)
built_surface_m2         float64         Luas terbangun (m²) — 1 nilai unik per pos
landcover_class          int64           Kelas: 10(Tree), 50(Built-up), 60(Water), 80(Cropland)
landcover_name           str             Tree cover 53%, Built-up 23%, Permanent water 20%, Cropland 3%

# INDEKS IKLIM MJO
rmm1                     float64         Komponen RMM1 MJO (harian) — range -4.18 to 2.98
rmm2                     float64         Komponen RMM2 MJO (harian) — range -2.32 to 3.11
mjo_phase                float64         Fase MJO (1–8)
mjo_amplitude            float64         Amplitudo MJO — within-pos corr +0.075
mjo_active               float64         MJO aktif: 1 (65.5%) / 0 (34.5%)
                                         Semua MJO: missing 720 baris = cutoff artifact

# INDEKS IKLIM ENSO
nino_34                  float64         Indeks ENSO Nino 3.4 (°C, bulanan)
                                         Missing: 12.960 baris = seluruh Mei 2026
                                         → Forward-fill atau impute untuk test period
```

### Kolom koordinat_pos.csv

```
nama_pos     str      Nama pos (30 nilai, sama dengan train/env)
latitude     float64  Koordinat lintang
longitude    float64  Koordinat bujur
```

### 30 Pos Pemantauan (sorted by mean TMA desc)

```
                          AC-lag1  mean TMA   obs    catatan
Ngadipiro                  0.804   143.6    2901
Ngrembang                  0.723   140.1    2901
Wonogiri Dam               0.998   132.3    2901   seasonal range 8.76 (terbesar)
Badegan                    0.851   122.4    2901
Colo Weir                  0.988   107.8    2903
Kali Pepe - Tugu Boto      0.611    94.8    2900
Peren                      0.047    91.3    2902   ⚠️ spike-prone
Jarum                      0.024    90.7    2864   ⚠️ spike-prone, gap berulang 2023
Sekayu                     0.878    87.3    2901
Kali Anyar - Kreteg Abang  0.036    86.5    2901   ⚠️ spike-prone, max 323.2
Serenan                    0.953    86.3    2901
Kali Pepe - PTPN           0.009    82.5    2901   ⚠️ spike-prone (AC terendah)
Jurug                      0.382    78.8    2903
Kedungupit                 0.233    64.9    2901
Kajangan                   0.225    51.0    2901
Ketonggo                   0.815    37.5    2901
Napel                      0.094    34.2    2901   ⚠️ spike-prone, max 325.8 (tertinggi)
Karangnongko               0.078    22.7    2901   ⚠️ spike-prone
Cepu                       0.976    17.5    2893
Brangkal                   0.895    13.0    2901
Lorog                      0.813    12.9    2901
Gunungsari                 0.904    10.0    1126   ⚠️⚠️ data sangat sedikit (39% normal)
Bengkelolor                0.955     9.8    2901
Bojonegoro - Kali Kethek   0.133     9.1    2688   ⚠️ data sedikit + AC rendah + spike
Sumberrejo                 0.974     7.5    2878
Babat                      0.964     6.3    2838
Boboh Kali Lamong          0.978     4.1    2902
Floodway Bridge C          0.553     3.9    2377   ⚠️ gap 163 hari di training
Karanggeneng               0.981     2.2    2903
Arjowinangun - Pacitan     0.624     1.1    2903

⚠️ = pos bermasalah | AC = autocorrelation lag-1 tma_mdpl
```

---

## EXPLORATION REPORT STATUS (Jeremy)

```
Jeremy stage saat ini : E8 DONE — Exploration Report SENT TO ABABIL
Exploration Report    : ✅ SENT TO ABABIL (11 Juli 2026)
Post-FE Report (E9)   : BELUM — standby menunggu delegasi dari Ababil
```

### Key Findings dari Jeremy (E1–E8)

```
FINDING 1 — TMA adalah 30 distribusi terpisah, bukan satu
  Tiap pos punya baseline elevasi unik (1.1–143.6 mdpl).
  Model global tanpa identitas pos akan bingung.

FINDING 2 — Dua karakter pos yang sangat berbeda
  STABIL (AC > 0.8, ~14 pos): Wonogiri Dam, Colo Weir, Karanggeneng, dst.
    → TMA hampir flat, lag kemarin sangat prediktif
  SPIKE (AC < 0.15, ~6 pos): Kali Pepe PTPN, Jarum, Kali Anyar, Peren, Napel, Karangnongko
    → TMA loncat-loncat, didorong event banjir, lag tidak prediktif
  CAMPURAN (0.15–0.8): Jurug, Kajangan, Kedungupit, Kali Pepe Tugu Boto, dll.

FINDING 3 — Soil moisture lapisan dangkal = prediktor terbaik (within-pos)
  soil_moisture_7_28cm   : corr +0.241 (terkuat)
  soil_moisture_0_7cm    : corr +0.229
  soil_moisture_28_100cm : corr +0.206
  dew_point_c            : corr +0.180
  rainfall_max_24h_mm    : corr +0.173
  humidity_pct           : corr +0.152

FINDING 4 — rainfall_mm == rainfall_openmeteo_mm (100% identik, duplikat)
  Salah satu harus di-drop.

FINDING 5 — Akumulasi hujan lebih penting dari intensitas sesaat
  rainfall_mm corr +0.052 vs rainfall_max_24h_mm corr +0.173
  Sungai butuh waktu respons — rolling window penting.

FINDING 6 — Gap sistemik Feb–Mar 2025 di hampir semua pos (~25 hari)
  Hampir semua 30 pos: gap 4 Feb → 1 Mar 2025 (outage sistemik).
  Lag features yang melewati gap ini akan corrupt.

FINDING 7 — Floodway Bridge C: gap 163 hari (Agu 2023 – Feb 2024)
  Pos termuda / paling sedikit data (2.377 obs).

FINDING 8 — Gunungsari: hanya 1.126 observasi (~39% pos lain)
  Dropout frekuent 18 jam sepanjang Agustus 2024.

FINDING 9 — Outlier ekstrem = event banjir nyata, bukan error
  Napel max 325.83 (5× baseline), Bojonegoro max 225.13 (6× baseline).
  JANGAN di-clip atau di-drop — ini target yang harus diprediksi.

FINDING 10 — 5 nilai anomali kecil di train
  4 nilai nol (Bojonegoro, Jurug, Kali Pepe PTPN, Karanggeneng)
  1 nilai negatif (Ketonggo -0.06) → semua sensor glitch.
```

### Hipotesis yang Sudah Divalidasi

```
H1 [HIGH]  Lag TMA = prediktor terkuat untuk pos stabil (AC > 0.8)
           Evidence: AC lag-1 Wonogiri Dam 0.998, Colo Weir 0.988
           Status: PENDING validasi Ababil

H2 [HIGH]  Soil moisture lapisan dangkal (0–28cm) = prediktor utama dinamika TMA
           Evidence: Within-pos corr SM 7-28cm = 0.241
           Status: PENDING validasi Ababil

H3 [HIGH]  Akumulasi hujan (rolling window) > intensitas sesaat
           Evidence: rainfall_max_24h corr 0.173 vs rainfall_mm 0.052
           Status: PENDING validasi Ababil

H4 [HIGH]  surface_pressure_hpa = proxy elevasi, bukan prediktor kausal
           Evidence: Global corr -0.947, within-pos corr -0.033
           Status: CONFIRMED — jangan pakai mentah (leakage risk)

H5 [MED]   Pos spike-prone butuh strategi berbeda dari pos stabil
           Evidence: AC lag-1 < 0.15 di 6 pos, spike ratio 3–9×
           Status: PENDING keputusan Ababil

H6 [HIGH]  Gap Feb–Mar 2025 akan corrupt lag features
           Evidence: ~25 hari gap di hampir semua pos bersamaan
           Status: CONFIRMED — perlu masking di pipeline

H7 [MED]   solar_radiation_mj_m2 tidak bisa dipakai langsung di test
           Evidence: 57.4% test period bernilai -999
           Status: CONFIRMED — replace -999 → NaN, lalu impute atau drop
```

### Risk Flags dari Jeremy

```
🔴 FLAG 1 [KRITIS] : surface_pressure_hpa — korelasi spurious -0.947 via elevasi
                     Jangan dipakai mentah. Detrend per pos atau drop.

🔴 FLAG 2 [KRITIS] : solar_radiation_mj_m2 — sentinel -999 di 57.4% test period
                     Train valid semua, test mayoritas -999 → mismatch parah.

🟠 FLAG 3 [TINGGI] : Gap sistemik Feb–Mar 2025 hampir semua pos (~25 hari)
                     Lag features yang melewati gap ini akan corrupt.

🟠 FLAG 4 [TINGGI] : Gunungsari hanya 1.126 obs (~39% pos lain)
                     Model mungkin underfit untuk pos ini.

🟠 FLAG 5 [TINGGI] : Floodway Bridge C gap 163 hari di training
                     Sangat sedikit data historis pos ini.

🟡 FLAG 6 [MED]    : Bojonegoro — data rendah + AC sangat rendah (0.133)
                     Prediksi pos ini paling tidak reliable.

🟡 FLAG 7 [MED]    : rainfall_openmeteo_mm — duplikat 100% rainfall_mm → drop.

🟡 FLAG 8 [MED]    : nino_34 missing seluruh Mei 2026 → forward-fill / impute.

🟡 FLAG 9 [LOW]    : 5 baris anomali train (4 zero + 1 negatif) → sensor glitch.
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

## FEATURE SET (Ababil)

```
Ababil stage saat ini : BELUM MULAI — menunggu membaca Exploration Report
FE status             : BELUM
Leakage check         : BELUM
Vierico FE review     : BELUM
```

**Kandidat fitur dari Jeremy (rekomendasi untuk Ababil):**

```
PRIORITAS TINGGI:
  tma_mdpl lag-1, lag-2, lag-3       — per pos, dengan masking di gap Feb-Mar 2025
  soil_moisture_0_7cm                — within-pos corr +0.229
  soil_moisture_7_28cm               — within-pos corr +0.241 (terkuat)
  soil_moisture_28_100cm             — within-pos corr +0.206
  rainfall_max_24h_mm                — within-pos corr +0.173
  dew_point_c                        — within-pos corr +0.180
  humidity_pct                       — within-pos corr +0.152
  nama_pos                           — identitas pos (label/target encode)

PRIORITAS SEDANG:
  cloud_cover_pct                    — within-pos corr +0.121
  rolling rainfall 48h, 72h          — perlu dibuat dari rainfall_mm
  mjo_amplitude                      — within-pos corr +0.075
  mjo_phase, rmm1, rmm2              — sinyal lemah, mungkin berguna sebagai modulator
  wind_direction_deg                 — within-pos corr +0.080
  koordinat lat/lon                  — karakteristik spasial pos

PERLU TREATMENT KHUSUS SEBELUM PAKAI:
  surface_pressure_hpa               — detrend per pos atau DROP (proxy elevasi)
  solar_radiation_mj_m2             — replace -999 → NaN dulu, lalu impute/drop
  nino_34                            — forward-fill untuk Mei 2026
  soil_moisture_100_255cm            — global corr tinggi tapi karena elevasi

DROP KANDIDAT:
  rainfall_openmeteo_mm              — 100% duplikat rainfall_mm
  built_surface_m2                   — statis per pos (sudah terwakili oleh nama_pos)
  landcover_class / landcover_name   — statis per pos (idem)
```

**Features yang sudah di-approve:**

| Feature | Type | Source | Leakage Check | Vierico | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| [belum ada] | | | | | |

**Features yang di-drop:**
```
- [belum ada — menunggu keputusan Ababil]
```

---

## EXPERIMENT LOG SUMMARY

```
Anchor model (Slot 1) : [belum ada]
Anchor model (Slot 2) : [belum ada]
Best CV so far        : [belum ada]
Best LB so far        : [belum ada]
```

**Recent experiments:**

| exp_id | Stage | Model | CV | LB | Delta | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| [belum ada] | | | | | | |

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
⚠️ Gap Feb–Mar 2025 TIDAK BOLEH jadi fold boundary (corrupt lag features)
```

---

## PENDING ACTIONS

```
[HIGH] Ababil   — baca Exploration Report (E8) dan mulai Stage 3 (Strategy)
[HIGH] Ababil   — putuskan treatment: solar_radiation (-999 → NaN → impute/drop?)
[HIGH] Ababil   — putuskan treatment: surface_pressure (detrend per pos / drop?)
[HIGH] Ababil   — putuskan: masking lag features di gap Feb–Mar 2025
[MED]  Ababil   — putuskan: drop rainfall_openmeteo_mm (duplikat)
[MED]  Ababil   — putuskan: impute nino_34 untuk Mei 2026
[MED]  Ababil   — putuskan: 5 baris anomali train → impute atau drop
[MED]  Vierico  — mulai B1 Problem Brief
[LOW]  Jeremy   — standby E9 atau delegasi dari Ababil
```

---

## CONTEXT RESET PROTOCOL

Saat ganti akun Claude (limit habis), lakukan urutan ini:

**Untuk Ababil:**
1. Buka akun baru
2. Pesan 1: paste isi `agents/AGENTS_ababil.md`
3. Pesan 2: paste isi file ini (`project_state.md`)
4. Pesan 3: "Lanjutkan dari [stage terakhir Ababil]. Semua context ada di atas."

**Untuk Jeremy:**
1. Buka akun baru
2. Pesan 1: paste isi `agents/AGENTS_jeremy.md`
3. Pesan 2: paste isi file ini (`project_state.md`)
4. Pesan 3: "Lanjutkan dari E9 / standby delegasi. Semua context ada di atas."

**Untuk Vierico:**
1. Buka akun baru
2. Pesan 1: paste isi `agents/AGENTS_vierico.md`
3. Pesan 2: paste isi file ini (`project_state.md`)
4. Pesan 3: "Lanjutkan dari checkpoint [B-sekian]. Semua context ada di atas."

**Jangan skip langkah ini.** Tanpa project_state, Claude mulai dari nol.

---

## CATATAN BEBAS

```
11 Juli 2026 (Tim)    : Project state diinisialisasi. Kompetisi baru mulai ~13 jam lalu.
11 Juli 2026 (Jeremy) : EDA E1–E7 selesai. Semua temuan, risk flags, dan hipotesis
                        terdokumentasi. Key insight: 30 pos = 30 karakter berbeda.
                        Pos stabil butuh lag, pos spike butuh fitur curah hujan.
                        surface_pressure jangan dipakai mentah.
                        solar_radiation bermasalah di test period.
11 Juli 2026 (Jeremy) : E8 Exploration Report selesai dan dikirim ke Ababil.
                        Jeremy standby untuk E9 atau delegasi.
                        Bola ada di tangan Ababil (Stage 3) dan Vierico (B1).
```