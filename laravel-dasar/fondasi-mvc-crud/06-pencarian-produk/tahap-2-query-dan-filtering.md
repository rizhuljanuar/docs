# Tahap 2: Memahami Query dan Filtering

## `where` untuk Menyaring Data

Query adalah permintaan aplikasi kepada database. Untuk daftar produk, kita dapat menambahkan syarat dengan `where()`.

```php
Product::where('name', 'like', '%laptop%')->get();
```

Artinya: ambil produk yang nilai `name`-nya mengandung `laptop`.

- `name` adalah nama kolom pada tabel `products`.
- `like` mencari teks sebagian.
- `%` berarti teks apa pun boleh ada sebelum atau sesudah kata kunci.

Berbeda dengan `=`, `like` cocok untuk kata yang diketik bebas oleh pengguna.

## `category_id` untuk Filter Kategori

Kategori dipilih dari dropdown sehingga nilainya harus sama persis dengan ID kategori.

```php
Product::where('category_id', 2)->get();
```

Ini bukan pencarian teks. `2` adalah ID pada tabel `categories`, sehingga kita memakai `where()` biasa, bukan `like`.

## Huruf Besar dan Kecil

Hasil `LIKE` bergantung pada database dan *collation*-nya. Banyak konfigurasi MySQL/MariaDB dengan collation berakhiran `_ci` bersifat tidak peka huruf besar-kecil, tetapi konfigurasi binary atau case-sensitive dapat berbeda. Jangan menganggap perilakunya selalu sama pada setiap database.

## Catatan Performa

Indeks B-tree biasa dapat membantu filter sama persis seperti `category_id = 2` dan pencarian awalan seperti `name LIKE 'lap%'` pada banyak database. Namun, indeks B-tree biasa **tidak mengoptimalkan** pencarian mengandung teks dengan pola `%laptop%` karena wildcard berada di awal.

Untuk data sangat besar, evaluasi kebutuhan pencarian khusus (misalnya full-text search) berdasarkan database dan kebutuhan aplikasi. Materi ini tetap memakai fitur standar Laravel dan query sederhana.

## Inti Tahap 2

> Gunakan `like` untuk kata kunci teks pada `name` atau `description`, serta `where('category_id', ...)` untuk kategori yang dipilih. Tahap berikutnya menerapkannya langsung di controller.
