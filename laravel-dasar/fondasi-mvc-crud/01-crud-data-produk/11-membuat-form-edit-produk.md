# Tahap 11 — Membuat Form Edit Produk

> **Tujuan tahap ini:** Membuat halaman form edit produk yang sudah terisi data lama. **Belum melakukan proses update ke database.** Form hanya menampilkan data lama yang siap diubah. Update akan dibuat di tahap berikutnya.

---

## 1. Pengantar Sederhana

Pada tahap-tahap sebelumnya, kita sudah bisa:

- **Menampilkan daftar produk** (Tahap 7).
- **Menambahkan produk baru** (Tahap 8 + 9).
- **Melihat detail satu produk** (Tahap 10).

Sekarang kita akan membuat **halaman form edit produk** - yaitu form yang
**sudah terisi data lama** produk, sehingga admin bisa memperbaiki isi
nama, deskripsi, harga, atau stok.

### Analogi: Kartu Produk vs Formulir Edit

| Hal                    | Analogi                                                |
| ---------------------- | ------------------------------------------------------ |
| Halaman detail produk  | Kartu informasi produk (hanya bisa dibaca, tidak bisa diubah) |
| **Form edit produk**   | **Formulir yang sudah terisi data lama**, siap diperbaiki |

### Alur penggunaan

1. Admin melihat ada produk yang perlu diperbaiki (misal: salah ketik harga).
2. Admin klik tombol **Edit** pada produk tersebut.
3. Browser pindah ke halaman form edit yang **sudah terisi data lama**.
4. Admin memperbaiki field yang salah.
5. Admin klik tombol **Update** (tapi pada tahap ini belum benar-benar update).

### Penting!

Pada tahap ini, kita **hanya membuat tampilan form edit** dan **menampilkan data lama**.

- Form akan terisi otomatis dari database.
- Tombol "Update" **belum benar-benar menyimpan perubahan**.
- Proses update ke database akan dibuat di **tahap berikutnya**.

Tujuannya: fokus pada satu hal - cara membuat form edit yang menampilkan data lama.

---

## 2. Tujuan Tahap Ini

Ketika user membuka alamat seperti:

```text
http://127.0.0.1:8000/products/1/edit
```

Maka browser akan menampilkan halaman form edit yang **sudah terisi data lama**
produk dengan ID = 1:

```text
+------------------------------------------+
|  Edit Produk: Kaos Hitam                 |
|                                          |
|  Nama:        [Kaos Hitam_____________]  |
|  Deskripsi:   [Kaos bahan cotton combed] |
|               [________________________] |
|  Harga:       [100000_________________]  |
|  Stok:        [20_____________________]  |
|                                          |
|  [ Update ]                              |
+------------------------------------------+
```

Perhatikan bedanya dengan form tambah produk (Tahap 8):

| Form Tambah Produk                | Form Edit Produk                          |
| --------------------------------- | ----------------------------------------- |
| Semua field **kosong**             | Semua field **terisi data lama**           |
| Tujuan: memasukkan produk baru     | Tujuan: memperbaiki produk yang sudah ada  |
| Tombol: **Simpan**                 | Tombol: **Update**                         |
| URL: `/products/create`            | URL: `/products/{id}/edit`                 |

---

## 3. Langkah 1: Mengubah Method `edit()` di Controller

Saat ini method `edit($id)` masih mengembalikan teks polos. Kita ubah agar:

1. Mengambil satu produk dari database berdasarkan ID.
2. Mengirim data produk tersebut ke View.

### Bukka file `app/Http/Controllers/ProductController.php`

Ubah method `edit()` menjadi:

```php
public function edit($id)
{
    $product = Product::findOrFail($id);
    return view('products.edit', compact('product'));
}
```

### Penjelasan tiap baris

#### `public function edit($id)`

Sama seperti `show($id)`, parameter `$id` berisi nilai dari URL.

Misal URL `/products/1/edit` -> `$id` berisi `'1'`.

#### `$product = Product::findOrFail($id);`

Mengambil produk berdasarkan ID. Jika tidak ditemukan, Laravel otomatis
menampilkan halaman **404 Not Found** (sama seperti di method `show`).

#### `return view('products.edit', compact('product'));`

Artinya: "Tampilkan View `products.edit` (file: `resources/views/products/edit.blade.php`),
dan kirimkan variabel `$product` ke View."

Variabel `$product` ini berisi objek produk lama yang akan kita tampilkan di form.

### Kode controller lengkap (bagian method edit)

```php
public function edit($id)
{
    $product = Product::findOrFail($id);
    return view('products.edit', compact('product'));
}
```

---

## 4. Langkah 2: Pastikan Route `/products/{id}/edit` Sudah Ada

