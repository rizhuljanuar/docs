# Tahap 12 — Mengupdate Produk

> **Tujuan tahap ini:** Memproses data perubahan dari form edit dan menyimpannya ke database. **Belum membuat fitur hapus produk, pagination, search, atau upload gambar.** Hanya fokus pada proses Update.

---

## 1. Pengantar Sederhana

Pada **Tahap 11**, kita sudah membuat **form edit produk** yang bisa:

- Dibuka melalui URL `/products/{id}/edit`.
- Menampilkan data lama produk di dalam field form.
- Menampilkan input nama, deskripsi, harga, dan stok.

Namun, tombol **Update** belum benar-benar menyimpan perubahan ke database.
Saat diklik, tidak terjadi apa-apa (atau error) karena `action="#"` dan
route PUT belum diproses.

Di tahap ini, kita akan memperbaiki itu: membuat **proses update** agar
perubahan di form edit benar-benar tersimpan ke tabel `products`.

### Analogi: Formulir Lama yang Diperbaiki

| Hal                    | Analogi                                                  |
| ---------------------- | -------------------------------------------------------- |
| Form edit produk       | Formulir lama yang dibuka kembali                         |
| Admin memperbaiki form | Menghapus tulisan lama, menulis ulang dengan data baru   |
| Tombol **Update**      | Menyerahkan formulir yang sudah diperbaiki ke petugas    |
| **Controller** (`update`) | **Petugas** yang menerima perubahan                   |
| **Model** `Product`    | Petugas arsip yang memperbarui data di lemari database   |
| **Database**           | Lemari arsip tempat data lama diganti dengan data baru   |

### Penting!

Pada tahap ini kita akan membuat **proses update** ke database. User akan benar-benar
bisa:

1. Membuka form edit.
2. Mengubah data.
3. Klik Update.
4. Perubahan tersimpan ke tabel `products`.
5. Lihat hasilnya di halaman detail produk.

Tahap ini adalah bagian **Update** dalam CRUD.

---

## 2. Tujuan Tahap Ini

Ketika user membuka halaman edit seperti:

```text
http://127.0.0.1:8000/products/1/edit
```

Lalu mengubah data produk dan menekan tombol:

```text
Update Produk
```

Maka sistem akan:

1. Mengirim data perubahan ke route PUT `/products/{id}`.
2. Menjalankan method `update()` di `ProductController`.
3. Mencari produk berdasarkan ID.
4. Memvalidasi data.
5. Mengupdate data produk di tabel `products`.
6. Mengarahkan user ke halaman detail produk.
7. Menampilkan pesan sukses.

Tahap ini adalah bagian **Update** dalam CRUD.

---

## 3. Alur Kerja Update Produk

```text
User membuka form edit produk
        |
        v
User mengubah data produk
        |
        v
User klik tombol Update Produk
        |
        v
Form mengirim data ke /products/{id}
        |
        v
Laravel membaca @method('PUT')
        |
        v
Route PUT /products/{id} menerima data
        |
        v
Route memanggil ProductController@update
        |
        v
Controller mencari produk berdasarkan ID
        |
        v
Controller memvalidasi data
        |
        v
Model Product mengupdate data di database
        |
        v
User diarahkan ke halaman detail produk
        |
        v
Muncul pesan sukses
```

Inilah alur update data dalam Laravel.

---

## 4. Mengecek Kembali Form Edit

Buka file:

```text
resources/views/products/edit.blade.php
```

Pastikan form edit memiliki bagian berikut:

```blade
<form action="/products/{{ $product->id }}" method="POST">
    @csrf
    @method('PUT')

    ...
</form>
```

### Penjelasan bagian penting

#### `action="/products/{{ $product->id }}"`

Form akan mengirim data ke alamat produk yang sedang diedit.

Contoh: jika produk memiliki ID 1, maka alamatnya menjadi:

```text
/products/1
```

#### `method="POST"`

HTML form biasa hanya mendukung GET dan POST. Karena itu, kita tetap menulis
`method="POST"` di tag `<form>`. Method sebenarnya (PUT) ditangani oleh `@method`.

#### `@method('PUT')`

Laravel menggunakan `@method('PUT')` untuk memberi tahu bahwa form ini sebenarnya
adalah permintaan **update data**, bukan tambah data baru.

**Analogi:** Form seperti amplop utama. `@method('PUT')` seperti catatan di dalam
amplop yang berkata:

> "Ini bukan tambah data baru, ini permintaan mengubah data lama."

#### `@csrf`

`@csrf` adalah pelindung keamanan bawaan Laravel agar form tidak disalahgunakan
oleh pihak luar (serangan CSRF).

---

## 5. Membuat Route PUT `/products/{id}`

Buka file:

```text
routes/web.php
```

Pastikan ada route berikut:

```php
Route::put('/products/{id}', [ProductController::class, 'update']);
```

### Susunan route sementara yang disarankan

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ProductController;

Route::get('/', function () {
    return view('welcome');
});

Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/create', [ProductController::class, 'create']);
Route::post('/products', [ProductController::class, 'store']);
Route::get('/products/{id}/edit', [ProductController::class, 'edit']);
Route::put('/products/{id}', [ProductController::class, 'update']);
Route::get('/products/{id}', [ProductController::class, 'show']);
```

### Penjelasan route

Route `Route::put('/products/{id}', [ProductController::class, 'update'])` berarti:

> Jika ada data dikirim ke `/products/{id}` menggunakan method PUT,
> Laravel akan menjalankan method `update()` di `ProductController`.

### Perhatikan: URL yang sama, method berbeda

| Route                | Method | Fungsi                       |
| -------------------- | ------- | ---------------------------- |
| `/products`          | GET     | Menampilkan daftar produk    |
| `/products/create`   | GET     | Menampilkan form tambah      |
| `/products`          | POST    | Menyimpan produk baru        |
| `/products/{id}`     | GET     | Menampilkan detail produk    |
| `/products/{id}/edit`| GET     | Menampilkan form edit        |
| `/products/{id}`     | PUT     | Mengupdate produk            |

URL `/products/{id}` bisa dipakai untuk **detail** dan **update** karena HTTP
method-nya berbeda:

- GET `/products/1` berarti melihat detail produk ID 1.
- PUT `/products/1` berarti mengupdate produk ID 1.

---

## 6. Membuat Method `update()` di ProductController

Buka file:

```text
app/Http/Controllers/ProductController.php
```

Tambahkan method `update()` di dalam class `ProductController`:

```php
public function update(Request $request, $id)
{
    $product = Product::findOrFail($id);

    $validated = $request->validate([
        'name'        => 'required',
        'description' => 'nullable',
        'price'       => 'required|integer|min:0',
        'stock'       => 'required|integer|min:0',
    ]);

    $product->update($validated);

    return redirect('/products/' . $product->id)->with('success', 'Produk berhasil diperbarui.');
}
```

Method `update()` adalah tempat menerima data perubahan dari form edit dan
menyimpannya ke database.

---

## 7. Contoh ProductController Lengkap Sementara

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    public function index()
    {
        $products = Product::all();

        return view('products.index', compact('products'));
    }

    public function create()
    {
        return view('products.create');
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'name'        => 'required',
            'description' => 'nullable',
            'price'       => 'required|integer|min:0',
            'stock'       => 'required|integer|min:0',
        ]);

        Product::create($validated);

        return redirect('/products')->with('success', 'Produk berhasil ditambahkan.');
    }

    public function show($id)
    {
        $product = Product::findOrFail($id);

        return view('products.show', compact('product'));
    }

    public function edit($id)
    {
        $product = Product::findOrFail($id);

        return view('products.edit', compact('product'));
    }

    public function update(Request $request, $id)
    {
        $product = Product::findOrFail($id);

        $validated = $request->validate([
            'name'        => 'required',
            'description' => 'nullable',
            'price'       => 'required|integer|min:0',
            'stock'       => 'required|integer|min:0',
        ]);

        $product->update($validated);

        return redirect('/products/' . $product->id)->with('success', 'Produk berhasil diperbarui.');
    }
}
```

### Penjelasan singkat setiap method

| Method       | Fungsi                                |
| ------------ | ------------------------------------- |
| `index()`    | Menampilkan daftar produk              |
| `create()`   | Menampilkan form tambah produk         |
| `store()`    | Menyimpan produk baru                  |
| `show()`     | Menampilkan detail satu produk         |
| `edit()`     | Menampilkan form edit produk           |
| `update()`   | **Menyimpan perubahan produk**         |

Belum ada method `destroy()` karena fitur hapus akan dibuat di tahap berikutnya.

---

## 8. Menjelaskan Request pada Update

Perhatikan baris method:

```php
public function update(Request $request, $id)
```

