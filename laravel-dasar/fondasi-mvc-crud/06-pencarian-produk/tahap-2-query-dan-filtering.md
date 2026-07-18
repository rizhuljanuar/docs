# Tahap 2 — Memahami Query Database dan Filtering

> Materi Laravel Dasar · Topik 6: Pencarian Produk
> Mentor: penjelasan bertahap untuk pemula total
> Prasyarat: sudah baca **Tahap 1** (apa itu pencarian produk)

---

## 1. Goal Tahap Ini

Di akhir Tahap 2 ini, kamu diharapkan paham:

- Apa itu **query database** (dengan analogi sederhana)
- Apa itu **filtering** (penyaringan data)
- Arti dari `where('name', 'like', '%laptop%')` dalam bahasa manusia
- Kenapa kita pakai **`like`**, bukan **`=`**
- Arti simbol `%` (persen) di pencarian

**Belum ada kode Laravel di tahap ini.** Kita pahami dulu. Baru di Tahap 3 kita nulis kodenya.

---

## 2. Apa Itu "Query Database"?

### Analogi: Memesan ke Petugas Toko

Bayangkan kamu di apotek. Kamu tidak boleh masuk ke gudan obat sendiri.
Yang kamu lakukan: **meminta** ke petugas.

> "Tolong ambilkan saya obat paracetamol, yang harga di bawah 10.000."

Petugas masuk ke gudan, mencari, lalu **menyerahkan** obat itu ke kamu.

Dalam dunia database, **petugas itu = sistem database (MySQL)**. Dan **permintaan kamu = query**.

> **Query = permintaan yang kita kirim ke database supaya database melakukan sesuatu terhadap data.**

Database bisa melakukan 4 hal utama (CRUD juga):

| Aksi             | Query SQL umum         | Arti                       |
|------------------|------------------------|----------------------------|
| Ambil data       | `SELECT`               | "Berikan saya data..."     |
| Tambah data      | `INSERT`               | "Simpan data baru."        |
| Ubah data        | `UPDATE`               | "Ubah data yang ada."      |
| Hapus data       | `DELETE`               | "Hapus data ini."          |

Untuk fitur pencarian, kita pakai **`SELECT`** (mengambil data), lalu **menyaring** data itu supaya hanya yang cocok yang kembali.

---

## 3. Contoh Query Pencarian dalam Bahasa SQL

Misal kita punya tabel `produk` seperti Tahap 1:

| id | nama                 | kategori   | harga      |
|----|----------------------|------------|------------|
| 1  | Laptop Asus ROG      | Elektronik | 15.000.000 |
| 2  | Laptop Acer Predator | Elektronik | 18.000.000 |
| 3  | Kaos Hitam Polos     | Fashion    | 50.000     |
| 4  | Sepatu Running Nike  | Olahraga   | 750.000    |

Kalau kita ingin **mengambil semua produk**, query SQL-nya:

```sql
SELECT * FROM produk;
```

Arti: *"Berikan saya SEMUA kolom (`*`) dari tabel `produk`."*

Hasilnya: **4 baris** (semua produk muncul).

---

### Sekarang, kita mau HANYA produk yang namanya = "Laptop Asus ROG"

```sql
SELECT * FROM produk WHERE nama = 'Laptop Asus ROG';
```

Arti: *"Berikan saya semua kolom dari tabel produk, **DI MANA (WHERE)** nilai kolom `nama` sama persis dengan 'Laptop Asus ROG'."*

