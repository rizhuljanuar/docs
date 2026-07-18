# Tahap 8 (Opsional): CRUD Kategori

> **Catatan:** Tahap ini opsional. Kalau kamu mau langsung pakai kategori di form produk, skip ke **Tahap 9**. Tapi kalau kamu ingin admin bisa tambah/edit/hapus kategori sendiri (tanpa edit seeder), tahap ini penting.

## Tujuan Tahap Ini

Kita sudah punya tabel `categories`, model `Category`, dan data contoh dari seeder. Sekarang kita buat **CRUD lengkap** untuk kategori, sama seperti CRUD produk di materi sebelumnya.

CRUD = **C**reate, **R**ead, **U**pdate, **D**elete.

| Operasi | Arti                         | Method/Route              |
|---------|------------------------------|---------------------------|
| Create  | Tambah kategori baru         | `GET create` + `POST store`|
| Read    | Tampilkan daftar & detail    | `GET index` + `GET show`  |
| Update  | Ubah kategori                | `GET edit` + `PUT update` |
| Delete  | Hapus kategori               | `DELETE destroy`          |

## Langkah 1: Buat Controller CategoryController

Di terminal:

```bash
php artisan make:controller CategoryController --resource
```

Penjelasan:

| Bagian              | Arti                                                                      |
|---------------------|---------------------------------------------------------------------------|
| `make:controller`   | Perintah buat controller baru                                             |
| `CategoryController`| Nama controller (PascalCase + suffix `Controller`)                        |
| `--resource`        | Buat otomatis 7 method CRUD (index, create, store, show, edit, update, destroy) |

File yang dibuat:

```
app/Http/Controllers/CategoryController.php
```

Isi default (dipotong):

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Category;

class CategoryController extends Controller
{
    public function index() { /* ... */ }
    public function create() { /* ... */ }
    public function store(Request $request) { /* ... */ }
    public function show(Category $category) { /* ... */ }
    public function edit(Category $category) { /* ... */ }
    public function update(Request $request, Category $category) { /* ... */ }
    public function destroy(Category $category) { /* ... */ }
}
```

## Langkah 2: Isi Method `index()` (Tampilkan Daftar Kategori)

```php
public function index()
{
    $categories = Category::withCount('products')->latest()->get();
    return view('categories.index', compact('categories'));
}
```

Penjelasan:

| Bagian                     | Arti                                                            |
|----------------------------|-----------------------------------------------------------------|
| `Category::withCount(...)` | Ambil kategori sekaligus hitung jumlah produk per kategori      |
| `'products'`               | Nama relasi hasMany yang sudah kita buat di tahap 6             |
| `->latest()`               | Urutkan dari yang paling baru (berdasarkan `created_at`)        |
| `->get()`                  | Ambil semua data sebagai collection                             |
| `view('categories.index')` | Tampilkan file view `resources/views/categories/index.blade.php`|
| `compact('categories')`    | Kirim variabel `$categories` ke view                            |

## Langkah 3: Isi Method `create()` dan `store()` (Tambah Kategori)

```php
public function create()
{
    return view('categories.create');
}

public function store(Request $request)
{
    $validated = $request->validate([
        'name' => 'required|string|max:255|unique:categories,name',
    ]);

    Category::create($validated);

    return redirect()->route('categories.index')
                     ->with('success', 'Kategori berhasil ditambahkan.');
}
```

### Penjelasan Validasi

```php
$request->validate([
    'name' => 'required|string|max:255|unique:categories,name',
]);
```

Aturan validasi kolom `name`:

| Aturan                | Arti                                                              |
|-----------------------|-------------------------------------------------------------------|
| `required`            | Tidak boleh kosong                                                |
| `string`              | Harus teks                                                        |
| `max:255`             | Maksimal 255 karakter                                             |
| `unique:categories,name` | Nama harus unik di tabel `categories` kolom `name` (tidak boleh duplikat) |

Kalau validasi gagal, Laravel otomatis redirect balik ke form dengan pesan error.

## Langkah 4: Isi Method `show()` (Detail Kategori + Produk-produknya)

```php
public function show(Category $category)
{
    $category->load('products');
    return view('categories.show', compact('category'));
}
```

Penjelasan:

| Bagian                   | Arti                                                          |
|--------------------------|---------------------------------------------------------------|
| `Category $category`     | Route model binding: Laravel otomatis cari kategori by id      |
| `$category->load(...)`   | Load relasi produk (eager loading) sebelum kirim ke view       |
| `'products'`             | Nama method relasi `hasMany` di model Category                 |

Di view, kamu bisa akses `$category->products` untuk menampilkan semua produk di kategori ini.

## Langkah 5: Isi Method `edit()` dan `update()` (Ubah Kategori)

```php
public function edit(Category $category)
{
    return view('categories.edit', compact('category'));
}

