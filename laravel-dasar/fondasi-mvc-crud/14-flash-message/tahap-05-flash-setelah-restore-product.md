# Tahap 5 — Flash Message Setelah Restore Product

> Fokus: memberi tahu user bahwa product berhasil keluar dari trash dan kembali ke daftar aktif.

## Apa arti restore?

Pada tahap 4, tombol delete melakukan **soft delete**. Product tidak langsung hilang permanen. Laravel mengisi `deleted_at`, lalu product dipindahkan dari daftar aktif ke:

```text
/products/trash
```

**Restore** adalah tindakan mengembalikan product tersebut. Laravel mengosongkan `deleted_at`, sehingga product kembali muncul pada query Product biasa di `/products`.

Analogi sederhananya: product dipindahkan ke kotak barang sementara. Restore berarti mengambilnya kembali dari kotak itu dan menaruhnya lagi di rak utama.

## Restore hanya untuk product di trash

Method `restore()` perlu mencari product memakai `onlyTrashed()`, bukan query Product biasa:

```php
$product = Product::onlyTrashed()->findOrFail($id);
```

Arti sederhananya:

- `onlyTrashed()` berarti “cari hanya di trash”.
- `findOrFail($id)` mencari product berdasarkan ID.
- Jika ID itu bukan product di trash, Laravel memberi respons 404. Ini mencegah product aktif dipulihkan oleh endpoint restore.

## Tambahkan flash success setelah restore

Di `ProductController`, pertahankan method restore yang sudah ada. Setelah `$product->restore()` berhasil, hapus cache dashboard lalu redirect kembali ke trash dengan pesan berikut:

```php
public function restore(int $id)
{
    $product = Product::onlyTrashed()->findOrFail($id);
    $product->restore();

    Cache::forget('dashboard.products.summary');

    return redirect('/products/trash')->with('success', 'Product berhasil dikembalikan');
}
```

Baca kode secara urut:

| Baris | Fungsi |
| --- | --- |
| `Product::onlyTrashed()->findOrFail($id)` | Mengambil hanya product yang sedang berada di trash. |
| `$product->restore()` | Mengosongkan `deleted_at` dan mengembalikan product ke daftar aktif. |
| `Cache::forget(...)` | Membuat dashboard menghitung data terbaru setelah restore. |
| `redirect('/products/trash')` | Mengembalikan user ke halaman trash. |
| `with('success', ...)` | Membawa pesan sementara ke halaman tujuan. |

Pesan yang tampil adalah:

```text
Product berhasil dikembalikan
```

## Mengapa redirect kembali ke trash?

Setelah restore berhasil, product tersebut tidak lagi berada di trash. Redirect ke `/products/trash` membantu user melihat bahwa barisnya sudah hilang dari daftar trash.

User juga dapat membuka `/products` untuk melihat product tersebut kembali pada daftar aktif. Query index yang sudah ada tetap mengurus category relation, search, filter, sort, pagination, dan query string.

## Form restore tetap memakai ID

Di halaman `/products/trash`, form restore yang telah dibuat sebelumnya tetap mengirim POST ke endpoint literal berbasis ID:

```blade
<form action="/products/{{ $product->id }}/restore" method="POST">
    @csrf
    <x-button type="submit" variant="primary">Restore</x-button>
</form>
```

Tidak ada perubahan pada route atau bentuk form. Kita hanya menambahkan pesan flash setelah restore benar-benar berhasil.

## Jangan tertukar dengan `is_active`

Restore hanya mengubah `deleted_at`. Restore tidak mengubah `is_active`.

| Tindakan | `deleted_at` | `is_active` |
| --- | --- | --- |
| Soft delete | Diisi | Tetap seperti sebelumnya |
| Restore | Dikosongkan | Tetap seperti sebelumnya |
| Ubah status | Tetap | Diubah menjadi aktif atau tidak aktif |

Jadi product yang sebelumnya tidak aktif akan tetap tidak aktif setelah dipulihkan. Itu adalah perilaku yang benar karena lifecycle trash dan status publikasi adalah dua hal berbeda.

## Coba sendiri

1. Hapus satu product dari `/products`.
2. Buka `/products/trash`.
3. Tekan tombol **Restore** pada product tersebut.
4. Pesan **Product berhasil dikembalikan** harus muncul.
5. Product menghilang dari trash dan kembali tersedia di `/products`.

Blok `session('success')` pada layout dari tahap 3 tetap digunakan. Tidak perlu membuat blok Blade baru untuk restore.

## Yang tidak berubah

- Restore dan delete memakai ID; detail product tetap memakai slug.
- Daftar aktif tetap memakai `$products` paginator, search, category filter, sort, dan pagination.
- Cache dashboard tetap dihapus hanya setelah mutasi berhasil.
- Pesan error atau validasi gagal belum ditambahkan. Itu dibahas pada tahap berikutnya.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: memahami pesan error dan validasi gagal saat menyimpan product?**
