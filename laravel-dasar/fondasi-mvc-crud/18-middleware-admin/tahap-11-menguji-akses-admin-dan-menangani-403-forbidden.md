# Tahap 11 — Menguji Akses Admin dan Menangani 403 Forbidden

> Fokus: membuktikan middleware benar-benar melindungi halaman admin untuk tiga keadaan, belum login, role `user`, dan role `admin`. Kita juga membuat halaman 403 yang lebih mudah dipahami.

Pada tahap 10, route area admin sudah memakai dua middleware:

```php
Route::middleware(['auth', 'admin'])
```

Kode terlihat benar belum cukup. Kita harus menguji hasilnya dari sudut pandang user.

## Mengapa middleware harus diuji?

Bayangkan kamu sudah memasang kunci pada pintu ruang pengelola toko. Kamu tetap perlu mencoba tiga hal:

1. Apakah orang tanpa kartu akses benar-benar tidak bisa masuk?
2. Apakah pelanggan yang punya kartu gedung, tetapi bukan pengelola, tetap ditolak?
3. Apakah pengelola yang sah bisa masuk?

Middleware admin juga harus lulus tiga pengujian tersebut.

```text
Belum login → menuju login
User biasa → 403 Forbidden
Admin → halaman admin tampil
```

Jika salah satu hasil berbeda, jangan menganggap perlindungan sudah selesai. Periksa kembali migration role, akun latihan, `AdminMiddleware`, alias `admin`, dan route group.

## Sebelum menguji

Pastikan semua bagian tahap sebelumnya sudah ada:

| Bagian | Yang perlu diperiksa |
| --- | --- |
| Database | Tabel `users` memiliki kolom `role` |
| Akun user biasa | Ada satu akun dengan role `user` |
| Akun admin latihan | Ada satu akun dengan role `admin` |
| Middleware | `app/Http/Middleware/AdminMiddleware.php` memeriksa role `admin` |
| Alias | `bootstrap/app.php` mendaftarkan `'admin' => AdminMiddleware::class` |
| Route | URL admin memakai `auth` dan `admin` |

Lalu periksa route admin dari root project Laravel:

```bash
php artisan route:list --path=admin
```

Pastikan kolom Middleware menunjukkan `auth` dan `admin` pada route dashboard atau Product yang benar-benar ada di project kamu.

Contoh URL yang bisa diuji:

```text
/admin/dashboard
/admin/products
/admin/products/create
```

Gunakan URL yang controllernya sudah tersedia. Jika belum membuat dashboard, uji `/admin/products` terlebih dahulu.

## Pengujian 1, pengunjung belum login

### Langkah pengujian

1. Logout dari aplikasi jika sebelumnya sedang login.
2. Buka browser dalam mode incognito/private, atau pastikan session login sudah keluar.
3. Buka salah satu URL admin, misalnya:

```text
/admin/products
```

### Hasil yang diharapkan

Browser harus diarahkan ke halaman login, bukan menampilkan daftar Product admin.

Alurnya:

```text
Pengunjung belum login
        ↓
Membuka /admin/products
        ↓
Middleware auth menolak request
        ↓
Browser diarahkan ke login
        ↓
ProductController@index tidak dijalankan
```

Jika daftar Product tetap terlihat tanpa login, berarti route tersebut kemungkinan belum berada di group middleware `auth` dan `admin`, atau masih ada route lama tanpa perlindungan.

## Pengujian 2, user biasa mencoba URL admin

Sekarang login menggunakan akun dengan role `user`, misalnya Andi.

### Langkah pengujian

1. Login sebagai user biasa.
2. Ketik URL admin langsung di address bar browser:

```text
/admin/products/create
```

3. Coba juga URL aksi lain yang tersedia, misalnya:

```text
/admin/products
/admin/dashboard
```

### Hasil yang diharapkan

Laravel harus menampilkan respons **403 Forbidden**.

Alurnya:

```text
Andi sudah login sebagai user
        ↓
Membuka /admin/products/create
        ↓
Middleware auth: lolos, karena Andi sudah login
        ↓
Middleware admin: role Andi bukan admin
        ↓
abort_unless(..., 403)
        ↓
Laravel memberi respons 403
        ↓
ProductController@create tidak dijalankan
```

