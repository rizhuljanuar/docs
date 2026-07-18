# Tahap 6: Model Category dan Relasi hasMany / belongsTo

## Apa itu Model di Laravel?

### Analogi Sederhana: Perpustakaan

Bayangkan kamu di perpustakaan. Ada dua hal:

- **Rak buku fisik** -> tempat menyimpan buku-buku.
- **Pustakawan** -> orang yang tahu cara mencari, meminjam, dan menambah buku dari rak itu.

Di Laravel:

- **Tabel database** (misal `categories`) -> rak fisik tempat data disimpan.
- **Model** (misal `Category`) -> "pustakawan" yang tahu cara ambil, simpan, ubah, dan hapus data dari tabel itu.

Model adalah **jembatan** antara kode PHP kita dengan tabel di database. Kamu tidak perlu menulis SQL manual. Cukup panggil method di model, Laravel yang urus sisanya.

## Konvensi Nama Model dan Tabel

Laravel pakai konvensi otomatis:

| Model (singular) | Tabel otomatis (plural) |
|------------------|--------------------------|
| `Category`       | `categories`             |
| `Product`        | `products`               |
| `User`           | `users`                  |

Jadi kalau kamu bikin model bernama `Category`, Laravel otomatis anggap model ini mengelola tabel `categories`.

## Langkah 1: Membuat Model Category

Di terminal:

```bash
php artisan make:model Category
```

Penjelasan bagian per bagian:

| Bagian            | Arti                                                            |
|-------------------|-----------------------------------------------------------------|
| `php artisan`     | Memanggil alat bantu Laravel                                    |
| `make:model`      | Perintah untuk membuat file model baru                          |
| `Category`        | Nama model (singular/PascalCase)                                |

File yang dibuat:

```
app/Models/Category.php
```

## Langkah 2: Isi Default Model Category

Buka file `app/Models/Category.php`. Isi defaultnya:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    //
}
```

Penjelasan:

| Bagian                | Arti                                                                      |
|-----------------------|---------------------------------------------------------------------------|
| `namespace App\Models`| Folder tempat model berada                                                |
| `use ...Model`        | Mengimpor kelas dasar Model dari Laravel                                  |
| `class Category extends Model` | Kelas `Category` mewarisi semua kemampuan model Eloquent         |
| `//`                  | Tempat kamu menulis konfigurasi dan relasi                                |

## Langkah 3: Tambah `$fillable` dan Relasi `products()`

Ubah file `app/Models/Category.php` menjadi:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;
use App\Models\Product;

class Category extends Model
{
    protected $fillable = ['name'];

    public function products(): HasMany
    {
        return $this->hasMany(Product::class);
    }
}
```

### Penjelasan `$fillable`

```php
protected $fillable = ['name'];
```

Arti: kolom `name` boleh diisi massal (mass assignment).

Tanpa `$fillable`, kamu tidak bisa pakai cara singkat seperti:

```php
Category::create(['name' => 'Elektronik']);
```

Laravel akan menolak demi keamanan (mass assignment vulnerability).

### Penjelasan `products()` dengan `hasMany`

```php
public function products(): HasMany
{
    return $this->hasMany(Product::class);
}
```

Bagian per bagian:

| Bagian                  | Arti                                                                              |
|-------------------------|-----------------------------------------------------------------------------------|
| `public function products()` | Method bernama `products()` (plural, karena satu kategori punya banyak produk) |
| `: HasMany`             | Tipe kembalian: relasi HasMany                                                    |
| `$this->hasMany(...)`   | "Model ini (Category) punya banyak data di model..."                              |
| `Product::class`        | ...yaitu model `Product`                                                          |

**Artinya**: "Satu kategori bisa memiliki banyak produk."

Cara pakai nanti:

```php
$category = Category::find(1);
$category->products;  // <- otomatis ambil semua produk di kategori ini
```

## Langkah 4: Tambah Relasi `category()` di Model Product

Buka file `app/Models/Product.php` (sudah ada dari materi CRUD sebelumnya). Tambahkan method baru:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use App\Models\Category;

class Product extends Model
{
    protected $fillable = ['name', 'price', 'description', 'category_id'];

    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }
}
```

### Perhatikan: `$fillable` ditambah `category_id`

Baris ini:

```php
protected $fillable = ['name', 'price', 'description', 'category_id'];
```

`category_id` ditambahkan supaya bisa diisi massal saat membuat produk baru.

### Penjelasan `category()` dengan `belongsTo`

```php
public function category(): BelongsTo
{
    return $this->belongsTo(Category::class);
}
```

Bagian per bagian:

