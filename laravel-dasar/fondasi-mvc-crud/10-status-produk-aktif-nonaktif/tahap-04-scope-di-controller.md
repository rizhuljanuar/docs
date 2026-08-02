# Tahap 4 — Controller dan Route Status

Materi sebelumnya sudah memfinalkan index manajemen `/products`. Jangan menggantinya dengan query sederhana dan jangan membuat `adminIndex()` atau halaman admin kedua. Index ini tetap menjadi sumber daftar produk aktif dan nonaktif yang belum di-soft-delete.

## Index yang harus dipertahankan

```php
public function index(Request $request)
{
    $allowedSorts = [
        'newest' => ['created_at', 'desc'],
        'oldest' => ['created_at', 'asc'],
        'price-asc' => ['price', 'asc'],
        'price-desc' => ['price', 'desc'],
        'name-asc' => ['name', 'asc'],
        'name-desc' => ['name', 'desc'],
    ];

    $validated = $request->validate([
        'search' => ['nullable', 'string', 'max:100'],
        'category_id' => ['nullable', 'integer', 'exists:categories,id'],
        'sort' => ['nullable', 'string', 'in:' . implode(',', array_keys($allowedSorts))],
    ]);

    [$column, $direction] = $allowedSorts[$validated['sort'] ?? 'newest'];

    $products = Product::with('category')
        ->search($validated['search'] ?? null)
        ->filterByCategory($validated['category_id'] ?? null)
        ->orderBy($column, $direction)
        ->orderBy('id')
        ->paginate(10)
        ->withQueryString();

    $categories = Category::orderBy('name')->get();

    return view('products.index', compact('products', 'categories'));
}
```

Tidak ada `active()` di query ini. Soft delete tetap otomatis mengecualikan produk di tong sampah.

## Endpoint status

Tambahkan method terpisah pada `ProductController`:

```php
public function updateStatus(Request $request, int $id)
{
    $validated = $request->validate([
        'is_active' => ['required', 'boolean'],
    ]);

    $product = Product::findOrFail($id);
    $product->update($validated);

    return redirect('/products')->with('success', 'Status produk berhasil diperbarui.');
}
```

`Product::findOrFail($id)` sengaja tidak memakai `withTrashed()`: produk di tong sampah menghasilkan 404 dan harus direstore terlebih dahulu. Validasi status hanya berada di method ini; jangan mencampurnya dengan aturan create/update yang sudah ada kecuali nanti fitur itu memang diperluas.

Daftarkan route manual ini sebelum route slug catch-all terakhir:

```php
Route::patch('/products/{id}/status', [ProductController::class, 'updateStatus']);

// Route CRUD manual yang telah ada: index/create/store, trash, restore,
// force delete, edit/update/destroy berdasarkan ID.
Route::get('/products/{slug}', [ProductController::class, 'show']);
```

Gunakan deklarasi route manual dan URL literal sesuai kontrak yang dipertahankan; jangan memakai named/resource route atau helper URL. Pada aplikasi nyata, tambahkan authorization sebelum update status; materi ini tidak membuat policy.
