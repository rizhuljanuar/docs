# Tahap 8 — Memasukkan CRUD Product dengan `Route::resource()` ke dalam Route Group Admin

> Fokus: memasukkan seluruh route CRUD Product ke group admin dengan satu baris `Route::resource()`, sehingga daftar, tambah, simpan, edit, update, dan hapus Product memakai URL, nama route, serta middleware yang sama-sama rapi.

Pada tahap 7, dashboard admin sudah masuk ke route group:

```php
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');
    });
```

Sekarang kita memasukkan **CRUD Product** ke dalam group yang sama.

## Mengapa CRUD Product harus berada di area admin?

CRUD berarti:

```text
Create → menambah Product
Read   → melihat daftar atau detail Product
Update → mengubah Product
Delete → menghapus Product
```

Pada aplikasi toko, tindakan tersebut adalah pekerjaan pengelola. User biasa tidak boleh sembarangan menambah, mengubah, atau menghapus data Product.

Karena itu, Product harus melewati pintu yang sama dengan dashboard:

```text
/admin/products
        ↓
auth memeriksa login
        ↓
admin memeriksa role
        ↓
ProductController menjalankan tindakan CRUD
```

## Ingat kembali `Route::resource()`

Pada materi CRUD Product sebelumnya, kamu sudah memakai bentuk seperti ini:

```php
Route::resource('products', ProductController::class);
```

`Route::resource()` adalah fitur Laravel yang membuat beberapa route CRUD sekaligus. Kita tidak perlu menulis route daftar, tambah, simpan, edit, update, dan hapus secara satu per satu.

Bayangkan `Route::resource()` seperti satu formulir permintaan untuk beberapa pekerjaan Product:

```text
Satu Route::resource()
        ↓
Laravel membuat seluruh pintu CRUD Product
```

## Route Product sebelum masuk group admin

Route lama biasanya seperti ini:

```php
Route::resource('products', ProductController::class);
```

Laravel menghasilkan URL dan nama route seperti:

```text
/products                 → products.index
/products/create          → products.create
/products/{product}       → products.show
/products/{product}/edit  → products.edit
```

Route ini belum mendapatkan prefix `/admin`, awalan nama `admin.`, atau middleware group admin.

## Langkah 1, pastikan `ProductController` di-import

Buka file:

```text
routes/web.php
```

Di bagian atas, pastikan import controller Product tersedia:

```php
use App\Http\Controllers\ProductController;
```

Jika file sudah memiliki import ini, jangan menulisnya dua kali.

## Langkah 2, pindahkan `Route::resource()` ke dalam group

Masukkan route berikut di dalam group admin, tepat setelah route dashboard:

```php
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');

        Route::resource('products', ProductController::class);
    });
```

Bagian baru yang kita tambahkan hanya ini:

```php
Route::resource('products', ProductController::class);
```

Perhatikan: karena group sudah mempunyai `prefix('admin')`, tulis `products`, **bukan** `admin/products` dan bukan `/admin/products`.

## Penjelasan satu baris Product

```php
Route::resource('products', ProductController::class);
```

- `Route::resource(...)` meminta Laravel membuat route CRUD standar.
- `'products'` adalah nama resource dan bagian URL untuk Product.
- `ProductController::class` adalah controller yang menjalankan method CRUD Product.

Semua aturan group otomatis diwariskan oleh route resource ini:

```text
['auth', 'admin'] → semua tindakan Product harus login dan role admin
prefix('admin')   → seluruh URL Product dimulai /admin
name('admin.')    → seluruh nama route Product dimulai admin.
```

## Route CRUD yang dihasilkan Laravel

Setelah `Route::resource('products', ProductController::class)` berada di dalam group, Laravel menghasilkan route berikut.

| Method | URL | Nama route | Method controller | Kegunaan |
| --- | --- | --- | --- | --- |
| `GET` | `/admin/products` | `admin.products.index` | `index` | Menampilkan daftar Product. |
| `GET` | `/admin/products/create` | `admin.products.create` | `create` | Menampilkan form tambah Product. |
| `POST` | `/admin/products` | `admin.products.store` | `store` | Menyimpan Product baru. |
| `GET` | `/admin/products/{product}` | `admin.products.show` | `show` | Menampilkan satu Product. |
| `GET` | `/admin/products/{product}/edit` | `admin.products.edit` | `edit` | Menampilkan form edit Product. |
| `PUT` atau `PATCH` | `/admin/products/{product}` | `admin.products.update` | `update` | Menyimpan perubahan Product. |
| `DELETE` | `/admin/products/{product}` | `admin.products.destroy` | `destroy` | Menghapus Product. |

`{product}` adalah penanda data Product tertentu. Misalnya jika Product memiliki ID `5`, alamat edit dapat menjadi:

```text
/admin/products/5/edit
```

Laravel kemudian memberikan Product tersebut kepada `ProductController` melalui route model binding, sesuai pola yang sudah kamu pelajari pada materi CRUD sebelumnya.

## Mengapa ini lebih rapi daripada menulis semua route sendiri?

