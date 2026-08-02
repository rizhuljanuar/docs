# Tahap 3 — Memahami `paginate(10)` dan Query String

## Hasil dari `paginate(10)`

Setelah query ini dijalankan, `$products` dapat di-loop seperti koleksi biasa:

```blade
@foreach ($products as $product)
    {{ $product->name }}
@endforeach
```

Namun `$products` juga memiliki method pagination berikut:

| Method | Kegunaan |
|---|---|
| `total()` | jumlah seluruh hasil yang cocok |
| `currentPage()` | halaman yang sedang dibuka |
| `lastPage()` | halaman terakhir |
| `firstItem()` | nomor urut hasil pertama di halaman ini |
| `lastItem()` | nomor urut hasil terakhir di halaman ini |
| `links()` | membuat link navigasi halaman |

## Alur URL

Laravel membaca halaman dari parameter `page`.

```text
/products?page=2
```

Jika pengguna sedang mencari laptop dan memfilter kategori ID 2, parameter lama harus ikut disimpan:

```text
/products?search=laptop&category_id=2&page=2
```

Method ini yang menjaga parameter tersebut:

```php
->paginate(10)
->withQueryString();
```

Panggil `withQueryString()` setelah `paginate(10)` karena method itu bekerja pada paginator yang sudah dibuat.

## Periksa dengan cepat

1. Buka `/products?search=laptop&category_id=2`.
2. Klik halaman berikutnya.
3. Pastikan URL masih memuat `search=laptop` dan `category_id=2` serta menambahkan `page=2`.

Jika parameter hilang, periksa kembali bahwa `->withQueryString()` masih ada di akhir query controller.
