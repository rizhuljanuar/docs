# Tahap 3 — Aggregation `count()` dan Soft Delete

> Fokus: menghitung keadaan product dengan benar

## Empat angka yang tidak boleh tertukar

Dashboard membutuhkan empat hitungan berikut:

| Variable | Query | Arti |
|---|---|---|
| `$managedProductsCount` | `Product::count()` | Semua product non-trash: aktif dan tidak aktif |
| `$activeProductsCount` | `where('is_active', true)` | Product non-trash yang aktif |
| `$inactiveProductsCount` | `where('is_active', false)` | Product non-trash yang tidak aktif |
| `$trashedProductsCount` | `onlyTrashed()` | Product dalam trash |

`SoftDeletes` membuat tiga query pertama otomatis mengabaikan baris yang memiliki `deleted_at`. Sebaliknya, `onlyTrashed()` khusus mengambil baris yang sudah di-soft delete.

## Kode controller tahap ini

Perbarui method `index()` di `app/Http/Controllers/DashboardController.php` menjadi:

```php
public function index()
{
    $managedProductsCount = Product::count();
    $activeProductsCount = Product::where('is_active', true)->count();
    $inactiveProductsCount = Product::where('is_active', false)->count();
    $trashedProductsCount = Product::onlyTrashed()->count();

    return view('dashboard.index', compact(
        'managedProductsCount',
        'activeProductsCount',
        'inactiveProductsCount',
        'trashedProductsCount',
    ));
}
```

## Pemeriksaan angka

Untuk data non-trash, berlaku:

```text
$managedProductsCount = $activeProductsCount + $inactiveProductsCount
```

Nilai `$trashedProductsCount` tidak ditambahkan ke rumus itu. Product yang masuk trash bukan product yang sedang dikelola pada index `/products`.

Tahap akhir akan menyatukan query ini dalam satu payload cache agar seluruh kartu berasal dari keadaan data yang sama.
