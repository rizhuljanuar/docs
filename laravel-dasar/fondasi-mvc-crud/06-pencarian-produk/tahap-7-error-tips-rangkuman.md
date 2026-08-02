# Tahap 7: Pengujian, Error Umum, dan Rangkuman

## Kode Final

### `app/Models/Product.php`

Tambahkan import dan scope ini sambil mempertahankan `$fillable` berisi `name`, `price`, `stock`, `description`, `category_id`, `image`, dan `slug`, serta relasi `category()` yang sudah dibuat pada materi kategori.

```php
use Illuminate\Database\Eloquent\Builder;

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

### `ProductController@index`

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

### Form pada `resources/views/products/index.blade.php`

```blade
<form action="/products" method="GET">
    <input type="text" name="search" value="{{ request('search') }}" maxlength="100">

    <select name="category_id">
        <option value="">Semua kategori</option>
        @foreach ($categories as $category)
            <option value="{{ $category->id }}"
                @selected((string) request('category_id') === (string) $category->id)>
                {{ $category->name }}
            </option>
        @endforeach
    </select>

    <button type="submit">Cari</button>
    <a href="/products">Reset</a>
</form>

{{ $products->links() }}
```

## Checklist Pengujian

- [ ] `/products` menampilkan seluruh produk dengan pagination.
- [ ] `/products?search=laptop` mencari pada `name` dan `description`.
- [ ] `/products?category_id=2` hanya menampilkan kategori ID `2`.
- [ ] Kombinasi `search` dan `category_id` memakai logika `(name OR description) AND category_id`.
- [ ] Dropdown kategori berisi data dari `$categories` dan mempertahankan pilihan.
- [ ] Pindah halaman mempertahankan parameter pencarian dan kategori.
- [ ] Tautan detail tetap `/products/{{ $product->slug }}`.
- [ ] Edit dan hapus tetap memakai `$product->id`.

## Error Umum

### Hasil kategori tidak sesuai

Pastikan field form bernama `category_id`, validasi memakai `exists:categories,id`, dan scope memakai `where('category_id', $categoryId)`. Jangan mengganti relasi kategori dengan kolom teks `category`.

### Produk kategori lain ikut muncul

Periksa bahwa `orWhere('description', ...)` berada dalam closure `where(function (...) {})`. Closure menjaga prioritas logika query.

### Kotak pencarian atau pilihan kategori hilang

Gunakan `value="{{ request('search') }}"` dan `@selected(...)` pada option. Pastikan pagination memakai `->withQueryString()`.

### Hasil terlalu banyak atau lambat

Tetap gunakan `->paginate(10)->withQueryString()`, bukan `->get()`. Validasi juga sudah membatasi kata kunci sampai 100 karakter. Untuk `%kata%`, indeks B-tree biasa tidak mempercepat pencarian mengandung teks; ukur kebutuhan sebelum memilih solusi pencarian lain.

### Form GET memiliki token CSRF

Hapus `@csrf` dari form pencarian/filter GET. Form POST untuk hapus tetap membutuhkan `@csrf`.

## Rangkuman

1. Tahap 3 membuat pencarian `name` sederhana di controller.
2. Tahap 4 menambahkan form GET di view daftar produk.
3. Tahap 5 memindahkan query awal ke local scope `scopeSearch()` dan menangani input kosong.
4. Tahap 6 menyelesaikan versi multi-field dan filter kategori dinamis.

Kontrak aplikasi tetap konsisten: `Product`, `ProductController`, tabel `products`, `$products`, `$categories`, dan route produk yang dideklarasikan satu per satu. Detail memakai slug, sedangkan edit dan hapus memakai ID.
