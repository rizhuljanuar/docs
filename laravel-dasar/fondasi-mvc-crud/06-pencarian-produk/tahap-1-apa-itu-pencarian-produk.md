# Tahap 1 — Apa Itu Pencarian Produk dan Kenapa Fitur Ini Penting?

> Materi Laravel Dasar · Topik 6: Pencarian Produk
> Mentor: penjelasan bertahap untuk pemula total

---

## 1. Analogi dari Kehidupan Sehari-hari

Bayangkan kamu masuk ke sebuah **toko buku besar** yang punya **10.000 buku**.

Buku-buku itu ditumpuk di meja panjang tanpa label, tanpa rak bernomor, tanpa petugas.
Kamu ingin mencari **satu buku**:

> "Buku Laravel untuk Pemula"

Apa yang akan kamu lakukan?

Kamu akan **mengigit jari**. Kamu harus membolak-balik satu per satu dari tumpukan paling atas sampai ketemu.
Bisa saja **seharian** tidak ketemu, atau ketemunya buku yang **mirip tapi salah**.

Sekarang bayangkan toko yang sama, tapi ada:

- **Rak bernomor** (kategori: Pemrograman, Novel, Masak)
- **Label abjad** di setiap rak (A-Z)
- **Petugas** yang bisa kamu tanya: *"Ada buku Laravel?"*
- **Komputer pencarian** di pojok toko

Cukup ketik **"Laravel"**, dan layar menunjukkan:

```
Rak 12 · Pemrograman · Baris ke-3 · Judul: Laravel untuk Pemula
```

**Nempel jari 3 detik.**

Inilah inti dari **fitur pencarian** di sebuah aplikasi.

---

## 2. Apa Itu "Pencarian Produk" dalam CRUD?

CRUD Produk = kita sudah bisa:

- **C**reate  → tambah produk baru
- **R**ead    → lihat daftar produk
- **U**pdate  → ubah produk
- **D**elete  → hapus produk

Sampai di sini, halaman daftar produk kita **menampilkan SEMUA produk** dari database, baris demi baris.

Masalahnya, di dunia nyata, **daftar produk itu bisa sangat panjang**.

**Pencarian produk** = fitur tambahan di bagian **Read** yang memungkinkan user **menyaring (memfilter)** daftar produk itu, sehingga yang tampil hanya produk yang **cocok dengan kata kunci** yang user ketik.

Contoh:

```
User ketik di kotak pencarian:  "Laptop"
↓
Yang tampil:
  - Laptop Asus ROG
  - Laptop Acer Predator
  - Laptop Gaming Murah
↓
Yang tidak tampil:
  - Kaos Hitam
  - Sepatu Running
  - Buku Laravel
```

---

## 3. Kenapa Fitur Pencarian Itu Penting?

Pikirkan toko online yang kamu pakai sehari-hari: **Tokopedia, Shopee, Bukalapak, Lazada**.

Bayangkan kalau mereka **tidak punya kotak pencarian**.

Setiap kamu mau beli sesuatu, kamu harus **scroll** dari produk pertama sampai ketemu.
Di Tokopedia saja ada **ratusan juta produk**. Berapa lama kamu harus scroll?

**Tidak akan selesai dalam seumur hidup.**

Maka, pencarian adalah fitur yang **wajib** di hampir semua aplikasi yang menampilkan daftar data:

- Toko online → cari produk
- YouTube → cari video
- WhatsApp → cari kontak
- Google Maps → cari tempat
- File Explorer di laptop → cari file

Tanpa pencarian, aplikasi **tidak bisa dipakai** saat data sudah banyak.

---

## 4. Apa yang Terjadi Kalau Toko Online Tidak Ada Pencarian?

Mari kita lihat masalahnya satu per satu. Ini akan menjelaskan **kenapa kita harus belajar fitur ini**.

### Masalah 1: Waktu yang Lama

