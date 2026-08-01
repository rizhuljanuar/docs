# Bagian 6: Menerapkan FormRequest di Controller CRUD (Integrasi Penuh)

> Modul: A. Level Dasar — Fondasi Laravel, MVC, CRUD
> Topik: Request Validation dengan FormRequest
> Prasyarat: Selesai membaca `01` sampai `05`

---

## Tujuan Bagian Ini

Di Tahap 1-5 kamu sudah belajar semua **bagian-bagian**:
- FormRequest (`StoreProductRequest`)
- Controller (`store()` dan `update()`)
- View (form dengan `@error` dan `old()`)

Sekarang di Tahap 6, kita **rangkai semua bagian jadi satu sistem
utuh** dan **uji end-to-end** supaya kamu yakin semuanya bekerja
sesuai harapan.

> Tujuan akhir Tahap 6: kamu punya **CRUD produk lengkap dengan
> validasi** yang bekerja di semua operasi (tambah dan edit).

---

## 1. Checklist Integrasi

Sebelum mulai, pastikan kamu punya **semua komponen** berikut.
Centang satu per satu:

- [ ] File `app/Http/Requests/StoreProductRequest.php` ada dan berisi `rules()`.
- [ ] File `app/Http/Controllers/ProductController.php` ada.
- [ ] Method `store()` dan `update()` di controller sudah pakai
      `StoreProductRequest $request` (bukan `Request $request`).
- [ ] File `resources/views/products/create.blade.php` ada dengan form
      yang punya `@error` dan `old()`.
- [ ] File `resources/views/products/edit.blade.php` ada dengan form
      yang punya `@error` dan `old('field', $product->field)`.
- [ ] Route di `routes/web.php` sudah mendaftarkan CRUD produk.

Kalau ada yang belum, balik ke Tahap 3/4/5 dulu.

---

## 2. Cek Route (routes/web.php)

Buka file:

```
routes/web.php
```

Pastikan ada **route individual** (seperti yang kamu tulis di **Materi 1,
Tahap 5 dan Tahap 6**) untuk CRUD produk:

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ProductController;

Route::get('/', function () {
    return 'Selamat datang di Toko Produk';
});

// Route produk diarahkan ke ProductController
Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/create', [ProductController::class, 'create']);
Route::post('/products', [ProductController::class, 'store']);
Route::get('/products/{id}/edit', [ProductController::class, 'edit']);
Route::put('/products/{id}', [ProductController::class, 'update']);
Route::delete('/products/{id}', [ProductController::class, 'destroy']);
Route::get('/products/{id}', [ProductController::class, 'show']);
```

> Yang penting bagi validasi: **POST ke `/products`** (jalankan
> `store()`) dan **PUT ke `/products/{id}`** (jalankan `update()`).
> Keduanya harus memakai `StoreProductRequest`.

**Alternatif lebih singkat**: Kalau mau, kamu boleh mengganti 7 route
di atas dengan satu baris `Route::resource`:

```php
Route::resource('products', ProductController::class);
```

Ini otomatis bikin **7 route** yang sama. Tapi untuk konsistensi
dengan **Materi 1**, kita tetap pakai route individual.

**Cara cek route aktif**: jalankan di terminal:

```bash
php artisan route:list --path=products
```

Ini akan menampilkan semua route produk yang terdaftar.

---

## 3. Struktur Akhir yang Diharapkan

Sebelum menulis kode, mari lihat **struktur file lengkap** setelah
integrasi selesai:

```
app/
├── Http/
│   ├── Controllers/
│   │   └── ProductController.php        ← pakai StoreProductRequest
│   └── Requests/
│       └── StoreProductRequest.php      ← aturan validasi
routes/
└── web.php                              ← route produk (individual atau resource)
resources/
└── views/
    └── products/
        ├── index.blade.php              ← daftar produk
        ├── create.blade.php             ← form tambah (+ @error, old)
        ├── show.blade.php               ← detail produk
        └── edit.blade.php               ← form edit (+ @error, old)
