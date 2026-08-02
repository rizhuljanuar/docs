# Tahap 1: Apa Itu Pencarian Produk?

## Tujuan

Pada materi sebelumnya, daftar produk sudah memakai model `Product`, tabel `products`, controller `ProductController`, dan view `resources/views/products/index.blade.php`. Variabel daftar produknya adalah `$products`.

**Pencarian produk** menyaring daftar tersebut agar pengguna tidak perlu membaca semua data satu per satu.

Contoh: saat pengguna mencari `laptop`, daftar dapat menampilkan produk yang nama atau deskripsinya memuat kata tersebut.

## Data yang Sudah Ada

Produk memakai field berikut:

| Field | Kegunaan |
|---|---|
| `name` | Nama produk |
| `price` | Harga produk |
| `stock` | Jumlah stok |
| `description` | Penjelasan produk |
| `category_id` | ID kategori produk |
| `image` | Lokasi gambar |
| `slug` | URL detail produk |

Model `Product` tetap memiliki relasi `category()`. Karena kategori adalah relasi tersendiri, filter kategori nanti memakai `category_id`, bukan kolom teks `category` pada tabel `products`.

## Alur yang Akan Dibuat

```text
Pengguna mengisi form GET
        ↓
/products?search=laptop&category_id=2
        ↓
ProductController@index membaca input
        ↓
Product mencari name/description dan category_id
        ↓
$products dikirim ke products/index.blade.php
```

Halaman detail tetap menggunakan:

```blade
<a href="/products/{{ $product->slug }}">Lihat</a>
```

Edit dan hapus tetap menggunakan ID sesuai route dari materi sebelumnya.

## Inti Tahap 1

> Pencarian adalah penyaringan data pada halaman daftar produk. Kita akan mempertahankan kontrak Product dan slug dari Materi 5, lalu menambah pencarian secara bertahap.

Tahap berikutnya membahas `where`, `like`, dan filter kategori.
