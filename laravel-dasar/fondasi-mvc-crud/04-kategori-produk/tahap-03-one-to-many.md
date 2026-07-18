# Tahap 3: Relasi One-to-Many (Satu ke Banyak)

## Analogi Sederhana: Induk Ayam dan Anak Ayam

Bayangkan kamu di sebuah peternakan ayam.

- Ada **satu induk ayam**.
- Induk ayam ini punya **banyak anak ayam**: 5, 10, atau bahkan 20 ekor.

Perhatikan:

- Satu induk -> **banyak** anak. (one-to-many)
- Tapi satu anak ayam -> **hanya punya satu** induk. (tidak bisa punya dua ibu biologis)

Ini persis seperti relasi **kategori dan produk**:

- **Satu kategori** (induk) bisa berisi **banyak produk** (anak).
- **Satu produk** (anak) **hanya masuk ke satu kategori** (induk).

## Apa itu One-to-Many?

**One-to-many** = hubungan di mana **satu baris di tabel A** bisa dihubungkan ke **banyak baris di tabel B**, tapi **satu baris di tabel B hanya boleh merujuk ke satu baris di tabel A**.

Dalam kasus kita:

- Tabel A = `categories` (kategori)
- Tabel B = `products` (produk)

Aturannya:

- **Satu kategori** bisa punya **banyak produk**.
- **Satu produk** hanya boleh masuk **satu kategori**.

## Contoh Konkret

### Satu Kategori, Banyak Produk

Kategori **Elektronik** (id = 1) bisa berisi:

| id | nama    | category_id |
|----|---------|-------------|
| 1  | Laptop  | 1           |
| 2  | HP      | 1           |
| 3  | Tablet  | 1           |
| 4  | Charger | 1           |

Lihat kolom `category_id`. Semua produk di atas nilainya `1`, artinya semua milik kategori **Elektronik**.

Jadi, **satu kategori (Elektronik) -> banyak produk (Laptop, HP, Tablet, Charger)**.

### Satu Produk, Satu Kategori

Sebaliknya, produk **Laptop** hanya punya satu `category_id`, yaitu `1`.

Laptop tidak bisa masuk ke Elektronik sekaligus Pakaian. Produk hanya boleh punya **satu induk kategori**.

## Cara Membaca Relasinya

Pikirkan dua pertanyaan ini:

1. **Satu kategori bisa punya berapa produk?** -> **Banyak**. (one-to-many dari sisi kategori)
2. **Satu produk bisa masuk ke berapa kategori?** -> **Hanya satu**. (banyak-to-one dari sisi produk)

Jawaban pertama "banyak" dan jawaban kedua "satu" -> inilah **one-to-many**.

## Perbandingan dengan Jenis Relasi Lain

Supaya makin jelas, mari bandingkan dengan jenis relasi lain.

### One-to-One (Satu ke Satu)

Contoh: **User dan Profil**.

- Satu user hanya punya **satu** profil.
- Satu profil hanya milik **satu** user.

Bukan one-to-many karena tidak ada yang punya "banyak".

### One-to-Many (Satu ke Banyak) <- INI YANG KITA PAKAI

Contoh: **Kategori dan Produk**.

- Satu kategori punya **banyak** produk.
- Satu produk milik **satu** kategori.

### Many-to-Many (Banyak ke Banyak)

Contoh: **Siswa dan Mata Pelajaran**.

- Satu siswa ambil **banyak** mapel (Matematika, Bahasa, Sains).
- Satu mapel diambil oleh **banyak** siswa (Budi, Sinta, Andi).

Ini butuh **tabel perantara** (pivot), dan **bukan** yang kita pakai untuk kategori-produk.

## Visualisasi Sederhana

```
[Kategori: Elektronik]
        |
        +-- Product: Laptop
        +-- Product: HP
        +-- Product: Tablet
        +-- Product: Charger

[Kategori: Pakaian]
        |
        +-- Product: Kaos
        +-- Product: Celana

[Kategori: Makanan]
        |
        +-- Product: Roti
        +-- Product: Mi Instan
```

Lihat:

- Satu kotak kategori punya banyak "anak" produk di bawahnya.
- Tapi satu produk (misal Kaos) tidak pernah muncul di bawah dua kategori sekaligus.

Inilah **one-to-many**.

## Cara Laravel Menyebut Relasi Ini

Nanti di Laravel kita akan tulis relasi ini dengan dua istilah:

- **`hasMany`** -> dari sisi kategori. "Satu kategori **memiliki banyak** (has many) produk."
- **`belongsTo`** -> dari sisi produk. "Satu produk **milik** (belongs to) satu kategori."

Kedua istilah ini menggambarkan **sisi yang berbeda** dari relasi yang sama.

- Kategori berkata: "Aku punya banyak produk." (`hasMany`)
- Produk berkata: "Aku milik kategori ini." (`belongsTo`)

Sama seperti:

- Ibu berkata: "Aku punya banyak anak." (`hasMany`)
- Anak berkata: "Aku milik ibu ini." (`belongsTo`)

## Inti Pelajaran Tahap 3

> One-to-many = satu kategori bisa berisi banyak produk, tapi satu produk hanya boleh masuk satu kategori.

Analogi: induk ayam dan anak ayam. Satu induk banyak anak, satu anak satu induk.

Di Laravel relasi ini ditulis:

- Kategori: `hasMany` Product
- Produk: `belongsTo` Category

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke langkah berikutnya: **membuat migration tabel categories**?

Di tahap 4 kita akan mulai coding Laravel:

- Membuat migration baru untuk tabel `categories`.
- Menjelaskan arti kolom-kolom di migration (id, name, timestamps).
- Menjalankan migration supaya tabel benar-benar dibuat di database.

Ketik **"lanjut"** kalau siap masuk ke kode.
