# TP3 - Life Expectancy Clustering

Proyek ini bertujuan untuk mengelompokkan negara-negara berdasarkan profil kesehatan dan sosio-ekonomi (angka harapan hidup, mortalitas, imunisasi, GDP, pendidikan, dll.) menggunakan pendekatan *unsupervised learning* (clustering). Hasil pengelompokan diharapkan dapat membantu memetakan negara yang membutuhkan prioritas bantuan internasional. Kualitas cluster dievaluasi terutama menggunakan metrik **Silhouette Score**, dengan **Davies-Bouldin Index** sebagai pembanding.

## 👥 Pembagian Tugas Kelompok

Berikut adalah pembagian eksplorasi dan pemodelan machine learning untuk masing-masing anggota tim:

| Nama Anggota | Algoritma Clustering | Fokus Eksperimen & Pemodelan |
|---|---|---|
| Anderson Tirza | Hierarchical Clustering | Agglomerative clustering, pemilihan metode linkage, imputasi (KNN/Simple), scaling, dan optimasi via Optuna |
| Raymundo Rafaelito | Bisecting K-Means | Varian K-Means berbasis pembagian cluster secara hierarkis (top-down), Elbow & Silhouette analysis, serta pembandingan dengan K-Means, DBSCAN, dan Agglomerative |
| Angelo Benhanan | GMM Clustering | Gaussian Mixture Model (probabilistik) dengan reduksi dimensi PCA, pemilihan tipe kovarians, dan optimasi via Optuna |
| Jovanus Irwan | K-Means | Clustering berbasis centroid dengan fitur komposit (Socio-Economic Index & Health Risk Index), penentuan K via Elbow Method dan Silhouette Score |

## 📊 Deskripsi Dataset

Dataset yang digunakan dalam proyek ini:

- `train.csv` — Berisi 10.070 baris dan 21 kolom (setelah pembersihan baris tanpa nilai `Life expectancy`: 10.062 baris).
- `test.csv` — Berisi 1.259 baris dan 19 kolom (dimuat pada bagian Hierarchical Clustering).

Kolom utama meliputi `Year`, `Status` (Developed/Developing), `Life expectancy`, `Adult Mortality`, `infant deaths`, `under-five deaths`, `HIV/AIDS`, `Measles`, imunisasi (`Hepatitis B`, `Polio`, `Diphtheria`), `Alcohol`, `BMI`, `thinness`, `GDP`, `percentage expenditure`, `Total expenditure`, `Schooling`, `Income composition of resources`, dan `Population`.

### Temuan Awal (Exploratory Data Analysis)

- **Missing Values:** Terdapat beberapa kolom dengan nilai kosong. Selain itu, nilai `0` pada `Hepatitis B` sebelum tahun 1982 diperlakukan sebagai missing karena penyakit tersebut belum diuji pada masa itu.
- **Distribusi & Outlier:** Fitur seperti `GDP`, `HIV/AIDS`, dan `infant deaths` sangat *right-skewed* dan memiliki banyak outlier ekstrem, sehingga diperlukan log-transform dan/atau `RobustScaler`.
- **Preston Curve (GDP vs Life Expectancy):** Peningkatan GDP meningkatkan harapan hidup, tetapi kenaikannya melandai (*diminishing returns*). Negara Developed dan Developing menunjukkan perbedaan yang jelas namun masih ada area tumpang tindih.
- **Pengeluaran Kesehatan:** Korelasi Spearman antara `percentage expenditure` dan `Life expectancy` sebesar ±0,40 (hubungan positif sedang), dengan titik jenuh sekitar ±3.100 USD berdasarkan deteksi *knee point* pada kurva LOWESS.
- **Proyeksi PCA 2D:** Negara Developed dan Developing terpisah cukup terlihat pada sumbu PC1 ("sumbu kesejahteraan"), tetapi tidak sempurna.

## ⚙️ Alur Pengerjaan (Workflow)

1. **Load Data & Imports:** Memuat dataset serta mengimpor pustaka analisis data dan pemodelan (pandas, numpy, scikit-learn, optuna, kneed, statsmodels, dll.).
2. **Exploratory Data Analysis (EDA):** Menganalisis missing values, distribusi dan outlier fitur, korelasi antar fitur, perbandingan indikator berdasarkan status negara, serta proyeksi PCA 2D.
3. **Data Preprocessing & Feature Engineering:**
   - Penanganan nilai kosong dan duplikat (imputasi `SimpleImputer` median/mean atau `KNNImputer`).
   - Penskalaan fitur (`StandardScaler` / `RobustScaler`) dan encoding variabel kategorikal.
   - Log-transform pada fitur yang skewed, serta pembuatan fitur komposit (misalnya `mortality_burden`, `human_capital`, `Total Deaths`, `Socio-Economic Index`, `Health Risk Index`).
4. **Modeling & Penentuan Jumlah Cluster:**
   - Membangun masing-masing model clustering sesuai pembagian tugas kelompok.
   - Menentukan jumlah cluster dan hyperparameter terbaik dengan Elbow Method, Silhouette Analysis, dan Optuna untuk memaksimalkan Silhouette Score.
5. **Evaluasi & Interpretasi:** Menghitung Silhouette Score dan Davies-Bouldin Index, memvisualisasikan cluster (PCA 2D dan silhouette diagram), serta membuat profil karakteristik tiap cluster (termasuk komposisi Developed vs Developing).

## 📈 Ringkasan Hasil

| Algoritma | Konfigurasi Terbaik | Silhouette Score |
|---|---|---|
| Hierarchical Clustering | `n_clusters=2`, linkage `average`, KNN imputer (k=16), RobustScaler | 0,7563 |
| Bisecting K-Means | `K=2`, RobustScaler, imputasi median | 0,9174 |
| GMM | `n_components=2`, kovarians `tied`, PCA 3 komponen, RobustScaler, imputasi median | 0,7596 (DBI: 0,4840) |
| K-Means | `K=2`, fitur komposit (Socio-Economic & Health Risk Index), RobustScaler | 0,7274 |

> **Catatan:** Nilai Silhouette tidak sepenuhnya sebanding antar algoritma karena preprocessing dan set fitur yang digunakan masing-masing anggota berbeda.

Pada semua model, jumlah cluster optimal adalah **2**: satu kelompok besar (negara dengan kondisi relatif stabil) dan satu kelompok kecil yang rentan (GDP rendah, mortalitas tinggi, imunisasi dan nutrisi lemah) yang dapat diprioritaskan untuk bantuan internasional.

## 🛠️ Cara Menjalankan Proyek

Pastikan Python 3.10+ sudah terpasang beserta pustaka yang dibutuhkan:

```bash
pip install numpy pandas matplotlib seaborn scikit-learn scipy optuna kneed statsmodels
```

Letakkan `train.csv` (dan `test.csv`) di direktori yang sama dengan notebook, lalu jalankan:

```bash
jupyter notebook TP3_B06_JMK48.ipynb
```