public function update(Request $request, Category $category)
{
    $validated = $request->validate([
        'name' => 'required|string|max:255|unique:categories,name,' . $category->id,
    ]);

    $category->update($validated);

    return redirect()->route('categories.index')
                     ->with('success', 'Kategori berhasil diperbarui.');
}
```

### Penjelasan Unik: `unique:categories,name,{id}`

Perhatikan ada `$category->id` di akhir aturan unique:

```php
'name' => 'required|string|max:255|unique:categories,name,' . $category->id,
```

Arti: "Nama harus unik, **tapi abaikan baris dengan id ini** (baris yang sedang di-edit)."

Tanpa ini, kalau kamu cuma ubah deskripsi tanpa ubah nama, Laravel akan protes "nama sudah dipakai" karena nama itu sendiri yang sedang di-edit.

## Langkah 6: Isi Method `destroy()` (Hapus Kategori)

```php
public function destroy(Category $category)
{
    $category->delete();

    return redirect()->route('categories.index')
                     ->with('success', 'Kategori berhasil dihapus.');
}
```

Karena di tahap 5 kita pakai `onDelete('cascade')`, semua produk di kategori yang dihapus akan ikut terhapus otomatis oleh database.

> **Peringatan:** Ini bisa berbahaya di production. Sebelum hapus kategori yang punya banyak produk, biasanya tampilkan konfirmasi dulu di view. Untuk keamanan ekstra, bisa pakai `restrict` di migration dan validasi manual di controller.

## Langkah 7: Tambah Route Resource

Buka file `routes/web.php`. Tambahkan:

```php
use App\Http\Controllers\CategoryController;

Route::resource('categories', CategoryController::class);
```

Satu baris `Route::resource(...)` otomatis bikin 7 route:

| Method    | URL                   | Action   | Nama Route          |
|-----------|-----------------------|----------|---------------------|
| GET       | `/categories`         | index    | `categories.index`  |
| GET       | `/categories/create`  | create   | `categories.create` |
| POST      | `/categories`         | store    | `categories.store`  |
| GET       | `/categories/{id}`    | show     | `categories.show`   |
| GET       | `/categories/{id}/edit` | edit   | `categories.edit`   |
| PUT/PATCH | `/categories/{id}`    | update   | `categories.update` |
| DELETE    | `/categories/{id}`    | destroy  | `categories.destroy`|

Cek semua route yang dibuat:

```bash
php artisan route:list
```

## Langkah 8: Buat View (Blade)

Buat folder `resources/views/categories/` dan 4 file:

### `index.blade.php` (Daftar Kategori)

```blade
<h1>Daftar Kategori</h1>
<a href="{{ route('categories.create') }}">+ Tambah Kategori</a>

@if (session('success'))
    <p style="color: green;">{{ session('success') }}</p>
@endif

<table border="1">
    <tr>
        <th>ID</th>
        <th>Nama</th>
        <th>Jumlah Produk</th>
        <th>Aksi</th>
    </tr>
    @foreach ($categories as $category)
    <tr>
        <td>{{ $category->id }}</td>
        <td>{{ $category->name }}</td>
        <td>{{ $category->products_count }}</td>
        <td>
            <a href="{{ route('categories.show', $category) }}">Lihat</a>
            <a href="{{ route('categories.edit', $category) }}">Edit</a>
            <form action="{{ route('categories.destroy', $category) }}" method="POST" style="display:inline;">
                @csrf
                @method('DELETE')
                <button type="submit" onclick="return confirm('Hapus kategori ini? Semua produk ikut terhapus.')">Hapus</button>
            </form>
        </td>
    </tr>
    @endforeach
