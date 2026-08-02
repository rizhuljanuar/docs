# Tahap 4 — Refactor Daftar Products

> Fokus: hanya tampilan `products/index.blade.php`; perilaku tahap 06–08 tetap utuh.

## Kontrak halaman daftar

Daftar sudah menerima `$products` sebagai paginator. Ia sudah mendukung pencarian, filter `Category`, pengurutan, dan pagination. Layout tidak memberi alasan untuk mengganti query tersebut dengan `Product::all()` atau daftar buatan. Pertahankan form, parameter, tabel hasil, dan `{{ $products->links() }}` yang sudah selesai pada pelajaran sebelumnya.

## Bentuk view setelah refactor

Pindahkan hanya kerangka HTML bersama ke layout. Isi operasional daftar tetap berada di section berikut.

```blade
{{-- resources/views/products/index.blade.php --}}
@extends('layouts.app')

@section('content')
    <h1>{{ $title ?? 'Products' }}</h1>

    {{-- Pertahankan form pencarian, filter Category, dan pilihan sort dari tahap 06–08. --}}
    <form method="GET" action="/products">
        <input name="search" value="{{ request('search') }}" placeholder="Search products">

        <select name="category_id">
            <option value="">All categories</option>
            @foreach ($categories as $category)
                <option value="{{ $category->id }}" @selected((string) request('category_id') === (string) $category->id)>
                    {{ $category->name }}
                </option>
            @endforeach
        </select>

        <select name="sort">
            <option value="newest" @selected(request('sort', 'newest') === 'newest')>Newest</option>
            <option value="oldest" @selected(request('sort') === 'oldest')>Oldest</option>
            <option value="price-asc" @selected(request('sort') === 'price-asc')>Price: low to high</option>
            <option value="price-desc" @selected(request('sort') === 'price-desc')>Price: high to low</option>
            <option value="name-asc" @selected(request('sort') === 'name-asc')>Name: A to Z</option>
            <option value="name-desc" @selected(request('sort') === 'name-desc')>Name: Z to A</option>
        </select>

        <button type="submit">Apply</button>
    </form>

    <p><a href="/products/create">Add product</a></p>

    <table>
        <thead>
            <tr>
                <th>Name</th>
                <th>Category</th>
                <th>Price</th>
                <th>Stock</th>
                <th>Status</th>
                <th>Action</th>
            </tr>
        </thead>
        <tbody>
            @forelse ($products as $product)
                <tr>
                    <td><a href="/products/{{ $product->slug }}">{{ $product->name }}</a></td>
                    <td>{{ $product->category?->name ?? '-' }}</td>
                    <td>{{ $product->price }}</td>
                    <td>{{ $product->stock }}</td>
                    <td>{{ $product->is_active ? 'Active' : 'Inactive' }}</td>
                    <td>
                        <a href="/products/{{ $product->id }}/edit">Edit</a>
                        <form method="POST" action="/products/{{ $product->id }}">
                            @csrf
                            @method('DELETE')
                            <button type="submit">Delete</button>
                        </form>
                    </td>
                </tr>
            @empty
                <tr><td colspan="6">No products found.</td></tr>
            @endforelse
        </tbody>
    </table>

    {{ $products->links() }}
@endsection
```

Contoh ini menunjukkan titik integrasi layout, bukan pengganti form final. Salin kembali seluruh pilihan filter dan sort yang sudah berfungsi dari view tahap 06–08 ke dalam form tersebut. Parameter pagination juga harus terus mempertahankan pencarian, filter, dan sort seperti sebelumnya.

## Yang tidak berubah

- Endpoint daftar tetap `/products`.
- Detail memakai slug: `/products/{{ $product->slug }}`.
- Edit memakai ID: `/products/{{ $product->id }}/edit`.
- Penghapusan memakai `DELETE` ke `/products/{{ $product->id }}`.
- `$products` tetap paginator dari controller yang telah ada.

Kartu komponen pada tahap 8 boleh menjadi presentasi tambahan, tetapi tabel paginator ini tetap presentasi utama agar fungsi final tidak hilang.
