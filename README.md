# CookieDong Sales & Profitability Analysis

Analisis penjualan, profitabilitas, dan perilaku pelanggan bisnis kue rumahan **CookieDong**, mulai dari data mentah hingga menjadi dashboard interaktif berbasis Power BI yang bisa dipakai untuk pengambilan keputusan bisnis.

---

## 1. Problem Statement

CookieDong adalah bisnis kue rumahan yang sudah berjalan sekitar 2 bulan, menjual produk melalui WhatsApp dan Instagram. Sejauh ini pemilik bisnis mencatat transaksi secara manual, tapi belum punya cara sistematis untuk menjawab pertanyaan penting seperti:

- Bagaimana tren penjualan dari waktu ke waktu?
- Produk apa yang paling laris — dan apakah produk paling laris itu juga yang paling menguntungkan?
- Channel mana yang lebih dominan dan layak difokuskan?
- Kategori produk mana yang secara keseluruhan paling sehat dari sisi margin?

Project ini menjawab pertanyaan-pertanyaan tersebut lewat proses analisis data dari ujung ke ujung: membersihkan data mentah, menggali insight lewat eksplorasi data, mengelompokkan pelanggan berdasarkan perilaku belanja, sampai menyusunnya jadi dashboard 2 halaman yang mudah dibaca dan actionable.

---

## 2. Data Overview

- **Sumber data:** Data transaksi riil bisnis CookieDong, digunakan dengan izin pemilik, nama pelanggan dianonimkan sebagian untuk menjaga privasi.
- **Periode:** Mei 2026 – Juli 2026 (± 2 bulan)
- **Jumlah data mentah:** 183 baris (tiap baris mewakili satu produk dalam satu pesanan)
- **Jumlah data setelah dibersihkan:** 181 baris (2 baris duplikat dihapus)

**Kolom pada data mentah:**

| Kolom | Deskripsi |
|---|---|
| Customer | Nama pelanggan |
| Channel | Saluran pemesanan (WhatsApp, Instagram) |
| Tanggal Order | Tanggal pesanan dibuat |
| Produk_ID | Kode unik tiap produk |
| Menu | Nama produk yang dipesan |
| Qty | Jumlah item yang dipesan |
| Harga | Harga jual satuan produk |
| Subtotal | Qty × Harga (nilai transaksi untuk baris itu) |
| HPP | Harga Pokok Produksi per satuan produk (biaya modal untuk membuat 1 pcs) |

Dengan tersedianya kolom **HPP**, analisis pada project ini bisa menjangkau lebih dari sekadar omzet — mencakup **laba bersih dan margin keuntungan** di setiap level: per produk, per kategori, per channel, hingga per pelanggan.

---

## 3. Tools & Tech Stack

- **Python** (pandas, numpy) — pembersihan data, perhitungan laba/margin, segmentasi pelanggan
- **Google Colab** — tempat menjalankan seluruh proses
- **Matplotlib & Seaborn** — visualisasi eksplorasi data
- **Power BI** — dashboard interaktif (2 halaman)
- **Google Drive** — penyimpanan checkpoint data antar tahap pengerjaan

---

## 4. Struktur Repository

```
cookiedong-sales-analysis/
├── README.md
├── data/
│   ├── Cookiedong_Order_2026-05-01_sampai_2026-07-31.csv   
│   └── processed/
│       ├── order_items_clean.csv
│       ├── orders_clean.csv
│       └── rfm_customer.csv
├── notebooks/
│   ├── 01_data_cleaning.ipynb
│   └── 02_eda_rfm_segmentation.ipynb
├── dashboard/
│   ├── Dashboard Cookiedong.pbix
│   └── Dashboard Preview.pdf/
│       
└── CookieDong Order Management System/
    ├── Login Page.jpeg
    └── Dashboard Page.jpeg
    └── Action Page.jpeg

```

---

## 5. Proses Kerja & Penjelasan Tiap Tahap

### 5.1 Data Cleaning (`01_data_cleaning.ipynb`)

**Tujuan tahap ini:** memastikan data mentah bebas dari kesalahan, konsisten formatnya, dan siap diolah lebih lanjut.

**Langkah-langkah yang dilakukan:**

1. **Membaca file dengan pemisah yang benar.** File data menggunakan tanda titik koma (`;`) sebagai pemisah kolom, bukan koma seperti default.

2. **Mengubah format Rupiah menjadi angka.** Kolom `HPP` awalnya tertulis dalam format seperti `Rp2.633,45` (teks). Dikonversi menjadi angka murni dengan menghapus simbol Rp dan pemisah ribuan, lalu mengganti koma desimal menjadi titik.

3. **Memastikan Produk_ID konsisten dengan HPP dan nama produk.** Hasilnya: seluruhnya konsisten, tidak ada anomali.

4. **Menyeragamkan penulisan nama produk.** Variasi penulisan untuk produk yang sama disatukan lewat mapping manual, mengurangi jumlah nama produk unik dari 30 menjadi 27.

5. **Memvalidasi kebenaran nilai transaksi.** Dicek apakah `Qty × Harga` selalu sama dengan `Subtotal`. Hasilnya 100% cocok.