Penting: jangan hanya menguji halaman daftar Product. User biasa juga harus gagal saat mencoba membuka form tambah, edit, mengirim request simpan, update, atau hapus Product.

Ini membuktikan bahwa middleware menjaga route, bukan hanya menyembunyikan tombol pada tampilan.

## Pengujian 3, admin membuka halaman admin

Logout, lalu login sebagai akun admin latihan yang dibuat pada tahap 6, misalnya Siti Admin.

### Langkah pengujian

1. Login sebagai Siti Admin dengan role `admin`.
2. Buka:

```text
/admin/products
```

3. Jika halaman dan fitur sudah tersedia, coba juga:

```text
/admin/products/create
/admin/products/{id}/edit
/admin/dashboard
```

Ganti `{id}` dengan ID Product yang benar-benar ada di database latihan.

### Hasil yang diharapkan

Halaman admin harus terbuka dan controller dapat berjalan.

```text
Siti sudah login sebagai admin
        ↓
Membuka /admin/products
        ↓
Middleware auth: lolos
        ↓
Middleware admin: role Siti adalah admin
        ↓
return $next($request)
        ↓
ProductController@index dijalankan
        ↓
Daftar Product admin tampil
```

Jika admin juga menerima 403, periksa role akun melalui Tinker:

```bash
php artisan tinker
```

Kemudian jalankan, dengan email akun admin local kamu sendiri:

```php
App\Models\User::where('email', 'siti.admin@example.test')
    ->firstOrFail()
    ->only(['name', 'email', 'role']);
```

Pastikan hasil `role` adalah:

```text
admin
```

Keluar dari Tinker dengan:

```text
exit
```

## Ringkasan hasil pengujian

Catat hasilmu dengan tabel berikut:

| Kondisi | URL diuji | Hasil yang harus terjadi |
| --- | --- | --- |
| Belum login | `/admin/products` | Dialihkan ke login |
| Login sebagai `user` | `/admin/products/create` | 403 Forbidden |
| Login sebagai `admin` | `/admin/products/create` | Form tambah Product tampil |
| Login sebagai `user` | URL edit atau hapus Product | 403 Forbidden |
| Login sebagai `admin` | URL edit atau hapus Product | Akses diizinkan sesuai route |

Jika semua hasil sesuai, middleware admin telah bekerja untuk skenario dasar.

## Membuat halaman 403 yang lebih ramah

Secara default, Laravel dapat menampilkan halaman error sederhana saat menerima 403. Untuk aplikasi CRUD Product, kita dapat menyiapkan halaman yang memberi penjelasan lebih baik kepada user biasa.

Buat file baru:

```text
resources/views/errors/403.blade.php
```

Isi file tersebut dengan contoh sederhana ini:

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Akses Ditolak</title>
</head>
<body>
    <h1>403 - Akses Ditolak</h1>

    <p>Kamu tidak memiliki izin untuk membuka halaman admin ini.</p>

    <a href="{{ url('/') }}">Kembali ke halaman utama</a>
