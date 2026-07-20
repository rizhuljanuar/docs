# Tahap 2 — Membuat DashboardController & Route (Versi Paling Sederhana)

> Materi: Dashboard Admin Sederhana
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: Produk, User, Order

---

## 1. Pengantar Sederhana

Pada **Tahap 1**, kita sudah belajar konsep dashboard:

- Apa itu dashboard admin.
- Kenapa admin butuh ringkasan data.
- Bagaimana sakitnya kerja tanpa dashboard.

Sekarang, di **Tahap 2** ini, kita akan **memulai coding**.

Tapi tenang, kita **tidak akan langsung** menghitung jumlah produk, user, atau order. Itu semua ditunda ke **Tahap 3**.

Di tahap 2 ini, tujuannya **sangat sederhana**:

> **Buat "rumah kosong" dulu untuk dashboard kita.**

Maksudnya: kita buat **DashboardController** (controller untuk dashboard), **route** (alamat URL), dan **view** (halaman tampilannya) yang **masih kosong**. Isinya hanya teks "Halo, ini dashboard".

Kenapa kosong dulu? Karena kita ingin **paham struktur dulu** sebelum mengisi logika. Sama seperti waktu belajar CRUD Produk dulu, kita buat `ProductController` kosong dulu, baru isi method `index()`, `create()`, `store()`, dst.

### Analogi: Membangun Rumah

Bayangkan kamu mau membangun **rumah**.

Apakah kamu langsung beli **meja, kursi, kulkas, TV** di hari pertama? Tidak.

Urutan yang waras:

1. **Hari 1**: Bikin pondasi + dinding + atap. Rumah masih kosong. ✅
2. **Hari 2**: Pasang listrik + air.
3. **Hari 3**: Beli meja, kursi, kulkas.
4. **Hari 4**: Pindah masuk.

Di tahap 2 ini, kita di **Hari 1**: bikin struktur kosongnya dulu.

---

## 2. Apa Itu Controller? (Singkat: Ingatan Ulang)

Sebelum lanjut, kita ingat-ingat lagi apa itu Controller dari materi CRUD Produk.

Bayangkan aplikasi Laravel itu seperti **toko**:

| Bagian Laravel | Peran di Toko |
|---|---|
| **Route** (`web.php`) | Resepsionis yang arahkan pengunjung |
| **Controller** | **Manajer toko** yang menentukan apa yang harus dilakukan |
| **Model** | Petugas arsip yang ambil data dari gudang (database) |
| **View** (`.blade.php`) | Etalase yang dilihat pengunjung |

Jadi, **Controller** = tempat kita menulis **logika aplikasi**.

- Untuk produk, logikanya ada di `ProductController`.
- Untuk user, logikanya ada di `UserController`.
- Untuk order, logikanya ada di `OrderController`.
- **Untuk dashboard, logikanya akan ada di `DashboardController`.**

### Kenapa Dashboard Butuh Controller Sendiri?

Pertanyaan bagus: "Bukankah dashboard cuma menampilkan angka? Kenapa tidak ditulis langsung di route?"

Jawabannya: **"Best practice" (praktik yang baik).**

| Kalau logika dashboard ditulis di... | Maka... |
|---|---|
| Route (`web.php`) | File route jadi **panjang dan berantakan** karena semua angka (count produk, count user, sum pendapatan) ditulis di sana. |
| Controller sendiri (`DashboardController`) | File route tetap **rapi**, logika dashboard terpisah di satu file, mudah dibaca dan diubah. |

Jadi, **setiap fitur besar** punya controller sendiri. Dashboard adalah satu fitur, maka dia punya `DashboardController`.

---

## 3. Struktur Folder yang Akan Kita Buat

Sebelum mulai ngoding, mari lihat dulu **gambar besar** file apa saja yang akan kita buat di tahap 2 ini.

```
app/
└── Http/
    └── Controllers/
        └── Admin/
            └── DashboardController.php     ← BARU (controller kita)

routes/
└── web.php                                  ← EDIT (tambah route dashboard)

resources/
└── views/
    └── admin/
        └── dashboard.blade.php             ← BARU (view dashboard)
```

Tiga file saja:
1. `DashboardController.php` (baru).
2. `web.php` (edit, tambah 1 route).
3. `dashboard.blade.php` (baru).

Setelah selesai, nanti kita bisa buka browser ke `http://localhost:8000/admin/dashboard` dan lihat halaman dashboard (masih kosong, tapi sudah "hidup").

---

## 4. Langkah 1: Membuat DashboardController

