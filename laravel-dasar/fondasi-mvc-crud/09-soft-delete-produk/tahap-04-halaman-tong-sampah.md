# Tahap 4 — Halaman Tong Sampah produk

Tong sampah adalah daftar pengelolaan terpisah dari index aktif. Ia sengaja diurutkan berdasarkan waktu penghapusan terbaru, bukan memakai kontrol pencarian, kategori, dan sorting dari `/products`.

## Tambahkan method `trash`

Di `ProductController`, tambahkan:

```php
public function trash()
{
    $trashedProducts = Product::onlyTrashed()
        ->with('category')
        ->orderByDesc('deleted_at')
        ->orderBy('id')
        ->paginate(10)
        ->withQueryString();

    return view('products.trash', compact('trashedProducts'));
}
```

`onlyTrashed()` hanya mengambil model yang telah di-soft-delete. `with('category')` menjaga akses `category` efisien. `withQueryString()` tetap berguna untuk mempertahankan parameter URL jika kelak ada parameter pada halaman ini.

## Urutan route lengkap

Tambahkan route secara manual di `routes/web.php`. Route `/products/trash` wajib berada sebelum catchall slug supaya kata `trash` tidak dianggap slug produk.

```php
use App\Http\Controllers\ProductController;
use Illuminate\Support\Facades\Route;

Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/create', [ProductController::class, 'create']);
Route::post('/products', [ProductController::class, 'store']);

Route::get('/products/trash', [ProductController::class, 'trash']);
Route::post('/products/{id}/restore', [ProductController::class, 'restore']);
Route::delete('/products/{id}/force-delete', [ProductController::class, 'forceDelete']);

Route::get('/products/{id}/edit', [ProductController::class, 'edit']);
Route::put('/products/{id}', [ProductController::class, 'update']);
Route::delete('/products/{id}', [ProductController::class, 'destroy']);

Route::get('/products/{slug}', [ProductController::class, 'show']);
```

Tidak menggunakan named route atau resource route agar konsisten dengan materi sebelumnya.

## Buat view `resources/views/products/trash.blade.php`

```blade
<h1>Tong Sampah produk</h1>
<a href="/products">Kembali ke daftar produk</a>

@if (session('success'))
    <p>{{ session('success') }}</p>
@endif

@forelse ($trashedProducts as $product)
    <article>
        <h2>{{ $product->name }}</h2>
        <p>Harga: {{ $product->price }}</p>
        <p>Stok: {{ $product->stock }}</p>
        <p>Kategori: {{ $product->category?->name ?? '-' }}</p>
        <p>Dihapus: {{ $product->deleted_at?->format('d M Y, H:i') ?? '-' }}</p>

        @if ($product->image)
            <img src="{{ asset('storage/' . $product->image) }}" alt="{{ $product->name }}">
        @endif

        <form action="/products/{{ $product->id }}/restore" method="POST">
            @csrf
            <button type="submit">Restore</button>
        </form>

        <form action="/products/{{ $product->id }}/force-delete" method="POST" onsubmit="return confirm('Hapus permanen produk ini?');">
            @csrf
            @method('DELETE')
            <button type="submit">Hapus permanen</button>
        </form>
    </article>
@empty
    <p>Tong sampah kosong.</p>
@endforelse

{{ $trashedProducts->links() }}
```

Konfirmasi HTML standar cukup untuk mengurangi salah klik. Keamanan utama force delete tetap berada di server melalui `onlyTrashed()`; authorization harus ditambahkan pada aplikasi nyata.

## Tautkan dari daftar aktif

Di dekat heading pada `resources/views/products/index.blade.php`, tambahkan link berikut tanpa mengubah form GET search/filter/sort yang sudah ada:

```blade
<h1>Daftar produk</h1>
<a href="/products/trash">Tong Sampah</a>
```

Pagination index aktif tetap hanya `{{ $products->links() }}`.
