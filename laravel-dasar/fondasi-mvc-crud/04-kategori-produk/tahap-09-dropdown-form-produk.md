# Tahap 9: Dropdown Kategori di Form Produk

## Tujuan Tahap Ini

Sekarang admin bisa tambah produk. Tapi di form produk, belum ada cara untuk **memilih kategori**. Kita perlu tambah elemen **dropdown** berisi daftar kategori, supaya tiap produk langsung dihubungkan ke satu kategori.

## Analogi Sederhana: Formulir Pendaftaran Sekolah

Bayangkan kamu mengisi formulir pendaftaran sekolah.

- Ada kolom "Nama": kamu ketik bebas.
- Ada kolom "Tanggal Lahir": kamu pilih dari kalender.
- Ada kolom **"Kelas"**: kamu **tidak mengetik bebas**, tapi **memilih dari daftar** yang sudah ditentukan (X-A, X-B, X-C).

Kenapa kelas harus pilih dari daftar? Karena kalau diketik bebas, bisa terjadi typo (X-A vs X.A vs X-A Reguler). Pilih dari daftar = data rapi dan konsisten.

Di Laravel, elemen HTML untuk "pilih dari daftar" adalah `<select>`. Kita pakai untuk pilih kategori.

## Apa itu Elemen `<select>` di HTML?

```html
<select name="category_id">
    <option value="1">Elektronik</option>
    <option value="2">Pakaian</option>
    <option value="3">Makanan</option>
    <option value="4">Buku</option>
</select>
```

Penjelasan:

| Bagian                       | Arti                                                              |
|------------------------------|-------------------------------------------------------------------|
| `<select name="category_id">`| Wadah dropdown, nama field saat dikirim ke server                 |
| `<option value="1">...</option>` | Salah satu pilihan. `value` yang dikirim, isi teks yang tampil|
| `value="1"`                  | Nilai yang dikirim saat dipilih (id kategori)                     |
| `Elektronik`                 | Teks yang tampil ke user (nama kategori)                          |

Saat user pilih "Pakaian" lalu submit, browser mengirim `category_id=2` ke server.

Tapi kita tidak ingin menulis setiap `<option>` manual. Kita ingin **otomatis dibuat dari database**. Itulah yang akan kita lakukan.

## Langkah 1: Ubah Controller ProductController

Buka file `app/Http/Controllers/ProductController.php`. Di method `create()` dan `edit()`, kirim data kategori ke view.

> **Catatan:** Struktur controller ini mengikuti **Materi 1, 2, dan 3**.
> Kita tetap memakai parameter `$id` (bukan route model binding) dan URL
> biasa seperti `/products` (bukan named route), supaya konsisten.

### Method `create()`

```php
public function create()
{
    $categories = Category::orderBy('name')->get();
    return view('products.create', compact('categories'));
}
```

### Method `edit()`

```php
public function edit($id)
{
    $product = Product::findOrFail($id);
    $categories = Category::orderBy('name')->get();
    return view('products.edit', compact('categories', 'product'));
}
```

Penjelasan:

| Bagian                       | Arti                                                            |
|------------------------------|-----------------------------------------------------------------|
| `Category::orderBy('name')`  | Ambil semua kategori, urutkan berdasarkan nama                  |
| `->get()`                    | Eksekusi query, kembalikan collection                           |
| `compact('categories')`      | Kirim variabel `$categories` ke view                            |
| `compact('categories', 'product')` | Kirim dua variabel ke view (untuk edit)                  |
| `Product::findOrFail($id)`   | Cari produk berdasarkan ID (sama seperti Materi 1 Tahap 11)     |

Kenapa kirim variabel `$categories` ke view? Supaya di form kita bisa looping daftar kategori untuk bikin `<option>`.

Jangan lupa import di atas file:

```php
use App\Models\Category;
```

## Langkah 2: Tambah Validasi `category_id` di `store()` dan `update()`

