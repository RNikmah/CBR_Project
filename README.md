# Proyek Sistem Case-Based Reasoning (CBR) untuk Analisis Putusan Pengadilan

Proyek ini merupakan implementasi sistem *Case-Based Reasoning* (CBR) sederhana yang dirancang untuk mendukung analisis putusan pengadilan di Indonesia, sesuai dengan deskripsi proyek Ujian Akhir Semester mata kuliah Penalaran Komputer. Sistem ini dibangun menggunakan Python dan memanfaatkan data putusan yang diambil dari Direktori Putusan Mahkamah Agung Republik Indonesia.

Sistem ini mengimplementasikan siklus CBR yang mencakup tahapan *case base building* (pengumpulan dan pembersihan data), *case representation* (ekstraksi fitur dan metadata), *case retrieval* (pencarian kasus serupa), *solution reuse* (prediksi solusi), dan *model evaluation* (evaluasi performa). Secara khusus, pada tahap *retrieval*, sistem ini mengimplementasikan dan membandingkan dua pendekatan:
1.  **Pendekatan Statistik**: Menggunakan TF-IDF dan Cosine Similarity.
2.  **Pendekatan Semantik**: Menggunakan *Text Embedding* dari model Transformer (IndoBERT).

**Domain Kasus**: Perceraian 

## Struktur Direktori

Berikut adalah struktur direktori yang digunakan dalam proyek ini.

```
.
├── notebooks/                # Berisi 5 Jupyter Notebooks untuk setiap tahap CBR 
├── data/
│   ├── raw/                  # Hasil teks bersih dari setiap putusan (.txt) 
│   ├── processed/            # File data terstruktur (CSV, JSON, .npy embeddings) 
│   ├── eval/                 # File untuk evaluasi (queries.json, metrics.csv) 
│   └── results/              # Hasil akhir prediksi (predictions.csv) 
├── Scrap_CSVs/               # Tempat penyimpanan CSV mentah dari scraper
├── PDFs_Putusan/             # Tempat penyimpanan file PDF mentah yang diunduh
├── logs/                     # File log untuk proses scraping dan cleaning 
├── requirements.txt          # Daftar pustaka (library) Python yang dibutuhkan
└── README.md                 # File panduan ini
```

## Instalasi

Untuk menjalankan proyek ini, pastikan Anda memiliki Python 3.8+ dan Git terinstal di sistem Anda.

1.  **Clone Repository**
    Buka terminal atau command prompt, lalu jalankan perintah berikut:
    ```bash
    git clone https://github.com/RNikmah/CBR_Project.git
    cd CBR_Project
    ```

2.  **Buat Virtual Environment (Sangat Direkomendasikan)**
    Untuk menjaga dependensi proyek tetap terisolasi, buat dan aktifkan *virtual environment*:
    ```bash
    # Untuk Windows
    python -m venv venv
    .\venv\Scripts\activate

    # Untuk macOS/Linux
    python3 -m venv venv
    source venv/bin/activate
    ```

3.  **Instal Dependensi**
    Instal semua pustaka Python yang dibutuhkan dengan menjalankan perintah berikut dari `requirements.txt`:
    ```bash
    pip install -r requirements.txt
    ```

## Cara Menjalankan Pipeline End-to-End

Proyek ini terdiri dari 5 *notebook* yang harus dijalankan secara berurutan, karena *output* dari satu *notebook* menjadi *input* untuk *notebook* berikutnya. Jalankan *notebook* dari direktori `/notebooks/`.

---

### **Tahap 1: `01_Scraping_and_Case_Base_Building.ipynb`**

* **Tujuan**: Mengumpulkan data putusan (scraping) dari situs Mahkamah Agung, mengekstrak teks dari PDF, membersihkannya, dan menyimpan metadata awal.
* **Tindakan Pengguna**:
    1.  Buka *notebook* `01_...`.
    2.  **PENTING**: Di dalam sel "Configuration Section", atur nilai variabel `BASE_DRIVE_PATH` dan `MA_SEARCH_RESULT_URL` sesuai dengan konfigurasi proyek Anda.
    3.  Jalankan semua sel secara berurutan.
