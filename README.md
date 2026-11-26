# Fik-Fair-Competition
Prediksi Anonymous Competition Score (ACS)
Memprediksi skor biaya hidup relatif antarwilayah dari 24 fitur anonim, menggunakan alur kerja CRISP-DM dan validasi silang 5-fold.

Data Science Competition — FIK FAIR 2026

Python scikit-learn TabICL Metode

Ringkasan Proyek
Kompetisi ini meminta peserta memprediksi Anonymous Competition Score (ACS) — sebuah skor biaya hidup relatif antarwilayah yang sudah dianonimkan. Semakin tinggi ACS, semakin tinggi biaya hidup relatif yang diwakilinya.

Ini adalah masalah regresi: belajar dari data berlabel, lalu menghasilkan satu prediksi angka untuk tiap record uji. Karena identitas wilayah dan arti asli setiap fitur tidak diberikan, fokus proyek adalah pola data dan ketepatan prediksi, bukan menebak wilayah atau menyimpulkan sebab-akibat.

Tipe masalah	Regresi
Target	ACS (skor relatif, tanpa satuan uang)
Data latih	382 record (berlabel)
Data uji	96 record (label disembunyikan)
Fitur	24 fitur numerik anonim (feature_01–feature_24)
Metrik utama	RMSE (makin kecil makin baik)
Metrik pendukung	MAE, R²
Alur Kerja (CRISP-DM)
Proyek mengikuti enam tahap CRISP-DM agar mudah diaudit dan dapat dibuat ulang:

Business Understanding → Data Understanding → Data Preparation → Modeling → Evaluation → Deployment

Business Understanding — menetapkan target, metrik, dan aturan penggunaan data.
Data Understanding — audit kualitas data (nilai kosong, ID ganda, angka tidak valid), distribusi target, korelasi fitur, dan PCA.
Data Preparation — memisahkan 24 fitur dari record_id dan target; standardisasi serta cabang PCA sesuai kebutuhan model.
Modeling — membandingkan 7 pendekatan dengan validasi 5-fold.
Evaluation — menilai kesalahan secara keseluruhan dan per kelompok ACS.
Deployment — melatih ulang pada seluruh data latih dan memverifikasi 96 prediksi uji.
Metodologi
Pemodelan. Tujuh pendekatan dibandingkan: baseline (prediksi rata-rata), Ridge, Extra Trees, SVR standar, gabungan SVR, TabICL, dan gabungan akhir. Model sederhana dipakai sebagai patokan agar manfaat model yang lebih rumit terlihat jelas.

Model akhir — gabungan berbobot:

40% gabungan SVR (lima cabang: sebagian memakai bobot target tinggi, jarak berbasis korelasi, dan PCA 12/16/20 komponen)
60% TabICL (model pretrained untuk data tabular, 16 estimator, seed 2026)
Validasi. KFold 5-bagian dengan pengacakan (seed 314159). Setiap record latih memperoleh satu prediksi saat ia tidak dipakai melatih model — prediksi out-of-fold (OOF) — sehingga penilaian tidak bergantung pada satu pembagian data saja. Label bagian validasi tidak pernah dipakai untuk melatih model.

Hasil
Validasi 5-fold pada data latih:

Model	RMSE	MAE
Baseline (rata-rata)	12,9442	—
Gabungan akhir (40% SVR + 60% TabICL)	7,6194	5,1723
RMSE turun ~41% dibanding baseline rata-rata.
R² OOF ≈ 0,65; rata-rata residual ≈ +0,37 (nyaris tak bias secara keseluruhan).
Rentang prediksi data uji: 33,89 – 86,52.
96 prediksi uji lolos verifikasi format, ID, nilai numerik, dan reproduksibilitas (dicek dengan toleransi 1e-4 dan sidik SHA-256).
TabICL sendiri punya MAE sedikit lebih kecil, tetapi gabungan akhir unggul pada RMSE — metrik resmi kompetisi — sehingga gabungan akhir yang dipilih.

Temuan & Interpretasi
1. Model jauh lebih baik daripada tebakan sederhana. Penurunan RMSE ~41% terhadap baseline menunjukkan 24 fitur memang memuat sinyal yang bisa dipelajari.

2. Prediksi ACS tinggi masih jadi kelemahan. Pada kelompok ACS ≥ 75 (hanya 19 record), RMSE melonjak ke ~20,7 dengan bias ~−18,2 — artinya nilai tinggi cenderung diprediksi terlalu rendah. Pembobotan pada salah satu SVR belum sepenuhnya mengatasinya. Dengan sampel sekecil ini, hasilnya perlu dibaca hati-hati.

3. Pola fitur bisa dipelajari, tetapi maknanya terbatas. PCA menunjukkan 4 komponen pertama merangkum ~91% variasi data. Namun karena fitur anonim, hubungan statistik tidak bisa diartikan sebagai penyebab ekonomi tertentu.

4. ACS adalah skor relatif, bukan rupiah. Hasil model tidak bisa langsung dipakai sebagai besaran anggaran atau keputusan operasional.

Teknologi
Python, pandas, NumPy, Matplotlib
scikit-learn — SVR, Ridge, Extra Trees, PCA, StandardScaler, KFold
TabICL — model pretrained untuk data tabular
Struktur Repositori
.
├── notebook_prediksi_acs.ipynb   # Notebook utama (CRISP-DM, kode model tertanam)
├── data/
│   ├── train.csv                 # 382 record berlabel
│   ├── test.csv                  # 96 record uji
│   └── data_dictionary.csv
├── hasil/                        # Ringkasan metrik & prediksi OOF (JSON/NPZ)
├── assets/                       # Grafik hasil eksplorasi & evaluasi (PNG)
└── submission.csv                # 96 prediksi akhir
Cara Menjalankan
Notebook dirancang mandiri — seluruh kode model tertanam di dalamnya.

Buka notebook di Google Colab.
Pilih Runtime → Run all.
Unggah train.csv dan test.csv resmi bila diminta.
Tunggu validasi 5-fold dan pelatihan akhir selesai (beberapa menit di CPU; TabICL mengunduh checkpoint publik pada eksekusi pertama).
submission.csv dibuat, diperiksa otomatis, dan ditawarkan untuk diunduh.
Butuh koneksi internet untuk instalasi dependensi dan unduhan checkpoint TabICL.

Catatan
Hanya feature_01–feature_24 yang menjadi prediktor; record_id hanya untuk mencocokkan hasil, bukan fitur.
Nilai validasi di notebook adalah perkiraan kemampuan model, bukan skor private leaderboard.
Karena fitur dan target anonim, hubungan statistik tidak dianggap sebagai hubungan sebab-akibat.