Pada **Tahap 6**, kita sudah membuat route ini. Pastikan baris berikut
**masih ada** di `routes/web.php`:

```php
Route::get('/products/{id}/edit', [ProductController::class, 'edit']);
```

Artinya: "Jika user membuka `/products/1/edit` dengan GET, panggil method `edit`
di `ProductController`."

Perhatikan urutan route berikut di `routes/web.php`:

```php
Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/create', [ProductController::class, 'create']);
Route::post('/products', [ProductController::class, 'store']);
Route::get('/products/{id}', [ProductController::class, 'show']);
Route::get('/products/{id}/edit', [ProductController::class, 'edit']);  // <-- ini
Route::put('/products/{id}', [ProductController::class, 'update']);
Route::delete('/products/{id}', [ProductController::class, 'destroy']);
```

Route `/products/{id}/edit` ini **spesifik** (ada `/edit` di akhir), jadi tidak
bentrok dengan `/products/{id}` yang lebih umum.

---

## 5. Langkah 3: Membuat View Form Edit

Sekarang kita buat file View untuk form edit.

### Lokasi file

Buat file baru di:

```
resources/views/products/edit.blade.php
```

Struktur folder menjadi:

```
resources/
  views/
    products/
      index.blade.php     <-- sudah ada (Tahap 7)
      create.blade.php    <-- sudah ada (Tahap 8)
      show.blade.php      <-- sudah ada (Tahap 10)
      edit.blade.php      <-- file baru yang dibuat sekarang
```

### Kode `edit.blade.php`

Isi file `resources/views/products/edit.blade.php` dengan kode berikut:

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Edit Produk: {{ $product->name }}</title>
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
            background-color: #ffc107;
            color: #333;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover {
            background-color: #e0a800;
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
    <h1>Edit Produk: {{ $product->name }}</h1>

    <!--
        Catatan: pada tahap ini form belum benar-benar menyimpan perubahan.
        Atribut action dan method akan diatur di tahap berikutnya
        saat kita membuat fitur update produk (PUT /products/{id}).
    -->
    <form action="#" method="POST">
        @csrf
        @method('PUT')

        <div class="form-group">
            <label for="name">Nama Produk</label>
            <input type="text" name="name" id="name" value="{{ $product->name }}">
        </div>

        <div class="form-group">
            <label for="description">Deskripsi</label>
            <textarea name="description" id="description">{{ $product->description }}</textarea>
        </div>

        <div class="form-group">
            <label for="price">Harga (Rp)</label>
            <input type="number" name="price" id="price" value="{{ $product->price }}" min="0">
        </div>

        <div class="form-group">
            <label for="stock">Stok</label>
            <input type="number" name="stock" id="stock" value="{{ $product->stock }}" min="0">
        </div>

        <button type="submit">Update</button>
    </form>

    <p style="margin-top: 20px;">
        <a href="/products">&larr; Kembali ke Daftar Produk</a>
    </p>
</body>
</html>
```

### Perbedaan utama dengan form tambah produk (`create.blade.php`)

Ada beberapa perbedaan penting:

#### 1. Atribut `value` di setiap input

Form edit **menampilkan data lama**. Caranya: setiap input punya atribut `value`
yang berisi nilai produk dari database.

```blade
<input type="text" name="name" id="name" value="{{ $product->name }}">
```

Saat halaman dibuka, input "Nama Produk" akan **sudah terisi** dengan nama produk lama.

#### 2. `<textarea>` berbeda dengan `<input>`

Untuk `<textarea>`, data lama **tidak diletakkan di atribut `value`**, melainkan
**di antara tag pembuka dan penutup**:

```blade
<textarea name="description" id="description">{{ $product->description }}</textarea>
```

Ini adalah aturan HTML, bukan Laravel.

#### 3. `@method('PUT')`

HTML hanya mengenal dua method form: **GET** dan **POST**. Tapi untuk update
produk di Laravel, kita pakai method **PUT** (konvensi REST).

Blade menyediakan direktif `@method('PUT')` untuk "mengakali" batasan ini:
form tetap dikirim sebagai POST di HTML, tapi Laravel akan membaca method-nya
sebagai PUT berkat hidden field yang dihasilkan `@method('PUT')`.

> Catatan: Di tahap berikutnya (Update), kita akan mengubah `action="#"` menjadi
> `action="/products/{{ $product->id }}"`. Untuk sekarang `action="#"` dulu.

#### 4. Tombol berbeda

Form tambah memakai tombol **Simpan**, form edit memakai tombol **Update**.

### Penjelasan `@csrf`

Sama seperti form tambah, form edit juga **wajib** memakai `@csrf` untuk
keamanan CSRF. Jangan lupakan.

### Perbandingan kode: Form Tambah vs Form Edit

| Bagian                 | Form Tambah (`create.blade.php`)           | Form Edit (`edit.blade.php`)                 |
| ---------------------- | ------------------------------------------ | -------------------------------------------- |
| `value` di input       | Tidak ada (form kosong)                    | Ada: `value="{{ $product->name }}"` dst.    |
| `<textarea>`           | Kosong                                     | Terisi: `{{ $product->description }}`       |
| `@method(...)`         | Tidak ada (default POST)                   | `@method('PUT')`                             |
| `action`               | `action="/products"`                       | `action="#"` (akan diubah di tahap update)   |
| Tombol                 | Simpan                                     | Update                                        |
| Judul                  | Tambah Produk                              | Edit Produk: [Nama Produk]                    |

---

## 6. Langkah 4: Menambahkan Link "Edit" di Halaman Daftar

Agar user bisa membuka form edit dari halaman daftar, kita tambahkan link
**Edit** di kolom Aksi (yang sudah dibuat di Tahap 10).

### Bukka file `resources/views/products/index.blade.php`

Cari bagian kolom Aksi di `<tbody>`:

```blade
<td>
    <a href="/products/{{ $product->id }}">Detail</a>
