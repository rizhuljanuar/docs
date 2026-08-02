# Tahap 6 — Memakai Component Error Message di Form Create dan Edit Product

> Fokus: mengganti semua blok `@error(...)` yang berulang dengan `<x-error-message />` pada form tambah dan edit product.

Pada tahap 5, kita sudah membuat component ini:

```text
resources/views/components/error-message.blade.php
```

Dengan isi:

```blade
@props(['field'])

@error($field)
    <p role="alert">{{ $message }}</p>
@enderror
```

Sekarang kita akan benar-benar memakai component itu pada setiap field Product. Tujuannya sederhana: seluruh pesan error punya bentuk yang konsisten, tetapi form CRUD yang sudah ada tetap bekerja seperti sebelumnya.

## Ingat: yang berubah hanya tampilan pesan error

Saat mengganti kode error menjadi component, **jangan** mengubah hal-hal berikut:

- Route create tetap mengirim `POST` ke `/products`.
- Route edit tetap mengirim `PUT` ke `/products/{{ $product->id }}`.
- Form upload tetap memakai `enctype="multipart/form-data"`.
- `@csrf` dan `@method('PUT')` tetap ada.
- `name` pada setiap field tidak berubah.
- `old()` tetap dipakai agar input user tidak hilang.
- `$categories` tetap dipakai untuk dropdown kategori.
- Validasi pada controller atau Form Request tidak berubah.
- Flash message success dari materi 14 tetap tampil setelah simpan atau update berhasil.

Component hanya mengganti blok panjang ini:

```blade
@error('price')
    <p role="alert">{{ $message }}</p>
@enderror
```

menjadi ini:

```blade
<x-error-message field="price" />
```

## Form create product

Buka file:

```text
resources/views/products/create.blade.php
```

Pastikan setiap component diletakkan **tepat setelah field yang sesuai**. Berikut contoh bagian form lengkap dengan enam field pembelajaran kita:

```blade
<form method="POST" action="/products" enctype="multipart/form-data">
    @csrf

    <label for="name">Nama product</label>
    <input id="name" type="text" name="name" value="{{ old('name') }}">
    <x-error-message field="name" />

    <label for="price">Harga</label>
    <input id="price" type="number" name="price" value="{{ old('price') }}" min="0">
    <x-error-message field="price" />

    <label for="stock">Stok</label>
    <input id="stock" type="number" name="stock" value="{{ old('stock') }}" min="0">
    <x-error-message field="stock" />

    <label for="description">Deskripsi</label>
    <textarea id="description" name="description">{{ old('description') }}</textarea>
    <x-error-message field="description" />

    <label for="category_id">Kategori</label>
    <select id="category_id" name="category_id">
        <option value="">Pilih kategori</option>

        @foreach ($categories as $category)
            <option value="{{ $category->id }}" @selected(old('category_id') == $category->id)>
                {{ $category->name }}
            </option>
        @endforeach
    </select>
    <x-error-message field="category_id" />

    <label for="image">Gambar product</label>
    <input id="image" type="file" name="image" accept="image/jpeg,image/png,image/webp">
    <x-error-message field="image" />

    <x-button type="submit" variant="primary">Save product</x-button>
</form>
```

## Perhatikan pasangan field dan component

| Field HTML | Component yang diletakkan setelahnya | Mengapa sama? |
| --- | --- | --- |
| `name="name"` | `<x-error-message field="name" />` | Memeriksa error nama product. |
| `name="price"` | `<x-error-message field="price" />` | Memeriksa error harga. |
| `name="stock"` | `<x-error-message field="stock" />` | Memeriksa error stok. |
| `name="description"` | `<x-error-message field="description" />` | Memeriksa error deskripsi. |
| `name="category_id"` | `<x-error-message field="category_id" />` | Memeriksa error pilihan kategori. |
| `name="image"` | `<x-error-message field="image" />` | Memeriksa error file gambar. |

Nama di kolom pertama dan nilai `field` harus persis sama. Misalnya, jangan menulis `<x-error-message field="kategori" />`, karena Laravel menyimpan error kategori dengan key `category_id`.

## Form edit product

Sekarang buka:

```text
resources/views/products/edit.blade.php
```

