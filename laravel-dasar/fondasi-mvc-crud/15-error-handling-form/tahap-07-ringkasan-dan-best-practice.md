# Tahap 7 — Ringkasan dan Best Practice Error Handling Form

> Penutup materi 15: membantu user memperbaiki form Product dengan pesan yang jelas, dekat dengan field yang salah, dan konsisten.

## Apa yang sudah dipelajari?

Kita mulai dari masalah sederhana: user mengirim form product, tetapi ada data yang belum benar. Laravel menolak data yang salah agar database tetap rapi. Namun user juga perlu tahu **bagian mana** yang harus diperbaiki.

Berikut alur lengkap yang sudah kita buat:

1. Controller atau Form Request memvalidasi data product.
2. Jika validasi gagal, Laravel tidak menyimpan atau mengubah Product.
3. Laravel kembali ke halaman form dan membawa input lama serta pesan error.
4. Blade menerima kumpulan error melalui `$errors`.
5. Component `<x-error-message />` mengambil pesan untuk satu field.
6. User melihat pesan tepat di bawah field yang perlu diperbaiki.
7. User memperbaiki form, lalu mengirimnya kembali.
8. Jika semua valid, Product disimpan atau diperbarui dan flash message success dari materi 14 ditampilkan.

## Tiga bagian utama yang bekerja bersama

### 1. Validasi di controller atau Form Request

Validasi menentukan data mana yang boleh masuk ke database. Contoh aturan Product:

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

Aturan ini menjaga agar user tidak menyimpan nama kosong, harga negatif, stok kosong, kategori yang tidak tersedia, atau file yang bukan gambar.

### 2. Component pesan error

File:

```text
resources/views/components/error-message.blade.php
```

```blade
@props(['field'])

@error($field)
    <p role="alert">{{ $message }}</p>
@enderror
```

Component menerima nama field, lalu memeriksa `$errors` untuk field tersebut. Jika ada error, component menampilkan pesan. Jika tidak ada, component tidak menampilkan apa pun.

### 3. Form create dan edit

Di bawah tiap field, panggil component dengan nama field yang sama:

```blade
<input name="price" value="{{ old('price') }}">
<x-error-message field="price" />
```

Pada edit, data product lama dipakai sebagai nilai cadangan:

```blade
<input name="price" value="{{ old('price', $product->price) }}">
<x-error-message field="price" />
```

## Peta field Product

Gunakan pasangan ini agar nama input, rule validasi, dan component tidak tertukar:

| Yang dilihat user | Nama pada form dan validasi | Component | Contoh error yang jelas |
| --- | --- | --- | --- |
| Nama product | `name` | `<x-error-message field="name" />` | Nama produk wajib diisi |
| Harga | `price` | `<x-error-message field="price" />` | Harga tidak boleh kurang dari 0 |
| Stok | `stock` | `<x-error-message field="stock" />` | Stok wajib diisi |
| Deskripsi | `description` | `<x-error-message field="description" />` | Deskripsi terlalu pendek |
| Kategori | `category_id` | `<x-error-message field="category_id" />` | Kategori wajib dipilih |
| Gambar product | `image` | `<x-error-message field="image" />` | File harus berupa gambar / Ukuran gambar terlalu besar |

Jika input memakai `name="category_id"`, component juga harus memakai `field="category_id"`, bukan `field="kategori"`.

## Error validasi dan flash message memiliki tugas berbeda

Materi ini melanjutkan materi 14. Keduanya sama-sama membantu user, tetapi dipakai pada waktu yang berbeda.

| Kondisi | Product berubah? | Pesan yang dipakai | Tempat pesan muncul |
| --- | --- | --- | --- |
| Nama kosong atau harga negatif | Tidak | Validation error melalui `<x-error-message />` | Tepat di bawah field yang salah |
| Product berhasil dibuat | Ya | Flash `success` | Halaman daftar `/products` |
| Product berhasil diperbarui | Ya | Flash `success` | Halaman daftar `/products` |
| Kegagalan umum yang sudah ditangani | Tidak atau bergantung kasus | Flash `error` | Layout aplikasi |

Jangan memakai flash success untuk validasi gagal. Jika user mengosongkan nama, pesan **"Data berhasil disimpan"** tentu membingungkan karena Product memang belum disimpan.

## Best practice untuk UX validation

Berikut kebiasaan kecil yang membuat form lebih mudah dipakai:

1. **Pesan harus spesifik.**
   Gunakan *Harga tidak boleh kurang dari 0*, bukan hanya *Input salah*.