### Cara 1: Pakai Perintah Artisan (Rekomendasi)

Buka terminal, pastikan kamu di folder project Laravel, lalu jalankan:

```bash
php artisan make:controller Admin/DashboardController
```

Penjelasan per bagian:

| Bagian | Arti |
|---|---|
| `php artisan` | Memanggil "asisten" Laravel. |
| `make:controller` | Perintah untuk membuat file controller baru. |
| `Admin/DashboardController` | Nama controller-nya, diletakkan di dalam folder `Admin/`. |

### Kenapa Pakai Folder `Admin/`?

Karena nanti kita akan punya banyak controller untuk panel admin:

```
app/Http/Controllers/
├── Admin/
│   ├── DashboardController.php    ← milik admin
│   ├── ProductController.php      ← milik admin (kalau kita pindah)
│   ├── UserController.php         ← milik admin
│   └── OrderController.php        ← milik admin
├── ProductController.php          ← milik halaman publik (kalau ada)
└── ...
```

Memisahkan controller admin ke folder `Admin/` membuat project **lebih rapi**, terutama kalau aplikasi membesar.

### Hasil File yang Dibuat Artisan

Setelah jalankan perintah di atas, Laravel akan membuat file di:

```
app/Http/Controllers/Admin/DashboardController.php
```

Isi default-nya seperti ini:

```php
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class DashboardController extends Controller
{
    //
}
```

Mari kita bedah bagian per bagian supaya tidak bingung.

### Penjelasan Kode Defaultnya

**Bagian 1: `<?php`**

```php
<?php
```

Ini tanda "file ini berisi kode PHP". Wajib ada di setiap file PHP. Tanpa ini, Laravel tidak akan menjalankan kodenya.

**Bagian 2: `namespace`**

```php
namespace App\Http\Controllers\Admin;
```

Ini alamat "rumah" controller ini. Karena file ini ada di folder `Admin/`, maka namespace-nya `App\Http\Controllers\Admin`.

Bayangkan seperti **alamat rumah**:
- `App\Http\Controllers` = nama jalan.
- `Admin` = nomor rumah (folder khusus admin).

**Bagian 3: `use ... Controller`**

```php
use App\Http\Controllers\Controller;
```

Baris ini bilang: "Saya mau **pakai** controller induk (`Controller`) yang ada di `App\Http\Controllers`."

Semua controller Laravel **harus mewarisi** (extends) class `Controller` induk. Itu yang memberi mereka "kemampuan dasar" controller Laravel.

**Bagian 4: `use Illuminate\Http\Request`**

```php
use Illuminate\Http\Request;
```

Baris ini **impor** class `Request`. Class ini dipakai kalau kita mau **membaca input dari user** (misalnya form input). Untuk dashboard sederhana, kita **belum butuh ini**, tapi biarkan saja tidak masalah.

**Bagian 5: Deklarasi Class**

```php
class DashboardController extends Controller
{
    //
}
```

- `class DashboardController` = nama class-nya. Harus sama dengan nama file.
- `extends Controller` = mewarisi kemampuan controller induk.
- `{ ... }` = isi class (di sinilah kita akan tulis method).
- `//` = komentar kosong, belum ada kode.

---

## 5. Langkah 2: Tambah Method `index()`

Controller tanpa method seperti **rumah tanpa kamar**. Saat pengunjung datang, tidak ada ruangan untuk menyambut mereka.

Kita akan tambahkan **satu method** bernama `index()`. Method ini bertugas **menampilkan halaman utama dashboard**.

Ubah isi `DashboardController.php` menjadi:

```php
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class DashboardController extends Controller
{
    public function index()
    {
        // Untuk sementara, kita kembalikan teks sederhana.
        // Nanti di tahap berikutnya, kita akan ganti dengan view + data asli.
        return 'Halo, ini halaman Dashboard Admin.';
    }
}
```

### Penjelasan Method `index()`

**Bagian 1: `public function index()`**

```php
public function index()
```

- `public` = method ini bisa diakses dari luar class.
- `function` = kata kunci untuk membuat fungsi/method.
- `index` = nama method. Konvensi Laravel: method `index()` biasanya untuk **menampilkan daftar/halaman utama**.

Kenapa namanya `index()`? Karena ini adalah **halaman utama** dashboard. Sama seperti di `ProductController`, method `index()` dipakai untuk menampilkan daftar produk.

**Bagian 2: Isi method (komentar + return)**

```php
// Untuk sementara, kita kembalikan teks sederhana.
return 'Halo, ini halaman Dashboard Admin.';
```