Method `update()` menerima **dua hal**:

1. `$request` berisi data dari form edit.
2. `$id` berisi ID produk dari URL.

### Analogi

- `$request` seperti **amplop** berisi data perubahan.
- `$id` seperti **nomor map produk** yang ingin diperbarui.

### Contoh

Jika user membuka:

```text
/products/3/edit
```

Lalu submit form, maka:

- `$id` bernilai `3`.
- `$request` berisi nama, deskripsi, harga, dan stok baru.

---

## 9. Menjelaskan `Product::findOrFail($id)`

```php
$product = Product::findOrFail($id);
```

Artinya: Laravel mencari produk berdasarkan ID.

- Jika produk **ditemukan**, data produk disimpan ke variabel `$product`.
- Jika produk **tidak ditemukan**, Laravel menampilkan halaman **404 Not Found**.

### Analogi

Petugas arsip mencari map produk berdasarkan nomor ID. Kalau map ada, map dibuka
dan diperbarui. Kalau map tidak ada, sistem memberi tahu bahwa data tidak ditemukan.

---

## 10. Menjelaskan Validasi Update

```php
$validated = $request->validate([
    'name'        => 'required',
    'description' => 'nullable',
    'price'       => 'required|integer|min:0',
    'stock'       => 'required|integer|min:0',
]);
```

Sebelum data lama diganti dengan data baru, Laravel memeriksa dulu apakah data
baru masuk akal dan sesuai aturan.

### Tabel aturan validasi

| Field        | Aturan    | Arti                                     |
| ------------ | --------- | ---------------------------------------- |
| `name`       | required  | Nama produk wajib diisi                   |
| `description`| nullable  | Deskripsi boleh kosong                    |
| `price`      | required  | Harga wajib diisi                         |
| `price`      | integer   | Harga harus angka bulat                   |
| `price`      | min:0     | Harga tidak boleh kurang dari 0           |
| `stock`      | required  | Stok wajib diisi                          |
| `stock`      | integer   | Stok harus angka bulat                    |
| `stock`      | min:0     | Stok tidak boleh kurang dari 0            |

### Analogi

Sebelum formulir lama diganti, petugas mengecek apakah formulir baru **lengkap**
dan **masuk akal**. Jika ada yang salah, form ditolak dan dikembalikan ke pengirim.

---

## 11. Menjelaskan `$product->update($validated)`

```php
$product->update($validated);
```

Artinya: data produk yang ditemukan akan diperbarui menggunakan data yang sudah
lolos validasi.

### Analogi

Petugas arsip membuka map produk lama, lalu mengganti isinya dengan data baru
yang sudah diperiksa.

### Penting: kode ini bekerja karena `$fillable`

Kode ini bisa bekerja karena pada **Tahap 4** kita sudah menambahkan `$fillable`
di Model `Product`.

Buka file:

```text
app/Models/Product.php
```

Pastikan isinya seperti ini:

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
    ];
}
```

`$fillable` adalah daftar kolom yang **boleh diisi atau diubah secara massal**.
Tanpa `$fillable`, perintah `$product->update($validated)` akan **mengabaikan**
semua perubahan demi keamanan.

---

## 12. Menjelaskan Redirect Setelah Update

```php
return redirect('/products/' . $product->id)->with('success', 'Produk berhasil diperbarui.');
```

Artinya: setelah produk berhasil diupdate, user diarahkan ke halaman **detail produk**.

### Contoh

Jika produk ID 1 berhasil diupdate, user diarahkan ke:

```text
/products/1
```

### Bagian `->with('success', '...')`

Laravel membawa **pesan sukses sementara** ke halaman detail produk. Pesan ini
disimpan di session dan otomatis hilang setelah ditampilkan sekali.

### Analogi

Setelah petugas memperbarui data, petugas memberi catatan kecil:

> "Produk berhasil diperbarui."

---

## 13. Menampilkan Pesan Sukses di Halaman Detail Produk

Buka file:

```text
resources/views/products/show.blade.php
```

Tambahkan kode berikut **di bawah judul halaman**:

```blade
@if (session('success'))
    <div style="padding: 10px; background: #d1e7dd; border: 1px solid #badbcc; color: #0f5132; margin-bottom: 15px;">
        {{ session('success') }}
    </div>
@endif
```

### Contoh posisi

```blade
<h1>Detail Produk</h1>