Buka method `store()` di ProductController. Aturan untuk `name`, `price`,
`stock`, `description`, dan `image` **tetap sama persis** seperti di Materi 2
dan 3. Kita hanya **menambah satu baris** baru: `'category_id' => ...`.

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'name'        => 'required|min:3',
        'price'       => 'required|numeric|min:0',
        'stock'       => 'required|integer|min:0',
        'description' => 'nullable|min:10',

        // Dari Materi 3 (upload gambar):
        'image'       => 'required|image|mimes:jpg,jpeg,png,webp|max:2048',

        // TAMBAHAN BARU di Materi 4:
        'category_id' => 'required|exists:categories,id',
    ]);

    // Dari Materi 3: simpan gambar dulu
    $validated['image'] = $request->file('image')->store('products', 'public');

    Product::create($validated);

    return redirect('/products')->with('success', 'Produk berhasil ditambahkan.');
}
```

Lakukan hal sama di method `update()`:

```php
public function update(Request $request, $id)
{
    $product = Product::findOrFail($id);

    $validated = $request->validate([
        'name'        => 'required|min:3',
        'price'       => 'required|numeric|min:0',
        'stock'       => 'required|integer|min:0',
        'description' => 'nullable|min:10',

        // Di update, image bisa nullable (boleh tidak diganti)
        'image'       => 'nullable|image|mimes:jpg,jpeg,png,webp|max:2048',

        // TAMBAHAN BARU di Materi 4:
        'category_id' => 'required|exists:categories,id',
    ]);

    // Dari Materi 3: kalau ada file baru, hapus gambar lama lalu simpan baru
    if ($request->hasFile('image')) {
        if ($product->image && Storage::disk('public')->exists($product->image)) {
            Storage::disk('public')->delete($product->image);
        }
        $validated['image'] = $request->file('image')->store('products', 'public');
    }

    $product->update($validated);

    return redirect('/products/' . $product->id)->with('success', 'Produk berhasil diperbarui.');
}
```

### Penjelasan Aturan Validasi `category_id`

```php
'category_id' => 'required|exists:categories,id',
```

Aturan bagian per bagian:

| Aturan             | Arti                                                                              |
|--------------------|-----------------------------------------------------------------------------------|
| `required`         | Wajib dipilih, tidak boleh kosong                                                |
| `exists:categories,id` | Nilai `category_id` harus ada di kolom `id` tabel `categories`               |

Kenapa penting? Kalau user "iseng" kirim `category_id=99` lewat manipulasi form, padahal kategori 99 tidak ada, Laravel otomatis tolak dengan pesan error. Ini mencegah foreign key violation di database.

## Langkah 3: Tambah Dropdown di Form Create

Buka file `resources/views/products/create.blade.php`. Form ini sudah berisi
field `name`, `price`, `stock`, `description`, `image` dari materi sebelumnya.
Kita **tambah satu field baru**: dropdown kategori.

```blade
<form action="/products" method="POST" enctype="multipart/form-data">
    @csrf

    <div class="form-group">
        <label for="name">Nama Produk</label>
        <input type="text" name="name" id="name" value="{{ old('name') }}">
        @error('name') <span style="color: red;">{{ $message }}</span> @enderror
    </div>

    <div class="form-group">
        <label for="price">Harga (Rp)</label>
        <input type="number" name="price" id="price" min="0" value="{{ old('price') }}">
        @error('price') <span style="color: red;">{{ $message }}</span> @enderror
    </div>

    <div class="form-group">
        <label for="stock">Stok</label>
        <input type="number" name="stock" id="stock" min="0" value="{{ old('stock') }}">
        @error('stock') <span style="color: red;">{{ $message }}</span> @enderror
    </div>

    {{-- FIELD BARU: dropdown kategori --}}
    <div class="form-group">
        <label for="category_id">Kategori</label>
        <select name="category_id" id="category_id">
            <option value="">-- Pilih Kategori --</option>
            @foreach ($categories as $category)
                <option value="{{ $category->id }}" @selected(old('category_id') == $category->id)>
                    {{ $category->name }}
                </option>
            @endforeach
        </select>
        @error('category_id') <span style="color: red;">{{ $message }}</span> @enderror
    </div>

    <div class="form-group">
        <label for="description">Deskripsi</label>
        <textarea name="description" id="description">{{ old('description') }}</textarea>
        @error('description') <span style="color: red;">{{ $message }}</span> @enderror
    </div>

    {{-- Field dari Materi 3 (upload gambar) --}}
    <div class="form-group">
        <label for="image">Gambar Produk</label>
        <input type="file" name="image" id="image">
        @error('image') <span style="color: red;">{{ $message }}</span> @enderror
    </div>

    <button type="submit">Simpan</button>
