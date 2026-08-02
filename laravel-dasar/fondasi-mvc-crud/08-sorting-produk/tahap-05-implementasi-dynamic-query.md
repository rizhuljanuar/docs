# Tahap 5 — Implementasi Sorting di `ProductController`

Gunakan method `index` berikut. Kode ini mempertahankan seluruh fitur dari materi pencarian dan pagination, lalu menambahkan sorting yang tervalidasi.

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
        'sort' => ['nullable', 'string', 'in:'.implode(',', array_keys($allowedSorts))],
    ]);

    $sort = $validated['sort'] ?? 'newest';
    [$column, $direction] = $allowedSorts[$sort];

    $products = Product::with('category')
        ->search($validated['search'] ?? '')
        ->filterByCategory($validated['category_id'] ?? null)
        ->orderBy($column, $direction)
        ->orderBy('id')
        ->paginate(10)
        ->withQueryString();

    $categories = Category::orderBy('name')->get();

    return view('products.index', compact('products', 'categories'));
}
```

Pastikan controller tetap memiliki import yang telah dipakai oleh materi sebelumnya:

```php
use App\Models\Category;
use App\Models\Product;
use Illuminate\Http\Request;
```

## Mengapa urutannya seperti ini?

1. `$allowedSorts` dibuat terlebih dahulu karena dipakai oleh aturan validasi.
2. `validate()` menghasilkan input tepercaya untuk `search`, `category_id`, dan `sort`.
3. Jika `sort` tidak ada, `$sort` menjadi `newest`.
4. Destructuring mengambil `$column` dan `$direction` dari map.
5. Query menjalankan eager loading, pencarian, filter, pengurutan utama, pengurutan ID, lalu pagination.

Urutan kedua `->orderBy('id')` penting saat harga atau nama sama. Dengan ID naik, produk yang sama tidak berpindah-pindah antarhalaman hanya karena urutan tie tidak pasti.

## Uji melalui URL

| URL | Hasil |
|---|---|
| `/products` | Produk terbaru sebagai default. |
| `/products?sort=oldest` | Produk terlama dahulu. |
| `/products?sort=price-asc` | Harga rendah ke tinggi. |
| `/products?sort=price-desc` | Harga tinggi ke rendah. |
| `/products?sort=name-asc` | Nama A–Z. |
| `/products?sort=name-desc` | Nama Z–A. |
| `/products?search=laptop&category_id=2&sort=price-asc&page=2` | Kombinasi pencarian, kategori, sorting, dan halaman kedua. |

Untuk `sort` eksplisit yang tidak terdaftar, Laravel mengembalikan respons validasi. Ini lebih jelas daripada membiarkan kolom tak dikenal masuk ke query.

Tahap terakhir menambahkan select sorting pada form GET yang sudah ada.