| Bagian                   | Arti                                                                          |
|--------------------------|-------------------------------------------------------------------------------|
| `public function category()` | Method bernama `category()` (singular, karena satu produk milik satu kategori)|
| `: BelongsTo`            | Tipe kembalian: relasi BelongsTo                                              |
| `$this->belongsTo(...)`  | "Model ini (Product) milik model..."                                          |
| `Category::class`        | ...yaitu model `Category`                                                     |

**Artinya**: "Satu produk hanya dimiliki oleh satu kategori."

Cara pakai nanti:

```php
$product = Product::find(1);
$product->category;  // <- otomatis ambil data kategori dari produk ini
$product->category->name;  // <- ambil nama kategori, misal "Elektronik"
```

## Cara Mengingat: hasMany vs belongsTo

Kuncinya: **tanya "siapa yang punya banyak?"**

| Pertanyaan                                       | Jawaban              | Relasi         |
|--------------------------------------------------|----------------------|----------------|
| Kategori punya banyak produk?                    | Ya                   | Category: `hasMany` |
| Produk punya banyak kategori?                    | Tidak, hanya satu    | Product: `belongsTo` |
| Produk ini milik siapa?                          | Kategori tertentu    | Product: `belongsTo` |

Analogi keluarga:

- **Ibu** berkata: "Aku **punya banyak** anak." -> `hasMany`
- **Anak** berkata: "Aku **milik** ibu ini." -> `belongsTo`

Sama persis dengan kategori dan produk.

## Cara Konvensi Laravel Bekerja Otomatis

Mengapa kita tidak perlu menyebut kolom `category_id` secara eksplisit?

Karena Laravel menebak berdasarkan **konvensi nama method**:

- Method bernama `category()` di model Product -> Laravel cari kolom `category_id` di tabel products.
- Method bernama `products()` di model Category -> Laravel cari kolom `category_id` di tabel products.

Kalau kamu menyalahi konvensi (misal kolomnya `kat_id`), kamu harus tulis manual:

```php
return $this->belongsTo(Category::class, 'kat_id');
```

Tapi selama ikut konvensi, kode singkat dan otomatis.

## Tes Relasi dengan Tinker

Sekarang mari kita coba relasi yang sudah kita buat memakai Tinker.

### Langkah 1: Buka Tinker

```bash
php artisan tinker
```

### Langkah 2: Isi Data Contoh

Di dalam Tinker (prompt `>`):

```php
>>> $kategori = Category::create(['name' => 'Elektronik']);
>>> $produk1 = Product::create(['name' => 'Laptop', 'price' => 10000000, 'category_id' => $kategori->id]);
>>> $produk2 = Product::create(['name' => 'HP', 'price' => 3000000, 'category_id' => $kategori->id]);
```

### Langkah 3: Coba Relasi `products()` dari Kategori

```php
>>> $kategori->products;
# Hasil: kumpulan produk yang category_id-nya sama dengan id kategori
```

Output (dipotong):

```
=> Illuminate\Database\Eloquent\Collection {
     items: [
       App\Models\Product { name: "Laptop", ... },
       App\Models\Product { name: "HP", ... },
     ],
   }
```

Satu kategori -> banyak produk. Sesuai `hasMany`.

### Langkah 4: Coba Relasi `category()` dari Produk

```php
>>> $produk1->category;
=> App\Models\Category { id: 1, name: "Elektronik" }

>>> $produk1->category->name;
=> "Elektronik"
```

Satu produk -> satu kategori. Sesuai `belongsTo`.

### Langkah 5: Keluar dari Tinker

```php
>>> exit
```

## Inti Pelajaran Tahap 6

> Model = jembatan antara kode PHP dan tabel database. Relasi `hasMany` dan `belongsTo` mengizinkan kita akses data terkait tanpa tulis SQL manual.

Yang sudah kita lakukan:

1. Buat model `Category` dengan `php artisan make:model Category`.
2. Tambah `$fillable = ['name']` di Category.
3. Tambah method `products()` dengan `hasMany` di model Category.
4. Tambah method `category()` dengan `belongsTo` di model Product.
5. Tambah `category_id` ke `$fillable` di model Product.
6. Tes relasi memakai Tinker.

Sekarang struktur database dan model sudah siap. Tinggal isi data dan tampilkan ke user.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke langkah berikutnya: **mengisi data awal kategori memakai Seeder**?

Di tahap 7 kita akan:

- Membuat seeder dengan `php artisan make:seeder CategorySeeder`.
- Mengisi kategori contoh (Elektronik, Pakaian, Makanan, Buku).
- Menjalankan seeder dengan `php artisan db:seed`.
- Supaya tiap kali database di-reset, data contoh otomatis terisi.

Ketik **"lanjut"** kalau siap.
