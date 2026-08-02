# Tahap 7 — Memasukkan Route Dashboard Admin ke dalam Group

> Fokus: memindahkan route dashboard ke area admin yang sudah memiliki `auth`, `admin`, prefix `/admin`, dan nama route `admin.`.

Pada tahap 6, kita sudah membuat bentuk pembungkus route group admin:

```php
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        // Route admin ditulis di sini.
    });
```

Sekarang kita memasukkan satu halaman terlebih dahulu, yaitu **dashboard admin**.

Kita mulai dari dashboard karena dashboard adalah pintu utama pengelola aplikasi. Setelah dashboard berhasil masuk ke group, pola yang sama akan dipakai untuk CRUD Product, order, dan user.

## Tujuan tahap ini

Setelah tahap ini, route dashboard admin mempunyai hasil berikut:

| Bagian | Hasil |
| --- | --- |
| URL halaman | `/admin/dashboard` |
| Nama route | `admin.dashboard` |
| Controller | `AdminDashboardController@index` |
| Middleware | `auth` dan `admin` |

Artinya, hanya user yang sudah login dan memiliki role `admin` yang dapat membuka dashboard.

## Bayangkan memindahkan ruang dashboard

Sebelumnya, dashboard mungkin masih berada di jalur umum:

```text
/dashboard
```

Sekarang kita memindahkannya ke area belakang toko:

```text
/admin/dashboard
```

Perpindahan ini bukan hanya mengganti alamat. Dashboard juga melewati pintu pemeriksaan yang sama dengan semua halaman admin:

```text
/admin/dashboard
        ↓
auth memeriksa login
        ↓
admin memeriksa role
        ↓
Dashboard ditampilkan jika user adalah admin
```

## Bentuk route dashboard sebelum memakai group

Pada project kamu, bentuk route lama mungkin mirip salah satu contoh berikut:

### Menggunakan controller

```php
Route::get('/dashboard', [AdminDashboardController::class, 'index'])
    ->name('dashboard');
```

### Menggunakan closure sederhana

```php
Route::get('/dashboard', function () {
    return view('dashboard');
})->name('dashboard');
```

Gunakan bentuk yang sesuai projectmu. Materi ini menggunakan controller karena lebih cocok untuk dashboard yang nantinya menampilkan ringkasan Product, order, dan user.

## Langkah 1, pastikan import controller tersedia

Buka:

```text
routes/web.php
```

Jika dashboard memakai controller, pastikan bagian atas file memiliki import berikut:

```php
use App\Http\Controllers\AdminDashboardController;
```

Import ini membuat Laravel mengenali nama singkat `AdminDashboardController` pada route.

Jika nama controller dashboard di project kamu berbeda, pakai nama controller yang benar. Jangan membuat class baru hanya untuk menyesuaikan contoh dokumentasi.

## Langkah 2, letakkan dashboard di dalam group admin

Tulis route dashboard **di dalam** `group(function () { ... })`:

```php
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');
    });
```

Perhatikan posisi route dashboard:

```php
->group(function () {
    // Di dalam area admin
    Route::get('/dashboard', [AdminDashboardController::class, 'index'])
        ->name('dashboard');
});
```

Karena berada di dalam group, route dashboard otomatis menerima semua aturan group. Kita tidak perlu menulis `/admin`, `admin.`, `auth`, dan `admin` lagi pada route dashboard itu sendiri.

## Cara Laravel membentuk hasil akhirnya

Route dashboard yang kita tulis di dalam group:

```php
Route::get('/dashboard', [AdminDashboardController::class, 'index'])
    ->name('dashboard');
```

Laravel menggabungkannya dengan aturan pembungkus:

```text
prefix('admin') + /dashboard
        ↓
/admin/dashboard

name('admin.') + dashboard
        ↓
admin.dashboard

['auth', 'admin']
        ↓
Diterapkan sebelum controller dijalankan
```

Hasil akhir:

```text
GET /admin/dashboard → admin.dashboard → AdminDashboardController@index
```

## Penjelasan kode route dashboard

```php
Route::get('/dashboard', [AdminDashboardController::class, 'index'])
```

- `Route::get` berarti route ini menerima request untuk membuka halaman.
- `'/dashboard'` adalah bagian akhir URL. Prefix group mengubahnya menjadi `/admin/dashboard`.
- `AdminDashboardController::class` adalah class controller yang mengurus dashboard.
- `'index'` adalah method pada controller yang dijalankan.

