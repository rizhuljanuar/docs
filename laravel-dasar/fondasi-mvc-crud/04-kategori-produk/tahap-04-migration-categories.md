# Tahap 4: Membuat Migration Tabel Categories

## Apa itu Migration di Laravel?

### Analogi Sederhana: Cetak Biru Rumah

Bayangkan kamu mau membangun rumah. Sebelum tukang mulai menumpuk batu bata, kamu butuh **cetak biru** (blueprint):

- Berapa kamar tidur?
- Dimana pintu?
- Ketinggian langit-langit?

Cetak biru ini disepakati dulu, baru tukang bangun rumahnya sesuai gambar.

Di Laravel, **migration** itu = **cetak biru untuk tabel database**.

Migration adalah **file PHP** yang berisi instruksi:

> "Buat tabel dengan nama `categories`, kolom-kolomnya: id, name, timestamps."

Kamu tidak perlu buka phpMyAdmin atau ketik SQL manual. Cukup tulis migration, lalu jalankan perintah, dan Laravel akan membuat tabelnya otomatis.

## Cara Membuat Migration Baru

### Langkah 1: Jalankan Perintah Artisan

Buka terminal di folder project Laravel, lalu ketik:

```bash
php artisan make:migration create_categories_table
```

Penjelasan bagian per bagian:

| Bagian              | Arti                                                          |
|---------------------|---------------------------------------------------------------|
| `php artisan`       | Memanggil alat bantu Laravel lewat terminal                   |
| `make:migration`    | Perintah untuk membuat file migration baru                    |
| `create_categories_table` | Nama migration. Konvensi: `create_nama_table_table` |

### Langkah 2: Lihat File yang Dibuat

Laravel akan membuat file baru di folder:

```
database/migrations/2026_07_18_000000_create_categories_table.php
```

Tanggal dan jam otomatis ditambahkan di depan nama file, supaya migration dijalankan sesuai urutan waktu.

### Langkah 3: Buka File Migration

Isi file kurang lebih seperti ini:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('categories');
    }
};
```

## Penjelasan Bagian Per Bagian

### `up()` = Membuat Tabel

Method `up()` dijalankan **saat migration berjalan**. Di sinilah kita mendefinisikan struktur tabel.

```php
public function up(): void
{
    Schema::create('categories', function (Blueprint $table) {
        $table->id();
        $table->timestamps();
    });
}
```

| Bagian                          | Arti                                                            |
|---------------------------------|-----------------------------------------------------------------|
| `Schema::create('categories')`  | Buat tabel baru bernama `categories`                            |
| `Blueprint $table`              | Cetak biru tabel, kita pakai untuk menambah kolom               |
| `$table->id()`                  | Kolom `id` otomatis (primary key, auto increment, bigint)       |
| `$table->timestamps()`          | Tambah 2 kolom: `created_at` dan `updated_at` otomatis          |

### `down()` = Membatalkan/Menghapus Tabel

Method `down()` dijalankan **kalau migration dibatalkan** (rollback).

```php
public function down(): void
{
    Schema::dropIfExists('categories');
}
```

Arti: "Kalau dibatalkan, hapus tabel `categories` kalau ada." Berguna kalau kita salah bikin dan mau ulang.

## Tambah Kolom `name` untuk Nama Kategori

Sekarang kita ubah migration supaya tabel `categories` punya kolom `name`:

```php
public function up(): void
{
    Schema::create('categories', function (Blueprint $table) {
        $table->id();
        $table->string('name');   // <-- tambahkan baris ini
        $table->timestamps();
    });
}
```

Penjelasan `$table->string('name')`:

| Bagian        | Arti                                                            |
|---------------|-----------------------------------------------------------------|
| `string(...)` | Kolom bertipe VARCHAR (teks pendek, maksimal 255 karakter)      |
| `'name'`      | Nama kolomnya `name`                                            |

Cocok untuk menyimpan nama kategori seperti "Elektronik", "Pakaian", dst.

## Hasil Tabel Setelah Migration

Kalau dijalankan, tabel `categories` akan terlihat seperti ini:

| Kolom       | Tipe          | Keterangan                              |
|-------------|---------------|-----------------------------------------|
| `id`        | bigint        | Primary key, auto increment             |
| `name`      | varchar(255)  | Nama kategori                           |
| `created_at`| timestamp     | Kapan kategori dibuat                   |
| `updated_at`| timestamp     | Kapan kategori terakhir diubah          |

## Cara Menjalankan Migration

### Langkah 1: Pastikan Database Sudah dikonfigurasi

Cek file `.env` di folder project Laravel, bagian:

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nama_database_kamu
DB_USERNAME=root
DB_PASSWORD=
```

Sesuaikan `DB_DATABASE`, `DB_USERNAME`, dan `DB_PASSWORD` dengan setting database kamu.

### Langkah 2: Jalankan Migration

Di terminal:

```bash
php artisan migrate
```

Laravel akan:

1. Membaca semua file migration yang belum dijalankan.
2. Membuat tabel sesuai instruksi.
3. Mencatat tabel mana yang sudah dibuat di tabel khusus `migrations`.

### Output yang Muncul

Kira-kira seperti ini:

```
2026_07_18_000000_create_categories_table ..................... 52ms DONE
```

Artinya tabel `categories` sudah berhasil dibuat di database.

## Cara Cek Apakah Tabel Sudah Dibuat

Buka phpMyAdmin (atau DB client apapun), masuk ke database kamu. Kamu akan melihat tabel baru:

- `categories`
- `migrations` (tabel Laravel untuk catatan migration)

Struktur `categories`:

| Field      | Type         | Null | Key | Extra          |
|------------|--------------|------|-----|----------------|
| id         | bigint       | NO   | PRI | auto_increment |
| name       | varchar(255) | NO   |     |                |
| created_at | timestamp    | YES  |     |                |
| updated_at | timestamp    | YES  |     |                |

## Tips Penting

### 1. Jangan Edit Migration yang Sudah Dijalankan

Kalau migration sudah dijalankan dan kamu ingin mengubah struktur tabel:

- Buat **migration baru** untuk perubahan (misal `add_description_to_categories_table`).
- **Bukan** edit file lama.

Kecuali kalau kamu masih dalam tahap development, bisa pakai:

```bash
php artisan migrate:rollback
```

Ini membatalkan migration terakhir (menjalankan `down()`), lalu kamu bisa edit dan `migrate` ulang.

### 2. Konvensi Nama Tabel

Laravel pakai konvensi:

- Nama tabel **jamak** (plural): `categories`, `products`, `users`.
- Nama model **tunggal** (singular): `Category`, `Product`, `User`.

Jadi tabel `categories` nantinya akan dipasangkan dengan model `Category`.

## Inti Pelajaran Tahap 4

> Migration = cetak biru tabel database dalam bentuk file PHP.

Langkah yang sudah kita lakukan:

1. Buat migration: `php artisan make:migration create_categories_table`.
2. Tambah kolom `name` di method `up()`.
3. Jalankan dengan `php artisan migrate`.

Sekarang tabel `categories` sudah ada di database, tapi masih kosong datanya.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke langkah berikutnya: **menambah kolom `category_id` di tabel `products` dan membuat foreign key**?

Di tahap 5 kita akan:

- Mengubah migration tabel `products` yang sudah ada (atau buat migration baru untuk menambah kolom).
- Menambah kolom `category_id` sebagai foreign key.
- Menjelaskan apa itu foreign key dan kenapa penting.

Ketik **"lanjut"** kalau siap lanjut.
