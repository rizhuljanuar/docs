# Tahap 2 — Memahami Prefix `admin` pada URL

> Fokus: memahami bagaimana `prefix('admin')` menambahkan `/admin` pada semua URL di dalam route group.

Pada tahap 1, kita sudah mengetahui bahwa route group adalah pembungkus untuk route yang mempunyai aturan sama. Sekarang kita pelajari aturan pertama: **prefix**.

## Apa itu prefix?

**Prefix** berarti awalan.

Bayangkan semua ruangan pengelola toko berada di lantai bernama **Admin**. Nama lantai itu ditulis di depan setiap petunjuk ruangan:

```text
Admin / Dashboard
Admin / Produk
Admin / Order
Admin / User
```

Di URL Laravel, “lantai Admin” itu ditulis sebagai `/admin`:

```text
/admin/dashboard
/admin/products
/admin/orders
/admin/users
```

Jadi, prefix `admin` membuat semua alamat dalam group mempunyai awalan `/admin`.

## Tanpa prefix, kita menulis `/admin` berulang kali

```php
Route::get('/admin/dashboard', [AdminDashboardController::class, 'index']);
Route::get('/admin/products', [ProductController::class, 'index']);
Route::get('/admin/orders', [OrderController::class, 'index']);
```

Setiap route harus membawa tulisan `/admin` sendiri.

## Dengan prefix, cukup tulis satu kali

```php
Route::prefix('admin')->group(function () {
    Route::get('/dashboard', [AdminDashboardController::class, 'index']);
    Route::get('/products', [ProductController::class, 'index']);
    Route::get('/orders', [OrderController::class, 'index']);
});
```

Penjelasan setiap bagian:

- `Route::prefix('admin')` membuat aturan awalan URL `admin`.
- `->group(function () { ... })` membuka area untuk route yang mengikuti aturan tersebut.
- `Route::get('/dashboard', ...)` tetap ditulis sebagai `/dashboard` di dalam group.
- Laravel menggabungkan keduanya menjadi `/admin/dashboard`.

## Cara Laravel menggabungkannya

```text
prefix('admin') + /dashboard
        ↓
/admin/dashboard

prefix('admin') + /products
        ↓
/admin/products
```

Prefix tidak mengubah controller. Prefix hanya merapikan **URL halaman**.

| Route di dalam group | URL setelah memakai prefix |
| --- | --- |
| `/dashboard` | `/admin/dashboard` |
| `/products` | `/admin/products` |
| `/products/create` | `/admin/products/create` |
| `/orders` | `/admin/orders` |
| `/users` | `/admin/users` |

## Hubungannya dengan CRUD Product

Saat nanti kita memakai resource route untuk Product di dalam group prefix, Laravel membuat URL CRUD seperti berikut:

```text
GET    /admin/products                 daftar produk
GET    /admin/products/create          form tambah produk
POST   /admin/products                 simpan produk baru
GET    /admin/products/{product}/edit  form edit produk
PUT    /admin/products/{product}       simpan perubahan produk
DELETE /admin/products/{product}       hapus produk
```

Kamu tidak perlu menulis `/admin` lagi di setiap route Product.

## Hal yang perlu diperhatikan

- Tulis `prefix('admin')`, bukan `prefix('/admin')`. Laravel akan membentuk garis miring URL dengan benar.
- Prefix hanya membuat URL lebih teratur. Prefix **bukan perlindungan akses**.
- User biasa masih dapat mencoba membuka `/admin/products`. Karena itu, kita tetap membutuhkan middleware `auth` dan `admin` dari materi 18.

## Ringkasan

`prefix('admin')` berarti:

> “Tambahkan `/admin` di depan URL semua route yang ada di dalam group ini.”

Prefix membantu pembaca kode langsung tahu bahwa dashboard, produk, order, dan user adalah bagian dari area admin.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: memahami nama route dan awalan `admin.`?**
