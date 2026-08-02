# Tahap 3 — Validasi dan Whitelist Sorting

Input dari URL tetap merupakan input pengguna. Walaupun hanya berupa pilihan sorting, nilai itu harus divalidasi sebelum dipakai untuk menentukan identifier SQL.

## Map sekaligus whitelist

Tambahkan map lokal pada method `ProductController@index`:

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

Map ini adalah whitelist karena hanya key yang ada di `array_keys($allowedSorts)` yang diterima. Map juga memastikan `orderBy()` hanya menerima kolom `created_at`, `price`, atau `name`, serta arah huruf kecil `asc` atau `desc`.

## Validasi bersama input lama

Materi sebelumnya sudah memvalidasi pencarian dan kategori. Pertahankan keduanya, kemudian tambahkan `sort`:

```php
$validated = $request->validate([
    'search' => ['nullable', 'string', 'max:100'],
    'category_id' => ['nullable', 'integer', 'exists:categories,id'],
    'sort' => ['nullable', 'string', 'in:'.implode(',', array_keys($allowedSorts))],
]);
```

Aturan `in:` membuat URL seperti berikut menerima respons validasi Laravel karena `unknown` bukan opsi yang diizinkan:

```text
/products?sort=unknown
```

Jangan menyatakan input tidak valid akan diam-diam kembali ke default. Default `newest` dipakai hanya saat `sort` tidak dikirim.

## Ambil pasangan yang sudah aman

Setelah validasi berhasil, tentukan key aktif lalu pecah pasangan map:

```php
$sort = $validated['sort'] ?? 'newest';
[$column, $direction] = $allowedSorts[$sort];
```

Tidak diperlukan `request()` langsung untuk data controller. Nilai diambil dari `$validated`, lalu `$column` dan `$direction` hanya berasal dari map yang aman.

## Ringkasan

- Validasi input `search`, `category_id`, dan `sort` dalam satu `validate()`.
- Whitelist lebih aman daripada mencoba memblokir nilai-nilai buruk satu per satu.
- Parameter binding tidak dapat menggantikan whitelist untuk nama kolom.
- Tahap berikutnya menyatukan kode ini dengan query pencarian, kategori, dan pagination yang sudah ada.
