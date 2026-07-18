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

Pastikan ada **route resource** atau **route individual** untuk CRUD
produk. Cara paling umum di Laravel:

```php
use App\Http\Controllers\ProductController;
use Illuminate\Support\Facades\Route;

Route::resource('products', ProductController::class);
```

**Arti `Route::resource`**: Laravel otomatis bikin **7 route** sekaligus:

| Method HTTP | URL                  | Method Controller | Fungsi               |
|-------------|----------------------|-------------------|----------------------|
| GET         | `/products`          | `index()`         | Lihat daftar produk  |
| GET         | `/products/create`   | `create()`        | Form tambah          |
| POST        | `/products`          | `store()`         | Simpan produk baru   |
| GET         | `/products/{id}`     | `show()`          | Detail satu produk   |
| GET         | `/products/{id}/edit`| `edit()`          | Form edit            |
| PUT/PATCH   | `/products/{id}`     | `update()`        | Update produk        |
| DELETE      | `/products/{id}`     | `destroy()`       | Hapus produk         |

> Yang penting bagi validasi: **POST ke `/products`** (jalankan
> `store()`) dan **PUT ke `/products/{id}`** (jalankan `update()`).
> Keduanya harus memakai `StoreProductRequest`.

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
└── web.php                              ← route resource products
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
semua integrasi selesai:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\StoreProductRequest;
use App\Models\Product;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\View\View;

class ProductController extends Controller
{
    // Tampilkan daftar produk
    public function index(): View
    {
        $products = Product::latest()->get();
        return view('products.index', compact('products'));
    }

    // Tampilkan form tambah produk
    public function create(): View
    {
        return view('products.create');
    }

    // Simpan produk baru
    public function store(StoreProductRequest $request): RedirectResponse
    {
        Product::create($request->validated());

        return redirect('/products')
            ->with('success', 'Produk berhasil ditambahkan.');
    }

    // Tampilkan detail produk
    public function show(Product $product): View
    {
        return view('products.show', compact('product'));
    }

    // Tampilkan form edit produk
    public function edit(Product $product): View
    {
        return view('products.edit', compact('product'));
    }

    // Update produk
    public function update(StoreProductRequest $request, Product $product): RedirectResponse
    {
        $product->update($request->validated());

        return redirect('/products')
            ->with('success', 'Produk berhasil diupdate.');
    }

    // Hapus produk
    public function destroy(Product $product): RedirectResponse
    {
        $product->delete();

        return redirect('/products')
            ->with('success', 'Produk berhasil dihapus.');
    }
}
```

### Catatan penting

1. **`use App\Http\Requests\StoreProductRequest;`** di atas file:
   import class FormRequest.
2. **`store(StoreProductRequest $request)`** dan **`update(StoreProductRequest $request, Product $product)`**:
   Kedua method ini **memakai tipe yang sama** (StoreProductRequest),
   jadi aturan validasi dijalankan **otomatis** sebelum isi methodnya.
3. **`$request->validated()`**: Ambil hanya field yang sudah lolos
   validasi (lebih aman dari `$request->all()`).
4. **Route model binding**: Pakai `Product $product` di parameter
   (bukan `$id`) → Laravel otomatis ambil produk berdasarkan ID di
   URL. Ini lebih rapi daripada `Product::findOrFail($id)`.

> Kalau controller kamu sebelumnya pakai `$id` (bukan route model
> binding), tidak masalah. Tinggal sesuaikan. Yang penting
> `StoreProductRequest` dipakai di `store()` dan `update()`.

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
            'nama'      => 'required',
            'harga'     => 'required|numeric|min:0',
            'stok'      => 'required|integer|min:0',
            'deskripsi' => 'nullable|min:10',
        ];
    }
}
```

---

## 6. Kode Lengkap create.blade.php

