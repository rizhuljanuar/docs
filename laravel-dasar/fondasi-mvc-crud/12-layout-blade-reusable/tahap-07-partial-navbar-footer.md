# Tahap 7 — Partial Navbar dan Footer

> Fokus: memecah bagian statis layout tanpa menambah logika aplikasi.

Partial cocok untuk fragmen kecil yang tidak membutuhkan props atau slot. Navbar dan footer pada layout utama adalah contoh sederhana. Pemecahan ini opsional: satu layout sederhana juga valid bila markupnya masih pendek.

## Navbar

```blade
{{-- resources/views/partials/navbar.blade.php --}}
<nav aria-label="Navigasi utama">
    <a href="/products">Products</a>
    <a href="/products/trash">Trash</a>
    <a href="/dashboard">Dashboard</a>
</nav>
```

## Footer

```blade
{{-- resources/views/partials/footer.blade.php --}}
<footer>
    <small>Product Management</small>
</footer>
```

## Pakai di layout

Ganti markup yang setara pada `layouts/app.blade.php` dengan include berikut.

```blade
<header>
    @include('partials.navbar')
</header>

<main>
    @yield('content')
</main>

@include('partials.footer')
```

Partial tidak membuat route, controller, atau data baru. Tautan tetap literal dan tidak berisi logika akses. Satu layout tetap menjadi konvensi untuk `/products`, `/products/trash`, serta `/dashboard`.

## Pilih alat yang tepat

| Kebutuhan | Alat |
|---|---|
| Kerangka HTML seluruh halaman | Layout |
| Fragmen statis pendek seperti navbar/footer | Partial |
| UI berulang dengan input data | Anonymous component |
| UI berulang dengan isi markup dari pemanggil | Component dengan slot |

Jangan memecah setiap satu baris menjadi partial. Gunakan saat nama file membuat struktur layout lebih mudah dibaca.
