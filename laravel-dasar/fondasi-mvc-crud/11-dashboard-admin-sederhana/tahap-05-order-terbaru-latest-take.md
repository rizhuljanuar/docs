# Tahap 5 — Product Terbaru dengan `latest()` dan `take()`

> Fokus: lima product non-trash terbaru

## Query daftar terbaru

Tambahkan query ini di controller:

```php
$latestProducts = Product::with('category')
    ->latest('created_at')
    ->orderByDesc('id')
    ->take(5)
    ->get();
```

Penjelasannya:

- `with('category')` melakukan eager loading relasi category agar view tidak menjalankan query tambahan untuk setiap baris;
- `latest('created_at')` mengurutkan `created_at` dari terbaru;
- `orderByDesc('id')` menjadi penentu yang konsisten bila beberapa product memiliki waktu dibuat sama;
- `take(5)` membatasi hasil menjadi lima;
- `get()` menjalankan query dan menghasilkan collection.

Seperti query `Product` biasa lain, global scope `SoftDeletes` berarti daftar ini tidak memuat product dalam trash.

## Kode sementara lengkap

```php
public function index()
{
    $managedProductsCount = Product::count();
    $activeProductsCount = Product::where('is_active', true)->count();
    $inactiveProductsCount = Product::where('is_active', false)->count();
    $trashedProductsCount = Product::onlyTrashed()->count();
    $totalStock = Product::sum('stock');

    $latestProducts = Product::with('category')
        ->latest('created_at')
        ->orderByDesc('id')
        ->take(5)
        ->get();

    return view('dashboard.index', compact(
        'managedProductsCount',
        'activeProductsCount',
        'inactiveProductsCount',
        'trashedProductsCount',
        'totalStock',
        'latestProducts',
    ));
}
```

Pada tahap berikutnya nilai-nilai ini ditampilkan dalam kartu dan tabel Blade. Tahap cache sesudahnya akan menjadi kode final: seluruh query di atas berjalan dalam satu payload cache.