```

---

## 4. Kode Lengkap ProductController.php

Berikut **versi lengkap dan final** dari controller produk setelah
semua integrasi selesai. Ini adalah kelanjutan dari **Materi 1: CRUD
Data Produk**, jadi struktur route, model binding, dan redirect
tetap konsisten dengan apa yang sudah kamu buat sebelumnya.

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\StoreProductRequest;
use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    // Tampilkan daftar produk
    public function index()
    {
        $products = Product::all();

        return view('products.index', compact('products'));
    }

    // Tampilkan form tambah produk
    public function create()
    {
        return view('products.create');
    }

    // Simpan produk baru
    public function store(StoreProductRequest $request)
    {
        Product::create($request->validated());

        return redirect('/products')
            ->with('success', 'Produk berhasil ditambahkan.');
    }

    // Tampilkan detail produk
    public function show($id)
    {
        $product = Product::findOrFail($id);

        return view('products.show', compact('product'));
    }

    // Tampilkan form edit produk
    public function edit($id)
    {
        $product = Product::findOrFail($id);

        return view('products.edit', compact('product'));
    }

    // Update produk
    public function update(StoreProductRequest $request, $id)
    {
        $product = Product::findOrFail($id);
        $product->update($request->validated());

        return redirect('/products/' . $product->id)
            ->with('success', 'Produk berhasil diperbarui.');
    }

    // Hapus produk
    public function destroy($id)
    {
        $product = Product::findOrFail($id);
        $product->delete();

        return redirect('/products')
            ->with('success', 'Produk berhasil dihapus.');
    }
}
```

### Catatan penting

1. **`use App\Http\Requests\StoreProductRequest;`** di atas file:
   import class FormRequest.
2. **`store(StoreProductRequest $request)`** dan **`update(StoreProductRequest $request, $id)`**:
   Kedua method ini **memakai tipe yang sama** (StoreProductRequest),
   jadi aturan validasi dijalankan **otomatis** sebelum isi methodnya.
3. **`$request->validated()`**: Ambil hanya field yang sudah lolos
   validasi (lebih aman dari `$request->all()`).
4. **Struktur konsisten dengan Materi 1**: Method `show`, `edit`,
   `update`, dan `destroy` tetap memakai parameter `$id` (bukan route
   model binding `Product $product`), sama seperti yang kamu tulis di
   **Materi 1 (Tahap 6, 10, 11, 12, 13)**.
5. **Redirect update tetap ke detail produk** (`/products/{id}`),
   sama seperti **Materi 1 (Tahap 12)**.

> Kalau nanti kamu mau refactor ke route model binding (`Product $product`),
   itu boleh. Tapi untuk konsistensi dengan materi sebelumnya, kita
   tetap pakai `$id`.

---

## 5. Kode Lengkap StoreProductRequest.php

Sebagai pengingat, ini isi final FormRequest kamu:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreProductRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name'        => 'required|min:3',
            'price'       => 'required|numeric|min:0',
            'stock'       => 'required|integer|min:0',
            'description' => 'nullable|min:10',
        ];
    }
}
```

---

## 6. Kode Lengkap create.blade.php

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tambah Produk</title>
</head>
<body>
    <h1>Tambah Produk</h1>

    <a href="/products">&larr; Kembali ke Daftar Produk</a>

    <br><br>

    {{-- Ringkasan error di atas --}}
    @if ($errors->any())
        <div style="background-color: #f8d7da; color: #721c24; padding: 10px 15px; border: 1px solid #f5c6cb; border-radius: 4px; margin-bottom: 15px;">
            <strong>Maaf, ada masalah:</strong>
            <ul style="margin: 5px 0 0 20px;">
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <form action="/products" method="POST">
        @csrf

        <div>
            <label for="name">Nama Produk</label><br>
            <input type="text" name="name" id="name" value="{{ old('name') }}">
            @error('name')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <div>
            <label for="price">Harga (Rp)</label><br>
            <input type="number" name="price" id="price" min="0" value="{{ old('price') }}">
            @error('price')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <div>
            <label for="stock">Stok</label><br>
            <input type="number" name="stock" id="stock" min="0" value="{{ old('stock') }}">
            @error('stock')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <div>
            <label for="description">Deskripsi</label><br>
            <textarea name="description" id="description">{{ old('description') }}</textarea>
            @error('description')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <button type="submit">Simpan</button>
    </form>
</body>
</html>
```

---

## 7. Kode Lengkap edit.blade.php

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Edit Produk</title>
</head>
<body>
    <h1>Edit Produk: {{ $product->name }}</h1>

    <a href="/products">&larr; Kembali ke Daftar Produk</a>

    <br><br>

    @if ($errors->any())
        <div style="background-color: #f8d7da; color: #721c24; padding: 10px 15px; border: 1px solid #f5c6cb; border-radius: 4px; margin-bottom: 15px;">
            <strong>Maaf, ada masalah:</strong>
            <ul style="margin: 5px 0 0 20px;">
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <form action="/products/{{ $product->id }}" method="POST">
        @csrf
        @method('PUT')

        <div>
            <label for="name">Nama Produk</label><br>
            <input type="text" name="name" id="name" value="{{ old('name', $product->name) }}">
            @error('name')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <div>
            <label for="price">Harga (Rp)</label><br>
            <input type="number" name="price" id="price" min="0" value="{{ old('price', $product->price) }}">
            @error('price')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <div>
            <label for="stock">Stok</label><br>
            <input type="number" name="stock" id="stock" min="0" value="{{ old('stock', $product->stock) }}">
            @error('stock')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <div>
            <label for="description">Deskripsi</label><br>
            <textarea name="description" id="description">{{ old('description', $product->description) }}</textarea>
            @error('description')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <button type="submit">Update</button>
    </form>
