# Tahap 5: Mengubah Route Detail untuk Menerima Slug

## Tujuan Tahap Ini

Pada Tahap 4, setiap produk sudah memiliki slug yang unik. Sekarang kita akan
memakai slug tersebut untuk mencari produk dari URL.

Perubahannya:

```text
Sebelum: /products/7
Sesudah: /products/kaos-hitam-7
```

Di tahap ini kita hanya mengubah route detail dan method `show()`. Tautan pada
halaman daftar akan diubah pada tahap berikutnya.

## Apa Itu Pencarian Berdasarkan Slug?

Pada materi-materi sebelumnya, route detail memakai parameter `{id}` dan
method `show($id)` mencari produk dengan `Product::findOrFail($id)`.

Sekarang kita ingin route detail menerima **slug** alih-alih ID, lalu mencari
produk berdasarkan kolom `slug`.

## Analogi Sederhana: Petugas Perpustakaan

Bayangkan kamu memberikan kode buku kepada petugas perpustakaan:

```text
belajar-laravel-12
```

Petugas mencari kode itu di katalog, mengambil bukunya, lalu menyerahkan buku
tersebut kepadamu.

Dalam Laravel:

- Slug pada URL adalah kode buku.
- Method `show()` adalah petugas perpustakaan.
- Tabel `products` adalah katalog.
- `$product` adalah produk yang sudah ditemukan.

Controller mencari produk berdasarkan slug, lalu mengirimkannya ke view.

## Langkah 1: Mengubah Route Detail

Buka:

```text
routes/web.php
```

> **Catatan:** Sepanjang **Materi 1 sampai 4**, kita menulis route produk
> **satu per satu** (bukan `Route::resource`). Kita tetap konsisten dengan
> pola itu. Hanya route detail yang diubah.

Cari route detail lama:

```php
Route::get('/products/{id}', [ProductController::class, 'show']);
```

Ganti menjadi:

```php
Route::get('/products/{slug}', [ProductController::class, 'show']);
```

Nama parameter berubah dari `{id}` menjadi `{slug}`. Ini menandakan bahwa
bagian URL tersebut sekarang dianggap sebagai slug.

### Kenapa Tidak Pakai `Route::resource`?

Di **Materi 1 (Tahap 5 dan 6)** kita menulis route produk satu per satu dan
tetap memakai pola itu sampai **Materi 4**. Pola ini lebih eksplisit dan
konsisten dengan apa yang sudah kamu pelajari.

Route lain tetap bekerja seperti sebelumnya:

- Edit tetap memakai ID: `/products/{id}/edit`.
- Update tetap memakai ID: PUT `/products/{id}`.
- Hapus tetap memakai ID: DELETE `/products/{id}`.

Hanya halaman detail yang memakai slug.

### Susunan Route Setelah Perubahan

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ProductController;
use App\Http\Controllers\CategoryController;

// Route produk
Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/create', [ProductController::class, 'create']);
Route::post('/products', [ProductController::class, 'store']);
Route::get('/products/{id}/edit', [ProductController::class, 'edit']);
Route::put('/products/{id}', [ProductController::class, 'update']);
Route::delete('/products/{id}', [ProductController::class, 'destroy']);
Route::get('/products/{slug}', [ProductController::class, 'show']);