</td>
```

Ubah menjadi:

```blade
<td>
    <a href="/products/{{ $product->id }}">Detail</a>
    |
    <a href="/products/{{ $product->id }}/edit">Edit</a>
</td>
```

Sekarang setiap baris produk punya dua link: **Detail** dan **Edit**.

---

## 7. Langkah 5: Menambahkan Link "Edit" di Halaman Detail

Selain dari halaman daftar, user juga bisa membuka form edit dari halaman detail
produk. Mari kita tambahkan tombol Edit di halaman `show.blade.php`.

### Bukka file `resources/views/products/show.blade.php`

Di bagian bawah kartu detail (sebelum link "Kembali"), tambahkan:

```blade
<a href="/products/{{ $product->id }}/edit"
   style="
        display: inline-block;
        background-color: #ffc107;
        color: #333;
        padding: 8px 16px;
        text-decoration: none;
        border-radius: 4px;
        margin-top: 15px;
   ">
    Edit Produk
</a>
```

Sekarang halaman detail produk punya tombol kuning **Edit Produk** yang
jika diklik akan membuka form edit untuk produk tersebut.

---

## 8. Langkah 6: Menjalankan dan Menguji

### Jalankan server

```bash
php artisan serve
```

### Skenario uji 1: Buka form edit dari daftar

1. Buka `http://127.0.0.1:8000/products`.
2. Pada salah satu produk, klik **Edit**.
3. Browser pindah ke `/products/1/edit` (atau ID yang sesuai).
4. Form edit tampil dengan **field sudah terisi data lama**:
   - Nama sudah terisi
   - Deskripsi sudah terisi
   - Harga sudah terisi
   - Stok sudah terisi

### Skenario uji 2: Buka form edit dari halaman detail

1. Buka `http://127.0.0.1:8000/products`.
2. Klik **Detail** pada produk tertentu.
3. Di halaman detail, klik **Edit Produk**.
4. Browser pindah ke halaman form edit produk tersebut.

### Skenario uji 3: Akses langsung via URL

1. Buka langsung `http://127.0.0.1:8000/products/1/edit`.
2. Form edit langsung tampil dengan data produk ID 1.

### Skenario uji 4: ID tidak ada (404)

1. Buka `http://127.0.0.1:8000/products/9999/edit` (asumsi ID 9999 tidak ada).
2. Laravel menampilkan halaman **404 Not Found** (karena `findOrFail`).

### Skenario uji 5: Edit field (belum tersimpan)

1. Buka form edit produk.
2. Ubah nama produk menjadi "Kaos Hitam Edit".
3. Klik **Update**.
4. Tidak terjadi apa-apa (atau error 404 karena `action="#"` dan route PUT belum dibuat).
5. Itu wajar - kita belum membuat proses update.

Di **tahap berikutnya**, tombol Update akan benar-benar menyimpan perubahan.

---

## 9. Apa yang Terjadi di Balik Layar?

Mari kita telusuri alurnya:

```
1. User klik "Edit" produk dengan ID 1
   atau buka http://127.0.0.1:8000/products/1/edit (GET)
        |
        v
2. Route::get('/products/{id}/edit', [ProductController::class, 'edit'])
   Route menangkap id = 1 dari URL
        |
        v
3. ProductController::edit($id)
   Method edit dipanggil dengan $id = 1
        |
        v
4. Product::findOrFail(1)
   Model Product mencari produk dengan ID = 1.
   Jika tidak ada -> 404. Jika ada -> objek Product.
        |
        v
5. return view('products.edit', compact('product'))
   Controller mengirim objek product ke View.
        |
        v
6. resources/views/products/edit.blade.php
   View menampilkan form dengan field yang sudah terisi data lama
   (value="{{ $product->name }}" dst.)
        |
        v
7. Browser menerima HTML, tampilkan form edit dengan data lama.
```