</form>
```

### Penjelasan Bagian Penting Dropdown

#### Opsi Placeholder (Pertama)

```blade
<option value="">-- Pilih Kategori --</option>
```

- `value=""` kosong, jadi kalau user tidak pilih apa-apa, validasi `required` akan tolak dengan pesan "kategori wajib dipilih".
- Memberi tahu user bahwa harus **memilih** sesuatu.

#### Looping Kategori

```blade
@foreach ($categories as $category)
    <option value="{{ $category->id }}" @selected(old('category_id') == $category->id)>
        {{ $category->name }}
    </option>
@endforeach
```

Bagian per bagian:

| Bagian                                              | Arti                                                              |
|-----------------------------------------------------|-------------------------------------------------------------------|
| `@foreach ($categories as $category)`               | Loop tiap kategori dari variabel yang dikirim controller          |
| `value="{{ $category->id }}"`                       | Yang dikirim ke server adalah id kategori                         |
| `@selected(old('category_id') == $category->id)`    | Tandai sebagai "dipilih" kalau cocok dengan input sebelumnya      |
| `{{ $category->name }}`                             | Teks yang tampil ke user adalah nama kategori                     |

#### `@selected(...)` Itu Apa?

`@selected(...)` adalah Blade directive yang menghasilkan `selected="selected"` kalau kondisi true.

Fungsinya: kalau **validasi gagal** dan form dikembalikan, dropdown harus tetap menampilkan pilihan user sebelumnya, bukan reset ke awal.

Contoh: user pilih "Pakaian" lalu submit, tapi nama produk kosong. Validasi gagal, form kembali. Dropdown harus tetap di "Pakaian", bukan di placeholder.

## Langkah 4: Tambah Dropdown di Form Edit

Buka file `resources/views/products/edit.blade.php`. Hampir sama, tapi nilai defaultnya diambil dari `$product->category_id`:

```blade
<form action="/products/{{ $product->id }}" method="POST" enctype="multipart/form-data">
    @csrf
    @method('PUT')

    <div class="form-group">
        <label for="name">Nama Produk</label>
        <input type="text" name="name" id="name" value="{{ old('name', $product->name) }}">
        @error('name') <span style="color: red;">{{ $message }}</span> @enderror
    </div>

    <div class="form-group">
        <label for="price">Harga (Rp)</label>
        <input type="number" name="price" id="price" min="0" value="{{ old('price', $product->price) }}">
        @error('price') <span style="color: red;">{{ $message }}</span> @enderror
    </div>

    <div class="form-group">
        <label for="stock">Stok</label>
        <input type="number" name="stock" id="stock" min="0" value="{{ old('stock', $product->stock) }}">
        @error('stock') <span style="color: red;">{{ $message }}</span> @enderror
    </div>

    <div class="form-group">
        <label for="category_id">Kategori</label>
        <select name="category_id" id="category_id">
            <option value="">-- Pilih Kategori --</option>
            @foreach ($categories as $category)
                <option value="{{ $category->id }}"
                    @selected(old('category_id', $product->category_id) == $category->id)>
                    {{ $category->name }}
                </option>
            @endforeach
        </select>
        @error('category_id') <span style="color: red;">{{ $message }}</span> @enderror
    </div>

    <div class="form-group">
        <label for="description">Deskripsi</label>
        <textarea name="description" id="description">{{ old('description', $product->description) }}</textarea>
        @error('description') <span style="color: red;">{{ $message }}</span> @enderror
    </div>

    <div class="form-group">
        <label for="image">Gambar Produk</label>
        <input type="file" name="image" id="image">
        @error('image') <span style="color: red;">{{ $message }}</span> @enderror
    </div>

    <button type="submit">Update</button>