@if (session('success'))
    <div style="padding: 10px; background: #d1e7dd; border: 1px solid #badbcc; color: #0f5132; margin-bottom: 15px;">
        {{ session('success') }}
    </div>
@endif
```

### Penjelasan

Kode ini akan menampilkan pesan sukses **jika ada pesan** dari controller.
Jika tidak ada pesan, bagian ini tidak muncul.

---

## 14. Menampilkan Error Validasi di Form Edit

Buka file:

```text
resources/views/products/edit.blade.php
```

Tambahkan bagian ini **di atas form**:

```blade
@if ($errors->any())
    <div style="padding: 10px; background: #f8d7da; border: 1px solid #f5c2c7; color: #842029; margin-bottom: 15px;">
        <strong>Terjadi kesalahan:</strong>
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

### Penjelasan

Jika user mengisi data yang tidak sesuai, Laravel akan:

1. **Menolak** menyimpan data.
2. Mengembalikan user ke form edit.
3. Menampilkan **pesan error** di atas form.

### Contoh kesalahan yang akan ditolak

- Nama produk dikosongkan.
- Harga dikosongkan.
- Harga bernilai negatif.
- Stok bernilai negatif.

---

## 15. Menggunakan `old()` di Form Edit

### Masalah

Jika user salah mengisi form, halaman akan kembali ke form. Tanpa `old()`,
semua field akan **kembali ke data database** - artinya, perubahan yang sudah
diketik user **hilang** dan harus diketik ulang.

### Solusi: pakai `old()`

Pada form edit, gunakan `old()` dengan data produk sebagai **cadangan**.

### Untuk input nama

```blade
<input type="text" id="name" name="name" value="{{ old('name', $product->name) }}">
```

Artinya:

- Jika ada input lama dari user (saat validasi gagal), tampilkan input lama tersebut.
- Jika tidak ada (saat pertama kali buka form), tampilkan data produk dari database.

### Untuk deskripsi (textarea)

```blade
<textarea id="description" name="description" rows="4">{{ old('description', $product->description) }}</textarea>
```

### Untuk harga

```blade
<input type="number" id="price" name="price" value="{{ old('price', $product->price) }}">
```

### Untuk stok

```blade
<input type="number" id="stock" name="stock" value="{{ old('stock', $product->stock) }}">
```

### Analogi

Jika formulir ditolak karena ada kesalahan, isi yang baru diketik tidak langsung
dibuang. Petugas mengembalikan formulir yang sudah diketik sebagian, agar user
cukup memperbaiki bagian yang salah saja.

---

## 16. Contoh `edit.blade.php` Setelah Ditambahkan Error dan `old()`

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Edit Produk</title>

    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 40px;
            background: #f8f9fa;
            color: #222;
        }

        .card {
            background: #ffffff;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
            max-width: 600px;
        }

        .form-group {
            margin-bottom: 15px;
        }

        label {
            display: block;
            margin-bottom: 6px;
            font-weight: bold;
        }

        input,
        textarea {
            width: 100%;
            padding: 10px;
            border: 1px solid #bbb;
            border-radius: 6px;
            box-sizing: border-box;
        }

        button {
            padding: 10px 16px;
            background: #ffc107;
            color: #222;
            border: none;
            border-radius: 6px;
            cursor: pointer;
        }

        button:hover {
            background: #e0a800;
        }

        a {
            display: inline-block;
            margin-top: 12px;
            color: #0d6efd;
            text-decoration: none;
        }

        .links {
            margin-top: 15px;
        }

        .error-box {
            padding: 10px;
            background: #f8d7da;
            border: 1px solid #f5c2c7;
            color: #842029;
            margin-bottom: 15px;
            border-radius: 6px;
        }
    </style>
</head>
<body>
    <div class="card">
        <h1>Edit Produk</h1>

        <p>Ubah data produk pada form berikut.</p>

        @if ($errors->any())
            <div class="error-box">
                <strong>Terjadi kesalahan:</strong>
                <ul>
                    @foreach ($errors->all() as $error)
                        <li>{{ $error }}</li>
                    @endforeach
                </ul>
            </div>
        @endif

        <form action="/products/{{ $product->id }}" method="POST">
            @csrf
            @method('PUT')

            <div class="form-group">
                <label for="name">Nama Produk</label>
                <input type="text" id="name" name="name" value="{{ old('name', $product->name) }}">
            </div>

            <div class="form-group">
                <label for="description">Deskripsi Produk</label>
                <textarea id="description" name="description" rows="4">{{ old('description', $product->description) }}</textarea>
            </div>

            <div class="form-group">
                <label for="price">Harga Produk</label>
                <input type="number" id="price" name="price" value="{{ old('price', $product->price) }}">
            </div>

            <div class="form-group">
                <label for="stock">Stok Produk</label>
                <input type="number" id="stock" name="stock" value="{{ old('stock', $product->stock) }}">
            </div>

            <button type="submit">Update Produk</button>
        </form>

        <div class="links">
            <a href="/products/{{ $product->id }}">Kembali ke Detail Produk</a>
            <br>
            <a href="/products">Kembali ke Daftar Produk</a>
        </div>
    </div>
