# Tahap 9 — Menyimpan Produk ke Database

> **Tujuan tahap ini:** Memproses data dari form tambah produk dan menyimpannya ke database. **Belum membuat fitur detail, edit, update, atau hapus.** Hanya fokus pada proses simpan (Create).

---

## 1. Pengantar Sederhana

Pada **Tahap 8**, kita sudah membuat **form tambah produk**. Form tersebut
sudah punya input:

- Nama produk
- Deskripsi produk
- Harga produk
- Stok produk

Tapi form tersebut **belum bisa menyimpan data**. Saat tombol "Simpan" diklik,
tidak terjadi apa-apa karena `action="#"` dan belum ada route POST yang menangkap data.

Di tahap ini, kita akan memperbaiki itu: membuat **proses penyimpanan** agar
data dari form benar-benar masuk ke tabel `products`.

### Analogi: Formulir dan Petugas Arsip

| Hal                  | Analogi                                         |
| -------------------- | ----------------------------------------------- |
| Form tambah produk   | Formulir kosong yang diisi admin                |
| Admin mengisi form   | Menulis di formulir                              |
| Tombol **Simpan**    | Menyerahkan formulir ke petugas                 |
| **Controller** (`store`) | **Petugas** yang menerima formulir          |
| **Model** `Product`  | Petugas arsip yang menyimpan data ke lemari      |
| **Database**         | Lemari arsip tempat data produk disimpan         |

### Alur lengkap

1. Admin buka form (`/products/create`).
2. Admin mengisi nama, deskripsi, harga, stok.
3. Admin klik **Simpan** -> form dikirim (POST) ke `/products`.
4. Route menugaskan `ProductController@store`.
5. Controller mengambil data dari form.
6. Controller meminta Model menyimpan data ke database.
7. Controller mengarahkan (redirect) admin kembali ke daftar produk.
8. Produk baru muncul di daftar.

---

## 2. Tujuan Tahap Ini

Ketika user membuka:

```text
http://127.0.0.1:8000/products/create
```

Lalu mengisi form:

| Field        | Isi                       |
| ------------ | ------------------------- |
| Nama         | Topi Hitam                |
| Deskripsi    | Topi棒球 cap bahan katun   |
| Harga        | 75000                     |
| Stok         | 30                        |

Lalu klik **Simpan**, maka:

1. Data **tersimpan ke tabel `products`** di database.
2. Browser **kembali ke halaman daftar produk** (`/products`).
3. **Pesan sukses** muncul: "Produk berhasil ditambahkan."
4. **Topi Hitam** muncul sebagai baris baru di tabel daftar produk.

---

## 3. Langkah 1: Mengubah `action` di Form

Pada Tahap 8, form memakai `action="#"`. Sekarang kita arahkan ke URL yang benar.

### Bukka file `resources/views/products/create.blade.php`

Cari baris ini:

```blade
<form action="#" method="POST">
```

Ubah menjadi:

```blade
<form action="/products" method="POST">
    @csrf
    ...
</form>
```

### Penjelasan

- `action="/products"` artinya: saat form disubmit, kirim data ke URL `/products`.
- `method="POST"` artinya: kirim data dengan metode POST (bukan GET).
- POST dipakai untuk **mengirim data baru** ke server.

> Ingat: POST dipakai untuk **membuat/menyimpan** data baru.
> GET dipakai untuk **melihat/membaca** halaman.

---

## 4. Langkah 2: Pastikan Route POST `/products` Sudah Ada

Pada Tahap 6, kita sudah membuat route POST ini. Pastikan baris berikut
**masih ada** di `routes/web.php`:

```php
Route::post('/products', [ProductController::class, 'store']);
```

Artinya: "Jika ada permintaan POST ke `/products`, panggil method `store`
di `ProductController`."

Perhatikan bahwa ada **dua route** untuk URL `/products`, tapi dengan method berbeda:

| Method | URL         | Method Controller | Tujuan                 |
| ------ | ----------- | ----------------- | ---------------------- |
| GET    | `/products` | `index`           | Melihat daftar produk  |
| POST   | `/products` | `store`           | Menyimpan produk baru  |

