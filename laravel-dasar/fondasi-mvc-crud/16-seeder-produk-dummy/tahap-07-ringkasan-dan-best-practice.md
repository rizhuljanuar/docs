# Tahap 7 — Ringkasan dan Best Practice Seeder Produk Dummy

> Penutup materi 16: menyiapkan data Category dan Product dummy agar fitur CRUD Laravel dapat diuji dengan lebih mudah.

## Apa yang sudah dibuat?

Materi ini dimulai dari masalah database kosong. Jika belum ada data, sulit menguji daftar Product, search, pagination, sorting, dashboard, dan relasi Category.

Sekarang kita sudah memahami alur lengkap data dummy:

```text
CategoryFactory dan ProductFactory
        ↓ membuat pola data dummy
CategorySeeder dan ProductSeeder
        ↓ mengatur pengisian data
DatabaseSeeder
        ↓ menjalankan seeder dengan urutan yang benar
php artisan db:seed
        ↓ memasukkan data ke database local atau development
Fitur CRUD Product
        ↓ dapat diuji dengan banyak data contoh
```

## Ringkasan file yang dipakai

| File | Tugas |
| --- | --- |
| `app/Models/Category.php` | Memakai `HasFactory`, memiliki field `name`, serta relasi `products()`. |
| `app/Models/Product.php` | Memakai `HasFactory`, `SoftDeletes`, field Product, dan relasi `category()`. |
| `database/factories/CategoryFactory.php` | Cetakan Category dummy tambahan. |
| `database/factories/ProductFactory.php` | Cetakan nama, harga, stok, deskripsi, slug, gambar, status, dan Category Product dummy. |
| `database/seeders/CategorySeeder.php` | Memastikan lima Category utama tersedia. |
| `database/seeders/ProductSeeder.php` | Membuat 30 Product dummy dari ProductFactory. |
| `database/seeders/DatabaseSeeder.php` | Memanggil CategorySeeder lebih dulu, lalu ProductSeeder. |

## Kode inti yang perlu diingat

### Factory Product

`ProductFactory` menentukan bentuk dasar satu Product dummy:

```php
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
```

Factory membuat data yang bervariasi, tetapi tetap mengikuti kebutuhan Product yang sudah dibuat pada materi sebelumnya.

### Seeder Category utama

`CategorySeeder` menyiapkan Category yang mudah dikenali saat menguji filter dan relasi:

```php
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
```

`firstOrCreate()` berarti Category tidak dibuat ganda jika seeder Category dijalankan lagi.

### Seeder Product dummy

`ProductSeeder` memakai ProductFactory untuk menghasilkan banyak Product dan memilih Category utama yang sudah ada:

```php
$categories = Category::query()->get();

Product::factory()
    ->count(30)
    ->state(fn () => [
        'category_id' => $categories->random()->id,
    ])
    ->create();
```

### Urutan di `DatabaseSeeder`

```php
$this->call([
    CategorySeeder::class,
    ProductSeeder::class,
]);
```

Urutan ini tidak boleh dibalik, karena Product membutuhkan Category yang telah tersimpan.

## Perbedaan factory dan seeder, sekali lagi

| Factory | Seeder |
| --- | --- |
| Cetakan data untuk satu model | Pengatur proses pengisian database |
| Menentukan bentuk nama, harga, stok, slug, dan status Product | Menentukan Category utama, jumlah Product, dan urutan kerja |
| Dipakai melalui `Product::factory()` | Dijalankan dari `DatabaseSeeder` atau perintah Artisan |
| Tidak langsung berjalan ketika `db:seed` dipanggil | Dijalankan saat didaftarkan dan dipanggil oleh `db:seed` |

Kalimat singkatnya:

> **Factory membuat bentuk data dummy, seeder memasukkan data dummy ke database.**

## Hubungan dengan fitur CRUD Product sebelumnya

Data dummy membuat fitur berikut lebih mudah diuji:

| Fitur yang sudah ada | Cara mengujinya dengan data dummy |
| --- | --- |
| Daftar Product | Pastikan 30 Product tampil dan tampilan list tetap rapi. |
| Search Product | Cari salah satu kata dari nama Product hasil factory. |
| Pagination | Dengan 30 Product dan 10 item per halaman, pastikan ada halaman 1, 2, dan 3. |
| Sorting | Urutkan harga atau stok, lalu pastikan urutan berubah. |
| Filter Category | Pilih Elektronik, Pakaian, Makanan, Buku, atau Aksesoris. |
| Detail Product | Buka detail melalui slug, lalu pastikan Product yang tepat tampil. |
| Status aktif | Periksa Product aktif dan nonaktif dari `is_active`. |
| Soft delete | Product dummy baru tidak berada di trash karena `deleted_at` kosong. |
| Dashboard admin | Periksa jumlah Product dan daftar Product terbaru. |
| Form CRUD | Buat atau ubah Product manual, lalu pastikan validasi dan flash message tetap berfungsi. |

Seeder hanya menambah data. Ia tidak mengubah route, controller CRUD, Blade, validasi, component error, flash message, cache dashboard, atau query fitur yang sudah dibuat.

## Best practice

1. **Gunakan data dummy hanya untuk local, development, atau testing.**
   Jangan mengisi database production dengan data latihan.

2. **Pisahkan factory dan seeder berdasarkan tugas.**
   Factory untuk pola data, seeder untuk urutan dan jumlah data. Jangan menulis 30 `Product::create()` manual jika satu factory sudah cukup.

