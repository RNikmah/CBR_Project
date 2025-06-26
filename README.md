# Proyek Sistem Case-Based Reasoning (CBR) untuk Analisis Putusan Pengadilan

Proyek ini merupakan implementasi sistem *Case-Based Reasoning* (CBR) sederhana yang dirancang untuk mendukung analisis putusan pengadilan di Indonesia.  Sistem ini dibangun menggunakan Python dan memanfaatkan data putusan yang dipublikasikan di Direktori Putusan Mahkamah Agung Republik Indonesia. 

Siklus CBR yang diimplementasikan mencakup tahapan *case base building* (pengumpulan dan pembersihan data), *case representation* (ekstraksi fitur dan metadata), *case retrieval* (pencarian kasus serupa), *solution reuse* (prediksi solusi), dan *model evaluation* (evaluasi performa). 

**Domain Kasus**: Perceraian

## Struktur Direktori

Berikut adalah struktur direktori yang digunakan dalam proyek ini sesuai dengan panduan. 

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
└── README.md                 # File ini
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
    Instal semua pustaka Python yang dibutuhkan dengan menjalankan perintah berikut: 
    ```bash
    pip install -r requirements.txt
    ```

## Cara Menjalankan Pipeline End-to-End

Proyek ini terdiri dari 5 *notebook* yang harus dijalankan secara berurutan, karena *output* dari satu *notebook* menjadi *input* untuk *notebook* berikutnya.  Jalankan *notebook* dari direktori `/notebooks/`.

---

### **Tahap 1: `01_Scraping_and_Case_Base_Building.ipynb`**

* **Tujuan**: Mengumpulkan data putusan dari situs Mahkamah Agung, mengekstrak teks dari PDF, membersihkannya, dan menyimpan metadata awal.
* **Tindakan Pengguna**:
    1.  Buka *notebook* `01_...`.
    2.  **PENTING**: Di dalam sel "Configuration Section", atur nilai variabel `BASE_DRIVE_PATH` agar sesuai dengan path proyek Anda di Google Drive.
    3.  **PENTING**: Atur nilai `MA_SEARCH_RESULT_URL` dengan URL hasil pencarian dari situs Direktori Putusan untuk domain "Perceraian".
    4.  Jalankan semua sel secara berurutan.
* **Output Utama**:
    * File-file `.txt` di direktori `data/raw/`.
    * File CSV metadata awal di direktori `Scrap_CSVs/`.

---

### **Tahap 2: `02_Case_Representation.ipynb`**

* **Tujuan**: Memperkaya metadata awal dengan mengekstrak fitur-fitur kunci dari teks putusan yang sudah bersih.
* **Tindakan Pengguna**:
    1.  Buka *notebook* `02_...`.
    2.  Pastikan `BASE_DRIVE_PATH` dan `KEYWORD_FOR_FILENAMING` sudah sesuai dengan yang ada di Tahap 1.
    3.  Jalankan semua sel. Anda mungkin perlu menyesuaikan pola *regex* pada fungsi-fungsi ekstraksi (`extract_pihak_from_text`, `extract_pasal_rules_from_text`, dll.) untuk hasil yang lebih akurat.
* **Output Utama**:
    * File `data/processed/cases_represented.csv`, yang merupakan *case base* utama kita.

---

### **Tahap 3: `03_Case_Retrieval_BERT.ipynb`**

* **Tujuan**: Membuat representasi vektor (*embedding*) untuk setiap kasus menggunakan model Transformer (IndoBERT) dan membangun fungsi *retrieval*.
* **Tindakan Pengguna**:
    1.  Buka *notebook* `03_...`.
    2.  Pastikan *runtime* Colab Anda menggunakan GPU untuk mempercepat proses.
    3.  Jalankan semua sel. Proses pembuatan *embedding* mungkin memakan waktu beberapa menit.
    4.  Setelah selesai, *notebook* akan membuat *template* `data/eval/queries.json`. **Anda harus mengedit file ini secara manual** untuk mengisi `relevant_case_ids` dengan ID kasus yang Anda anggap relevan untuk setiap *query*.
* **Output Utama**:
    * File `data/processed/case_embeddings_bert.npy` (vektor *embedding*).
    * File `data/processed/case_ids_bert.json` (urutan ID kasus).
    * File `data/eval/queries.json` (template *query* untuk pengujian).
    * Fungsi `retrieve_cases_bert()` yang teruji.

---

### **Tahap 4: `04_Solution_Reuse.ipynb`**

* **Tujuan**: Menggunakan kasus-kasus serupa yang ditemukan di Tahap 3 untuk memprediksi "solusi" (kategori amar putusan).
* **Tindakan Pengguna**:
    1.  Buka *notebook* `04_...`.
    2.  Jalankan semua sel. *Notebook* ini akan menggunakan `queries.json` dari Tahap 3 untuk menjalankan demo prediksi.
* **Output Utama**:
    * File `data/results/predictions.csv` yang berisi prediksi solusi untuk setiap *query*.

---

### **Tahap 5: `05_Model_Evaluation.ipynb`**

* **Tujuan**: Mengukur performa sistem *retrieval* dan prediksi solusi.
* **Tindakan Pengguna**:
    1.  Buka *notebook* `05_...`.
    2.  **PENTING**: Edit kembali *file* `data/eval/queries.json`. Tambahkan *key* baru `"correct_outcome"` pada setiap *query* dengan nilai kategori amar putusan yang benar menurut Anda (misalnya, `"MENGABULKAN GUGATAN"`).
    3.  Jalankan semua sel untuk melihat metrik performa (Precision, Recall, F1-Score) dan visualisasinya.
* **Output Utama**:
    * File `data/eval/retrieval_metrics.csv`.
    * File `data/eval/prediction_metrics.csv`.
    * Visualisasi performa dan panduan untuk analisis kegagalan.