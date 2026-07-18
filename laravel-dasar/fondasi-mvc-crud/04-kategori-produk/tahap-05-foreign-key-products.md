# Tahap 5: Tambah Kolom `category_id` di Tabel Products + Foreign Key

## Tujuan Tahap Ini

Di tahap 4 kita sudah membuat tabel `categories`. Sekarang kita harus **menghubungkan** tabel `products` ke tabel `categories`.

Caranya: tambah kolom baru bernama `category_id` di tabel `products`. Kolom ini akan menjadi **penghubung** (foreign key) ke tabel `categories`.

## Analogi: Nomor Kamar di Hotel

Bayangkan kamu tamu hotel. Kamu tidak bawa nama kamar sendiri, kamu hanya dikasih **nomor kunci kamar**, misalnya kamar 312.

- Kamu (produk) tinggal di **satu** kamar tertentu.
- Nomor 312 di kunci itu **menunjuk** ke kamar fisik di lantai 3.

Sama seperti:

- Produk (Laptop) punya `category_id = 1`.
- Angka `1` itu **menunjuk** ke kategori "Elektronik" di tabel `categories`.

## Apa itu Foreign Key?

**Foreign key** = kolom di sebuah tabel yang **menunjuk** ke primary key di tabel lain.

| Tabel sumber  | Kolom sumber  | Tabel tujuan | Kolom tujuan |
|---------------|---------------|--------------|--------------|
| `products`    | `category_id` | `categories` | `id`         |

Aturan foreign key:

- Nilai `category_id` di tabel products **harus ada** di kolom `id` tabel categories.
- Tidak boleh `category_id = 99` kalau di tabel categories tidak ada id 99.

Ini menjaga **integritas data**: tidak ada produk yang numpang ke kategori yang tidak ada.

## Dua Pendekatan: Edit Migration Lama vs Buat Migration Baru

Ada dua cara menambah kolom `category_id`:

### Cara A: Edit Migration Lama (hanya di awal development)

Kalau project masih baru dan tabel products belum terisi data penting, kamu bisa:

1. Rollback semua migration.
2. Edit file migration `create_products_table`.
3. Jalankan ulang migration.

Kelemahan: data yang sudah ada akan hilang. Cocok untuk tahap belajar atau project baru.

### Cara B: Buat Migration Baru (yang benar di production)

Buat migration baru yang **mengubah** tabel yang sudah ada:

```bash
php artisan make:migration add_category_id_to_products_table
```

Migration ini hanya menambah kolom, tanpa mengganggu data yang sudah ada. **Ini yang direkomendasikan.**

## Kita Pakai Cara B: Migration Baru

### Langkah 1: Buat Migration Baru

Di terminal:

```bash
php artisan make:migration add_category_id_to_products_table
```

Konvensi nama: `add_namakolom_to_namatabel_table`.

### Langkah 2: Edit File Migration

Buka file:

```
database/migrations/2026_07_18_010000_add_category_id_to_products_table.php
```

Isi default:

```php
public function up(): void
{
    Schema::table('products', function (Blueprint $table) {
        //
    });
}
```

Ubah menjadi:

```php
public function up(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->foreignId('category_id')
                  ->constrained()
                  ->onDelete('cascade');
    });
}
```

## Penjelasan Bagian Per Bagian

### `Schema::table(...)` vs `Schema::create(...)`

| Perintah              | Dipakai untuk                       |
|-----------------------|-------------------------------------|
| `Schema::create(...)` | Membuat tabel baru                  |
| `Schema::table(...)`  | Mengubah tabel yang sudah ada       |

Karena tabel `products` sudah ada, kita pakai `Schema::table(...)`.

### `$table->foreignId('category_id')`

| Bagian        | Arti                                                              |
|---------------|-------------------------------------------------------------------|
| `foreignId`   | Bikin kolom bertipe bigint unsigned, mirip `id()` tapi ditandai sebagai foreign key |
| `'category_id'` | Nama kolomnya. Konvensi Laravel: nama_tabel_tunggal + `_id`      |

Konvensi Laravel:

- Tabel tujuan: `categories` (jamak).
- Model: `Category` (tunggal).
- Foreign key: `category_id` (model lowercase + `_id`).

Kalau kamu ikuti konvensi ini, Laravel bisa **menebak** tabel tujuannya otomatis.

### `->constrained()`

Arti: "Bikin foreign key constraint otomatis ke tabel yang sesuai konvensi."

Karena kolomnya `category_id`, Laravel otomatis tahu harus hubungkan ke tabel `categories` kolom `id`.

Tanpa `constrained()`, Laravel hanya bikin kolom biasa, tanpa aturan foreign key.

### `->onDelete('cascade')`

Arti: "Kalau sebuah kategori dihapus, otomatis hapus semua produk di kategori itu."

Pilihan perilaku `onDelete`:

| Nilai        | Perilaku                                                                 |
|--------------|--------------------------------------------------------------------------|
| `cascade`    | Kategori dihapus -> semua produknya juga ikut dihapus                     |
| `restrict`   | Kategori tidak boleh dihapus kalau masih ada produk (default kalau kosong)|
| `set null`   | Kategori dihapus -> `category_id` produk jadi NULL                       |
| `null`       | Sama dengan `set null`                                                    |

Kita pakai `cascade` agar tidak ada produk yatim (produk tanpa kategori) tersisa.

> **Catatan:** Untuk `set null`, kolom harus nullable: `$table->foreignId('category_id')->nullable()->constrained()->onDelete('set null');`

## Method `down()`

Jangan lupa isi method `down()` supaya migration bisa di-rollback:

```php
public function down(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->dropForeign(['category_id']);
        $table->dropColumn('category_id');
    });
}
```

| Baris                       | Arti                                            |
|-----------------------------|-------------------------------------------------|
| `dropForeign(['category_id'])` | Hapus aturan foreign key-nya dulu             |
| `dropColumn('category_id')`    | Baru hapus kolomnya                           |

Urutan penting: hapus foreign key dulu, baru hapus kolom. Kalau tidak, database protes.

## Jalankan Migration

Di terminal:

```bash
php artisan migrate
```

Output:

```
2026_07_18_010000_add_category_id_to_products_table ........... 45ms DONE
```

## Cek Hasil di phpMyAdmin

Buka tabel `products`, lihat struktur. Akan muncul kolom baru `category_id`.

| Field        | Type          | Null | Key | Extra          |
|--------------|---------------|------|-----|----------------|
| id           | bigint        | NO   | PRI | auto_increment |
| name         | varchar(255)  | NO   |     |                |
| price        | ...           | ...  |     |                |
| category_id  | bigint unsigned | NO | MUL |                | <-- KOLON BARU
| created_at   | timestamp     | YES  |     |                |
| updated_at   | timestamp     | YES  |     |                |

Pada kolom `category_id`, di bagian Key terdapat **MUL** yang artinya ini foreign key.

## Tampilan Visual: Dua Tabel yang Sudah Terhubung

```
Tabel: categories                    Tabel: products
+----+-------------+                 +----+--------+-------------+
| id | name        |                 | id | name   | category_id |
+----+-------------+                 +----+--------+-------------+
| 1  | Elektronik  | <--- di tunjuk | 1  | Laptop | 1           |
| 2  | Pakaian     | <--- di tunjuk | 2  | Kaos   | 2           |
| 3  | Makanan     | <--- di tunjuk | 3  | Roti   | 3           |
| 4  | Buku        | <--- di tunjuk | 4  | Novel  | 4           |
+----+-------------+                 +----+--------+-------------+
```

Setiap `category_id` di tabel `products` **menunjuk** ke salah satu `id` di tabel `categories`.

## Tips Penting

### 1. Urutan Migration Penting

Tabel `categories` harus dibuat **lebih dulu** dari penambahan `category_id` di tabel `products`. Kalau tidak, foreign key tidak bisa dibuat karena tabel tujuan belum ada.

Laravel mengurutkan migration **berdasarkan timestamp** di nama file, jadi pastikan tanggal di file migration categories lebih awal dari file add_category_id.

### 2. Kalau Migration Gagal

Error umum:

```
SQLSTATE[HY000]: General error: 1215 Cannot add foreign key constraint
```

Penyebabnya biasanya:

- Tabel `categories` belum ada.
- Tipe data `id` di categories berbeda dengan `category_id` di products.
- Kamu pakai `foreignId(...)` tapi tabel tujuan pakai `id($type)` yang bukan bigint unsigned.

Solusi paling gampang di tahap belajar:

```bash
php artisan migrate:fresh
```

Peringatan: ini akan **menghapus semua tabel dan data**, lalu menjalankan ulang semua migration dari awal. Cocok kalau masih development, **jangan dipakai di production**.

## Inti Pelajaran Tahap 5

> Foreign key = kolom `category_id` di tabel products yang menunjuk ke kolom `id` di tabel categories. Ini yang menjadikan relasi "nyata" di database.

Yang sudah kita lakukan:

1. Buat migration baru: `add_category_id_to_products_table`.
2. Pakai `$table->foreignId('category_id')->constrained()->onDelete('cascade')`.
3. Isi method `down()` untuk rollback.
4. Jalankan `php artisan migrate`.

Sekarang tabel `products` punya kolom `category_id` dan **terhubung** ke tabel `categories`. Tapi tabel-tabel ini masih kosong datanya.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke langkah berikutnya: **membuat Model `Category` dan menambah relasi `hasMany`/`belongsTo`**?

Di tahap 6 kita akan:

- Membuat model `Category` dengan `php artisan make:model Category`.
- Menambah method `products()` dengan relasi `hasMany` di model Category.
- Menambah method `category()` dengan relasi `belongsTo` di model Product.
- Mencoba ambil data via relasi memakai Tinker.

Ketik **"lanjut"** kalau siap.