Hasilnya: **1 baris** saja (produk #1).

Kata kunci `WHERE` ini = **filter** (penyaring).

---

## 4. Apa Itu "Filtering"?

**Filtering** = menyaring data supaya hanya data yang **memenuhi syarat tertentu** yang muncul.

### Analogi: Saring Pasir

Bayangkan kamu punya ember berisi campuran **pasir + batu + kerikil**.
Kamu mau hanya **pasir halus**.
Kamu pakai **ayakan** (saringan) dengan lubang kecil.

Batu besar dan kerikil **tersangkut** di atas (tidak lewat).
Hanya **pasir halus** yang lewat ke ember di bawah.

Ayakan itu = **filter**.

Dalam query database, **`WHERE` adalah ayakan itu**:

```
Semua produk (4 baris)
        ↓
  [ FILTER: nama mengandung 'laptop' ]   ← WHERE
        ↓
Hasil tersaring (2 baris: Laptop Asus + Laptop Acer)
```

Jadi:

> **Filter = ayakan yang menyaring data supaya hanya yang cocok yang lewat.**

---

## 5. Masalah dengan Tanda Sama Dengan (`=`)

Coba perhatikan:

```sql
WHERE nama = 'Laptop Asus ROG'
```

Ini artinya: nama harus **SAMA PERSIS** dengan teks `Laptop Asus ROG`.

Sekarang coba pikir user di aplikasi. User tidak akan hafal nama produk secara persis.
User biasanya hanya **mengetik sebagian**, misal:

- `laptop` (huruf kecil semua)
- `Laptop` (kapital depannya saja)
- `asus`
- `lap` (ketik setengah)
- `ROG`

Kalau kita pakai tanda `=`, user ketik `laptop` → **tidak akan ketemu**, karena di database namanya `Laptop Asus ROG` (kapital `L`).

**Ini masalah besar.** User merasa produk tidak ada, padahal ada.

---

## 6. Solusi: Pakai `LIKE` alih-alih `=`

SQL punya operator khusus untuk pencarian **"mirip / mengandung"**, yaitu **`LIKE`**.

```sql
SELECT * FROM produk WHERE nama LIKE '%laptop%';
```

Bedanya `=` dan `LIKE`:

| Operator | Arti                       | Cocok untuk              |
|----------|----------------------------|--------------------------|
| `=`      | Sama **persis**            | ID, status, angka pasti  |
| `LIKE`   | Mirip / mengandung sebagian| Pencarian teks bebas     |

**Aturan praktis:** untuk fitur **pencarian**, selalu pakai `LIKE`, bukan `=`.

---

## 7. Arti Simbol `%` (Persen)

Simbol `%` di dunia pencarian database = **wildcard** (karakter bebas).

Artinya: *"di sini boleh ada teks apa saja, atau bahkan kosong."*

### Tiga Posisi Tanda `%`

#### A. `%laptop%` → di awal DAN di akhir

Arti: nama produk yang **mengandung kata "laptop" di mana pun posisinya**.

Cocok dengan:
- `Laptop Asus ROG` ✅ (kata "Laptop" di awal)
- `Asus Laptop Gaming` ✅ (kata "Laptop" di tengah)
- `ROG Laptop` ✅ (kata "Laptop" di akhir)
- `LAPTOP` ✅ (huruf besar)
- `laptop` ✅ (huruf kecil)

**Ini yang paling sering dipakai di kotak pencarian.**

#### B. `laptop%` → hanya di akhir

Arti: nama produk yang **diawali dengan** "laptop".

Cocok dengan:
- `Laptop Asus ROG` ✅
- `Laptop Gaming Murah` ✅

Tidak cocok dengan:
- `Asus Laptop Gaming` ❌ (kata "Laptop" di tengah)

**Dipakai untuk fitur "produk yang namanya diawali kata tertentu".**

#### C. `%laptop` → hanya di awal

Arti: nama produk yang **diakhiri dengan** "laptop".

Cocok dengan:
- `Asus Laptop` ✅
- `ROG Laptop` ✅

Tidak cocok dengan:
- `Laptop Asus` ❌

**Jarang dipakai, tapi ada kalanya berguna.**

---

## 8. Visual: Cara Kerja `%laptop%`

```
User ketik di kotak pencarian:  "laptop"
↓
Query terbentuk:  WHERE nama LIKE '%laptop%'
↓
Database memeriksa setiap baris di tabel produk:

  Baris 1: "Laptop Asus ROG"      → mengandung "laptop"? ✅ TAMPIL
  Baris 2: "Laptop Acer Predator" → mengandung "laptop"? ✅ TAMPIL
  Baris 3: "Kaos Hitam Polos"     → mengandung "laptop"? ❌ skip
  Baris 4: "Sepatu Running Nike"  → mengandung "laptop"? ❌ skip
  Baris 5: "Buku Laravel Pemula"  → mengandung "laptop"? ❌ skip
  Baris 6: "Headset Bluetooth"    → mengandung "laptop"? ❌ skip
↓
Hasil yang dikirim balik ke aplikasi: 2 baris (baris 1 & 2)
↓
Browser user menampilkan 2 produk saja
```

---

## 9. Eh, Tapi `LIKE` Itu Case-Sensitive atau Tidak?

Pertanyaan bagus. Jawabannya tergantung pengaturan database (collation):

- Default MySQL dengan collation `utf8mb4_general_ci` → **tidak case-sensitive**.
  - `LIKE '%laptop%'` cocok dengan `Laptop`, `LAPTOP`, `laptop`, `LaPtOp`.
  - Akhiran `_ci` di nama collation = **case insensitive**.

- Kalau collation-nya `utf8mb4_bin` atau akhiran `_cs` → **case-sensitive**.
  - `LIKE '%laptop%'` hanya cocok dengan `laptop` (huruf kecil).

**Untuk pemula:** asumsikan saja **tidak case-sensitive** (default Laravel biasanya begitu). User bisa ketik huruf besar/kecil, tetap ketemu.

---

## 10. Rangkuman Bahasa Manusia dari `where('name', 'like', '%laptop%')`

Kalimat ini dalam bahasa Laravel nanti akan ditulis begini (preview, belum kita tulis kodenya):

```php
->where('nama', 'like', '%laptop%')
```

Dibaca: *"Saring data di mana nilai kolom `nama` **mirip dengan** teks yang **mengandung kata 'laptop' di mana pun posisinya**."*

Sekarang kamu sudah paham:

- **`where`**  → kata kunci untuk **menyaring** (filter)
- **`'nama'`** → kolom yang diperiksa
- **`'like'`** → operator "mirip / mengandung sebagian" (bukan sama persis)
- **`'%laptop%'`** → teks yang dicari, dengan `%` artinya "bebas apa pun di kiri dan kanan"

---

## 11. Kesimpulan Tahap 2

- **Query** = permintaan ke database (mirip memesan ke petugas apotek).
- **Filtering** = menyaring data dengan syarat tertentu (mirip ayakan pasir).
- **`WHERE`** adalah ayakan itu di SQL.
- Untuk pencarian, kita pakai **`LIKE`**, bukan `=`, karena user tidak hafal nama produk persis.
- **`%`** = wildcard = "di sini boleh teks apa pun".
- **`%laptop%`** paling umum dipakai untuk kotak pencarian karena cocok dengan "laptop" di posisi mana pun.
- **Tidak case-sensitive** di pengaturan default MySQL/Laravel.

Sekarang kamu punya bekal konsep yang kuat untuk masuk ke kode Laravel di **Tahap 3**.

---

## Pertanyaan Berikutnya

**Apakah kamu ingin lanjut ke langkah berikutnya: menulis kode pertama `where('name','like',...)` di Controller Produk?**

Pada Tahap 3 kita akan:

- Lihat kode Controller Produk yang sudah ada (yang menampilkan semua produk)
- Tambahkan **satu baris** `->where(...)` untuk menyaring berdasarkan input user
- Pahami bagaimana data dari kotak pencarian (input user) masuk ke Controller

Kita akan mulai ngoding di Tahap 3, tapi **pelan-pelan, satu langkah kecil**.

— **Mentor Laravel**
