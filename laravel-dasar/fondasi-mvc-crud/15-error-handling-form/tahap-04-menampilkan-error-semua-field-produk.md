# Tahap 4 — Menampilkan Error untuk Semua Field Form Product

> Fokus: menerapkan pola `@error(...)` dari field nama ke harga, stok, deskripsi, kategori, dan gambar product.

Pada tahap 3, kita sudah berhasil menampilkan error untuk satu field:

```blade
@error('name')
    <p role="alert">{{ $message }}</p>
@enderror
```

Sekarang polanya sama, hanya nama field di dalam `@error(...)` yang berubah. Kita akan menaruh setiap pesan tepat setelah field yang diperiksa.

## Peta nama field dan error

Ingat aturan penting dari tahap 2: nama pada `@error(...)` harus sama dengan atribut `name` pada input HTML dan key validasinya.

| Field yang dilihat user | `name` pada form | `@error(...)` | Contoh pesan |
| --- | --- | --- | --- |
| Nama product | `name` | `@error('name')` | Nama produk wajib diisi |
| Harga | `price` | `@error('price')` | Harga tidak boleh kurang dari 0 |
| Stok | `stock` | `@error('stock')` | Stok wajib diisi |
| Deskripsi | `description` | `@error('description')` | Deskripsi minimal 10 karakter |
| Kategori | `category_id` | `@error('category_id')` | Kategori wajib dipilih |
| Gambar product | `image` | `@error('image')` | File harus berupa gambar / Ukuran gambar terlalu besar |

`category_id` mungkin terlihat berbeda dari tulisan **Kategori** di halaman. Ini wajar. Yang dikirim form adalah ID kategori, misalnya ID kategori *Minuman*. Karena itu nama fieldnya adalah `category_id`.

## Aturan validasi harus cocok dengan pesan yang diharapkan

Pesan error baru muncul jika ada aturan validasi yang gagal. Dalam CRUD Product sebelumnya, aturan create biasanya sudah mencakup pola berikut:

```php
$validated = $request->validate([
    'name' => ['required', 'string', 'min:3'],
    'price' => ['required', 'numeric', 'min:0'],
    'stock' => ['required', 'integer', 'min:0'],
    'description' => ['nullable', 'string', 'min:10'],
    'category_id' => ['required', 'exists:categories,id'],
    'image' => ['required', 'image', 'mimes:jpg,jpeg,png,webp', 'max:2048'],
]);
```

Mari baca bagian yang penting untuk error handling:

| Rule | Yang diperiksa | Contoh kesalahan |
| --- | --- | --- |
| `required` | Field harus diisi | Nama, stok, atau kategori kosong |
| `numeric` | Harga harus angka | Harga diisi `sepuluh ribu` |
| `min:0` | Nilai angka tidak boleh negatif | Harga `-5000` |
| `nullable` | Field boleh kosong | Deskripsi boleh tidak diisi |
| `min:10` | Jika deskripsi diisi, minimal 10 karakter | Deskripsi terlalu pendek |
| `exists:categories,id` | Kategori harus ada di tabel `categories` | ID kategori tidak valid |
| `image` | File harus gambar | User memilih PDF atau file teks |
| `max:2048` | Ukuran file maksimal 2.048 KB, sekitar 2 MB | Gambar terlalu besar |

Untuk form edit, gambar lama biasanya tidak perlu diunggah ulang. Karena itu rule `image` pada update menggunakan `nullable`:

```php
'image' => ['nullable', 'image', 'mimes:jpg,jpeg,png,webp', 'max:2048'],
```

Kita tidak perlu mengubah aturan yang sudah ada hanya untuk menampilkan error. Yang kita tambahkan di tahap ini adalah Blade di bawah setiap field.

## Tambahkan error pada form create

Buka:

```text
resources/views/products/create.blade.php
```

Gunakan pola berikut setelah masing-masing field. Contoh ini hanya memperlihatkan bagian field form, bukan seluruh halaman create.

```blade
<label for="name">Nama product</label>
<input id="name" type="text" name="name" value="{{ old('name') }}">
@error('name')
    <p role="alert">{{ $message }}</p>
@enderror

<label for="price">Harga</label>
<input id="price" type="number" name="price" value="{{ old('price') }}" min="0">
@error('price')
    <p role="alert">{{ $message }}</p>
@enderror

<label for="stock">Stok</label>
<input id="stock" type="number" name="stock" value="{{ old('stock') }}" min="0">
@error('stock')
    <p role="alert">{{ $message }}</p>
@enderror

<label for="description">Deskripsi</label>
<textarea id="description" name="description">{{ old('description') }}</textarea>
@error('description')
    <p role="alert">{{ $message }}</p>
@enderror

<label for="category_id">Kategori</label>
<select id="category_id" name="category_id">
    <option value="">Pilih kategori</option>
    @foreach ($categories as $category)
        <option value="{{ $category->id }}" @selected(old('category_id') == $category->id)>
            {{ $category->name }}
        </option>
    @endforeach
</select>
@error('category_id')
    <p role="alert">{{ $message }}</p>
@enderror

<label for="image">Gambar product</label>
<input id="image" type="file" name="image" accept="image/jpeg,image/png,image/webp">
@error('image')
    <p role="alert">{{ $message }}</p>
@enderror
```

