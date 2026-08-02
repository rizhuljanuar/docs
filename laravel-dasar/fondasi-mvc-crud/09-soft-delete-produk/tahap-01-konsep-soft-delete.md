# Tahap 1 — Konsep Soft Delete produk

Pada materi 08, daftar aktif tetap berada di `/products`. Saat admin menghapus produk, kita tidak langsung membuang baris database. Laravel mengisi kolom `deleted_at`, sehingga produk pindah ke tong sampah dan masih dapat dipulihkan.

## Bedanya dengan hapus permanen

| Aksi | Hasil |
|---|---|
| Soft delete | `deleted_at` terisi; produk tidak muncul pada query `Product` biasa. |
| Restore | `deleted_at` kembali `NULL`; produk muncul lagi pada daftar aktif. |
| Force delete | Baris dihapus permanen dan tidak dapat direstore. |

Soft delete berguna saat admin salah menghapus produk atau ingin menjual produk yang sama lagi. Data `name`, `price`, `stock`, `description`, `category_id`, `image`, dan `slug` tetap tersimpan, termasuk file gambar asli. Karena itu, proses soft delete **tidak boleh menghapus gambar**; gambar harus tetap tersedia ketika produk direstore.

## Dampak pada daftar dari materi 08

Setelah model memakai trait `SoftDeletes`, query normal seperti `Product::with('category')` secara otomatis hanya mengambil produk aktif (`deleted_at` bernilai `NULL`). Jadi index `/products` dari materi sorting tidak perlu diubah atau disederhanakan. Pencarian, filter kategori, sorting, dan pagination-nya tetap bekerja seperti sebelumnya.

Halaman baru `/products/trash` sengaja dipisahkan untuk mengelola produk terhapus. Halaman ini memakai urutan waktu penghapusan, bukan kontrol pencarian/filter/sort daftar aktif.

> Catatan aplikasi nyata: halaman hapus, restore, dan force delete perlu authorization. Materi ini berfokus pada mekanisme soft delete, sehingga authorization dibahas di luar cakupan.

Tahap berikutnya menyiapkan kolom `deleted_at` dan trait `SoftDeletes`.
