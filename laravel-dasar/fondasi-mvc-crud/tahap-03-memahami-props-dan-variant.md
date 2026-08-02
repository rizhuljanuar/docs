# Tahap 3 — Props dan Variant

`variant` adalah props: nilai konfigurasi yang menentukan peran visual tombol. Untuk pelajaran ini, gunakan allowlist sempit berikut:

```blade
@props(['variant' => 'primary'])

@php
    $variants = ['primary', 'secondary', 'danger', 'warning', 'info'];
    $variant = in_array($variant, $variants, true) ? $variant : 'primary';
@endphp

<button {{ $attributes->class(['btn', "btn-{$variant}"]) }}>
    {{ $slot }}
</button>
```

Pemetaan perannya tetap:

- `primary`: Add, Save, Update.
- `secondary`: Cancel, Back.
- `info`: Detail.
- `warning`: Edit.
- `danger`: Delete.

Komponen tidak menambahkan emoji atau ikon berdasarkan variant. Isi tombol tetap ditulis jelas oleh pemanggil.

Contoh submit form:

```blade
<x-button type="submit" variant="primary">Update product</x-button>
```

Contoh di atas tetap button karena tidak memiliki `href`. Link mode baru ditambahkan pada tahap 7.
