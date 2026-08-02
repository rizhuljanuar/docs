# Tahap 9 — Mendaftarkan Alias Middleware Admin di `bootstrap/app.php`

> Fokus: memberi nama singkat `admin` kepada class `AdminMiddleware` agar Laravel 13+ dapat memakainya pada route.

Pada tahap 8, kita sudah menulis aturan pemeriksaan pada file ini:

```text
app/Http/Middleware/AdminMiddleware.php
```

Class tersebut sudah tahu cara memeriksa login dan role `admin`. Tetapi Laravel belum tahu bahwa saat kita menulis kata `admin` pada route, Laravel harus menjalankan `AdminMiddleware`.

Karena itu, kita perlu membuat **alias middleware**.

## Apa itu alias middleware?

Alias adalah nama pendek yang menunjuk ke nama class yang lebih panjang.

Bayangkan nama lengkap petugas toko adalah:

```text
Bapak Penjaga Ruang Pengelola
```

Agar mudah dipanggil, semua orang cukup menyebut:

```text
penjaga-admin
```

Dalam Laravel, class lengkap middleware kita adalah:

```php
App\Http\Middleware\AdminMiddleware::class
```

Kita akan memberinya nama pendek:

```text
admin
```

Setelah alias dibuat, route nanti dapat memakai:

```php
->middleware('admin')
```

Daripada harus menulis nama class lengkap berulang-ulang.

## Mengapa alias penting?

Alias membuat route lebih singkat dan mudah dibaca.

Bandingkan dua bentuk berikut:

```php
->middleware(\App\Http\Middleware\AdminMiddleware::class)
```

Dengan:

```php
->middleware('admin')
```

Untuk materi pemula dan project CRUD Product, bentuk alias lebih mudah dipahami. Saat membaca route, kamu langsung tahu bahwa halaman tersebut memakai pemeriksaan admin.

## Lokasi pendaftaran middleware pada Laravel 13+

Pada Laravel 13+, alias middleware didaftarkan di file:

```text
bootstrap/app.php
```

Ini berbeda dari banyak tutorial Laravel lama yang memakai:

```text
app/Http/Kernel.php
```

Untuk Laravel 13+, **jangan mencari atau membuat `Kernel.php` hanya untuk alias middleware**. Gunakan `bootstrap/app.php`.

File bawaan Laravel 13+ mempunyai bagian seperti ini:

```php
->withMiddleware(function (Middleware $middleware): void {
    //
})
```

Bagian `withMiddleware(...)` adalah tempat kita mengatur middleware aplikasi.

## Langkah 1, tambahkan import `AdminMiddleware`

Buka file:

```text
bootstrap/app.php
```

Di bagian atas file, tambahkan import untuk middleware yang kita buat:

```php
use App\Http\Middleware\AdminMiddleware;
```

Bagian atas file kemudian menjadi seperti ini:

```php
<?php

use App\Http\Middleware\AdminMiddleware;
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;
```

Penjelasan baris baru:

```php
use App\Http\Middleware\AdminMiddleware;
```

Baris ini memberi tahu file `bootstrap/app.php` lokasi class `AdminMiddleware`.

Dengan import ini, kita dapat menulis nama pendek class berikut di dalam konfigurasi:

```php
AdminMiddleware::class
```

## Langkah 2, daftarkan alias `admin`

Masih di file `bootstrap/app.php`, ubah bagian `withMiddleware(...)` menjadi:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'admin' => AdminMiddleware::class,
    ]);
})
```

Mari pahami bagian pentingnya.

### `$middleware->alias([...])`

```php
$middleware->alias([
    // Alias middleware ditulis di sini.
]);
```

- `$middleware` adalah alat konfigurasi middleware yang diberikan Laravel.
- `alias([...])` mendaftarkan pasangan nama pendek dan class middleware.
- Isi di dalam `[ ... ]` adalah daftar alias yang ingin kita buat.

### `'admin' => AdminMiddleware::class`

```php
'admin' => AdminMiddleware::class,
```

Baris ini berarti:

```text
Saat route memakai alias admin
        ↓
Laravel menjalankan App\Http\Middleware\AdminMiddleware
```

Penjelasannya:

- `'admin'` adalah nama alias yang nanti ditulis di route.
- `=>` berarti “menunjuk ke”.
- `AdminMiddleware::class` adalah class middleware yang dibuat pada tahap 7 dan diisi aturannya pada tahap 8.

## Bentuk lengkap `bootstrap/app.php`

Untuk project Laravel 13+ standar, bagian penting file akan menjadi seperti ini:

```php
<?php

