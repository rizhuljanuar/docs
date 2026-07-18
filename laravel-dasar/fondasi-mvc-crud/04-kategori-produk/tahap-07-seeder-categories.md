# Tahap 7: Mengisi Data Awal Kategori dengan Seeder

## Apa itu Seeder?

### Analogi Sederhana: Perlengkapan Hotel

Bayangkan kamu masuk ke kamar hotel baru. Kamu tidak perlu minta satu-satu:

- Handuk sudah tersedia di kamar mandi.
- Sabun dan shampo sudah di wastafel.
- Botol air mineral sudah di meja.

Semua itu sudah **disiapkan dari awal**, supaya kamu tinggal pakai.

Di Laravel, **seeder** = **penyiap data awal otomatis**.

Seeder adalah file PHP yang berisi:

> "Kalau database masih kosong, isi dengan data berikut..."

Supaya tiap kali kita reset database, data contoh otomatis terisi dan siap dipakai untuk testing atau belajar.

## Kenapa Pakai Seeder?

Tanpa seeder, setiap reset database kamu harus:

1. Buka phpMyAdmin.
2. Ketik manual: INSERT INTO categories (name) VALUES ('Elektronik');
3. Ulangi untuk Pakaian, Makanan, Buku.
4. Capek dan rawan typo.

Dengan seeder, cukup satu perintah:

```bash
php artisan db:seed
```

Selesai. Semua data contoh otomatis terisi.

## Kapan Seeder Berguna?

| Situasi                                   | Pakai Seeder? |
|-------------------------------------------|---------------|
| Project baru, mau coba dengan data contoh | Ya             |
| Data master (kategori, role, negara)       | Ya             |
| Data transaksi (pesanan, pembayaran)      | Tidak (itu data user) |
| Data sementara untuk testing              | Ya (bisa pakai factory) |

Di tahap ini, kita isi data master: **daftar kategori contoh**.

## Langkah 1: Buat Seeder

Di terminal:

```bash
php artisan make:seeder CategorySeeder
```

Penjelasan:

| Bagian            | Arti                                                  |
|-------------------|-------------------------------------------------------|
| `make:seeder`     | Perintah buat file seeder baru                        |
| `CategorySeeder`  | Nama seeder (PascalCase + suffix `Seeder`)            |

File yang dibuat:

```
database/seeders/CategorySeeder.php
```

## Langkah 2: Isi Default Seeder

Buka file tersebut. Isi default:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class CategorySeeder extends Seeder
{
    public function run(): void
    {
        //
    }
}
```

Penjelasan:

| Bagian                          | Arti                                                          |
|---------------------------------|---------------------------------------------------------------|
| `class CategorySeeder`          | Nama kelas seeder                                             |
| `extends Seeder`                | Mewarisi kemampuan seeder Laravel                             |
| `public function run()`         | Method yang dijalankan saat seeding                           |

## Langkah 3: Tambah Kode Isi Data

Ubah method `run()` menjadi:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Category;

class CategorySeeder extends Seeder
{
    public function run(): void
    {
        $categories = [
            'Elektronik',
            'Pakaian',
            'Makanan',
            'Buku',
        ];

        foreach ($categories as $name) {
            Category::create(['name' => $name]);
        }
    }
}
```

### Penjelasan Bagian Per Bagian

#### `use App\Models\Category;`

Impor model Category supaya bisa dipanggil di file ini.

#### Array `$categories`

```php
$categories = [
    'Elektronik',
    'Pakaian',
    'Makanan',
    'Buku',
];
```

Daftar nama kategori yang ingin kita isi. Tinggal tambah atau hapus sesuai kebutuhan.

#### `foreach ($categories as $name)`

```php
foreach ($categories as $name) {
    Category::create(['name' => $name]);
}
```

Looping tiap nama kategori, lalu buat baris baru di tabel `categories`.

`Category::create([...])` ini pakai mass assignment, jadi kolom `name` harus sudah ada di `$fillable` model Category (sudah kita atur di tahap 6).

## Langkah 4: Daftarkan Seeder di `DatabaseSeeder`

Agar seeder ikut dijalankan saat `php artisan db:seed`, daftarkan dulu di file utama seeder.

Buka file:

```
database/seeders/DatabaseSeeder.php
```

Cari method `run()` dan ubah menjadi:

```php
public function run(): void
{
    $this->call([
        CategorySeeder::class,
    ]);
}
```

Penjelasan:

| Bagian                        | Arti                                                          |
|-------------------------------|---------------------------------------------------------------|
| `$this->call([...])`          | Panggil seeder lain dari sini                                 |
| `CategorySeeder::class`       | Daftar seeder yang mau dijalankan                             |