URL-nya sama, tapi karena method-nya beda, Laravel tahu keduanya adalah route berbeda.

---

## 5. Langkah 3: Mengisi Method `store()` di Controller

Ini adalah inti dari tahap ini. Method `store()` bertugas:

1. Mengambil data yang dikirim dari form.
2. Memvalidasi data dasar (opsional, tapi disarankan).
3. Menyimpan data ke database via Model.
4. Mengarahkan (redirect) ke halaman daftar produk dengan pesan sukses.

### Bukka file `app/Http/Controllers/ProductController.php`

Ganti method `store()` menjadi:

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'name'        => 'required|string|max:255',
        'description' => 'nullable|string',
        'price'       => 'required|integer|min:0',
        'stock'       => 'required|integer|min:0',
    ]);

    Product::create($validated);

    return redirect('/products')->with('success', 'Produk berhasil ditambahkan.');
}
```

### Penjelasan tiap baris

#### `public function store(Request $request)`

Parameter `Request $request` adalah cara Laravel memberikan akses ke **semua
data yang dikirim dari form**. Bayangkan `$request` sebagai **amplop berisi formulir
yang sudah diisi admin**. Controller bisa membuka amplop ini dan mengambil isinya.

#### `$request->validate([...])`

Fungsi ini **memeriksa** data yang dikirim user. Jika ada yang tidak valid,
Laravel otomatis:

- Mengembalikan user ke halaman form.
- Menampilkan pesan error di setiap field yang bermasalah.
- Data tidak jadi disimpan.

Aturan validasi yang dipakai:

| Field         | Aturan                              | Arti                                     |
| ------------- | ----------------------------------- | ---------------------------------------- |
| `name`        | `required \| string \| max:255`     | Wajib diisi, teks, maksimal 255 karakter |
| `description` | `nullable \| string`                | Boleh kosong, harus teks                  |
| `price`       | `required \| integer \| min:0`      | Wajib, harus angka bulat, minimal 0      |
| `stock`       | `required \| integer \| min:0`      | Wajib, harus angka bulat, minimal 0      |

Hasil validasi yang lolos disimpan di `$validated`. Hanya data yang sudah
divalidasi yang dipakai untuk menyimpan - ini lebih aman.

#### `Product::create($validated)`

Ini adalah perintah Eloquent untuk **menyimpan produk baru** ke database.

`Product::create([...])` menerima array berisi data produk, lalu:

1. Membuat objek `Product` baru.
2. Mengisi atribut dari array (nama, deskripsi, harga, stok).
3. Menyimpan ke tabel `products` di database.
4. Otomatis mengisi `id`, `created_at`, dan `updated_at`.

> Catatan: `Product::create()` hanya bisa mengisi kolom yang ada di
> `$fillable` di Model `Product`. Inilah mengapa di Tahap 4 kita sudah
> mendaftarkan `name`, `description`, `price`, `stock` di `$fillable`.

#### `return redirect('/products')->with('success', '...')`

- `redirect('/products')` artinya: arahkan browser kembali ke `/products`.
- `->with('success', '...')` artinya: kirim pesan sukses yang bisa ditampilkan
  di halaman tujuan (selama satu request berikutnya).

Pesan ini disimpan sementara di **session** Laravel dan otomatis hilang setelah
ditampilkan sekali.

---

## 6. Langkah 4: Menampilkan Pesan Sukses di Halaman Daftar

Controller sudah mengirim pesan sukses, tapi View belum menampilkannya.
Kita perlu menambahkan kode di `index.blade.php`.

### Bukka file `resources/views/products/index.blade.php`

Tambahkan blok berikut **setelah `<h1>Daftar Produk</h1>`** dan sebelum tombol tambah:

```blade
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
```

### Penjelasan

- `session('success')` mengambil pesan sukses yang dikirim controller lewat `->with('success', ...)`.
- Jika ada pesan, tampilkan di kotak hijau.
- Jika tidak ada pesan, blok ini tidak muncul.

### Kode `index.blade.php` lengkap (dengan pesan sukses)

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

## 7. Langkah 5: Menampilkan Pesan Error Validasi di Form

Jika user mengisi form dengan salah (misal: nama kosong, harga negatif),
Laravel otomatis kembali ke form. Tapi kita perlu menampilkan **pesan error**
agar user tahu apa yang salah.

### Bukka file `resources/views/products/create.blade.php`

Tambahkan blok error di bagian atas form, setelah `<h1>Tambah Produk</h1>`:

```blade
@if ($errors->any())
    <div style="
        background-color: #f8d7da;
        color: #721c24;
        padding: 10px 15px;
        border: 1px solid #f5c6cb;
        border-radius: 4px;
        margin-bottom: 15px;
    ">
        <strong>Terjadi kesalahan:</strong>
        <ul style="margin: 5px 0 0 20px;">
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