</form>
```

### Perbedaan dengan Form Create

| Bagian                       | Create                           | Edit                                                |
|------------------------------|----------------------------------|-----------------------------------------------------|
| `action`                     | `action="/products"`             | `action="/products/{{ $product->id }}"`             |
| `@method`                    | Tidak perlu (POST default)       | `@method('PUT')`                                    |
| `value` input                | `old('name')`                    | `old('name', $product->name)`                       |
| Default dropdown             | `old('category_id')`             | `old('category_id', $product->category_id)`         |

Pola `old('field', $model->field)` artinya: "Pakai input user sebelumnya kalau ada, kalau tidak ada pakai data dari database."

## Coba di Browser

### Tambah Produk

Akses:

```
http://localhost:8000/products/create
```

- Form muncul dengan dropdown kategori berisi 4 kategori contoh.
- Isi nama, harga, pilih kategori, submit.
- Cek database: produk baru tersimpan dengan `category_id` sesuai pilihan.

### Edit Produk

Akses:

```
http://localhost:8000/products/{id}/edit
```

- Dropdown sudah menunjuk ke kategori produk saat ini.
- Ubah kategori, submit, data berubah.

## Tips Penting

### 1. Selalu Pakai `exists` Validasi

Jangan percaya form HTML. User bisa inspect element dan ubah `<option value="1">` jadi `<option value="999">`. Validasi `exists:categories,id` memastikan hanya id kategori yang valid yang bisa masuk.

### 2. Kalau Form Tidak Tampil Dropdown

Kemungkinan penyebab:

- Lupa kirim `$categories` dari controller (cek `compact`).
- Nama variabel salah (`$category` vs `$categories`).
- Migration categories belum dijalankan, jadi tabel kosong.

Cek dengan `dd($categories);` di controller kalau ragu:

```php
public function create()
{
    $categories = Category::orderBy('name')->get();
    dd($categories);  // sementara, untuk debugging
}
```

`dd` = "dump and die". Tampilkan isi variabel dan hentikan eksekusi. Hapus setelah yakin.

### 3. Konvensi Penamaan `category_id`

Nama field `category_id` di form harus persis sama dengan kolom di database dan `$fillable` di model. Laravel otomatis cocokkan tiga hal ini:

- Field form: `name="category_id"`
- Kolom tabel: `category_id`
- Fillable model: `'category_id'`

Kalau nama beda, mass assignment tidak akan jalan.

## Inti Pelajaran Tahap 9

> Dropdown kategori = elemen `<select>` yang isinya di-looping dari `$categories` yang dikirim controller. Validasi `exists:categories,id` memastikan hanya kategori valid yang bisa masuk.

Yang sudah kita lakukan:

1. Kirim `$categories` dari `create()` dan `edit()` di ProductController.
2. Tambah validasi `category_id` di `store()` dan `update()`.
3. Tambah elemen `<select>` + looping `<option>` di view create dan edit.
4. Pakai `@selected(old(...) == ...)` untuk tahan pilihan user saat validasi gagal.
5. Pakai pola `old('field', $model->field)` di form edit.

Sekarang form produk sudah bisa pilih kategori. Tapi di daftar produk, nama kategori belum tampil, masih angka id. Itu yang akan kita selesaikan di tahap berikutnya.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke langkah berikutnya: **menampilkan nama kategori di list produk**?

Di tahap 10 kita akan:

- Mengubah method `index()` di ProductController untuk eager load kategori.
- Mengakses `$product->category->name` di view index.
- Supaya daftar produk tidak tampil "category_id: 1" tapi "Kategori: Elektronik".

Ketik **"lanjut"** kalau siap.
