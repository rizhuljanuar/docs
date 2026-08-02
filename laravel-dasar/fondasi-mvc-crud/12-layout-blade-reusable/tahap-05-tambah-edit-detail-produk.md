# Tahap 5 — Refactor Create, Edit, dan Detail Product

> Fokus: membungkus view yang ada dengan layout tanpa mengubah kontrak form atau detail.

Semua view berikut memakai awal yang sama:

```blade
@extends('layouts.app')

@section('content')
    {{-- isi khusus halaman --}}
@endsection
```

## Create

Form create tetap mengirim ke endpoint literal `/products` dan hanya memakai kolom `Product` yang telah dipelajari.

```blade
@extends('layouts.app')

@section('content')
    <h1>{{ $title ?? 'Add product' }}</h1>

    <form method="POST" action="/products" enctype="multipart/form-data">
        @csrf
        <label>Name <input name="name" value="{{ old('name') }}"></label>
        <label>Price <input type="number" name="price" value="{{ old('price') }}"></label>
        <label>Stock <input type="number" name="stock" value="{{ old('stock') }}"></label>
        <label>Description <textarea name="description">{{ old('description') }}</textarea></label>
        <label>
            Category
            <select name="category_id">
                @foreach ($categories as $category)
                    <option value="{{ $category->id }}" @selected(old('category_id') == $category->id)>
                        {{ $category->name }}
                    </option>
                @endforeach
            </select>
        </label>
        <label>Image <input type="file" name="image"></label>
        <label><input type="checkbox" name="is_active" value="1" @checked(old('is_active', false))> Active</label>
        <button type="submit">Save</button>
    </form>
@endsection
```

## Edit

Edit tetap membuka `/products/{{ $product->id }}/edit` dan mengirim `PUT` ke `/products/{{ $product->id }}`. Pertahankan pilihan `Category`, nilai lama, error validation, serta penanganan image lama dari view sebelumnya.

```blade
@extends('layouts.app')

@section('content')
    <h1>{{ $title ?? 'Edit product' }}</h1>

    <form method="POST" action="/products/{{ $product->id }}" enctype="multipart/form-data">
        @csrf
        @method('PUT')
        <input name="name" value="{{ old('name', $product->name) }}">
        <input type="number" name="price" value="{{ old('price', $product->price) }}">
        <input type="number" name="stock" value="{{ old('stock', $product->stock) }}">
        <textarea name="description">{{ old('description', $product->description) }}</textarea>
        <select name="category_id">
            @foreach ($categories as $category)
                <option value="{{ $category->id }}" @selected(old('category_id', $product->category_id) == $category->id)>
                    {{ $category->name }}
                </option>
            @endforeach
        </select>
        <input type="file" name="image">
        <label><input type="checkbox" name="is_active" value="1" @checked(old('is_active', $product->is_active))> Active</label>
        <button type="submit">Update</button>
    </form>
@endsection
```

## Detail

Detail memakai slug, bukan ID. Tampilkan data yang benar dan lindungi relasi kategori yang mungkin kosong.

```blade
@extends('layouts.app')

@section('content')
    <h1>{{ $product->name }}</h1>

    @if ($product->image)
        <img src="{{ asset('storage/' . $product->image) }}" alt="{{ $product->name }}">
    @endif

    <p>Category: {{ $product->category?->name ?? '-' }}</p>
    <p>Price: {{ $product->price }}</p>
    <p>Stock: {{ $product->stock }}</p>
    <p>Status: {{ $product->is_active ? 'Active' : 'Inactive' }}</p>
    <p>{{ $product->description }}</p>
    <a href="/products">Back to products</a>
@endsection
```

## Pemeriksaan kontinuitas

- `is_active` adalah status ketersediaan, bukan penanda SoftDeletes.
- Data yang dihapus sementara ditandai `deleted_at` dan dilihat melalui `/products/trash`.
- Refactor ini tidak mengubah controller, validasi, upload image, endpoint, atau mutasi yang sudah ada.
- Setelah mutasi `Product` yang berhasil, invalidasi cache dashboard tetap mengikuti aturan tahap 11; layout sendiri tidak mengubah cache.
