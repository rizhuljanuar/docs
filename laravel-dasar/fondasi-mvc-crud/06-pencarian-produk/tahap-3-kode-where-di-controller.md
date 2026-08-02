# Tahap 3: Query Sederhana di Controller

## Tujuan

Pada tahap ini kita membuat pencarian paling sederhana: hanya berdasarkan `name`, langsung di `ProductController@index`. Form belum dibuat; pengujian dilakukan melalui URL.

## Ubah Method `index()`

Buka `app/Http/Controllers/ProductController.php`. Pastikan import ini tersedia:

```php
use App\Models\Product;
use Illuminate\Http\Request;
```

Ubah method `index()` menjadi:

```php
public function index(Request $request)
{
    $validated = $request->validate([
        'search' => ['nullable', 'string', 'max:100'],
    ]);

    $keyword = $validated['search'] ?? '';

    $products = Product::with('category')
        ->when($keyword !== '', function ($query) use ($keyword) {
            $query->where('name', 'like', '%' . $keyword . '%');
        })
        ->paginate(10)
        ->withQueryString();

    return view('products.index', compact('products'));
}
```

## Penjelasan

- Validasi `nullable|string|max:100` membuat pencarian opsional dan membatasi panjang kata kunci dengan aman.
- `when()` hanya menambah `where` jika pengguna benar-benar mengirim kata kunci.
- `with('category')` mengambil relasi kategori bersama daftar produk sehingga view tidak memicu query kategori tambahan per baris.
- `paginate(10)->withQueryString()` menampilkan sepuluh produk per halaman dan mempertahankan parameter `search` saat pindah halaman.

## Uji Melalui URL

Dengan route daftar dari materi sebelumnya:

```php
Route::get('/products', [ProductController::class, 'index']);
```

uji URL berikut:

```text
/products
/products?search=laptop
/products?search=sepatu
```

URL tanpa `search` menampilkan semua produk. URL dengan `search=laptop` hanya menampilkan produk yang `name`-nya memuat `laptop`.

## Inti Tahap 3

> Ini adalah versi awal yang sengaja sederhana: query `name` berada langsung di controller. Tahap berikutnya menambahkan form GET pada `products/index.blade.php`.
