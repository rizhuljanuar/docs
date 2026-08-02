# Tahap 4 — Memahami Middleware `auth` pada Route Group

> Fokus: memasang middleware `auth` pada route group agar semua halaman admin hanya dapat dibuka oleh user yang sudah login.

Pada tahap 2, kita memakai `prefix('admin')` untuk merapikan **URL** admin.

Pada tahap 3, kita memakai `name('admin.')` untuk merapikan **nama route** admin.

Dua hal tersebut membuat struktur route lebih jelas, tetapi belum mengunci pintu area admin. Siapa pun masih dapat mengetik URL seperti ini di browser:

```text
/admin/dashboard
/admin/products
```

Sekarang kita memakai middleware `auth` sebagai pemeriksaan pertama.

## Ingat kembali materi 18

Pada materi 18, kita membedakan dua pemeriksaan:

| Middleware atau konsep | Pertanyaan yang dijawab |
| --- | --- |
| `auth` | “Apakah user sudah login?” |
| `admin` | “Apakah user yang sudah login memiliki role admin?” |

Tahap ini hanya fokus pada `auth`.

Bayangkan area admin sebagai ruang belakang toko:

```text
User datang ke pintu admin
        ↓
Middleware auth memeriksa apakah user membawa kartu masuk, yaitu sudah login
        ↓
Belum login → arahkan ke halaman login
Sudah login → boleh menuju pemeriksaan berikutnya
```

Middleware `auth` belum memeriksa role. Artinya, pada tahap ini user biasa yang sudah login masih bisa melewati pemeriksaan `auth`. Nanti pada tahap 5, middleware `admin` dari materi sebelumnya akan memeriksa apakah user tersebut benar-benar admin.

## Mengapa `auth` dipasang pada route group?

Area admin memiliki banyak halaman:

- dashboard admin,
- daftar produk,
- tambah produk,
- edit produk,
- hapus produk,
- daftar order,
- daftar user.

Semua halaman ini mempunyai aturan awal yang sama:

> User harus login terlebih dahulu.

Kalau `auth` ditulis satu per satu, kode menjadi berulang:

```php
Route::get('/admin/dashboard', [AdminDashboardController::class, 'index'])
    ->middleware('auth');

Route::get('/admin/products', [ProductController::class, 'index'])
    ->middleware('auth');

Route::get('/admin/orders', [OrderController::class, 'index'])
    ->middleware('auth');
```

Lebih rapi jika `auth` dipasang sekali pada pembungkus group.

## Bentuk dasar middleware group

Tambahkan middleware dengan `Route::middleware(...)` sebelum membuka group:

```php
Route::middleware('auth')->group(function () {
    // Semua route di sini harus login.
});
```

Penjelasan:

- `Route::middleware('auth')` memberi tahu Laravel untuk memakai middleware bawaan bernama `auth`.
- `->group(function () { ... })` mengumpulkan route yang harus melewati middleware tersebut.
- Semua route di dalam kurung kurawal otomatis memakai `auth`.

Jadi, daripada memasang penjaga pada setiap pintu, kita membuat satu pintu masuk untuk seluruh area.

## Menggabungkan `auth`, prefix, dan nama route

Sekarang sambungkan pengetahuan tahap 2 dan tahap 3:

```php
Route::middleware('auth')
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');
    });
```

Kode ini belum memakai middleware `admin`. Anggap ini sebagai latihan memahami satu lapis pemeriksaan terlebih dahulu.

### Penjelasan baris per baris

```php
Route::middleware('auth')
```

Semua route di dalam group wajib melalui pemeriksaan login.

```php
->prefix('admin')
```

Semua URL di dalam group mendapat awalan `/admin`.

```php
->name('admin.')
```

Semua nama route di dalam group mendapat awalan `admin.`.

```php
->group(function () {
```

Membuka area tempat kita menulis route admin.

```php
Route::get('/dashboard', [AdminDashboardController::class, 'index'])
    ->name('dashboard');
```

Membuat route dashboard. Setelah aturan group digabungkan, hasilnya adalah:

| Bagian | Hasil akhir |
| --- | --- |
| URL | `/admin/dashboard` |
| Nama route | `admin.dashboard` |
| Middleware | `auth` |
| Controller | `AdminDashboardController@index` |

## Apa yang terjadi saat user membuka dashboard?

Alurnya seperti ini:

```text
User membuka /admin/dashboard
        ↓
Laravel menemukan route di dalam group admin
        ↓
Middleware auth memeriksa status login
        ↓
Belum login?
        ├── Ya → Laravel mengarahkan user ke halaman login
        └── Tidak, sudah login → AdminDashboardController@index dijalankan
```

Middleware berjalan **sebelum** controller. Karena itu, controller tidak perlu menulis pemeriksaan login yang sama pada setiap method.

## Menerapkan `auth` pada route Product

Saat route Product dimasukkan nanti, semua route CRUD juga otomatis wajib login:

```php
Route::middleware('auth')
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');

        Route::resource('/products', ProductController::class);
    });
```

Karena `Route::resource()` berada di dalam group, route berikut semuanya mewarisi `auth`:

```text
/admin/products
/admin/products/create
/admin/products/{product}/edit
```

Termasuk request untuk menyimpan, mengubah, dan menghapus Product.

## Penting: `auth` bukan middleware admin

Perhatikan perbandingan ini:

| Kondisi user | Dengan `auth` saja | Dengan `auth` dan `admin` |
| --- | --- | --- |
| Belum login | Ditolak, diarahkan ke login | Ditolak, diarahkan ke login |
| Login sebagai `user` | Diizinkan masuk | Ditolak dengan 403 Forbidden |
| Login sebagai `admin` | Diizinkan masuk | Diizinkan masuk |

Jadi, kode berikut belum cukup untuk area admin sungguhan:

```php
Route::middleware('auth')
```

Kode itu hanya memastikan user mempunyai akun dan sudah masuk. User biasa juga dapat lolos.

Pada aplikasi CRUD Product ini, area dashboard, tambah Product, edit Product, hapus Product, daftar order, dan daftar user hanya boleh dijalankan admin. Karena itu, langkah selanjutnya adalah memasang middleware `admin` bersama `auth`.

## Kesalahan umum

### Mengira prefix melindungi halaman

```php
Route::prefix('admin')->group(function () {
    // ...
});
```

`prefix('admin')` hanya memberi awalan URL. Ia tidak memeriksa login.

### Mengira nama route melindungi halaman

```php
Route::name('admin.')->group(function () {
    // ...
});
```

`name('admin.')` hanya merapikan nama route. Ia juga tidak memeriksa login.

### Menaruh `auth` pada satu route saja

Jika hanya dashboard memakai `auth`, tetapi route Product tidak, maka Product bisa tidak terlindungi. Pasang `auth` pada group supaya semua route admin konsisten.

## Ringkasan

`middleware('auth')` berarti:

> “Sebelum menjalankan route ini, pastikan user sudah login.”

Sekarang route group admin telah mempunyai tiga aturan:

```text
middleware('auth') → memeriksa login
prefix('admin')    → merapikan URL admin
name('admin.')     → merapikan nama route admin
```

Namun, setelah user lolos login, Laravel masih perlu memeriksa role admin. Middleware tersebut sudah kamu buat pada materi 18 dan akan kita hubungkan ke route group pada tahap berikutnya.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: menghubungkan middleware `admin` dari materi 18 ke route group admin?**
