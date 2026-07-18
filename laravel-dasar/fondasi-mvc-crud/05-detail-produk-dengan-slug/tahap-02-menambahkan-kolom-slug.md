# Tahap 2: Menambahkan Kolom Slug ke Tabel Products

## Tujuan Tahap Ini

Pada Tahap 1 kita sudah mengenal slug. Sekarang kita akan menyiapkan tempat
untuk menyimpan slug pada setiap produk.

Di tahap ini kita hanya akan:

1. Membuat migration baru.
2. Menambahkan kolom `slug` ke tabel `products`.
3. Menjalankan migration.

Kita belum membuat slug secara otomatis dan belum mengubah URL produk.

## Kenapa Slug Perlu Disimpan?

Setiap produk memiliki nama dan slug:

| Nama produk        | Slug                 |
|--------------------|----------------------|
| Sepatu Lari Pria   | `sepatu-lari-pria`   |
| Kaos Hitam Polos   | `kaos-hitam-polos`   |
| Tas Sekolah Anak   | `tas-sekolah-anak`   |

Laravel perlu mengetahui slug milik setiap produk. Karena itu, slug disimpan
di dalam tabel `products`.

## Analogi Sederhana: Menambah Kolom pada Buku Data

Bayangkan data produk dicatat dalam sebuah buku:

| ID | Nama Produk      | Harga  |
|----|------------------|--------|
| 1  | Sepatu Lari Pria | 350000 |

Saat ini buku tersebut belum memiliki tempat untuk mencatat slug. Kita perlu
menambahkan satu kolom baru:

| ID | Nama Produk      | Slug                 | Harga  |
|----|------------------|----------------------|--------|
| 1  | Sepatu Lari Pria | `sepatu-lari-pria`   | 350000 |

Migration berfungsi sebagai instruksi kepada database untuk menambahkan kolom
tersebut.

## Langkah 1: Membuat Migration

Buka terminal di dalam folder project Laravel, lalu jalankan:

```bash
php artisan make:migration add_slug_to_products_table
```

Laravel akan membuat file baru di:

```text
database/migrations/
```

Nama file diawali tanggal dan waktu, contohnya:

```text
2026_07_18_120000_add_slug_to_products_table.php
```

Tanggal dan waktu pada nama file milikmu mungkin berbeda. Itu normal.

## Langkah 2: Mengisi Migration

Buka file migration yang baru dibuat. Ubah method `up()` dan `down()` menjadi:

```php
public function up(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->string('slug')->nullable()->after('name');
    });
}

public function down(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->dropColumn('slug');
    });
}
```

## Penjelasan Kode

### `Schema::table('products', ...)`

Perintah ini berarti:

> Ubah tabel `products` yang sudah ada.

Kita menggunakan `Schema::table`, bukan `Schema::create`, karena tabel
`products` sudah dibuat pada materi CRUD sebelumnya.

### `$table->string('slug')`

Perintah ini menambahkan kolom bernama `slug` dengan tipe teks pendek.

Tipe `string` cocok karena slug berisi teks seperti:

```text
sepatu-lari-pria
```

### `->nullable()`

`nullable()` berarti kolom slug untuk sementara boleh kosong.

Ini diperlukan karena produk lama mungkin sudah ada di database, tetapi belum
memiliki slug. Pada tahap berikutnya kita akan mengisi slug tersebut.

### `->after('name')`

Bagian ini menempatkan kolom `slug` setelah kolom `name` agar struktur tabel
lebih mudah dibaca.

Urutannya menjadi:

```text
id, name, slug, description, price, stock, ...
```

`after('name')` hanya mengatur posisi kolom. Bagian ini tidak memengaruhi cara
slug bekerja.

### Method `down()`

Method `down()` berisi kebalikan dari `up()`.

Jika migration dibatalkan, baris berikut akan menghapus kolom slug:

```php
$table->dropColumn('slug');
```

## Langkah 3: Menjalankan Migration

Jalankan:

```bash
php artisan migrate
```

Jika berhasil, terminal akan menampilkan status `DONE`.

## Langkah 4: Memeriksa Hasil

Kamu dapat memeriksa struktur tabel `products` melalui phpMyAdmin atau aplikasi
database yang digunakan.

Kolom `slug` sekarang sudah tersedia:

| Kolom         | Contoh nilai          |
|---------------|-----------------------|
| `id`          | `1`                   |
| `name`        | `Sepatu Lari Pria`    |
| `slug`        | `NULL`                |
| `description` | `Sepatu ringan...`    |

Nilai `NULL` belum menjadi masalah. Itu berarti slug belum diisi. Kita akan
mengisinya pada tahap berikutnya.

## Checklist Tahap 2

- [ ] Migration `add_slug_to_products_table` sudah dibuat.
- [ ] Method `up()` menambahkan kolom `slug`.
- [ ] Method `down()` menghapus kolom `slug`.
- [ ] Perintah `php artisan migrate` berhasil.
- [ ] Kolom `slug` terlihat pada tabel `products`.

## Inti Tahap 2

> Slug memerlukan tempat penyimpanan. Kita menambahkan kolom `slug` ke tabel
> `products` menggunakan migration baru.

Pada tahap ini slug masih boleh kosong. Kita belum membuat slug otomatis,
belum memastikan slug unik, dan belum menggunakannya pada URL.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke **Tahap 3: membuat slug otomatis dari nama
produk**?

Ketik **"lanjut"** jika sudah siap.
