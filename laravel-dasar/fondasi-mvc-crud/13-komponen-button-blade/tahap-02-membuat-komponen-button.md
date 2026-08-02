# Tahap 2 — Membuat Anonymous Component Button

Anonymous component adalah file Blade tanpa class PHP. Buat `resources/views/components/button.blade.php`:

```blade
@props(['variant' => 'primary'])

<button {{ $attributes->class(['btn', "btn-{$variant}"]) }}>
    {{ $slot }}
</button>
```

`@props` menerima konfigurasi `variant`; `$attributes` meneruskan atribut seperti `type="submit"`; dan `{{ $slot }}` menampilkan isi tombol. Komponen pertama ini **hanya** menghasilkan submit button. Jangan menggunakan `href` sebelum tahap 7.

Pada form create yang sudah ada, pertahankan layout, form `POST /products`, `enctype="multipart/form-data"`, semua field, `old()` values, `$categories`, image, dan status. Ganti hanya submit terakhir:

```blade
<x-button type="submit" variant="primary">Save product</x-button>
```

Cancel tetap link biasa sampai tahap 7:

```blade
<a class="btn btn-secondary" href="/products">Cancel</a>
```
