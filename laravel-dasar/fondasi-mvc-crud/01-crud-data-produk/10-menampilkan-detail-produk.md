# Tahap 10 — Menampilkan Detail Produk

> **Tujuan tahap ini:** Menampilkan halaman detail untuk satu produk berdasarkan ID. **Belum membuat fitur edit, update, atau hapus.** Hanya fokus pada Read (detail satu produk).

---

## 1. Pengantar Sederhana

Pada tahap-tahap sebelumnya, kita sudah bisa:

- **Menampilkan daftar produk** (Tahap 7).
- **Membuka form tambah produk** (Tahap 8).
- **Menyimpan produk baru ke database** (Tahap 9).

Saat ini, halaman daftar produk menampilkan semua produk dalam bentuk tabel
dengan info singkat (ID, nama, deskripsi, harga, stok). Tapi user belum bisa
**mengklik satu produk** untuk melihat halaman khusus produk tersebut.

Di tahap ini, kita akan membuat **halaman detail produk** - yaitu halaman yang
menampilkan **informasi lengkap** untuk **satu produk** berdasarkan ID-nya.

### Analogi: Etalase vs Kartu Produk

| Hal                  | Analogi                                                  |
| -------------------- | -------------------------------------------------------- |
| Halaman daftar produk | Rak etalase berisi banyak produk (semua tampil sekaligus) |
| **Halaman detail produk** | **Kartu informasi lengkap** untuk satu produk yang dipilih |

### Alur penggunaan

1. User membuka halaman daftar produk (`/products`).
2. User melihat ada produk tertentu yang menarik.
3. User **mengklik nama produk** (atau tombol "Detail").
4. Browser pindah ke halaman detail produk tertentu, misal: `/products/1`.
5. User melihat informasi lengkap produk tersebut: nama, deskripsi panjang,
   harga, stok, kapan dibuat, kapan terakhir diubah.
6. User bisa klik "Kembali" untuk kembali ke daftar produk.

### Penting!

Pada tahap ini kita **hanya melihat detail produk**. Belum ada:

- Fitur edit produk.
- Fitur update produk.
- Fitur hapus produk.

Itu akan dibuat di tahap-tahap berikutnya.

---

## 2. Tujuan Tahap Ini

Ketika user membuka alamat seperti:

```text
http://127.0.0.1:8000/products/1
```

Maka browser akan menampilkan halaman detail untuk produk dengan ID = 1:

```text
+------------------------------------------+
|  Detail Produk: Kaos Hitam               |
|                                          |
|  ID:          1                          |
|  Nama:        Kaos Hitam                 |
|  Deskripsi:   Kaos bahan cotton combed   |
|  Harga:       Rp 100.000                 |
|  Stok:        20                         |
|  Dibuat:      2026-07-16 10:30:00        |
|  Diubah:      2026-07-16 10:30:00        |
|                                          |
|  [&larr; Kembali ke Daftar Produk]        |
+------------------------------------------+
```

Jika user buka `/products/2`, maka tampil detail produk dengan ID = 2.
Jika user buka `/products/3`, maka tampil detail produk dengan ID = 3.

Bagian angka di URL (`1`, `2`, `3`) disebut **parameter route** (`{id}`),
yang sudah kita buat di Tahap 5.

---

## 3. Langkah 1: Mengubah Method `show()` di Controller

Saat ini method `show($id)` masih mengembalikan teks polos. Kita ubah agar:

1. Mengambil satu produk dari database berdasarkan ID.
2. Mengirim data produk tersebut ke View.

### Bukka file `app/Http/Controllers/ProductController.php`

Ubah method `show()` menjadi:

```php
public function show($id)
{
    $product = Product::findOrFail($id);
    return view('products.show', compact('product'));
}
```

### Penjelasan tiap baris

#### `public function show($id)`

Parameter `$id` berisi nilai dari URL. Misal:

- URL `/products/5` -> `$id` berisi `'5'`.
- URL `/products/12` -> `$id` berisi `'12'`.

Ini sudah diatur oleh route `Route::get('/products/{id}', [ProductController::class, 'show'])`
yang kita buat di Tahap 5.

#### `Product::findOrFail($id)`

Ini adalah perintah Eloquent untuk **mengambil satu produk berdasarkan ID**.

- `find($id)` -> mencari produk dengan ID tersebut. Jika tidak ditemukan, mengembalikan `null`.
- `findOrFail($id)` -> sama, tapi jika tidak ditemukan, **otomatis melempar error 404** (halaman tidak ditemukan).

Memakai `findOrFail` lebih aman: jika user membuka `/products/9999` padahal
ID 9999 tidak ada, Laravel otomatis menampilkan halaman **404 Not Found**.
Kita tidak perlu menulis pengecekan manual.

