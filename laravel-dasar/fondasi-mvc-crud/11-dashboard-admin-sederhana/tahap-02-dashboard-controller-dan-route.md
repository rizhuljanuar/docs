# Tahap 2 — DashboardController dan Route

> Fokus: membuat kerangka dashboard product

## File yang digunakan

Dashboard memakai controller biasa, bukan namespace tambahan:

```text
app/Http/Controllers/DashboardController.php
resources/views/dashboard/index.blade.php
routes/web.php
```

Buat controller dari root project:

```bash
php artisan make:controller DashboardController
```

## Controller dasar

Isi `app/Http/Controllers/DashboardController.php` pada tahap ini:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;

class DashboardController extends Controller
{
    public function index()
    {
        $managedProductsCount = Product::count();

        return view('dashboard.index', compact('managedProductsCount'));
    }
}
```

`Product::count()` adalah query biasa sehingga hanya menghitung product non-trash. Untuk sementara kita kirim satu nilai ini ke view; metrik lain ditambahkan bertahap pada tahap berikutnya.

## Route literal

Tambahkan import dan route berikut di `routes/web.php` bersama route yang sudah ada:

```php
use App\Http\Controllers\DashboardController;

Route::get('/dashboard', [DashboardController::class, 'index']);
```

Route ini sengaja memakai URL literal `/dashboard` dan tidak memakai nama route. CRUD lama tetap menggunakan keluarga URL literal `/products`; jangan memindahkan atau mengubah query index CRUD tersebut.

## View dasar

Buat `resources/views/dashboard/index.blade.php`:

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Dashboard Product</title>
</head>
<body>
    <h1>Dashboard Product</h1>
    <p>Total product non-trash: {{ $managedProductsCount }}</p>
</body>
</html>
```

Buka `http://localhost:8000/dashboard`. Bila halaman muncul, route memanggil `DashboardController@index` dan view telah ditemukan.

## Catatan keamanan

Route ini hanya kerangka materi. Sebelum dipakai di aplikasi nyata, tambahkan authorization dan middleware yang tepat. Seri ini tidak mengimplementasikannya.
