# Tahap 6 — Memahami Bentuk Lengkap Route Group Admin di `routes/web.php`

> Fokus: menyatukan `auth`, `admin`, `prefix('admin')`, dan `name('admin.')` dalam satu route group yang utuh di Laravel 13+, tanpa terburu-buru memasukkan semua route aplikasi.

Pada tahap 5, kita sudah mempunyai empat aturan untuk area admin:

```text
['auth', 'admin'] → memeriksa login dan role admin
prefix('admin')   → URL diawali /admin
name('admin.')    → nama route diawali admin.
group(...)         → tempat mengumpulkan route admin
```

Sekarang kita akan melihat bagaimana semua bagian itu diletakkan dengan rapi di file route Laravel.

## Di mana route web ditulis?

Untuk halaman website biasa, Laravel 13+ memakai file:

```text
routes/web.php
```

File ini adalah daftar petunjuk URL untuk halaman web aplikasi, misalnya halaman awal, halaman login, dashboard, dan halaman CRUD Product.

Bayangkan `routes/web.php` seperti papan petunjuk di gedung aplikasi:

```text
Halaman umum
Halaman login
Area admin
```

Kita akan membuat satu bagian yang jelas untuk **area admin**.

## Sebelum menulis group, periksa import controller

Route yang memanggil controller perlu mengenal nama controller tersebut.

Di bagian atas `routes/web.php`, gunakan import untuk controller yang memang sudah ada di project kamu. Contoh pada materi CRUD Product dan dashboard admin:

```php
<?php

use App\Http\Controllers\AdminDashboardController;
use App\Http\Controllers\ProductController;
use Illuminate\Support\Facades\Route;
```

Penjelasan:

- `<?php` menandakan file PHP dimulai.
- `use App\Http\Controllers\AdminDashboardController;` membuat route dapat memakai `AdminDashboardController`.
- `use App\Http\Controllers\ProductController;` membuat route dapat memakai `ProductController`.
- `use Illuminate\Support\Facades\Route;` membuat kita dapat menulis `Route::get(...)`, `Route::middleware(...)`, dan `Route::resource(...)`.

> Gunakan nama controller yang sesuai dengan projectmu. Jika dashboard kamu masih memakai closure atau nama controller berbeda, jangan membuat import yang tidak ada hanya karena contoh ini.

## Bentuk lengkap pembungkus route group admin

Tambahkan group ini setelah route umum yang memang sudah ada di `routes/web.php`:

```php
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');
    });
```

Saat ini group hanya berisi satu route dashboard. Ini sengaja dilakukan agar kamu memahami pembungkusnya terlebih dahulu. Pada tahap berikutnya, kita akan menambah route dashboard dengan lebih terarah, lalu Product, order, dan user secara bertahap.

## Membaca kode dari atas ke bawah

Bayangkan setiap baris menambahkan satu aturan pada “map admin”.

```php
Route::middleware(['auth', 'admin'])
```

Semua route di dalam map harus lolos pemeriksaan login dan role admin.

```php
->prefix('admin')
```

Semua URL di dalam map berada di bawah alamat `/admin`.

```php
->name('admin.')
```

Semua route di dalam map mempunyai nama yang dimulai dengan `admin.`.

```php
->group(function () {
```

Mulai bagian tempat kita menaruh route admin.

```php
Route::get('/dashboard', [AdminDashboardController::class, 'index'])
    ->name('dashboard');
```

Buat satu route dashboard di dalam group.

```php
});
```

Tutup area admin.

## Hasil akhir untuk route dashboard

Walaupun route di dalam group hanya ditulis seperti ini:

```php
Route::get('/dashboard', [AdminDashboardController::class, 'index'])
    ->name('dashboard');
```

Laravel menggabungkan seluruh aturan dari group:

| Bagian | Nilai akhir |
| --- | --- |
| HTTP method | `GET` |
| URL | `/admin/dashboard` |
| Nama route | `admin.dashboard` |
| Controller | `AdminDashboardController@index` |
| Middleware | `auth`, lalu `admin` |

Cara membacanya:

```text
GET /admin/dashboard
        ↓
auth memeriksa login
        ↓
admin memeriksa role
        ↓
AdminDashboardController@index menampilkan dashboard
```

## Posisi route group yang rapi

Kamu dapat mengatur `routes/web.php` seperti pola sederhana ini:

```php
<?php

use App\Http\Controllers\AdminDashboardController;
use App\Http\Controllers\ProductController;
use Illuminate\Support\Facades\Route;

// Route halaman umum ditulis di sini.
Route::get('/', function () {
    return view('welcome');
});

// Route group area admin ditulis bersama di sini.
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');
    });
```

Pada contoh ini, `ProductController` sudah di-import untuk persiapan tahap Product. Import tersebut boleh ditambahkan ketika route Product benar-benar akan kamu masukkan pada tahap 8.

Yang paling penting adalah pemisahnya jelas:

```text
Route umum
        ↓
Route group admin
```

Jangan mencampur route admin ke banyak tempat berbeda tanpa alasan. Jika seluruh route admin berada dalam satu group, orang yang membaca file dapat langsung melihat batas area yang dilindungi.

## Jangan menggandakan route lama tanpa memeriksa

Sebelum memindahkan route dashboard atau Product ke group ini, periksa dahulu apakah route lama sudah ada di `routes/web.php`.

Contoh route dashboard lama:

```php
Route::get('/dashboard', [AdminDashboardController::class, 'index'])
    ->name('dashboard');
```

Jika kamu menambahkan route admin baru tetapi membiarkan route lama ini tetap aktif, maka akan ada dua pintu:

```text
/dashboard        → mungkin masih terbuka atau memiliki aturan lama
/admin/dashboard  → memakai aturan admin baru
```

Pada tahap implementasi nanti, route lama harus dipindahkan atau dihapus dengan hati-hati, bukan dibiarkan menjadi jalur tanpa perlindungan.

Prinsip yang sama berlaku untuk Product:

```text
/products        → route lama
/admin/products  → route admin baru
```

Jika aplikasi memang membutuhkan halaman Product publik, route publik dan route admin boleh sama-sama ada, tetapi controller, tindakan, dan aturan aksesnya harus dirancang dengan jelas. Untuk materi CRUD Product dasar, fokus kita adalah memindahkan pengelolaan Product ke area admin.

## Laravel 13+ dan konfigurasi middleware

Route group ini ditulis di `routes/web.php`, sama seperti route web lainnya.

Yang khusus dari Laravel 13+ ada pada **pendaftaran alias middleware custom**. Alias `admin` dari materi 18 berada di:

```text
bootstrap/app.php
```

Bukan di `app/Http/Kernel.php` seperti tutorial Laravel lama.

Ringkasnya:

| Lokasi | Tugas |
| --- | --- |
| `routes/web.php` | Menulis group dan route admin. |
| `bootstrap/app.php` | Mendaftarkan alias `'admin'` ke `AdminMiddleware::class`. |
| `app/Http/Middleware/AdminMiddleware.php` | Memeriksa role user. |

## Cara memeriksa bentuk route, belum menjalankan perubahan

Setelah route benar-benar dipasang di project, Laravel menyediakan perintah:

```bash
php artisan route:list
```

Perintah itu menampilkan URL, nama route, controller, dan middleware yang terdaftar.

Untuk mencari route admin saja, kamu dapat memakai:

```bash
php artisan route:list --path=admin
```

Nanti pada tahap 11, kita akan memakai perintah ini untuk memeriksa apakah route dashboard dan CRUD Product benar-benar memiliki URL `/admin/...`, nama `admin....`, serta middleware `auth` dan `admin`.

## Ringkasan

Bentuk lengkap pembungkus route group admin adalah:

```php
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        // Route dashboard, Product, order, dan user ditulis di sini.
    });
```

Setiap bagian mempunyai satu tugas yang jelas:

```text
auth + admin → menjaga akses
prefix        → menata URL
name          → menata nama route
group         → mengumpulkan route admin
```

Pada tahap berikutnya, kita akan menaruh route dashboard admin ke dalam group ini dan memahami perubahan URL serta nama route yang terjadi.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: memasukkan route dashboard admin ke dalam route group?**
