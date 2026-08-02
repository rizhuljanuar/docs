# Tahap 12 — Ringkasan dan Checklist Aman Middleware Admin

> Penutup materi 18: melindungi dashboard admin dan CRUD Product agar hanya user dengan role `admin` yang dapat mengelola data penting.

Pada tahap 1, kita mulai dari masalah sederhana:

> User biasa dapat mengetik URL admin di browser dan membuka halaman yang seharusnya hanya untuk pengelola aplikasi.

Sekarang kita sudah memiliki solusi bertahap: role disimpan di database, middleware memeriksa role, alias didaftarkan pada Laravel 13+, lalu route admin dilindungi dengan `auth` dan `admin`.

## Gambaran besar solusi

Bayangkan kembali aplikasi CRUD Product sebagai toko.

```text
User membuka pintu area admin
        ↓
Middleware auth memeriksa kartu masuk, apakah sudah login?
        ↓
Middleware admin memeriksa kartu tugas, apakah role-nya admin?
        ↓
Jika dua pemeriksaan lolos
        ↓
Dashboard atau CRUD Product dapat dijalankan
```

Jika user belum login, Laravel mengarahkannya ke login.

Jika user sudah login tetapi role-nya `user`, Laravel menghentikan request dengan 403 Forbidden.

Jika user memiliki role `admin`, Laravel meneruskan request ke controller.

## Ringkasan 12 tahap

| Tahap | Yang dipelajari |
| --- | --- |
| 1 | Halaman admin adalah area pengelolaan yang tidak boleh terbuka untuk semua user. |
| 2 | Request dari browser melewati route dan middleware sebelum sampai ke controller. |
| 3 | Authentication memeriksa login, sedangkan authorization memeriksa izin berdasarkan role. |
| 4 | Kolom `role` ditambahkan ke tabel `users` melalui migration. |
| 5 | User baru harus memiliki role default `user` dan tidak boleh memilih `admin` saat register. |
| 6 | Satu akun local dipilih dengan sengaja untuk menjadi admin latihan. |
| 7 | File `AdminMiddleware` dibuat dengan Artisan. |
| 8 | Middleware memeriksa login dan role `admin`, lalu menolak user biasa dengan 403. |
| 9 | Alias `admin` didaftarkan di `bootstrap/app.php` untuk Laravel 13+. |
| 10 | Route dashboard, CRUD Product, order, dan user dikelompokkan dengan middleware `auth` dan `admin`. |
| 11 | Akses diuji sebagai guest, user biasa, dan admin. Halaman error 403 yang ramah dapat dibuat. |
| 12 | Menyatukan alur, kode inti, checklist, dan kesalahan umum. |

## File dan tanggung jawabnya

Saat fitur middleware admin selesai, setiap bagian berada di tempat yang jelas:

| File atau lokasi | Tanggung jawab |
| --- | --- |
| `database/migrations/...add_role_to_users_table.php` | Menambahkan kolom `role` dengan default `user`. |
| `app/Models/User.php` | Model untuk membaca data user, termasuk `role`. Jangan membuka mass assignment `role` untuk form register umum. |
| `app/Http/Middleware/AdminMiddleware.php` | Memeriksa apakah user yang sedang login memiliki role `admin`. |
| `bootstrap/app.php` | Mendaftarkan alias middleware `admin` pada Laravel 13+. |
| `routes/web.php` | Memasang `auth` dan `admin` pada route area admin. |
| `resources/views/errors/403.blade.php` | Menampilkan pesan yang ramah ketika user login tidak mempunyai izin. |

Dengan pembagian ini, kita tidak mencampur tugas database, pemeriksaan akses, konfigurasi, route, dan tampilan pada satu file.

## Kode inti yang perlu diingat

### 1. Migration kolom role

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('role')->default('user');
});
```

Maknanya:

- setiap user mempunyai kolom `role`,
- nilai aman awalnya adalah `user`,
- tidak ada akun yang otomatis menjadi admin.

### 2. Pemeriksaan pada `AdminMiddleware`

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

Maknanya:

```text
Tidak ada user login → arahkan ke login
Role bukan admin → hentikan dengan 403
Role admin → teruskan request
```

### 3. Alias pada Laravel 13+

Di `bootstrap/app.php`:

```php
use App\Http\Middleware\AdminMiddleware;

// ...

->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'admin' => AdminMiddleware::class,
    ]);
})
```

Maknanya:

```text
Nama admin pada route
        ↓
Menjalankan AdminMiddleware
```

Pada Laravel 13+, konfigurasi ini berada di `bootstrap/app.php`. Jangan memakai tutorial lama yang menyuruh menambahkan alias ke `app/Http/Kernel.php`.

### 4. Route group area admin

Di `routes/web.php`:

```php
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');

        Route::resource('/products', ProductController::class);
    });