Hasilnya disimpan di variabel `$product` (tunggal, karena ini hanya satu produk).

#### `return view('products.show', compact('product'));`

Artinya: "Tampilkan View `products.show` (file: `resources/views/products/show.blade.php`),
dan kirimkan variabel `$product` ke View."

### Kode controller lengkap (bagian method show)

```php
public function show($id)
{
    $product = Product::findOrFail($id);
    return view('products.show', compact('product'));
}
```

### Alternatif: Route Model Binding

Sebenarnya Laravel punya cara yang lebih elegan, yaitu **Route Model Binding**:

```php
public function show(Product $product)
{
    return view('products.show', compact('product'));
}
```

Di sini kita tidak menerima `$id`, tapi langsung menerima objek `Product`.
Laravel **otomatis** mencari produk berdasarkan ID dari URL dan menyuntikkan
objeknya ke method.

Tapi untuk belajar, kita tetap memakai `findOrFail($id)` dulu supaya
**alurnya jelas**: kita terima ID, kita cari produk, kita kirim ke View.
Setelah paham, kamu bisa pakai Route Model Binding di proyek sungguhan.

---

## 4. Langkah 2: Membuat View Detail Produk

Sekarang kita buat file View untuk halaman detail.

### Lokasi file

Buat file baru di:

```
resources/views/products/show.blade.php
```

Struktur folder menjadi:

```
resources/
  views/
    products/
      index.blade.php     <-- sudah ada (Tahap 7)
      create.blade.php    <-- sudah ada (Tahap 8)
      show.blade.php      <-- file baru yang dibuat sekarang
```

### Kode `show.blade.php`

Isi file `resources/views/products/show.blade.php` dengan kode berikut:

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Detail Produk: {{ $product->name }}</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        .detail-card {
            background-color: #f9f9f9;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 20px;
            max-width: 500px;
        }
        .detail-card h1 {
            margin-top: 0;
            color: #333;
        }
        .detail-row {
            display: flex;
            padding: 8px 0;
            border-bottom: 1px solid #eee;
        }
        .detail-row:last-child {
            border-bottom: none;
        }
        .detail-label {
            font-weight: bold;
            width: 120px;
            color: #555;
        }
        .detail-value {
            flex: 1;
        }
        a {
            text-decoration: none;
            color: #007bff;
            display: inline-block;
            margin-top: 15px;
        }
    </style>
</head>
<body>
    <div class="detail-card">
        <h1>{{ $product->name }}</h1>

        <div class="detail-row">
            <span class="detail-label">ID</span>
            <span class="detail-value">{{ $product->id }}</span>
        </div>

        <div class="detail-row">
            <span class="detail-label">Deskripsi</span>
            <span class="detail-value">
                @if ($product->description)
                    {{ $product->description }}
                @else
                    <em>Tidak ada deskripsi.</em>
                @endif
            </span>
        </div>

        <div class="detail-row">
            <span class="detail-label">Harga</span>
            <span class="detail-value">Rp {{ number_format($product->price, 0, ',', '.') }}</span>
        </div>

        <div class="detail-row">
            <span class="detail-label">Stok</span>
            <span class="detail-value">{{ $product->stock }}</span>
        </div>

        <div class="detail-row">
            <span class="detail-label">Dibuat pada</span>
            <span class="detail-value">{{ $product->created_at->format('d M Y, H:i') }}</span>
        </div>

        <div class="detail-row">
            <span class="detail-label">Diubah pada</span>
            <span class="detail-value">{{ $product->updated_at->format('d M Y, H:i') }}</span>
        </div>
    </div>

    <a href="/products">&larr; Kembali ke Daftar Produk</a>
