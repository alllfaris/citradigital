# Mini-Project: Deteksi Tanda Tangan (Signature Detection)

**Mata Kuliah:** Pengolahan Citra Digital  
**Nama:** La Ode Muhamad Faris  
**NIM:** F1G124035  
**Kelas:** Kelas C

---

## 📌 Deskripsi Proyek

Proyek ini merupakan implementasi pengolahan citra digital berbasis **Document Image Analysis (DIA)** menggunakan Python dan OpenCV. Sistem ini dirancang untuk melakukan verifikasi keberadaan tanda tangan (tanda tangan **Rektor** pada ijazah) secara otomatis dengan mengisolasi Region of Interest (ROI), memproses citra melalui binerisasi thresholding, menyempurnakan bentuk dengan operasi morfologi, serta menghitung rasio piksel tinta (*foreground*) sebagai aturan keputusan.

---

## 📁 Struktur Repositori

```
.
├── data/                             # Direktori sampel citra dokumen pengujian
├── Tugas_Citra_Digital.ipynb         # Jupyter Notebook berisi seluruh kode & eksperimen
└── readme.md                         # Dokumentasi lengkap proyek
```

Isi `data/`: 9 citra asli (`01`–`09`, dokumen sama dengan degradasi berbeda, semuanya **bertanda tangan**) dan 3 citra uji **tanpa tanda tangan** (`10_Blank_Document`, `11_Blank_Faded`, `12_Blank_HighNoise`) yang dibuat otomatis oleh notebook dengan menimpa area tanda tangan dengan kertas kosong.

---

## ⚙️ Alur Pemrosesan (Pipeline)

1. **Region of Interest (ROI) Cropping:** Memotong area tanda tangan Rektor berdasarkan proporsi persentase resolusi citra (x: 70,3%–81,8%, y: 61,3%–85,6%) agar adaptif terhadap ukuran citra.
2. **Grayscale Conversion:** Mengubah citra RGB menjadi skala keabuan, lalu Gaussian blur 3×3 untuk meredam noise.
3. **Thresholding (Global vs Otsu vs Adaptive):** Mengubah citra grayscale menjadi biner untuk memisahkan goresan tinta (*foreground*) dari kertas (*background*).
   - **Global:** T tetap = 127.
   - **Otsu:** T otomatis dari histogram, **dibatasi** maksimal `median latar − 40`. Otsu murni selalu membelah histogram menjadi dua kelas sehingga pada ROI kertas kosong ia menghasilkan foreground palsu (32–42%); pembatas ini mencegahnya.
   - **Adaptive Gaussian:** blok 51, C = 15 (pembanding).
4. **Morphological Operations:**
   - **Opening** (erosi → dilasi, kernel elips 3×3): menghilangkan bintik noise.
   - **Closing** (dilasi → erosi, kernel elips 7×7): menyambungkan goresan tinta yang terputus.
5. **Karakteristik Area & Decision Rule:** dihitung jumlah piksel foreground, rasio terhadap total piksel ROI, jumlah komponen terhubung, dan luas komponen terbesar.
   - **Jika Pixel Ratio ≥ 1,0% DAN komponen terbesar ≥ 1500 px** → `SIGNATURE PRESENT`
   - **Selain itu** → `SIGNATURE ABSENT`

---

## 📊 Hasil Pengujian & Evaluasi

Pipeline lengkap (Otsu dibatasi + opening + closing). Akurasi: **12/12 (100%)**.

| Nama Sampel                    | Otsu Threshold | Foreground (px) | Pixel Ratio (%) | Status Deteksi        |
| ------------------------------ | -------------- | --------------- | --------------- | --------------------- |
| `01_HighQuality_Enhanced`      | 153            | 9.777           | 4.03%           | **SIGNATURE PRESENT** |
| `02_LowContrast`               | 195            | 9.804           | 4.04%           | **SIGNATURE PRESENT** |
| `03_Blurred`                   | 192            | 13.697          | 5.64%           | **SIGNATURE PRESENT** |
| `04_HighNoise`                 | 160            | 9.688           | 3.99%           | **SIGNATURE PRESENT** |
| `05_LowResolution_Upsampled`   | 173            | 10.519          | 4.33%           | **SIGNATURE PRESENT** |
| `06_Faded_Underexposed`        | 119            | 9.642           | 3.97%           | **SIGNATURE PRESENT** |
| `07_ColorShift_WarmTint`       | 160            | 9.795           | 4.03%           | **SIGNATURE PRESENT** |
| `08_JPEGCompression_Artifacts` | 165            | 9.793           | 4.03%           | **SIGNATURE PRESENT** |
| `09_CombinedDegradation`       | 139            | 10.358          | 4.27%           | **SIGNATURE PRESENT** |
| `10_Blank_Document`            | 208            | 0               | 0.00%           | **SIGNATURE ABSENT**  |
| `11_Blank_Faded`               | 119            | 0               | 0.00%           | **SIGNATURE ABSENT**  |
| `12_Blank_HighNoise`           | 200            | 0               | 0.00%           | **SIGNATURE ABSENT**  |