</body>
</html>
```

### Penjelasan

Sekarang form edit lebih nyaman digunakan karena:

- Bisa **menampilkan pesan error** saat validasi gagal.
- Bisa **mempertahankan input lama** yang sudah diketik user.
- Tetap **menampilkan data produk** dari database saat pertama kali dibuka.

---

## 17. Mengecek Proses Update Produk

Lakukan langkah berikut:

1. Jalankan server Laravel:

```bash
php artisan serve
```

2. Buka halaman daftar produk:

```text
http://127.0.0.1:8000/products
```

3. Klik link **Edit** pada salah satu produk.

4. Ubah data produk, misalnya:

```text
Nama Produk: Kaos Hitam Premium
Deskripsi:   Kaos bahan cotton combed premium
Harga:       120000
Stok:        25
```

5. Klik tombol **Update Produk**.

6. Pastikan diarahkan ke halaman detail produk.

7. Pastikan muncul pesan:

```text
Produk berhasil diperbarui.
```

8. Pastikan data produk sudah berubah sesuai input.

---

## 18. Mengecek Data di Database

Cara mengecek langsung:

1. Buka phpMyAdmin / Adminer / TablePlus / database client.
2. Pilih database yang digunakan.
3. Buka tabel `products`.
4. Cari produk yang tadi diupdate.
5. Pastikan data sudah berubah.

Jika data berubah di halaman detail **dan juga berubah di tabel database**,
berarti fitur update berhasil.

---

## 19. Menguji Validasi Update

Coba skenario ini:

1. Buka halaman edit produk.
2. Kosongkan nama produk.
3. Isi harga dengan angka negatif, misalnya:

```text
-1000
```

4. Klik tombol **Update Produk**.

### Hasil yang diharapkan

- Laravel **tidak menyimpan** data.
- Laravel **mengembalikan** user ke form edit.
- Laravel **menampilkan pesan error**:
  - "The name field is required."
  - "The price field must be at least 0."
- Input yang sudah diketik user **tetap ada** berkat `old()`.

Ini berarti validasi dasar sudah bekerja.

---

## 20. Masalah Umum dan Solusinya

### Error: `The PUT method is not supported for route products/{id}`

**Penyebab:** Route PUT `/products/{id}` belum dibuat.

**Solusi:** Pastikan di `routes/web.php` ada:

```php
Route::put('/products/{id}', [ProductController::class, 'update']);
```

### Error: `Method App\Http\Controllers\ProductController::update does not exist`

**Penyebab:** Method `update()` belum dibuat di controller.

**Solusi:** Tambahkan method `update()` di `ProductController`.

### Error: `Class "Product" not found`

**Penyebab:** Model `Product` belum di-import di controller.

**Solusi:** Pastikan di bagian atas controller ada:

```php
use App\Models\Product;
```

### Error: `Add fillable property to allow mass assignment`

**Penyebab:** Model `Product` belum memiliki `$fillable`.

**Solusi:** Pastikan `app/Models/Product.php` berisi:

```php
protected $fillable = [
    'name',
    'description',
    'price',
    'stock',
];
```

### Setelah klik Update, kembali ke form dan muncul error

**Penyebab:** Validasi gagal.

**Solusi:** Pastikan:

- Nama produk diisi.
- Harga diisi angka.
- Harga tidak negatif.
- Stok diisi angka.
- Stok tidak negatif.

### Data tidak berubah setelah update

**Kemungkinan penyebab:**

- Route PUT belum benar.
- Form belum memiliki `@method('PUT')`.
- Method `update()` belum memanggil `$product->update($validated)`.
- Field input tidak memiliki atribut `name` yang benar.

**Solusi:** Cek kembali:

- `routes/web.php`
- `edit.blade.php`
- `ProductController`
- `Product.php`

---

## 21. Struktur File Sampai Tahap Ini

```text
database/
  migrations/
    xxxx_xx_xx_xxxxxx_create_products_table.php

