# Tahap 6 — Refactor Dashboard ke Layout

> Fokus: tampilan `dashboard.index` saja. Controller dashboard tahap 11 tidak ditulis ulang.

Dashboard memakai layout yang sama; tidak diperlukan layout kedua atau sidebar. Route dan controller yang diteruskan tetap tepat seperti berikut:

```php
Route::get('/dashboard', [DashboardController::class, 'index']);
```

Controller `DashboardController@index` sudah memiliki cache final berikut. Jangan menulis ulang atau mengubah controller ini saat refactor layout.

```php
Cache::remember('dashboard.products.summary', now()->addMinutes(5), function () {
    // satu payload: managedProductsCount, activeProductsCount,
    // inactiveProductsCount, trashedProductsCount, totalStock, latestProducts
});
```

Khususnya, `latestProducts` di dalam payload tetap berasal dari:

```php
Product::with('category')->latest('created_at')->orderByDesc('id')->take(5)->get()
```

Tidak ada metrik jumlah harga atau nilai penjualan pada dashboard ini.

## View yang direfactor

Bungkus metrik dan tabel yang sudah ada dengan layout. Nama variabel harus persis sama dengan payload controller.

```blade
{{-- resources/views/dashboard/index.blade.php --}}
@extends('layouts.app')

@section('content')
    <h1>{{ $title ?? 'Dashboard' }}</h1>

    <dl>
        <div><dt>Managed products</dt><dd>{{ $managedProductsCount }}</dd></div>
        <div><dt>Active products</dt><dd>{{ $activeProductsCount }}</dd></div>
        <div><dt>Inactive products</dt><dd>{{ $inactiveProductsCount }}</dd></div>
        <div><dt>Trashed products</dt><dd>{{ $trashedProductsCount }}</dd></div>
        <div><dt>Total stock</dt><dd>{{ $totalStock }}</dd></div>
    </dl>

    <h2>Latest products</h2>
    <table>
        <thead><tr><th>Name</th><th>Category</th><th>Stock</th><th>Status</th></tr></thead>
        <tbody>
            @forelse ($latestProducts as $product)
                <tr>
                    <td><a href="/products/{{ $product->slug }}">{{ $product->name }}</a></td>
                    <td>{{ $product->category?->name ?? '-' }}</td>
                    <td>{{ $product->stock }}</td>
                    <td>{{ $product->is_active ? 'Active' : 'Inactive' }}</td>
                </tr>
            @empty
                <tr><td colspan="4">No products yet.</td></tr>
            @endforelse
        </tbody>
    </table>
@endsection
```

## Batas refactor

Layout hanya mengambil kerangka HTML yang berulang. Ia tidak mengubah cache, query, metrik, atau controller. Invalidasi cache tetap hanya dilakukan setelah mutasi `Product` yang sudah ada berhasil, sesuai tahap 11. Pelajaran ini tidak membahas authorization atau middleware.
