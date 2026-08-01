# Upload Gambar Produk — Tahap 4: Menyimpan & Menampilkan Gambar

> Bagian dari: Laravel Dasar — Fondasi MVC & CRUD
> Topik: 3. Upload Gambar Produk
> Tahap: 4 dari 5 — **Menyimpan & Menampilkan Gambar** (kode lengkap)

---

## 1. Tujuan Tahap Ini

Setelah tahap 3, file yang masuk sudah **divalidasi** (penjaga pintu bekerja).
Sekarang kita kerjakan:

1. Menyimpan file yang lolos validasi ke disk `public`.
2. Menyimpan **path** gambar ke database (bukan file-nya!).
3. Menampilkan gambar di halaman daftar produk dengan `Storage::url()`.

> **Ponytail:** Kita pakai API bawaan Laravel (`Storage`, `UploadedFile`),
> tidak ada package tambahan. Cukup satu symlink di awal, sisanya kode native.

---

## 2. Persiapan: Pastikan Symlink Sudah Dibuat

Sebelum mulai, jalankan **sekali** (sudah dibahas di tahap 2):

```bash
php artisan storage:link
```

Cek hasilnya: harus ada folder/file `public/storage` yang menunjuk ke
`storage/app/public`. Tanpa symlink ini, gambar yang disimpan tidak bisa
diakses dari browser meskipun filenya ada.

---

## 3. Migration: Tambah Kolom `image` ke Tabel `products`

Tabel `products` sudah kamu buat di **Materi 1 (Tahap 3)** dengan kolom:
`id`, `name`, `description`, `price`, `stock`, `created_at`, `updated_at`.

Sekarang kita **tambah satu kolom baru**: `image`, untuk menyimpan path gambar.

Karena tabelnya sudah ada, kita tidak membuat migration `create`, tapi
migration **`add_column`**. Jalankan perintah berikut di terminal:

```bash
php artisan make:migration add_image_to_products_table --table=products
```

Tanda `--table=products` memberitahu Laravel bahwa migration ini untuk
**mengubah tabel yang sudah ada** (bukan membuat tabel baru).

Buka file migration baru itu di `database/migrations/` dan isi seperti ini:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('products', function (Blueprint $table) {
            // Kolom baru untuk menyimpan PATH gambar (bukan file-nya).
            // nullable() supaya produk yang sudah ada tidak error saat kolom ini ditambahkan.
            $table->string('image')->nullable()->after('description');
        });
    }

    public function down(): void
    {
        Schema::table('products', function (Blueprint $table) {
            $table->dropColumn('image');
        });
    }
};
```

Jalankan migration:

```bash
php artisan migrate
```

### Penjelasan penting

- **`Schema::table`** (bukan `Schema::create`) karena kita mengubah tabel yang
  sudah ada, bukan membuat tabel baru. Ini beda dengan **Materi 1 (Tahap 3)**
  yang memakai `Schema::create` karena tabelnya belum ada.
- **`->nullable()`** supaya produk lama (yang sudah disimpan tanpa gambar)
  tidak error saat kolom `image` ditambahkan ke tabel.
- **`->after('description')`** opsional, hanya untuk mengatur urutan kolom
  di phpMyAdmin supaya rapi. Boleh dihapus.
- **Tipe kolom `string`** (path teks), **bukan** `binary` atau `blob`.
  Kita menyimpan **lokasi file**, bukan isi file.

> Bandingkan dengan **Materi 1 (Tahap 3)**: di sana kamu membuat tabel dari
> nol dengan `Schema::create`. Sekarang kamu **mengubah** tabel yang sudah
> ada dengan `Schema::table`. Inilah cara Laravel mengelola perubahan database
> secara bertahap tanpa menghapus data lama.

---

## 4. Update Model Product: Tambah `image` ke `$fillable`

Buka `app/Models/Product.php` (yang sudah kamu buat di **Materi 1, Tahap 4**).
Tambahkan `image` ke daftar `$fillable`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    protected $fillable = [
        'name',
        'description',
        'price',
        'stock',

        // TAMBAHAN BARU:
        'image',
    ];
}
```

Tanpa mendaftarkan `image` di `$fillable`, `Product::create()` akan
**mengabaikan** field `image` karena fitur keamanan mass assignment Laravel.

---

## 5. Controller: Simpan File + Path ke Database

Buka `app/Http/Controllers/ProductController.php` yang sudah kamu buat di
**Materi 1** dan diperbaiki di **Materi 2**. Kita ubah method `store()` agar
menangani upload file, dan tambahkan import facade `Storage`.

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;  // TAMBAHAN: untuk upload file

