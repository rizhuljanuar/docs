# Tahap 5 — Membuat `CategorySeeder` dan `ProductSeeder`

> Fokus: membuat seeder kategori utama terlebih dahulu, lalu seeder yang meminta `ProductFactory` membuat banyak Product dummy.

Pada tahap 4, kita sudah memiliki cetakan `ProductFactory`. Namun cetakan belum mengisi database sendiri. Sekarang kita membuat dua seeder sebagai petugas yang menjalankan proses pengisian data.

Urutannya harus seperti ini:

```text
1. Buat kategori utama lebih dulu.
2. Ambil kategori yang sudah dibuat.
3. Buat banyak Product dummy dan hubungkan masing-masing ke salah satu kategori.
```

Mengapa? Karena Product membutuhkan `category_id`. Product tidak dapat memilih Category yang belum ada.

## Bagian A, buat `CategorySeeder`

### Buat file seeder

Di terminal, jalankan:

```bash
php artisan make:seeder CategorySeeder
```

Laravel membuat file:

```text
database/seeders/CategorySeeder.php
```

Seeder selalu memiliki method `run()`. Method inilah yang akan dijalankan Laravel ketika seeder dipanggil.

### Isi kategori utama

Buka `database/seeders/CategorySeeder.php`, lalu isi seperti ini:

```php
<?php

namespace Database\Seeders;

use App\Models\Category;
use Illuminate\Database\Seeder;

class CategorySeeder extends Seeder
{
    public function run(): void
    {
        $categories = [
            'Elektronik',
            'Pakaian',
            'Makanan',
            'Buku',
            'Aksesoris',
        ];

        foreach ($categories as $name) {
            Category::firstOrCreate(['name' => $name]);
        }
    }
}
```

## Membaca `CategorySeeder`

| Kode | Arti sederhana |
| --- | --- |
| `use App\Models\Category;` | Agar seeder dapat memakai model Category. |
| `$categories = [...]` | Daftar lima Category utama yang ingin dibuat. |
| `foreach (...)` | Ulangi proses untuk setiap nama Category. |
| `Category::firstOrCreate(...)` | Cari Category dengan nama itu. Jika belum ada, buat Category baru. |

Kita memakai `firstOrCreate()` agar menjalankan seeder dua kali tidak membuat `Elektronik` dua kali, `Pakaian` dua kali, dan seterusnya.

Contohnya, untuk `Elektronik`, Laravel melakukan ini:

```text
Apakah Category bernama Elektronik sudah ada?
- Jika belum ada, buat.
- Jika sudah ada, pakai data yang sudah ada.
```

Lima Category ini ditulis langsung di seeder, bukan dibuat acak oleh `CategoryFactory`, karena ini adalah data utama yang ingin kita lihat dan pakai saat menguji filter kategori serta relasi Product.

## Bagian B, buat `ProductSeeder`

### Buat file seeder

Di terminal, jalankan:

```bash
php artisan make:seeder ProductSeeder
```

Laravel membuat file:

```text
database/seeders/ProductSeeder.php
```

### Isi ProductSeeder

Buka file tersebut, lalu isi seperti ini:

```php
<?php

namespace Database\Seeders;

use App\Models\Category;
use App\Models\Product;
use Illuminate\Database\Seeder;

class ProductSeeder extends Seeder
{
    public function run(): void
    {
        $categories = Category::query()->get();

        Product::factory()
            ->count(30)
            ->state(fn () => [
                'category_id' => $categories->random()->id,
            ])
            ->create();
    }
}
```

## Membaca `ProductSeeder` pelan-pelan

### Ambil Category yang sudah ada

```php
$categories = Category::query()->get();
```

Baris ini mengambil semua Category dari tabel `categories`, yaitu Elektronik, Pakaian, Makanan, Buku, dan Aksesoris yang dibuat oleh `CategorySeeder`.

Hasilnya disimpan dalam `$categories`, seperti sekotak pilihan kategori.

### Buat 30 Product dari factory

```php
Product::factory()
    ->count(30)
```

Artinya: gunakan `ProductFactory`, lalu siapkan 30 Product dummy.