Penempatan component sama seperti form create. Perbedaannya, form edit memakai data `$product` sebagai nilai awal.

```blade
<form method="POST" action="/products/{{ $product->id }}" enctype="multipart/form-data">
    @csrf
    @method('PUT')

    <label for="name">Nama product</label>
    <input id="name" type="text" name="name" value="{{ old('name', $product->name) }}">
    <x-error-message field="name" />

    <label for="price">Harga</label>
    <input id="price" type="number" name="price" value="{{ old('price', $product->price) }}" min="0">
    <x-error-message field="price" />

    <label for="stock">Stok</label>
    <input id="stock" type="number" name="stock" value="{{ old('stock', $product->stock) }}" min="0">
    <x-error-message field="stock" />

    <label for="description">Deskripsi</label>
    <textarea id="description" name="description">{{ old('description', $product->description) }}</textarea>
    <x-error-message field="description" />

    <label for="category_id">Kategori</label>
    <select id="category_id" name="category_id">
        <option value="">Pilih kategori</option>

        @foreach ($categories as $category)
            <option
                value="{{ $category->id }}"
                @selected(old('category_id', $product->category_id) == $category->id)
            >
                {{ $category->name }}
            </option>
        @endforeach
    </select>
    <x-error-message field="category_id" />

    <label for="image">Gambar product baru</label>
    <input id="image" type="file" name="image" accept="image/jpeg,image/png,image/webp">
    <x-error-message field="image" />

    <x-button type="submit" variant="primary">Update product</x-button>
</form>
```

## Create dan edit, apa bedanya?

Component error-nya **sama persis**. Yang berbeda hanya cara mengambil nilai field.

| Bagian | Create | Edit |
| --- | --- | --- |
| Nama | `old('name')` | `old('name', $product->name)` |
| Harga | `old('price')` | `old('price', $product->price)` |
| Stok | `old('stock')` | `old('stock', $product->stock)` |
| Deskripsi | `old('description')` | `old('description', $product->description)` |
| Kategori | `old('category_id')` | `old('category_id', $product->category_id)` |
| Error price | `<x-error-message field="price" />` | `<x-error-message field="price" />` |

`old()` selalu didahulukan. Artinya ketika update gagal karena harga negatif, input terakhir user tetap terlihat, bukan langsung kembali ke harga lama dari database.

## Tentang gambar di form edit

Untuk `<input type="file">`, browser tidak mengizinkan nilai file lama ditampilkan kembali. Karena itu kita tidak memakai kode seperti ini:

```blade
{{-- Jangan lakukan ini. --}}
<input type="file" name="image" value="{{ old('image') }}">
```

Jika user memilih file yang bukan gambar atau ukurannya terlalu besar, component tetap menampilkan error di bawah input gambar:

```blade
<x-error-message field="image" />
```

Jika validasi gagal karena field lain, user mungkin perlu memilih ulang file gambar. Ini aturan keamanan browser, bukan masalah pada component Laravel.

## Uji alur akhir

Lakukan pengujian ini pada create dan edit agar yakin component benar-benar dapat dipakai ulang:

1. Buka `/products/create`, kosongkan nama, harga, stok, dan kategori, lalu kirim form.
2. Pastikan pesan setiap field muncul dekat field yang tepat.
3. Isi harga `-5000`, lalu pastikan pesan error harga muncul dari `<x-error-message field="price" />`.
4. Pilih file PDF atau gambar lebih besar dari 2 MB, lalu pastikan error gambar muncul dari component yang sama.
5. Perbaiki semua input dan simpan. Pastikan product dibuat dan flash success **Data berhasil disimpan** muncul sekali di `/products`.
6. Buka halaman edit salah satu product, ubah harga menjadi negatif, lalu kirim update.
7. Pastikan product tidak berubah, input terakhir tetap tampil, dan error harga muncul tepat di bawahnya.
8. Perbaiki harga, update kembali, lalu pastikan flash success **Data berhasil diperbarui** muncul sekali.

Sekarang create dan edit memakai satu cetakan pesan error yang sama. Pada tahap terakhir, kita akan merangkum alur lengkap serta best practice agar flash message dan validation error tidak tertukar.

---

**Apakah kamu ingin lanjut ke langkah terakhir: ringkasan dan best practice error handling form?**
