# Tahap 2 — Memahami Cara Kerja Middleware Admin di Laravel

> Fokus: memahami perjalanan request dari browser sampai ke route, middleware, controller, dan halaman. Pada tahap ini kita belum membuat middleware admin sendiri.

Pada tahap 1, kita belajar bahwa middleware admin adalah penjaga pintu menuju area admin.

Sekarang mari pahami **kapan** penjaga itu bekerja. Pemahaman ini penting supaya nanti kamu tahu mengapa middleware dapat menghentikan user biasa sebelum dashboard atau data Product dibuka.

## Bayangkan user mengetuk pintu toko

Misalnya Siti adalah admin. Ia mengetik alamat berikut di browser:

```text
/admin/dashboard
```

Tindakan mengetik alamat dan menekan Enter disebut **request**.

Request adalah permintaan dari browser kepada aplikasi Laravel. Dalam contoh ini, browser meminta:

> “Laravel, tolong tampilkan halaman dashboard admin.”

Laravel tidak langsung menampilkan halaman. Laravel memproses request tersebut melalui beberapa bagian terlebih dahulu.

Bayangkan alurnya seperti ini:

```text
Browser
   ↓ request
Laravel mencari route yang sesuai
   ↓
Middleware memeriksa request
   ↓
Controller menjalankan pekerjaan
   ↓
Blade menyiapkan halaman
   ↓ response
Browser menampilkan halaman
```

## Apa itu route?

**Route** adalah daftar alamat halaman yang dikenali aplikasi Laravel.

Anggap route sebagai daftar petunjuk lokasi di toko:

| Alamat yang diminta | Tujuan di aplikasi |
| --- | --- |
| `/admin/dashboard` | Dashboard admin |
| `/admin/products` | Daftar Product untuk admin |
| `/admin/orders` | Daftar order |
| `/admin/users` | Daftar user |

Contoh route sederhana untuk dashboard admin adalah:

```php
use App\Http\Controllers\AdminDashboardController;
use Illuminate\Support\Facades\Route;

Route::get('/admin/dashboard', [AdminDashboardController::class, 'index']);
```

Penjelasan setiap bagian:

- `Route::get(...)` berarti route ini menerima request saat user membuka halaman dengan metode **GET**. Browser biasanya memakai GET ketika membuka URL.
- `'/admin/dashboard'` adalah URL yang dibuka user.
- `AdminDashboardController::class` adalah controller yang akan menangani request.
- `'index'` adalah method di dalam controller yang dijalankan.

Kode itu memberi tahu Laravel:

> “Jika ada user membuka `/admin/dashboard`, jalankan method `index` pada `AdminDashboardController`.”

Untuk sementara, bayangkan route sebagai **peta alamat**, bukan sebagai penjaga keamanan.

## Apa itu controller?

**Controller** adalah tempat kita menulis langkah kerja untuk sebuah halaman atau fitur.

Pada aplikasi CRUD Product yang sudah kamu pelajari, controller biasanya melakukan pekerjaan seperti:

- mengambil daftar Product dari database,
- menyimpan Product baru,
- mencari Product yang akan diedit,
- menghapus Product,
- lalu mengirim data ke Blade.

Contoh controller dashboard admin yang sangat sederhana:

```php
namespace App\Http\Controllers;

class AdminDashboardController extends Controller
{
    public function index()
    {
        return view('admin.dashboard');
    }
}
```

Penjelasan setiap bagian:

- `namespace App\Http\Controllers;` menunjukkan bahwa file ini berada di kelompok controller aplikasi.
- `class AdminDashboardController extends Controller` membuat controller bernama `AdminDashboardController`.
- `public function index()` adalah pekerjaan yang dijalankan ketika route memanggil `index`.
- `return view('admin.dashboard');` meminta Laravel menampilkan file Blade dashboard admin.

Nantinya dashboard dapat berisi jumlah Product, order, dan user. Namun sebelum controller melakukan pekerjaan itu, kita harus memastikan orang yang datang memang admin.

Di sinilah middleware digunakan.

## Letak middleware di antara route dan controller

Middleware berada di jalur request **sebelum** controller melakukan pekerjaannya.

Bayangkan kembali Siti membuka dashboard:

```text
1. Siti membuka /admin/dashboard di browser.
2. Laravel menemukan route /admin/dashboard.
3. Middleware memeriksa request Siti.
4. Jika Siti boleh masuk, Laravel menjalankan AdminDashboardController.
5. Controller mengirim halaman dashboard ke browser.
```

Jika yang membuka URL adalah Andi dengan role `user`, middleware dapat menghentikan proses pada langkah 3:

```text
1. Andi membuka /admin/dashboard di browser.
2. Laravel menemukan route /admin/dashboard.
3. Middleware memeriksa role Andi.
4. Role Andi adalah user, bukan admin.
5. Middleware menghentikan request.
6. Controller dashboard tidak dijalankan.
```

Ini adalah alasan middleware admin penting: data dan aksi di controller tidak sempat dijalankan oleh user yang tidak berhak.

## Alur tiga kondisi yang perlu dibedakan

Middleware admin nanti perlu memahami tiga kondisi user berikut.

### 1. User belum login

