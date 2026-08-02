# Tahap 6 — Force Delete dan Ringkasan

Force delete menghapus baris produk secara permanen. Tidak ada restore setelah aksi ini, jadi tombol hanya tersedia di tong sampah.

## Tambahkan method `forceDelete`

Trait `SoftDeletes` menyediakan method `forceDelete()`. Karena itu trait tersebut harus sudah dipakai oleh model `Product`.

```php
public function forceDelete(int $id)
{
    $product = Product::onlyTrashed()->findOrFail($id);
    $name = $product->name;

    $product->forceDelete();

    return redirect('/products/trash')->with('success', "produk {$name} dihapus permanen.");
}
```

`onlyTrashed()` adalah guard di server: produk aktif tidak bisa di-force-delete melalui endpoint ini. `$name` disimpan sebelum `forceDelete()` karena setelah baris dihapus, data produk tidak lagi tersedia. Pada aplikasi produksi, tambahkan authorization sebelum aksi ini.

Jika ada file gambar produk, tentukan kebijakan penyimpanan dengan hati-hati. Materi ini sengaja tidak menghapus gambar saat soft delete agar restore aman. Pembersihan file yang sudah benar-benar tidak lagi dipakai dapat dirancang terpisah setelah memastikan tidak ada referensi lain.

## Route dan form force delete

```php
Route::delete('/products/{id}/force-delete', [ProductController::class, 'forceDelete']);
```

```blade
<form action="/products/{{ $product->id }}/force-delete" method="POST" onsubmit="return confirm('Hapus permanen produk ini?');">
    @csrf
    @method('DELETE')
    <button type="submit">Hapus permanen</button>
</form>
```

Form HTML biasa dengan `confirm()` sudah cukup sebagai peringatan pengguna. Perlindungan yang penting tetap query `Product::onlyTrashed()->findOrFail($id)` di controller, ditambah authorization pada aplikasi nyata.

## Ringkasan alur

1. Migration menambahkan `$table->softDeletes()` dan rollback memakai `$table->dropSoftDeletes()`.
2. Model mengimpor serta memakai `Illuminate\Database\Eloquent\SoftDeletes`; `$fillable` tetap berisi `name`, `price`, `stock`, `description`, `category_id`, `image`, dan `slug`.
3. `destroy(int $id)` memakai `Product::findOrFail($id)->delete()` lalu mengarahkan ke `/products` dengan pesan bahwa produk dipindahkan ke tong sampah.
4. Daftar aktif `/products` tetap menggunakan query sorting final materi 08; trashed product otomatis tidak muncul.
5. `trash()` memakai `Product::onlyTrashed()->with('category')->orderByDesc('deleted_at')->orderBy('id')->paginate(10)->withQueryString()` dan mengirim `$trashedProducts` ke `products.trash`.
6. `restore(int $id)` dan `forceDelete(int $id)` sama-sama memakai `Product::onlyTrashed()->findOrFail($id)` lalu redirect ke `/products/trash`.
7. URL detail tetap `/products/{{ $product->slug }}`, sedangkan edit, soft delete, restore, dan force delete memakai ID.

Dengan susunan ini, admin dapat memindahkan produk ke tong sampah, meninjau produk yang dihapus berdasarkan waktu penghapusan, merestore data beserta gambar yang masih tersedia, atau menghapusnya secara permanen bila benar-benar diperlukan.