- Baris komentar (`//`) hanya catatan untuk manusia, tidak dijalankan Laravel.
- `return '...';` = kembalikan teks ini ke browser.

Dalam Laravel, kalau sebuah method controller `return` sebuah **string**, Laravel akan langsung menampilkannya sebagai respons HTTP. Jadi browser akan menampilkan teks "Halo, ini halaman Dashboard Admin."

Kenapa kita mulai dengan teks sederhana? Karena kita mau **uji apakah controller sudah jalan** dulu, sebelum pusing dengan query database, view blade, dll. Ini cara debugging yang sehat: **satu langkah kecil, uji, baru lanjut.**

---

## 6. Langkah 3: Tambah Route di `web.php`

Sekarang, controller sudah ada. Tapi belum bisa diakses karena belum ada **route** yang mengarah ke controller ini.

Buka file:

```
routes/web.php
```

Kamu akan melihat isinya kira-kira seperti ini (tergantung materi sebelumnya):

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Admin\ProductController;

Route::get('/', function () {
    return view('welcome');
});

// Rute-rute produk dari materi sebelumnya
Route::get('/products', [ProductController::class, 'index'])->name('products.index');
// ... dst
```

### Tambahkan Kode Berikut di Akhir File

```php
use App\Http\Controllers\Admin\DashboardController;

Route::get('/admin/dashboard', [DashboardController::class, 'index'])
    ->name('admin.dashboard');
```

### Penjelasan Kode di Atas

**Bagian 1: `use App\Http\Controllers\Admin\DashboardController;`**

```php
use App\Http\Controllers\Admin\DashboardController;
```

Baris ini **mengimpor** class `DashboardController` supaya bisa dipakai di file `web.php`. Tanpa ini, Laravel tidak tahu `DashboardController` itu apa.

Letakkan baris `use ...` ini di **bagian atas file** (setelah `<?php`), bersama dengan baris `use` lainnya.

**Bagian 2: `Route::get(...)`**

```php
Route::get('/admin/dashboard', [DashboardController::class, 'index'])
    ->name('admin.dashboard');
