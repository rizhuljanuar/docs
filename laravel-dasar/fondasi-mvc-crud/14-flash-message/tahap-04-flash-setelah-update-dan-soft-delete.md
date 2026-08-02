# Tahap 4 — Flash Message Setelah Update dan Soft Delete

> Fokus: memberi pesan setelah data product diperbarui atau dipindahkan ke trash.

## Polanya sama, pesannya berbeda

Pada tahap 2, kita menambahkan pesan setelah menyimpan product:

```php
return redirect('/products')->with('success', 'Data berhasil disimpan');
```

Untuk update dan delete, polanya tetap sama:

1. Jalankan perubahan data terlebih dahulu.
2. Hapus cache dashboard setelah perubahan berhasil.
3. Redirect ke halaman yang tepat.
4. Titipkan pesan `success` dengan `->with(...)`.

Yang berbeda hanya tindakan dan teks pesannya.

## Flash success setelah update

Method `update()` menerima perubahan dari form edit, lalu mengarahkan user kembali ke `/products`. Pada bagian akhir method itu, setelah `$product->update(...)` berhasil, gunakan pola berikut:

```php
$product->update($validated);

Cache::forget('dashboard.products.summary');

return redirect('/products')->with('success', 'Data berhasil diperbarui');
```

Baca alurnya:

- `$product->update($validated)` mengubah data yang sudah divalidasi.
- `Cache::forget(...)` membuat ringkasan dashboard tidak memakai data lama.
- `with('success', ...)` membawa informasi hasil update ke halaman daftar.

Saat user kembali ke `/products`, blok Blade dari tahap 3 membaca `session('success')` dan menampilkan:

```text
Data berhasil diperbarui
```

Jangan menyalin ulang seluruh method `update()`. Pertahankan validasi, upload atau penggantian image, dan field `Product` yang sudah ada. Ubah hanya redirect terakhir setelah seluruh proses berhasil.

## Flash success setelah soft delete

Pada aplikasi ini, tombol **Delete** mengirim `DELETE` ke `/products/{{ $product->id }}`. Method `destroy()` menjalankan soft delete, bukan penghapusan permanen.

Di bagian akhir `destroy()`, setelah `delete()` berhasil, gunakan:

```php
$product->delete();

Cache::forget('dashboard.products.summary');

return redirect('/products')->with('success', 'Data berhasil dihapus');
```

Pesan **Data berhasil dihapus** berarti product telah keluar dari daftar aktif dan dipindahkan ke trash. Baris database belum dihapus permanen, karena `SoftDeletes` mengisi `deleted_at`.

Product tersebut dapat dilihat dari:

```text
/products/trash
```

## Jangan tertukar: delete dan status aktif

Ada dua keadaan yang berbeda:

| Keadaan | Yang berubah | Arti |
| --- | --- | --- |
| Soft delete | `deleted_at` | Product dipindahkan ke trash. |
| Status publikasi | `is_active` | Product aktif atau tidak aktif, tetapi tetap berada di daftar aktif. |

Menekan tombol delete tidak seharusnya mengubah `is_active`. Sebaliknya, mengubah `is_active` tidak seharusnya memindahkan product ke trash.

## Urutan kode itu penting

Jangan membuat flash message sebelum perubahan berhasil. Misalnya, pola berikut salah secara urutan:

```php
return redirect('/products')->with('success', 'Data berhasil diperbarui');

$product->update($validated);
```

Baris setelah `return` tidak akan dijalankan. User juga dapat menerima pesan berhasil padahal data belum diubah.

Gunakan urutan ini:

```text
ubah atau hapus product berhasil
        |
        v
hapus cache dashboard
        |
        v
redirect sambil membawa flash success
```

## Coba sendiri

1. Ubah redirect terakhir pada `update()` menjadi pesan **Data berhasil diperbarui**.
2. Edit satu product dari `/products/{{ $product->id }}/edit`, lalu simpan perubahan.
3. Pastikan pesan update muncul setelah kembali ke `/products`.
4. Hapus satu product dari daftar aktif.
5. Pastikan pesan **Data berhasil dihapus** muncul, lalu pastikan product tersebut tidak ada lagi di daftar aktif dan tersedia di `/products/trash`.

Blok `session('success')` di layout dari tahap 3 tidak perlu diubah. Ia sudah dapat menampilkan semua pesan yang memakai key `success`.

## Yang tidak berubah

- Daftar tetap memakai `$products` paginator, search, category filter, sort, dan pagination.
- Detail tetap menggunakan slug, sedangkan update dan delete tetap menggunakan ID.
- `is_active`, SoftDeletes, dan cache dashboard tetap memiliki tanggung jawab yang berbeda.
- Pesan restore dan pesan gagal disimpan belum ditambahkan. Keduanya dibahas pada tahap berikutnya.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: menambahkan flash message saat product berhasil dikembalikan dari trash?**
