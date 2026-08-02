# Tahap 3 — Memahami `name('admin.')` pada Nama Route

> Fokus: memahami nama route, alasan nama route lebih aman daripada menulis URL langsung, dan cara `name('admin.')` menambahkan awalan pada route admin.

Pada tahap 2, kita memakai `prefix('admin')` agar semua **URL halaman admin** dimulai dengan `/admin`.

Sekarang kita belajar bagian yang berbeda tetapi tetap berhubungan: **nama route**.

Ingat perbedaannya:

```text
URL route       → alamat yang dibuka di browser
Nama route      → nama panggilan route di dalam kode Laravel
```

## Bayangkan alamat dan nama ruangan

Di sebuah toko, ruangan dapat memiliki dua informasi:

```text
Alamat ruangan: Lantai Admin, Ruang Produk
Nama pada denah: admin.products.index
```

Alamat membantu orang menemukan ruangan secara fisik. Nama pada denah membantu petugas menyebut ruangan itu dengan konsisten.

Laravel juga demikian:

```text
URL:        /admin/products
Nama route: admin.products.index
```

- URL dipakai browser untuk membuka halaman.
- Nama route dipakai kode Laravel, misalnya saat membuat link, redirect, atau tujuan form.

## Apa itu nama route?

Nama route adalah nama khusus yang kita berikan pada sebuah route.

Contoh sederhana di `routes/web.php`:

```php
Route::get('/admin/dashboard', [AdminDashboardController::class, 'index'])
    ->name('admin.dashboard');
```

Penjelasan:

- `Route::get(...)` membuat route untuk membuka halaman.
- `'/admin/dashboard'` adalah URL yang akan dibuka di browser.
- `AdminDashboardController::class` dan `index` adalah controller serta method yang menjalankan halaman.
- `->name('admin.dashboard')` memberi route itu nama `admin.dashboard`.

Nama route tidak terlihat sebagai URL browser. Nama ini adalah “nama panggilan” untuk Laravel.

## Mengapa memakai nama route?

Kamu dapat membuat link dengan menulis URL langsung:

```blade
<a href="/admin/products">Daftar Produk</a>
```

Cara tersebut bekerja, tetapi kurang fleksibel. Jika nanti URL berubah, misalnya dari `/admin/products` menjadi `/admin/catalog/products`, kamu harus mencari dan mengganti URL itu di banyak file Blade atau controller.

Cara yang lebih rapi adalah memakai helper `route()`:

```blade
<a href="{{ route('admin.products.index') }}">Daftar Produk</a>
```

Maknanya:

```text
“Laravel, cari URL untuk route bernama admin.products.index.”
```

Laravel akan menghasilkan URL yang sesuai. Saat URL route berubah, kamu cukup memperbarui definisi route, sementara link yang memakai nama route tetap mengikuti route tersebut.

## Contoh penggunaan nama route pada CRUD Product

Nama route dapat dipakai di beberapa tempat:

### Link menuju daftar produk

```blade
<a href="{{ route('admin.products.index') }}">Daftar Produk</a>
```

Laravel mengarahkan link ke URL daftar produk admin.

### Link menuju form tambah produk

```blade
<a href="{{ route('admin.products.create') }}">Tambah Produk</a>
```

Laravel mengarahkan link ke form tambah produk.

### Redirect setelah produk berhasil disimpan

Di dalam `ProductController`, nanti kamu dapat menulis:

```php
return redirect()->route('admin.products.index');
```

Artinya:

> Setelah proses simpan selesai, arahkan admin ke route daftar produk admin.

### Form untuk menyimpan produk baru

```blade
<form action="{{ route('admin.products.store') }}" method="POST">
```

Artinya form mengirim data ke route khusus untuk menyimpan produk baru.

Kita akan membahas pembaruan link, redirect, dan form secara lebih lengkap pada tahap 10. Untuk saat ini, cukup pahami bahwa nama route membantu Laravel menemukan URL yang tepat.

## Masalah jika nama route admin ditulis berulang satu per satu

