# Tahap 4 — Slot sebagai Isi Tombol

Lesson 12 sudah memperkenalkan component dan slot. Slot adalah markup di antara tag pembuka dan penutup component. Pada component button, lokasi slot adalah:

```blade
<button {{ $attributes->class(['btn', "btn-{$variant}"]) }}>
    {{ $slot }}
</button>
```

Gunakan `{{ $slot }}`, bukan output mentah. Props adalah konfigurasi seperti `variant` dan `type`; slot adalah isi yang tampil.

```blade
<x-button type="submit" variant="primary">
    <strong>Save</strong> product
</x-button>
```

Label harus eksplisit. Jangan memakai self-closing component untuk button karena slot kosong:

```blade
<x-button type="submit" variant="primary">Save product</x-button>
```

Tidak diperlukan library ikon maupun ikon otomatis berdasarkan variant.