</body>
</html>
```

### Beda `create` dan `edit` (peringatan ulang):

| Bagian                 | create.blade.php                  | edit.blade.php                                |
|------------------------|-----------------------------------|-----------------------------------------------|
| Action form            | `action="/products"`              | `action="/products/{{ $product->id }}"`       |
| HTTP method            | `POST` (default)                  | `POST` + `@method('PUT')`                     |
| Nilai input            | `value="{{ old('field') }}"`      | `value="{{ old('field', $product->field) }}"` |
| Tombol submit          | "Simpan"                          | "Update"                                      |

---

## 8. Uji End-to-End (Wajib Dilakukan!)

Setelah semua kode siap, jalankan **skenario uji berikut** satu per
satu. Ini penting untuk memastikan integrasi benar-benar bekerja.

### Skenario A: Tambah produk dengan data BENAR

1. Buka `http://localhost:8000/products/create`.
2. Isi:
   - name: `Buku Tulis`
   - price: `5000`
   - stock: `20`
   - description: `Buku tulis 50 lembar berkualitas`
3. Klik **Simpan**.

**Ekspektasi:**
- Diarahkan ke halaman daftar produk.
- Produk baru muncul di daftar.

### Skenario B: Tambah produk dengan data SALAH

1. Buka `http://localhost:8000/products/create`.
2. Isi:
   - name: **kosong**
   - price: `-5000`
   - stock: `2.5`
   - description: `ok`
3. Klik **Simpan**.

**Ekspektasi:**
- Halaman **kembali ke form**.
- Kotak error merah muncul di atas.
- Pesan error muncul di tiap field yang salah.
- Field yang sudah kamu isi (misal description `ok`) tetap ada di
  textarea (karena `old()` bekerja).

### Skenario C: Edit produk dengan data BENAR

1. Buka halaman edit produk, misal: `http://localhost:8000/products/1/edit`.
2. Ubah:
   - price: `6000` (dari sebelumnya 5000)
3. Klik **Update**.

**Ekspektasi:**
- Diarahkan ke halaman detail produk (`/products/1`), sesuai Materi 1 Tahap 12.
- Harga produk berubah jadi 6000.

### Skenario D: Edit produk dengan data SALAH

1. Buka halaman edit produk: `http://localhost:8000/products/1/edit`.
2. Hapus isi field name (kosongkan).
3. Ubah price jadi `-1000`.
4. Klik **Update**.

**Ekspektasi:**
- Halaman **kembali ke form edit**.
- Pesan error muncul di field name dan price.
- Field lain (stock, description) **tetap berisi data dari database**
  karena tidak kamu ubah.
- Field yang kamu ubah (name kosong, price -1000) tetap ada sebagai
  feedback "inilah yang salah, perbaiki".

### Skenario E: Coba kirim data ekstra (mass assignment test)

Ini uji keamanan. Buka **DevTools browser → Console**, jalankan:

```javascript
document.querySelector('form').insertAdjacentHTML('beforeend',
    '<input type="hidden" name="id" value="999">');
document.querySelector('form').submit();
```

Ini menambahkan field `id=999` secara sembunyi ke form.

**Ekspektasi**: Walaupun field `id` dikirim, data tersimpan tetap
**aman** karena:
- `$request->validated()` hanya mengambil field yang ada di `rules()`.
- Field `id` tidak ada di rules, jadi diabaikan.

> Inilah alasan kita pakai `$request->validated()` dan bukan
> `$request->all()`. Ini mencegah serangan **mass assignment**.

---

## 9. Penanganan Masalah Umum

Kalau ada yang tidak bekerja, cek daftar berikut:

### Masalah: Halaman 404 saat buka `/products/create`

**Kemungkinan penyebab**:
- Route belum terdaftar. Cek `php artisan route:list --path=products`.
- Pastikan route `/products/create` ditulis **sebelum** `/products/{id}`
  (seperti yang kamu pelajari di **Materi 1, Tahap 5 dan Tahap 8**).
- Kalau pakai `Route::resource`, pastikan
  `Route::resource('products', ProductController::class);` ada di
  `routes/web.php`.

### Masalah: Error 500 saat submit form

**Kemungkinan penyebab**:
- Lupa `@csrf` di form. Tambahkan `<@csrf>` setelah `<form>`.
- Lupa `use App\Http\Requests\StoreProductRequest;` di controller.

### Masalah: Error 403 Forbidden

