# Tahap 10 — Ringkasan dan Checklist Layout Blade

> Penutup topik 12: rapikan tampilan tanpa mengubah kontrak aplikasi dari tahap 05–11.

## Empat alat Blade

| Alat | Tanggung jawab | Contoh pada pelajaran ini |
|---|---|---|
| Layout | kerangka halaman bersama | `layouts/app.blade.php` |
| `@extends` + `@section` | child view mengisi kerangka | `products/index.blade.php` |
| Partial | fragmen statis kecil | navbar dan footer |
| Anonymous component | UI berulang dengan props atau slots | kartu `Product`, badge, panel |

Konvensi akhir hanya memakai satu layout: `resources/views/layouts/app.blade.php`. Semua child template memakai `@extends('layouts.app')` dan `@section('content')`. Layout menyediakan judul default, `@stack('css')`, `@yield('content')`, `@stack('js')`, serta navigasi literal ke `/products`, `/products/trash`, dan `/dashboard`.

## Checklist refactor view

### Daftar Products

- [ ] View tetap menerima `$products` paginator.
- [ ] Pencarian, filter `Category`, sort, dan pagination tahap 06–08 tetap ada.
- [ ] Detail adalah `/products/{{ $product->slug }}`.
- [ ] Edit adalah `/products/{{ $product->id }}/edit`.
- [ ] Form hapus memakai `DELETE` ke `/products/{{ $product->id }}`.

### Create, edit, dan detail

- [ ] Form create `POST` ke `/products`.
- [ ] Form edit `PUT` ke `/products/{{ $product->id }}`.
- [ ] Data memakai `name`, `price`, `stock`, `description`, `category_id`, `image`, dan `is_active`.
- [ ] Relasi ditampilkan sebagai `$product->category?->name`.
- [ ] Image diperiksa sebelum ditampilkan.

### Dashboard

- [ ] Route tetap `Route::get('/dashboard', [DashboardController::class, 'index']);`.
- [ ] View tetap `dashboard.index`.
- [ ] Metrik memakai `managedProductsCount`, `activeProductsCount`, `inactiveProductsCount`, `trashedProductsCount`, `totalStock`, dan `latestProducts`.
- [ ] Tidak ada perubahan pada cache `dashboard.products.summary`, durasi lima menit, atau query latest products dari tahap 11.

### Status dan data sampah

- [ ] `is_active` menyatakan status `Product`.
- [ ] SoftDeletes memakai `deleted_at`; data tersebut dibuka di `/products/trash`.
- [ ] Keduanya tidak disamakan.
- [ ] Invalidasi cache dashboard tetap dilakukan hanya setelah mutasi `Product` yang sudah ada berhasil.

## Kesalahan yang dihindari

1. Membuat layout kedua padahal satu layout sudah cukup.
2. Memasukkan query, perubahan cache, atau logika mutasi ke layout.
3. Mengganti daftar paginator dengan data tanpa pencarian/filter/sort/pagination.
4. Menggunakan ID untuk tautan detail yang seharusnya memakai slug.
5. Merender image atau relasi tanpa pemeriksaan yang diperlukan.
6. Memperlakukan status aktif sebagai data SoftDeletes.

Pelajaran ini hanya mengatur presentasi Blade. Authorization dan middleware berada di luar cakupan. Dengan batas itu, tampilan menjadi lebih konsisten sambil seluruh kontrak CRUD dan dashboard sebelumnya tetap aman.