class ProductController extends Controller
{
    public function create()
    {
        return view('products.create');
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'name'        => ['required', 'min:3'],
            'price'       => ['required', 'numeric', 'min:0'],
            'stock'       => ['required', 'integer', 'min:0'],
            'description' => ['nullable', 'min:10'],

            // TAMBAHAN untuk upload gambar:
            'image'       => ['required', 'image', 'mimes:jpg,jpeg,png,webp', 'max:2048'],
        ]);

        // 1. Simpan file ke disk 'public', subfolder 'products'
        $path = $request->file('image')->store('products', 'public');
        // contoh hasil $path: "products/aB3xK9pQ.jpg"

        // 2. Tambahkan path gambar ke array validated, lalu simpan ke database
        $validated['image'] = $path;

        Product::create($validated);

        return redirect('/products')->with('success', 'Produk berhasil ditambahkan.');
    }
}
```

### Penjelasan perubahan dari materi sebelumnya

1. **`use Illuminate\Support\Facades\Storage;`** ditambahkan di bagian import.
   Kita baru membutuhkannya sekarang karena sebelumnya tidak ada upload file.
2. **Aturan validasi** untuk `name`, `price`, `stock`, `description` tetap sama
   seperti **Materi 2**. Hanya ditambah satu baris baru: `'image' => [...]`.
3. **`$request->file('image')->store('products', 'public')`** menyimpan file
   gambar ke storage.
4. **`$validated['image'] = $path;`** menambahkan path gambar ke array
   `$validated` sebelum disimpan ke database. Cara ini aman karena kita tetap
   memakai `Product::create($validated)` (mass assignment lewat `$fillable`).

### Penjelasan `$request->file('image')->store('products', 'public')`:

- `file('image')` → ambil instance `UploadedFile` dari input bernama `image`.
- `->store('products', 'public')` → simpan ke disk `public`, di subfolder
  `products`.
- Laravel otomatis membuat nama file unik (terenkripsi) supaya tidak tabrakan
  dengan file lama, contoh: `products/aB3xK9pQ.jpg`.
- Fungsi ini **mengembalikan path relatif** terhadap disk, misalnya
  `"products/aB3xK9pQ.jpg"`. Path inilah yang disimpan ke database.

### Alternatif: `storeAs` (nama yang kita tentukan)

Kalau mau nama file mengikuti nama asli / bisa dibaca manusia:

```php
$namaFile = time() . '-' . $request->file('image')->getClientOriginalName();
$path = $request->file('image')->storeAs('products', $namaFile, 'public');
// contoh: "products/1718000000-sepatu-lari.jpg"
```

Untuk pemula, **`store()` saja sudah cukup** (lebih aman, anti tabrakan).
> **Ponytail:** Jangan pakai `storeAs` kecuali kamu butuh nama yang bisa dibaca
> manusia. Default lebih sederhana.

---

## 6. Blade: Menampilkan Gambar di Halaman Daftar Produk

Buka `resources/views/products/index.blade.php` yang sudah kamu buat di
**Materi 1 (Tahap 7 dan Tahap 9)**. Tambahkan kolom **Gambar** di tabel daftar
produk, atau tampilkan gambar di atas nama produk.

Contoh versi sederhana (hanya bagian yang ditambahkan):

```blade
@use('Illuminate\Support\Facades\Storage')

<!-- ... kode header dan pesan sukses tetap seperti Materi 1 ... -->

@if ($products->isEmpty())
    <p>Belum ada produk.</p>
@else
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>Gambar</th>      {{-- TAMBAHAN: kolom baru --}}
                <th>Nama</th>
                <th>Deskripsi</th>
                <th>Harga</th>
                <th>Stok</th>
            </tr>
        </thead>
        <tbody>
            @foreach ($products as $product)
                <tr>
                    <td>{{ $product->id }}</td>

                    {{-- TAMBAHAN: tampilkan gambar kalau ada --}}
                    <td>
                        @if ($product->image)
                            <img src="{{ Storage::url($product->image) }}"
                                 alt="{{ $product->name }}"
                                 width="100">
                        @else
                            <em>(Tidak ada gambar)</em>
                        @endif
                    </td>

                    <td>{{ $product->name }}</td>
                    <td>{{ $product->description }}</td>
                    <td>Rp {{ number_format($product->price, 0, ',', '.') }}</td>
                    <td>{{ $product->stock }}</td>
                </tr>
            @endforeach
        </tbody>
    </table>
@endif
```

Hal penting:

- `Storage::url($product->image)` → menghasilkan URL publik, contoh:
  `http://toko.test/storage/products/aB3xK9pQ.jpg`.
- Blade butuh directive `@if` karena `image` bisa `null` (di migration tadi
  kita set `nullable()`).
- Gunakan `{{ ... }}` (double curly) supaya path di-escape (aman dari XSS).
- `@use('Illuminate\Support\Facades\Storage')` di atas file Blade diperlukan
  supaya facade `Storage` dikenali. (Di beberapa versi Blade ini otomatis, tapi
  lebih aman tulis eksplisit.)

