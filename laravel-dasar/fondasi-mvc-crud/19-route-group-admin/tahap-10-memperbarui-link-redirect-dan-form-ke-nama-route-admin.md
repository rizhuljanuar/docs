# Tahap 10 — Memperbarui Link, Redirect, dan Form ke Nama Route `admin.*`

> Fokus: memperbarui seluruh pemanggilan nama route CRUD Product setelah route dipindahkan ke group `name('admin.')`, agar link, redirect, dan form tidak menunjuk ke nama route lama.

Pada tahap 8, kita memindahkan CRUD Product ke route group admin:

```php
Route::middleware(['auth', 'admin'])
    ->prefix('admin')
    ->name('admin.')
    ->group(function () {
        Route::get('/dashboard', [AdminDashboardController::class, 'index'])
            ->name('dashboard');

        Route::resource('products', ProductController::class);
    });
```

Perpindahan ini mengubah dua hal pada route Product:

```text
URL lama:        /products/...
URL baru:        /admin/products/...

Nama route lama: products....
Nama route baru: admin.products....
```

Laravel sudah membuat route baru dengan benar. Namun file Blade dan controller mungkin masih memanggil nama route lama. Pada tahap ini, kita memperbaruinya secara hati-hati.

## Mengapa link dan form harus diperbarui?

Bayangkan kamu memindahkan ruang produk ke area admin. Peta gedung sudah diperbarui, tetapi papan petunjuk lama masih menunjuk ke ruangan lama.

```text
Peta baru: admin.products.index
Papan petunjuk lama: products.index
```

Laravel tidak otomatis mengubah semua tulisan `route('products.index')` di file lain. Kita perlu menggantinya agar semua bagian aplikasi memakai nama route yang baru.

Jika tidak diperbarui, Laravel dapat menampilkan error seperti:

```text
Route [products.index] not defined.
```

## Daftar perubahan nama route Product

Gunakan tabel ini sebagai peta perubahan.

| Sebelum route group | Setelah route group admin |
| --- | --- |
| `products.index` | `admin.products.index` |
| `products.create` | `admin.products.create` |
| `products.store` | `admin.products.store` |
| `products.show` | `admin.products.show` |
| `products.edit` | `admin.products.edit` |
| `products.update` | `admin.products.update` |
| `products.destroy` | `admin.products.destroy` |

Pola sederhananya:

```text
products.nama-route
        ↓
admin.products.nama-route
```

## Langkah 1, cari penggunaan nama route Product lama

Cari di folder project, terutama pada:

```text
resources/views/
app/Http/Controllers/
```

Nama yang perlu dicari antara lain:

```text
route('products.
redirect()->route('products.
```

Kamu juga dapat mencari `products.` secara umum, tetapi periksa konteks setiap hasil. Jangan mengganti teks secara membabi buta.

Contoh penggunaan yang biasanya ada pada CRUD Product:

```text
resources/views/products/index.blade.php
resources/views/products/create.blade.php
resources/views/products/edit.blade.php
app/Http/Controllers/ProductController.php
```

Nama dan lokasi view mungkin berbeda di projectmu.

## Langkah 2, perbarui link di halaman daftar Product

Di `resources/views/products/index.blade.php`, biasanya ada link tombol tambah dan link edit.

### Tombol tambah Product

Sebelum:

```blade
<a href="{{ route('products.create') }}">Tambah Produk</a>
```

Sesudah:

```blade
<a href="{{ route('admin.products.create') }}">Tambah Produk</a>
```

Penjelasan:

- `route(...)` meminta Laravel membuat URL dari nama route.
- `admin.products.create` menghasilkan URL `/admin/products/create`.
- Kita tidak menulis URL secara manual, sehingga link tetap mengikuti konfigurasi route.

### Link edit Product

Sebelum:

```blade
<a href="{{ route('products.edit', $product) }}">Edit</a>
```

Sesudah:

```blade
<a href="{{ route('admin.products.edit', $product) }}">Edit</a>
```

`$product` tetap dikirim agar Laravel mengetahui Product mana yang akan diedit. Yang berubah hanya nama route-nya.

### Form hapus Product

Sebelum:

```blade
<form action="{{ route('products.destroy', $product) }}" method="POST">
    @csrf
    @method('DELETE')

    <button type="submit">Hapus</button>
</form>
```

Sesudah:

```blade
<form action="{{ route('admin.products.destroy', $product) }}" method="POST">
    @csrf
    @method('DELETE')

    <button type="submit">Hapus</button>
</form>
```

Penjelasan:

- `admin.products.destroy` membentuk URL hapus di bawah `/admin/products/...`.
- `@csrf` tetap dibutuhkan untuk perlindungan form web Laravel.
- `@method('DELETE')` tetap dibutuhkan karena HTML form asli hanya mendukung `GET` dan `POST`.
- Yang berubah hanya nama tujuan route.

## Langkah 3, perbarui form tambah Product

Di file form tambah, misalnya:

```text
resources/views/products/create.blade.php
```

Perbarui tujuan form simpan.

Sebelum:

```blade
<form action="{{ route('products.store') }}" method="POST">
    @csrf

    <!-- Input Product -->
</form>
```

Sesudah:

```blade
<form action="{{ route('admin.products.store') }}" method="POST">
    @csrf

    <!-- Input Product -->
</form>
```

Method form masih `POST` karena resource route memakai `POST /admin/products` untuk menjalankan method `store()` pada `ProductController`.

## Langkah 4, perbarui form edit Product

Di file form edit, misalnya:

```text
resources/views/products/edit.blade.php
```

Perbarui tujuan form update.

Sebelum:

```blade
<form action="{{ route('products.update', $product) }}" method="POST">
    @csrf
    @method('PUT')

    <!-- Input Product -->
</form>
```

