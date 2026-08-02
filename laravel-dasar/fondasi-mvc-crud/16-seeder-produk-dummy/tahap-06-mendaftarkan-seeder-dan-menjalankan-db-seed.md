# Tahap 6 — Mendaftarkan Seeder di `DatabaseSeeder` dan Menjalankan `php artisan db:seed`

> Fokus: menghubungkan `CategorySeeder` dan `ProductSeeder`, menjalankannya dalam urutan yang benar, lalu memeriksa data dummy.

Pada tahap 5, kita sudah membuat dua seeder:

```text
database/seeders/CategorySeeder.php
database/seeders/ProductSeeder.php
```

Namun Laravel belum tahu bahwa keduanya harus dijalankan saat kita mengetik `php artisan db:seed`.

Untuk itu, Laravel menyediakan **`DatabaseSeeder`**, yaitu seeder utama.

## Analogi: daftar tugas utama

Bayangkan kita punya dua petugas:

- Petugas A mengisi lima rak kategori.
- Petugas B mengisi 30 product ke rak yang sudah ada.

Kita tetap butuh satu daftar tugas utama yang berkata:

```text
1. Jalankan petugas kategori terlebih dahulu.
2. Setelah selesai, jalankan petugas product.
```

Di Laravel, daftar tugas utama itu adalah:

```text
database/seeders/DatabaseSeeder.php
```

## Daftarkan kedua seeder

Buka file:

```text
database/seeders/DatabaseSeeder.php
```

Isi method `run()` seperti ini:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            CategorySeeder::class,
            ProductSeeder::class,
        ]);
    }
}
```

## Penjelasan kode

| Kode | Arti sederhana |
| --- | --- |
| `class DatabaseSeeder extends Seeder` | Ini adalah seeder utama Laravel. |
| `run()` | Isi method ini dijalankan saat `php artisan db:seed`. |
| `$this->call([...])` | Jalankan seeder lain yang ada di dalam daftar. |
| `CategorySeeder::class` | Jalankan seeder kategori utama. |
| `ProductSeeder::class` | Jalankan seeder 30 Product dummy. |

Urutan di dalam array sangat penting:

```php
$this->call([
    CategorySeeder::class,
    ProductSeeder::class,
]);
```

Laravel menjalankan baris pertama lebih dahulu.

```text
CategorySeeder selesai
        ↓
ProductSeeder mengambil Category yang sudah ada
        ↓
ProductFactory membuat 30 Product dengan category_id yang valid
```

Jangan membalik urutan itu. Jika `ProductSeeder` berjalan sebelum Category tersedia, `$categories->random()` tidak mempunyai Category untuk dipilih.

## Jalankan semua seeder

Setelah `DatabaseSeeder` sudah benar, buka terminal dari folder project Laravel dan jalankan:

```bash
php artisan db:seed
```

Perintah ini mengubah isi database lokal dengan menambahkan data dummy. Pastikan koneksi database pada file `.env` mengarah ke database development atau local, bukan database production.

Alur yang terjadi:

1. Laravel menjalankan `DatabaseSeeder`.
2. `DatabaseSeeder` memanggil `CategorySeeder`.
3. `CategorySeeder` memastikan lima Category utama tersedia.
4. `DatabaseSeeder` memanggil `ProductSeeder`.
5. `ProductSeeder` meminta `ProductFactory` membuat 30 Product dummy.
6. Product dummy disimpan dengan `category_id` yang menunjuk ke salah satu Category.

Output terminal biasanya mirip seperti ini:

```text
INFO  Seeding database.

