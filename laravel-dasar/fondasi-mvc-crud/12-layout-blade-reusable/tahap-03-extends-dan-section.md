# Tahap 3 — Memakai `@extends` dan `@section`

> Fokus: pola child view tanpa membuat halaman, route, controller, atau model baru.

## Pola dasar

Setiap view yang dipindahkan ke layout mulai dengan `@extends('layouts.app')`, lalu mengisi area bernama `content`.

```blade
@extends('layouts.app')

@section('content')
    <h1>Product Management</h1>
    <p>Isi halaman berada di sini.</p>
@endsection
```

Saat Blade dirender, `@extends` mengambil kerangka `layouts.app`; `@section('content')` mengisi `@yield('content')` di dalam kerangka itu.

## Judul halaman

Controller tetap tempat yang baik untuk menyiapkan judul karena data presentasi terkumpul di satu tempat. Namun Blade dapat memakai variabel yang sudah tersedia ketika `@extends` dirender; Blade tidak dilarang memakai variabel itu.

Contoh controller yang hanya menambah data presentasi pada view yang sudah ada:

```php
return view('products.index', [
    'products' => $products,
    'title' => 'Products',
]);
```

Lalu child view dapat tetap sangat sederhana:

```blade
@extends('layouts.app')

@section('content')
    <h1>{{ $title ?? 'Products' }}</h1>
    {{-- tabel dan paginator $products tetap di sini --}}
@endsection
```

## `@push` untuk tambahan khusus

Jika satu view benar-benar membutuhkan CSS lokal, dorong isinya ke stack yang telah disediakan layout.

```blade
@push('css')
    <style>
        .product-image { max-width: 12rem; }
    </style>
@endpush
```

Letakkan `@push` di child view, bukan di layout. Jangan memakai `@push` untuk memindahkan CSS umum berulang; CSS umum tetap milik layout.

## Checklist

- [ ] Semua child template nanti memakai `@extends('layouts.app')`.
- [ ] Nama section sama persis: `content`.
- [ ] Layout memiliki pasangan `@yield('content')`.
- [ ] Refactor view tidak mengubah route atau query controller.

Tahap 4 menerapkan pola ini pada daftar yang sudah memiliki pencarian, filter, pengurutan, dan pagination.
