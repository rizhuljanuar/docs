# Tahap 8 — Membuat Form Tambah Produk

> **Tujuan tahap ini:** Membuat halaman form tambah produk (tampilan saja). **Belum menyimpan data ke database.** Proses penyimpanan (POST) akan dibuat di tahap berikutnya.

---

## 1. Pengantar Sederhana

Pada **Tahap 7**, kita sudah berhasil **menampilkan daftar produk** dari database
ke halaman web. Tapi sampai sekarang, satu-satunya cara menambah produk adalah
lewat **Tinker**, yang tentu tidak praktis untuk pengguna sungguhan.

Sekarang kita akan membuat **halaman form tambah produk** - yaitu form kosong
yang bisa diisi oleh admin atau user untuk memasukkan produk baru.

### Analogi: Etalase dan Formulir

| Hal             | Analogi                                      |
| --------------- | --------------------------------------------- |
| Halaman daftar produk | Etalase toko (tempat produk dipajang)   |
| **Form tambah produk** | **Formulir kosong** yang diisi admin ketika ada produk baru |

Admin membuka form, lalu mengisi:

- Nama produk
- Deskripsi produk
- Harga
- Stok

### Penting!

Pada tahap ini, form **hanya dibuat tampilannya dulu**.

- Data **belum disimpan** ke database.
- Tombol "Simpan" belum berfungsi nyata.
- Proses penyimpanan akan dibuat pada **tahap berikutnya**.

Tujuannya supaya kita fokus pada satu hal: cara membuat form yang benar di Laravel.

---

## 2. Tujuan Tahap Ini

Saat user membuka alamat:

```text
http://127.0.0.1:8000/products/create
```

Maka browser akan menampilkan halaman form seperti ini:

```text
+--------------------------------------+
|  Tambah Produk                       |
|                                      |
|  Nama:        [____________________] |
|  Deskripsi:   [____________________] |
|               [____________________] |
|  Harga:       [____________________] |
|  Stok:        [____________________] |
|                                      |
|  [ Simpan ]                          |
+--------------------------------------+
```

Selain itu, di halaman daftar produk (`/products`), kita akan menambahkan
tombol/link **"Tambah Produk"** yang mengarah ke form ini.

---

## 3. Langkah 1: Mengubah Method `create()` di Controller

Saat ini method `create()` di `ProductController` masih mengembalikan teks polos.
Kita ubah agar menampilkan View form.

### Bukka file `app/Http/Controllers/ProductController.php`

Ubah method `create()` menjadi:

```php
public function create()
{
    return view('products.create');
}
```

### Penjelasan

- `view('products.create')` artinya: tampilkan View yang ada di
  `resources/views/products/create.blade.php`.
- Karena form tambah produk **masih kosong** (belum ada data lama yang perlu
  ditampilkan), kita tidak perlu mengirim variabel apa pun ke View.

### Kode controller lengkap (bagian method create)

```php
public function create()
{
    return view('products.create');
}
```

Method lain tetap seperti semula. Jangan diubah dulu.

---

## 4. Langkah 2: Membuat View Form Tambah Produk

Sekarang kita buat file View untuk form tambah produk.

### Lokasi file

Buat file baru di:

```
resources/views/products/create.blade.php
```

Struktur folder menjadi:

```
resources/
  views/
    products/
      index.blade.php    <-- sudah ada (Tahap 7)
      create.blade.php   <-- file baru yang dibuat sekarang
```

### Kode `create.blade.php`

Isi file `resources/views/products/create.blade.php` dengan kode berikut:

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tambah Produk</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input[type="text"],
        input[type="number"],
        textarea {
            width: 100%;
            max-width: 400px;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
        }
        textarea {
            height: 80px;
            resize: vertical;
        }
        button {
            background-color: #28a745;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover {
            background-color: #218838;
        }
        a {
            text-decoration: none;
            color: #007bff;
        }
        h1 {
            color: #333;
        }
    </style>
</head>
<body>
    <h1>Tambah Produk</h1>

    <!--
        Catatan: pada tahap ini form belum benar-benar menyimpan data.
        Atribut action dan method akan diatur di tahap berikutnya
        saat kita membuat fitur simpan produk (POST /products).
    -->
    <form action="#" method="POST">
        @csrf

        <div class="form-group">
            <label for="name">Nama Produk</label>
            <input type="text" name="name" id="name" placeholder="Contoh: Kaos Hitam">
        </div>

        <div class="form-group">
            <label for="description">Deskripsi</label>
            <textarea name="description" id="description" placeholder="Contoh: Kaos bahan cotton combed"></textarea>
        </div>

        <div class="form-group">
            <label for="price">Harga (Rp)</label>
            <input type="number" name="price" id="price" placeholder="Contoh: 100000" min="0">
        </div>

        <div class="form-group">
            <label for="stock">Stok</label>
            <input type="number" name="stock" id="stock" placeholder="Contoh: 20" min="0">
        </div>

        <button type="submit">Simpan</button>
    </form>

    <p style="margin-top: 20px;">
        <a href="/products">&larr; Kembali ke Daftar Produk</a>
    </p>
