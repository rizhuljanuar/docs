# Tahap 7 — Membuat File Middleware Admin di Laravel 13+

> Fokus: membuat tempat khusus untuk pemeriksaan admin. Pada tahap ini kita membuat file middleware dan memahami bentuk dasarnya. Pemeriksaan role akan ditulis pada tahap berikutnya.

Pada tahap 6, kita sudah menyiapkan dua kondisi untuk latihan:

```text
Andi → role user
Siti Admin → role admin
```

Sekarang kita akan membuat penjaga yang nantinya membedakan keduanya saat mencoba membuka dashboard admin atau halaman CRUD Product.

## Mengapa membuat file middleware khusus?

Kita sebenarnya bisa saja menulis pemeriksaan role di dalam setiap controller.

Contohnya, kita harus mengulang pemeriksaan pada:

- `AdminDashboardController`,
- method `create()` Product,
- method `store()` Product,
- method `edit()` Product,
- method `update()` Product,
- method `destroy()` Product,
- halaman daftar order,
- halaman daftar user.

Cara itu mudah terlupa dan membuat kode yang sama muncul di banyak tempat.

Middleware admin membuat satu penjaga khusus:

```text
Semua request menuju area admin
        ↓
AdminMiddleware memeriksa request
        ↓
Lolos → controller boleh bekerja
Ditolak → controller tidak dijalankan
```

Jadi, pemeriksaan akses berada di satu tempat yang sesuai, bukan tersebar di banyak controller.

## Lokasi file middleware

Middleware buatan aplikasi Laravel disimpan di folder:

```text
app/Http/Middleware/
```

Kita akan membuat file dengan nama:

```text
AdminMiddleware.php
```

Lokasi lengkapnya nanti:

```text
app/Http/Middleware/AdminMiddleware.php
```

Nama `AdminMiddleware` dipilih karena tugasnya jelas, yaitu memeriksa akses admin.

## Langkah 1, buat middleware dengan Artisan

Dari **root project Laravel**, jalankan perintah berikut:

```bash
php artisan make:middleware AdminMiddleware
```

Penjelasan setiap bagian:

- `php artisan` menjalankan perintah bawaan Laravel.
- `make:middleware` meminta Laravel membuat file middleware baru.
- `AdminMiddleware` adalah nama class middleware yang akan dibuat.

Setelah perintah berhasil, Laravel membuat file:

```text
app/Http/Middleware/AdminMiddleware.php
```

Jangan membuat file secara manual jika kamu masih belajar. Perintah Artisan membantu Laravel membuat namespace dan struktur awal yang benar.

## Isi awal middleware

Buka file `app/Http/Middleware/AdminMiddleware.php`. Struktur yang dibuat Laravel 13+ umumnya seperti ini:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class AdminMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }
}
```

Mari pahami bagian-bagiannya secara pelan-pelan.

### `namespace App\Http\Middleware;`

```php
namespace App\Http\Middleware;
```

Baris ini memberi tahu PHP bahwa class ini berada di kelompok middleware aplikasi.

Lokasi file dan namespace harus sesuai:

```text
app/Http/Middleware/AdminMiddleware.php
        ↓