```blade
<!DOCTYPE html>
<html>
<head>
    <title>Tambah Produk</title>
</head>
<body>
    <h1>Tambah Produk</h1>

    <a href="/products">&larr; Kembali ke Daftar Produk</a>

    <br><br>

    {{-- Ringkasan error di atas --}}
    @if ($errors->any())
        <div style="background-color: #fee; border: 1px solid red; padding: 10px;">
            <strong>Maaf, ada masalah:</strong>
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <form action="/products" method="POST">
        @csrf

        <div>
            <label>Nama Produk:</label><br>
            <input type="text" name="nama" value="{{ old('nama') }}">
            @error('nama')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <div>
            <label>Harga:</label><br>
            <input type="number" name="harga" step="0.01" value="{{ old('harga') }}">
            @error('harga')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <div>
            <label>Stok:</label><br>
            <input type="number" name="stok" value="{{ old('stok') }}">
            @error('stok')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <div>
            <label>Deskripsi:</label><br>
            <textarea name="deskripsi">{{ old('deskripsi') }}</textarea>
            @error('deskripsi')
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
<html>
<head>
    <title>Edit Produk</title>
</head>
<body>
    <h1>Edit Produk: {{ $product->nama }}</h1>

    <a href="/products">&larr; Kembali ke Daftar Produk</a>

    <br><br>

    @if ($errors->any())
        <div style="background-color: #fee; border: 1px solid red; padding: 10px;">
            <strong>Maaf, ada masalah:</strong>
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

        <div>
            <label>Nama Produk:</label><br>
            <input type="text" name="nama" value="{{ old('nama', $product->nama) }}">
            @error('nama')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <div>
            <label>Harga:</label><br>
            <input type="number" name="harga" step="0.01" value="{{ old('harga', $product->harga) }}">
            @error('harga')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <div>
            <label>Stok:</label><br>
            <input type="number" name="stok" value="{{ old('stok', $product->stok) }}">
            @error('stok')
                <span style="color: red; font-size: 12px;">{{ $message }}</span>
            @enderror
        </div>

        <div>
            <label>Deskripsi:</label><br>
            <textarea name="deskripsi">{{ old('deskripsi', $product->deskripsi) }}</textarea>
            @error('deskripsi')
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
   - nama: `Buku Tulis`
   - harga: `5000`
   - stok: `20`
   - deskripsi: `Buku tulis 50 lembar berkualitas`
3. Klik **Simpan**.

**Ekspektasi:**
- Diarahkan ke halaman daftar produk.
- Produk baru muncul di daftar.

### Skenario B: Tambah produk dengan data SALAH

1. Buka `http://localhost:8000/products/create`.
2. Isi:
   - nama: **kosong**
   - harga: `-5000`
   - stok: `2.5`
   - deskripsi: `ok`
3. Klik **Simpan**.

**Ekspektasi:**
- Halaman **kembali ke form**.
- Kotak error merah muncul di atas.
- Pesan error muncul di tiap field yang salah.
- Field yang sudah kamu isi (misal deskripsi `ok`) tetap ada di
  textarea (karena `old()` bekerja).

### Skenario C: Edit produk dengan data BENAR

1. Buka halaman edit produk, misal: `http://localhost:8000/products/1/edit`.
2. Ubah:
   - harga: `6000` (dari sebelumnya 5000)
3. Klik **Update**.

**Ekspektasi:**
- Diarahkan ke halaman daftar produk.
- Harga produk berubah jadi 6000.

### Skenario D: Edit produk dengan data SALAH

1. Buka halaman edit produk: `http://localhost:8000/products/1/edit`.
2. Hapus isi field nama (kosongkan).
3. Ubah harga jadi `-1000`.
4. Klik **Update**.

**Ekspektasi:**
- Halaman **kembali ke form edit**.
- Pesan error muncul di field nama dan harga.
- Field lain (stok, deskripsi) **tetap berisi data dari database**
  karena tidak kamu ubah.
- Field yang kamu ubah (nama kosong, harga -1000) tetap ada sebagai
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
- Pastikan `Route::resource('products', ProductController::class);`
  ada di `routes/web.php`.

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

1. **Tambah field baru**: tambahkan field `kategori` (string) ke tabel
   products (lewat migration baru). Lalu tambahkan aturan validasi
   `'kategori' => 'required|in:elektronik,makanan,pakaian'` di
   `StoreProductRequest`. Coba input dengan kategori yang tidak ada
   di daftar. Apa yang terjadi?

2. **Pisahkan FormRequest**: bikin dua FormRequest terpisah,
   `StoreProductRequest` dan `UpdateProductRequest`. Buat aturannya
   **sedikit beda** (misal di update, nama boleh sama dengan nama
   sebelumnya → rules `unique` ignore ID). Latihan ini mensimulasikan
   kasus nyata.

3. **Custom pesan error** (preview Tahap 7): tambahkan method
   `messages()` di `StoreProductRequest`:

   ```php
   public function messages(): array
   {
       return [
           'nama.required' => 'Nama produk wajib diisi ya.',
           'harga.min'     => 'Harga tidak boleh minus.',
       ];
   }
   ```

   Coba input salah dan lihat pesannya sekarang pakai bahasa Indonesia.

---

## 12. Ringkasan Bagian 6

- **Integrasi penuh** menggabungkan: FormRequest + Controller + Route +
  View menjadi satu sistem yang bekerja.
- **Route resource** memberi 7 route otomatis untuk CRUD produk.
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