app/
  Models/
    Product.php
  Http/
    Controllers/
      ProductController.php

routes/
  web.php

resources/
  views/
    products/
      index.blade.php
      create.blade.php
      show.blade.php
      edit.blade.php
```

### Fungsi masing-masing file pada proses update

| File               | Fungsi dalam proses update                          |
| ------------------ | --------------------------------------------------- |
| `edit.blade.php`   | Mengirim data perubahan                              |
| `routes/web.php`   | Route PUT menerima permintaan update                 |
| `ProductController`| Method `update()` memproses perubahan               |
| `Product.php`      | Model mengupdate data di tabel `products`            |
| `show.blade.php`   | Menampilkan hasil update + pesan sukses              |

---

## 22. Apa yang Sudah Berhasil Dibuat?

| Bagian CRUD         | Status       |
| ------------------- | ------------ |
| Create              | Sudah dibuat |
| Read daftar produk  | Sudah dibuat |
| Read detail produk  | Sudah dibuat |
| Form Update/Edit    | Sudah dibuat |
| Proses Update       | Sudah dibuat |
| Delete              | Belum dibuat |

Sekarang user sudah bisa:

- Menambahkan produk.
- Melihat daftar produk.
- Melihat detail produk.
- Membuka form edit produk.
- **Menyimpan perubahan produk.**

---

## 23. Apa yang Belum Dilakukan?

Sampai tahap ini kita **belum** membuat:

- Fitur hapus produk.
- Route DELETE `/products/{id}`.
- Method `destroy()`.
- Tombol hapus produk.
- Konfirmasi sebelum hapus.
- Pagination.
- Search produk.
- Upload gambar produk.
- Layout Blade reusable.

Tahap ini hanya fokus pada:

- Menerima data update dari form edit.
- Memvalidasi data update.
- Mengupdate data produk di database.
- Menampilkan pesan sukses setelah update.

---

## 24. Checklist Keberhasilan

Centang (`x`) jika sudah selesai:

- [ ] Saya tahu alur update produk.
- [ ] Saya tahu fungsi route PUT `/products/{id}`.
- [ ] Saya tahu kenapa form edit memakai `method="POST"` dan `@method('PUT')`.
- [ ] Saya sudah membuat route PUT `/products/{id}`.
- [ ] Saya sudah membuat method `update()` di `ProductController`.
- [ ] Saya tahu fungsi `Product::findOrFail($id)`.
- [ ] Saya tahu fungsi `$request->validate()`.
- [ ] Saya tahu fungsi `$product->update($validated)`.
- [ ] Saya sudah memastikan `$fillable` ada di Model `Product`.
- [ ] Saya bisa mengubah data produk dari form edit.
- [ ] Saya bisa melihat pesan sukses setelah update.
- [ ] Saya bisa melihat data produk berubah di halaman detail.
- [ ] Saya belum membuat fitur hapus produk pada tahap ini.

Jika semua sudah tercentang, fitur **Update** sudah berfungsi penuh.

---

## 25. Kesimpulan Tahap 12

Pada tahap ini kita sudah berhasil membuat **proses update produk**.

### Ringkasan perjalanan belajar (dengan analogi)

| Tahap  | Yang kita buat                                   |
| ------ | ------------------------------------------------ |
| Tahap 3 | Membuat **lemari arsip** produk (migration + tabel)|
| Tahap 4 | Membuat **petugas arsip** bernama Model Product    |
| Tahap 5 | Membuat **alamat halaman** produk (route)          |
| Tahap 6 | Membuat **manajer** bernama ProductController      |
| Tahap 7 | Membuat **etalase** daftar produk (Read daftar)    |
| Tahap 8 | Membuat **formulir tambah** produk (Form Create)   |
| Tahap 9 | Membuat **proses menyimpan** produk baru (Create)  |
| Tahap 10| Membuat **kartu detail** produk (Read detail)      |
| Tahap 11| Membuat **formulir edit** berisi data lama (Form Edit)|
| **Tahap 12** | **Membuat proses update data di database** (Update) |

### Jika sudah paham dan produk berhasil diupdate, ketik:

```text
lanjut
```

Jangan lanjut ke tahap 13 sebelum kamu meminta. Pelan-pelan, asal paham.
