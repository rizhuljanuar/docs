# Tahap 1 — Layout Blade untuk Product Management

> Mentor: Laravel Dasar — Fondasi MVC, CRUD
> Topik: 12. Layout Blade Reusable

## Tujuan

Setelah tahap ini kamu memahami alasan memakai layout Blade sebelum mulai memindahkan tampilan yang sudah dibuat pada tahap 05–11.

Aplikasi yang diteruskan di sini hanya mengelola `Product` dan `Category`. Halaman yang sudah ada adalah:

- `/products` untuk daftar `Product`;
- `/products/trash` untuk data `Product` yang dihapus sementara;
- `/dashboard` untuk ringkasan pengelolaan.

Setiap halaman biasanya memiliki `<html>`, `<head>`, navigasi, dan footer yang sama. Jika ditulis ulang pada setiap view, perubahan satu tautan harus dilakukan di banyak tempat. Layout menaruh bagian bersama itu di satu file; child view hanya mengisi isi halaman.

```text
layouts/app.blade.php
├── judul halaman dan CSS bersama
├── navigasi /products, /products/trash, /dashboard
├── @yield('content')  ← isi berbeda dari tiap child view
└── JavaScript bersama
```

Layout bukan controller, route, atau model baru. Ia hanya cara menyusun Blade. Refactor ini tidak mengubah query daftar `$products`, endpoint mutasi, SoftDeletes, atau cache dashboard dari tahap sebelumnya.

## Kontrak yang diteruskan

Gunakan istilah dan data berikut secara konsisten:

| Hal | Nilai yang dipakai |
|---|---|
| Model | `Product`, `Category` |
| Kolom `Product` | `name`, `price`, `stock`, `description`, `category_id`, `image`, `slug`, `is_active` |
| Relasi kategori | `$product->category?->name` |
| Detail | `/products/{{ $product->slug }}` |
| Hapus | SoftDeletes, ditandai `deleted_at` |
| Status | `is_active`, terpisah dari data sampah |

Satu layout sederhana cukup untuk tiga halaman tersebut. Kita tidak membuat layout kedua atau sidebar khusus. Authorization dan middleware juga berada di luar cakupan pelajaran ini.

## Latihan pikir

1. Bagian mana yang seharusnya hanya ditulis sekali? Jawab: kerangka dokumen, navigasi, footer, dan tempat CSS/JavaScript bersama.
2. Bagian mana yang tetap berada di child view? Jawab: tabel `$products`, form, detail satu `Product`, dan metrik dashboard.
3. Apakah layout mengubah data dari controller? Tidak. Variabel yang sudah dikirim controller tetap dipakai oleh view.

Tahap berikutnya membuat satu file layout yang menjadi kerangka tersebut.