```

Maknanya:

- semua route di dalam group harus melewati `auth` dan `admin`,
- URL memakai awalan `/admin`,
- nama route memakai awalan `admin.`,
- seluruh CRUD Product dilindungi, termasuk tambah, simpan, edit, update, dan hapus.

Tambahkan route order dan user ke group ini hanya ketika controllernya sudah ada.

## Alur lengkap request admin

Gunakan alur ini ketika ingin mengingat hubungan semua bagian:

```text
User membuka /admin/products
        ↓
routes/web.php menemukan route group admin
        ↓
auth memeriksa apakah user sudah login
        ├── Belum login → redirect ke login
        └── Sudah login
              ↓
        admin memanggil AdminMiddleware
              ↓
        role === admin?
              ├── Tidak → 403 Forbidden
              └── Ya → ProductController dijalankan
                            ↓
                         Blade menampilkan halaman
```

Hubungan alias Laravel 13+ di tengah alur tersebut adalah:

```text
routes/web.php memakai 'admin'
        ↓
bootstrap/app.php memetakan 'admin' ke AdminMiddleware::class
        ↓
AdminMiddleware.php menjalankan pemeriksaan role
```

## Checklist implementasi

Gunakan checklist ini setelah menyelesaikan kode.

### Database dan role

- [ ] Saya membuat migration baru untuk menambahkan kolom `role` pada tabel `users`.
- [ ] Migration memakai nilai default aman: `default('user')`.
- [ ] Saya menjalankan `php artisan migrate` pada database local yang benar.
- [ ] Saya memeriksa `.env` dan database aktif sebelum menjalankan migration.
- [ ] User lama dan user baru memiliki role `user` jika belum sengaja diubah.
- [ ] Hanya satu atau beberapa akun yang memang dipilih diberi role `admin`.

### Register dan data user

- [ ] Form register tidak menyediakan pilihan role `admin`.
- [ ] Proses register tidak menerima role langsung dari input user.
- [ ] Saya tidak memakai `User::create($request->all())` untuk pendaftaran.
- [ ] Saya tidak menambahkan `role` ke `$fillable` hanya agar form register umum bisa mengisinya.
- [ ] Password selalu disimpan dengan hash, bukan teks biasa.

### Middleware

- [ ] File `app/Http/Middleware/AdminMiddleware.php` ada.
- [ ] Namespace file adalah `App\Http\Middleware`.
- [ ] Middleware memeriksa `$request->user()` sebelum membaca role.
- [ ] Middleware membandingkan role dengan `=== 'admin'`.
- [ ] User dengan role selain `admin` menerima respons 403.
- [ ] `return $next($request)` hanya dicapai setelah pemeriksaan admin lolos.

### Konfigurasi Laravel 13+

- [ ] Saya meng-import `AdminMiddleware` di `bootstrap/app.php`.
- [ ] Saya mendaftarkan alias `'admin' => AdminMiddleware::class` di dalam `withMiddleware(...)`.
- [ ] Saya tidak mencoba mendaftarkan alias di `app/Http/Kernel.php`.
- [ ] Jika konfigurasi lama masih terbaca, saya menjalankan `php artisan config:clear` lalu memeriksa ulang.

### Route admin

- [ ] Dashboard admin berada dalam route group `middleware(['auth', 'admin'])`.
- [ ] CRUD Product berada dalam group admin yang sama.
- [ ] Route order dan daftar user, jika sudah ada, juga berada dalam group admin.
- [ ] URL admin memakai prefix `/admin`.
- [ ] Saya memperbarui link, form action, dan redirect ke nama route `admin.products...` bila Product dipindahkan ke group bernama `admin.`.
- [ ] Tidak ada route CRUD lama yang masih terbuka di luar group admin.
- [ ] Saya menjalankan `php artisan route:list --path=admin` dan melihat `auth` serta `admin` pada route terkait.

### Pengujian akses

- [ ] Saya menguji URL admin saat belum login dan browser diarahkan ke login.
- [ ] Saya menguji URL admin saat login sebagai role `user` dan menerima 403.
- [ ] Saya menguji URL admin saat login sebagai role `admin` dan halaman tampil.
- [ ] Saya menguji setidaknya route daftar dan tambah Product.
- [ ] Saya memastikan route edit, update, dan hapus Product juga berada di resource group admin.
- [ ] Saya tidak menganggap tombol yang disembunyikan di Blade sebagai satu-satunya perlindungan.
- [ ] Halaman 403 tidak membocorkan data, route internal, database, atau informasi admin.

## Kesalahan umum dan cara memperbaikinya

| Kesalahan | Dampak | Cara memperbaiki |
| --- | --- | --- |
| Hanya memakai middleware `auth` | Semua user yang login dapat membuka halaman admin | Tambahkan middleware `admin` setelah `auth` |
| Menentukan admin dari nama atau email dalam controller | Akses mudah salah dan kode berulang | Simpan role di tabel `users`, periksa melalui middleware |
| Memberi default role `admin` | Akun baru dapat menjadi admin tanpa sengaja | Gunakan `default('user')` |
| Menampilkan pilihan admin di register | User dapat mencoba memilih akses admin | Jangan kirim atau terima role admin dari form register |
| Memakai `$request->all()` ketika membuat user | Input tak terduga dapat ikut diproses | Pilih dan validasi field yang diperlukan saja |
| Menambahkan `role` ke `$fillable` untuk register umum | Hak akses dapat lebih mudah diisi dari input massal | Biarkan role terlindungi, gunakan default database |
| Menaruh pemeriksaan role di setiap controller | Kode berulang dan beberapa aksi bisa lupa diamankan | Gunakan satu `AdminMiddleware` pada route group |
| Menyembunyikan tombol admin saja | User masih dapat mengetik URL langsung | Lindungi route dengan middleware |
| Mendaftarkan alias pada `Kernel.php` | Cara tidak sesuai struktur Laravel 13+ | Daftarkan pada `bootstrap/app.php` |
| Menaruh resource Product admin dan lama sekaligus | Jalur lama bisa tetap terbuka | Hapus, pindahkan, atau pisahkan route publik dengan jelas |
| Menganggap 403 adalah bug | User biasa tampak seperti error padahal ditolak dengan benar | Pahami 403 sebagai authorization yang berhasil |
| Menguji hanya sebagai admin | Guest atau user biasa mungkin masih bisa masuk | Uji guest, user, dan admin |

## Batas materi ini

Materi ini menggunakan dua role sederhana:

```text
admin
user
```

Ini cukup untuk aplikasi CRUD Product dasar.

Pada aplikasi yang lebih besar, kamu mungkin membutuhkan role lain seperti `staff`, `manager`, atau `customer`, serta izin yang lebih rinci, misalnya user tertentu boleh edit Product tetapi tidak boleh hapus Product.

Untuk kebutuhan seperti itu, Laravel menyediakan konsep authorization yang lebih lanjut seperti Gates dan Policies. Namun jangan buru-buru menggunakannya sebelum kamu benar-benar memahami role sederhana dan middleware admin pada materi ini.

## Hubungan dengan materi sebelumnya

Middleware admin menyatukan beberapa hal yang telah kamu pelajari:

| Materi sebelumnya | Hubungannya dengan middleware admin |
| --- | --- |
| CRUD Product | Menjaga create, read admin, update, dan delete Product agar hanya admin yang dapat mengelola. |
| Validasi form dan error handling | Validasi memeriksa input yang dikirim, middleware memeriksa siapa yang boleh mengirimnya. |
| Flash message | Flash message dapat ditampilkan setelah admin berhasil membuat atau mengubah Product, tetapi user biasa tidak boleh mencapai aksi tersebut. |
| Dashboard admin sederhana | Dashboard menjadi area yang tepat untuk diproteksi oleh middleware admin. |
| Seeder dan factory | Membantu menyiapkan user atau Product dummy pada database local untuk pengujian akses. |
| Konfigurasi environment | Membantu memastikan migration dan akun admin latihan dibuat di database local yang benar, bukan production. |

Kalimat penting untuk diingat:

> **Validation memeriksa apakah data yang dikirim benar. Authentication memeriksa siapa user yang masuk. Authorization memeriksa apakah user tersebut boleh melakukan tindakan. Middleware admin menjalankan pemeriksaan authorization sebelum controller mengelola data.**

## Penutup

Materi **18. Middleware Admin** selesai.

Sekarang halaman dashboard admin, tambah Product, edit Product, hapus Product, daftar order, dan daftar user tidak lagi hanya mengandalkan tombol yang disembunyikan di Blade.

Akses diperiksa pada route sebelum controller dijalankan:

```text
Belum login → login
Login sebagai user → 403 Forbidden
Login sebagai admin → akses diizinkan
```

Dengan kebiasaan menyimpan role secara aman, memberi user baru role default `user`, membuat middleware khusus, mendaftarkan alias pada `bootstrap/app.php` Laravel 13+, memasang middleware pada route group, serta menguji tiga kondisi akses, aplikasi CRUD Product kamu menjadi lebih aman dan lebih rapi.
