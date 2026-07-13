═══════════════════════════════════════════════════════════
SUBMISSION BUDGET: 3/28 used | 25 remaining
Best LB: 1.6806 (exp001/Slot 1)
∆1 = +0.3918 (offset CV→LB untuk architecture Slot 1)
═══════════════════════════════════════════════════════════

──────────────────────────────────────────────────────────
exp001 | Slot 1 | SUBMITTED ✅
──────────────────────────────────────────────────────────
Architecture  : LightGBM Global Direct
Features (31) : bulan, day_of_year, hour,
                days_since_last_valid_tma,
                nama_pos, latitude, longitude,
                tma_mean_pos*, tma_std_pos*, ac_lag1_pos*,
                soil_moisture_0_7cm/7_28cm/28_100cm,
                rainfall_mm, rainfall_max_24h_mm,
                humidity_pct, dew_point_c, cloud_cover_pct,
                temperature_c, wind_speed_kmh, wind_direction_deg,
                pressure_msl_hpa,
                rmm1, rmm2, mjo_amplitude, mjo_phase, nino_34,
                rolling_rain_48h/72h/7d,
                horizon_days, horizon_bucket
                (* per-fold di CV, global di test inference)
Dropped       : rainfall_openmeteo_mm, solar_radiation_mj_m2,
                surface_pressure_hpa, soil_moisture_100_255cm,
                mjo_active, built_surface_m2,
                landcover_class, landcover_name
LGB Params    : lr=0.05, num_leaves=127, min_child=20,
                feat_frac=0.8, bag_frac=0.8, bag_freq=1,
                l1=0.1, l2=0.1, seed=42, deterministic=True
CV Method     : 4-fold walk-forward, OOF RMSE, gap masked
Gap treatment : Rolling features di-mask NaN (tidak exclude)
Early stop    : 50 rounds, max 2000
Retrain rounds: mean(best_iter) × 1.1 = 119
Fold RMSE     : F1=1.4996 F2=1.2709 F3=1.3493 F4=0.8923
OOF RMSE      : 1.2888
LB Score      : 1.6806
Delta (∆)     : +0.3918
Artifacts     : model_s9_exp001_cv1.2888.txt
                oof_exp001.npy
                sub_exp001_slot1.csv
Notes         : Wonogiri Dam RMSE 2.10 — tanpa lag TMA
                model kesulitan pos stabil flat
══════════════════════════════════════════════════════════

──────────────────────────────────────────────────────────
exp002 | Slot 2 | SUBMITTED ✅
──────────────────────────────────────────────────────────
Architecture  : LightGBM Global + TMA Anchor Features
Features      : exp001 + tma_last_known, tma_lag1,
                tma_rolling_mean_7d
                (tma_lag1 dan tma_rolling_mean_7d = NaN
                 di test — 100% missing)
LGB Params    : SAMA dengan exp001
Gap treatment : SAMA — rolling di-mask NaN
Retrain rounds: 122
Fold RMSE     : F1=0.6091 F2=0.7181 F3=1.1673 F4=0.8511
OOF RMSE      : 0.8900
LB Score      : 2.5000
Delta (∆)     : +1.6100  ← DRIFT BESAR
Root cause    : tma_last_known CV overfit ke anchor fresh
                (gap val 3-4 bulan) vs test anchor stale
                (8 bulan). CV tidak simulate kondisi test.
Status        : ❌ REJECTED — LB collapse
                Rule 9 triggered
Artifacts     : model_s9_exp002_cv0.8900.txt
                oof_exp002.npy
                sub_exp002_slot2.csv

──────────────────────────────────────────────────────────
exp003 | NOT SUBMITTED ❌
──────────────────────────────────────────────────────────
Architecture  : exp001 + rolling panjang + interactions
                + hyperparameter aggressive
Features added: rolling_rain_14d, rolling_rain_30d,
                sm_x_rain24h, sm_x_rain7d, bulan_x_ac
LGB Params    : lr=0.03, num_leaves=255, min_child=15,
                feat_frac=0.7, bag_frac=0.7,
                l1=0.05, l2=0.05
