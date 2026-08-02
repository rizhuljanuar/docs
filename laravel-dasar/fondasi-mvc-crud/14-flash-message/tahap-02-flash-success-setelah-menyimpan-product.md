# Tahap 2 — Flash Success Setelah Menyimpan Product

> Fokus: mengirim pesan **Data berhasil disimpan** setelah product baru benar-benar tersimpan.

## Ingat alur sebelumnya

Pada form Add product, user mengisi data lalu menekan tombol **Save product**. Form mengirim `POST` ke `/products`, kemudian method `store()` pada `ProductController` memprosesnya.

Jika penyimpanan berhasil, controller sudah mengarahkan user kembali ke daftar:

```php
return redirect('/products');
```

Redirect ini seperti petugas yang mengantar user kembali ke rak daftar product. Namun petugas belum mengatakan hasilnya. Kita akan menambahkan pesan singkat saat redirect.

## Satu tambahan kecil: `with()`

Di akhir method `store()`, ubah redirect menjadi:

```php
return redirect('/products')->with('success', 'Data berhasil disimpan');
```

Baca dari kiri ke kanan:

| Bagian | Arti sederhana |
| --- | --- |
| `redirect('/products')` | Setelah proses selesai, pindahkan browser ke daftar product. |
| `->with(...)` | Bawa informasi sementara saat berpindah halaman. |
| `'success'` | Nama atau label pesan. Nanti Blade memakai nama ini untuk mengambil pesan sukses. |
| `'Data berhasil disimpan'` | Isi pesan yang akan dibaca user. |

Jadi Laravel tidak menyimpan pesan ini sebagai kolom database. Laravel hanya menitipkannya di session flash untuk halaman tujuan `/products`.

## Letak kode di controller

Jangan menyalin ulang seluruh method `store()`. Validasi, upload image, dan pembuatan `Product` yang telah dibuat pada materi sebelumnya tetap dipertahankan.

Temukan bagian paling akhir method `store()`, **setelah** product berhasil dibuat dan setelah invalidasi cache dashboard yang telah diwariskan dari materi dashboard:

```php
$product = Product::create($validated);

Cache::forget('dashboard.products.summary');

return redirect('/products')->with('success', 'Data berhasil disimpan');
```

Penjelasannya:

1. `Product::create($validated)` menyimpan product setelah data tervalidasi.
2. `Cache::forget('dashboard.products.summary')` tetap diperlukan karena dashboard harus membaca data product terbaru. Jangan memindahkannya sebelum proses simpan berhasil.
3. Baris `return` baru dijalankan setelah dua langkah itu selesai. Baris ini mengarahkan user ke `/products` sambil membuat flash message `success`.

> Jika method `store()` kamu sudah memiliki proses upload image atau cara membuat product yang sedikit berbeda, jangan menggantinya. Cukup ubah `return redirect('/products')` terakhir menjadi redirect yang memakai `->with('success', 'Data berhasil disimpan')`. Pastikan invalidasi cache tetap berada setelah penyimpanan berhasil dan sebelum redirect.

## Apa yang terjadi saat user menekan Save?

```text
Form Add product
        |
        | POST /products
        v
ProductController@store
        |
        | product berhasil disimpan
        v
Cache dashboard dihapus
        |
        v
redirect('/products')->with('success', 'Data berhasil disimpan')
        |
        v
Halaman /products menerima pesan sementara
```

Pada tahap ini pesan belum terlihat di layar karena view `/products` belum diberi kode untuk menampilkannya. Pesan sudah dibawa oleh session flash, tetapi tahap berikutnya yang akan membuat Blade membacanya.

## Coba sendiri

1. Tambahkan `->with('success', 'Data berhasil disimpan')` pada redirect akhir `store()`.
2. Buat product baru melalui `/products/create`.
3. Setelah menekan **Save product**, browser kembali ke `/products`.

Belum ada pesan visual, dan itu normal. Kita baru menaruh “catatan” di session flash. Tahap berikutnya akan menempelkan catatan itu ke halaman daftar.

## Yang tidak berubah

- Form tetap `POST /products` dan tetap memakai field Product yang ada.
- Daftar `/products` tetap memakai `$products` paginator, pencarian, category filter, sorting, dan pagination.
- Detail tetap memakai slug.
- Soft delete, `is_active`, trash `/products/trash`, serta cache dashboard tetap memiliki fungsi masing-masing.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: menampilkan flash message success di halaman daftar product?**
