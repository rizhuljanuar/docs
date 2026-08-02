# Tahap 6 — View Blade Dashboard Product

> Fokus: kartu ringkasan dan tabel product terbaru

Buat atau perbarui `resources/views/dashboard/index.blade.php` dengan view berikut. View ini menerima seluruh variable dari controller, termasuk `$latestProducts` yang sudah eager load relasi `category`.

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Dashboard Product</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 2rem; color: #1f2937; }
        nav { margin-bottom: 2rem; }
        nav a { margin-right: 1rem; }
        .cards { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 1rem; margin-bottom: 2rem; }
        .card { border: 1px solid #d1d5db; border-radius: .5rem; padding: 1rem; }
        .label { color: #4b5563; margin: 0; }
        .value { font-size: 1.5rem; font-weight: bold; margin: .5rem 0 0; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #d1d5db; padding: .75rem; text-align: left; }
        .badge { border-radius: 999px; padding: .2rem .55rem; font-size: .875rem; }
        .active { background: #dcfce7; color: #166534; }
        .inactive { background: #fee2e2; color: #991b1b; }
    </style>
</head>
<body>
    <nav>
        <a href="/products">Kelola product</a>
        <a href="/products/trash">Trash product</a>
    </nav>

    <h1>Dashboard Product</h1>

    <section class="cards" aria-label="Ringkasan product">
        <article class="card">
            <p class="label">Total product non-trash</p>
            <p class="value">{{ $managedProductsCount }}</p>
        </article>
        <article class="card">
            <p class="label">Product aktif</p>
            <p class="value">{{ $activeProductsCount }}</p>
        </article>
        <article class="card">
            <p class="label">Product tidak aktif</p>
            <p class="value">{{ $inactiveProductsCount }}</p>
        </article>
        <article class="card">
            <p class="label">Product di trash</p>
            <p class="value">{{ $trashedProductsCount }}</p>
        </article>
        <article class="card">
            <p class="label">Total stok product non-trash</p>
            <p class="value">{{ $totalStock }}</p>
        </article>
    </section>

    <h2>Lima product terbaru</h2>
    <table>
        <thead>
            <tr>
                <th>Nama</th>
                <th>Harga</th>
                <th>Stok</th>
                <th>Category</th>
                <th>Status</th>
            </tr>
        </thead>
        <tbody>
            @forelse ($latestProducts as $product)
                <tr>
                    <td><a href="/products/{{ $product->slug }}">{{ $product->name }}</a></td>
                    <td>Rp {{ number_format($product->price, 0, ',', '.') }}</td>
                    <td>{{ $product->stock }}</td>
                    <td>{{ $product->category?->name ?? '-' }}</td>
                    <td>
                        @if ($product->is_active)
                            <span class="badge active">Aktif</span>
                        @else
                            <span class="badge inactive">Tidak aktif</span>
                        @endif
                    </td>
                </tr>
            @empty
                <tr>
                    <td colspan="5">Belum ada product non-trash.</td>
                </tr>
            @endforelse
        </tbody>
    </table>
</body>
</html>
```

## Mengapa memakai `@forelse`?

`@forelse` menampilkan baris table untuk setiap product. Bila collection kosong, bagian `@empty` memberikan pesan yang jelas tanpa membuat tabel kosong.

Link detail memakai slug karena route detail product yang sudah ada memakai `/products/{slug}`. Sementara tindakan perubahan tetap memakai ID di CRUD lama. Tidak ada route helper pada view ini.

Kode controller final dengan cache ada di tahap berikutnya.