6. **Menghapus data yang tercatat dobel.** Ditemukan 2 baris duplikat persis, salah satunya dihapus. Data akhir menjadi 181 baris dari 183 baris awal.

7. **Mengubah format tanggal** dari teks menjadi tipe `datetime`, agar bisa dikelompokkan berdasarkan minggu, bulan, dan hari.

8. **Menambahkan kolom kategori produk** — Cookies, Scoopable, Fudgy Brownies, Banana Bread — berdasarkan kata kunci di nama produk.

9. **Menghitung Laba dan Margin di setiap baris:**
   - **Total HPP** = HPP per satuan × Qty
   - **Laba** = Subtotal − Total HPP
   - **Margin (%)** = Laba ÷ Subtotal × 100

   Semua angka hasil perhitungan dibulatkan untuk menghindari residu floating-point yang bisa menyebabkan salah baca angka saat diimpor ke tools visualisasi.

10. **Mengubah data dari level "per item" menjadi level "per transaksi"** — dikelompokkan berdasarkan kombinasi pelanggan + tanggal untuk membentuk tabel `orders`, lengkap dengan `Order_ID`, total quantity, omzet, HPP, laba, dan margin per transaksi (181 transaksi).

**Output tahap ini:**
- `order_items_clean.csv` — data level item, lengkap dengan kategori, laba, dan margin
- `orders_clean.csv` — data level transaksi, lengkap dengan total omzet, laba, dan margin per order

**Asumsi & batasan:**
- Satu kombinasi pelanggan + tanggal dianggap sebagai satu transaksi.

### 5.2 EDA & Customer Segmentation (`02_eda_rfm_segmentation.ipynb`)

**Bagian EDA:**

1. **Tren omzet dan laba mingguan** — ditemukan lonjakan besar di minggu 22–25 Juni, didorong beberapa transaksi besar dari pelanggan yang sama.
2. **Produk terlaris dari 3 sudut pandang** — quantity, omzet, dan laba — sengaja dipisah karena urutannya bisa sangat berbeda.
3. **Margin per produk** — ditemukan bahwa produk dengan omzet tertinggi (Scoopable Kunafa Pistachio) justru punya margin terendah, sementara Scoopable Double Choco punya margin tertinggi meski volumenya jauh lebih kecil.
4. **Performa per kategori** — Cookies jauh lebih sehat marginnya dibanding Scoopable, meski omzet keduanya berkontribusi besar terhadap total penjualan.
5. **Performa per channel** — WhatsApp mendominasi volume dan omzet, Instagram berkontribusi lebih kecil namun tetap signifikan.

**Bagian RFM Segmentation:**

Pelanggan dikelompokkan menggunakan pendekatan **Recency, Frequency, Monetary**, dengan skor 1–4 per dimensi menggunakan threshold manual (bukan quantile otomatis, karena jumlah pelanggan yang kecil membuat pembagian otomatis kurang representatif). Hasilnya, 29 pelanggan terbagi ke 4 segmen: **Champion**, **Loyal Customer**, **New/Recent Customer**, **At Risk/Lapsed**.

**Catatan metodologis:** Pendekatan clustering statistik (K-Means) sengaja tidak dipakai karena jumlah pelanggan yang kecil berisiko menghasilkan kelompok yang tidak stabil. Pendekatan rule-based dipilih karena lebih sesuai dan mudah dipertanggungjawabkan untuk skala data ini.

**Output tahap ini:** `rfm_customer.csv`, serta seluruh visualisasi EDA yang menjadi dasar desain dashboard.

---

## 6. Key Insights

1. **Produk paling laris ≠ produk paling menguntungkan.** Scoopable Kunafa Pistachio adalah penyumbang omzet tertinggi (Rp 1.740.000), tetapi marginnya paling rendah dari seluruh menu (21,65%). Sebaliknya, Scoopable Double Choco memiliki margin tertinggi (77,99%) meski omzetnya jauh lebih kecil (Rp 495.000) — di ranking Top 10 by Laba, Double Choco justru naik ke posisi pertama, melampaui Kunafa Pistachio.

2. **Kategori Cookies adalah kategori paling sehat secara keseluruhan**, dengan margin 57,73% — jauh di atas Fudgy Brownies (41,27%), Banana Bread (38,48%), dan terutama Scoopable (33,85%) yang menjadi kategori dengan margin terendah meski menyumbang beberapa produk terlaris.

3. **WhatsApp adalah channel dominan**, menyumbang 76,78% dari total omzet (Rp 5.771.000), dibanding Instagram yang menyumbang 23,22% (Rp 1.745.000).

4. **Scatter chart Omzet vs Margin menunjukkan pengelompokan yang jelas** — sebagian besar produk Cookies berada di kuadran margin sehat (50–60%) dengan omzet menengah, sementara mayoritas produk Scoopable (kecuali Double Choco) berkumpul di kuadran margin rendah (20–35%), dan Kunafa Pistachio berdiri sendiri sebagai outlier ekstrem di kuadran omzet tinggi–margin rendah.