use App\Http\Middleware\AdminMiddleware;
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware): void {
        $middleware->alias([
            'admin' => AdminMiddleware::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions): void {
        //
    })->create();
```

Jangan mengganti seluruh file secara membabi buta. Project kamu mungkin sudah memiliki pengaturan middleware atau routing lain. Tambahkan import dan alias tanpa menghapus konfigurasi yang sudah ada.

## Hubungan alias dengan route

Setelah alias didaftarkan, Laravel memahami hubungan ini:

```text
routes/web.php
    ↓
->middleware('admin')
    ↓
bootstrap/app.php
    ↓
'admin' => AdminMiddleware::class
    ↓
app/Http/Middleware/AdminMiddleware.php
    ↓
Periksa login dan role admin
```

Artinya, kita sudah menyambungkan tiga bagian yang sebelumnya terpisah:

| Bagian | Tugas |
| --- | --- |
| `AdminMiddleware.php` | Menulis aturan pemeriksaan role |
| `bootstrap/app.php` | Memberi alias `admin` kepada middleware |
| `routes/web.php` | Nanti memasang alias `admin` pada halaman yang dilindungi |

Tahap berikutnya akan melakukan bagian terakhir, yaitu memasang `auth` dan `admin` pada kelompok route dashboard serta CRUD Product.

## Apakah perlu menjalankan `php artisan config:clear`?

Biasanya Laravel membaca perubahan `bootstrap/app.php` saat request baru dijalankan.

Namun, jika project sebelumnya memakai cache konfigurasi dan Laravel masih seperti tidak mengenali perubahan, jalankan dari root project:

```bash
php artisan config:clear
```

Perintah ini menghapus cache konfigurasi aplikasi, bukan menghapus tabel atau data Product maupun user.

Kamu tidak perlu menjalankannya setiap kali membuat perubahan. Gunakan jika memang perubahan konfigurasi belum terbaca.

## Cara memeriksa alias tanpa membuka halaman admin

Setelah alias didaftarkan, kamu dapat melihat daftar route dengan:

```bash
php artisan route:list
```

Pada tahap ini, route belum memakai alias `admin`, jadi alias belum terlihat pada kolom Middleware route. Pemeriksaan yang lebih nyata akan dilakukan setelah tahap 10 saat route admin sudah dilindungi.

Jika menjalankan `php artisan route:list` menghasilkan error class tidak ditemukan, periksa tiga hal berikut:

1. File benar-benar ada di `app/Http/Middleware/AdminMiddleware.php`.
2. Class di dalam file bernama `AdminMiddleware`.
3. Import pada `bootstrap/app.php` ditulis tepat:

```php
use App\Http\Middleware\AdminMiddleware;
```

## Kesalahan umum yang perlu dihindari

### 1. Mendaftarkan alias di `app/Http/Kernel.php`

Cara ini berasal dari struktur Laravel lama. Untuk Laravel 13+, daftarkan alias di `bootstrap/app.php`.

### 2. Salah menulis alias

Jika kamu mendaftarkan:

```php
'admin' => AdminMiddleware::class,
```

maka route harus memakai:

```php
->middleware('admin')
```

Alias harus sama persis. `Admin`, `administrator`, atau `isAdmin` adalah nama yang berbeda dan tidak otomatis dikenali.

### 3. Lupa import class

Jika memakai `AdminMiddleware::class` tanpa import yang benar, PHP tidak tahu lokasi class tersebut.

### 4. Menghapus konfigurasi middleware yang sudah ada

Jika callback `withMiddleware(...)` sudah berisi pengaturan lain, pertahankan pengaturan tersebut. Tambahkan `alias(...)`, jangan menghapus konfigurasi sebelumnya.

### 5. Mengira alias saja sudah melindungi halaman

Mendaftarkan alias hanya memperkenalkan nama `admin` kepada Laravel. Halaman belum terlindungi sampai alias itu dipasang pada route.

## Yang perlu diingat pada tahap ini

1. Alias adalah nama singkat untuk class middleware.
2. Alias `admin` menunjuk ke `AdminMiddleware::class`.
3. Pada Laravel 13+, alias didaftarkan di `bootstrap/app.php` melalui `withMiddleware(...)`.
4. Tambahkan import `use App\Http\Middleware\AdminMiddleware;` sebelum memakai class tersebut.
5. Alias tidak melindungi URL dengan sendirinya. Alias harus dipasang pada route.
6. Jangan mengikuti cara tutorial lama yang memakai `app/Http/Kernel.php`.

Tahap berikutnya akan memasang middleware `auth` dan `admin` pada kelompok route dashboard admin, CRUD Product, daftar order, dan daftar user.

---

**Apakah kamu ingin lanjut ke tahap 10: melindungi dashboard admin dan CRUD Product dengan middleware?**
