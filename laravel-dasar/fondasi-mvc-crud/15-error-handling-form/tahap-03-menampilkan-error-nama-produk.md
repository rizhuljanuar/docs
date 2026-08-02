# Tahap 3 — Menampilkan Error Nama Product dengan `@error`

> Fokus: menampilkan satu pesan validasi tepat di bawah field nama product.

Pada tahap 2, kita sudah mengenal `$errors`, yaitu kumpulan catatan kesalahan yang Laravel bawa kembali ke Blade saat validasi form gagal.

Sekarang kita akan memakai cara Blade yang lebih praktis: **`@error`**. Kita fokus pada **satu field saja**, yaitu nama product.

## Tujuan kecil kita

Saat user menekan **Save product** tanpa mengisi nama product, user perlu melihat pesan ini tepat di dekat kolom nama:

```text
Nama produk wajib diisi
```

Kita belum mengubah error harga, stok, kategori, atau gambar. Semua itu akan menyusul pada tahap berikutnya.

## Pastikan validasi `name` sudah ada

Agar Laravel bisa membuat pesan error, controller atau Form Request harus memiliki aturan untuk field `name`.

Contoh aturan yang sudah sesuai dengan CRUD Product:

```php
'name' => ['required', 'string'],
```

Artinya:

| Bagian | Arti sederhana |
| --- | --- |
| `'name'` | Data dari input yang memakai `name="name"`. |
| `'required'` | Nama product tidak boleh kosong. |
| `'string'` | Nama product harus berupa teks. |

Saat `name` kosong, Laravel menambahkan pesan untuk key `name` ke dalam `$errors`. Pada aplikasi berbahasa Indonesia, pesan ini dapat diatur menjadi **"Nama produk wajib diisi"**. Kita akan membahas pengaturan pesan khusus secara terpisah, bukan pada tahap ini.

## Tambahkan `@error` di form create

Buka view form tambah product, biasanya:

```text
resources/views/products/create.blade.php
```

Cari field nama product yang sebelumnya kira-kira seperti ini:

```blade
<label for="name">Nama product</label>
<input
    id="name"
    type="text"
    name="name"
    value="{{ old('name') }}"
>
```

Lalu tambahkan blok `@error('name')` **tepat setelah input**:

```blade
<label for="name">Nama product</label>
<input
    id="name"
    type="text"
    name="name"
    value="{{ old('name') }}"
>

@error('name')
    <p role="alert">{{ $message }}</p>
@enderror
```

## Membaca kode baris demi baris

| Kode | Fungsi |
| --- | --- |
| `<label for="name">` | Teks penjelas untuk kolom nama product. |
| `id="name"` | Identitas input yang dihubungkan dengan `for="name"` pada label. |
| `name="name"` | Nama data yang dikirim ke Laravel. Ini harus cocok dengan rule validasi dan `@error('name')`. |
| `old('name')` | Menampilkan kembali nama yang tadi diketik user setelah validasi gagal. Jika belum ada input lama, nilainya kosong. |
| `@error('name')` | Menampilkan isi blok hanya jika `$errors` memiliki error untuk `name`. |
| `$message` | Pesan error pertama dari Laravel untuk field `name`. |
| `role="alert"` | Memberi tahu teknologi bantu bahwa teks ini adalah pesan penting. |
| `@enderror` | Menutup blok `@error`. |

`@error('name')` sebenarnya memeriksa `$errors` untuk kita. Jadi, kita tidak perlu menulis kondisi `$errors->has('name')` sendiri.

## Apa yang user lihat?

### Saat pertama membuka form

User belum pernah mengirim form. Tidak ada error `name`, sehingga blok `@error('name')` tidak ditampilkan.

```text
Nama product
[                         ]
```

### Saat nama product dikosongkan lalu form dikirim

Validasi `required` gagal. Laravel kembali ke form dan `@error('name')` menampilkan pesan:

```text
Nama product
[                         ]
Nama produk wajib diisi
```

Pesan dekat dengan input membuat user langsung tahu bahwa kolom nama perlu diisi.

### Saat nama product sudah diisi dengan benar

User mengisi, misalnya, `Kopi Arabika`, lalu mengirim ulang form. Error `name` tidak ada lagi. Laravel tidak menampilkan blok `@error`, kemudian validasi dapat melanjutkan ke field lain atau menyimpan product jika semua field valid.

## Kenapa `old('name')` tetap penting?

Misalnya user mengisi:

- Nama product: `Kopi Arabika`
- Harga: `-5000`
- Stok: `10`

Validasi gagal karena harga negatif. Walaupun error ada di `price`, user tidak perlu mengetik ulang nama product. `old('name')` membuat `Kopi Arabika` tetap tampil pada field nama saat form kembali dibuka.

Error handling yang baik adalah gabungan dari:

- `@error('name')`, untuk menunjukkan bagian yang salah.
- `old('name')`, untuk menjaga data yang sudah diketik user.

## Terapkan juga pada form edit

Di form edit product, field yang sama biasanya memiliki nilai awal dari database:

```blade
<input
    id="name"
    type="text"
    name="name"
    value="{{ old('name', $product->name) }}"
>

@error('name')
    <p role="alert">{{ $message }}</p>
@enderror
```

Perbedaannya hanya pada `old()`:

| Halaman | Nilai field nama |
| --- | --- |
| Create | `old('name')` |
| Edit | `old('name', $product->name)` |

Pada form edit, jika tidak ada error sebelumnya, Laravel menampilkan nama product yang sudah tersimpan, misalnya `Kopi Arabika`. Jika validasi gagal, input terakhir user lebih diprioritaskan daripada data lama di database.

## Coba sendiri

1. Tambahkan blok `@error('name')` ke form create dan edit product.
2. Buka `/products/create`.
3. Kosongkan nama product, lalu isi field wajib lain dengan data yang valid.
4. Tekan **Save product**.
5. Pastikan kembali ke form dan pesan error nama muncul tepat di bawah input.
6. Isi nama product, lalu kirim ulang form.
7. Pastikan pesan nama hilang. Jika semua field lain valid, product tersimpan dan flash message success dari materi 14 tampil di daftar product.

Jangan menambahkan flash message error untuk kasus ini. `@error('name')` sudah memberi petunjuk yang lebih tepat kepada user.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: menampilkan error untuk semua field form product?**