* **Output Utama**:
    * File-file `.txt` di direktori `data/raw/`.
    * File CSV metadata awal di direktori `Scrap_CSVs/`.

---

### **Tahap 2: `02_Case_Representation.ipynb`**

* **Tujuan**: Memperkaya metadata awal dengan mengekstrak fitur-fitur kunci dari teks putusan yang sudah bersih, seperti ringkasan fakta, argumen hukum, dan pasal yang diterapkan.
* **Tindakan Pengguna**:
    1.  Buka *notebook* `02_...`.
    2.  Pastikan `BASE_DRIVE_PATH` dan `KEYWORD_FOR_FILENAMING` sudah sesuai dengan yang ada di Tahap 1.
    3.  Jalankan semua sel. Anda mungkin perlu menyesuaikan pola *regex* pada fungsi-fungsi ekstraksi untuk hasil yang lebih akurat.
* **Output Utama**:
    * File `data/processed/cases_represented.csv`, yang merupakan *case base* utama kita.

---

### **Tahap 3: `03_Case_Retrieval.ipynb`**

* **Tujuan**: Membuat representasi vektor untuk setiap kasus menggunakan dua metode berbeda (TF-IDF dan IndoBERT) dan membangun fungsi *retrieval* untuk keduanya.
* **Tindakan Pengguna**:
    1.  Buka *notebook* `03_...`.
    2.  Pastikan *runtime* Colab Anda menggunakan GPU untuk mempercepat proses pembuatan *embedding* BERT.
    3.  Jalankan semua sel. *Notebook* ini akan membangun model untuk kedua pendekatan.
    4.  Setelah selesai, *notebook* akan membuat *template* `data/eval/queries.json`. **Anda harus mengedit file ini secara manual** untuk mengisi `relevant_case_ids` dengan ID kasus yang Anda anggap relevan untuk setiap *query*.
* **Output Utama**:
    * Model dan matriks untuk TF-IDF (`.pkl`, `.npz`).
    * *Embeddings* untuk BERT (`.npy`).
    * Fungsi `retrieve_cases_tfidf()` dan `retrieve_cases_bert()`.

---

### **Tahap 4: `04_Solution_Reuse.ipynb`**

* **Tujuan**: Menggunakan kasus-kasus serupa yang ditemukan di Tahap 3 untuk memprediksi "solusi" (kategori amar putusan).
* **Tindakan Pengguna**:
    1.  Buka *notebook* `04_...`.
    2.  Jalankan semua sel. *Notebook* ini akan menjalankan prediksi solusi menggunakan hasil *retrieval* dari **kedua model (BERT dan TF-IDF)** untuk perbandingan.
* **Output Utama**:
    * File `data/results/predictions_comparison.csv` yang berisi prediksi solusi dari kedua model untuk setiap *query*.

---

### **Tahap 5: `05_Model_Evaluation.ipynb`**

* **Tujuan**: Mengukur dan membandingkan performa sistem *retrieval* dan prediksi solusi dari kedua model.
* **Tindakan Pengguna**:
    1.  Buka *notebook* `05_...`.
    2.  **PENTING**: Edit kembali *file* `data/eval/queries.json`. Tambahkan *key* baru `"correct_outcome"` pada setiap *query* dengan nilai kategori amar putusan yang benar menurut Anda (misalnya, `"MENGABULKAN GUGATAN"`).
    3.  Jalankan semua sel untuk melihat metrik performa (Precision, Recall, F1-Score) dan visualisasi perbandingannya.
* **Output Utama**:
    * File `data/eval/retrieval_metrics.csv`.
    * File `data/eval/prediction_metrics.csv`.
    * Visualisasi perbandingan performa dan panduan untuk analisis kegagalan.