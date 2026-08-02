# Tahap 5 — Ringkasan Pagination dan Penyesuaian Per Halaman

## Ringkasan yang aman saat hasil kosong

`firstItem()` dan `lastItem()` bernilai `null` bila tidak ada hasil. Karena itu, tampilkan keduanya hanya ketika total lebih dari nol.

```blade
@if ($products->total() > 0)
    <p>
        Menampilkan produk {{ $products->firstItem() }}–{{ $products->lastItem() }}
        dari {{ $products->total() }} produk.
    </p>
@else
    <p>Tidak ada produk untuk ditampilkan.</p>
@endif
```

Pesan `@empty` pada daftar produk tetap diperlukan agar area daftar juga menjelaskan bahwa hasil kosong.

## Opsional: jumlah per halaman

Jika kebutuhan tampilan berubah, ganti angka `10` pada query yang sama. Tetap pertahankan pencarian, filter, relasi, dan query string.

```php
$products = Product::with('category')
    ->search($validated['search'] ?? '')
    ->filterByCategory($validated['category_id'] ?? null)
    ->paginate(20)
    ->withQueryString();
```

Contoh tersebut menampilkan maksimal 20 hasil per halaman. Pilih angka sesuai kepadatan tampilan, tetapi jangan menghilangkan `withQueryString()`.

## Checklist

- [ ] Route daftar adalah `Route::get('/products', [ProductController::class, 'index']);`.
- [ ] Controller memvalidasi `search` dan `category_id`.
- [ ] Query memakai `Product::with('category')->search()->filterByCategory()->paginate(...)->withQueryString()`.
- [ ] Form GET mengulang `$categories`, mempertahankan nilai request, dan tidak memakai CSRF.
- [ ] Daftar memakai `$products` dan `@forelse`.
- [ ] Kategori ditampilkan melalui `$product->category?->name`.
- [ ] Detail memakai slug; edit dan hapus memakai ID.
- [ ] Form hapus memakai `@csrf` dan `@method('DELETE')`.
- [ ] Link halaman dari `{{ $products->links() }}` mempertahankan pencarian dan filter.

## Kesimpulan

Pagination pada halaman produk sudah terintegrasi dengan fitur pencarian dan filter dari materi sebelumnya. Laravel menghitung total hasil, mengambil data untuk halaman yang diminta, lalu `links()` membuat navigasi menggunakan paginator yang sama.
