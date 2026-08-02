# Tahap 2 — Parameter `sort` dan Query Dinamis

Parameter query adalah bagian URL setelah tanda `?`. Pada halaman produk, kita akan memakai parameter `sort`.

```text
/products?sort=name-asc
```

Nilai `name-asc` adalah bahasa yang mudah dibaca pengguna. Database justru membutuhkan kolom `name` dan arah `asc`.

## Dari pilihan pengguna ke query

Secara konsep, controller akan melakukan langkah berikut:

1. membaca `sort` dari `Request`;
2. memvalidasi bahwa nilainya termasuk pilihan yang didukung;
3. mengambil pasangan kolom dan arah dari map;
4. memakai pasangan itu pada `orderBy()`.

Contoh query akhir untuk pilihan harga termurah:

```php
Product::orderBy('price', 'asc');
```

Ini disebut query dinamis karena kolom dan arahnya berubah berdasarkan pilihan yang valid.

## Jangan langsung memakai input sebagai kolom

Contoh berikut tidak boleh dipakai:

```php
Product::orderBy($request->input('sort'));
```

Parameter binding SQL melindungi nilai data, tetapi tidak dapat membind nama kolom atau arah `ORDER BY`. Jadi, kolom dan arah harus berasal dari daftar yang ditulis pengembang sendiri, bukan langsung dari URL.

Kita akan memakai map berikut:

```php
$allowedSorts = [
    'newest' => ['created_at', 'desc'],
    'oldest' => ['created_at', 'asc'],
    'price-asc' => ['price', 'asc'],
    'price-desc' => ['price', 'desc'],
    'name-asc' => ['name', 'asc'],
    'name-desc' => ['name', 'desc'],
];
```

Key seperti `price-asc` adalah nilai yang boleh dikirim pengguna. Value seperti `['price', 'asc']` adalah perintah query yang sudah kita kontrol.

## Default

Jika parameter `sort` tidak ada, halaman memakai `newest`. Ini hanya berlaku untuk input yang kosong/tidak dikirim. Input `sort` yang dikirim tetapi tidak valid akan ditolak oleh validasi Laravel; bukan diam-diam diganti default.

## Ringkasan

- URL memakai key ramah pengguna, misalnya `sort=price-asc`.
- `orderBy()` menerima kolom dan arah yang berasal dari map lokal.
- Jangan memasukkan input URL langsung ke `orderBy()`.
- Berikutnya kita menambahkan validasi dan whitelist tersebut ke controller.