5. **Margin keseluruhan bisnis sehat di angka 45,21%**, dengan total 642 produk terjual dari 181 transaksi (AOV sekitar Rp 41.525), dan tidak ada satupun produk yang terjual di bawah harga modal.

---

## 7. Dashboard Preview

Dashboard dibangun di Power BI dengan **2 halaman**, dilengkapi filter periode waktu (Bulan) di kedua halaman untuk eksplorasi lebih lanjut:

**Halaman 1 — Overview**
- KPI utama: Total Omzet, Total Laba, Profit Margin, Total Order, AOV, Total Produk Terjual
- Tren Omzet & Laba Mingguan (bar + line chart)
- Kontribusi Per Channel (donut chart)
- Kontribusi Per Kategori Menu (bar chart)
- Top 5 Profit Margin by Menu & Top 5 Menu by Quantity (tabel berdampingan)

**Halaman 2 — Sales & Product Performance**
- KPI ringkas: Total Menu Aktif, Menu Margin Tertinggi, Menu Margin Terendah
- Top 10 Menu by Omzet & Top 10 Menu by Laba (bar chart berdampingan dengan color scale margin)
- Scatter Chart Omzet vs Margin — memetakan seluruh 27 menu dalam satu visual untuk melihat pola volume vs profitabilitas
- Profit Margin by Kategori Menu (tabel ringkas dengan conditional formatting)

Halaman lanjutan (Channel Performance detail, Customer Insights berdasarkan segmentasi RFM) sengaja tidak dimasukkan ke dashboard versi ini agar cerita dashboard tetap fokus pada insight yang paling actionable — omzet/profitabilitas keseluruhan dan performa produk. Hasil segmentasi RFM tetap didokumentasikan lengkap di notebook `02_eda_rfm_segmentation.ipynb` sebagai bagian dari analisis, dan berpotensi dikembangkan menjadi halaman dashboard terpisah di iterasi berikutnya.

---

## 8. Rekomendasi Solusi Bisnis

Berdasarkan insight yang ditemukan, berikut rekomendasi yang bisa dipertimbangkan pemilik bisnis CookieDong:

1. **Evaluasi harga atau efisiensi biaya produksi Scoopable Kunafa Pistachio.** Sebagai produk dengan omzet tertinggi tapi margin terendah (21,65%), peningkatan kecil pada harga jual atau efisiensi bahan baku akan berdampak besar terhadap laba total karena volumenya yang tinggi.

2. **Dorong promosi produk bermargin tinggi namun kurang populer**, seperti Scoopable Double Choco (margin 77,99%, laba tertinggi di seluruh menu meski omzetnya bukan yang terbesar). Cross-selling produk ini kepada pembeli Kunafa Pistachio berpotensi menaikkan laba tanpa menambah kompleksitas produksi baru.

3. **Jadikan kategori Cookies sebagai andalan strategi promosi**, karena secara konsisten menunjukkan kombinasi volume dan margin paling sehat dibanding tiga kategori lainnya.

4. **Evaluasi ulang kategori Scoopable secara menyeluruh** (margin 33,85%, terendah dari semua kategori) — kecuali Double Choco, mayoritas produk Scoopable berada di kuadran margin rendah pada scatter chart. Perlu ditelusuri apakah ini karena harga jual belum disesuaikan dengan kenaikan bahan baku, atau memang karakteristik biaya produksi kategori ini secara umum lebih tinggi.

5. **Pertahankan dan optimalkan channel WhatsApp** sebagai kontributor omzet utama (76,78%), sambil terus mengembangkan Instagram sebagai channel pendukung.

---

## 9. Limitations

- **Periode data terbatas (~2 bulan)**, sehingga analisis musiman tidak bisa dilakukan secara reliable.
- **Jumlah pelanggan kecil (29 orang)**, sehingga segmentasi pelanggan menggunakan pendekatan rule-based, bukan clustering statistik.
- **Asumsi satu transaksi = kombinasi pelanggan + tanggal yang sama**, berpotensi menggabungkan dua transaksi berbeda di hari yang sama menjadi satu order.
- **HPP yang digunakan adalah HPP per satuan produk**, belum memperhitungkan biaya operasional lain (kemasan, ongkos kirim, tenaga kerja), sehingga margin yang dihasilkan adalah margin kotor produk, bukan margin bersih usaha secara keseluruhan.
- **Dashboard sengaja dibatasi pada 2 halaman** (Overview dan Sales & Product Performance) untuk menjaga fokus cerita data; analisis Customer Segmentation (RFM) sudah lengkap dilakukan di tahap notebook namun belum divisualisasikan dalam bentuk dashboard interaktif.

---

## 10. Related Project

Data pada analisis ini bersumber dari sistem pencatatan order CookieDong yang juga dikembangkan secara mandiri. Lihat **CookieDong Order Management System** — **https://cookiedong.my.id/** aplikasi web berbasis PHP/MySQL untuk pencatatan transaksi bisnis ini secara real-time.

---

## 11. Author

**Achmad Dylan Alfaris**
Portfolio: [dylan--portfolio.vercel.app](https://dylan--portfolio.vercel.app)
