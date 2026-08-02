# Tahap 1 — Konsep Status Aktif/Nonaktif

Materi ini melanjutkan CRUD produk yang sudah selesai sampai materi 09 (soft delete). Kontrak yang sudah ada tetap dipakai: tabel `products`, model `Product`, controller `ProductController`, URL `/products`, dan field `name`, `price`, `stock`, `description`, `category_id`, `image`, serta `slug`.

## Dua arti status yang berbeda

Tambahkan `is_active` untuk menyatakan **status publikasi**:

| Kondisi | Arti |
|---|---|
| `is_active = true` | Item siap dipublikasikan pada permukaan publik yang akan dibuat nanti. |
| `is_active = false` | Item masih draft atau sementara tidak dipublikasikan. |

Jangan samakan dengan soft delete:

| Konsep | Kolom | Arti |
|---|---|---|
| Soft delete | `deleted_at` | Item berada di tong sampah. |
| Status publikasi | `is_active` | Item non-trashed boleh atau belum boleh dipublikasikan. |

Daftar manajemen yang ada di `/products` **tetap menampilkan produk aktif dan nonaktif**, selama produk belum di-soft-delete. Itu sengaja: admin perlu melihat serta mengelola draft. `SoftDeletes` pada query normal hanya berarti `deleted_at IS NULL`; ia tidak berarti `is_active = true`.

Item yang direstore dari tong sampah mempertahankan nilai `is_active` terakhirnya. Contoh: produk nonaktif yang dihapus lalu direstore akan tetap nonaktif.

## Alur materi

1. Tambahkan kolom boolean `is_active` dengan default `false`.
2. Tambahkan cast dan local scope `active()` pada `Product`.
3. Pertahankan index manajemen `/products` beserta pencarian, kategori, sorting, dan pagination final.
4. Tambahkan badge serta kontrol status pada index yang sama.
5. Kirim perubahan status eksplisit melalui form `PATCH`.

Scope `active()` disiapkan untuk query **public storefront di masa depan**. Materi ini tidak membuat route, controller, view, atau perubahan perilaku detail publik/manajemen baru. Detail yang telah ada tetap memakai slug.