### Pilihkan satu Category untuk setiap Product

```php
->state(fn () => [
    'category_id' => $categories->random()->id,
])
```

Artinya: saat tiap Product dibuat, ambil satu Category secara acak dari `$categories`, lalu pakai ID Category itu sebagai `category_id` Product.

Misalnya hasilnya dapat seperti ini:

| Product dummy | Category yang dipilih |
| --- | --- |
| Wireless Sound Speaker | Elektronik |
| Cotton Casual Shirt | Pakaian |
| Organic Snack Package | Makanan |
| Modern Story Collection | Buku |
| Travel Key Holder | Aksesoris |

Pembagian ini acak, jadi jumlah Product pada setiap Category mungkin tidak persis sama. Itu justru cukup baik untuk mencoba filter Category, daftar Product, pagination, dan dashboard dengan data yang lebih bervariasi.

### Simpan ke database

```php
->create();
```

`create()` meminta Laravel menyimpan 30 Product dummy ke tabel `products`.

`ProductFactory` sudah menentukan nama, harga, stok, deskripsi, slug, gambar `null`, dan status aktif. `state(...)` di ProductSeeder hanya mengganti bagian `category_id` agar memakai Category utama yang sudah ada.

## Kenapa tidak langsung memakai `Product::create()` 30 kali?

Kita bisa membuat data seperti ini:

```php
Product::create([
    'name' => 'Contoh Product',
    // dan seterusnya...
]);
```

Tetapi menulisnya 30 kali membuat seeder panjang dan sulit dirawat. `ProductFactory` sudah menjadi cetakan untuk nilai yang berulang, sehingga ProductSeeder cukup mengatur jumlah Product dan Category-nya.

| Tugas | Tempat yang tepat |
| --- | --- |
| Bentuk nama, harga, stok, slug, status Product | `ProductFactory` |
| Lima Category utama tetap | `CategorySeeder` |
| Jumlah Product dummy dan pemilihan Category | `ProductSeeder` |

## Jangan jalankan `ProductSeeder` sendirian dulu

`ProductSeeder` membutuhkan Category. Jika dijalankan saat tabel `categories` masih kosong, `$categories->random()` tidak dapat memilih apa pun.

Karena itu `CategorySeeder` harus dijalankan lebih dulu. Pada tahap berikutnya, kita akan mendaftarkan keduanya pada `DatabaseSeeder` dengan urutan yang benar:

```text
CategorySeeder
lalu
ProductSeeder
```

Setelah itu, satu perintah `php artisan db:seed` dapat menjalankan seluruh proses.

## Hubungan dengan fitur sebelumnya

Data yang dibuat oleh seeder tidak melalui browser. Maka ketika seeder berjalan:

- Tidak ada form create atau edit yang ditampilkan.
- Tidak ada validation error component dari materi 15.
- Tidak ada flash success dari materi 14.
- Tidak ada Product yang di-soft-delete, karena `deleted_at` dibiarkan kosong.
- `is_active` tetap dibuat oleh ProductFactory untuk menguji fitur status.
- `image` bernilai `null`, sehingga Blade harus tetap memeriksa gambar sebelum menampilkan `<img>`.

Setelah data dummy masuk, fitur daftar, search, pagination, sorting, detail slug, relasi Category, status, dan dashboard dapat diuji dengan data yang lebih nyata.

## Cek pemahaman

1. Buat `CategorySeeder` dan `ProductSeeder` dengan Artisan.
2. Pastikan `CategorySeeder` membuat Elektronik, Pakaian, Makanan, Buku, dan Aksesoris dengan `firstOrCreate()`.
3. Pastikan `ProductSeeder` mengambil Category yang telah ada.
4. Pastikan `ProductSeeder` memakai `Product::factory()->count(30)`.
5. Pastikan `category_id` setiap Product memakai ID Category yang valid.
6. Belum perlu menjalankan `php artisan db:seed`. Kita harus mendaftarkan kedua seeder terlebih dahulu pada tahap berikutnya.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: mendaftarkan seeder di `DatabaseSeeder` dan menjalankan `php artisan db:seed`?**
