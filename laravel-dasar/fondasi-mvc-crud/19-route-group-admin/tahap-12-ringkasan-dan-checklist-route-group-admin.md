# Tahap 12 — Ringkasan dan Checklist Route Group Admin

> Penutup materi 19: merapikan route dashboard admin, CRUD Product, order, dan user dengan route group Laravel 13+, sekaligus menjaga aksesnya melalui middleware.

Pada tahap 1, kita mulai dari masalah sederhana:

> Route admin ditulis satu per satu sehingga file `routes/web.php` panjang, berulang, dan mudah salah.

Sekarang kita sudah mempunyai solusi yang rapi. Semua route admin berada di dalam satu group yang memiliki aturan bersama.

## Gambaran besar route group admin

Bayangkan aplikasi CRUD Product adalah toko.

Area depan toko dapat dipakai pelanggan. Area belakang digunakan pengelola untuk membuka dashboard, menambah Product, mengubah Product, menghapus Product, melihat order, dan melihat user.

Route group admin adalah seperti satu pintu menuju area belakang tersebut.

```text
User membuka URL admin
        ↓
Route group menemukan aturan area admin
        ↓
auth memeriksa apakah user sudah login
        ↓
admin memeriksa apakah role user adalah admin
        ↓
Jika lolos, Laravel menjalankan controller yang tepat
        ↓
Controller menampilkan halaman atau memproses CRUD Product
```

Jika user belum login, `auth` mengarahkan user ke login.

Jika user sudah login tetapi role-nya `user`, middleware `admin` menolak akses dengan 403 Forbidden.

Jika user mempunyai role `admin`, Laravel meneruskan request ke controller.

## Kode inti yang perlu diingat

Di `routes/web.php`, bentuk inti route group admin adalah:

```php
use App\Http\Controllers\AdminDashboardController;
use App\Http\Controllers\ProductController;
use Illuminate\Support\Facades\Route;

Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');

        Route::resource('products', ProductController::class);
    });
```

Kode ini adalah kelanjutan langsung dari middleware admin yang dibuat pada materi 18.

### Arti setiap bagian

| Bagian kode | Fungsi sederhana |
| --- | --- |
| `middleware(['auth', 'admin'])` | Memastikan user sudah login dan memiliki role admin. |
| `prefix('admin')` | Menambahkan `/admin` di depan setiap URL di dalam group. |
| `name('admin.')` | Menambahkan `admin.` di depan setiap nama route di dalam group. |
| `group(function () { ... })` | Menjadi tempat mengumpulkan seluruh route area admin. |
| `Route::get('/dashboard', ...)` | Membuat halaman dashboard admin. |
| `Route::resource('products', ...)` | Membuat seluruh route CRUD Product. |

Hasil dashboard:

```text
URL:        /admin/dashboard
Nama route: admin.dashboard
Controller: AdminDashboardController@index
Middleware: auth, admin
```

Hasil route Product:

```text
URL:        /admin/products/...
Nama route: admin.products....
Controller: ProductController
Middleware: auth, admin
```

## Ringkasan 12 tahap

| Tahap | Yang dipelajari |
| --- | --- |
| 1 | Route adalah aturan URL Laravel. Route admin perlu dikelompokkan agar tidak berantakan. |
| 2 | `prefix('admin')` menambahkan `/admin` pada URL seluruh route di dalam group. |
| 3 | `name('admin.')` menambahkan `admin.` pada nama route di dalam group. |
| 4 | Middleware `auth` memastikan user sudah login sebelum membuka area admin. |
| 5 | Middleware `admin` dari materi 18 memeriksa role user setelah `auth`. |
| 6 | Struktur lengkap route group ditulis di `routes/web.php` Laravel 13+. |
| 7 | Route dashboard dipindahkan menjadi `/admin/dashboard` dengan nama `admin.dashboard`. |
| 8 | `Route::resource('products', ProductController::class)` memindahkan seluruh CRUD Product ke area admin. |
| 9 | Route daftar order dan user dapat ditambahkan jika controller serta method-nya sudah ada. |
| 10 | Link Blade, form action, dan redirect controller diperbarui dari `products.*` ke `admin.products.*`. |
| 11 | Struktur route diperiksa dengan Artisan, lalu akses diuji sebagai guest, user biasa, dan admin. |
| 12 | Menyatukan konsep, kode inti, checklist, serta kesalahan umum. |