### Penjelasan

- `$errors->any()` mengecek apakah ada error validasi.
- `$errors->all()` mengembalikan array berisi semua pesan error.
- Kita looping dengan `@foreach` untuk menampilkan setiap error sebagai item `<li>`.

Laravel otomatis menyediakan variabel `$errors` di setiap halaman yang memiliki
form. Variabel ini selalu ada, jadi tidak perlu dikirim manual dari controller.

---

## 8. Langkah 6: Menjalankan dan Menguji

### Jalankan server

```bash
php artisan serve
```

### Skenario uji 1: Input valid

1. Buka `http://127.0.0.1:8000/products/create`.
2. Isi form:
   - Nama: `Topi Hitam`
   - Deskripsi: `Topi baseball cap bahan katun`
   - Harga: `75000`
   - Stok: `30`
3. Klik **Simpan**.
4. Browser **kembali ke `/products`**.
5. Pesan sukses muncul: "Produk berhasil ditambahkan."
6. **Topi Hitam** muncul sebagai baris baru di tabel.

### Skenario uji 2: Input invalid

1. Buka `http://127.0.0.1:8000/products/create`.
2. Isi form dengan sengaja salah:
   - Nama: (kosong)
   - Harga: `-5000`
   - Stok: (kosong)
3. Klik **Simpan**.
4. Laravel **menolak** dan kembali ke form.
5. Pesan error muncul:
   - "The name field is required."
   - "The price field must be at least 0."
   - "The stock field is required."
6. Data tidak tersimpan ke database.

### Skenario uji 3: Cek database

Buka phpMyAdmin atau jalankan `php artisan tinker`, lalu `Product::all();`.
Pastikan hanya produk yang valid yang tersimpan di database.

---

## 9. Apa yang Terjadi di Balik Layar?

Mari kita telusuri alurnya secara lengkap:

```
1. User buka http://127.0.0.1:8000/products/create (GET)
        |
        v
2. User isi form, klik Simpan
   Form dikirim (POST) ke /products
        |
        v
3. Route::post('/products', [ProductController::class, 'store'])
   Route menugaskan method store
        |
        v
4. ProductController::store(Request $request)
   Method store dijalankan
        |
        v
5. $request->validate([...])
   Cek data form. Jika invalid -> kembali ke form dengan error.
        |
        v
6. Product::create($validated)
   Model Product menyimpan data ke tabel products di database.
        |
        v
7. redirect('/products')->with('success', '...')
   Browser diarahkan ke halaman daftar produk + pesan sukses.
        |
        v
8. Halaman /products dimuat ulang, pesan sukses tampil,
   produk baru muncul di tabel.
```

---

## 10. Pola PRG: Post - Redirect - Get

Perhatikan bahwa setelah form POST disubmit, kita tidak langsung menampilkan
View di method `store()`. Sebagai gantinya, kita **redirect** ke `/products`.

Pola ini disebut **PRG (Post - Redirect - Get)** dan adalah praktik terbaik di
pengembangan web:

1. **P**ost: user mengirim form (POST).
2. **R**edirect: server mengarahkan browser ke URL baru.
3. **G**et: browser mengambil halaman baru (GET).

### Mengapa penting?

Jika user mengakses URL hasil POST (misal: refresh halaman setelah submit),
browser akan **mengirim ulang form**. Ini bisa menyebabkan **data tersimpan dua kali**.

