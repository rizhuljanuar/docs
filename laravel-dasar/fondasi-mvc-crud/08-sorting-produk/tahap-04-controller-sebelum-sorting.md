# Tahap 4 — Controller Sebelum Sorting

Sebelum menambahkan sorting, halaman indeks sudah memiliki pencarian, filter kategori, relasi kategori, dan pagination. Semua bagian itu harus dipertahankan.

```php
public function index(Request $request)
{
    $validated = $request->validate([
        'search' => ['nullable', 'string', 'max:100'],
        'category_id' => ['nullable', 'integer', 'exists:categories,id'],
    ]);

    $products = Product::with('category')
        ->search($validated['search'] ?? '')
        ->filterByCategory($validated['category_id'] ?? null)
        ->paginate(10)
        ->withQueryString();

    $categories = Category::orderBy('name')->get();

    return view('products.index', compact('products', 'categories'));
}
```

## Bagian yang tidak berubah

| Bagian | Alasan |
|---|---|
| `Product::with('category')` | Mencegah query kategori berulang saat view memakai `$product->category?->name`. |
| `search()` | Tetap mencari `name` dan `description`. |
| `filterByCategory()` | Tetap menyaring `category_id`. |
| `paginate(10)` | Tetap membagi hasil per halaman. |
| `withQueryString()` | Mempertahankan `search`, `category_id`, dan nantinya `sort` pada link halaman. |
| `$categories` | Tetap mengisi dropdown kategori. |

## Posisi sorting

Sorting ditambahkan setelah scope pencarian dan kategori, sebelum pagination:

```php
Product::with('category')
    ->search(...)
    ->filterByCategory(...)
    ->orderBy($column, $direction)
    ->orderBy('id')
    ->paginate(10)
    ->withQueryString();
```

`orderBy('id')` adalah pengurutan kedua dengan arah naik bawaan. Ini membuat pagination deterministik ketika nilai kolom utama sama.

## Route tetap manual

Tidak perlu mengganti route menjadi resource route. Route daftar tetap:

```php
Route::get('/products', [ProductController::class, 'index']);
```

Tahap berikutnya menerapkan kode lengkap controller.
