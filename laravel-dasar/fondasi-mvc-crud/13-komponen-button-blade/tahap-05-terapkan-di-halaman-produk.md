# Tahap 5 — Terapkan pada View Product

Komponen mengganti kontrol saja. Semua child view tetap memakai satu layout:

```blade
@extends('layouts.app')

@section('content')
    {{-- isi halaman --}}
@endsection
```

## Index

Pertahankan seluruh form GET ke `/products`: `search`, select `category_id` dari `$categories`, pilihan `sort`, nilai request, tabel hasil, serta `{{ $products->links() }}`. `$products` tetap paginator dari controller yang telah ada. Submit filter boleh menjadi:

```blade
<x-button type="submit" variant="primary">Apply filters</x-button>
```

Sampai tahap 7, navigasi tetap link biasa:

```blade
<a class="btn btn-primary" href="/products/create">Add product</a>
<a class="btn btn-info" href="/products/{{ $product->slug }}">Detail</a>
<a class="btn btn-warning" href="/products/{{ $product->id }}/edit">Edit</a>
```

## Create, edit, dan detail

Jangan mengganti form lengkap. Create tetap `POST /products`; edit tetap `POST /products/{{ $product->id }}` dengan `@method('PUT')`. Keduanya tetap multipart dan mempertahankan field `name`, `price`, `stock`, `description`, `category_id`, `image`, `slug`, `is_active`, nilai lama, category, image, dan status.

Ganti hanya kontrol simpan:

```blade
<x-button type="submit" variant="primary">Save product</x-button>
<a class="btn btn-secondary" href="/products">Cancel</a>
```

```blade
<x-button type="submit" variant="primary">Update product</x-button>
<a class="btn btn-secondary" href="/products">Cancel</a>
```

Detail tetap di `/products/{{ $product->slug }}` dan tetap menampilkan name, image, price, stock, description, `{{ $product->category?->name }}`, dan `is_active`:

```blade
<a class="btn btn-secondary" href="/products">Back to products</a>
```

Pelajaran ini tidak mengubah dashboard query, cache, atau controller.