Database\Seeders\CategorySeeder ... DONE
Database\Seeders\ProductSeeder  ... DONE
```

Nama waktu dan tampilan detail dapat sedikit berbeda, tetapi kata `DONE` menandakan seeder selesai tanpa error.

## Periksa hasil melalui Tinker

Salah satu cara memeriksa data adalah memakai Laravel Tinker.

Di terminal:

```bash
php artisan tinker
```

Lalu jalankan:

```php
Category::count();
```

Hasilnya seharusnya minimal:

```text
5
```

Karena kita membuat lima Category utama.

Untuk memeriksa jumlah Product:

```php
Product::count();
```

Jika database sebelumnya kosong, hasilnya seharusnya:

```text
30
```

Untuk melihat beberapa Product beserta Category-nya:

```php
Product::with('category')->take(5)->get();
```

Yang perlu diperiksa:

- Product memiliki `name`, `price`, `stock`, `slug`, dan `is_active`.
- `category_id` tidak kosong.
- Relasi `category` menampilkan salah satu dari Elektronik, Pakaian, Makanan, Buku, atau Aksesoris.
- `image` dapat bernilai `null`, karena factory tidak mengunggah file fisik.

Keluar dari Tinker dengan:

```text
exit
```

## Periksa dari halaman aplikasi

Setelah data dummy tersedia, buka fitur yang sudah dibuat pada materi sebelumnya:

| Halaman atau fitur | Hal yang dapat dicek |
| --- | --- |
| `/products` | Daftar memiliki banyak Product dummy |
| Pencarian Product | Cari salah satu kata dari nama Product dummy |
| Pagination | Dengan 30 Product, halaman kedua dan seterusnya muncul jika paginator menampilkan 10 item per halaman |
| Sorting | Bandingkan urutan harga atau stok |
| Filter Category | Pilih Elektronik, Pakaian, Makanan, Buku, atau Aksesoris |
| Detail Product | Pastikan URL memakai slug yang dibuat factory |
| Dashboard admin | Angka Product dan daftar Product terbaru tidak lagi kosong |
| Status Product | Coba tampilan aktif dan nonaktif dari nilai `is_active` |

Seeder tidak membuka browser dan tidak memunculkan flash message. Flash message tetap hanya muncul saat user menyimpan atau mengubah Product melalui form CRUD.

## Penting: menjalankan `db:seed` lagi

`CategorySeeder` aman dijalankan lagi karena memakai `firstOrCreate()`. Lima Category utama tidak diduplikasi.

Tetapi `ProductSeeder` saat ini selalu membuat 30 Product baru setiap kali dijalankan:

```php
Product::factory()->count(30)->create();
```

Jadi jika kamu menjalankan:

```bash
php artisan db:seed
```

dua kali pada database yang sebelumnya kosong, jumlah Product menjadi 60.

Ini bukan error, karena tujuan ProductSeeder saat ini adalah membuat banyak data dummy baru. Namun ingat hal ini saat menguji pagination atau dashboard.

Jika kamu ingin kembali ke data awal yang bersih pada database development, Laravel menyediakan:

```bash
php artisan migrate:fresh --seed
```

Perintah ini akan:

1. Menghapus seluruh tabel pada database yang sedang dipakai.
2. Menjalankan semua migration dari awal.
3. Menjalankan `DatabaseSeeder` lagi.

> **Peringatan keras:** `php artisan migrate:fresh --seed` menghapus seluruh data pada database tujuan. Gunakan hanya pada database local atau development yang memang boleh dihapus. Jangan jalankan pada production atau database berisi data penting.

Untuk tahap belajar sehari-hari, gunakan `php artisan db:seed` jika memang ingin menambah data dummy. Jangan menjalankan `migrate:fresh --seed` hanya karena ingin melihat data baru.

## Jika muncul error

Berikut pemeriksaan paling dasar:

| Gejala | Periksa |
| --- | --- |
| `Class ...Seeder not found` | Nama file, nama class, dan namespace `Database\Seeders` harus cocok. |
| Error `random()` pada Category | Pastikan `CategorySeeder` terdaftar sebelum `ProductSeeder`. |
| Error kolom tidak ditemukan | Pastikan migration Product sudah memuat kolom yang digunakan factory, lalu migration sudah dijalankan. |
| Error duplicate slug | Pastikan `ProductFactory` membuat `$name` unik dan slug dibuat dari `$name`. |
| Halaman daftar belum berubah | Pastikan aplikasi memakai database yang sama dengan terminal, lalu periksa data melalui Tinker. |

Jangan menambahkan `try/catch` untuk menutupi error seeding. Lebih baik baca pesan error, lalu perbaiki nama class, urutan seeder, migration, atau factory yang terkait.

## Cek pemahaman

1. Pastikan `DatabaseSeeder` memanggil `CategorySeeder` sebelum `ProductSeeder`.
2. Jalankan `php artisan db:seed` pada database local atau development.
3. Pastikan ada lima Category utama dan 30 Product dummy jika database sebelumnya kosong.
4. Pastikan beberapa Product mempunyai Category melalui `Product::with('category')->take(5)->get()` di Tinker.
5. Buka daftar Product untuk menguji search, pagination, sorting, relasi, status, dan dashboard.
6. Ingat bahwa seeding ulang menambah 30 Product lagi.
7. Jangan gunakan `php artisan migrate:fresh --seed` pada database penting.

Tahap berikutnya adalah ringkasan dan best practice Seeder Produk Dummy.

---

**Apakah kamu ingin lanjut ke langkah terakhir: ringkasan dan best practice seeder produk dummy?**