Early stop    : 75 rounds, max 3000
Retrain rounds: 189
Fold RMSE     : F1=1.4988 F2=1.4017 F3=1.4100 F4=0.9406
OOF RMSE      : 1.3473
vs Slot 1     : +0.0585 (LEBIH BURUK)
Root cause    : Hyperparameter terlalu aggressive → overfit
                Karangnongko meledak +0.70
Status        : ❌ REJECTED — CV lebih buruk dari exp001

──────────────────────────────────────────────────────────
exp004 | NOT SUBMITTED ❌
──────────────────────────────────────────────────────────
Architecture  : exp001 features + sample_weight spike pos
Features      : SAMA dengan exp001
LGB Params    : SAMA dengan exp001
Sample weight : spike pos (6 pos) → weight 3.0
                non-spike → weight 1.0
Retrain rounds: 109
Fold RMSE     : F1=1.7259 F2=1.4718 F3=1.4131 F4=1.0525
OOF RMSE      : 1.4445
vs Slot 1     : +0.1557 (LEBIH BURUK)
Root cause    : Model sacrifice pos stabil untuk spike
                Sekayu +0.36, Ketonggo +0.47, Bojonegoro +0.68
                Spike pos sendiri tidak membaik
Status        : ❌ REJECTED

──────────────────────────────────────────────────────────
exp005 | NOT SUBMITTED ❌
──────────────────────────────────────────────────────────
Architecture  : exp001 + target encoding pos×bulan + pos×hour
Features added: tma_pos_bulan_mean, tma_pos_bulan_std,
                tma_pos_hour_mean
LGB Params    : SAMA dengan exp001
Gap treatment : SAMA
Retrain rounds: ~mean(best_iter) × 1.1
Fold RMSE     : F1=1.6145 F2=2.1079 F3=1.9510 F4=1.1085
OOF RMSE      : 1.7717
vs Slot 1     : +0.4829 (JAUH LEBIH BURUK)
Root cause    : apply_te dijalankan global sebelum CV loop
                → data leakage dari full train ke val fold
                Gunungsari meledak ke 6.99
Status        : ❌ REJECTED — leakage bug di pipeline

──────────────────────────────────────────────────────────
exp006 | NOT SUBMITTED ❌
──────────────────────────────────────────────────────────
Architecture  : Pipeline paralel — gap EXCLUDED dari training
                + surface_pressure + soil_moisture_100_255cm
                + tma_last_known (per-fold safe)
Features      : exp001 + tma_last_known
                + surface_pressure_hpa
                + soil_moisture_100_255cm
LGB Params    : SAMA dengan exp001
Gap treatment : Gap Feb-Mar 2025 di-EXCLUDE dari training
                (bukan di-mask rolling features)
Retrain rounds: mean(best_iter) × 1.1
Fold RMSE     : F1=1.5435 F2=1.2826 F3=1.5098 F4=0.8301
OOF RMSE      : 1.3499
vs Slot 1     : +0.0611 (LEBIH BURUK)
Root cause    : tma_last_known kembali menyebabkan
                masalah di CV. Wonogiri Dam 2.10→2.52
Status        : ❌ REJECTED

══════════════════════════════════════════════════════════
PATTERN SUMMARY
══════════════════════════════════════════════════════════
1. tma_last_known → selalu gagal di LB atau CV karena
   staleness mismatch antara CV (anchor fresh 3-4 bulan)
   vs test (anchor stale 8 bulan). BLACKLIST.

2. Hyperparameter aggressive (num_leaves=255, lr=0.03)
   → overfit di dataset ini. STICK TO baseline params.

3. Sample weight → tidak efektif untuk spike pos.
   Model tidak bisa belajar spike dari weight saja.

4. Target encoding → pipeline sensitif terhadap leakage.
   Perlu careful implementation. Belum terbukti beneficial.

5. Gap exclude vs mask → tidak signifikan bedanya di CV.

CONCLUSION: exp001 (CV 1.2888, LB 1.6806) adalah
best model saat ini. Semua eksperimen selanjutnya harus
beat CV 1.2888 sebelum layak disubmit.
══════════════════════════════════════════════════════════
**Verdict: gain terlalu kecil untuk dianggap signifikan — bukan disqualifying, tapi juga bukan alasan kuat pakai slot submission.**