Kamu bisa daftarkan banyak seeder di array itu. Laravel akan menjalankan berurutan.

## Langkah 5: Jalankan Seeder

Ada beberapa cara.

### Cara A: Jalankan Semua Seeder yang Terdaftar

```bash
php artisan db:seed
```

Ini menjalankan semua seeder di `DatabaseSeeder`.

### Cara B: Jalankan Seeder Tertentu

```bash
php artisan db:seed --class=CategorySeeder
```

Hanya menjalankan `CategorySeeder`. Berguna kalau mau coba ulang satu seeder saja.

### Cara C: Reset Database + Migration + Seeder Sekaligus

```bash
php artisan migrate:fresh --seed
```

Perintah ini akan:

1. Hapus semua tabel (`migrate:fresh`).
2. Jalankan semua migration ulang.
3. Jalankan semua seeder.

**Peringatan:** Ini menghapus semua data. Cocok di tahap development, **jangan dipakai di production**.

## Output yang Muncul

Saat menjalankan `php artisan db:seed`:

```
   INFO  Seeding database.  

  Database\Seeders\CategorySeeder ............................................................................. DONE
```

## Verifikasi Data Sudah Terisi

Cek lewat phpMyAdmin atau Tinker.

### Lewat Tinker

```bash
php artisan tinker
```

```php
>>> Category::all();
=> Illuminate\Database\Eloquent\Collection {
     items: [
       App\Models\Category { id: 1, name: "Elektronik" },
       App\Models\Category { id: 2, name: "Pakaian" },
       App\Models\Category { id: 3, name: "Makanan" },
       App\Models\Category { id: 4, name: "Buku" },
     ],
   }
```

Empat kategori sudah terisi, siap dipakai.

## Tambahan: Seeder untuk Produk dengan Kategori

Sekarang kategori sudah ada. Kita bisa buat seeder untuk produk contoh yang langsung punya `category_id`.

Buat seeder baru:

```bash
php artisan make:seeder ProductSeeder
```

Isi:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Product;
use App\Models\Category;

class ProductSeeder extends Seeder
{
    public function run(): void
    {
        $elektronik = Category::where('name', 'Elektronik')->first();
        $pakaian    = Category::where('name', 'Pakaian')->first();
        $makanan    = Category::where('name', 'Makanan')->first();
        $buku       = Category::where('name', 'Buku')->first();

        Product::create([
            'name'        => 'Laptop',
            'price'       => 10000000,
            'description' => 'Laptop murah untuk belajar',
            'category_id' => $elektronik->id,
        ]);

        Product::create([
            'name'        => 'Kaos',
            'price'       => 50000,
            'description' => 'Kaos katun nyaman',
            'category_id' => $pakaian->id,
        ]);

        Product::create([
            'name'        => 'Roti Tawar',
            'price'       => 15000,
            'description' => 'Roti segar',
            'category_id' => $makanan->id,
        ]);

        Product::create([
            'name'        => 'Novel',
            'price'       => 75000,
            'description' => 'Novel best seller',
            'category_id' => $buku->id,
        ]);
    }
}
```

Daftarkan di `DatabaseSeeder.php`:

```php
public function run(): void
{
    $this->call([
        CategorySeeder::class,
        ProductSeeder::class,
    ]);
}
```

Perhatikan urutannya: **CategorySeeder dulu**, baru `ProductSeeder`. Karena produk butuh kategori sudah ada dulu (foreign key).

Jalankan ulang:

```bash
php artisan migrate:fresh --seed
```

## Inti Pelajaran Tahap 7

> Seeder = penyiap data awal otomatis, seperti perlengkapan hotel yang sudah disiapkan dari awal.

Yang sudah kita lakukan:

1. Buat seeder: `php artisan make:seeder CategorySeeder`.
2. Isi data kategori contoh dengan looping `Category::create([...])`.
3. Daftarkan di `DatabaseSeeder.php`.
4. Jalankan dengan `php artisan db:seed` atau `php artisan migrate:fresh --seed`.
5. Tambah seeder produk yang langsung pakai `category_id`.

Sekarang database sudah berisi 4 kategori dan beberapa produk contoh. Tinggal tampilkan ke user lewat form dan list.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke langkah berikutnya: **menampilkan dropdown kategori di form produk**?

Di tahap 9 kita akan:

- Mengubah controller produk untuk mengirim data kategori ke view.
- Menambah elemen `<select>` di form create/edit produk.
- Supaya user bisa pilih kategori saat menambah produk.

(Tahap 8 opsional: CRUD kategori. Bisa skip kalau mau fokus ke relasi produk dulu.)

Ketik **"lanjut"** untuk tahap 9, atau **"tahap 8"** kalau mau belajar CRUD kategori dulu.
