# Tahap 1 — Konsep Sorting Produk

Sorting adalah cara mengurutkan daftar produk agar pengunjung dapat melihat data sesuai kebutuhannya. Contohnya, pembeli dapat memilih produk terbaru, harga termurah, atau nama A–Z.

Materi ini melanjutkan halaman daftar dari materi pencarian dan pagination. Kontrak yang sudah ada tetap dipakai:

- model `Product` dan controller `ProductController`;
- tabel `products`, variabel `$products` dan `$categories`;
- view `resources/views/products/index.blade.php`;
- route manual:

```php
Route::get('/products', [ProductController::class, 'index']);
```

## Pilihan urutan

| Nilai URL | Tampilan | Kolom | Arah |
|---|---|---|---|
| `newest` | Terbaru | `created_at` | `desc` |
| `oldest` | Terlama | `created_at` | `asc` |
| `price-asc` | Harga: rendah ke tinggi | `price` | `asc` |
| `price-desc` | Harga: tinggi ke rendah | `price` | `desc` |
| `name-asc` | Nama: A–Z | `name` | `asc` |
| `name-desc` | Nama: Z–A | `name` | `desc` |

`asc` berarti urutan naik (A–Z atau kecil ke besar), sedangkan `desc` berarti urutan turun.

## Sorting dalam URL

Pilihan dikirim melalui query string, misalnya:

```text
/products?sort=price-asc
```

Sorting dapat digabungkan dengan fitur sebelumnya:

```text
/products?search=laptop&category_id=2&sort=price-asc&page=2
```

Artinya: cari “laptop”, batasi ke kategori ID 2, urutkan harga termurah, lalu buka halaman 2.

## Mengapa pagination perlu urutan yang stabil?

Saat beberapa produk mempunyai harga atau nama yang sama, database tidak selalu menjamin urutan relatifnya bila hanya memakai satu `orderBy`. Karena itu, nanti kita menambahkan `orderBy('id')` sebagai urutan kedua. ID naik membuat isi halaman tetap konsisten saat pengguna berpindah halaman.

## Ringkasan

- Sorting mengubah urutan, bukan data produk.
- Pilihan pengguna dikirim melalui parameter `sort`.
- Sorting harus tetap bekerja bersama pencarian, filter kategori, dan pagination.
- Tahap berikutnya membahas cara menerjemahkan nilai `sort` yang ramah pengguna menjadi kolom database yang aman.
