# Tahap 1: Apa itu Kategori Produk dan Kenapa Produk Perlu Dikelompokkan?

## Masalah

Produk belum bisa dikelompokkan, sehingga semua produk bercampur menjadi satu.

Contoh produk yang bercampur:

- baju
- celana
- sepatu
- aksesoris

Jika tidak ada kategori, user akan sulit mencari produk berdasarkan kelompoknya.

## Analogi Sederhana: Rak Belanja di Minimarket

Bayangkan kamu masuk ke sebuah minimarket. Di sana ada **ratusan produk** ditumpuk di satu rak besar tanpa label apa pun.

- Baju, sabun, roti, novel, dan laptop semuanya tercampur jadi satu.
- Kamu mau beli roti, tapi harus mengais-ngais dari atas ke bawah.
- Capek, kan?

Nah, minimarket yang rapi membagi produk ke dalam **kelompok-kelompok**:

- Rak **Makanan**: roti, mi instan, coklat
- Rak **Minuman**: air mineral, teh, kopi
- Rak **Pakaian**: baju, celana, sepatu
- Rak **Elektronik**: laptop, charger, kabel

Kelompok itulah yang kita sebut **kategori**.

## Jadi, Apa Itu Kategori Produk?

**Kategori produk** adalah label atau pengelompokan yang membatasi produk berdasarkan **jenis** atau **kemiripan**.

Kalau di tabel database, kategori adalah **tabel terpisah** yang isinya nama-nama kelompok:

| id | nama_kategori |
|----|---------------|
| 1  | Elektronik    |
| 2  | Pakaian       |
| 3  | Makanan       |
| 4  | Buku          |

Setiap produk nanti akan **mengacu** ke salah satu baris di tabel kategori ini.

## Kenapa Produk Perlu Kategori?

Tanpa kategori, semua masalah ini muncul:

1. **Sulit mencari produk.** User harus scrolling semua produk untuk menemukan satu kaos.
2. **Sulit memfilter.** Tidak bisa tampilkan "hanya produk Pakaian".
3. **Tampilan berantakan.** Produk elektronik dan makanan tampil bercampur.
4. **Sulit membuat menu navigasi.** Tidak bisa bikin menu dropdown "Pakaian", "Makanan", dst.
5. **Sulit membuat laporan.** Tidak bisa hitung "ada berapa produk di kategori Elektronik?".

Dengan kategori, semua ini jadi mudah.

## Contoh Konkret

Bayangkan tabel produk **tanpa kategori**:

| id | nama   | harga    |
|----|--------|----------|
| 1  | Laptop | 10.000   |
| 2  | Kaos   | 50       |
| 3  | Roti   | 10       |
| 4  | Novel  | 75       |

Kamu tidak tahu mana yang makanan, mana yang pakaian. Semua bercampur.

Sekarang bayangkan tabel produk **dengan kategori**:

| id | nama   | harga  | kategori    |
|----|--------|--------|-------------|
| 1  | Laptop | 10.000 | Elektronik  |
| 2  | Kaos   | 50     | Pakaian     |
| 3  | Roti   | 10     | Makanan     |
| 4  | Novel  | 75     | Buku        |

Sekarang jelas:

- Laptop adalah **Elektronik**.
- Kaos adalah **Pakaian**.
- Roti adalah **Makanan**.
- Novel adalah **Buku**.

Cari produk Pakaian? Tinggal filter kolom kategori = "Pakaian". Selesai.

## Inti Pelajaran Tahap 1

> Kategori = cara mengelompokkan produk supaya mudah dicari, difilter, dan ditampilkan secara rapi.

Tanpa kategori, produk bercampur. Dengan kategori, produk tertata.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke langkah berikutnya: **memahami relasi one-to-many antara kategori dan produk**?

Di langkah berikutnya kita akan belajar:

- Apa itu relasi database (dengan analogi sederhana).
- Kenapa disebut **one-to-many** (satu kategori, banyak produk).
- Bagaimana satu kategori bisa punya banyak produk, tapi satu produk hanya masuk ke satu kategori.

Ketik **"lanjut"** kalau siap melanjutkan ke `tahap-02-relasi-database.md`.
