# Tahap 2: Apa itu Relasi Database?

## Analogi Sederhana: Buku Telepon di HP

Bayangkan kamu punya **buku telepon** di HP. Setiap kontak punya:

- Nama
- Nomor telepon

Tapi suatu hari kamu ingin menyimpan juga **foto profil** untuk tiap kontak.

Ada dua cara:

**Cara 1 (Salah / Berantakan):**

| nama    | nomor      | foto                                  |
|---------|------------|---------------------------------------|
| Budi    | 0812345678 | (file gambar besar dijejalkan ke sini)|
| Sinta   | 0899887766 | (file gambar besar dijejalkan ke sini)|

Foto ukuran besar dijejalkan ke satu tabel -> tabel jadi berat dan lambat.

**Cara 2 (Benar / Rapi):**

Buat **dua tabel**, lalu **hubungkan** mereka.

Tabel `kontak`:

| id | nama  | nomor      |
|----|-------|------------|
| 1  | Budi  | 0812345678 |
| 2  | Sinta | 0899887766 |

Tabel `foto`:

| id | kontak_id | file              |
|----|-----------|-------------------|
| 1  | 1         | budi.jpg          |
| 2  | 2         | sinta.jpg         |

Perhatikan kolom **`kontak_id`** di tabel foto. Kolom ini bilang:

> "Foto ini milik kontak nomor 1 (Budi)."
> "Foto itu milik kontak nomor 2 (Sinta)."

Itulah **relasi**: cara menghubungkan dua tabel supaya tahu "yang ini milik yang mana".

## Jadi, Apa itu Relasi Database?

**Relasi database** = **hubungan** antara dua tabel atau lebih, supaya data di satu tabel bisa **merujuk** ke data di tabel lain.

Cara menghubungkannya pakai **kolom penghubung** berupa angka id, contohnya `kontak_id` atau `category_id`. Kolom ini disebut **foreign key** (kunci asing).

Kenapa disebut "asing"? Karena id itu **bukan milik tabel ini**, tapi merujuk ke id di **tabel lain**.

## Kenapa Harus Pakai Relasi?

Relasi menghindari **pengulangan data** dan menjaga database tetap rapi.

**Tanpa relasi (borong tempat):**

| id | nama   | kategori_yang_dijabarkan |
|----|--------|--------------------------|
| 1  | Laptop | Elektronik               |
| 2  | HP     | Elektronik               |
| 3  | Tablet | Elektronik               |
| 4  | Kaos   | Pakaian                  |

Kata "Elektronik" ditulis berulang-ulang. Borong. Kalau salah ngetik jadi "Electronic", data jadi rusak.

**Dengan relasi (ringkas & rapi):**

Tabel `categories`:

| id | nama        |
|----|-------------|
| 1  | Elektronik  |
| 2  | Pakaian     |

Tabel `products`:

| id | nama   | category_id |
|----|--------|-------------|
| 1  | Laptop | 1           |
| 2  | HP     | 1           |
| 3  | Tablet | 1           |
| 4  | Kaos   | 2           |

"Elektronik" ditulis sekali di tabel `categories`. Tabel `products` cuma tulis angka `1` sebagai penghubung. Hemat, rapi, dan mudah diubah.

## Analogi Lagi: Nomor Induk Siswa (NIS)

Di sekolah, kamu tidak menulis **nama lengkap guru** di rapor tiap mapel. Kamu cuma tulis **kode guru** (misal G-012). Kode G-012 itu "foreign key" yang merujuk ke data guru di tabel guru.

Sama persis: `category_id` di tabel products merujuk ke id di tabel categories.

## Inti Pelajaran Tahap 2

> Relasi = cara menghubungkan dua tabel memakai kolom id (foreign key), supaya tidak ada data berulang dan database tetap rapi.

Contoh yang akan kita pakai:

- Tabel `categories` menyimpan daftar kategori.
- Tabel `products` menyimpan produk, dan punya kolom `category_id` untuk menunjuk ke kategori.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke langkah berikutnya: **memahami jenis relasi one-to-many antara kategori dan produk**?

Di tahap 3 kita akan pelajari:

- Kenapa relasi kategori <-> produk disebut **one-to-many**.
- Bedanya dengan one-to-one atau many-to-many.
- Analogi: "Satu induk ayam punya banyak anak ayam."

Ketik **"lanjut"** kalau siap lanjut ke `tahap-03-one-to-many.md`.