2. **Letakkan pesan dekat field yang salah.**
   User tidak perlu mencari-cari alasan form gagal dikirim.

3. **Gunakan bahasa yang mudah dimengerti.**
   Hindari istilah teknis seperti *validation exception* untuk user biasa.

4. **Jaga input lama dengan `old()`.**
   User tidak perlu mengetik ulang nama, harga, stok, dan deskripsi yang sudah benar.

5. **Samakan nama field.**
   Atribut `name`, key validasi, dan prop component harus sama, misalnya semuanya memakai `price`.

6. **Gunakan component untuk tampilan yang berulang.**
   Jika desain pesan ingin diubah, cukup ubah satu file component.

7. **Validasi tetap wajib di server.**
   Atribut HTML seperti `min="0"` dan `accept="image/..."` hanya membantu browser. Laravel tetap harus memvalidasi request di controller atau Form Request.

8. **Gunakan `nullable` untuk field yang benar-benar boleh kosong.**
   Misalnya deskripsi yang opsional. Pada update, gambar biasanya `nullable` karena user tidak wajib mengunggah ulang gambar lama.

9. **Jangan mencoba menangani validasi biasa dengan `try/catch` besar.**
   `$request->validate(...)` atau Form Request sudah mengembalikan user ke form, membawa `$errors`, dan membawa input lama secara otomatis.

10. **Gunakan `role="alert"` pada pesan error.**
    Ini membantu teknologi bantu mengenali pesan penting.

## Checklist akhir implementasi

- [ ] Form create memakai `POST /products`, `@csrf`, dan `enctype="multipart/form-data"`.
- [ ] Form edit memakai `POST /products/{{ $product->id }}`, `@csrf`, `@method('PUT')`, dan `enctype="multipart/form-data"`.
- [ ] Controller atau Form Request memvalidasi `name`, `price`, `stock`, `description`, `category_id`, dan `image` sesuai kebutuhan aplikasi.
- [ ] `resources/views/components/error-message.blade.php` memiliki prop `field` dan memakai `@error($field)`.
- [ ] Field create memakai `old('field_name')`.
- [ ] Field edit memakai `old('field_name', $product->field_name)`.
- [ ] Setiap field memasang `<x-error-message field="nama_field" />` yang cocok dengan atribut `name`.
- [ ] Input file tidak memakai `old('image')` atau atribut `value` untuk mengisi ulang file.
- [ ] Validasi gagal tidak membuat atau mengubah Product.
- [ ] Validasi gagal tidak menghapus cache dashboard dan tidak menampilkan flash success.
- [ ] Simpan dan update yang berhasil tetap menghapus cache dashboard, lalu menampilkan flash message success dari materi 14.

## Uji manual terakhir

Uji dari sudut pandang user, bukan hanya dari sudut pandang kode:

1. Buka `/products/create` dan kirim form kosong. Pastikan pesan muncul dekat field wajib yang kosong.
2. Masukkan harga `-5000`. Pastikan pesan harga menjelaskan bahwa nilai tidak boleh kurang dari 0.
3. Kosongkan stok. Pastikan error stok tampil tepat di bawah field stok.
4. Jangan memilih kategori. Pastikan error kategori tampil tepat di bawah dropdown.
5. Pilih file PDF atau teks sebagai gambar. Pastikan pesan **File harus berupa gambar** muncul di bawah upload file.
6. Pilih gambar lebih besar dari 2 MB. Pastikan pesan ukuran gambar muncul di bawah upload file.
7. Isi seluruh form dengan data valid. Pastikan Product tersimpan satu kali dan flash success **Data berhasil disimpan** muncul sekali.
8. Buka edit Product, ubah harga menjadi negatif, lalu update. Pastikan database tidak berubah dan input terakhir tetap terlihat.
9. Perbaiki harga, update lagi, lalu pastikan flash success **Data berhasil diperbarui** muncul sekali.
10. Setelah flash success muncul, muat ulang halaman daftar. Pastikan pesan success hilang, karena flash message memang sementara.

## Penutup

Error handling form bukan sekadar menampilkan teks merah. Ini adalah cara aplikasi berbicara dengan jelas saat user perlu memperbaiki data.

Dengan `$errors`, `@error`, `old()`, dan component `<x-error-message />`, form CRUD Product menjadi lebih ramah: data tetap aman, user tahu masalahnya, dan user dapat memperbaikinya tanpa kebingungan.

Materi **15. Error Handling Form** selesai.