## Pahami bagian yang sama

Setiap field mengikuti urutan yang mudah diingat:

```blade
<input atau select atau textarea ...>

@error('nama_field')
    <p role="alert">{{ $message }}</p>
@enderror
```

Artinya:

1. User melihat atau mengisi field.
2. Jika field itu gagal validasi, Laravel menyimpan pesannya dalam `$errors`.
3. `@error('nama_field')` menemukan pesan tersebut.
4. Blade menampilkan `$message` persis di bawah field yang perlu diperbaiki.

Tidak perlu menulis `@if ($errors->any())` berulang kali untuk setiap field. `@error` sudah memeriksa satu field yang tepat.

## Kenapa setiap `old()` berbeda?

Selain gambar, data lama dapat dikembalikan oleh Laravel setelah validasi gagal:

| Field | Cara mengembalikan input lama |
| --- | --- |
| Nama | `old('name')` |
| Harga | `old('price')` |
| Stok | `old('stock')` |
| Deskripsi | `old('description')` di dalam `<textarea>` |
| Kategori | `@selected(old('category_id') == $category->id)` |

Untuk alasan keamanan browser, file pada `<input type="file">` **tidak dapat** diisi kembali dengan `old('image')`. Jika validasi gagal karena field lain, user mungkin perlu memilih gambar lagi. Karena itu pesan error gambar harus jelas jika file-nya sendiri tidak valid.

Atribut HTML seperti `min="0"` dan `accept="image/..."` membantu browser memberi petunjuk awal, tetapi bukan pengganti validasi Laravel. User tetap bisa mengirim request yang melewati pemeriksaan browser. Aturan di controller atau Form Request tetap menjadi penjaga utama.

## Terapkan pada form edit

Di:

```text
resources/views/products/edit.blade.php
```

Letakkan blok `@error(...)` yang sama di bawah field yang sesuai. Bedanya, field edit memakai nilai product lama sebagai nilai cadangan.

```blade
<input
    id="price"
    type="number"
    name="price"
    value="{{ old('price', $product->price) }}"
    min="0"
>

@error('price')
    <p role="alert">{{ $message }}</p>
@enderror
```

Pola nilai edit untuk semua field:

| Field | Contoh nilai pada form edit |
| --- | --- |
| Nama | `old('name', $product->name)` |
| Harga | `old('price', $product->price)` |
| Stok | `old('stock', $product->stock)` |
| Deskripsi | `old('description', $product->description)` |
| Kategori | Bandingkan `old('category_id', $product->category_id)` dengan ID kategori |
| Gambar | Tidak memakai `old()` karena file tidak dapat diisi ulang oleh browser |

Contoh pilihan kategori pada edit:

```blade
<option
    value="{{ $category->id }}"
    @selected(old('category_id', $product->category_id) == $category->id)
>
    {{ $category->name }}
</option>
```

## Uji satu per satu

Jangan mencoba semua kesalahan sekaligus dahulu. Uji seperti daftar berikut agar jelas field mana yang sedang diperiksa:

1. Kosongkan nama product, lalu pastikan error `name` muncul di bawah kolom nama.
2. Isi harga `-5000`, lalu pastikan error `price` muncul di bawah harga.
3. Kosongkan stok, lalu pastikan error `stock` muncul di bawah stok.
4. Pilih **Pilih kategori**, lalu pastikan error `category_id` muncul di bawah dropdown.
5. Pilih file PDF atau teks untuk gambar, lalu pastikan error `image` muncul di bawah upload gambar.
6. Pilih gambar yang lebih besar dari 2 MB, lalu pastikan error ukuran gambar muncul di bawah upload gambar.
7. Setelah memperbaiki semua field, simpan product dan pastikan flash message success dari materi 14 tetap muncul di halaman daftar product.

Saat ini kode `<p role="alert">{{ $message }}</p>` sudah berulang di banyak tempat. Itu normal sebagai langkah belajar. Pada tahap berikutnya, kita akan memindahkan bagian yang berulang itu ke satu Blade component agar tampilan error lebih rapi dan mudah digunakan ulang.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: membuat component error message yang dapat dipakai ulang?**