</body>
</html>
```

### Penjelasan bagian penting Form

#### `<form action="#" method="POST">`

- `action="#"` artinya form belum diarahkan ke URL pemrosesan manapun.
  Ini hanya placeholder. Di **tahap berikutnya**, kita akan ganti `#` dengan
  URL tujuan yang benar (yaitu `action="/products"`).
- `method="POST"` artinya form akan mengirim data dengan metode POST.
  POST adalah method yang tepat untuk **mengirim data baru** ke server,
  bukan GET (yang hanya untuk melihat).

#### `@csrf`

Ini adalah **direktif Blade** yang menghasilkan **token keamanan** CSRF.

CSRF (Cross-Site Request Forgery) adalah serangan di mana pihak jahat
mencoba mengirim form palsu ke aplikasi kamu. Laravel melindungi dari serangan
ini dengan **token CSRF** - kode rahasia yang harus disertakan di setiap form.

`@csrf` wajib ada di **setiap form POST/PUT/DELETE** di Laravel.
Tanpa `@csrf`, Laravel akan menolak form dengan error `419 Page Expired`.

> Catatan: Untuk form yang "belum berfungsi" seperti sekarang, `@csrf` tetap
> harus ada karena Laravel mengharuskannya pada semua form POST.

#### `<input type="text" name="name" ...>`

- `name="name"` adalah **atribut penting**. Nama inilah yang nanti dipakai
  server untuk mengambil nilai yang diisi user.
- Atribut `name` harus cocok dengan **nama kolom** di tabel `products`:
  `name`, `description`, `price`, `stock`.

#### `<input type="number" name="price" ...>`

- `type="number"` membuat browser hanya menerima input angka.
- Cocok untuk kolom harga dan stok.
- `min="0"` memastikan tidak bisa input angka negatif.

#### `<textarea name="description" ...>`

- Untuk teks panjang seperti deskripsi.
- Berbeda dengan `<input>`, `<textarea>` punya tag penutup dan bisa multi-baris.

#### `<button type="submit">Simpan</button>`

- Tombol untuk mengirim form.
- `type="submit"` artinya klik tombol ini = kirim form ke server.
- **Saat ini tombol belum benar-benar menyimpan** karena `action="#"`.

#### `<a href="/products">...Kembali...</a>`

- Link untuk kembali ke halaman daftar produk.

---

## 5. Langkah 3: Pastikan Route `/products/create` Sudah Ada

Pada **Tahap 6**, kita sudah membuat route:

```php
Route::get('/products/create', [ProductController::class, 'create']);
```

Pastikan baris ini **masih ada** di file `routes/web.php`. Jika sudah ada,
tidak perlu menambah apa pun. Route inilah yang membuat URL `/products/create`
menampilkan halaman form.

### Urutan route penting!

Pastikan route `/products/create` ditulis **sebelum** route `/products/{id}`:

```php
// Benar: spesifik dulu
Route::get('/products/create', [ProductController::class, 'create']);
Route::get('/products/{id}', [ProductController::class, 'show']);
```

Jika terbalik, saat user buka `/products/create`, Laravel akan mengira
`create` adalah nilai `$id`, dan halaman form tidak terbuka.

---

## 6. Langkah 4: Menambahkan Tombol "Tambah Produk" di Halaman Daftar

Agar user bisa pindah dari halaman daftar produk ke halaman form, kita
tambahkan link di `index.blade.php`.

### Bukka file `resources/views/products/index.blade.php`

Di bagian atas (setelah `<h1>Daftar Produk</h1>`), tambahkan link:

```blade
<h1>Daftar Produk</h1>

<p>
    <a href="/products/create" style="
        display: inline-block;
        background-color: #28a745;
        color: white;
        padding: 8px 16px;
        text-decoration: none;
        border-radius: 4px;
    ">
        + Tambah Produk
    </a>
</p>
```

Sekarang halaman daftar produk punya tombol hijau "Tambah Produk" yang
jika diklik akan mengarah ke `/products/create`.

### Kode `index.blade.php` lengkap (dengan tombol tambah)

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

    <p>
        <a href="/products/create" style="
            display: inline-block;
            background-color: #28a745;
            color: white;
            padding: 8px 16px;
            text-decoration: none;
            border-radius: 4px;
        ">
            + Tambah Produk
        </a>
    </p>

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

---

## 7. Langkah 5: Menjalankan dan Menguji

### Jalankan server

```bash
php artisan serve
```

### Tes halaman form

Buka browser ke:

```text
http://127.0.0.1:8000/products/create
```

Kamu akan melihat form "Tambah Produk" dengan field:

- Nama Produk
- Deskripsi
- Harga (Rp)
- Stok

Tombol "Simpan" dan link "Kembali ke Daftar Produk".

### Tes navigasi

1. Buka `http://127.0.0.1:8000/products` (halaman daftar).
2. Klik tombol hijau **"+ Tambah Produk"**.
3. Browser pindah ke halaman form.
4. Di halaman form, klik **"Kembali ke Daftar Produk"**.
5. Browser kembali ke halaman daftar.

### Coba isi form

Kamu bisa mencoba mengetik di field-field form. Tapi saat klik "Simpan",
**tidak akan terjadi apa-apa** (atau mungkin error, karena `action="#"`
dan route POST belum dibuat).

Itu wajar. Di **tahap berikutnya** kita akan membuat proses penyimpanan
sehingga tombol Simpan benar-benar menyimpan produk ke database.

---

## 8. Gambaran Tahap Berikutnya (Bukan untuk Sekarang)

Hanya sekadar gambaran, di tahap berikutnya kita akan:

1. Mengubah `action="#"` di form menjadi `action="/products"` (URL tujuan).
2. Membuat route `Route::post('/products', [ProductController::class, 'store']);`
3. Mengisi method `store()` di controller untuk:
   - Mengambil data dari form (`$request`).
   - Menyimpan ke database via Model (`Product::create([...])`).
   - Redirect kembali ke halaman daftar produk.

Tapi **semua itu belum dilakukan sekarang**. Sekarang fokus pada form saja.

---

## 9. Hal-hal yang Perlu Diperhatikan

### 1. Atribut `name` di input harus sesuai kolom

| Input       | Atribut `name`   | Cocok dengan kolom |
| ------------| ----------------- | ------------------ |
| Nama        | `name="name"`     | `name`             |
| Deskripsi   | `name="description"` | `description`   |
| Harga       | `name="price"`    | `price`            |
| Stok        | `name="stock"`    | `stock`            |

Jika `name` tidak sesuai kolom, data tidak akan tersimpan dengan benar di tahap penyimpanan.

### 2. `@csrf` wajib di setiap form POST

Tanpa `@csrf`, Laravel akan menolak form dengan error `419 Page Expired`.
Ini fitur keamanan bawaan Laravel.

### 3. `method="POST"` untuk form tambah data

Gunakan **POST** untuk membuat data baru, bukan GET. GET hanya untuk melihat.

### 4. `action` di form

- `action="#"` = form belum diarahkan ke mana pun (placeholder).
- `action="/products"` = form dikirim ke URL `/products` (akan dipakai di tahap berikutnya).

---

## 10. Ringkasan Alur Tahap Ini

```
1. Ubah method create() di controller
   - return view('products.create');
        |
        v
2. Buat file resources/views/products/create.blade.php
   - Form HTML dengan field: name, description, price, stock
   - @csrf untuk keamanan
   - action="#" dulu (belum menyimpan)
        |
        v
3. Pastikan route /products/create sudah ada
        |
        v
4. Tambah tombol "Tambah Produk" di halaman daftar produk
        |
        v
5. php artisan serve, tes buka /products/create
        |
        v
6. Form tampil, tapi belum menyimpan data
```

---

## 11. Checklist Keberhasilan Tahap Ini

Centang (`x`) jika sudah selesai:

- [ ] Method `create()` di controller mengembalikan `view('products.create')`.
- [ ] File `resources/views/products/create.blade.php` sudah dibuat.
- [ ] Form punya field: `name`, `description`, `price`, `stock`.
- [ ] Form memakai `method="POST"` dan `@csrf`.
- [ ] Halaman `/products/create` menampilkan form tambah produk.
- [ ] Halaman daftar produk punya tombol/link "Tambah Produk".
- [ ] Saya bisa navigasi: daftar -> form -> kembali ke daftar.
- [ ] Saya paham bahwa form **belum menyimpan** data sampai tahap berikutnya.

Jika semua sudah tercentang, tampilan form sudah siap.

---

## 12. Penutup

Selamat! Kamu sudah:

- Membuat halaman form tambah produk.
- Memahami elemen dasar form HTML di Laravel (`<form>`, `<input>`, `<textarea>`).
- Menambahkan token keamanan `@csrf`.
- Menghubungkan halaman daftar dengan halaman form lewat link.

Form kamu sudah terlihat bagus, **tapi belum berfungsi menyimpan**. Itu disengaja
agar kita paham satu hal dalam satu waktu.

Di **tahap berikutnya**, kita akan membuat **proses penyimpanan** - yaitu
menangkap data dari form, menyimpannya ke database via Model, lalu kembali
ke halaman daftar produk. Setelah itu, produk baru akan muncul di daftar produk.

### Jika sudah paham tahap ini, ketik:

```
lanjut
```

Jangan lanjut sebelum kamu benar-benar siap. Pelan-pelan, asal paham.
