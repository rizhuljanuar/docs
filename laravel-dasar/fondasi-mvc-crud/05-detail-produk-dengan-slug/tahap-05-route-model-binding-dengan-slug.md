# Tahap 5: Route Model Binding Berdasarkan Slug

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

## Apa Itu Route Model Binding?

**Route model binding** adalah fitur Laravel yang mengambil data dari URL,
mencarinya di database, lalu memberikan objek model langsung kepada
controller.

Tanpa route model binding, kita perlu mencari produk sendiri:

```php
$product = Product::findOrFail($id);
```

Dengan route model binding, Laravel melakukan pencarian tersebut secara
otomatis:

```php
public function show(Product $product)
{
    // $product sudah berisi produk yang ditemukan.
}
```

## Analogi Sederhana: Petugas Perpustakaan

Bayangkan kamu memberikan kode buku kepada petugas perpustakaan:

```text
belajar-laravel-12
```

Petugas mencari kode itu di katalog, mengambil bukunya, lalu menyerahkan buku
tersebut kepadamu.

Dalam Laravel:

- Slug pada URL adalah kode buku.
- Route model binding adalah petugas perpustakaan.
- Tabel `products` adalah katalog.
- `$product` adalah produk yang sudah ditemukan.

Controller tidak perlu mencari ulang produk tersebut.

## Mengenal `{product:slug}`

Perhatikan parameter route berikut:

```php
{product:slug}
```

Artinya:

- `product` adalah nama parameter route.
- `slug` adalah kolom yang dipakai untuk mencari produk.

Jika URL-nya:

```text
/products/kaos-hitam-7
```

Laravel akan menjalankan pencarian seperti:

```text
Cari satu produk dengan slug = kaos-hitam-7
```

## Langkah 1: Mengubah Route Detail

Buka:

```text
routes/web.php
```

Jika project-mu memakai `Route::resource`, ubah:

```php
Route::resource('products', ProductController::class);
```

menjadi:

```php
Route::resource('products', ProductController::class)->except('show');

Route::get('/products/{product:slug}', [ProductController::class, 'show'])
    ->name('products.show');
```

### Kenapa `show` Dikeluarkan?

`Route::resource` biasanya membuat route detail berikut:

```text
/products/{product}
```

Route tersebut mencari produk berdasarkan ID. Kita mengeluarkan `show`, lalu
membuat route detail sendiri dengan `{product:slug}`.

Route lain tetap bekerja seperti sebelumnya:

- Edit tetap memakai ID.
- Update tetap memakai ID.
- Hapus tetap memakai ID.

Hanya halaman detail yang memakai slug.

## Jika Route Ditulis Satu per Satu

Jika project-mu tidak memakai `Route::resource`, cari route detail lama:

```php
Route::get('/products/{id}', [ProductController::class, 'show']);
```

Ganti menjadi:

```php
Route::get('/products/{product:slug}', [ProductController::class, 'show'])
    ->name('products.show');
```

Gunakan salah satu cara sesuai isi project-mu. Jangan mendaftarkan kedua versi
route detail sekaligus.

## Langkah 2: Mengubah Method `show()`

Buka:

```text
app/Http/Controllers/ProductController.php
```

Jika method `show()` masih menerima ID:

```php
public function show($id)
{
    $product = Product::findOrFail($id);

    return view('products.show', compact('product'));
}
```

ubah menjadi:

```php
public function show(Product $product): View
{
    return view('products.show', compact('product'));
}
```

Pastikan import berikut tersedia di bagian atas controller:

```php
use App\Models\Product;
use Illuminate\View\View;
```

Laravel sekarang mengisi `$product` secara otomatis berdasarkan slug dari
URL.

## Kenapa Namanya Harus Sama?

Nama parameter route:

```php
{product:slug}
```

harus cocok dengan nama parameter controller:

```php
Product $product
```

Keduanya menggunakan nama `product`. Type `Product` memberi tahu Laravel bahwa
data harus dicari melalui model `Product`.

## Alur yang Terjadi

Saat pengguna membuka:

```text
/products/kaos-hitam-7
```

Laravel melakukan langkah berikut:

```text
1. Route menerima slug "kaos-hitam-7"
2. Laravel mencari Product dengan kolom slug tersebut
3. Produk yang ditemukan diberikan ke method show()
4. Controller mengirim produk ke view products.show
5. Halaman detail ditampilkan
```

Kita tidak lagi menulis `findOrFail()` secara manual.

## Apa yang Terjadi Jika Slug Tidak Ada?

Jika pengguna membuka:

```text
/products/produk-tidak-ada-999
```

Laravel tidak menemukan produknya dan otomatis menampilkan halaman:

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
GET|HEAD  products/{product}
```

Pada kode route, parameter tersebut memakai `{product:slug}`. Tampilan
`route:list` pada beberapa versi Laravel mungkin hanya menampilkan
`{product}`.

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

## Kenapa Tidak Mengubah `getRouteKeyName()`?

Model `Product` juga bisa diatur agar semua route memakai slug melalui
`getRouteKeyName()`. Namun, cara tersebut ikut mengubah route edit, update,
dan hapus.

Pada materi ini kita memakai `{product:slug}` karena kebutuhannya hanya halaman
detail. Perubahannya lebih kecil dan fitur CRUD lain tetap bekerja.

## Checklist Tahap 5

- [ ] Route detail lama sudah diganti.
- [ ] Route detail memakai `{product:slug}`.
- [ ] Method `show()` menerima `Product $product`.
- [ ] `Product::findOrFail($id)` sudah tidak dipakai di method `show()`.
- [ ] URL dengan slug menampilkan produk yang benar.
- [ ] Slug yang tidak ada menghasilkan halaman 404.
- [ ] Route edit, update, dan hapus tetap bekerja.

## Inti Tahap 5

> `{product:slug}` meminta Laravel mencari model `Product` berdasarkan kolom
> `slug`, lalu memberikannya langsung kepada method `show()`.

Sekarang halaman detail sudah bisa dibuka melalui URL slug. Namun, tautan dari
halaman daftar masih perlu diperbarui.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke **Tahap 6: mengubah tautan detail produk agar
memakai slug**?

Ketik **"lanjut"** jika sudah siap.
