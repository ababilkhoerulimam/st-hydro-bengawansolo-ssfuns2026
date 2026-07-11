Cukup data dari jurnal. Ini rekomendasi lengkapnya berdasarkan karakteristik dataset kalian:

---

## Rekomendasi Model & Metode — TMA BBWS Bengawan Solo

*(Berdasarkan literature review dari 20+ jurnal 2023–2025)*

---

### 🥇 Tier 1 — Model Utama (wajib dicoba)

**1. Global LightGBM + Lag Features + Station ID**

Ini rekomendasi utama untuk kompetisi ini. Alasannya:

Studi 2024–2025 menunjukkan bahwa model LightGBM global yang menyertakan station identifier secara konsisten mengungguli model per-stasiun maupun model cluster, karena mampu memanfaatkan seluruh data lintas pos sekaligus melakukan penyesuaian lokal melalui identitas pos.

Pada prediksi tinggi muka air di Lago de Chapala Dam, Gradient Boosting Regressor dengan fitur lag 1-hari mengurangi RMSE dari 886 menjadi 32 — penurunan dramatis yang menegaskan betapa krusialnya lag features untuk prediksi hidrologi.

Setup yang direkomendasikan:
```
Model     : LightGBM (global, satu model untuk 30 pos)
Features  : tma lag-1/2/3, soil moisture, rainfall rolling,
            nama_pos (label/target encode), koordinat, bulan, jam
CV        : TimeSeriesSplit
Tuning    : Optuna (Bayesian search)
```

---

**2. LSTM Multivariate (per-cluster atau global)**

Untuk prediksi tinggi muka air sungai multivariate dan multi-horizon, LSTM terbukti memberikan hasil terbaik dibandingkan baseline, model linear, maupun MLP — karena model klasikal seperti ARIMA tidak bisa menangani time series multivariat.

LSTM mengungguli SVM dan XGBoost untuk *high flows* (event banjir ekstrem), meskipun XGBoost lebih baik untuk *low flows* — artinya kedua model saling melengkapi.

Setup yang direkomendasikan:
```
Model     : LSTM atau BiLSTM
Input     : sequence 24–72 jam terakhir (window lookback)
Features  : soil moisture, rainfall, TMA lag
Kluster   : pisahkan pos stabil (AC > 0.8) vs pos spike (AC < 0.15)
```

---

### 🥈 Tier 2 — Model Pendukung / Ensemble

**3. XGBoost (sebagai pembanding & ensemble partner)**

XGBoost mencapai akurasi terbaik (R² = 0.904) dibandingkan LSTM dan Random Forest untuk prediksi runoff satu langkah ke depan, dengan kemampuan mendeteksi turning point 22–34% lebih baik.

Berguna terutama untuk pos-pos stabil dengan AC tinggi.

**4. Stacking Ensemble: LightGBM + LSTM**

Studi 2024 pada prediksi inflow reservoir India menggunakan stacking ensemble dari BiLSTM, CNN, XGBoost, LightGBM, dan Random Forest dengan input: curah hujan, evaporasi, lag inflow, dan indeks iklim — dan menunjukkan bahwa kombinasi ini mengungguli model tunggal manapun.

Ensemble TCN-LSTM-LightGBM mengurangi MAE rata-rata 28.95% dibandingkan LightGBM saja, dan 4.88% dibandingkan TCN-LSTM saja.

---

### 🧠 Insight Kritis untuk Karakteristik Dataset Ini

| Karakteristik Data | Implikasi Model |
|---|---|
| 30 pos heterogen (AC 0.009–0.998) | Global model + pos ID > per-pos model |
| Pos stabil (AC > 0.8) | Lag TMA sangat prediktif → LightGBM kuat |
| Pos spike (AC < 0.15) | Lag lemah → LSTM lebih baik tangkap event |
| Soil moisture corr +0.24 | Wajib masuk sebagai fitur utama |
| Gap Feb–Mar 2025 | Mask lag yang melewati gap |
| Test horizon 8 bulan ke depan | Multi-step: recursive strategy atau direct |
| RMSE sebagai metric | Model sensitif error besar → prioritaskan prediksi spike |

---

### 🗺️ Strategi yang Direkomendasikan (urutan eksperimen)

```
EXP 1 — Baseline: Global LightGBM + lag-1/2/3 + nama_pos
         → Ukur RMSE baseline, cepat, interpretable

EXP 2 — LightGBM + full features (soil moisture, rainfall rolling, dll)
         → Ukur gain dari fitur eksogen

EXP 3 — LSTM per-cluster (stabil vs spike)
         → Lihat apakah LSTM lebih baik untuk pos spike

EXP 4 — Ensemble: weighted average LightGBM + LSTM
         → Biasanya +1–3% RMSE improvement

EXP 5 — Stacking: LightGBM & LSTM sebagai base, Ridge/LGB sebagai meta
         → Final submission candidate
```

---

### 📚 Referensi Jurnal Kunci

| Jurnal | Temuan Relevan |
|---|---|
| Springer 2024 (Lago de Chapala) | GBR + lag-1: RMSE 886→32 |
| IWA Hydroinformatics 2024 | Stacking BiLSTM+LGBM+XGB terbaik untuk inflow reservoir |
| HESS 2025 | LSTM unggul untuk high flows, XGBoost untuk low flows |
| MDPI Water 2025 | PatchTST unggul multi-step, LSTM unggul single-step |
| arXiv 2025 (multi-site LightGBM) | Global LightGBM + station ID > per-station model |
| ScienceDirect 2024 | TCN-LSTM-LightGBM ensemble: MAE turun 28.95% vs LGB alone |

---

Mau aku update ini ke `project_state.md` sekarang?