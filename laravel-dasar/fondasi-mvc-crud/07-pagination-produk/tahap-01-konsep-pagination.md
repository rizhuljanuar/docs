# Tahap 1 — Pagination Melanjutkan Pencarian dan Filter Produk

> Prasyarat: materi **Pencarian Produk** sudah selesai.

## Tujuan

Di materi sebelumnya, halaman `/products` sudah memakai pagination final berikut:

```php
$products = Product::with('category')
    ->search($validated['search'] ?? '')
    ->filterByCategory($validated['category_id'] ?? null)
    ->paginate(10)
    ->withQueryString();
```

Jadi materi ini **bukan** mengganti query lama dengan query yang lebih sederhana. Kita akan memperdalam hasil `paginate(10)`, link halaman, dan cara menjaga pencarian serta filter kategori saat pengguna berpindah halaman.

## Apa itu pagination?

Pagination membagi daftar produk panjang menjadi beberapa halaman. Dengan `paginate(10)`, satu halaman menampilkan paling banyak 10 produk.

Contoh: jika ada 25 hasil pencarian, halaman pertama berisi produk 1–10, halaman kedua 11–20, dan halaman ketiga 21–25.

## Yang dilakukan Laravel

`paginate(10)` tidak berarti database selalu hanya mengerjakan 10 data. Laravel melakukan:

1. query `count` untuk mengetahui jumlah seluruh hasil yang cocok;
2. query halaman untuk mengambil maksimal 10 produk pada halaman yang diminta.

Hasilnya adalah paginator: data produk yang dapat di-loop, serta informasi seperti jumlah total, halaman saat ini, dan link navigasi.

## Mengapa `withQueryString()` penting?

Pengguna dapat mencari dan memilih kategori, misalnya:

```text
/products?search=laptop&category_id=2
```

Saat ia membuka halaman berikutnya, `withQueryString()` membuat link tetap membawa parameter tersebut:

```text
/products?search=laptop&category_id=2&page=2
```

Tanpa itu, halaman kedua dapat kehilangan kata pencarian dan pilihan kategori.

## Ringkasan

- `paginate(10)` sudah ada sejak materi sebelumnya.
- `$products` tetap berisi hasil pencarian dan filter kategori.
- `withQueryString()` mempertahankan parameter GET pada link pagination.
