# Tahap 7 — Menampilkan Daftar Produk

> **Tujuan tahap ini:** Menampilkan daftar produk dari database ke halaman web menggunakan Model, Controller, dan View Blade. **Belum membuat form tambah/edit/hapus.** Hanya menampilkan data (Read).

---

## 1. Pengantar Sederhana

Pada tahap-tahap sebelumnya, kita sudah membuat:

- **Migration** untuk tabel `products` (Tahap 3).
- **Model** `Product` sebagai penghubung ke tabel `products` (Tahap 4).
- **Route** `/products` (Tahap 5).
- **Controller** `ProductController` dengan method `index` (Tahap 6).

Saat ini, saat user membuka `/products`, controller hanya mengembalikan teks
polos. Belum ada halaman cantik, belum ada data dari database.

Di tahap ini, kita akan mulai **menampilkan data produk ke halaman web**.

### Analogi: Alur Toko Lengkap

| Bagian Laravel   | Peran di Toko                              |
| ---------------- | ------------------------------------------ |
| **Database**     | Lemari arsip                               |
| **Model**        | Petugas arsip                              |
| **Controller**   | Manajer toko                               |
| **View**         | Etalase toko                               |
| **Browser/User** | Pengunjung yang melihat etalase            |

### Alur yang akan kita bangun

1. Pengunjung datang ke toko (User membuka URL `/products`).
2. Resepsionis (Route) menyambut, arahkan ke manajer (Controller).
3. Manajer (Controller) meminta data ke petugas arsip (Model).
4. Petugas arsip (Model) mengambil semua produk dari lemari arsip (Database).
5. Manajer menyerahkan data ke tim etalase (View).
6. Tim etalase menata produk secara rapi di etalase (halaman HTML).
7. Pengunjung melihat hasilnya di browser.

Jadi, sekarang alurnya bukan lagi hanya menampilkan teks biasa, tetapi
**menampilkan halaman daftar produk** yang sebenarnya, dengan data dari database.

---

## 2. Tujuan Tahap Ini

Saat user membuka alamat:

```text
http://127.0.0.1:8000/products
```

Maka browser akan menampilkan halaman seperti ini:

| ID | Nama Produk      | Harga    | Stok |
| -- | ---------------- | -------- | ---- |
| 1  | Kaos Hitam       | 100000   | 20   |
| 2  | Celana Jeans     | 250000   | 15   |
| 3  | Sepatu Putih     | 350000   | 10   |

Tapi karena database kita masih kosong, halaman akan menampilkan tabel kosong
dulu, atau pesan "Belum ada produk." Di tahap selanjutnya kita akan menambah
produk lewat form.

---

## 3. Langkah 1: Mengisi Database dengan Data Contoh

Karena tabel `products` masih kosong, kita perlu mengisi beberapa produk
sebagai contoh supaya halaman daftar tidak kosong.

### Cara cepat: Lewat Tinker

Buka terminal di folder project, jalankan:

```bash
php artisan tinker
```

Di dalam Tinker (`>>>`), ketik baris-baris berikut satu per satu:

```php
>>> Product::create(['name' => 'Kaos Hitam', 'description' => 'Kaos cotton combed', 'price' => 100000, 'stock' => 20]);
>>> Product::create(['name' => 'Celana Jeans', 'description' => 'Jeans slim fit', 'price' => 250000, 'stock' => 15]);
>>> Product::create(['name' => 'Sepatu Putih', 'description' => 'Sneakers putih', 'price' => 350000, 'stock' => 10]);
>>> Product::all();
>>> exit
```

Pastikan baris `use App\Models\Product;` tidak diperlukan di Tinker karena
Tinker modern sudah autoload model. Jika error "Class Product not found",
ketik dulu:

```php
>>> use App\Models\Product;
```

Setelah `Product::all()`, kamu akan melihat 3 produk yang baru saja dibuat.
Sekarang tabel `products` sudah berisi data contoh.

> Catatan: Ini hanya untuk keperluan belajar. Di aplikasi sungguhan, penambahan
> produk dilakukan lewat form (yang akan kita buat di tahap selanjutnya).

---

## 4. Langkah 2: Mengubah Method `index()` di Controller

Saat ini method `index()` di `ProductController` hanya mengembalikan teks.
Kita akan ubah agar:

1. Mengambil semua produk dari Model.
2. Mengirim data tersebut ke View.

### Bukka file `app/Http/Controllers/ProductController.php`

Ubah method `index()` menjadi seperti ini:

```php
public function index()
{
    $products = Product::all();
    return view('products.index', compact('products'));
}
```

### Jangan lupa panggil Model

Di bagian atas file controller, tambahkan baris `use`:

```php
use App\Models\Product;
```

### Kode controller lengkap (bagian atas dan method index)

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

    // ... method lain tetap seperti semula (create, store, show, dll.)
}
```

### Penjelasan tiap baris

#### `$products = Product::all();`

Artinya: "Petugas arsip (Model `Product`), tolong ambilkan **semua** produk
dari lemari arsip."

Hasilnya disimpan di variabel `$products`. Variabel ini berisi **koleksi**
(collection) berisi banyak objek produk.

#### `return view('products.index', compact('products'));`

Artinya: "Tampilkan View yang ada di `resources/views/products/index.blade.php`,
dan kirimkan variabel `$products` agar bisa dipakai di View."

#### `compact('products')`

Fungsi `compact()` adalah cara PHP untuk mengirim variabel ke View.
`compact('products')` artinya kirim variabel `$products` ke View.
Nama di `compact` **tanpa tanda `$`**, hanya nama variabel.

#### `'products.index'`

Ini adalah cara Laravel merujuk ke file View:

- `products` = nama folder di `resources/views/`.
- `index` = nama file Blade (tanpa `.blade.php`).
- Jadi, lengkapnya: `resources/views/products/index.blade.php`.

### Analogi: Manajer Mengatur Etalase

- Manajer (Controller) meminta data produk ke petugas arsip (`Product::all()`).
- Manajer menyerahkan data ke tim etalase (`view('products.index', ...)`).
- Tim etalase menata produk di etalase (file `index.blade.php`).

---

## 5. Langkah 3: Membuat View Blade

Sekarang controller sudah mengirim data ke View bernama `products.index`,
tapi View-nya belum ada. Kita perlu membuatnya.

### Lokasi file View

Di Laravel, semua file View disimpan di folder:

```
resources/views/
```

Kita akan membuat folder baru bernama `products` di dalamnya, lalu file
`index.blade.php` di dalam folder tersebut.

### Struktur folder yang akan dibuat

```
resources/
  views/
    products/
      index.blade.php   <-- file yang akan kita buat
```

### Bukka file `resources/views/products/index.blade.php`

Buat folder `products` di dalam `resources/views/`, lalu buat file
`index.blade.php` dengan kode berikut:

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Daftar Produk</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        table {
            border-collapse: collapse;
            width: 100%;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px 12px;
            text-align: left;
        }
        th {
            background-color: #f2f2f2;
        }
        h1 {
            color: #333;
        }
    </style>
</head>
<body>
    <h1>Daftar Produk</h1>

    @if ($products->isEmpty())
        <p>Belum ada produk.</p>
    @else
        <table>
            <thead>
                <tr>
                    <th>ID</th>
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
                        <td>{{ $product->name }}</td>
                        <td>{{ $product->description }}</td>
                        <td>Rp {{ number_format($product->price, 0, ',', '.') }}</td>
                        <td>{{ $product->stock }}</td>
                    </tr>
                @endforeach
            </tbody>
        </table>
    @endif
</body>
</html>
```

### Penjelasan bagian penting View

#### `@if ($products->isEmpty())`

Ini adalah **direktif Blade** untuk pengecekan kondisi. Blade adalah mesin
template Laravel yang memungkinkan kita menyisipkan logika PHP di dalam HTML.

`$products->isEmpty()` mengecek apakah koleksi produk kosong.
Jika ya, tampilkan pesan "Belum ada produk."

#### `@foreach ($products as $product)`

Perulangan untuk menampilkan setiap produk sebagai satu baris tabel.

Setiap kali perulangan berjalan, variabel `$product` berisi **satu produk**.
Kita akses atributnya pakai `->` (panah):

- `$product->id`
- `$product->name`
- `$product->price`
- dst.

#### `{{ $product->name }}`

Tanda `{{ ... }}` adalah cara Blade untuk **menampilkan** nilai variabel ke HTML.
Ini juga otomatis aman dari serangan XSS karena Blade akan **escape** karakter
berbahaya.

#### `number_format($product->price, 0, ',', '.')`

Fungsi PHP untuk memformat angka menjadi lebih mudah dibaca:

- `100000` -> `100.000` (dengan titik sebagai pemisah ribuan).

Hasilnya misal: "Rp 100.000".

#### Penutup direktif

Setiap `@if` harus ditutup dengan `@endif`.
Setiap `@foreach` harus ditutup dengan `@endforeach`.

---

## 6. Langkah 4: Menjalankan dan Menguji

### Jalankan server

```bash
php artisan serve
```

### Buka browser ke alamat:

```text
http://127.0.0.1:8000/products
```

### Hasil yang diharapkan

Kamu akan melihat halaman "Daftar Produk" dengan tabel berisi 3 produk yang
tadi kita buat lewat Tinker:

