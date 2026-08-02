# Tahap 4 — Tampilkan Produk dan Link Pagination di Blade

## Lokasi view

Gunakan view daftar yang sudah dipakai aplikasi:

```text
resources/views/products/index.blade.php
```

## Form pencarian dan filter

Form daftar memakai GET agar parameter terlihat di URL. Karena bukan perubahan data, form ini tidak memakai token CSRF.

```blade
<form action="/products" method="GET">
    <input type="text" name="search" value="{{ request('search') }}" maxlength="100">

    <select name="category_id">
        <option value="">Semua kategori</option>
        @foreach ($categories as $category)
            <option value="{{ $category->id }}"
                @selected((string) request('category_id') === (string) $category->id)>
                {{ $category->name }}
            </option>
        @endforeach
    </select>

    <button type="submit">Cari</button>
    <a href="/products">Reset</a>
</form>
```

## Daftar produk, keadaan kosong, dan link

Gunakan `@forelse` supaya pengguna mendapat pesan jika pencarian tidak menemukan hasil.

```blade
@forelse ($products as $product)
    <article>
        <h2>{{ $product->name }}</h2>

        @if ($product->image)
            <img src="{{ asset('storage/' . $product->image) }}" alt="{{ $product->name }}" width="120">
        @endif

        <p>Harga: Rp {{ $product->price }}</p>
        <p>Stok: {{ $product->stock }}</p>
        <p>Kategori: {{ $product->category?->name ?? 'Tanpa kategori' }}</p>
        <p>{{ $product->description }}</p>

        <a href="/products/{{ $product->slug }}">Lihat detail</a>
        <a href="/products/{{ $product->id }}/edit">Edit</a>

        <form action="/products/{{ $product->id }}" method="POST">
            @csrf
            @method('DELETE')
            <button type="submit">Hapus</button>
        </form>
    </article>
@empty
    <p>Tidak ada produk yang cocok.</p>
@endforelse

{{ $products->links() }}
```

`links()` memakai view pagination default yang sudah dikonfigurasi aplikasi. Tidak perlu menambah dependensi UI. Jika aplikasi memang memakai Bootstrap, pengaturan Bootstrap dapat dipakai sesuai konfigurasi aplikasi; jika tidak, biarkan `links()` memakai default.