```

Mari kita bedah per bagian:

| Bagian | Arti |
|---|---|
| `Route::get(...)` | Daftarkan route untuk method HTTP **GET** (saat user membuka URL di browser). |
| `'/admin/dashboard'` | URL yang diketik user di browser. |
| `[DashboardController::class, 'index']` | "Kalau URL ini dibuka, panggil class `DashboardController`, jalankan method `index`." |
| `->name('admin.dashboard')` | Beri **nama** untuk route ini. Nama berguna kalau nanti kita mau **redirect** atau **buat link** ke dashboard dari tempat lain. |

### Kenapa URL-nya `/admin/dashboard`, Bukan `/dashboard`?

Karena kita sedang bangun **panel admin**, bukan website publik. URL dengan prefix `/admin/` membuatnya jelas bahwa ini halaman admin.

Nanti kalau aplikasi membesar, semua halaman admin akan pakai prefix `/admin/`:

```
/admin/dashboard       ← dashboard
/admin/products        ← kelola produk
/admin/users           ← kelola user
/admin/orders          ← kelola order
```

Sedangkan halaman publik tetap pakai URL biasa:

```
/                      ← halaman depan toko
/products              ← katalog produk untuk pengunjung
/products/{slug}       ← detail produk untuk pengunjung
```

Pemisahan ini membuat aplikasi **lebih rapi dan aman**.

---

## 7. Langkah 4: Uji Coba di Browser

Sekarang waktunya **uji coba**. Langkah ini penting banget, karena kita mau pastikan apa yang kita tulis benar-benar jalan.

### Langkah Uji Coba

1. Pastikan server Laravel berjalan. Buka terminal, jalankan:

```bash
php artisan serve
```

2. Buka browser, ketik:

```
http://localhost:8000/admin/dashboard
```

3. Seharusnya kamu melihat teks:

```
Halo, ini halaman Dashboard Admin.
```

### Kalau Ada Error

Kalau yang muncul **bukan** teks di atas, berikut beberapa error umum dan cara perbaikannya:

**Error 1: "Target class [Admin\DashboardController] does not exist."**

Penyebab: File `DashboardController.php` tidak ditemukan, atau namespace-nya salah.

Solusi:
- Cek apakah file ada di `app/Http/Controllers/Admin/DashboardController.php`.
- Cek baris `namespace App\Http\Controllers\Admin;` di file controller.
- Jalankan `composer dump-autoload`.

**Error 2: "404 Not Found"**

Penyebab: URL yang kamu ketik tidak cocok dengan route.

Solusi:
- Cek apakah kamu sudah tulis `/admin/dashboard` (bukan `/dashboard`).
- Cek file `web.php`, pastikan tidak ada typo.

**Error 3: "Route [admin.dashboard] not defined."**

Penyebab: Kamu sudah pakai route name tapi belum didefinisikan.

Solusi: Pastikan ada `->name('admin.dashboard')` di route.

**Error 4: Blank Page / Error 500**

Penyebab: Ada error PHP di file controller.

Solusi:
- Cek storage log di `storage/logs/laravel.log`.
- Pastikan tidak ada typo di controller (terutama titik koma `;` dan kurung kurawal `{}`).

---

## 8. Kalau Sudah Berhasil: Apa Berikutnya?

Selamat! Kalau teks "Halo, ini halaman Dashboard Admin." muncul di browser, berarti:

- ✅ Controller `DashboardController` sudah jalan.
- ✅ Route `/admin/dashboard` sudah benar.
- ✅ Method `index()` berhasil dipanggil.

Sekarang kita siap untuk **mengisi dashboard dengan data asli** di tahap berikutnya.

### Tapi Sebelum Itu, Ringkasan Apa yang Sudah Kita Pelajari di Tahap 2:

| Konsep | Penjelasan Singkat |
|---|---|
| **Tujuan tahap 2** | Buat "rumah kosong" untuk dashboard (controller + route + uji coba). |
| **DashboardController** | Controller khusus untuk menampung logika dashboard. |
| **Kenapa controller sendiri?** | Supaya file route tetap rapi, logika dashboard terpisah dari logika produk/user/order. |
| **Folder `Admin/`** | Tempat menaruh semua controller admin agar project rapi. |
| **Method `index()`** | Method untuk menampilkan halaman utama (konvensi Laravel). |
| **Route dengan name** | `->name('admin.dashboard')` untuk memberi nama route, biar mudah dipanggil dari tempat lain. |
| **Prefix `/admin/`** | Memisahkan URL admin dari URL publik. |

---

## 9. Kenapa Belum Pakai View `.blade.php`?

Mungkin kamu bertanya: "Kok method `index()` cuma `return 'teks';`? Kenapa tidak langsung `return view('admin.dashboard');`?"

Jawabannya: **Sengaja, biar sederhana.**

Kalau di tahap 2 ini kita langsung:
- Buat file `dashboard.blade.php`.
- Buat query `Product::count()`.
- Kirim data ke view.
- Tampilkan kartu angka.

Maka **satu tahap jadi terlalu penuh**. Kamu bisa bingung: "Mana yang penting controller, mana yang penting query, mana yang penting view?"

Maka di materi ini, kita pecah jadi:

| Tahap | Fokus |
|---|---|
| **Tahap 2 (sekarang)** | Struktur controller + route saja. Belum view, belum query. |
| **Tahap 3** | Query `count()` di controller, masih `return` teks. |
| **Tahap 4** | Query `sum()` + `where()` di controller, masih `return` teks. |
| **Tahap 5** | Query `latest()` + `take()` untuk order terbaru. |
| **Tahap 6** | Baru buat `dashboard.blade.php` dan tampilkan data dengan rapi. |
| **Tahap 7** | Tambah cache biar dashboard cepat. |

Satu langkah kecil tiap tahap. Dijamin tidak tersesat.

---

## 10. Kesimpulan Tahap 2

Di tahap 2 ini kita sudah:

1. Membuat `DashboardController` di folder `Admin/`.
2. Menambahkan method `index()` yang `return` teks sederhana.
3. Menambahkan route `/admin/dashboard` di `web.php`.
4. Menguji bahwa halaman dashboard bisa diakses.

Dashboard kita masih **kosong** dan cuma menampilkan teks "Halo, ini halaman Dashboard Admin."

Di **tahap berikutnya**, kita akan ganti teks itu dengan **angka asli** hasil query:

- Jumlah produk.
- Jumlah user.
- Jumlah order.

Caranya pakai **aggregation query `count()`**.

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: belajar aggregation query `count()` untuk menghitung jumlah produk, user, dan order?**

Kalau **ya**, kita akan belajar:

- Apa itu aggregation query (dengan analogi sederhana).
- Cara pakai `Product::count()` di Laravel.
- Cara kirim hasilnya ke browser sementara (masih teks, belum view).

Kalau **belum**, kita bisa:
- Ulang penjelasan tentang controller.
- Ulang penjelasan tentang route.
- Praktik ulang dari awal (hapus file, bikin ulang).

Tinggal bilang saja.
