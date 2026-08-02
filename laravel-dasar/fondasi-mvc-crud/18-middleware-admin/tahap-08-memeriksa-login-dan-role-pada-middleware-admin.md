# Tahap 8 — Memeriksa Login dan Role pada Middleware Admin

> Fokus: mengisi `AdminMiddleware` agar request tanpa login diarahkan ke login, user dengan role `user` menerima respons 403, dan hanya role `admin` yang dapat diteruskan.

Pada tahap 7, kita membuat file berikut:

```text
app/Http/Middleware/AdminMiddleware.php
```

Middleware tersebut masih selalu meneruskan request dengan:

```php
return $next($request);
```

Sekarang kita akan menambahkan aturan penjaga yang sebenarnya.

## Tiga kemungkinan saat seseorang membuka halaman admin

Misalnya seseorang mencoba membuka:

```text
/admin/dashboard
```

Middleware harus membedakan tiga keadaan berikut:

| Keadaan | Contoh | Hasil yang benar |
| --- | --- | --- |
| Belum login | Pengunjung tidak punya session login | Arahkan ke halaman login |
| Sudah login sebagai `user` | Andi | Tolak akses dengan 403 Forbidden |
| Sudah login sebagai `admin` | Siti Admin | Teruskan ke controller dashboard |

Alur lengkapnya:

```text
Request menuju halaman admin
        ↓
Apakah ada user yang sedang login?
        ├── Tidak → arahkan ke login
        └── Ya
              ↓
        Apakah role user adalah admin?
              ├── Tidak → 403 Forbidden
              └── Ya → lanjutkan request
```

## Mengambil user dari request

Di dalam middleware, kita dapat mengambil user yang sedang login dengan:

```php
$request->user()
```

Artinya:

> “Ambil user yang dikenali Laravel untuk request ini.”

Jika Siti Admin sedang login, `$request->user()` secara konsep berisi data seperti ini:

```text
id: 2
name: Siti Admin
email: siti.admin@example.test
role: admin
```

Kemudian nilai role dapat dibaca dengan:

```php
$request->user()->role
```

Pada tahap 3, kita juga melihat bentuk lain:

```php
auth()->user()->role
```

Keduanya dapat dipakai saat user sudah login. Dalam middleware, kita memakai `$request->user()` karena user tersebut langsung berkaitan dengan request yang sedang diperiksa.

## Langkah 1, periksa apakah user sudah login

Buka file:

```text
app/Http/Middleware/AdminMiddleware.php
```

Ubah method `handle()` menjadi seperti ini:

```php
public function handle(Request $request, Closure $next): Response
{
    if (! $request->user()) {
        return redirect()->route('login');
    }

    return $next($request);
}
```

Penjelasan setiap bagian:

### `if (! $request->user())`

```php
if (! $request->user()) {
```

- `$request->user()` mencoba mengambil user yang sedang login.
- Jika belum ada user login, hasilnya kosong, atau `null`.
- Tanda `!` berarti **tidak**.
- Jadi, kondisi ini berarti: “Jika tidak ada user yang login.”

### `return redirect()->route('login');`

```php
return redirect()->route('login');
```

- `redirect()` membuat response untuk memindahkan browser ke halaman lain.
- `->route('login')` memakai route bernama `login`.
- Pada aplikasi yang sudah memiliki fitur login atau Laravel starter kit, route login umumnya bernama `login`.

Artinya:

> “User belum login, jadi hentikan request ke halaman admin dan pindahkan ke halaman login.”

`return` penting karena middleware harus berhenti di sini. Request tidak boleh diteruskan ke controller dashboard.

### `return $next($request);`

Jika user sudah login, kondisi `if` dilewati. Lalu request diteruskan ke langkah berikutnya:

```php
return $next($request);
```

Untuk sementara, user biasa dan admin masih sama-sama dapat lewat. Langkah berikutnya di tahap ini akan membedakan role mereka.

## Hubungan dengan middleware `auth`

Nantinya route admin tetap akan memakai middleware bawaan Laravel:

```text
auth
```

Middleware `auth` adalah pemeriksaan utama untuk login. Ia akan mengarahkan user yang belum login ke halaman login sesuai konfigurasi aplikasi.

Jadi, setelah semua tahap selesai, route admin akan memakai urutan:

```text
auth
    ↓
admin
```

Pada urutan tersebut, `auth` biasanya sudah menghentikan user yang belum login sebelum `AdminMiddleware` bekerja.

Lalu, mengapa `AdminMiddleware` masih mengecek `$request->user()`?

Karena pemeriksaan ini membuat middleware aman dan mudah dipahami bila suatu saat dipasang tanpa `auth`. Namun **jangan menganggap pemeriksaan ini sebagai pengganti middleware `auth`**. Kita tetap memasang keduanya pada route agar tujuan masing-masing jelas:

| Middleware | Tugas utama |
| --- | --- |
| `auth` | Memastikan user sudah login |
| `admin` | Memastikan user yang login memiliki role `admin` |

## Langkah 2, tolak user yang bukan admin

Setelah pemeriksaan login, tambahkan pemeriksaan role. Isi lengkap `handle()` menjadi:

```php
public function handle(Request $request, Closure $next): Response
{
    if (! $request->user()) {
        return redirect()->route('login');
    }

    abort_unless($request->user()->role === 'admin', 403);

    return $next($request);
}
```

Baris baru yang penting adalah:

```php
abort_unless($request->user()->role === 'admin', 403);
```

Mari pahami dari kiri ke kanan:

- `abort_unless(...)` berarti “hentikan request jika kondisi berikut **tidak** benar.”
- `$request->user()->role` mengambil role dari user yang sudah login.
- `=== 'admin'` memeriksa apakah role itu tepat bernilai `admin`.
- `403` adalah kode status HTTP untuk **Forbidden**, artinya aplikasi mengenali user tetapi user tersebut tidak punya izin.

Jadi arti seluruh barisnya adalah:

> “Jika role user bukan `admin`, hentikan request dan kirim respons 403 Forbidden.”

Jika role adalah `admin`, request tidak dihentikan. Laravel melanjutkan ke baris berikutnya:

```php
return $next($request);
```

## Isi lengkap `AdminMiddleware.php`

Setelah tahap ini, file middleware menjadi seperti berikut:

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
        if (! $request->user()) {
            return redirect()->route('login');
        }

        abort_unless($request->user()->role === 'admin', 403);

        return $next($request);
    }
}
```

## Cara kerja kode untuk setiap user

### Pengunjung belum login

```text
$request->user()
    ↓
Tidak ada user
    ↓
redirect()->route('login')
    ↓
Browser pindah ke login
```

### Andi dengan role `user`

```text
$request->user()
    ↓
Ada user bernama Andi
    ↓
Andi role: user
    ↓
role === admin? Tidak
    ↓
abort_unless(..., 403)
    ↓
Laravel mengirim 403 Forbidden
    ↓
Controller tidak dijalankan
```

### Siti dengan role `admin`

```text
$request->user()
    ↓
Ada user bernama Siti Admin
    ↓
Siti role: admin
    ↓
role === admin? Ya
    ↓
Tidak dihentikan
    ↓
$next($request)
    ↓
Controller dijalankan
```

## Apa arti 403 Forbidden?

Kode **403 Forbidden** berarti:

> “Kamu sudah dikenal oleh aplikasi, tetapi kamu tidak memiliki izin untuk membuka halaman ini.”

Ini berbeda dengan user yang belum login:

| Kondisi | Respons yang tepat |
| --- | --- |
| Belum login | Arahkan ke login |
| Sudah login, tetapi bukan admin | 403 Forbidden |
| Sudah login sebagai admin | Izinkan akses |

Kita memakai 403 karena user biasa seperti Andi memang mempunyai akun yang sah, hanya saja ia tidak boleh mengelola dashboard, Product, order, atau user.

Pada tahap 11, kita akan membahas pengalaman user saat menerima 403 dan cara membuat halaman error yang lebih ramah.

## Mengapa `===` digunakan, bukan `=`?

Perhatikan kode ini:

```php
$request->user()->role === 'admin'
```

Tiga tanda sama dengan, `===`, digunakan untuk membandingkan nilai secara tepat.

Jangan menulis satu tanda sama dengan seperti ini:

```php
$request->user()->role = 'admin';
```

Satu tanda `=` berarti **mengubah atau mengisi nilai**, bukan memeriksa nilai. Kesalahan itu dapat mengubah role user dan sangat berbahaya.

Gunakan `=== 'admin'` untuk pemeriksaan role.

## Jika route `login` belum ada

Contoh ini memakai:

```php
redirect()->route('login')
```

Karena aplikasi dengan login Laravel biasanya memiliki route bernama `login`.

Jika project kamu belum mempunyai sistem login atau route bernama `login`, jangan mengganti kode secara acak. Kamu perlu menyiapkan authentication terlebih dahulu atau menyesuaikan tujuan redirect dengan route login yang benar di projectmu.

Kamu dapat memeriksa route dengan:

```bash
php artisan route:list
```

Cari baris dengan nama route `login`. Perintah ini hanya menampilkan daftar route, tidak mengubah aplikasi.

Untuk materi middleware admin ini, kita mengasumsikan aplikasi sudah mempunyai login, sesuai pembahasan tahap 1 sampai 3.

## Jangan mendaftarkan atau memasang middleware dulu

Setelah menulis kode di `AdminMiddleware.php`, kita belum dapat memakai nama `admin` di route. Laravel perlu diberi tahu bahwa alias `admin` menunjuk ke class `AdminMiddleware`.

Tahap berikutnya akan mendaftarkan alias tersebut di lokasi yang benar untuk Laravel 13+:

```text
bootstrap/app.php
```

Baru setelah alias terdaftar, kita dapat memasangnya pada kelompok route dashboard dan CRUD Product.

## Yang perlu diingat pada tahap ini

1. `$request->user()` mengambil user yang sedang login untuk request tersebut.
2. User tanpa login diarahkan ke route `login`.
3. `abort_unless($request->user()->role === 'admin', 403)` menolak user yang bukan admin.
4. Kode 403 berarti user sudah login tetapi tidak mempunyai izin.
5. `return $next($request)` hanya dijalankan untuk user dengan role `admin`.
6. Route admin nanti tetap memakai `auth` sebelum `admin`.
7. Jangan menggunakan `=` saat memeriksa role, gunakan `===`.

Tahap berikutnya akan mendaftarkan alias `admin` di `bootstrap/app.php` agar Laravel 13+ dapat mengenali dan menggunakan middleware ini pada route.

---

**Apakah kamu ingin lanjut ke tahap 9: mendaftarkan alias middleware admin di `bootstrap/app.php`?**