</table>
```

Penjelasan bagian penting:

| Bagian                | Arti                                                                 |
|-----------------------|----------------------------------------------------------------------|
| `session('success')`  | Tampilkan flash message sukses dari redirect                         |
| `$category->products_count` | Hasil dari `withCount('products')` di controller              |
| `@csrf`               | Token keamanan Laravel untuk form POST                               |
| `@method('DELETE')`   | Form HTML hanya support GET/POST, kita "paksa" jadi DELETE           |
| `confirm('...')`      | Dialog konfirmasi JavaScript sebelum hapus                           |

### `create.blade.php` (Form Tambah)

```blade
<h1>Tambah Kategori</h1>

<form action="{{ route('categories.store') }}" method="POST">
    @csrf
    <label>Nama Kategori:</label>
    <input type="text" name="name" value="{{ old('name') }}">
    @error('name') <span style="color: red;">{{ $message }}</span> @enderror
    <br>
    <button type="submit">Simpan</button>
</form>
```

Penjelasan:

| Bagian          | Arti                                                              |
|-----------------|-------------------------------------------------------------------|
| `old('name')`   | Tampilkan input lama kalau validasi gagal                         |
| `@error('name')`| Tampilkan pesan error validasi untuk kolom name                   |

### `edit.blade.php` (Form Ubah)

```blade
<h1>Edit Kategori</h1>

<form action="{{ route('categories.update', $category) }}" method="POST">
    @csrf
    @method('PUT')
    <label>Nama Kategori:</label>
    <input type="text" name="name" value="{{ old('name', $category->name) }}">
    @error('name') <span style="color: red;">{{ $message }}</span> @enderror
    <br>
    <button type="submit">Update</button>
</form>
```

Perbedaan dengan create:

- `action` mengarah ke `categories.update` dengan parameter `$category`.
- Tambah `@method('PUT')`.
- `value="{{ old('name', $category->name) }}"` -> tampilkan data lama, atau input sebelumnya kalau validasi gagal.

### `show.blade.php` (Detail + Produk-produknya)

```blade
<h1>{{ $category->name }}</h1>

<h2>Produk di Kategori Ini</h2>
<ul>
    @foreach ($category->products as $product)
        <li>{{ $product->name }} - Rp {{ number_format($product->price, 0, ',', '.') }}</li>
    @endforeach
</ul>

<a href="{{ route('categories.index') }}">Kembali</a>
```

Di sinilah kegunaan relasi `hasMany` terlihat: `$category->products` otomatis ambil semua produk yang `category_id`-nya sama dengan kategori ini.

## Coba di Browser

Akses URL:

```
http://localhost:8000/categories
```

Kamu akan melihat daftar kategori, bisa tambah, edit, lihat detail, dan hapus.

## Tips Penting

### 1. Pakai Route Model Binding

Perhatikan method `show(Category $category)`. Laravel otomatis cari kategori berdasarkan id di URL. Tidak perlu tulis `Category::findOrFail($id)` manual.

Kalau id tidak ditemukan, Laravel otomatis return halaman 404.

### 2. Flash Message

`->with('success', '...')` di controller + `session('success')` di view = cara Laravel menyampaikan pesan sukses sekali setelah redirect.

### 3. Eager Loading vs Lazy Loading

Di `show()`, kita pakai `$category->load('products')` (eager loading eksplisit). Ini lebih efisien daripada memanggil `$category->products` di view tanpa load dulu, terutama untuk banyak data (mencegah N+1 query problem).

## Inti Pelajaran Tahap 8

> CRUD kategori = pola yang sama dengan CRUD produk, tinggal ganti model dan view. Satu baris `Route::resource(...)` menghasilkan 7 route otomatis.

Yang sudah kita lakukan:

1. Buat controller resource: `php artisan make:controller CategoryController --resource`.
2. Isi 7 method CRUD (index, create, store, show, edit, update, destroy).
3. Tambah `Route::resource('categories', CategoryController::class)`.
4. Buat 4 view: index, create, edit, show.
5. Pakai `withCount('products')` untuk tampilkan jumlah produk per kategori.
6. Pakai `$category->products` (relasi hasMany) untuk lihat produk di kategori tertentu.

Sekarang admin bisa kelola kategori mandiri, tanpa edit seeder.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke langkah berikutnya: **menampilkan dropdown kategori di form produk**?

Di tahap 9 kita akan:

- Mengubah controller produk untuk kirim data kategori ke view.
- Tambah elemen `<select>` di form create/edit produk.
- Supaya saat user tambah produk, dia bisa pilih kategori dari dropdown.

Ketik **"lanjut"** kalau siap.
