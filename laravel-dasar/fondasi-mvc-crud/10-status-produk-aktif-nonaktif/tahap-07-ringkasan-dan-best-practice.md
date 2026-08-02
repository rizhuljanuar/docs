# Tahap 7 — Ringkasan dan Best Practice

## Checklist implementasi

1. Migration baru menambah `$table->boolean('is_active')->default(false);` dan rollback memakai `$table->dropColumn('is_active');`.
2. `Product` tetap memakai `HasFactory`, `SoftDeletes`, `category(): BelongsTo`, dan fillable lama `name`, `price`, `stock`, `description`, `category_id`, `image`, `slug`; tambahkan `is_active`.
3. Model memiliki `casts(): array` untuk boolean dan local scope typed `scopeActive(Builder $query): Builder`.
4. Index `/products` tetap memakai pencarian, filter kategori, whitelist sorting, relasi kategori, pagination, dan `withQueryString()` tanpa `active()`.
5. Badge serta form PATCH status berada di `resources/views/products/index.blade.php`, bukan view baru.
6. Route manual `PATCH /products/{id}/status` terdaftar sebelum final `GET /products/{slug}`.
7. `updateStatus(Request $request, int $id)` hanya memvalidasi `is_active` dengan `required|boolean`, memakai `Product::findOrFail($id)`, lalu `$product->update($validated)` dan redirect ke `/products`.

## Lifecycle yang tepat

| Kondisi | `deleted_at` | `is_active` | Tampil pada index `/products`? |
|---|---:|---:|---|
| Draft | `NULL` | false | Ya |
| Siap publik | `NULL` | true | Ya |
| Di tong sampah | terisi | true/false | Tidak |

Status publikasi dan penghapusan mempunyai tanggung jawab berbeda. `Product::active()` untuk storefront masa depan menghasilkan produk aktif yang juga otomatis bukan trashed, sebab trait `SoftDeletes` memasang scope global. Sebaliknya, index manajemen tidak memakai `active()` agar admin melihat semua produk non-trashed. Halaman tong sampah tetap memakai query `Product::onlyTrashed()->with('category')->orderByDesc('deleted_at')->orderBy('id')->paginate(10)->withQueryString()` dan tetap terpisah dari kontrol status.

## Indeks database

Jangan otomatis menambah indeks tunggal untuk boolean `is_active`. Nilai boolean memiliki kardinalitas rendah sehingga indeks tunggal sering tidak menguntungkan. Putuskan berdasarkan pengukuran pola query storefront kelak; dalam banyak kasus indeks komposit bersama kolom filter atau ordering yang benar-benar dipakai lebih berguna. Tidak ada migration indeks tambahan pada materi ini.

## Keamanan dan pengembangan berikutnya

CSRF dan validasi melindungi format request serta mass assignment status. Aplikasi nyata juga harus menambahkan authorization pada endpoint status, tetapi materi ini tidak menambahkan policy. Saat nanti membuat storefront terpisah, gunakan scope `Product::active()` pada query halaman tersebut; jangan mengubah detail atau index manajemen yang sudah ada.