</body>
</html>
```

Penjelasan setiap bagian:

- `resources/views/errors/` adalah folder untuk halaman error Blade.
- Nama file `403.blade.php` dipakai Laravel saat aplikasi menghasilkan respons HTTP 403.
- `<h1>` menampilkan judul error yang jelas.
- `<p>` menjelaskan bahwa masalahnya adalah izin akses, bukan kesalahan password.
- `url('/')` membuat link kembali ke halaman utama aplikasi.

Setelah file ini dibuat, coba lagi membuka URL admin sebagai user biasa. Kamu seharusnya melihat halaman **Akses Ditolak** yang kamu buat.

> Halaman 403 menjelaskan penolakan akses. Halaman ini tidak boleh membocorkan data admin, daftar route rahasia, atau detail teknis aplikasi.

## Alternatif redirect untuk user biasa

Pada tahap 8, kita memilih respons yang tegas dan standar:

```php
abort_unless($request->user()->role === 'admin', 403);
```

Untuk materi ini, pertahankan 403 karena ia jelas menyatakan bahwa user sudah login tetapi tidak punya izin.

Ada aplikasi yang memilih mengarahkan user biasa ke halaman lain. Itu dapat dipakai pada kebutuhan tertentu, tetapi jangan mengubah ke redirect hanya untuk menyembunyikan masalah. Middleware harus tetap menghentikan request sebelum controller admin berjalan.

Jika kamu kelak memilih redirect, pastikan tidak membuat redirect berulang, misalnya user diarahkan ke halaman yang juga membutuhkan middleware admin.

## Menguji aksi simpan, update, dan hapus

Mengakses halaman dengan GET belum cukup. Untuk CRUD Product, aksi yang mengubah data juga harus melalui middleware route group.

Dengan `Route::resource()` di dalam group admin, route berikut ikut terlindungi:

| Aksi | HTTP method | Contoh URL | User biasa |
| --- | --- | --- | --- |
| Simpan Product | POST | `/admin/products` | 403 |
| Update Product | PUT/PATCH | `/admin/products/{product}` | 403 |
| Hapus Product | DELETE | `/admin/products/{product}` | 403 |

Jangan mencoba membuat request hapus manual pada data penting. Saat belajar, pakai Product dummy di database local jika ingin menguji tombol hapus sebagai admin.

Untuk user biasa, cukup pastikan tombol/URL admin ditolak. Jangan mencari cara untuk melewati middleware.

## Pilihan lanjutan, automated test Laravel

Pengujian browser di atas adalah langkah paling mudah untuk pemula. Saat project sudah memiliki folder `tests/Feature`, kamu dapat menambahkan pengujian otomatis agar aturan admin tidak mudah rusak ketika kode berubah.

Contoh konsep test untuk route admin yang sudah ada:

```php
use App\Models\User;

it('forbids a normal user from opening the admin products page', function () {
    $user = User::factory()->create(['role' => 'user']);

    $this->actingAs($user)
        ->get('/admin/products')
        ->assertForbidden();
});
```

Penjelasan:

- `User::factory()->create(['role' => 'user'])` membuat user dummy dengan role `user` untuk test.
- `$this->actingAs($user)` berpura-pura login sebagai user tersebut, tanpa membuka browser.
- `->get('/admin/products')` mengirim request GET ke halaman admin.
- `->assertForbidden()` memastikan responsnya benar-benar 403.

Contoh admin yang diizinkan, bila route dan data pendukungnya siap:

```php
it('allows an admin to open the admin products page', function () {
    $admin = User::factory()->create(['role' => 'admin']);

    $this->actingAs($admin)
        ->get('/admin/products')
        ->assertOk();
});
```

Jangan menambahkan contoh test ini sebelum test suite project dan route `/admin/products` sudah siap. Fokus utama tahap ini adalah memahami hasil tiga kondisi akses melalui browser.

## Kesalahan umum saat menguji

### 1. Menguji dengan session admin yang masih aktif

Jika browser masih login sebagai admin, hasil pengujian user biasa atau guest menjadi salah. Logout atau gunakan incognito/private window.

### 2. Memeriksa hanya menu, bukan URL langsung

Tombol admin dapat disembunyikan, tetapi user biasa tetap harus diuji dengan mengetik URL admin langsung.

### 3. Menganggap 403 adalah error aplikasi yang harus dihilangkan

Untuk user biasa pada area admin, 403 adalah hasil yang benar. Ia berarti authorization bekerja.

### 4. Tidak menguji route create, edit, dan hapus

Melindungi daftar Product saja tidak cukup. Resource CRUD harus seluruhnya berada di route group admin.

### 5. Menampilkan detail sensitif pada halaman 403

Jangan menulis data akun admin, path server, query database, atau informasi rahasia pada halaman error.

## Yang perlu diingat pada tahap ini

1. Uji middleware dengan tiga kondisi, guest, `user`, dan `admin`.
2. Guest harus diarahkan ke login.
3. User biasa harus menerima 403 saat mencoba URL admin langsung.
4. Admin harus dapat membuka dashboard dan CRUD Product sesuai route yang tersedia.
5. Halaman `resources/views/errors/403.blade.php` dapat membuat pesan penolakan lebih ramah.
6. 403 adalah hasil authorization yang benar untuk user login tanpa hak admin.
7. Route yang mengubah Product juga wajib diuji dan dilindungi.

Tahap berikutnya merangkum seluruh materi Middleware Admin, checklist keamanan, serta kesalahan yang paling perlu dihindari.

---

**Apakah kamu ingin lanjut ke tahap 12: ringkasan dan checklist aman middleware admin?**