// Route kategori (dari Materi 4)
Route::resource('categories', CategoryController::class);
```

> **Urutan route penting!** `/products/create` dan route edit sengaja ditulis
> **sebelum** route detail `/products/{slug}`. Dengan susunan dari yang paling
> spesifik ke yang lebih umum ini, route yang memiliki segmen tetap mudah
> dibaca dan dipelihara; `create` juga tidak akan dianggap sebagai slug.

## Langkah 2: Mengubah Method `show()`

Buka:

```text
app/Http/Controllers/ProductController.php
```

Method `show()` di **Materi 4 (Tahap 10)** masih menerima `$id` dan memakai
`findOrFail($id)` dengan eager loading `with('category')`:

```php
public function show($id)
{
    $product = Product::with('category')->findOrFail($id);
    return view('products.show', compact('product'));
}
```

Ubah menjadi pencarian berdasarkan kolom `slug`:

```php
public function show($slug)
{
    $product = Product::with('category')
        ->where('slug', $slug)
        ->firstOrFail();

    return view('products.show', compact('product'));
}
```

### Penjelasan Perubahan

| Bagian                | Sebelum (Materi 4)             | Sesudah (Materi 5)                     |
|-----------------------|--------------------------------|----------------------------------------|
| Parameter             | `$id`                          | `$slug`                                |
| Pencarian             | `findOrFail($id)`              | `where('slug', $slug)->firstOrFail()`  |
| Eager loading         | `with('category')`             | Tetap `with('category')` (dari Materi 4) |

Kita tetap memakai pola `findOrFail` / `firstOrFail` (bukan route model
binding `Product $product`) supaya konsisten dengan **Materi 1 sampai 4**
yang selalu memakai parameter `$id` dan pencarian manual.

### Kenapa `firstOrFail()`?

`->firstOrFail()` sama dengan `->first()`, tapi jika produk tidak ditemukan,
Laravel otomatis menampilkan halaman **404 Not Found**. Ini setara dengan
`findOrFail($id)` yang sudah kamu pakai sejak **Materi 1 (Tahap 10)**.

## Alur yang Terjadi

Saat pengguna membuka:

```text
/products/kaos-hitam-7
```

Laravel melakukan langkah berikut:

```text
1. Route menerima slug "kaos-hitam-7"
2. Method show($slug) dipanggil dengan $slug = "kaos-hitam-7"
3. Product::where('slug', $slug)->firstOrFail() mencari produk
4. Produk yang ditemukan dikirim ke view products.show
5. Halaman detail ditampilkan
```

## Apa yang Terjadi Jika Slug Tidak Ada?

Jika pengguna membuka:

```text
/products/produk-tidak-ada-999
```

Laravel tidak menemukan produknya (karena `firstOrFail()` gagal) dan
otomatis menampilkan halaman:

```text
404 Not Found
```

Kita tidak perlu membuat pemeriksaan manual.

## Langkah 3: Memeriksa Route

Jalankan:

```bash
php artisan route:list --path=products
```

Cari route detail dengan bentuk:

```text
GET|HEAD  products/{slug}
```

Pastikan hanya ada satu route `GET` untuk detail produk.

## Langkah 4: Menguji URL Slug

Lihat salah satu nilai slug di tabel `products`, misalnya:

```text
kaos-hitam-7
```

Buka di browser:

```text
http://127.0.0.1:8000/products/kaos-hitam-7
```

Hasil yang diharapkan:

- Halaman detail produk Kaos Hitam tampil.
- URL tetap memakai slug.
- Slug yang tidak terdaftar menghasilkan halaman 404.

Tautan **Detail** pada halaman daftar mungkin masih memakai ID. Itu normal
karena tautannya baru akan kita ubah pada Tahap 6.

## Checklist Tahap 5

- [ ] Route detail lama `{id}` sudah diganti menjadi `{slug}`.
- [ ] Method `show()` menerima parameter `$slug`.
- [ ] Method `show()` memakai `where('slug', $slug)->firstOrFail()`.
- [ ] Method `show()` tetap memakai `with('category')` dari Materi 4.
- [ ] URL dengan slug menampilkan produk yang benar.
- [ ] Slug yang tidak ada menghasilkan halaman 404.
- [ ] Route edit, update, dan hapus tetap bekerja dengan ID.

## Inti Tahap 5

> Route detail sekarang menerima `{slug}`, dan method `show()` mencari produk
> dengan `where('slug', $slug)->firstOrFail()`. Pola ini konsisten dengan
> materi sebelumnya yang memakai parameter dan pencarian manual.

Sekarang halaman detail sudah bisa dibuka melalui URL slug. Namun, tautan dari
halaman daftar masih perlu diperbarui.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke **Tahap 6: mengubah tautan detail produk agar
memakai slug**?

Ketik **"lanjut"** jika sudah siap.
