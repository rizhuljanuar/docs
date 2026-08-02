# Tahap 6: Pencarian Multi-Field dan Filter Kategori

## Tujuan

Versi final mencari kata kunci pada `name` **atau** `description`, lalu dapat menyaring hasil berdasarkan `category_id`. Kategori berasal dari model `Category`, bukan daftar kategori yang ditulis manual di Blade.

## Model `Product`

Buka `app/Models/Product.php`. Pastikan relasi `category()` tetap ada dan gunakan local scope berikut:

```php
use Illuminate\Database\Eloquent\Builder;
```

```php
public function scopeSearch(Builder $query, mixed $keyword): Builder
{
    $keyword = trim((string) $keyword);

    if ($keyword === '') {
        return $query;
    }

    return $query->where(function (Builder $query) use ($keyword) {
        $query->where('name', 'like', '%' . $keyword . '%')
            ->orWhere('description', 'like', '%' . $keyword . '%');
    });
}

public function scopeFilterByCategory(Builder $query, mixed $categoryId): Builder
{
    if (blank($categoryId)) {
        return $query;
    }

    return $query->where('category_id', $categoryId);
}
```

Closure pada `scopeSearch()` menghasilkan logika berikut:

```sql
WHERE (name LIKE '%laptop%' OR description LIKE '%laptop%')
  AND category_id = 2
```

Tanda kurung itu penting. Tanpa closure, `orWhere()` dapat membuat hasil pencarian dari kategori lain ikut tampil.

## Controller Final

Buka `app/Http/Controllers/ProductController.php`. Pastikan import berikut ada:

```php
use App\Models\Category;
use App\Models\Product;
use Illuminate\Http\Request;
```

Ubah method `index()` menjadi:

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

`with('category')` menghindari query tambahan saat Blade menampilkan kategori setiap produk. `$categories` dikirim agar dropdown selalu membaca kategori yang ada di database.

## Form Final pada `products/index.blade.php`

Ganti form pencarian dari Tahap 4 dengan:

```blade
<form action="/products" method="GET">
    <input
        type="text"
        name="search"
        value="{{ request('search') }}"
        maxlength="100"
        placeholder="Cari nama atau deskripsi produk..."
    >

    <select name="category_id">
        <option value="">Semua kategori</option>
        @foreach ($categories as $category)
            <option
                value="{{ $category->id }}"
                @selected((string) request('category_id') === (string) $category->id)
            >
                {{ $category->name }}
            </option>
        @endforeach
    </select>

    <button type="submit">Cari</button>
    <a href="/products">Reset</a>
</form>
```

Form ini GET, jadi tidak memakai `@csrf`. Pilihan kategori dipertahankan dengan `@selected(...)` setelah pengguna mengirim form.

## Uji Kombinasi

```text
/products
/products?search=laptop
/products?category_id=2
/products?search=gaming&category_id=2
```

Pastikan halaman berikutnya masih membawa `search` dan `category_id`; hal itu dikerjakan oleh `withQueryString()`.

## Inti Tahap 6

> Versi final memakai `scopeSearch()` untuk `name` dan `description`, `scopeFilterByCategory()` untuk `category_id`, kategori dinamis dari `$categories`, eager loading relasi `category`, serta pagination yang mempertahankan query string.