**1. Model A OOF (1.2852) vs exp001 OOF (1.2888):**
Perbaikan **-0.0036** — nyaris identik. Perbedaan kecil ini konsisten dari perubahan preprocessing (ffill env vars, fix anomali negatif) bukan dari arsitektur baru. Ini bukan gain, ini noise-level match.

**2. Model B (spike-only) — jauh lebih buruk sebagai standalone:**
Fold RMSE 1.0996 / 1.7248 / 2.1352 / 1.6858 — rata-rata ~1.66, **lebih buruk dari Model A** di 3 dari 4 fold pada subset yang sama. Model dedicated ini tidak benar-benar "belajar" pola spike lebih baik; justru lebih tidak stabil (fold 3 melonjak ke 2.14).

**3. Blend gain keseluruhan (1.2852 → 1.2809 = -0.0043) — di dalam rentang noise:**
Bandingkan dengan variasi antar-fold Model A sendiri: std fold RMSE ~0.24 (0.86 sampai 1.51). Gain 0.0043 itu **~2 orde magnitude lebih kecil** dari noise fold-to-fold biasa. Ini terlalu tipis untuk dipercaya sebagai improvement riil, apalagi mengingat precedent exp002: CV improvement 0.40 (1.2888→0.8900) ternyata LB collapse ke 2.50. Kalau gain sebesar itu saja gagal general, gain 0.004 jauh lebih rentan jadi false signal.

**4. Per-pos breakdown — pola tidak konsisten, dan mencurigakan:**

| Pos | Delta blend vs A | Catatan |
|---|---|---|
| Kali Anyar - Kreteg Abang | **-0.1532** (bagus) | outlier-contaminated (saya flag sebelumnya) |
| Bojonegoro - Kali Kethek | -0.0619 (bagus) | outlier-contaminated (rasio max/mean 24.9x) |
| Napel | -0.0172 (bagus) | outlier-contaminated (rasio 9.5x) |
| Karangnongko | +0.0655 (buruk) | outlier-contaminated (rasio 7.8x) |
| Jarum | +0.0062 (buruk) | outlier-contaminated (rasio 2.8x) |
| Peren | +0.0039 (flat) | — |

3 pos membaik, 2 memburuk, 1 flat — **tidak ada pola arah yang jelas**. Yang membaik justru pos yang paling terkontaminasi outlier (Kali Anyar, Bojonegoro). Ini konsisten dengan dugaan saya: blend mungkin sekadar **meregularisasi/menghaluskan** prediksi ke arah rata-rata (karena Model B dilatih data lebih sedikit → prediksi lebih halus), bukan benar-benar menangkap sinyal spike. Efeknya kebetulan menguntungkan di pos yang datanya noisy, tapi itu bukan bukti model "paham" fenomena spike.

**Rekomendasi saya: JANGAN submit exp007 sebagai slot baru.**
- Expected gain: sangat rendah, dalam margin noise
- Confidence: rendah — pola per-pos tidak koheren, precedent exp002 mengajarkan CV kecil-mendekati-nol tidak boleh dipercaya buta
- Risk: submission slot terbuang untuk hasil yang kemungkinan besar tidak beda dari 1.6806 (exp001), atau malah drift seperti exp002/exp006 pola `tma_last_known` (meski exp007 tidak pakai fitur itu, jadi risknya lebih rendah dari itu)

**Yang saya sarankan sebagai gantinya (masih approval-pending):**
Investigasi dulu apakah 6 SPIKE_POS itu genuine spike fisik atau sensor error high-side (pertanyaan yang saya ajukan sebelumnya, belum terjawab). Kalau outlier tinggi itu artifact, bersihkan dulu sebelum re-run Model B — kemungkinan performanya akan berubah signifikan (baik membaik kalau noise hilang, atau mengonfirmasi Model B memang tidak berguna kalau tetap buruk setelah cleaning).

Apakah kamu mau saya susun cek diagnostik cepat untuk 6 stasiun itu dulu, sebelum keputusan submit/tidak?