</body>
</html>
```

### Penjelasan bagian penting

#### `{{ $product->name }}`

Karena controller mengirim variabel `$product` (satu objek produk), kita
mengakses atributnya dengan panah `->`:

- `$product->id`
- `$product->name`
- `$product->description`
- `$product->price`
- `$product->stock`
- `$product->created_at`
- `$product->updated_at`

#### `@if ($product->description) ... @else ... @endif`

Deskripsi produk **boleh kosong** (di migration, kolom `description` memakai
`->nullable()`). Jika kosong, kita tampilkan pesan "Tidak ada deskripsi."
agar lebih ramah pengguna.

#### `number_format($product->price, 0, ',', '.')`

Sama seperti di halaman daftar (Tahap 7), fungsi ini memformat angka harga:

- `100000` -> `100.000`

Agar lebih enak dibaca: "Rp 100.000".

#### `$product->created_at->format('d M Y, H:i')`

Kolom `created_at` dan `updated_at` adalah **timestamp** - objek tanggal yang
dikelola Laravel. Kita bisa memformatnya memakai method `format()`:

- `'d M Y, H:i'` artinya: tanggal singkat, bulan singkat, tahun 4 digit, jam:menit.
- Contoh hasil: `16 Jul 2026, 10:30`.

#### `<a href="/products">&larr; Kembali ke Daftar Produk</a>`

Link untuk kembali ke halaman daftar produk.

---

## 5. Langkah 3: Menambahkan Link "Detail" di Halaman Daftar

Agar user bisa **mengklik produk** dari daftar untuk melihat detailnya, kita
tambahkan kolom "Aksi" di tabel daftar produk, berisi link "Detail".

### Bukka file `resources/views/products/index.blade.php`

Kita perlu:

1. Menambahkan kolom **"Aksi"** di header tabel.
2. Menambahkan sel di setiap baris dengan link ke halaman detail.

#### Ubah bagian `<thead>`:

Tambahkan satu kolom baru di akhir:

```blade
<thead>
    <tr>
        <th>ID</th>
        <th>Nama</th>
        <th>Deskripsi</th>
        <th>Harga</th>
        <th>Stok</th>
        <th>Aksi</th>   <!-- kolom baru -->
    </tr>
</thead>
```

#### Ubah bagian `<tbody>`:

Di dalam `@foreach`, tambahkan sel dengan link detail:

```blade
<tbody>
    @foreach ($products as $product)
        <tr>
            <td>{{ $product->id }}</td>
            <td>{{ $product->name }}</td>
            <td>{{ $product->description }}</td>
            <td>Rp {{ number_format($product->price, 0, ',', '.') }}</td>
            <td>{{ $product->stock }}</td>
            <td>
                <a href="/products/{{ $product->id }}">Detail</a>
            </td>
        </tr>
    @endforeach
</tbody>
```

### Penjelasan link

```blade
<a href="/products/{{ $product->id }}">Detail</a>
```

- `href="/products/{{ $product->id }}"` menghasilkan URL seperti `/products/1`,
  `/products/2`, dst, tergantung ID produk di baris tersebut.
- Teks link: "Detail".

### Kode `index.blade.php` lengkap (dengan kolom Aksi)

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

    @if (session('success'))
        <div style="
            background-color: #d4edda;
            color: #155724;
            padding: 10px 15px;
            border: 1px solid #c3e6cb;
            border-radius: 4px;
            margin-bottom: 15px;
        ">
            {{ session('success') }}
        </div>
    @endif

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
                    <th>Aksi</th>
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
                        <td>
                            <a href="/products/{{ $product->id }}">Detail</a>
                        </td>
                    </tr>
                @endforeach
            </tbody>
        </table>
    @endif
</body>
</html>
```

---

## 6. Langkah 4: Menjalankan dan Menguji

### Jalankan server

```bash
php artisan serve
```

### Skenario uji 1: Klik dari daftar ke detail

1. Buka `http://127.0.0.1:8000/products` (halaman daftar).
2. Di setiap baris produk, sekarang ada kolom **Aksi** dengan link **Detail**.
3. Klik "Detail" pada salah satu produk.
4. Browser pindah ke `/products/1` (atau ID yang sesuai).
5. Halaman detail produk ditampilkan dengan semua informasi lengkap.

### Skenario uji 2: Akses langsung via URL

1. Buka browser langsung ke `http://127.0.0.1:8000/products/1`.
2. Halaman detail produk ID 1 langsung tampil.

### Skenario uji 3: ID tidak ada (404)

1. Buka `http://127.0.0.1:8000/products/9999` (asumsi ID 9999 tidak ada).
2. Laravel menampilkan halaman **404 Not Found** (karena `findOrFail` melempar
   error saat produk tidak ditemukan).

Ini adalah perilaku yang benar - kita tidak ingin menampilkan halaman detail
kosong untuk produk yang tidak ada.

### Skenario uji 4: Kembali ke daftar

1. Di halaman detail, klik **"Kembali ke Daftar Produk"**.
2. Browser kembali ke `/products`.

---

## 7. Apa yang Terjadi di Balik Layar?

Mari kita telusuri alurnya secara lengkap:

```
1. User klik "Detail" produk dengan ID 1
   atau buka langsung http://127.0.0.1:8000/products/1
        |
        v
2. Route::get('/products/{id}', [ProductController::class, 'show'])
   Route menangkap id = 1 dari URL
        |
        v
3. ProductController::show($id)
   Method show dipanggil dengan $id = 1
        |
        v
4. Product::findOrFail(1)
   Model Product mencari produk dengan ID = 1 di database.
   Jika tidak ada -> 404. Jika ada -> objek Product.
        |
        v
5. return view('products.show', compact('product'))
   Controller mengirim objek product ke View.
        |
        v
6. resources/views/products/show.blade.php
   View menampilkan detail produk dengan atribut-atributnya.
        |
        v
7. Browser menerima HTML, tampilkan halaman detail produk.
```

