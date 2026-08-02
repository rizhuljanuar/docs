# Tahap 4 — Membuat `ProductFactory` untuk Data Product Dummy

> Fokus: membuat cetakan Product dummy yang memiliki nama, harga, stok, deskripsi, slug, gambar, status aktif, dan Category.

Pada tahap 3, kita sudah membuat cetakan Category. Sekarang kita membuat cetakan yang lebih lengkap: `ProductFactory`.

Product membutuhkan lebih banyak data daripada Category. Selain itu, Product harus terhubung ke Category melalui `category_id`.

## Periksa model Product terlebih dahulu

Sama seperti Category, model Product harus memakai trait `HasFactory` agar kita dapat memanggil:

```php
Product::factory()
```

Pada materi sebelumnya, model Product sudah memakai `HasFactory` bersama `SoftDeletes`. Pastikan bagian awal model tetap seperti ini:

```php
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;

class Product extends Model
{
    use HasFactory, SoftDeletes;

    // $fillable, casts(), dan relasi category() tetap ada.
}
```

Kita tidak perlu mengubah `$fillable`, `casts()`, `category()`, `scopeActive()`, atau SoftDeletes untuk membuat factory. Factory hanya memakai kolom yang sudah tersedia pada tabel `products`.

## Buat file `ProductFactory`

Di terminal, jalankan:

```bash
php artisan make:factory ProductFactory --model=Product
```

Laravel akan membuat file:

```text
database/factories/ProductFactory.php
```

| Bagian perintah | Arti sederhana |
| --- | --- |
| `make:factory` | Buat cetakan data dummy. |
| `ProductFactory` | Nama factory untuk model Product. |
| `--model=Product` | Hubungkan factory dengan model Product. |

## Isi `ProductFactory`

Buka file `database/factories/ProductFactory.php`. Isi dengan kode berikut:

```php
<?php

namespace Database\Factories;

use App\Models\Category;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class ProductFactory extends Factory
{
    public function definition(): array
    {
        $name = fake()->unique()->words(3, true);

        return [
            'name' => $name,
            'price' => fake()->numberBetween(10_000, 5_000_000),
            'stock' => fake()->numberBetween(0, 100),
            'description' => fake()->paragraph(),
            'category_id' => Category::factory(),
            'slug' => Str::slug($name),
            'image' => null,
            'is_active' => fake()->boolean(80),
        ];
    }
}
```

## Pahami satu per satu

### Membuat nama dan slug yang cocok

```php
$name = fake()->unique()->words(3, true);
```

Baris ini membuat nama Product contoh, misalnya:

```text
Wireless Sound Speaker
```

Nilai itu disimpan dulu dalam variabel `$name`, supaya nama dan slug memakai sumber yang sama.

```php
'name' => $name,
'slug' => Str::slug($name),
```

Jika nama adalah `Wireless Sound Speaker`, `Str::slug($name)` menghasilkan:

```text
wireless-sound-speaker
```

Ini sesuai dengan materi detail Product sebelumnya, yaitu URL detail memakai slug, bukan ID.

`unique()` membantu nama dummy tidak sama dalam satu proses pembuatan factory. Karena slug dibuat dari nama, slug juga ikut berbeda. Tetap pertahankan unique index slug yang sudah dibuat di database, karena factory bukan pengganti aturan database.

### Harga dan stok

```php
'price' => fake()->numberBetween(10_000, 5_000_000),
'stock' => fake()->numberBetween(0, 100),
```

| Kode | Contoh hasil | Arti |
| --- | --- | --- |
| `numberBetween(10_000, 5_000_000)` | `250000` | Harga dummy antara Rp10.000 dan Rp5.000.000. |
| `numberBetween(0, 100)` | `15` | Stok dummy dari 0 sampai 100. |

Tanda garis bawah pada `10_000` hanya membantu manusia membaca angka. PHP tetap membaca nilainya sebagai `10000`.

Harga tidak dibuat negatif dan stok tidak dibuat kurang dari nol. Ini mengikuti aturan validasi Product dari materi sebelumnya.

### Deskripsi

```php
'description' => fake()->paragraph(),
```

`fake()->paragraph()` membuat teks contoh untuk deskripsi Product. Teksnya bukan deskripsi product nyata, tetapi cukup untuk menguji tampilan daftar, detail, dan pencarian.

### Hubungan ke Category

