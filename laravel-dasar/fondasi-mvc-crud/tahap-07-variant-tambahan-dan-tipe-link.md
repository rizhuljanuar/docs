# Tahap 7 — Link Mode pada Component yang Sama

Navigasi memakai `<a>`; submit form memakai `<button>`. Evolusikan component **yang sama**, bukan API component link kedua:

```blade
@props(['variant' => 'primary', 'href' => null])

@php
    $variants = ['primary', 'secondary', 'danger', 'warning', 'info'];
    $variant = in_array($variant, $variants, true) ? $variant : 'primary';
@endphp

@if ($href)
    <a href="{{ $href }}" {{ $attributes->except('href')->class(['btn', "btn-{$variant}"]) }}>
        {{ $slot }}
    </a>
@else
    <button {{ $attributes->class(['btn', "btn-{$variant}"]) }}>
        {{ $slot }}
    </button>
@endif
```

`except('href')` mencegah atribut `href` tampil dua kali. Jangan pernah memberi `href` kepada submit button; jika ada `href`, component menjadi link dan tidak mengirim form.

## Contoh literal URL

```blade
<x-button variant="primary" href="/products/create">Add product</x-button>
<x-button variant="info" href="/products/{{ $product->slug }}">Detail</x-button>
<x-button variant="warning" href="/products/{{ $product->id }}/edit">Edit</x-button>
<x-button variant="secondary" href="/products">Cancel</x-button>
```

Submit tetap tanpa `href`:

```blade
<x-button type="submit" variant="primary">Save product</x-button>
```

`<x-button-delete>` tetap memakai `<x-button type="submit" variant="danger">`; ia tidak memakai link mode.