Tanpa group nama, kita harus menulis awalan `admin.` sendiri pada setiap route:

```php
Route::get('/admin/dashboard', [AdminDashboardController::class, 'index'])
    ->name('admin.dashboard');

Route::get('/admin/products', [ProductController::class, 'index'])
    ->name('admin.products.index');

Route::get('/admin/orders', [OrderController::class, 'index'])
    ->name('admin.orders.index');
```

Ini mirip dengan masalah prefix pada tahap 2. Kode bisa berjalan, tetapi kita terus mengulang tulisan `admin.`.

## Solusi: `name('admin.')` pada route group

Laravel dapat menambahkan awalan nama route secara otomatis melalui group:

```php
Route::name('admin.')->group(function () {
    Route::get('/dashboard', [AdminDashboardController::class, 'index'])
        ->name('dashboard');

    Route::get('/products', [ProductController::class, 'index'])
        ->name('products.index');
});
```

Perhatikan cara Laravel menggabungkan nama:

```text
name('admin.') + name('dashboard')
        ↓
admin.dashboard

name('admin.') + name('products.index')
        ↓
admin.products.index
```

Penjelasan setiap bagian:

- `Route::name('admin.')` menetapkan awalan nama untuk seluruh route di dalam group.
- Titik `.` di akhir `admin.` sangat penting, supaya hasilnya rapi seperti `admin.dashboard`, bukan `admindashboard`.
- `->name('dashboard')` memberi nama bagian akhir route dashboard.
- `->name('products.index')` memberi nama bagian akhir route daftar produk.

## Digabung dengan prefix dari tahap 2

Prefix URL dan prefix nama route dapat dipakai bersama, tetapi tugasnya berbeda:

```php
Route::prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');
    });
```

Hasilnya:

| Pengaturan | Hasil |
| --- | --- |
| `prefix('admin')` | URL: `/admin/dashboard` |
| `name('admin.')` + `name('dashboard')` | Nama route: `admin.dashboard` |

Jadi jangan tertukar:

```text
prefix('admin')  → mengatur URL
name('admin.')   → mengatur nama route
```

## Hubungan dengan `Route::resource()` untuk Product

Saat nanti kita menulis resource route di dalam group bernama `admin.`, Laravel memberi awalan nama itu pada semua route Product secara otomatis:

```php
Route::name('admin.')->group(function () {
    Route::resource('/products', ProductController::class);
});
```

Contoh hasil nama route Product:

```text
admin.products.index
admin.products.create
admin.products.store
admin.products.edit
admin.products.update
admin.products.destroy
```

Kamu tidak perlu memberi `->name('admin.products.index')` satu per satu kepada setiap route resource.

## Hal yang perlu diperhatikan

- `name('admin.')` tidak mengubah URL. Untuk URL, gunakan `prefix('admin')`.
- Jangan lupa titik pada `admin.`. Titik adalah pemisah nama, bukan bagian dari URL.
- Jika sebelumnya kamu memakai `route('products.index')`, nama itu akan berubah menjadi `route('admin.products.index')` setelah Product dipindahkan ke group `name('admin.')`.
- Perubahan nama route berarti link Blade, redirect controller, dan `action` form yang lama perlu diperbarui. Ini akan dikerjakan dengan teliti pada tahap 10.

## Ringkasan

`name('admin.')` berarti:

> “Tambahkan `admin.` di depan nama semua route yang ada di dalam group ini.”

Dengan nama route, kode Laravel dapat membuat URL dengan `route(...)` tanpa menulis alamat admin secara manual di banyak tempat.

Sekarang kita sudah memiliki dua alat untuk merapikan route admin:

```text
prefix('admin')  → URL admin menjadi rapi
name('admin.')   → nama route admin menjadi rapi
```

Namun, keduanya belum menjaga akses. Pada langkah berikutnya, kita akan menempatkan pemeriksa login, yaitu middleware `auth`, pada route group.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: memahami middleware `auth` pada route group admin?**