```php
->name('dashboard');
```

Ini memberi nama bagian akhir `dashboard`. Awalan group membuat nama lengkapnya menjadi `admin.dashboard`.

## Memindahkan route lama, jangan membuat dua jalur tanpa sengaja

Jika route dashboard lama seperti ini masih ada:

```php
Route::get('/dashboard', [AdminDashboardController::class, 'index'])
    ->name('dashboard');
```

Jangan biarkan route tersebut tetap aktif setelah kamu membuat route group yang baru. Pindahkan route itu ke dalam group atau hapus route lama setelah memastikan route baru benar.

Mengapa?

```text
Route lama: /dashboard
Route baru: /admin/dashboard
```

Jika keduanya aktif, ada dua alamat menuju dashboard. Route lama mungkin tidak memakai middleware `admin`, sehingga dapat menjadi pintu yang tidak sengaja lebih longgar.

Untuk aplikasi CRUD Product dasar ini, gunakan satu jalur admin yang jelas:

```text
/admin/dashboard
```

## Memperbarui link dashboard

Setelah nama route berubah dari `dashboard` menjadi `admin.dashboard`, link lama perlu diperbarui.

### Sebelum memakai route group

```blade
<a href="{{ route('dashboard') }}">Dashboard</a>
```

### Setelah dashboard masuk group admin

```blade
<a href="{{ route('admin.dashboard') }}">Dashboard Admin</a>
```

Jangan menulis URL secara langsung seperti ini:

```blade
<a href="/admin/dashboard">Dashboard Admin</a>
```

Lebih baik memakai `route('admin.dashboard')` karena Laravel yang menentukan URL berdasarkan nama route. Jika URL kelak berubah, kamu tidak perlu mengganti link di setiap file Blade.

Jika kamu memiliki redirect menuju dashboard, perbarui juga:

```php
return redirect()->route('admin.dashboard');
```

Contoh yang perlu dicari di project:

```text
route('dashboard')
redirect()->route('dashboard')
```

Jangan mengubah route yang memang bukan dashboard admin. Periksa konteks setiap penggunaan sebelum menggantinya.

## Tiga kondisi akses dashboard

Setelah dashboard berada dalam group, hasil aksesnya harus seperti berikut:

| Pengunjung | Membuka `/admin/dashboard` | Hasil yang diharapkan |
| --- | --- | --- |
| Guest, belum login | Ya | Diarahkan ke halaman login oleh `auth`. |
| User biasa, role `user` | Ya | Ditolak dengan 403 Forbidden oleh `admin`. |
| User admin, role `admin` | Ya | Dashboard tampil. |

Inilah keuntungan group: route dashboard langsung memakai perlindungan yang sama seperti CRUD Product yang akan kita masukkan pada tahap berikutnya.

## Kesalahan umum

### Menulis `/admin` dua kali

Karena group sudah memakai `prefix('admin')`, jangan menulis `/admin/dashboard` lagi di dalam group:

```php
// Salah, URL akhirnya menjadi /admin/admin/dashboard
Route::get('/admin/dashboard', ...);
```

Yang benar:

```php
Route::get('/dashboard', ...);
```

### Menulis `admin.` dua kali

Karena group sudah memakai `name('admin.')`, jangan menulis nama lengkap di dalamnya:

```php
// Salah, nama route akhirnya menjadi admin.admin.dashboard
->name('admin.dashboard');
```

Yang benar:

```php
->name('dashboard');
```

### Lupa memperbarui link

Jika Blade masih memakai `route('dashboard')`, Laravel dapat menampilkan error route tidak ditemukan setelah route lama dipindahkan. Gunakan `route('admin.dashboard')` untuk dashboard admin.

## Ringkasan

Route dashboard di dalam group cukup ditulis seperti ini:

```php
Route::get('/dashboard', [AdminDashboardController::class, 'index'])
    ->name('dashboard');
```

Karena berada di dalam group, Laravel menghasilkan:

```text
URL:        /admin/dashboard
Nama route: admin.dashboard
Akses:      harus login dan harus admin
```

Sekarang dashboard sudah menjadi bagian resmi dari area admin. Langkah berikutnya akan memasukkan seluruh route CRUD Product ke group yang sama dengan `Route::resource()`.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: memasukkan route CRUD Product dengan `Route::resource()` ke dalam route group admin?**