## Perbedaan URL dan nama route

Dua hal ini sering tertukar. Ingat perbedaannya:

| Bagian | Contoh | Dipakai oleh |
| --- | --- | --- |
| URL | `/admin/products/create` | Browser saat membuka halaman. |
| Nama route | `admin.products.create` | Blade, controller, redirect, dan form Laravel. |

Hubungannya:

```blade
<a href="{{ route('admin.products.create') }}">Tambah Produk</a>
```

Laravel membaca nama `admin.products.create`, lalu menghasilkan URL `/admin/products/create`.

Jadi, lebih baik memakai `route(...)` daripada menulis URL langsung di banyak file.

## Route CRUD Product yang dihasilkan

Baris berikut di dalam group:

```php
Route::resource('products', ProductController::class);
```

menghasilkan route penting berikut:

| Method | URL | Nama route | Method controller | Kegunaan |
| --- | --- | --- | --- | --- |
| `GET` | `/admin/products` | `admin.products.index` | `index` | Daftar Product. |
| `GET` | `/admin/products/create` | `admin.products.create` | `create` | Form tambah Product. |
| `POST` | `/admin/products` | `admin.products.store` | `store` | Simpan Product baru. |
| `GET` | `/admin/products/{product}` | `admin.products.show` | `show` | Detail Product. |
| `GET` | `/admin/products/{product}/edit` | `admin.products.edit` | `edit` | Form edit Product. |
| `PUT` / `PATCH` | `/admin/products/{product}` | `admin.products.update` | `update` | Simpan perubahan. |
| `DELETE` | `/admin/products/{product}` | `admin.products.destroy` | `destroy` | Hapus Product. |

Semua route tersebut otomatis memakai middleware `auth` dan `admin`. Perlindungan bukan hanya pada halaman daftar, tetapi juga pada request simpan, update, dan hapus.

## Perubahan nama route yang wajib dipahami

Setelah Product masuk ke group `name('admin.')`, nama lama berubah:

| Nama lama | Nama baru |
| --- | --- |
| `products.index` | `admin.products.index` |
| `products.create` | `admin.products.create` |
| `products.store` | `admin.products.store` |
| `products.show` | `admin.products.show` |
| `products.edit` | `admin.products.edit` |
| `products.update` | `admin.products.update` |
| `products.destroy` | `admin.products.destroy` |

Perbarui pemakaian nama tersebut pada tiga tempat:

```text
Blade link
Blade form action
Redirect di ProductController
```

Contoh:

```blade
<a href="{{ route('admin.products.create') }}">Tambah Produk</a>

<form action="{{ route('admin.products.store') }}" method="POST">
    @csrf
</form>
```

```php
return redirect()->route('admin.products.index');
```

## Menambahkan order dan user bila fiturnya sudah ada

Jika controller sudah tersedia, route daftar order dan user dapat berada di group yang sama:

```php
use App\Http\Controllers\OrderController;
use App\Http\Controllers\UserController;

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

Hasilnya:

```text
/admin/orders → admin.orders.index
/admin/users  → admin.users.index
```

Jangan menambahkan route `OrderController` atau `UserController` jika controller dan method `index()` belum dibuat. Route harus selalu menunjuk ke class yang benar-benar ada.

## Hubungan dengan materi 18, Middleware Admin

Materi 18 dan 19 saling terhubung.

| Materi 18 | Materi 19 |
| --- | --- |
| Membuat `AdminMiddleware` | Memasang alias `admin` pada route group. |
| Mendaftarkan alias di `bootstrap/app.php` Laravel 13+ | Memakai `middleware(['auth', 'admin'])` di `routes/web.php`. |
| Memeriksa login dan role | Mengelompokkan seluruh halaman admin di satu tempat. |
| Menguji 403 Forbidden | Memastikan setiap route hasil group ikut dilindungi. |

Alurnya:

```text
routes/web.php memakai alias 'admin'
        ↓