| ID | Nama           | Deskripsi              | Harga     | Stok |
| -- | -------------- | ---------------------- | --------- | ---- |
| 1  | Kaos Hitam     | Kaos cotton combed     | Rp 100.000| 20   |
| 2  | Celana Jeans   | Jeans slim fit         | Rp 250.000| 15   |
| 3  | Sepatu Putih   | Sneakers putih         | Rp 350.000| 10   |

### Jika database kosong

Jika kamu belum mengisi data lewat Tinker, halaman akan menampilkan:

> "Belum ada produk."

Itu wajar, karena tabel `products` kosong.

---

## 7. Apa yang Terjadi di Balik Layar?

Mari kita telusuri kembali alurnya secara lengkap:

```
1. User buka browser ke http://127.0.0.1:8000/products
        |
        v
2. Route::get('/products', [ProductController::class, 'index'])
   Route menerima permintaan, arahkan ke Controller
        |
        v
3. ProductController::index()
   Method index dijalankan
        |
        v
4. $products = Product::all();
   Model Product mengambil semua produk dari database
        |
        v
5. return view('products.index', compact('products'));
   Controller mengirim data ke View
        |
        v
6. resources/views/products/index.blade.php
   View merender HTML dengan data produk
        |
        v
7. Browser menerima HTML, tampilkan tabel produk
```

Inilah alur MVC lengkap yang sebenarnya:

```
User -> Route -> Controller -> Model -> Database
                                     |
User <- View    <- Controller <- Model
```

---

## 8. Hal-hal yang Perlu Diperhatikan

### 1. Pastikan variabel dikirim ke View

Di controller, kita memakai `compact('products')`. Jika lupa mengirim variabel,
View akan error karena `$products` tidak dikenali.

Cek: method `index()` harus mengembalikan View **dengan** data `$products`.

### 2. Pastikan nama View benar

Di controller: `view('products.index', ...)` harus cocok dengan file:
`resources/views/products/index.blade.php`.

- Folder: `products`
- File: `index.blade.php`
- Dipisah dengan titik: `products.index`

### 3. Format harga

Kolom `price` disimpan sebagai integer (misal `100000`). Tampilan akan lebih
ramah jika diformat: `Rp 100.000`. Fungsi `number_format()` digunakan untuk ini.

### 4. Atribut produk di View

Kita mengakses atribut produk dengan panah (`->`):

- `$product->id` -> kolom `id`
- `$product->name` -> kolom `name`
- `$product->price` -> kolom `price`

Ini sama persis dengan nama kolom di migration.

---

## 9. Ringkasan Alur Tahap Ini

```
1. (Opsional) Isi data contoh via php artisan tinker
        |
        v
2. Edit ProductController::index()
   - $products = Product::all();
   - return view('products.index', compact('products'));
        |
        v
3. Tambahkan 'use App\Models\Product;' di atas controller
        |
        v
4. Buat file resources/views/products/index.blade.php
   - Tabel HTML
   - @foreach untuk looping data
        |
        v
5. php artisan serve, buka /products
        |
        v
6. Daftar produk tampil di browser
```

---

## 10. Checklist Keberhasilan Tahap Ini

Centang (`x`) jika sudah selesai:

- [ ] Saya sudah mengisi beberapa produk contoh lewat Tinker (opsional).
- [ ] Method `index()` di controller mengambil data dengan `Product::all()`.
- [ ] Controller mengembalikan `view('products.index', compact('products'))`.
- [ ] Saya sudah membuat file `resources/views/products/index.blade.php`.
- [ ] Halaman `/products` menampilkan tabel produk dari database.
- [ ] Halaman menampilkan pesan "Belum ada produk" jika database kosong.
- [ ] Saya paham alur: Route -> Controller -> Model -> View.

Jika semua sudah tercentang, fitur **Read (menampilkan daftar)** sudah selesai.

---

## 11. Penutup

Selamat! Kamu sudah berhasil:

- Mengambil data dari database lewat Model.
- Mengirim data dari Controller ke View.
- Menampilkan data ke halaman web dengan tabel HTML + Blade.
- Menerapkan alur MVC lengkap untuk pertama kalinya.

Ini adalah **fitur pertama CRUD** yang benar-benar berfungsi: **Read (menampilkan daftar)**.

Di **tahap berikutnya**, kita akan belajar cara **menambah produk baru** lewat
form. Jadi user tidak perlu lagi pakai Tinker - cukup isi form di browser,
lalu produk otomatis tersimpan ke database.

### Jika sudah paham tahap ini, ketik:

```
lanjut
```

Jangan lanjut sebelum kamu benar-benar siap. Pelan-pelan, asal paham.
