# Tahap 1 — Konsep Dashboard Product

> Materi lanjutan: CRUD product, sorting, soft delete, dan status publikasi
> Level: Pemula Laravel

## Tujuan dashboard

Dashboard adalah halaman ringkas untuk melihat keadaan data yang dikelola. Pada seri ini dashboard **hanya** merangkum `Product` dan `Category` yang sudah ada. Tidak ada tabel atau model baru.

Informasi yang berguna untuk pengelolaan product:

- jumlah product non-trash;
- jumlah product aktif dan tidak aktif;
- jumlah product di trash;
- total stok product non-trash;
- lima product yang paling baru dibuat.

## Dua keadaan yang berbeda

`is_active` adalah status publikasi. `deleted_at` dari `SoftDeletes` adalah status penghapusan. Keduanya tidak boleh dicampur.

| Keadaan | `deleted_at` | `is_active` | Masuk hitungan product dikelola? |
|---|---|---:|---|
| Draft | `NULL` | false | Ya |
| Dipublikasikan | `NULL` | true | Ya |
| Trash | terisi | true atau false | Tidak |

Karena global scope `SoftDeletes`, query biasa `Product::...` hanya membaca product non-trash. Trash selalu dihitung terpisah dengan `Product::onlyTrashed()`.

Rumus ringkasannya adalah:

```text
product non-trash = product aktif + product tidak aktif
trash = hitungan terpisah
```

## Batas materi

Dashboard ini membantu mengelola data, tetapi bukan panel yang aman untuk penggunaan nyata. Aplikasi nyata memerlukan authorization dan middleware yang sesuai sebelum dashboard dibuka oleh pihak tertentu. Pembahasan itu berada di luar seri ini.

Tahap berikutnya membuat alamat literal `/dashboard`, controller, dan view dasar tanpa mengubah CRUD `/products` yang telah selesai.
