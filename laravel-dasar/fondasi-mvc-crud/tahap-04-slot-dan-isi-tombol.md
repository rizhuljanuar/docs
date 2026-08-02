# Tahap 4 — Slot sebagai Isi Tombol

Lesson 12 sudah memperkenalkan component dan slot. Slot adalah markup di antara tag pembuka dan penutup component. Pada button, lokasi slot adalah:

```blade
<button {{ $attributes->class(['btn', "btn-{$variant}"]) }}>
    {{ $slot }}
</button>
```

Gunakan `{{ $slot }}`, bukan output mentah. Blade merender markup slot component dengan aman sesuai mekanisme component.

## Props vs slot

- Props adalah konfigurasi, misalnya `variant="warning"` atau `type="submit"`.
- Slot adalah isi yang terlihat, misalnya label tombol atau markup sederhana.

```blade
<x-button type="submit" variant="primary">
    <strong>Save</strong> product
</x-button>
```

Label harus eksplisit. Hindari component self-closing untuk tombol karena slot-nya kosong:

```blade
<x-button type="submit" variant="primary">Save product</x-button>
```

Tidak diperlukan library ikon atau pemetaan ikon otomatis. Jika nanti diperlukan markup tambahan, pemanggil dapat menaruhnya di slot tanpa mengubah API component.
