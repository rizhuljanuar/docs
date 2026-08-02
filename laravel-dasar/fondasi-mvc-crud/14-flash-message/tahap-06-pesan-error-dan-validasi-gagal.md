# Tahap 6 — Pesan Error dan Validasi Gagal

> Fokus: membedakan pesan sukses setelah mutasi berhasil dengan error validasi saat data form belum benar.

## Tidak semua pengiriman form berhasil

User dapat menekan **Save product** dengan data yang belum sesuai. Contohnya:

- `name` kosong;
- `price` bukan angka yang valid;
- `stock` kurang dari nilai yang diperbolehkan;
- `category_id` tidak sesuai data Category.

Pada keadaan ini, product **tidak boleh** disimpan. Karena itu user tidak boleh melihat pesan:

```text
Data berhasil disimpan
```

Pesan sukses hanya boleh dibuat setelah `Product::create(...)` atau `$product->update(...)` benar-benar berhasil.

## Apa yang dilakukan Laravel saat validasi gagal?

Pada method `store()` atau `update()`, controller sudah memakai validasi seperti berikut:

```php
$validated = $request->validate([
    'name' => ['required', 'string'],
    'price' => ['required', 'numeric'],
    'stock' => ['required', 'integer'],
]);
```

Jika salah satu aturan tidak terpenuhi, Laravel menghentikan method pada baris itu. Laravel kemudian:

1. mengembalikan user ke form sebelumnya;
2. membawa pesan error validasi;
3. membawa input lama agar user tidak perlu mengisi ulang semuanya;
4. tidak menjalankan `Product::create(...)`, `$product->update(...)`, atau flash success.

Jadi kita tidak perlu menulis `return redirect(...)` tambahan untuk setiap kegagalan validasi. Laravel sudah melakukannya.

## Menampilkan error di dekat field

Pada form create atau edit, tampilkan error `name` di dekat input `name`:

```blade
<label>
    Name
    <input name="name" value="{{ old('name') }}">
</label>

@error('name')
    <p role="alert">{{ $message }}</p>
@enderror
```

Baca kode ini perlahan:

| Kode | Arti sederhana |
| --- | --- |
| `old('name', ...)` | Gunakan input terakhir user jika validasi gagal. Jika tidak ada, gunakan nilai cadangan. |
| `@error('name')` | Tampilkan isi bagian ini hanya jika field `name` memiliki error validasi. |
| `$message` | Pesan dari Laravel, misalnya bahwa field Name wajib diisi. |
| `role="alert"` | Memberi tahu teknologi bantu bahwa ini adalah pesan penting. |

Untuk form create, nilai cadangan bisa kosong:

```blade
<input name="name" value="{{ old('name') }}">
```

Untuk edit, nilai cadangan tetap memakai data product saat ini:

```blade
<input name="name" value="{{ old('name', $product->name) }}">
```

Gunakan pola yang sama untuk `price`, `stock`, `description`, `category_id`, image, dan `is_active` sesuai form yang sudah dibuat. Jangan mengganti struktur form, multipart upload, atau `$categories` yang telah ada.

## Error validasi berbeda dari flash success

| Keadaan | Product berubah? | Halaman tujuan | Pesan |
| --- | --- | --- | --- |
| Validasi gagal | Tidak | Kembali ke form | Error per field dari `@error` |
| Simpan berhasil | Ya | `/products` | `success`: Data berhasil disimpan |
| Update berhasil | Ya | `/products` | `success`: Data berhasil diperbarui |
| Soft delete berhasil | Ya | `/products` | `success`: Data berhasil dihapus |
| Restore berhasil | Ya | `/products/trash` | `success`: Product berhasil dikembalikan |

Pesan **Terjadi kesalahan, silakan coba lagi** cocok untuk kegagalan aplikasi yang memang sudah ditangani secara khusus. Namun jangan memakai pesan umum itu untuk menutupi error validasi. Saat field `name` kosong, user lebih terbantu oleh pesan yang menjelaskan field mana yang perlu diperbaiki.

## Jika ada pesan error umum

Layout juga dapat menampilkan flash key `error`, terpisah dari `success`. Tambahkan tepat setelah blok success dari tahap 3:

```blade
@if (session('error'))
    <div role="alert">
        {{ session('error') }}
    </div>
@endif
```

Blok ini hanya membaca pesan `error` jika controller lain yang sudah menangani kegagalan memang mengirimnya. Saat ini jangan menambahkan `try/catch` besar hanya untuk membuat pesan umum. Validasi Laravel sudah memberi alur yang lebih tepat untuk input form yang salah.

Bila suatu proses yang sudah ditangani gagal dan perlu mengarahkan user, pola pesannya adalah:

```php
return redirect('/products/create')->with('error', 'Terjadi kesalahan, silakan coba lagi');
```

Gunakan pola itu hanya ketika product belum tersimpan dan kegagalan memang telah ditangani. Jangan menaruhnya setelah `Product::create(...)` yang berhasil, dan jangan menampilkan pesan sukses bersamaan dengan error.

## Coba sendiri

1. Buka `/products/create`.
2. Kosongkan field `name`, lalu tekan **Save product**.
3. User harus kembali ke form, input yang valid tetap terisi, dan error `name` tampil.
4. Pastikan tidak ada pesan **Data berhasil disimpan**.
5. Isi form dengan data valid, lalu simpan. Pesan success dari tahap 2 dan 3 harus kembali muncul di `/products`.

## Yang tidak berubah

- Validation gagal tidak mengubah Product dan tidak menghapus cache dashboard.
- Form tetap `POST /products`, memakai fields, image upload, `$categories`, dan button component yang sudah ada.
- Daftar tetap memakai `$products` paginator, search, category filter, sorting, dan pagination.
- Detail tetap berbasis slug; mutasi tetap memakai ID; SoftDeletes dan `is_active` tetap berbeda.

---

**Apakah kamu ingin lanjut ke langkah terakhir: rangkuman dan best practice flash message?**