---

## 10. Hal-hal yang Perlu Diperhatikan

### 1. Selalu pakai `findOrFail` untuk ID tunggal

Sama seperti method `show()`, jika ID tidak ditemukan, `findOrFail` akan
menampilkan halaman 404 yang bersih. Hindari `find()` saja (yang mengembalikan
`null` dan menyebabkan error di View).

### 2. Cara menampilkan data lama di form

| Tipe Input      | Cara Menampilkan Data Lama                                  |
| --------------- | ----------------------------------------------------------- |
| `<input>`       | Pakai atribut `value="{{ $product->nama_kolom }}"`           |
| `<textarea>`    | Letakkan data di antara tag: `>{{ $product->nama_kolom }}<`  |
| `<select>`      | Pakai kondisional (akan dibahas nanti)                       |

### 3. `@method('PUT')` untuk update

HTML hanya mendukung GET dan POST di form. Untuk method lain (PUT, PATCH, DELETE),
kita pakai direktif Blade `@method('NAMA_METHOD')`:

- Update: `@method('PUT')` atau `@method('PATCH')`
- Delete: `@method('DELETE')`

Ini konvensi REST yang akan kita pakai di tahap update dan delete.

### 4. Perbedaan method HTTP untuk CRUD

| Aksi   | Method HTTP | Tujuan                |
| ------ | ----------- | --------------------- |
| Create | POST        | Buat data baru         |
| Read   | GET         | Ambil/lihat data       |
| Update | PUT/PATCH   | Ubah data yang ada     |
| Delete | DELETE      | Hapus data             |

Karena form edit bertugas **mengubah** data, method-nya adalah **PUT**.

### 5. Atribut `name` harus tetap konsisten

Sama seperti form tambah, atribut `name` di input harus cocok dengan nama kolom:

- `name="name"`
- `name="description"`
- `name="price"`
- `name="stock"`

---

## 11. Ringkasan Alur Tahap Ini

```
1. Ubah method edit() di controller:
   - Product::findOrFail($id)
   - return view('products.edit', compact('product'))
        |
        v
2. Buat file resources/views/products/edit.blade.php
   - Form dengan field terisi data lama (value="{{ $product->... }}")
   - @method('PUT')
   - @csrf
   - action="#" dulu (belum menyimpan)
        |
        v
3. Tambah link "Edit" di halaman daftar (index.blade.php)
        |
        v
4. Tambah tombol "Edit Produk" di halaman detail (show.blade.php)
        |
        v
5. php artisan serve, tes buka /products/{id}/edit
        |
        v
6. Form edit tampil dengan data lama, tapi belum menyimpan perubahan
```

---

## 12. Checklist Keberhasilan Tahap Ini

Centang (`x`) jika sudah selesai:

- [ ] Method `edit($id)` di controller memakai `Product::findOrFail($id)`.
- [ ] Controller mengirim `$product` ke `view('products.edit')`.
- [ ] File `resources/views/products/edit.blade.php` sudah dibuat.
- [ ] Form edit menampilkan data lama di setiap field (input value, textarea content).
- [ ] Form memakai `@method('PUT')` dan `@csrf`.
- [ ] Halaman daftar produk punya link **Edit** di kolom Aksi.
- [ ] Halaman detail produk punya tombol **Edit Produk**.
- [ ] Form bisa dibuka dari daftar dan dari halaman detail.
- [ ] URL `/products/9999/edit` (ID tidak ada) tampil 404.
- [ ] Saya paham bahwa form **belum menyimpan** perubahan sampai tahap berikutnya.

Jika semua sudah tercentang, tampilan form edit sudah siap.

---

## 13. Penutup

Selamat! Kamu sudah:

- Membuat halaman form edit produk.
- Menampilkan **data lama** produk di dalam form.
- Memahami perbedaan antara form tambah dan form edit.
- Mengenal direktif `@method('PUT')` untuk update.
- Menambahkan link Edit di halaman daftar dan halaman detail.

Form edit sudah terlihat benar, **tapi belum menyimpan perubahan**. Itu disengaja
agar kita paham satu hal dalam satu waktu.

Di **tahap berikutnya**, kita akan membuat **proses update** - yaitu menangkap
data dari form edit, menyimpan perubahan ke database, lalu kembali ke halaman
detail atau daftar produk. Setelah itu, produk akan benar-benar berubah.

### Jika sudah paham tahap ini, ketik:

```
lanjut
```

Jangan lanjut sebelum kamu benar-benar siap. Pelan-pelan, asal paham.