Dengan redirect, URL di address bar berubah menjadi GET (`/products`), sehingga
refresh tidak akan mengirim ulang data.

> Catatan: Ini adalah konsep penting yang dipakai di hampir semua aplikasi web.
> Laravel membuatnya mudah dengan `redirect()`.

---

## 11. Ringkasan Alur Tahap Ini

```
1. Ubah action di form: action="/products"
        |
        v
2. Pastikan route POST /products sudah ada
        |
        v
3. Isi method store() di controller:
   - validate()
   - Product::create()
   - redirect()->with('success', '...')
        |
        v
4. Tambah pesan sukses di index.blade.php
        |
        v
5. Tambah pesan error di create.blade.php
        |
        v
6. Tes: isi form -> simpan -> produk muncul di daftar
```

---

## 12. Hal-hal yang Perlu Diperhatikan

### 1. `Product::create()` butuh `$fillable`

Pastikan Model `Product` (`app/Models/Product.php`) sudah mendaftar kolom yang
boleh diisi:

```php
protected $fillable = [
    'name',
    'description',
    'price',
    'stock',
];
```

Jika tidak, `Product::create()` akan **mengabaikan** kolom yang tidak terdaftar
(demi keamanan mass assignment).

### 2. Validasi mencegah data kotor

Jangan pernah percaya data yang dikirim user. Selalu validasi sebelum menyimpan.
Pada tahap ini kita pakai validasi sederhana; di produksi, validasi bisa lebih kompleks.

### 3. `redirect()->with()` butuh `session('...')` di View

Pesan sukses dikirim via session, jadi View harus mengambilnya dengan `session('success')`.
Jika View tidak mengecek session, pesan tidak akan tampil meski disimpan.

### 4. Error validasi otomatis

Jika `$request->validate()` gagal, Laravel otomatis redirect ke halaman sebelumnya
dengan variabel `$errors` terisi. Kita tidak perlu menulis redirect error manual.

### 5. Jangan simpan data yang belum divalidasi

Hindari menyimpan data mentah dari `$request->all()`. Lebih aman simpan hasil
validasi `$validated` saja:

```php
// Baik
Product::create($validated);

// Kurang aman (simpan semua, termasuk yang tidak divalidasi)
Product::create($request->all());
```

---

## 13. Checklist Keberhasilan Tahap Ini

Centang (`x`) jika sudah selesai:

- [ ] `action` di form sudah diubah menjadi `action="/products"`.
- [ ] Route `POST /products` sudah ada dan menugaskan `ProductController@store`.
- [ ] Method `store()` di controller sudah berisi: validate, create, redirect.
- [ ] Halaman daftar produk (`index.blade.php`) menampilkan pesan sukses.
- [ ] Halaman form (`create.blade.php`) menampilkan pesan error validasi.
- [ ] Form dengan input valid berhasil disimpan, produk muncul di daftar.
- [ ] Form dengan input invalid ditolak, error muncul, data tidak tersimpan.
- [ ] Saya paham pola PRG: Post - Redirect - Get.

Jika semua sudah tercentang, fitur **Create (tambah produk)** sudah berfungsi penuh.

---

## 14. Penutup

Selamat! Kamu sudah berhasil menyelesaikan fitur **Create (tambah produk)**
secara penuh. Sekarang user bisa:

- Membuka form tambah produk.
- Mengisi data produk.
- Menyimpannya ke database.
- Melihat produk baru muncul di daftar.

Kombinasi Tahap 7 + 8 + 9 memberi kita fitur **Read + Create**:

- **Read** (lihat daftar produk) - Tahap 7
- **Create** (tambah produk baru) - Tahap 8 + 9

Di **tahap berikutnya**, kita akan belajar cara **melihat detail satu produk**.
Saat ini daftar produk hanya menampilkan info singkat. Nanti, saat user klik
salah satu produk, mereka akan dibawa ke halaman detail yang menampilkan
informasi lengkap produk tersebut.

### Jika sudah paham tahap ini, ketik:

```
lanjut
```

Jangan lanjut sebelum kamu benar-benar siap. Pelan-pelan, asal paham.