### Perbandingan Metode Thresholding (rasio foreground sebelum morfologi, %)

| Sampel                  | Global T=127 | Otsu murni | Otsu dibatasi | Adaptive |
| ----------------------- | ------------ | ---------- | ------------- | -------- |
| `01_HighQuality_Enhanced` | 3.60       | 3.95       | 3.95          | 4.17     |
| `02_LowContrast`        | **0.62**     | 3.96       | 3.96          | 3.98     |
| `03_Blurred`            | 1.92         | 5.60       | 5.60          | 5.04     |
| `06_Faded_Underexposed` | 4.18         | 3.96       | 3.90          | 3.85     |
| `10_Blank_Document`     | 0.00         | **41.71**  | 0.00          | 0.02     |
| `11_Blank_Faded`        | 0.00         | **31.70**  | 0.00          | 0.00     |
| `12_Blank_HighNoise`    | 0.00         | **37.06**  | 0.00          | 0.20     |

Ringkasan: Global threshold gagal pada citra kontras rendah/blur (tinta tidak cukup gelap dari T=127); Otsu murni gagal pada kertas kosong; **Otsu dibatasi** stabil pada seluruh sampel. Tabel lengkap, gambar tiap tahap, dan grafik eksperimen ada di notebook.

---

## 🧠 Analisis & Jawaban Pertanyaan

### 1. Mengapa thresholding diperlukan sebelum melakukan analisis keberadaan tanda tangan?

Komputer tidak mengenali bentuk visual tanda tangan seperti manusia, melainkan hanya melihat matriks intensitas piksel (0–255) yang bercampur antara tinta, tekstur kertas, bayangan, dan noise. *Thresholding* mengubah citra grayscale menjadi citra biner sehingga tinta (*foreground*) terpisah tegas dari kertas (*background*). Setelah itu, besaran objektif seperti jumlah piksel foreground, rasio terhadap ROI, dan luas komponen terhubung terbesar dapat dihitung dan dipakai untuk mengambil keputusan.

### 2. Apa masalah yang terjadi jika threshold terlalu tinggi atau terlalu rendah?

Karena tinta lebih gelap dari kertas (foreground = piksel < T):

- **Threshold Terlalu Rendah:** hanya piksel sangat gelap yang lolos. Tinta tipis atau pudar hilang/terputus sehingga ROI tampak kosong → **False Negative** (tanda tangan ada tetapi terdeteksi tidak ada). Contoh: pada `02_LowContrast`, T = 100 menghasilkan 0% foreground dan T = 125 hanya 0,47%, padahal tanda tangan sebenarnya ≈ 4%.
- **Threshold Terlalu Tinggi:** piksel kertas yang agak gelap (tekstur latar, bayangan, noise pindaian) ikut menjadi foreground → **False Positive** (kertas kosong terdeteksi memiliki tanda tangan). Contoh: pada kertas kosong, Otsu murni menghasilkan 32–42% foreground palsu, dan T = 250 membuat 88% ROI menjadi foreground.
- **Solusi:** Otsu (T otomatis) dengan batas atas relatif terhadap kecerahan latar, ditambah **opening/closing** dan aturan ganda (rasio ≥ 1% dan komponen terbesar ≥ 1500 px).

### Keterbatasan

Kesembilan citra asli adalah dokumen yang sama dengan degradasi berbeda (posisi ROI identik), dan sampel *ABSENT* dibuat sintetis dengan menimpa area tanda tangan dengan kertas kosong. Akurasi 100% pada kumpulan ini belum menjamin kinerja pada dokumen lain dengan tata letak berbeda.

---

## 🚀 Panduan Menjalankan Kode (*How to Run*)

### 1. Prasyarat System

Pastikan Python 3.x dan library berikut telah terinstall:

```
pip install opencv-python numpy pandas matplotlib jupyter
```

### 2. Menjalankan Jupyter Notebook

1. Clone repositori ini ke komputer lokal Anda:

```
git clone https://github.com/<username-github-anda>/citradigital.git
cd citradigital
```

2. Jalankan Jupyter Notebook:

```
jupyter notebook Tugas_Citra_Digital.ipynb
```

3. Eksekusi seluruh cell secara berurutan (`Shift + Enter`) atau pilih **Run → Run All Cells**. Notebook akan membuat citra uji `10`–`12` di folder `data/` (jika belum ada) dan menyimpan gambar/CSV hasil di folder `hasil/`.