namespace App\Http\Middleware
```

Jangan mengubah namespace jika tidak ada kebutuhan khusus.

### `use Closure;`

```php
use Closure;
```

`Closure` dipakai untuk menerima `$next`, yaitu langkah berikutnya dalam perjalanan request.

Untuk sekarang, ingat saja bahwa `$next` adalah jalan yang membawa request ke middleware berikutnya atau ke controller.

### `use Illuminate\Http\Request;`

```php
use Illuminate\Http\Request;
```

Class `Request` menyimpan informasi request yang datang dari browser, seperti URL yang dibuka dan data user pada request tersebut.

Di tahap berikutnya, kita dapat memakai `$request` untuk memeriksa user yang sedang login.

### `use Symfony\Component\HttpFoundation\Response;`

```php
use Symfony\Component\HttpFoundation\Response;
```

`Response` adalah hasil yang Laravel kirim kembali ke browser, misalnya halaman Blade, redirect, atau pesan akses ditolak.

### `class AdminMiddleware`

```php
class AdminMiddleware
{
    // ...
}
```

Ini adalah class penjaga yang baru kita buat. Nanti nama class ini akan didaftarkan dengan alias `admin`, lalu dipasang pada route admin.

### Method `handle()`

```php
public function handle(Request $request, Closure $next): Response
{
    return $next($request);
}
```

Setiap middleware mempunyai method utama bernama `handle()`.

Laravel memanggil method ini ketika request melewati middleware.

Parameter pada method ini adalah:

| Bagian | Arti sederhana |
| --- | --- |
| `Request $request` | Request dari browser yang sedang diperiksa |
| `Closure $next` | Perintah untuk meneruskan request ke langkah berikutnya |
| `: Response` | Menandakan hasil akhirnya adalah response untuk browser |

Baris berikut adalah bagian paling penting dari bentuk awal middleware:

```php
return $next($request);
```

Artinya:

> “Saya belum menolak request ini. Lanjutkan request ke middleware berikutnya atau controller.”

Saat ini, `AdminMiddleware` belum memeriksa role apa pun. Jadi, jika middleware ini dipasang sekarang, ia masih mengizinkan siapa pun untuk lewat.

Itu normal. Kita baru membuat pintu dan tempat penjaga berdiri. Pada tahap 8, kita akan memberi penjaga aturan untuk memeriksa role `admin`.

## Analogi `$next($request)`

Bayangkan request adalah pengunjung yang membawa surat.

```text
Pengunjung datang ke penjaga
        ↓
Penjaga membaca aturan
        ↓
Jika boleh → penjaga meneruskan surat ke ruangan berikutnya
Jika tidak boleh → penjaga menghentikan surat
```

Pada bentuk awal ini, penjaga selalu meneruskan surat:

```php
return $next($request);
```

Nantinya bentuknya menjadi konsep seperti ini:

```text
Jika role admin
    teruskan dengan $next($request)

Jika bukan admin
    hentikan dan kirim response akses ditolak
```

## Jangan memasang middleware pada route dulu

Walaupun file middleware sudah dibuat, kita belum perlu memasangnya pada route admin.

Masih ada dua langkah penting:

1. Menulis pemeriksaan role di dalam `handle()`.
2. Mendaftarkan alias middleware `admin` pada `bootstrap/app.php` Laravel 13+.

Jika middleware dipasang sebelum pemeriksaannya benar, hasilnya dapat membingungkan. Kita akan menyelesaikannya sedikit demi sedikit.

## Jangan memakai `app/Http/Kernel.php` pada Laravel 13+

Kamu mungkin menemukan tutorial lama yang menyuruh menambahkan middleware ke:

```text
app/Http/Kernel.php
```

Untuk Laravel 13+, jangan memakai cara tersebut. Pengaturan alias middleware dilakukan pada:

```text
bootstrap/app.php
```

File `AdminMiddleware.php` tetap berada di:

```text
app/Http/Middleware/AdminMiddleware.php
```

Jadi, bedakan dua hal ini:

| Yang dilakukan | Lokasi Laravel 13+ |
| --- | --- |
| Menulis aturan pemeriksaan admin | `app/Http/Middleware/AdminMiddleware.php` |
| Mendaftarkan nama singkat middleware | `bootstrap/app.php` |
| Memasang middleware pada URL tertentu | `routes/web.php` |

## Cara memeriksa file berhasil dibuat

Setelah menjalankan perintah Artisan, pastikan file ini ada:

```text
app/Http/Middleware/AdminMiddleware.php
```

Pastikan class dan nama file juga sama:

```text
AdminMiddleware.php
        ↓
class AdminMiddleware
```

Jika muncul error `Class ... not found` pada tahap nanti, periksa kembali penulisan nama file, class, dan namespace ini.

## Yang perlu diingat pada tahap ini

1. `php artisan make:middleware AdminMiddleware` membuat file middleware khusus.
2. File middleware berada di `app/Http/Middleware/AdminMiddleware.php`.
3. Method `handle()` adalah tempat middleware memeriksa setiap request.
4. `return $next($request)` meneruskan request ke middleware selanjutnya atau controller.
5. Middleware baru ini belum aman untuk dipasang karena belum memeriksa role.
6. Laravel 13+ mendaftarkan alias middleware nanti melalui `bootstrap/app.php`, bukan `app/Http/Kernel.php`.

Tahap berikutnya akan mengisi `handle()` agar hanya user dengan role `admin` yang boleh meneruskan request.

---

**Apakah kamu ingin lanjut ke tahap 8: menulis pemeriksaan login dan role pada middleware admin?**
