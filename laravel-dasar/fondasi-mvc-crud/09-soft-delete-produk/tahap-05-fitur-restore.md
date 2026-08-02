# Tahap 5 — Restore produk dari Tong Sampah

Restore mengembalikan produk yang berada di tong sampah ke daftar aktif. Laravel mengosongkan `deleted_at`, sehingga produk kembali terlihat pada query index normal.

## Tambahkan method `restore`

Di `ProductController`, gunakan `onlyTrashed()` agar endpoint ini hanya menerima produk yang memang sedang berada di tong sampah.

```php
public function restore(int $id)
{
    $product = Product::onlyTrashed()->findOrFail($id);
    $product->restore();

    return redirect('/products/trash')->with('success', 'produk berhasil direstore.');
}
```

Jangan memakai query yang mencakup produk aktif untuk endpoint ini. Jika ID menunjuk produk yang aktif atau tidak ada, `findOrFail()` menghasilkan 404. Setelah restore, file gambar lama tetap dapat dipakai karena soft delete sebelumnya tidak menghapusnya.

## Form restore

View tong sampah mengirim POST ke URL manual berikut:

```blade
<form action="/products/{{ $product->id }}/restore" method="POST">
    @csrf
    <button type="submit">Restore</button>
</form>
```

Route yang menerima form tersebut:

```php
Route::post('/products/{id}/restore', [ProductController::class, 'restore']);
```

Setelah restore berhasil, redirect tetap ke `/products/trash`. produk yang dipulihkan menghilang dari tong sampah dan muncul kembali di `/products` sesuai query index final materi 08: relasi kategori, pencarian, filter kategori, safe sorting map, tie-breaker ID, pagination, dan query string tetap tidak berubah.

> Untuk aplikasi nyata, batasi aksi restore dengan authorization yang sesuai peran admin.
