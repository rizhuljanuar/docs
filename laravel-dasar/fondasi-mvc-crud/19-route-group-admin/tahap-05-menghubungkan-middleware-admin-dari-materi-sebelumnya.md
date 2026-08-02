# Tahap 5 — Menghubungkan Middleware `admin` dari Materi Sebelumnya

> Fokus: menambahkan middleware `admin` ke route group bersama `auth`, agar area admin hanya dapat dibuka oleh user yang sudah login **dan** memiliki role `admin`.

Pada tahap 4, kita sudah membuat pembungkus route admin dengan tiga aturan:

```text
middleware('auth') → memastikan user sudah login
prefix('admin')    → membuat URL diawali /admin
name('admin.')     → membuat nama route diawali admin.
```

Namun, ada satu masalah penting.

Middleware `auth` hanya memeriksa apakah user sudah login. User biasa yang memiliki role `user` juga dapat lolos dari `auth`.

Untuk halaman dashboard, CRUD Product, daftar order, dan daftar user, kita membutuhkan pemeriksaan tambahan: **apakah user tersebut adalah admin?**

Jawaban pemeriksaan itu sudah kita buat pada materi 18, yaitu middleware `admin`.

## Ingat kembali hasil materi 18

Pada materi 18, kita sudah menyiapkan tiga bagian berikut:

| Lokasi | Yang sudah dibuat | Tugasnya |
| --- | --- | --- |
| `app/Http/Middleware/AdminMiddleware.php` | Class `AdminMiddleware` | Memeriksa role user yang sedang login. |
| `bootstrap/app.php` | Alias `'admin' => AdminMiddleware::class` | Menghubungkan kata `admin` di route ke class middleware. |
| `routes/web.php` | Pemakaian nanti | Menempatkan `admin` pada route yang harus dilindungi. |

Hubungannya seperti ini:

```text
Route memakai 'admin'
        ↓
bootstrap/app.php menemukan alias 'admin'
        ↓
Laravel menjalankan AdminMiddleware
        ↓
AdminMiddleware memeriksa role === 'admin'
```

Karena alias sudah terdaftar pada Laravel 13+ di `bootstrap/app.php`, kita cukup menulis kata pendek `admin` pada route group.

## Mengapa memakai `auth` dan `admin` bersama?

Kedua middleware mempunyai tugas berbeda.

| Middleware | Pertanyaan yang diperiksa | Jika gagal |
| --- | --- | --- |
| `auth` | “Apakah user sudah login?” | Laravel mengarahkan user ke login. |
| `admin` | “Apakah user yang login mempunyai role `admin`?” | Laravel menolak akses dengan 403 Forbidden. |

Urutan pemeriksaannya:

```text
User membuka /admin/products
        ↓
auth memeriksa login
        ├── Belum login → ke halaman login
        └── Sudah login
              ↓
        admin memeriksa role
              ├── Role user → 403 Forbidden
              └── Role admin → controller dijalankan
```

`auth` ditulis lebih dulu supaya middleware `admin` menerima user yang sudah teridentifikasi oleh Laravel.

## Menulis dua middleware pada route group

Saat middleware lebih dari satu, tulis dalam array:

```php
Route::middleware(['auth', 'admin'])->group(function () {
    // Semua route di sini harus login dan harus admin.
});
```

Penjelasan:

- `['auth', 'admin']` adalah daftar dua middleware.
- `auth` memeriksa login lebih dahulu.
- `admin` adalah alias yang mengarah ke `AdminMiddleware` dari materi 18.
- Setiap route di dalam group harus lolos **dua pemeriksaan** sebelum controller berjalan.

## Menghubungkan dengan route group admin kita

Sekarang gabungkan kode dari tahap 2, tahap 3, dan tahap 4:

```php
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');
    });
```

### Penjelasan baris per baris

```php
Route::middleware(['auth', 'admin'])
```

Mewajibkan setiap route di dalam group melewati login dan pemeriksaan role admin.

```php
->prefix('admin')
```

Menambahkan `/admin` di depan URL. Route dashboard menjadi `/admin/dashboard`.

```php
->name('admin.')
```

Menambahkan `admin.` di depan nama route. Route dashboard menjadi `admin.dashboard`.

```php
->group(function () {
```