Sesudah:

```blade
<form action="{{ route('admin.products.update', $product) }}" method="POST">
    @csrf
    @method('PUT')

    <!-- Input Product -->
</form>
```

`$product`, `@csrf`, dan `@method('PUT')` tidak berubah. Hanya nama route tujuan yang berubah menjadi `admin.products.update`.

## Langkah 5, perbarui link kembali ke daftar Product

Di halaman tambah atau edit, sering ada tombol kembali.

Sebelum:

```blade
<a href="{{ route('products.index') }}">Kembali</a>
```

Sesudah:

```blade
<a href="{{ route('admin.products.index') }}">Kembali</a>
```

Link daftar Product di menu navigasi admin juga memakai nama yang sama:

```blade
<a href="{{ route('admin.products.index') }}">Produk</a>
```

## Langkah 6, perbarui redirect pada `ProductController`

Setelah Product dibuat, diubah, atau dihapus, controller biasanya mengarahkan admin kembali ke halaman daftar Product.

Buka:

```text
app/Http/Controllers/ProductController.php
```

Cari contoh seperti:

```php
return redirect()->route('products.index');
```

Ubah menjadi:

```php
return redirect()->route('admin.products.index');
```

Biasanya perubahan ini diperlukan pada method:

```text
store()
update()
destroy()
```

Contoh pola lengkap:

```php
public function store(StoreProductRequest $request)
{
    Product::create($request->validated());

    return redirect()->route('admin.products.index')
        ->with('success', 'Produk berhasil ditambahkan.');
}
```

Jangan mengubah validasi, penyimpanan data, atau flash message jika tidak diperlukan. Pada tahap ini, fokusnya hanya tujuan redirect.

## Langkah 7, perbarui link dashboard jika sudah berubah

Pada tahap 7, nama dashboard berubah dari `dashboard` menjadi `admin.dashboard`.

Jika ada link atau redirect dashboard admin, ubah:

```text
route('dashboard')
redirect()->route('dashboard')
```

menjadi:

```text
route('admin.dashboard')
redirect()->route('admin.dashboard')
```

Sekali lagi, periksa konteksnya. Jangan mengganti route dashboard milik fitur lain apabila aplikasi nantinya memiliki dashboard user yang terpisah.

## Ringkasan perubahan per file

| Lokasi umum | Contoh nama lama | Nama baru |
| --- | --- | --- |
| `products/index.blade.php` | `products.create`, `products.edit`, `products.destroy` | `admin.products.create`, `admin.products.edit`, `admin.products.destroy` |
| `products/create.blade.php` | `products.store`, `products.index` | `admin.products.store`, `admin.products.index` |
| `products/edit.blade.php` | `products.update`, `products.index` | `admin.products.update`, `admin.products.index` |
| `ProductController.php` | `products.index` pada redirect | `admin.products.index` |
| Menu admin | `dashboard`, `products.index` | `admin.dashboard`, `admin.products.index` |

## Jangan mengganti URL manual jika memakai `route()`

Hindari pola berikut:

```blade
<a href="/admin/products/create">Tambah Produk</a>
```

Kode tersebut mungkin bekerja, tetapi menyebarkan alamat URL ke banyak view. Jika prefix berubah lagi, banyak file harus diperbarui.

Gunakan:

```blade
<a href="{{ route('admin.products.create') }}">Tambah Produk</a>
```

Dengan cara ini, Blade hanya tahu nama route. Laravel yang menentukan URL.

## Kesalahan umum

### Mengganti hanya tombol tambah

CRUD memiliki banyak tujuan route. Jika hanya `products.create` yang diganti, form simpan, edit, update, hapus, atau redirect mungkin masih error.

Gunakan tabel perubahan nama route sebagai checklist.

### Mengubah route yang belum ada

Pastikan route group dan `Route::resource('products', ProductController::class)` sudah ditulis sebelum memakai `admin.products.*` di Blade atau controller.

### Menulis awalan `admin.` dua kali

Di Blade dan controller, gunakan nama route lengkap **sekali**:

```php
route('admin.products.index')
```

Bukan:

```php
route('admin.admin.products.index')
```

### Lupa route `show`

Jika aplikasimu mempunyai halaman detail Product yang sebelumnya memakai:

```blade
route('products.show', $product)
```

ubah menjadi:

```blade
route('admin.products.show', $product)
```

Jika halaman detail Product adalah halaman publik, jangan memindahkannya ke route admin tanpa memisahkan desain dan aksesnya terlebih dahulu.

## Urutan aman saat menerapkan perubahan

Saat menerapkan ke project Laravel, lakukan secara berurutan:

1. Pastikan route group admin dan resource Product sudah terdaftar.
2. Pindahkan atau hapus resource Product lama yang tidak seharusnya terbuka.
3. Perbarui link dan form di Blade.
4. Perbarui redirect di `ProductController`.
5. Periksa route dengan `php artisan route:list --path=admin`.
6. Uji halaman daftar, tambah, simpan, edit, update, dan hapus sebagai admin.

Tahap 11 akan membahas pemeriksaan route dan pengujian akses secara khusus.

## Ringkasan

Setelah CRUD Product masuk ke group `name('admin.')`, semua pemanggilan nama route Product harus memakai awalan `admin.`:

```text
products.* → admin.products.*
```

Perbarui tiga tempat utama:

```text
Link Blade
Form action Blade
Redirect controller
```

Dengan begitu, setiap tombol, form, dan redirect Product menuju URL admin yang benar dan tetap terlindungi middleware `auth` serta `admin`.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: memeriksa route dengan `php artisan route:list` dan menguji akses sebagai guest, user biasa, serta admin?**