Misalnya seseorang belum masuk ke aplikasi, lalu membuka:

```text
/admin/products
```

Laravel perlu mengarahkannya ke halaman login terlebih dahulu.

```text
Belum login
    ↓
Buka /admin/products
    ↓
Arahkan ke halaman login
```

### 2. User sudah login, tetapi role-nya `user`

Misalnya Andi sudah login, tetapi role-nya adalah `user`. Ia mencoba membuka:

```text
/admin/orders
```

Andi dikenal oleh aplikasi, tetapi ia bukan pengelola order. Middleware harus menolak aksesnya.

```text
Sudah login sebagai user
    ↓
Buka /admin/orders
    ↓
Tolak akses
    ↓
Controller daftar order tidak dijalankan
```

### 3. User sudah login dengan role `admin`

Misalnya Siti sudah login dan role-nya `admin`. Saat ia membuka:

```text
/admin/users
```

middleware mengizinkan request tersebut diteruskan ke controller.

```text
Sudah login sebagai admin
    ↓
Buka /admin/users
    ↓
Middleware mengizinkan
    ↓
Controller daftar user dijalankan
    ↓
Halaman daftar user tampil
```

## Hubungan `auth` dan middleware admin

Laravel biasanya sudah memiliki middleware bernama `auth`.

Tugas `auth` hanya memeriksa satu hal:

> “Apakah user sudah login?”

Middleware admin yang akan kita buat memiliki tugas tambahan:

> “Apakah user yang sudah login ini memiliki role `admin`?”

Karena itu, untuk halaman admin kita nantinya memakai dua lapisan pemeriksaan:

```text
Request halaman admin
        ↓
Middleware auth
        ↓
Pastikan user sudah login
        ↓
Middleware admin
        ↓
Pastikan role user adalah admin
        ↓
Controller
```

Analogi sederhananya:

- `auth` seperti petugas yang memeriksa apakah kamu punya kartu masuk gedung.
- middleware `admin` seperti petugas ruang khusus yang memeriksa apakah kartu kamu bertuliskan **admin**.

Memiliki kartu masuk gedung tidak otomatis berarti boleh masuk ke semua ruangan.

## Di mana middleware diatur pada Laravel 13+?

Pada Laravel 13+, pengaturan middleware aplikasi berada di:

```text
bootstrap/app.php
```

Di file tersebut terdapat bagian seperti ini:

```php
->withMiddleware(function (Middleware $middleware): void {
    //
})
```

Penjelasan sederhana:

- `withMiddleware(...)` adalah tempat Laravel menerima pengaturan middleware aplikasi.
- `function (Middleware $middleware)` memberi kita objek `$middleware` untuk melakukan pengaturan.
- Nantinya kita akan mendaftarkan nama singkat, atau **alias**, untuk middleware admin di bagian ini.

Pada Laravel versi lama, kamu mungkin menemukan tutorial yang mendaftarkan middleware di:

```text
app/Http/Kernel.php
```

Untuk materi ini, **jangan mengikuti cara itu** jika project kamu memakai Laravel 13+. Laravel 13+ menggunakan pengaturan modern di `bootstrap/app.php`.

Kita belum perlu mengubah file tersebut sekarang. Cukup ingat lokasi ini karena akan dipakai pada tahap pendaftaran middleware.

## Gambaran lengkap untuk CRUD Product

Nantinya satu kelompok route admin akan dilindungi dengan dua middleware:

```php
Route::middleware(['auth', 'admin'])->group(function () {
    Route::get('/admin/dashboard', [AdminDashboardController::class, 'index']);

    Route::resource('/admin/products', ProductController::class);

    Route::get('/admin/orders', [OrderController::class, 'index']);
    Route::get('/admin/users', [UserController::class, 'index']);
});
```

Jangan salin kode ini ke project dulu. Ini hanya peta tujuan agar kamu memahami konsepnya.

Artinya:

- `Route::middleware(['auth', 'admin'])` memasang dua penjaga.
- `auth` memastikan user sudah login.
- `admin` memastikan role user adalah `admin`.
- `group(...)` berarti semua route di dalam kelompok tersebut memakai penjaga yang sama.
- Dashboard admin, tambah Product, edit Product, hapus Product, daftar order, dan daftar user akan terlindungi.

Dengan pendekatan ini, kita tidak perlu mengulang pemeriksaan role di setiap method `create`, `store`, `edit`, `update`, atau `destroy` pada `ProductController`.

## Yang perlu diingat pada tahap ini

1. Request dimulai saat user membuka URL dari browser.
2. Route menentukan controller atau tindakan mana yang akan menangani URL tersebut.
3. Middleware berada sebelum controller, sehingga dapat menghentikan request lebih awal.
4. Middleware `auth` memeriksa login, sedangkan middleware admin memeriksa role `admin`.
5. Pada Laravel 13+, pengaturan middleware dilakukan melalui `bootstrap/app.php`, bukan `app/Http/Kernel.php`.

Tahap berikutnya akan membahas lebih dalam perbedaan authentication dan authorization, lalu menghubungkannya dengan data role `admin` dan `user` pada tabel `users`.

---

**Apakah kamu ingin lanjut ke tahap 3: memahami authentication, authorization, dan data role pada user?**