---

## 7. Route (Tetap Sama Seperti Materi 1)

Route untuk CRUD produk sudah kamu buat di **Materi 1 (Tahap 5 dan Tahap 6)**.
Kita **tidak perlu menambah route baru** karena upload gambar memakai route
yang sudah ada (`POST /products`).

`routes/web.php` tetap seperti ini:

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ProductController;

Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/create', [ProductController::class, 'create']);
Route::post('/products', [ProductController::class, 'store']);  // dipakai juga untuk upload
Route::get('/products/{id}/edit', [ProductController::class, 'edit']);
Route::put('/products/{id}', [ProductController::class, 'update']);
Route::delete('/products/{id}', [ProductController::class, 'destroy']);
Route::get('/products/{id}', [ProductController::class, 'show']);
```

> Yang penting: route `Route::post('/products', ...)` menangani **semua** data
> dari form tambah produk, termasuk file gambar (karena form memakai
> `enctype="multipart/form-data"`).

---

## 8. Alur Lengkap dari Upload ke Tampil

```
1. User pilih sepatu.png, klik Simpan
       │
2. POST /products  (dengan multipart/form-data)
       │
3. ProductController->store()
       │
       ├─ validate()      → cek name, price, stock, description,
       │                     dan image (rule image, mimes, max:2048)
       │
       ├─ ->store('products','public')
       │     └─ file masuk: storage/app/public/products/aB3xK9pQ.jpg
       │
       ├─ $validated['image'] = 'products/aB3xK9pQ.jpg'
       ├─ Product::create($validated)
       │     └─ path masuk DB (kolom image)
       │
       └─ redirect ke /products
              │
4. Halaman products/index.blade.php
       │
       └─ <img src="{{ Storage::url('products/aB3xK9pQ.jpg') }}">
              └─ browser request: /storage/products/aB3xK9pQ.jpg
                  └─ symlink public/storage → storage/app/public
                  └─ gambar muncul di browser ✅
```

---

## 9. Cek Masalah Umum (Troubleshooting)

| Gejala                                     | Penyebab                                | Solusi                                  |
| ------------------------------------------ | --------------------------------------- | --------------------------------------- |
| Gambar tidak muncul (404)                  | Belum jalankan `storage:link`           | `php artisan storage:link`              |
| Gambar broken, padahal filenya ada         | Salah disk / path                       | Pastikan `store('products', 'public')`  |
| Error "disk public does not exist"         | Konfigurasi `config/filesystems.php`    | Cek disk `public` masih ada             |
| Upload gagal besar (>2MB) padahal belum 2MB| Batas `upload_max_filesize` di `php.ini` | Ubah `php.ini`, restart PHP             |
| Error `Storage::url` not found di Blade    | Lupa import facade                      | Pakai `@use` atau `\Storage::url(...)`   |
| Kolom `image` tidak tersimpan di DB        | `image` belum didaftarkan di `$fillable` | Tambahkan `'image'` ke `$fillable` di Model |

---

## 10. Ringkasan Tahap 4

- **Migration**: pakai `Schema::table` (bukan `create`) untuk **menambah**
  kolom `image` ke tabel `products` yang sudah ada. Kolom tipe **string**
  (path), `nullable()`.
- **Model `Product`**: tambahkan `'image'` ke `$fillable`.
- **Simpan file**: `$file->store('products', 'public')` → kembalikan path
  relatif, lalu masukkan ke `$validated['image']` sebelum `Product::create()`.
- **Simpan ke DB**: path saja, bukan file.
- **Tampilkan**: `{{ Storage::url($product->image) }}` di tag `<img>`.
- **Symlink**: jalankan **sekali** `php artisan storage:link`.
- **Cek `@if ($product->image)`** sebelum tampilkan (karena bisa null).
- **Route tidak berubah** dari Materi 1; upload memakai `POST /products`
  yang sudah ada.

Sekarang upload gambar produk sudah berfungsi end-to-end!

---

## 11. Cek Pemahaman

1. Apa yang dikembalikan oleh `$file->store('products', 'public')`?
2. Apa yang disimpan di kolom `image` di database: file atau path?
3. Kenapa harus ada `@if ($product->image)` sebelum tampilkan `<img>`?
4. Kenapa migration di materi ini memakai `Schema::table`, bukan `Schema::create`
   seperti di Materi 1?

---

> **Pertanyaan untuk kamu:** Sudah berhasil alurnya?
> Mau lanjut ke **Tahap 5 — Latihan / Praktik & Rangkuman Akhir**
> (latihan soal, kasus tambahan seperti hapus gambar saat produk dihapus,
> dan rangkuman seluruh materi), atau ulas ulang tahap 4 dulu?
