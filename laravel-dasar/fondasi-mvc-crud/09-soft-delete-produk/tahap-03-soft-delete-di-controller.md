# Tahap 3 — Soft Delete pada `ProductController`

Route hapus aktif dari materi sebelumnya tetap memakai ID dan method `DELETE`. Dengan trait `SoftDeletes` pada `Product`, `delete()` pada model aktif akan mengisi `deleted_at`, bukan menghapus baris secara permanen.

## Method `destroy`

Di `app/Http/Controllers/ProductController.php`, gunakan method berikut.

```php
public function destroy(int $id)
{
    Product::findOrFail($id)->delete();

    return redirect('/products')->with('success', 'produk dipindahkan ke tong sampah.');
}
```

Jangan hapus file pada kolom `image` di method ini. produk dapat direstore, sehingga gambar aslinya harus tetap tersedia.

## Form pada daftar aktif

Daftar aktif tetap mempertahankan form ID-based berikut.

```blade
<form action="/products/{{ $product->id }}" method="POST">
    @csrf
    @method('DELETE')
    <button type="submit">Hapus</button>
</form>
```

Setelah sukses, admin kembali ke `/products`. produk yang baru dihapus tidak lagi muncul karena query normal `Product` dari index materi 08 otomatis mengecualikan trashed model.

## Index materi 08 tetap utuh

Jangan mengganti query index dengan query yang lebih sederhana. Query final tetap mencakup validasi `search`, `category_id`, dan `sort`, safe map sorting, lalu:

```php
Product::with('category')
    ->search($validated['search'] ?? '')
    ->filterByCategory($validated['category_id'] ?? null)
    ->orderBy($column, $direction)
    ->orderBy('id')
    ->paginate(10)
    ->withQueryString();
```

SoftDeletes bekerja sebagai global scope pada query tersebut. Tahap berikutnya membuat query khusus untuk isi tong sampah.