bootstrap/app.php memetakan alias tersebut
        ↓
AdminMiddleware.php memeriksa role user
        ↓
Controller hanya berjalan jika role adalah admin
```

Pada Laravel 13+, pendaftaran alias middleware custom berada di:

```text
bootstrap/app.php
```

Bukan di `app/Http/Kernel.php` seperti pada tutorial Laravel versi lama.

## Checklist implementasi route group admin

Gunakan checklist ini setelah menerapkan materi ke project Laravel.

### Struktur dan import route

- [ ] Route web ditulis di `routes/web.php`.
- [ ] `use Illuminate\Support\Facades\Route;` tersedia.
- [ ] `AdminDashboardController` di-import jika route dashboard memakainya.
- [ ] `ProductController` di-import jika resource Product memakainya.
- [ ] `OrderController` dan `UserController` hanya di-import apabila masing-masing benar-benar ada dan dipakai.

### Route group

- [ ] Saya membuat satu group `Route::middleware(['auth', 'admin'])` untuk area admin.
- [ ] Group memakai `->prefix('admin')`.
- [ ] Group memakai `->name('admin.')` dengan titik di akhir.
- [ ] Dashboard ditulis di dalam `->group(function () { ... })`.
- [ ] Resource Product ditulis di dalam group dengan `Route::resource('products', ProductController::class)`.
- [ ] Route order dan user, jika sudah ada, berada di dalam group yang sama.
- [ ] Saya tidak menulis `/admin` atau `admin.` dua kali pada route yang sudah berada di dalam group.

### Middleware dan Laravel 13+

- [ ] Middleware bawaan `auth` tersedia dari sistem authentication project.
- [ ] File `app/Http/Middleware/AdminMiddleware.php` sudah dibuat pada materi 18.
- [ ] Alias `'admin' => AdminMiddleware::class` terdaftar di `bootstrap/app.php`.
- [ ] Saya tidak mencoba mendaftarkan alias custom di `app/Http/Kernel.php`.
- [ ] Group menulis urutan middleware `['auth', 'admin']`.

### Link, form, dan redirect

- [ ] Link dashboard admin memakai `route('admin.dashboard')`.
- [ ] Link daftar dan tambah Product memakai `admin.products.index` serta `admin.products.create`.
- [ ] Form tambah memakai `route('admin.products.store')`.
- [ ] Form edit memakai `route('admin.products.update', $product)`.
- [ ] Form hapus memakai `route('admin.products.destroy', $product)`.
- [ ] Redirect `store()`, `update()`, dan `destroy()` menuju `admin.products.index`.
- [ ] Tidak ada pemanggilan `products.*` lama yang seharusnya menuju CRUD admin.

### Route lama dan keamanan

- [ ] Saya memindahkan atau menghapus route Product lama yang tidak terlindungi, jika CRUD Product hanya untuk admin.
- [ ] Saya tidak membiarkan `/products` menjadi jalur CRUD lama tanpa middleware admin.
- [ ] Jika memiliki katalog Product publik, saya memisahkan route baca publik dan route pengelolaan admin dengan jelas.
- [ ] Saya tidak menganggap `prefix('admin')` sebagai perlindungan. Perlindungan dilakukan oleh middleware.
- [ ] Saya tidak hanya menyembunyikan tombol admin di Blade. Route tetap memakai middleware.

### Verifikasi dan pengujian

- [ ] Saya menjalankan `php artisan route:list --path=admin`.
- [ ] URL route Product dimulai `admin/products` pada hasil Artisan.
- [ ] Nama route Product dimulai `admin.products.` pada hasil Artisan.
- [ ] Kolom middleware menampilkan `auth` dan `admin` untuk route terkait.
- [ ] Guest yang membuka `/admin/products` diarahkan ke login.
- [ ] User dengan role `user` yang membuka URL admin mendapat 403 Forbidden.
- [ ] User dengan role `admin` dapat membuka dashboard dan CRUD Product.
- [ ] Saya menguji setidaknya daftar, tambah, simpan, edit, update, dan hapus Product menggunakan data local atau dummy.

## Kesalahan umum dan cara memperbaikinya

| Kesalahan | Dampak | Cara memperbaiki |
| --- | --- | --- |
| Menulis route admin satu per satu tanpa group | Prefix, nama, dan middleware berulang sehingga mudah lupa | Gunakan satu route group admin. |
| Hanya memakai `prefix('admin')` | URL tampak admin, tetapi user biasa masih dapat mengakses | Tambahkan `middleware(['auth', 'admin'])`. |
| Hanya memakai `auth` | Semua user yang login dapat masuk area admin | Tambahkan alias middleware `admin`. |
| Menulis `/admin` di dalam group prefix | URL menjadi `/admin/admin/...` | Di dalam group cukup tulis `/dashboard` atau `products`. |
| Menulis `admin.` di nama route dalam group | Nama menjadi `admin.admin....` | Di dalam group cukup tulis `dashboard` atau `orders.index`. |
| Menaruh `Route::resource('products', ...)` di luar group | CRUD Product tidak mewarisi prefix dan middleware admin | Pindahkan resource ke dalam group. |
| Membiarkan resource lama dan resource admin aktif bersamaan | Jalur lama mungkin terbuka tanpa perlindungan | Hapus, pindahkan, atau pisahkan route publik dengan jelas. |
| Blade masih memanggil `products.index` | Error route tidak ditemukan | Ubah menjadi `admin.products.index`. |
| Controller masih redirect ke `products.index` | Redirect gagal setelah simpan, update, atau hapus | Ubah menjadi `admin.products.index`. |
| Alias `admin` didaftarkan di `Kernel.php` | Tidak sesuai struktur Laravel 13+ | Daftarkan alias di `bootstrap/app.php`. |
| Menambahkan route order atau user tanpa controller | Laravel gagal memuat class controller | Tambahkan route hanya sesudah controller dan `index()` tersedia. |
| Menguji hanya sebagai admin | Guest atau user biasa mungkin masih bisa masuk | Uji guest, user biasa, dan admin. |

## Batas materi ini

Materi ini memakai satu role sederhana:

```text
admin
```

bersama user biasa dengan role:

```text
user
```

Ini cukup untuk aplikasi CRUD Product dasar.

Pada aplikasi yang lebih besar, izin bisa lebih rinci. Contohnya, seorang `staff` boleh membuat Product tetapi tidak boleh menghapusnya, atau seorang `manager` boleh melihat order tetapi tidak boleh mengelola user.

Laravel menyediakan Gates dan Policies untuk authorization yang lebih detail. Namun jangan buru-buru memakai keduanya sebelum kamu benar-benar memahami:

```text
route
route name
prefix
middleware
authentication
authorization
route group
```

## Penutup

Materi **19. Route Group Admin** selesai.

Sekarang route area admin tidak lagi tersebar dan berulang. Dashboard, CRUD Product, daftar order, serta daftar user dapat tersusun di satu tempat yang jelas:

```text
Route group admin
├── /admin/dashboard
├── /admin/products
├── /admin/orders
└── /admin/users
```

Setiap route di dalam group memiliki aturan yang sama:

```text
URL diawali /admin
Nama route diawali admin.
User harus login
User harus memiliki role admin
```

Kalimat penting untuk diingat:

> **Route group merapikan banyak route yang memiliki aturan sama. Prefix merapikan URL. Name merapikan nama route. Middleware menjaga siapa yang boleh melewati route tersebut.**

Dengan struktur ini, `routes/web.php` lebih mudah dibaca, CRUD Product lebih aman, dan kamu mempunyai pola yang dapat dipakai lagi saat aplikasi Laravel bertambah besar.
