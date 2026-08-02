# Tahap 10 — Melindungi Route Dashboard dan CRUD Product

> Fokus: memasang middleware `auth` dan `admin` pada route dashboard admin, CRUD Product, daftar order, dan daftar user.

Pada tahap 9, kita sudah mendaftarkan alias berikut di Laravel 13+:

```php
'admin' => AdminMiddleware::class,
```

Sekarang Laravel sudah mengenali nama `admin`. Langkah terakhir untuk mengaktifkan penjaga adalah memasang alias tersebut pada route yang benar.

## Mengapa middleware dipasang pada route?

Middleware hanya bekerja pada request yang melewatinya.

Bayangkan `AdminMiddleware` adalah penjaga yang sudah berdiri di gedung. Penjaga itu harus ditempatkan di pintu area admin, bukan di ruang lain yang tidak berkaitan.

Dalam Laravel, route menentukan pintu atau URL mana yang memakai penjaga.

```text
/admin/dashboard
/admin/products
/admin/orders
/admin/users
        ↓
Route memakai auth dan admin
        ↓
Request diperiksa sebelum controller berjalan
```

## Mengapa memakai dua middleware?

Kita akan memasang:

```php
['auth', 'admin']
```

Tugasnya berbeda:

| Middleware | Tugas |
| --- | --- |
| `auth` | Memastikan user sudah login |
| `admin` | Memastikan user yang login mempunyai role `admin` |

Urutannya perlu mudah dibaca:

```text
auth
    ↓
admin
    ↓
controller
```

Artinya:

1. Laravel memeriksa login terlebih dahulu.
2. Jika belum login, user diarahkan ke login.
3. Jika sudah login, middleware admin memeriksa role.
4. Hanya role `admin` yang dapat sampai ke controller.

## Apa itu route group?

**Route group** adalah cara mengumpulkan beberapa route yang mempunyai aturan sama.

Pada kasus ini, semua halaman admin mempunyai aturan sama:

```text
Harus login
Harus memiliki role admin
URL diawali /admin
```

Daripada menulis middleware yang sama berulang kali pada setiap route, kita membuat satu group.

Analogi sederhananya:

```text
Satu pintu masuk area admin
        ↓
Semua ruangan di belakang pintu memakai pemeriksaan yang sama
```

## Siapkan import controller yang dipakai

Buka file:

```text
routes/web.php
```

Di bagian atas, pastikan controller yang benar-benar digunakan di project kamu sudah di-import. Contoh untuk dashboard dan Product:

```php
use App\Http\Controllers\AdminDashboardController;
use App\Http\Controllers\ProductController;
use Illuminate\Support\Facades\Route;
```

Penjelasan:

- `AdminDashboardController` menangani halaman dashboard admin.
- `ProductController` adalah controller CRUD Product dari materi sebelumnya.
- `Route` dipakai untuk menulis route Laravel.

Jika nama atau lokasi controller Product di project kamu berbeda, gunakan nama controller yang memang ada pada projectmu.

## Langkah 1, buat group area admin

Tambahkan group berikut di `routes/web.php`:

```php
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');

        Route::resource('/products', ProductController::class);
    });
```

Mari pahami satu per satu.

### `Route::middleware(['auth', 'admin'])`

```php
Route::middleware(['auth', 'admin'])
```

Baris ini memasang dua pemeriksaan pada semua route di dalam group:

- `auth` untuk login,
- `admin` untuk role admin.

Jika user tidak lolos salah satu pemeriksaan, controller di dalam group tidak dijalankan.

### `->prefix('admin')`

```php
->prefix('admin')
```

`prefix('admin')` menambahkan `/admin` di depan setiap URL dalam group.

Jadi route berikut:

```php
Route::get('/dashboard', ...)
```

menjadi URL lengkap:

```text
/admin/dashboard
```

Dan resource berikut:

```php
Route::resource('/products', ProductController::class);
```

memiliki URL seperti:

```text
/admin/products
/admin/products/create
/admin/products/{product}/edit
```

Kita tidak perlu menulis `/admin` berulang kali pada setiap route.

### `->name('admin.')`

```php
->name('admin.')
```

`name('admin.')` menambahkan awalan `admin.` pada nama route di dalam group.

Contohnya:

```php
->name('dashboard')
```

menjadi nama route lengkap:

```text
admin.dashboard
```

Untuk resource Product, nama route-nya menjadi seperti:

```text
admin.products.index
admin.products.create
admin.products.store
admin.products.edit
admin.products.update
admin.products.destroy
```

Awalan nama ini membantu kita membedakan route Product admin dari route Product umum, jika suatu saat aplikasi memiliki halaman katalog Product untuk user biasa.