3. **Buat data master jelas dan dapat dibaca.**
   Category utama seperti Elektronik, Pakaian, Makanan, Buku, dan Aksesoris lebih baik ditulis jelas di `CategorySeeder`.

4. **Jalankan seeder sesuai urutan relasi.**
   Parent lebih dulu, kemudian child. Pada contoh ini: Category dulu, kemudian Product.

5. **Gunakan `firstOrCreate()` untuk data tetap yang tidak boleh berulang.**
   Ini cocok untuk Category utama. Jangan mengandalkan factory acak untuk data master yang harus mudah dikenali.

6. **Pastikan data factory sesuai migration dan model.**
   Jangan menambahkan kolom yang tidak ada di tabel Product. Perhatikan `name`, `price`, `stock`, `description`, `category_id`, `slug`, `image`, dan `is_active`.

7. **Buat slug dari nama Product.**
   Gunakan `Str::slug($name)` agar URL detail sesuai Product yang dibuat.

8. **Jangan membuat path gambar palsu.**
   Factory menggunakan `image => null` karena tidak mengunggah file fisik. Blade perlu tetap memeriksa `$product->image` sebelum menampilkan gambar.

9. **Bedakan `is_active` dan soft delete.**
   `is_active` adalah status Product, sedangkan `deleted_at` menandakan Product berada di trash.

10. **Pahami efek menjalankan seeder berulang kali.**
    `CategorySeeder` aman karena memakai `firstOrCreate()`. `ProductSeeder` saat ini menambah 30 Product baru setiap kali dijalankan.

11. **Gunakan `php artisan migrate:fresh --seed` dengan sangat hati-hati.**
    Perintah ini menghapus seluruh tabel pada database yang sedang aktif, lalu menjalankan migration dan seeder dari awal. Gunakan hanya pada database local atau development yang aman dihapus.

12. **Periksa hasil seeding.**
    Gunakan Tinker, database viewer, dan halaman aplikasi. Jangan hanya menganggap seeder benar karena command selesai tanpa error.

## Perintah penting

| Perintah | Fungsi | Dampak data |
| --- | --- | --- |
| `php artisan make:factory ProductFactory --model=Product` | Membuat file factory Product | Tidak mengubah database |
| `php artisan make:seeder ProductSeeder` | Membuat file seeder Product | Tidak mengubah database |
| `php artisan db:seed` | Menjalankan `DatabaseSeeder` | Menambah atau memperbarui data sesuai seeder |
| `php artisan db:seed --class=CategorySeeder` | Menjalankan CategorySeeder saja | Menjamin Category utama tersedia |
| `php artisan tinker` | Membuka tempat mencoba query Laravel | Tidak mengubah data jika hanya membaca query |
| `php artisan migrate:fresh --seed` | Menghapus tabel, migration ulang, lalu seeding | Menghapus semua data pada database aktif |

## Checklist akhir

- [ ] Model Category dan Product memakai trait `HasFactory`.
- [ ] `CategoryFactory` dan `ProductFactory` ada di `database/factories`.
- [ ] ProductFactory membuat `name`, `price`, `stock`, `description`, `category_id`, `slug`, `image`, dan `is_active`.
- [ ] CategorySeeder membuat Elektronik, Pakaian, Makanan, Buku, dan Aksesoris menggunakan `firstOrCreate()`.
- [ ] ProductSeeder membuat 30 Product dummy dan memilih Category yang valid.
- [ ] DatabaseSeeder memanggil CategorySeeder sebelum ProductSeeder.
- [ ] `php artisan db:seed` selesai tanpa error pada database local atau development.
- [ ] Product dummy dapat dilihat di daftar Product, search, pagination, sorting, filter Category, detail slug, status, dan dashboard.
- [ ] View tidak mencoba menampilkan gambar jika `image` bernilai `null`.
- [ ] Seeder ulang dipahami dapat menambah 30 Product lagi.
- [ ] `migrate:fresh --seed` tidak digunakan pada database penting.

## Uji manual terakhir

1. Periksa `.env` dan pastikan database yang dipakai adalah database local atau development.
2. Jalankan `php artisan db:seed` sekali.
3. Pastikan ada lima Category utama dan 30 Product jika database sebelumnya kosong.
4. Buka `/products`, lalu coba search, pagination, sorting, dan filter Category.
5. Buka detail salah satu Product dengan URL slug.
6. Buka dashboard dan pastikan ringkasan tidak lagi nol.
7. Coba satu Product aktif dan satu nonaktif.
8. Coba create atau edit Product dari form. Pastikan materi 14 dan 15 tetap bekerja, yaitu validation error muncul dekat field dan flash success muncul setelah simpan berhasil.
9. Jalankan `php artisan db:seed` sekali lagi hanya jika memang ingin menambah data, lalu periksa jumlah Product bertambah 30.
10. Gunakan `php artisan migrate:fresh --seed` hanya bila kamu benar-benar ingin menghapus seluruh data development dan memulai dari awal.

## Penutup

Seeder dan factory membantu developer menyiapkan data contoh tanpa mengetik banyak Product satu per satu.

Dengan lima Category utama dan 30 Product dummy, aplikasi CRUD Product menjadi lebih mudah diuji. Search, pagination, sorting, relasi Category, dashboard, slug, status aktif, serta form CRUD sekarang memiliki data nyata untuk dicoba.

Materi **16. Seeder Produk Dummy** selesai.
