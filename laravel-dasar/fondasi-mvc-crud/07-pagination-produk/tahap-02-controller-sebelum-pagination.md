# Tahap 2 — Periksa Controller yang Sudah Final

## Tujuan

Sebelum membuat tampilan pagination, pastikan `ProductController@index` tetap meneruskan kontrak dari materi pencarian: validasi input, eager loading relasi, pencarian, filter kategori, pagination, lalu query string.

## Route daftar produk

Route daftar tetap dideklarasikan secara manual di `routes/web.php`:

```php
use App\Http\Controllers\ProductController;
use Illuminate\Support\Facades\Route;

Route::get('/products', [ProductController::class, 'index']);
```

## Controller

```php
use App\Models\Category;
use App\Models\Product;
use Illuminate\Http\Request;

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

## Urutan query penting

Urutan di atas membuat `$products` hanya memuat hasil yang cocok dengan `search` dan `category_id`. `with('category')` menyiapkan relasi kategori agar Blade dapat membaca `$product->category?->name` tanpa query tambahan untuk setiap produk.

Jangan menghapus scope pencarian atau filter ketika membahas pagination. Pagination adalah tahap terakhir dari query yang sama.