```php
'category_id' => Category::factory(),
```

Ingat bahwa `Product` memiliki `category_id`. Baris ini memberitahu Laravel:

> Jika belum ada Category yang diberikan, buat satu Category dummy lalu gunakan ID-nya untuk Product ini.

Dengan begitu, satu Product dummy tidak pernah menunjuk ke Category yang tidak ada.

Namun, pada seeder nanti kita sudah mempunyai lima kategori utama: Elektronik, Pakaian, Makanan, Buku, dan Aksesoris. Karena itu `ProductSeeder` akan memberikan Category yang sudah ada kepada ProductFactory. Dengan cara tersebut, Product dummy dibagi ke kategori utama, bukan selalu membuat kategori acak baru.

Jadi baris `Category::factory()` adalah **nilai cadangan yang aman** untuk factory. Seeder nanti dapat menggantinya saat menyiapkan data yang lebih terarah.

### Gambar

```php
'image' => null,
```

Kita sengaja membuat nilai gambar kosong. Factory tidak mengunggah file gambar fisik ke folder `storage`.

Ini aman jika kolom `image` pada tabel Product boleh kosong, seperti yang sudah dibuat pada materi upload gambar. Saat menampilkan image di Blade, tetap lakukan pemeriksaan seperti:

```blade
@if ($product->image)
    <img src="{{ asset('storage/' . $product->image) }}" alt="{{ $product->name }}">
@endif
```

Jangan membuat path gambar palsu seperti `products/contoh.jpg` jika file tersebut sebenarnya tidak ada, karena browser akan menampilkan gambar rusak.

### Status aktif

```php
'is_active' => fake()->boolean(80),
```

`boolean(80)` berarti sebagian besar Product dummy, kira-kira 80 persen, dibuat aktif. Sisanya dibuat tidak aktif.

Data seperti ini berguna untuk mencoba fitur status aktif dan nonaktif yang sudah dibuat sebelumnya.

`is_active` bukan soft delete:

- `is_active` menentukan status Product aktif atau nonaktif.
- `deleted_at` menentukan Product berada di tong sampah atau tidak.

Kita tidak mengisi `deleted_at` di factory ini. Product dummy baru tetap muncul dalam query normal, kecuali status aktifnya memang nonaktif dan halaman memakai scope `active()`.

## Peta field factory Product

| Field database | Nilai dari factory | Tujuan |
| --- | --- | --- |
| `name` | Tiga kata dummy unik | Menguji list, search, dan detail |
| `price` | Rp10.000 sampai Rp5.000.000 | Menguji sorting harga |
| `stock` | 0 sampai 100 | Menguji stok dan sorting |
| `description` | Paragraf dummy | Menguji tampilan detail |
| `category_id` | Category factory sebagai cadangan | Menjaga relasi valid |
| `slug` | Dibuat dari nama | Menguji URL detail berbasis slug |
| `image` | `null` | Menghindari path file palsu |
| `is_active` | Mayoritas aktif | Menguji status aktif dan nonaktif |

## Factory belum membuat 30 Product

Menulis factory hanya membuat **cetakan**. Database belum diisi dan tidak ada Product baru sampai kita memanggil factory dengan `create()`.

Contoh penggunaan di masa depan:

```php
Product::factory()->count(30)->create();
```

Jangan menjalankan kode itu sekarang. Pada tahap berikutnya, kita akan membuat seeder yang menyiapkan lima Category utama, lalu meminta ProductFactory membuat banyak Product yang terhubung dengan category tersebut.

## Cek pemahaman

1. Pastikan `Product` tetap memakai `HasFactory` satu kali.
2. Jalankan `php artisan make:factory ProductFactory --model=Product`.
3. Pastikan `ProductFactory` mengisi semua kolom Product yang diperlukan untuk data dummy.
4. Pastikan `slug` dibuat dari `$name`, bukan dibuat acak terpisah.
5. Pastikan `image` memakai `null` jika tidak ada file gambar sungguhan.
6. Ingat bahwa `category_id` harus selalu menunjuk ke Category yang valid.

Pada langkah berikutnya, kita akan membuat `CategorySeeder` dan `ProductSeeder`. Seeder akan memasukkan lima kategori utama terlebih dahulu, kemudian meminta ProductFactory membuat banyak Product dummy.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: membuat `CategorySeeder` dan `ProductSeeder`?**
