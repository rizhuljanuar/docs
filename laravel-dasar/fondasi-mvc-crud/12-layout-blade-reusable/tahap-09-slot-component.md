# Tahap 9 — Slot pada Component

> Fokus: slot default dan named slot dengan pemeriksaan Blade yang benar.

Props membawa nilai terstruktur, misalnya satu `Product`. Slot membawa markup yang ditulis pemanggil. Gunakan slot ketika isi markup memang berbeda, bukan untuk menggantikan prop yang sederhana.

## Component panel

Buat component anonymous berikut.

```blade
{{-- resources/views/components/panel.blade.php --}}
@props(['title'])

<section>
    @isset($header)
        <header>{{ $header }}</header>
    @endisset

    <div>
        <h2>{{ $title }}</h2>
        @if (! $slot->isEmpty())
            {{ $slot }}
        @endif
    </div>

    @isset($footer)
        <footer>{{ $footer }}</footer>
    @endisset
</section>
```

`$slot` tetap ada meskipun pemanggil tidak mengirim isi, sehingga pemeriksaan yang aman untuk default slot adalah `! $slot->isEmpty()`. Named slot berbeda: gunakan `@isset($header)` atau `@isset($footer)` sebelum merendernya.

## Pemakaian dengan data Product

```blade
<x-panel title="{{ $product->name }}">
    <x-slot:header>
        <p>{{ $product->category?->name ?? '-' }}</p>
    </x-slot:header>

    <p>Price: {{ $product->price }}</p>
    <p>Stock: {{ $product->stock }}</p>
    <x-status-badge :is-active="$product->is_active" />

    <x-slot:footer>
        <a href="/products/{{ $product->slug }}">View details</a>
    </x-slot:footer>
</x-panel>
```

Contoh di atas aman karena panel hanya menerima judul dan menampilkan markup slot. Tautan detail tetap literal dan memakai slug. Tidak ada component tombol yang menganggap atribut tautan akan otomatis bekerja.

## Kapan memakai slot

| Kondisi | Pilihan |
|---|---|
| Nilai satu `Product` | prop `:product="$product"` |
| Badge menerima boolean | prop `:is-active="$product->is_active"` |
| Isi panel berbeda antar pemakaian | default slot |
| Bagian atas atau bawah opsional | named slot |

Slot tidak mengubah endpoint, status `is_active`, SoftDeletes, maupun query yang sudah ada.
