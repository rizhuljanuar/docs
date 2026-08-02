# Tahap 5 — Terapkan pada View Product

Komponen mengganti kontrol saja. Semua view tetap memakai satu layout:

```blade
@extends('layouts.app')

@section('content')
    {{-- isi halaman --}}
@endsection
```

## Index

Pertahankan **seluruh** form GET ke `/products`: input `search`, select `category_id` dari `$categories`, pilihan `sort`, nilai request, tabel hasil, dan pagination `{{ $products->links() }}`. `$products` tetap paginator dari controller yang sudah ada.

Sampai tahap 7, kontrol navigasi masih link biasa. Submit filter dapat memakai component:

```blade
<form method="GET" action="/products">
    {{-- search, category filter, dan sort yang sudah ada tetap di sini --}}
    <x-button type="submit" variant="primary">Apply filters</x-button>
</form>

<a class="btn btn-primary" href="/products/create">Add product</a>
```

Di tabel, detail tetap `/products/{{ $product->slug }}` dan edit tetap `/products/{{ $product->id }}/edit`.

## Create dan edit

Jangan mengganti form lengkap yang sudah ada. Create tetap `POST /products`; edit tetap `POST /products/{{ $product->id }}` dengan `@method('PUT')`. Keduanya tetap multipart dan mempertahankan `old()` values, category selection, image handling, serta `is_active`.

Ganti hanya Save/Update dan pertahankan Cancel sebagai link sampai tahap 7:

```blade
<x-button type="submit" variant="primary">Save product</x-button>
<a class="btn btn-secondary" href="/products">Cancel</a>
```

```blade
<x-button type="submit" variant="primary">Update product</x-button>
<a class="btn btn-secondary" href="/products">Cancel</a>
```

## Detail

Detail tetap di URL slug `/products/{{ $product->slug }}` dan tetap menampilkan image, name, price, stock, description, `category?->name`, dan `is_active`. Kontrol Back masih:

```blade
<a class="btn btn-secondary" href="/products">Back to products</a>
```

Button lesson tidak mengubah dashboard controller, query, atau cache.
