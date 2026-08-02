# Tahap 3 — Membuat `CategoryFactory` untuk Data Kategori Dummy

> Fokus: membuat cetakan data Category dummy dan menghubungkannya ke model `Category`.

Pada tahap 2, kita sudah membedakan tugas factory dan seeder:

- **Factory** menentukan bentuk satu data dummy.
- **Seeder** mengatur kapan dan berapa banyak data disimpan.

Sekarang kita mulai dari data yang paling sederhana, yaitu **Category**. Category hanya memiliki satu data utama, yaitu `name`, sehingga cocok untuk latihan factory pertama.

## Sebelum membuat factory, periksa model Category

Agar Laravel dapat memakai perintah seperti ini nanti:

```php
Category::factory()
```

model `Category` perlu memakai trait `HasFactory`.

Buka file:

```text
app/Models/Category.php
```

Model Category sebelumnya sudah memiliki `$fillable` dan relasi `products()`. Tambahkan import serta trait `HasFactory`, tanpa menghapus kode yang sudah ada:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Category extends Model
{
    use HasFactory;

    protected $fillable = ['name'];

    public function products(): HasMany
    {
        return $this->hasMany(Product::class);
    }
}
```

Penjelasan bagian yang baru:

| Kode | Fungsi sederhana |
| --- | --- |
| `use Illuminate\Database\Eloquent\Factories\HasFactory;` | Memanggil kemampuan factory dari Laravel. |
| `use HasFactory;` | Memberi model Category method `Category::factory()`. |

`$fillable` tetap diperlukan untuk form CRUD yang sudah dibuat. Relasi `products()` juga tetap diperlukan agar Category dapat memiliki banyak Product. Menambah `HasFactory` tidak mengubah keduanya.

> Jika model Category kamu sudah memakai `HasFactory`, jangan menambahkannya dua kali.

## Buat file `CategoryFactory`

Di terminal, jalankan perintah berikut:

```bash
php artisan make:factory CategoryFactory --model=Category
```

Penjelasan:

| Bagian | Arti sederhana |
| --- | --- |
| `php artisan` | Menjalankan alat bantu perintah milik Laravel. |
| `make:factory` | Meminta Laravel membuat file factory baru. |
| `CategoryFactory` | Nama cetakan yang ingin dibuat. |
| `--model=Category` | Memberi tahu bahwa factory ini dipakai untuk model `Category`. |

Laravel membuat file:

```text
database/factories/CategoryFactory.php
```

Nama file mengikuti aturan Laravel:

```text
Model Category → CategoryFactory
Model Product  → ProductFactory
```

Karena nama dan folder mengikuti aturan ini, Laravel 13+ dapat menemukan factory ketika kita menulis `Category::factory()`.

## Isi factory dengan nama Category contoh

Buka `database/factories/CategoryFactory.php`, lalu isi method `definition()` seperti ini:

```php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

class CategoryFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->unique()->words(2, true),
        ];
    }
}
```

## Membaca kode pelan-pelan

| Kode | Arti sederhana |
| --- | --- |
| `namespace Database\Factories;` | File ini berada pada kelompok factory database. |
| `extends Factory` | Class ini mendapat kemampuan membuat data dummy dari Laravel. |
| `definition()` | Tempat kita menentukan bentuk dasar satu Category dummy. |
| `return [...]` | Mengembalikan data yang akan dipakai saat Category dibuat. |
| `'name' => ...` | Mengisi kolom `name` pada tabel `categories`. |
| `fake()` | Memanggil pembuat data contoh dari Laravel. |
| `unique()` | Mengusahakan nama yang dibuat tidak sama dalam satu proses factory. |
| `words(2, true)` | Membuat dua kata dan menggabungkannya menjadi satu teks. |

Contoh nama yang dapat dibuat factory:

```text
Silver Forest
Urban Market
Bright Corner
```

Nama ini hanya contoh acak. Ingat, lima kategori utama **Elektronik, Pakaian, Makanan, Buku, dan Aksesoris** tetap akan dibuat secara jelas oleh `CategorySeeder` pada tahap berikutnya. `CategoryFactory` berguna ketika kita ingin Category dummy tambahan atau ingin membuat data Category pada testing.

## Mengapa memakai `unique()`?

Pada materi CRUD Category sebelumnya, nama kategori dapat dibuat unik agar tidak ada dua kategori dengan nama yang sama.

Karena itu factory juga sebaiknya mencoba membuat nama berbeda:

```php
fake()->unique()->words(2, true)
```

Jika factory membuat 10 Category, hasilnya lebih aman untuk diuji karena nama cenderung tidak berulang.

Namun `unique()` bukan pengganti aturan database. Aturan unik di migration atau validasi tetap menjadi penjaga data utama. Factory hanya membantu menyiapkan data contoh yang masuk akal.

## Factory belum memasukkan data ke database

Setelah menulis `CategoryFactory`, database belum berubah. Factory baru saja menjadi **cetakan**.

Nanti ada dua cara umum untuk memakai cetakan ini:

```php
Category::factory()->make();
```

Artinya: buat object Category contoh di memori, tetapi **jangan simpan** ke database.

```php
Category::factory()->create();
```

Artinya: buat satu Category contoh dan **simpan** ke database.

Kita belum perlu menjalankan keduanya. Pada tahap ini, cukup pahami perbedaannya:

| Method | Membuat object? | Menyimpan ke database? |
| --- | --- | --- |
| `make()` | Ya | Tidak |
| `create()` | Ya | Ya |

Nanti seeder akan memakai `create()` ketika waktunya mengisi database dummy.

## Hubungannya dengan Product

CategoryFactory membuat data untuk tabel `categories`. ProductFactory, yang dibuat pada tahap berikutnya, akan membutuhkan Category karena setiap Product memiliki `category_id`.

Alurnya tetap seperti ini:

```text
CategoryFactory membuat bentuk Category
        ↓
CategorySeeder memastikan kategori utama tersedia
        ↓
ProductFactory membuat Product yang terhubung ke Category
        ↓
ProductSeeder menyimpan banyak Product dummy
```

Kita sengaja mulai dari Category supaya saat Product dummy dibuat, ia sudah punya tempat kategori yang valid.

## Cek pemahaman

1. Pastikan `Category` memakai `HasFactory` satu kali.
2. Jalankan `php artisan make:factory CategoryFactory --model=Category`.
3. Pastikan file `database/factories/CategoryFactory.php` dibuat.
4. Pastikan `definition()` hanya mengisi kolom yang benar-benar ada, yaitu `name`.
5. Ingat bahwa membuat factory tidak otomatis menambah data ke database.

Pada tahap berikutnya, kita akan membuat `ProductFactory`. Factory itu akan lebih lengkap karena harus membuat nama, harga, stok, deskripsi, slug, gambar, status aktif, dan hubungan ke Category.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: membuat `ProductFactory` untuk data produk dummy?**
