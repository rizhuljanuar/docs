# Tahap 4 — `sum('stock')` dan Filter Boolean

> Fokus: total stok product non-trash

## Menjumlahkan stok

Kolom `stock` menyimpan jumlah unit product. Query berikut menjumlahkan stok seluruh product non-trash:

```php
$totalStock = Product::sum('stock');
```

Label yang tepat untuk nilai ini adalah **total stok product non-trash**. Product di trash tidak ikut karena `Product` memakai `SoftDeletes`.

## Boolean `is_active`

Gunakan boolean PHP saat menyaring status publikasi:

```php
$activeProductsCount = Product::where('is_active', true)->count();
$inactiveProductsCount = Product::where('is_active', false)->count();
```

Database dapat menyimpan boolean dengan representasi internalnya sendiri, tetapi pada kode aplikasi kita menyatakan maksudnya dengan `true` dan `false`. Status ini tetap berbeda dari `deleted_at`.

## Versi sementara controller

Berikut gabungan metrik sampai tahap ini:

```php
public function index()
{
    $managedProductsCount = Product::count();
    $activeProductsCount = Product::where('is_active', true)->count();
    $inactiveProductsCount = Product::where('is_active', false)->count();
    $trashedProductsCount = Product::onlyTrashed()->count();
    $totalStock = Product::sum('stock');

    return view('dashboard.index', compact(
        'managedProductsCount',
        'activeProductsCount',
        'inactiveProductsCount',
        'trashedProductsCount',
        'totalStock',
    ));
}
```

Jangan menjumlahkan stok dari `Product::onlyTrashed()` kecuali kebutuhan bisnis memang menyebut stok trash. Dashboard ini secara sengaja hanya menampilkan stok product non-trash.

Tahap selanjutnya menambahkan daftar lima product terbaru beserta category-nya.