### `->group(function () { ... })`

```php
->group(function () {
    // Semua route admin ditulis di sini.
});
```

Semua route di dalam callback ini otomatis menerima:

- middleware `auth`,
- middleware `admin`,
- prefix URL `/admin`,
- prefix nama route `admin.`.

## Dashboard admin di dalam group

Route dashboard yang kita tulis adalah:

```php
Route::get('/dashboard', [AdminDashboardController::class, 'index'])
    ->name('dashboard');
```

Karena berada di dalam group, hasil akhirnya adalah:

| Bagian | Hasil |
| --- | --- |
| URL | `/admin/dashboard` |
| Nama route | `admin.dashboard` |
| Middleware | `auth`, `admin` |
| Controller | `AdminDashboardController@index` |

Jadi, hanya user yang sudah login dan memiliki role `admin` yang dapat menjalankan method `index()` pada dashboard.

Jika kamu belum membuat `AdminDashboardController` atau Blade dashboard, jangan membuat route ini terlebih dahulu. Kamu dapat menyelesaikan route Product dulu, lalu menambahkan dashboard ketika controllernya sudah ada.

## CRUD Product di dalam group

Baris berikut melindungi seluruh resource CRUD Product:

```php
Route::resource('/products', ProductController::class);
```

Karena berada di dalam group admin, satu baris ini melindungi semua tindakan CRUD Product:

| Tindakan | Method controller | URL lengkap | Dilindungi? |
| --- | --- | --- | --- |
| Daftar Product | `index()` | `/admin/products` | Ya |
| Form tambah Product | `create()` | `/admin/products/create` | Ya |
| Simpan Product | `store()` | `/admin/products` | Ya |
| Form edit Product | `edit()` | `/admin/products/{product}/edit` | Ya |
| Update Product | `update()` | `/admin/products/{product}` | Ya |
| Hapus Product | `destroy()` | `/admin/products/{product}` | Ya |

Ini menjawab masalah utama materi ini: user biasa tidak hanya dicegah melihat daftar Product admin, tetapi juga dicegah membuka form tambah, mengirim penyimpanan, mengedit, memperbarui, atau menghapus Product melalui URL dan request langsung.

> Menyembunyikan tombol tambah, edit, atau hapus di Blade membantu tampilan. Namun group middleware pada route inilah yang melindungi aksi sebenarnya.

## Perbarui link Blade yang lama

Sebelumnya project CRUD mungkin memakai nama route seperti:

```blade
route('products.index')
route('products.create')
route('products.edit', $product)
```

Setelah Product berada di group dengan `->name('admin.')`, nama route berubah menjadi:

```blade
route('admin.products.index')
route('admin.products.create')
route('admin.products.edit', $product)
```

Contoh link tombol tambah Product:

```blade
<a href="{{ route('admin.products.create') }}">
    Tambah Product
</a>
```

Penjelasan:

- `route(...)` membuat URL dari nama route.
- `admin.products.create` adalah nama route resource Product setelah ditambah prefix `admin.`.
- Hasil URL-nya adalah `/admin/products/create`.

Lakukan pencarian pada Blade dan controller untuk semua `route('products...')`, lalu ubah hanya yang memang mengarah ke CRUD Product admin.

Jangan asal mengganti link Product umum bila suatu saat kamu memang memiliki halaman Product publik yang terpisah.

## Daftar order dan daftar user

Ketika controller untuk order dan user sudah tersedia, tambahkan route-nya **di dalam group yang sama**.

Contoh:

```php
use App\Http\Controllers\OrderController;
use App\Http\Controllers\UserController;

Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');

        Route::resource('/products', ProductController::class);

        Route::get('/orders', [OrderController::class, 'index'])
            ->name('orders.index');

        Route::get('/users', [UserController::class, 'index'])
            ->name('users.index');
    });
```

Hasilnya:

| Halaman | URL | Nama route | Middleware |
| --- | --- | --- | --- |
| Dashboard admin | `/admin/dashboard` | `admin.dashboard` | `auth`, `admin` |
| Daftar Product | `/admin/products` | `admin.products.index` | `auth`, `admin` |
| Daftar order | `/admin/orders` | `admin.orders.index` | `auth`, `admin` |
| Daftar user | `/admin/users` | `admin.users.index` | `auth`, `admin` |

Jika kamu belum membuat `OrderController` atau `UserController`, jangan menyalin dua route terakhir dulu. Laravel akan error jika route menunjuk ke controller yang belum ada. Tambahkan setelah materi order dan manajemen user dibuat.

## Jangan meninggalkan route Product lama tanpa perlindungan