Membuka area tempat route admin ditulis.

```php
Route::get('/dashboard', [AdminDashboardController::class, 'index'])
    ->name('dashboard');
```

Route dashboard di dalam group. Hasil lengkapnya:

| Bagian | Hasil akhir |
| --- | --- |
| URL | `/admin/dashboard` |
| Nama route | `admin.dashboard` |
| Middleware | `auth`, lalu `admin` |
| Controller | `AdminDashboardController@index` |

## Tiga hasil saat URL admin dibuka

Gunakan tiga kondisi ini untuk memahami kenapa dua middleware dibutuhkan.

### 1. Guest, belum login

```text
Membuka /admin/dashboard
        ↓
auth tidak menemukan user login
        ↓
Laravel mengarahkan ke halaman login
```

Middleware `admin` tidak perlu memeriksa role karena belum ada user yang login.

### 2. User biasa sudah login

```text
Membuka /admin/dashboard
        ↓
auth berhasil, karena user sudah login
        ↓
admin memeriksa role
        ↓
Role adalah user, bukan admin
        ↓
Laravel memberi 403 Forbidden
```

403 berarti user sudah dikenali, tetapi tidak mempunyai izin untuk mengakses area tersebut.

### 3. Admin sudah login

```text
Membuka /admin/dashboard
        ↓
auth berhasil
        ↓
admin berhasil
        ↓
AdminDashboardController@index dijalankan
        ↓
Halaman dashboard ditampilkan
```

## Mengapa tidak cukup menyembunyikan menu admin?

Kamu mungkin menyembunyikan link dashboard dari user biasa di Blade. Itu baik untuk tampilan, tetapi bukan perlindungan utama.

User masih dapat mengetik URL secara manual:

```text
/admin/products
```

Route group dengan `['auth', 'admin']` tetap memeriksa akses sebelum controller dan halaman dijalankan. Jadi perlindungan terjadi di tempat yang benar, yaitu di pintu masuk route.

## Pastikan alias `admin` sudah ada

Kode route berikut hanya bekerja jika alias dari materi 18 sudah didaftarkan pada Laravel 13+:

```php
// bootstrap/app.php
use App\Http\Middleware\AdminMiddleware;

// ...
->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'admin' => AdminMiddleware::class,
    ]);
})
```

Jangan menyalin ulang konfigurasi ini jika project kamu sudah memilikinya. Cukup periksa bahwa:

- file `AdminMiddleware.php` ada,
- alias `admin` sudah terdaftar di `bootstrap/app.php`,
- middleware memeriksa role `admin`.

Jika alias belum dibuat atau namanya berbeda, Laravel tidak tahu middleware mana yang harus dijalankan ketika route memakai `'admin'`.

## Kesalahan umum

### Hanya memakai `admin`

```php
Route::middleware('admin')
```

Pada contoh middleware materi 18, `AdminMiddleware` memang juga memeriksa user login. Namun tetap gunakan `['auth', 'admin']` pada route group ini.

Alasannya: kode route menjadi jelas, urutan pemeriksaan mudah dipahami, dan `auth` menangani pengalihan guest ke login sebelum pemeriksaan role.

### Menulis middleware dalam format yang keliru

Gunakan array untuk dua middleware:

```php
Route::middleware(['auth', 'admin'])
```

Bukan:

```php
Route::middleware('auth', 'admin')
```

### Mengira `admin` adalah nama role langsung

Pada route:

```php
->middleware('admin')
```

Kata `admin` adalah **alias middleware**, bukan nilai role yang dibaca langsung oleh route. Middleware itulah yang membaca nilai role user, misalnya:

```php
$request->user()->role === 'admin'
```

## Ringkasan

Sekarang route group admin kita mempunyai empat aturan yang bekerja bersama:

```text
['auth', 'admin'] → memeriksa login dan izin admin
prefix('admin')   → merapikan URL admin
name('admin.')    → merapikan nama route admin
```

Kode group saat ini baru berisi dashboard. Pada tahap berikutnya, kita akan melihat bentuk route group admin secara utuh, termasuk import controller dan posisi kode yang aman di `routes/web.php` Laravel 13+.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: memahami bentuk lengkap route group admin di `routes/web.php` Laravel 13+?**
