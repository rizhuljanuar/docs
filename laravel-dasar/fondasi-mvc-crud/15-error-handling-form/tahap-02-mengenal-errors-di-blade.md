# Tahap 2 — Mengenal `$errors` di Blade

> Fokus: memahami dari mana pesan validasi datang dan bagaimana Blade dapat membacanya.

Pada tahap 1, kita sudah belajar bahwa error handling form membantu user mengetahui isian product mana yang harus diperbaiki.

Sekarang kita belum akan menampilkan pesan di setiap field. Kita hanya akan mengenal **`$errors`**, yaitu tempat Blade menerima kumpulan pesan validasi dari Laravel.

## Bayangkan ada map catatan dari pemeriksa formulir

Kembali ke contoh loket.

User menyerahkan formulir product. Petugas memeriksa formulir itu, lalu menaruh catatan kesalahan ke dalam sebuah map:

```text
nama product: Nama produk wajib diisi
harga: Harga tidak boleh kurang dari 0
stok: Stok wajib diisi
```

Saat formulir dikembalikan kepada user, map catatan itu ikut dibawa. Blade adalah halaman yang membuka map tersebut lalu menampilkan catatan yang tepat.

Di Laravel, map itu bernama **`$errors`**.

## Dari mana `$errors` berasal?

Pada controller CRUD Product, validasi biasanya dimulai dengan kode seperti ini:

```php
$validated = $request->validate([
    'name' => ['required', 'string'],
    'price' => ['required', 'numeric', 'min:0'],
    'stock' => ['required', 'integer'],
    'category_id' => ['required'],
    'image' => ['nullable', 'image'],
]);
```

Kita tidak perlu mengubah kode controller ini pada tahap ini. Mari pahami fungsi setiap bagiannya:

| Bagian | Arti sederhana |
| --- | --- |
| `$request` | Data yang dikirim user dari form. |
| `validate(...)` | Perintah untuk memeriksa data berdasarkan aturan. |
| `'name'` | Nama field form, yaitu `<input name="name">`. |
| `'required'` | Field harus diisi. |
| `'min:0'` | Nilai harga tidak boleh kurang dari 0. |
| `'image'` | Jika user mengirim file gambar, file itu harus benar-benar berupa gambar. |

Jika semua aturan lolos, Laravel melanjutkan ke `Product::create(...)` atau `$product->update(...)`.

Jika ada aturan yang gagal, Laravel otomatis melakukan tiga hal:

1. Menghentikan proses simpan atau update, sehingga data yang salah tidak masuk database.
2. Mengarahkan user kembali ke form sebelumnya.
3. Membawa pesan error dan input lama untuk request berikutnya.

Pesan error yang dibawa itu kemudian tersedia di Blade sebagai **`$errors`**.

> Laravel 13+ sudah membagikan variabel `$errors` ke seluruh Blade view yang memakai kelompok middleware `web`. Jadi kita tidak perlu membuat variabel `$errors` sendiri di controller.

## Bentuk isi `$errors`

Anggap user mengirim form dengan nama kosong, harga `-5000`, dan stok kosong. Secara sederhana, `$errors` menyimpan informasi seperti ini:

```text
name        → Nama produk wajib diisi
price       → Harga tidak boleh kurang dari 0
stock       → Stok wajib diisi
```

Nama di sebelah kiri harus cocok dengan nilai atribut `name` pada field HTML:

```blade
<input name="name">
<input name="price">
<input name="stock">
<select name="category_id"></select>
<input name="image" type="file">
```

Contohnya:

- Error `name` dipakai untuk field nama product.
- Error `price` dipakai untuk field harga.
- Error `category_id` dipakai untuk pilihan kategori.
- Error `image` dipakai untuk upload gambar product.

Kecocokan nama ini penting. Jika input memakai `name="price"`, tetapi Blade mencoba mencari error `harga`, pesan error tidak akan ditemukan.

## Memeriksa apakah ada error sama sekali

Sebelum nanti menampilkan daftar pesan, Blade dapat bertanya apakah `$errors` memiliki setidaknya satu error:

```blade
@if ($errors->any())
    <p role="alert">Ada data product yang perlu diperbaiki.</p>
@endif
```

Penjelasan pelan-pelan:

| Kode | Fungsi |
| --- | --- |
| `@if (...)` | Mulai kondisi di Blade. |
| `$errors` | Kumpulan pesan validasi dari Laravel. |
| `->any()` | Menghasilkan `true` jika minimal ada satu error. |
| `<p role="alert">` | Pesan umum untuk memberi tahu user bahwa ada masalah. |
| `@endif` | Menutup kondisi `@if`. |

Blok ini tidak menjelaskan field mana yang salah. Ia hanya memberi tanda bahwa ada isian yang perlu diperiksa. Pada tahap berikutnya, kita akan menampilkan pesan khusus untuk satu field, yaitu `name`.

## `$errors` bukan flash message success

Karena materi ini melanjutkan materi 14, mari bedakan keduanya:

| Keadaan | Yang dipakai | Contoh pesan |
| --- | --- | --- |
| Product berhasil disimpan | `session('success')` | Data berhasil disimpan |
| Form product gagal validasi | `$errors` | Nama produk wajib diisi |

`session('success')` muncul setelah tindakan berhasil dan redirect ke daftar product. Sedangkan `$errors` muncul saat validasi gagal dan Laravel mengembalikan user ke form.

Jadi, jangan menampilkan **"Data berhasil disimpan"** ketika `name` kosong atau `price` negatif. Pada kondisi itu, user membutuhkan petunjuk perbaikan dari `$errors`.

## Coba bayangkan alurnya

1. User membuka `/products/create`.
2. User mengosongkan nama product lalu menekan **Save product**.
3. Controller menjalankan `$request->validate(...)`.
4. Aturan `required` untuk `name` gagal.
5. Laravel kembali ke `/products/create`.
6. Blade menerima `$errors` yang berisi error untuk `name`.
7. Blade dapat memakai `$errors->any()` untuk menunjukkan bahwa form perlu diperbaiki.

Belum perlu menulis component atau menyalin kode error ke semua field. Kita akan mulai dari satu field agar alurnya mudah dipahami.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: menampilkan error `name` dengan `@error` di Blade?**
