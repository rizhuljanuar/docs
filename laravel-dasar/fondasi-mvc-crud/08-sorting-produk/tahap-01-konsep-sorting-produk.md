# Tahap 1 — Apa itu Sorting Produk & Kenapa User Butuh Fitur Mengurutkan Produk

> Materi: Sorting Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi Sehari-hari: Belanja di Minimarket

Bayangkan kamu lagi belanja di minimarket. Kamu mau beli **minuman**, dan di rak ada **puluhan pilihan** dengan harga yang beda-beda.

Sekarang kamu punya pertanyaan di kepala:

- "Aku mau lihat **yang paling murah** dulu."
- "Aku mau lihat **yang paling mahal** dulu."
- "Aku mau lihat **minuman baru** yang baru masuk."
- "Aku mau lihat minuman **urutan abjad dari A** biar gampang cari merk."

Coba bayangkan kalau minimarket itu **tidak peduli dengan keinginanmu**. Semua minuman ditata asal-asalan: yang mahal campur yang murah, yang baru campur yang lama, acak total.

Kamu pasti akan:
- Capai membolak-balik rak.
- Lama menemukan minuman yang sesuai kantong.
- Akhirnya **keluar dari toko** karena capek.

Nah, minimarket yang baik pasti punya rak yang **sudah diurutkan**: minuman termurah di satu sisi, minuman premium di sisi lain, atau diurutkan berdasarkan merk (A-Z).

**Itulah sorting.** Mengurutkan barang supaya user gampang menemukan yang dicari.

---

## 2. Masalahnya: Kalau Produk Tidak Bisa Diurutkan

Sekarang kita pindah ke dunia **toko online** (website Laravel kita).

Misal kamu punya **500 produk** dengan field:

- nama produk
- harga
- stok
- deskripsi
- kategori
- slug
- gambar
- tanggal dibuat (created_at)

Kalau produk **tidak bisa diurutkan**, ini masalah yang muncul:

### a. User Tidak Bisa Cari Barang Sesuai Budget
User mau lihat produk **termurah** dulu karena budgetnya terbatas. Tapi karena produk tampil acak, ia harus bolak-balik halaman, manual cari yang murah. Capek.

### b. User Tidak Tahu Produk Baru
Toko online sering update produk baru. User ingin tahu "apa sih produk terbaru?". Kalau tidak ada fitur "urutkan dari yang terbaru", user harus scroll semua halaman. Produk baru tenggelam.

### c. User Susah Cari Berdasarkan Nama
User ingin cari produk dengan merk "A" saja. Kalau produk tidak bisa diurutkan A-Z, user harus scan satu-satu.

### d. User Pergi ke Toko Lain
User malas = user keluar. Di dunia online, kalau website kamu **tidak nyaman**, user pindah ke kompetitor. Feature kecil seperti sorting bisa menyelamatkan penjualan.

### e. Produk Terasa "Berantakan"
Daftar produk tanpa urutan yang jelas terlihat tidak profesional. User merasa toko tidak rapi, padahal produknya bagus.

---

## 3. Jadi, Apa Itu Sorting Produk?

**Sorting produk** = fitur untuk **mengurutkan daftar produk** berdasarkan kriteria tertentu.

Kriteria yang umum dipakai di toko online:

| Kriteria | Arti | Field di Database |
|---|---|---|
| Termurah | Harga dari kecil ke besar | `harga` naik |
| Termahal | Harga dari besar ke kecil | `harga` turun |
| Terbaru | Produk yang baru ditambahkan | `created_at` turun |
| Terlama | Produk pertama kali ditambahkan | `created_at` naik |
| Nama A-Z | Urutan abjad dari A | `nama` naik |
| Nama Z-A | Urutan abjad dari Z | `nama` turun |

Dua istilah penting yang akan sering kamu dengar:

- **ASC (Ascending)** = urutan **naik**. Dari kecil ke besar, dari A ke Z, dari lama ke baru.
- **DESC (Descending)** = urutan **turun**. Dari besar ke kecil, dari Z ke A, dari baru ke lama.

Contoh:
- Harga **ASC** = 1.000 → 5.000 → 10.000 (termurah dulu).
- Harga **DESC** = 10.000 → 5.000 → 1.000 (termahal dulu).

---

## 4. Kenapa User Butuh Fitur Sorting?

Karena **setiap user datang dengan tujuan berbeda**:

- User A: budget terbatas → butuh **urutan termurah**.
- User B: ingin produk terbaru -> butuh **urutan terbaru**.
- User C: cari merk tertentu → butuh **urutan nama A-Z**.
- User D: ingin lihat produk premium dulu → butuh **urutan termahal**.

Satu daftar produk, tapi **bisa "dipintal" sesuai kebutuhan user**. Itulah kekuatan sorting: **data yang sama, urutan yang berbeda-beda, sesuai pilihan user.**

---

## 5. Bagaimana Sorting Bekerja di Website?

Di website, sorting biasanya dilakukan lewat **URL**. User mengklik tombol di halaman, dan URL berubah.

Contoh URL:

```
/produk?sort=terbaru
/produk?sort=harga-termurah
/produk?sort=harga-termahal
/produk?sort=nama-az
```

Perhatikan bagian setelah tanda tanya `?`:

```
?sort=terbaru
```

- `sort` = nama parameter (kata kunci yang kita kasih).
- `terbaru` = nilai yang user pilih.

Jadi user bilang ke server: **"Tolong tampilkan produk, tapi urutkan yang terbaru."**

Server terima permintaan itu, lalu **mengubah query database** sesuai pilihan user.

Nah, di sinilah muncul istilah penting: **Dynamic Query**. Kita akan bahas pelan-pelan di tahap berikutnya.

---

## Ringkasan Tahap 1

| Hal | Isi |
|---|---|
| Masalah | Produk acak = user susah cari, pindah ke toko lain |
| Solusi | Fitur sorting (urutkan produk sesuai pilihan user) |
| Analogi | Rak minimarket yang sudah diurutkan termurah/merk |
| Kriteria umum | Termurah, termahal, terbaru, terlama, nama A-Z, nama Z-A |
| Istilah penting | ASC (naik) dan DESC (turun) |
| Cara kerja | User klik → URL berubah → server urutkan data |

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: memahami parameter sort dan dynamic query di Laravel?**

Kalau iya, tahap 2 kita akan:
1. Lihat apa itu **parameter sort** di URL Laravel.
2. Ambil parameter itu di controller pakai `request('sort')`.
3. Kenalin konsep **dynamic query** = query yang bisa berubah-ubah sesuai input user.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