---

## 8. Hal-hal yang Perlu Diperhatikan

### 1. Selalu pakai `findOrFail` untuk ID tunggal

Jika kamu memakai `Product::find($id)` tanpa `orFail`, dan ID tidak ditemukan,
variabel `$product` akan berisi `null`. View akan error saat mencoba
mengakses `$product->name` (karena `null` tidak punya atribut).

Dengan `findOrFail`, Laravel otomatis menampilkan halaman 404 yang bersih.

### 2. Parameter route `{id}` harus cocok dengan parameter method

Route:

```php
Route::get('/products/{id}', [ProductController::class, 'show']);
```

Method controller:

```php
public function show($id) { ... }
```

Nama variabel `id` di route **tidak harus sama** dengan nama parameter di
method (`$id`). Yang penting **urutan**. Tapi untuk konsistensi, sebaiknya
pakai nama yang sama: `{id}` -> `$id`.

### 3. Variabel yang dikirim ke View

Perhatikan bedanya:

| Method         | Variabel yang dikirim       | Jumlah data     |
| -------------- | --------------------------- | --------------- |
| `index()`      | `$products` (plural)         | Banyak produk   |
| `show($id)`    | `$product` (singular)        | Satu produk     |

Di `index`, kita kirim koleksi (banyak produk) untuk ditampilkan dalam tabel.
Di `show`, kita kirim satu objek produk untuk ditampilkan sebagai kartu detail.

### 4. Timestamp di Laravel

`created_at` dan `updated_at` otomatis diisi oleh Laravel:

- `created_at`: diisi sekali saat produk pertama kali dibuat.
- `updated_at`: diisi ulang setiap kali produk diubah (di tahap Update nanti).

Keduanya adalah objek **Carbon** (library tanggal Laravel), sehingga kita bisa
memakai method `format()`: `$product->created_at->format('d M Y, H:i')`.

---

## 9. Ringkasan Alur Tahap Ini

```
1. Ubah method show() di controller:
   - Product::findOrFail($id)
   - return view('products.show', compact('product'))
        |
        v
2. Buat file resources/views/products/show.blade.php
   - Tampilkan semua atribut produk dalam kartu detail
   - Format harga dan tanggal
        |
        v
3. Tambah kolom "Aksi" + link "Detail" di index.blade.php
        |
        v
4. php artisan serve, tes klik Detail dari daftar
        |
        v
5. Halaman detail produk tampil
```

---

## 10. Checklist Keberhasilan Tahap Ini

Centang (`x`) jika sudah selesai:

- [ ] Method `show($id)` di controller memakai `Product::findOrFail($id)`.
- [ ] Controller mengirim `$product` ke `view('products.show')`.
- [ ] File `resources/views/products/show.blade.php` sudah dibuat.
- [ ] Halaman detail menampilkan: ID, nama, deskripsi, harga, stok, tanggal dibuat, tanggal diubah.
- [ ] Halaman daftar produk punya kolom "Aksi" dengan link "Detail".
- [ ] Klik "Detail" pada produk -> pindah ke halaman detail produk tersebut.
- [ ] Akses URL `/products/9999` (ID tidak ada) -> tampil halaman 404.
- [ ] Halaman detail punya link "Kembali ke Daftar Produk".

Jika semua sudah tercentang, fitur **Read (detail produk)** sudah berfungsi.

---

## 11. Penutup

Selamat! Kamu sudah menyelesaikan fitur **Read (detail satu produk)**.
Sekarang user bisa:

- Melihat daftar semua produk.
- Mengklik salah satu produk.
- Melihat detail lengkap produk tersebut di halaman khusus.

Kombinasi yang sudah kita bangun sejauh ini:

- **Read** - Lihat daftar produk (Tahap 7).
- **Create** - Tambah produk baru (Tahap 8 + 9).
- **Read** - Lihat detail satu produk (**Tahap 10** ini).

Di **tahap berikutnya**, kita akan belajar cara **mengedit produk** - yaitu
membuka form edit yang sudah terisi data produk lama, mengubah isinya, lalu
menyimpan perubahan tersebut kembali ke database (Update).

### Jika sudah paham tahap ini, ketik:

```
lanjut
```

Jangan lanjut sebelum kamu benar-benar siap. Pelan-pelan, asal paham.
