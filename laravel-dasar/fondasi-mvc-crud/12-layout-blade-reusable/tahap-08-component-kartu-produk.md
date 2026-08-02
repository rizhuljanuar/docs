# Tahap 8 — Anonymous Component Kartu Product

> Fokus: presentasi opsional yang reusable; tabel paginator pada daftar tetap dipertahankan.

Anonymous component adalah file Blade di `resources/views/components` yang dipanggil memakai tag `<x-...>`. Untuk kartu `Product`, buat dua file berikut.

## Badge status

```blade
{{-- resources/views/components/status-badge.blade.php --}}
@props(['isActive'])

<span>
    {{ $isActive ? 'Active' : 'Inactive' }}
</span>
```

## Kartu product

```blade
{{-- resources/views/components/kartu-produk.blade.php --}}
@props(['product'])

<article>
    @if ($product->image)
        <img src="{{ asset('storage/' . $product->image) }}" alt="{{ $product->name }}">
    @else
        <p>No image available.</p>
    @endif

    <h2><a href="/products/{{ $product->slug }}">{{ $product->name }}</a></h2>
    <p>Category: {{ $product->category?->name ?? '-' }}</p>
    <p>Price: {{ $product->price }}</p>
    <p>Stock: {{ $product->stock }}</p>
    <x-status-badge :is-active="$product->is_active" />
</article>
```

Pemakaian:

```blade
<x-kartu-produk :product="$product" />
```

## Gunakan tanpa merusak daftar final

Kartu boleh dipakai pada area presentasi tambahan, misalnya daftar ringkas dari data yang memang sudah tersedia. Jangan menyuruh mengganti tabel utama `/products` dengan kartu apabila itu menghapus form pencarian, filter `Category`, sort, atau `{{ $products->links() }}`. `$products` tetap paginator hasil controller yang sudah selesai pada tahap 06–08.

## Checklist data

- [ ] Prop adalah `product`, bukan data buatan.
- [ ] Image dicek sebelum dirender.
- [ ] Detail selalu memakai `/products/{{ $product->slug }}`.
- [ ] Relasi memakai `$product->category?->name`.
- [ ] Badge menerima `is_active` dengan `:is-active`.
- [ ] Status aktif tidak disamakan dengan `deleted_at`.
