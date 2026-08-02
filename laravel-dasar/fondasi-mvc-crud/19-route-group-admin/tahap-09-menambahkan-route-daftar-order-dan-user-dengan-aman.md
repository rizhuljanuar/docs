# Tahap 9 — Menambahkan Route Daftar Order dan User dengan Aman

> Fokus: menambahkan route daftar order dan daftar user ke route group admin, tetapi hanya jika controller dan fitur dasarnya memang sudah tersedia di project.

Pada tahap 8, route group admin sudah memiliki dua bagian penting:

```text
Dashboard admin
CRUD Product
```

Bentuknya saat ini:

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

Area admin pada aplikasi yang lebih berkembang biasanya juga memiliki:

- daftar order,
- daftar user.

Sekarang kita akan belajar cara menambahkannya dengan struktur yang sama, **tanpa memaksa membuat fitur order atau manajemen user yang belum dipelajari**.

## Mengapa order dan user termasuk area admin?

Bayangkan toko memiliki dua buku penting di ruang belakang:

```text
Buku order → mencatat pesanan pelanggan
Buku user  → mencatat akun pengguna
```

Kedua buku itu tidak boleh dibuka oleh semua orang. Karena itu, route daftar order dan daftar user cocok dimasukkan ke group admin.

Hasil yang kita inginkan:

| Halaman | URL | Nama route | Controller |
| --- | --- | --- | --- |
| Daftar order | `/admin/orders` | `admin.orders.index` | `OrderController@index` |
| Daftar user | `/admin/users` | `admin.users.index` | `UserController@index` |

Keduanya otomatis memakai middleware `auth` dan `admin`, karena berada di dalam group yang sama.

## Penting: jangan menulis route ke controller yang belum ada

Pada urutan materi dasar ini, fokus utama sebelumnya adalah CRUD Product dan middleware admin. Project kamu mungkin belum memiliki:

```text
app/Http/Controllers/OrderController.php
app/Http/Controllers/UserController.php
```

Jika controller belum ada, jangan langsung menambahkan route berikut:

```php
Route::get('/orders', [OrderController::class, 'index'])
    ->name('orders.index');
```

Laravel akan gagal memuat route jika class controller yang ditunjuk tidak tersedia.

Jadi ada dua keadaan yang benar:

| Keadaan project | Tindakan yang benar |
| --- | --- |
| `OrderController` dan `UserController` belum dibuat | Biarkan route order dan user belum ditulis. Lanjutkan saat materinya tersedia. |
| Controller dan method `index` sudah dibuat | Tambahkan route ke dalam group admin. |

Materi route group ini mengajarkan **tempat dan pola** route-nya. Materi fitur order atau manajemen user akan menjelaskan database, controller, view, dan logika datanya secara terpisah.

## Langkah 1, periksa controller yang tersedia

Sebelum menambah route, periksa apakah project memiliki controller berikut:

```text
app/Http/Controllers/OrderController.php
app/Http/Controllers/UserController.php
```

Lalu pastikan masing-masing memiliki method `index()`.

Contoh bentuk sederhana controller order:

```php
class OrderController extends Controller
{
    public function index()
    {
        // Mengambil data order dan menampilkan view.
    }
}
```

Contoh bentuk sederhana controller user:

```php
class UserController extends Controller
{
    public function index()
    {
        // Mengambil data user dan menampilkan view.
    }
}
```

Kode di atas hanya menunjukkan bentuk method yang dibutuhkan route. Jangan membuat controller kosong hanya agar route tidak error. Controller harus dibuat saat kamu benar-benar membangun fitur order atau daftar user.

## Langkah 2, tambahkan import controller jika memang tersedia

Jika controller sudah ada, buka:

```text
routes/web.php
```

Tambahkan import di bagian atas file:

```php
use App\Http\Controllers\OrderController;
use App\Http\Controllers\UserController;
```

Import hanya diperlukan jika route memakai nama singkat `OrderController` dan `UserController`.

Jika salah satu fitur belum tersedia, jangan menambahkan import dan route untuk fitur tersebut.

## Langkah 3, tambahkan route order di dalam group

Jika `OrderController@index` sudah tersedia, tambahkan route berikut di dalam group admin:

```php
Route::get('/orders', [OrderController::class, 'index'])
    ->name('orders.index');
```

Perhatikan bahwa kita hanya menulis bagian lokalnya:

```text
/orders
orders.index
```

Aturan group melengkapinya menjadi:

```text
URL:        /admin/orders
Nama route: admin.orders.index
Middleware: auth, admin
```

## Langkah 4, tambahkan route user di dalam group

Jika `UserController@index` sudah tersedia, tambahkan route berikut di dalam group admin:

```php
Route::get('/users', [UserController::class, 'index'])
    ->name('users.index');
```

Laravel menggabungkannya dengan aturan group menjadi:

```text
URL:        /admin/users
Nama route: admin.users.index
Middleware: auth, admin
```

## Bentuk group jika dua controller sudah tersedia

Jika dashboard, Product, order, dan user semuanya sudah tersedia, isi group dapat menjadi seperti ini:

```php
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');

        Route::resource('products', ProductController::class);

        Route::get('/orders', [OrderController::class, 'index'])
            ->name('orders.index');

        Route::get('/users', [UserController::class, 'index'])
            ->name('users.index');
    });
```

Mari lihat setiap bagian:

| Baris di dalam group | Hasil akhir |
| --- | --- |
| `'/dashboard'` dan `dashboard` | `/admin/dashboard`, `admin.dashboard` |
| `Route::resource('products', ...)` | `/admin/products/...`, `admin.products....` |
| `'/orders'` dan `orders.index` | `/admin/orders`, `admin.orders.index` |
| `'/users'` dan `users.index` | `/admin/users`, `admin.users.index` |

Semua route berada di bawah satu pintu akses:

```text
auth → user harus login
admin → user harus memiliki role admin
```

## Mengapa order dan user memakai `Route::get()` dahulu?

Pada tahap ini, kita hanya membahas **halaman daftar** order dan user.

Karena halaman daftar dibuka dengan request `GET`, kita memakai:

```php
Route::get(...)
```

Kita belum memakai `Route::resource()` untuk order atau user karena belum tentu aplikasi mengizinkan admin untuk membuat, mengedit, atau menghapus semua data tersebut.

Contohnya:

- Order mungkin hanya boleh dilihat dan statusnya diubah melalui alur khusus.
- User mungkin hanya boleh dilihat, bukan dihapus sembarangan.

Jadi, jangan otomatis memakai resource route hanya karena ada controller. Pilih route sesuai tindakan yang memang dibutuhkan.

## Link ke halaman order dan user

Jika route sudah dibuat, link Blade dapat memakai nama route:

```blade
<a href="{{ route('admin.orders.index') }}">Daftar Order</a>
<a href="{{ route('admin.users.index') }}">Daftar User</a>
```

Jangan membuat link ini jika routenya belum ditulis. Jika Blade memanggil nama route yang belum ada, Laravel akan memberi error route tidak ditemukan.

## Kesalahan umum

### Menambahkan controller dan route palsu untuk “melengkapi” contoh

Jangan membuat `OrderController` atau `UserController` kosong hanya karena contoh route ada. Route group tidak menggantikan materi pembuatan fitur order atau user.

### Menulis `/admin` dua kali

Di dalam group ini, jangan lakukan:

```php
Route::get('/admin/orders', ...);
```

Hasilnya menjadi `/admin/admin/orders`.

Yang benar:

```php
Route::get('/orders', ...);
```

### Menulis `admin.` dua kali

Di dalam group ini, jangan lakukan:

```php
->name('admin.orders.index');
```

Hasilnya menjadi `admin.admin.orders.index`.

Yang benar:

```php
->name('orders.index');
```

### Meletakkan route di luar group

Jika route `/orders` atau `/users` ditulis di luar group, ia tidak otomatis memiliki middleware admin, prefix, dan nama route admin.

Pastikan route berada di antara:

```php
->group(function () {
    // Route order dan user ditulis di sini.
});
```

## Ringkasan

Route group admin tidak hanya untuk dashboard dan Product. Saat fitur pendukungnya sudah ada, group yang sama dapat menampung:

```php
Route::get('/orders', [OrderController::class, 'index'])
    ->name('orders.index');

Route::get('/users', [UserController::class, 'index'])
    ->name('users.index');
```

Hasilnya:

```text
/admin/orders → admin.orders.index
/admin/users  → admin.users.index
```

Keduanya otomatis dilindungi `auth` dan `admin`.

Pada tahap berikutnya, kita akan memperbarui link, redirect, dan form CRUD Product yang masih menggunakan nama route lama, supaya seluruh aplikasi benar-benar mengikuti nama route `admin.*`.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: memperbarui link, redirect, dan form action ke nama route `admin.*`?**
