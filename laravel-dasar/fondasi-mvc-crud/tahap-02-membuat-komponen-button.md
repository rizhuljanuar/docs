# Tahap 2 — Membuat Anonymous Component Button

Anonymous component adalah file Blade tanpa class PHP. Buat file berikut:

```text
resources/views/components/button.blade.php
```

Isi pertama harus hanya mendukung submit button:

```blade
@props(['variant' => 'primary'])

<button {{ $attributes->class(['btn', "btn-{$variant}"]) }}>
    {{ $slot }}
</button>
```

- `@props` menerima konfigurasi `variant`; default-nya `primary`.
- `$attributes` meneruskan atribut seperti `type="submit"` dan menggabungkan class pemanggil dengan `btn` serta `btn-{variant}`.
- `{{ $slot }}` menampilkan isi komponen.

Jangan kirim `href` pada tahap ini: outputnya masih selalu elemen `<button>`.

## Penggunaan pertama: form tambah yang sudah ada

Pertahankan seluruh form create yang sudah selesai—termasuk `@extends('layouts.app')`, `@section('content')`, `enctype`, input `name`, `price`, `stock`, `description`, `category_id`, `image`, `is_active`, nilai `old()`, dan perulangan `$categories`. Ganti **hanya** tombol submit terakhir:

```blade
<x-button type="submit" variant="primary">
    Save product
</x-button>
```

Karena belum ada link mode, tombol Cancel tetap memakai link biasa pada tahap ini:

```blade
<a class="btn btn-secondary" href="/products">Cancel</a>
```

Hasil render tombol Save adalah submit button dengan class `btn btn-primary`.