Tanpa `Route::resource()`, kamu harus membuat banyak route seperti ini:

```php
Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/create', [ProductController::class, 'create']);
Route::post('/products', [ProductController::class, 'store']);
Route::get('/products/{product}/edit', [ProductController::class, 'edit']);
Route::put('/products/{product}', [ProductController::class, 'update']);
Route::delete('/products/{product}', [ProductController::class, 'destroy']);
```

Dengan resource route di dalam group, satu baris membuat seluruh route CRUD sekaligus dan setiap route langsung menerima aturan admin.

```text
Satu Route::resource()
        ↓
Tujuh route CRUD Product
        ↓
Semua memakai /admin, admin., auth, dan admin
```

## Hapus atau pindahkan route Product lama

Jika `routes/web.php` masih memiliki route lama di luar group:

```php
Route::resource('products', ProductController::class);
```

Jangan biarkan route itu tetap aktif bersamaan dengan resource di dalam group.

Jika keduanya aktif, aplikasi memiliki dua jalur CRUD:

```text
/products        → jalur lama
/admin/products  → jalur admin baru
```

Jalur lama dapat tidak memakai middleware `admin`. User biasa yang sudah login, atau bahkan guest jika route lama tidak memakai `auth`, mungkin masih mengaksesnya.

Untuk materi ini, pindahkan resource lama ke dalam group. Hasil akhirnya hanya satu resource route pengelolaan Product:

```php
Route::resource('products', ProductController::class);
```

Baris tersebut harus berada **di dalam** group admin.

> Jika projectmu sengaja memiliki katalog Product publik, jangan langsung menghapus route publik. Pisahkan dengan jelas route publik yang hanya membaca data dan route admin yang mengelola data. Pada CRUD Product dasar ini, kita fokus pada satu area pengelolaan admin.

## Dampak perubahan nama route

Sebelum dipindahkan, kamu mungkin memakai nama berikut:

```text
products.index
products.create
products.store
products.edit
products.update
products.destroy
```

Setelah berada di group `name('admin.')`, semua nama berubah menjadi:

```text
admin.products.index
admin.products.create
admin.products.store
admin.products.edit
admin.products.update
admin.products.destroy
```

Contoh link daftar Product:

```blade
<a href="{{ route('admin.products.index') }}">Daftar Produk</a>
```

Contoh link form tambah Product:

```blade
<a href="{{ route('admin.products.create') }}">Tambah Produk</a>
```

Contoh redirect setelah menyimpan Product:

```php
return redirect()->route('admin.products.index');
```

Contoh form simpan Product baru:

```blade
<form action="{{ route('admin.products.store') }}" method="POST">
```

Contoh route edit dan hapus membutuhkan Product yang dipilih:

```blade
<a href="{{ route('admin.products.edit', $product) }}">Edit</a>

<form action="{{ route('admin.products.destroy', $product) }}" method="POST">
    @csrf
    @method('DELETE')

    <button type="submit">Hapus</button>
</form>
```

Kita akan memperbarui semua link, redirect, dan `action` form secara sistematis pada tahap 10. Untuk sekarang, pahami dahulu bahwa awalan nama berubah dari `products.` menjadi `admin.products.`.

## Jangan menambahkan `/admin` atau `admin.` dua kali

Kesalahan ini sering terjadi saat baru memakai group.

### URL ditulis dua kali

```php
// Salah, hasil URL menjadi /admin/admin/products
Route::resource('admin/products', ProductController::class);
```

Yang benar:

```php
Route::resource('products', ProductController::class);
```

### Nama route ditulis dua kali

```php
// Tidak diperlukan, group sudah menambahkan admin.
Route::resource('products', ProductController::class)
    ->names('admin.products');
```

Untuk kasus materi ini, cukup gunakan `Route::resource('products', ProductController::class);` di dalam group.

## Perlindungan berlaku untuk semua tindakan CRUD

Inilah bagian penting dari route group. Middleware tidak hanya melindungi halaman daftar Product.

```text
/admin/products                 → daftar, harus admin
/admin/products/create          → form tambah, harus admin
POST /admin/products            → simpan, harus admin
/admin/products/{product}/edit  → form edit, harus admin
PUT /admin/products/{product}   → update, harus admin
DELETE /admin/products/{product} → hapus, harus admin
```

Jadi, user tidak dapat melewati perlindungan hanya dengan mengirim request langsung ke URL simpan atau hapus.

## Ringkasan

Untuk memasukkan CRUD Product ke area admin, tambahkan satu baris ini di dalam route group:

```php
Route::resource('products', ProductController::class);
```

Laravel akan membuat seluruh route CRUD dengan hasil:

```text
URL:        /admin/products/...
Nama route: admin.products....
Akses:      harus login dan harus admin
Controller: ProductController
```

Sekarang dashboard dan CRUD Product sudah berada dalam area admin yang sama. Pada tahap berikutnya, kita akan membahas cara menambahkan route daftar order dan daftar user dengan aman, hanya ketika controllernya memang sudah tersedia.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: menambahkan route daftar order dan daftar user ke route group admin?**