**Kemungkinan penyebab**:
- Method `authorize()` di `StoreProductRequest` masih `return false;`.
  Ubah jadi `return true;` selama belajar.

### Masalah: Validasi tidak jalan (data salah tetap tersimpan)

**Kemungkinan penyebab**:
- Controller masih memakai `Request $request` (bukan
  `StoreProductRequest $request`). Cek di method `store()` dan `update()`.
- Kamu masih pakai `$request->validate([...])` di controller. Hapus
  karena FormRequest sudah handle.

### Masalah: Pesan error tidak muncul di form

**Kemungkinan penyebab**:
- Lupa tambahkan blok `@error('field')` di Blade.
- Lupa pakai `$errors->any()` di atas form.

### Masalah: Form kehilangan input saat validasi gagal

**Kemungkinan penyebab**:
- Lupa pakai `value="{{ old('field') }}"` di input.
- Untuk textarea, lupa taruh `old()` **di dalam tag**.

---

## 10. Visualisasi Alur Akhir (Satu Halaman)

```
USER ISI FORM
     │
     ▼
BROWSER KIRIM POST REQUEST
     │
     ▼
ROUTE /products (POST) ──→ ProductController@store
                              │
                              ▼
                  ┌─ parameter: StoreProductRequest
                  │
                  ▼
        LARAVEL OTOMATIS JALANKAN VALIDASI
                  │
        ┌─────────┴─────────┐
        │                   │
   VALID?              TIDAK VALID?
        │                   │
        ▼                   ▼
   Product::create()   kembali ke form
        │               + pesan error
        │               + old input
        ▼
   redirect ke /products
   + flash success
```

Alur ini sama untuk **tambah (store)** dan **edit (update)**.
Perbedaannya hanya di URL tujuan (POST ke `/products` vs PUT ke
`/products/{id}`).

---

## 11. Latihan Mandiri

1. **Tambah field baru**: tambahkan field `category` (string) ke tabel
   products (lewat migration baru). Lalu tambahkan aturan validasi
   `'category' => 'required|in:elektronik,makanan,pakaian'` di
   `StoreProductRequest`. Coba input dengan kategori yang tidak ada
   di daftar. Apa yang terjadi?

2. **Pisahkan FormRequest**: bikin dua FormRequest terpisah,
   `StoreProductRequest` dan `UpdateProductRequest`. Buat aturannya
   **sedikit beda** (misal di update, name boleh sama dengan name
   sebelumnya → rules `unique` ignore ID). Latihan ini mensimulasikan
   kasus nyata.

3. **Custom pesan error** (preview Tahap 7): tambahkan method
   `messages()` di `StoreProductRequest`:

   ```php
   public function messages(): array
   {
       return [
           'name.required'  => 'Nama produk wajib diisi ya.',
           'price.min'      => 'Harga tidak boleh minus.',
       ];
   }
   ```

   Coba input salah dan lihat pesannya sekarang pakai bahasa Indonesia.

---

## 12. Ringkasan Bagian 6

- **Integrasi penuh** menggabungkan: FormRequest + Controller + Route +
  View menjadi satu sistem yang bekerja.
- **Route** (individual atau `Route::resource`) mendaftarkan 7 endpoint
  CRUD produk.
- **Controller** memakai `StoreProductRequest $request` di method
  `store()` dan `update()` → validasi berjalan otomatis.
- **Form create** pakai `value="{{ old('field') }}"`.
- **Form edit** pakai `value="{{ old('field', $product->field) }}"`.
- **`$request->validated()`** lebih aman dari `$request->all()` karena
  hanya ambil field yang ada di rules (cegah mass assignment).
- **Uji end-to-end** wajib dilakukan untuk semua skenario: data benar,
  data salah, di create, di update, dan serangan mass assignment.
- Penanganan masalah umum sudah dibahas (404, 500, 403, validasi tidak
  jalan, error tidak muncul, input hilang).

---

## 13. Selamat!

Kalau semua skenario uji di Tahap 6 berhasil, kamu sudah punya:

> **Sistem CRUD produk lengkap dengan validasi yang bekerja di
> tambah produk dan edit produk.**

Ini adalah **capaian besar**. Kamu sudah melewati konsep yang banyak
developer pemula sering skip (validasi), dan sekarang kamu paham
**struktur best practice** untuk validasi di Laravel.

---

> **Berhenti di sini.**
>
> Pada bagian terakhir (Tahap 7), kita akan **review seluruh materi**
> + pelajari **best practice** dan **daftar rules Laravel yang sering
> dipakai** + cara **kustomisasi pesan error** ke bahasa Indonesia.
>
> **Apakah kamu ingin lanjut ke langkah berikutnya:
> Tahap 7 — Review, Best Practice, dan Custom Pesan Error?**