Kalau ada **1.000 produk** dan user cari **"Sepatu Running"**, user harus scroll dari atas sampai bawah.
Waktu yang dibutuhkan bisa **menit bahkan lebih**. User **bosan** dan **pergi ke toko lain**.

### Masalah 2: Produk Tidak Ketemu

User mau beli **"Headset Bluetooth"**, tapi produknya ada di **baris ke-732**.
User **nggak sabar** scroll sampai 732. Dia pikir **tokonya tidak jual**.
Padahal barangnya ada.

**Artinya: toko kehilangan uang** hanya karena user tidak bisa menemukan produknya.

### Masalah 3: Server Berat

Kalau aplikasi **selalu menampilkan semua produk sekaligus**, server harus **mengirim 1.000 baris data** ke browser user setiap kali halaman dibuka.
Browser berat. Internet boros. HP user nge-lag.

Pencarian membuat server **hanya mengirim produk yang relevan**, misal 10 baris saja. Hemat. Cepat.

### Masalah 4: User Marah

User ketik di kolom komentar: *"Cari barang susah banget, kenapa nggak ada search?"*
Review buruk. Toko tutup.

---

## 5. Contoh Daftar Produk Kita

Sebagai bahan belajar, kita pakai tabel **produk** dengan field berikut:

| Field       | Contoh Isi                |
|-------------|---------------------------|
| nama        | Laptop Asus ROG           |
| harga       | 15000000                  |
| stok        | 5                         |
| deskripsi   | Laptop gaming performa tinggi |
| kategori    | Elektronik                |
| slug        | laptop-asus-rog           |
| gambar      | laptop-asus-rog.jpg       |

Misal di database kita sudah ada **8 produk** begini:

| # | nama                  | kategori    | harga     |
|---|-----------------------|-------------|-----------|
| 1 | Laptop Asus ROG       | Elektronik  | 15.000.000|
| 2 | Laptop Acer Predator  | Elektronik  | 18.000.000|
| 3 | Kaos Hitam Polos      | Fashion     | 50.000    |
| 4 | Sepatu Running Nike   | Olahraga    | 750.000   |
| 5 | Sepatu Futsal Adidas  | Olahraga    | 600.000   |
| 6 | Buku Laravel Pemula   | Buku        | 120.000   |
| 7 | Buku PHP Modern       | Buku        | 150.000   |
| 8 | Headset Bluetooth JBL | Elektronik  | 350.000   |

Coba perhatikan:

- User ketik **"Laptop"**  → harus muncul produk #1 dan #2
- User ketik **"Sepatu"**  → harus muncul produk #4 dan #5
- User ketik **"Buku"**    → harus muncul produk #6 dan #7

**Inilah yang akan kita pelajari di tahap-tahap berikutnya.**

---

## 6. Kesimpulan Tahap 1

- **Pencarian produk** = fitur untuk **menyaring** daftar produk supaya hanya yang cocok yang tampil.
- Fitur ini **wajib** kalau daftar produk sudah banyak (puluhan, ratusan, ribuan).
- Tanpa pencarian, user **susah menemukan barang**, **bosan**, **pergi ke toko lain**.
- Contoh nyata: Tokopedia, Shopee, YouTube, WhatsApp — semuanya punya pencarian.
- Kita akan pakai tabel **produk** dengan field: **nama, harga, stok, deskripsi, kategori, slug, gambar**.

---

## Pertanyaan Berikutnya

**Apakah kamu ingin lanjut ke langkah berikutnya: memahami query `where` dan `like` untuk mencari produk?**

Pada tahap berikutnya kita akan belajar:

- Apa itu **query database** (dengan analogi sederhana)
- Apa itu **filtering** (penyaringan data)
- Arti dari `where('name', 'like', '%laptop%')` dalam bahasa manusia

Kita **tidak akan ngoding dulu** di tahap berikutnya. Kita pahami **konsepnya** dulu, baru setelah itu masuk ke kode Laravel pelan-pelan.

— **Mentor Laravel**
