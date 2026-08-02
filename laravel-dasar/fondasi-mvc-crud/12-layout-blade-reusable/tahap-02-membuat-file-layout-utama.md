# Tahap 2 — Membuat Layout Utama

> Fokus: satu layout `resources/views/layouts/app.blade.php`.

## Tujuan

Membuat kerangka HTML bersama dengan judul dinamis, tiga navigasi literal, satu area isi, serta stack CSS dan JavaScript.

Buat file berikut.

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{{ $title ?? 'Product Management' }}</title>
    @stack('css')
</head>
<body>
    <header>
        <nav aria-label="Navigasi utama">
            <a href="/products">Products</a>
            <a href="/products/trash">Trash</a>
            <a href="/dashboard">Dashboard</a>
        </nav>
    </header>

    <main>
        @yield('content')
    </main>

    <footer>
        <small>Product Management</small>
    </footer>

    @stack('js')
</body>
</html>
```

## Membaca bagian penting

- `{{ $title ?? 'Product Management' }}` memakai nilai `$title` bila tersedia, dan judul cadangan bila tidak ada.
- `@yield('content')` adalah lokasi yang akan diisi child view dengan `@section('content')`.
- `@stack('css')` dan `@stack('js')` adalah lokasi tambahan opsional dari child view melalui `@push`.
- Semua URL navigasi ditulis literal karena route aplikasi yang diteruskan memang literal.

Jangan mengganti navigasi dengan route baru atau helper bernama. Jangan membuat layout tambahan: tiga halaman yang ada mempunyai struktur yang cukup sama untuk memakai satu layout ini.

## Pemeriksaan

- [ ] Lokasi file tepat: `resources/views/layouts/app.blade.php`.
- [ ] Ada `@yield('content')` tepat sebagai area konten.
- [ ] Ada `@stack('css')` dan `@stack('js')`.
- [ ] Tiga URL nav adalah `/products`, `/products/trash`, dan `/dashboard`.
- [ ] Layout tidak berisi query database atau logika mutasi.

Setelah file ini ada, tahap 3 menunjukkan pola child view.