Ini bagian penting.

Misalnya sebelumnya kamu sudah memiliki route ini di luar group:

```php
Route::resource('/products', ProductController::class);
```

Jika kamu menambahkan resource baru dalam group admin tetapi membiarkan resource lama di luar group, maka URL lama `/products` masih dapat diakses tanpa middleware admin.

Pilih satu struktur yang jelas:

### Jika seluruh CRUD Product hanya untuk admin

Pindahkan resource Product ke dalam group admin dan hapus atau ganti route lama yang tidak terlindungi.

Hasil akhirnya, CRUD admin hanya memakai:

```text
/admin/products
```

### Jika Product juga memiliki halaman publik

Pisahkan route publik dan route pengelolaan:

```text
/products                 → halaman daftar Product publik, hanya baca
/admin/products           → CRUD pengelolaan Product, memakai auth dan admin
```

Untuk kondisi kedua, lebih baik gunakan controller yang berbeda agar tanggung jawabnya jelas, misalnya:

```text
ProductCatalogController  → halaman publik
ProductController         → pengelolaan Product admin
```

Pada materi dasar ini, fokus kita adalah melindungi CRUD pengelolaan Product. Jangan membuat dua set route tanpa memahami fungsi masing-masing.

## Periksa route yang sudah dilindungi

Dari root project Laravel, jalankan:

```bash
php artisan route:list --path=admin
```

Perintah ini hanya membaca dan menampilkan route yang URL-nya mengandung `admin`.

Periksa kolom berikut:

| Kolom | Yang diharapkan |
| --- | --- |
| URI | Memulai dengan `admin/` |
| Name | Memulai dengan `admin.` |
| Middleware | Memuat `auth` dan `admin` |

Contoh konsep hasil:

```text
GET|HEAD  admin/dashboard          admin.dashboard       auth, admin
GET|HEAD  admin/products           admin.products.index  auth, admin
GET|HEAD  admin/products/create    admin.products.create auth, admin
```

Jika route tidak muncul, periksa apakah group route ditulis di `routes/web.php` dan file tersebut didaftarkan pada `bootstrap/app.php` melalui `withRouting(...)`.

## Alur setelah route group dipasang

Sekarang alur akses `/admin/products/create` menjadi:

```text
Browser membuka /admin/products/create
        ↓
Laravel menemukan route di group admin
        ↓
Middleware auth memeriksa login
        ↓
Middleware admin memeriksa role
        ↓
Jika role admin, ProductController@create dijalankan
        ↓
Jika bukan admin, Laravel memberi respons 403
```

Untuk user belum login:

```text
Browser membuka /admin/products/create
        ↓
Middleware auth menemukan user belum login
        ↓
Laravel mengarahkan ke login
        ↓
ProductController@create tidak dijalankan
```

## Kesalahan umum yang perlu dihindari

### 1. Hanya melindungi route `index`

Jangan hanya memberi middleware pada daftar Product. Route `create`, `store`, `edit`, `update`, dan `destroy` juga harus dilindungi.

`Route::resource()` di dalam group melindungi semuanya sekaligus.

### 2. Meletakkan satu route di luar group

Route yang berada di luar group tidak otomatis menerima middleware `auth` dan `admin`.

### 3. Lupa memperbarui nama route di Blade atau controller

Setelah memakai `name('admin.')`, nama route Product berubah. Periksa link, redirect, form action, pagination, dan tombol terkait CRUD admin.

### 4. Menggunakan controller order atau user yang belum ada

Tambahkan route hanya untuk controller yang sudah dibuat di project.

### 5. Menganggap menu tersembunyi adalah perlindungan

User masih dapat mengetik URL. Perlindungan utama berada pada middleware route group.

## Yang perlu diingat pada tahap ini

1. Route group memasang aturan yang sama pada banyak halaman admin.
2. `middleware(['auth', 'admin'])` memastikan user sudah login dan mempunyai role admin.
3. `prefix('admin')` membuat URL seperti `/admin/products`.
4. `name('admin.')` membuat nama route seperti `admin.products.index`.
5. Resource Product di dalam group melindungi tambah, simpan, edit, update, dan hapus Product sekaligus.
6. Jangan membiarkan route CRUD lama yang tidak terlindungi di luar group.
7. Gunakan `php artisan route:list --path=admin` untuk memeriksa middleware route.

Tahap berikutnya akan menguji tiga kondisi akses, belum login, user biasa, dan admin, serta memahami respons 403 secara lebih ramah.

---

**Apakah kamu ingin lanjut ke tahap 11: menguji akses admin dan menangani respons 403 Forbidden?**
