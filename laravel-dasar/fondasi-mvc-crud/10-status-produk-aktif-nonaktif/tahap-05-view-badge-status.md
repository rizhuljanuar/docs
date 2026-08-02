# Tahap 5 — Badge Status pada Index Product

Ubah view yang sudah ada, `resources/views/products/index.blade.php`. Jangan membuat view admin baru. Form pencarian/filter/sorting tetap `GET`, tetap tanpa `@csrf`, dan pagination tetap memakai `{{ $products->links() }}` karena controller sudah memakai `withQueryString()`.

Tambahkan kolom status pada tabel/list produk dan tampilkan badge berdasarkan cast boolean:

```blade
<th>Status</th>
```

```blade
<td>
    @if ($product->is_active)
        <span>Aktif</span>
    @else
        <span>Nonaktif</span>
    @endif
</td>
```

Gunakan data produk yang sudah menjadi kontrak saat mempertahankan loop yang ada:

```blade
@forelse ($products as $product)
    <article>
        <h2>{{ $product->name }}</h2>
        <p>Harga: {{ number_format($product->price, 0, ',', '.') }}</p>
        <p>Stok: {{ $product->stock }}</p>
        <p>Kategori: {{ $product->category?->name }}</p>

        @if ($product->is_active)
            <span>Aktif</span>
        @else
            <span>Nonaktif</span>
        @endif

        <a href="/products/{{ $product->slug }}">Detail</a>
        <a href="/products/{{ $product->id }}/edit">Edit</a>

        <form action="/products/{{ $product->id }}" method="POST">
            @csrf
            @method('DELETE')
            <button type="submit">Hapus</button>
        </form>
    </article>
@empty
    <p>Data tidak ditemukan.</p>
@endforelse

{{ $products->links() }}
```

Detail selalu memakai slug. Edit dan hapus memakai ID. Jangan mengubah query/view ini menjadi daftar public-only: `/products` adalah daftar manajemen yang mempertahankan produk aktif dan nonaktif, tetapi bukan trashed products.

Pada tahap berikutnya, form status diletakkan berdampingan dengan kontrol mutasi yang sudah ada. Tombol trash/restore/force delete tetap berada pada lifecycle terpisah: view tong sampah tidak memperoleh badge atau kontrol status.
