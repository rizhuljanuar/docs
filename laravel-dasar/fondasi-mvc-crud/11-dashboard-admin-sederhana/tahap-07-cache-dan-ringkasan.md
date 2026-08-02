# Tahap 7 — Cache Koheren dan Ringkasan

> Fokus: satu snapshot dashboard product selama lima menit

## Mengapa satu payload cache?

Jika setiap kartu memakai key cache sendiri, angka pada dashboard dapat berasal dari waktu yang berbeda. Simpan seluruh ringkasan dalam satu payload agar kartu dan daftar terbaru berasal dari snapshot yang sama.

Berikut kode final `app/Http/Controllers/DashboardController.php`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Support\Facades\Cache;

class DashboardController extends Controller
{
    public function index()
    {
        $summary = Cache::remember('dashboard.products.summary', now()->addMinutes(5), function () {
            return [
                'managedProductsCount' => Product::count(),
                'activeProductsCount' => Product::where('is_active', true)->count(),
                'inactiveProductsCount' => Product::where('is_active', false)->count(),
                'trashedProductsCount' => Product::onlyTrashed()->count(),
                'totalStock' => Product::sum('stock'),
                'latestProducts' => Product::with('category')
                    ->latest('created_at')
                    ->orderByDesc('id')
                    ->take(5)
                    ->get(),
            ];
        });

        return view('dashboard.index', $summary);
    }
}
```

`Cache::remember()` membaca key yang ada. Jika belum ada atau TTL lima menit telah habis, closure dijalankan dan hasilnya disimpan. Semua query product reguler di closure mengecualikan soft delete; hanya `Product::onlyTrashed()` yang menghitung trash. Query terbaru juga berada di closure yang sama.

## Menghapus cache setelah perubahan product

Cache dapat menjadi lama sebelum TTL berakhir. Setelah mutasi product **berhasil**, hapus satu key yang dipakai dashboard:

```php
Cache::forget('dashboard.products.summary');
```

Tambahkan import berikut ke `ProductController` bila belum ada:

```php
use Illuminate\Support\Facades\Cache;
```

Panggil `Cache::forget('dashboard.products.summary')` hanya setelah operasi berikut sukses:

1. `store` setelah product berhasil dibuat;
2. `update` setelah product berhasil diperbarui;
3. `updateStatus` setelah status publikasi berhasil diperbarui;
4. `destroy` setelah soft delete berhasil;
5. `restore` setelah product berhasil dipulihkan;
6. `forceDelete` setelah product berhasil dihapus permanen.

Contoh pola untuk `updateStatus`:

```php
$product->update($validated);
Cache::forget('dashboard.products.summary');

return redirect('/products');
```

Jangan menghapus cache sebelum mutation berhasil, karena kegagalan validasi atau database tidak mengubah ringkasan. `Cache::forget()` menerima satu key; jangan memberikan array. Jangan menghapus semua cache aplikasi untuk kebutuhan dashboard ini.

Bila lokasi invalidasi semakin banyak dan mulai berulang, observer atau event dapat menjadi pilihan untuk memusatkan invalidasi. Materi ini tidak mengimplementasikannya karena enam lokasi mutation yang sudah ada masih jelas untuk dipelajari.

## Checklist akhir

- Route dashboard adalah `GET /dashboard` dengan `DashboardController` biasa dan tanpa nama route.
- View berada di `resources/views/dashboard/index.blade.php`.
- Total product non-trash mencakup aktif dan tidak aktif; trash dihitung terpisah.
- Total stok diberi label product non-trash.
- Product terbaru berjumlah maksimal lima, diurutkan `created_at` lalu `id`, dan memuat category.
- Navigasi dashboard memakai URL literal `/products` dan `/products/trash`; detail memakai slug.
- Dashboard ini belum memiliki authorization atau middleware untuk penggunaan nyata. Tambahkan keduanya sesuai kebutuhan aplikasi sebelum membuka aksesnya.
