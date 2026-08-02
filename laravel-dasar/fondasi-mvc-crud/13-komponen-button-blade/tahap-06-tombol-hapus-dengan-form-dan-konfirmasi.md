# Tahap 6 — Delete Form dan Konfirmasi

Delete bukan link GET. Browser mengirim form `POST`; `@method('DELETE')` memberi tahu Laravel bahwa tindakannya DELETE, sedangkan `@csrf` melindungi form dari CSRF.

Buat `resources/views/components/button-delete.blade.php`:

```blade
@props([
    'action',
    'confirm' => 'Delete this product?',
])

<form action="{{ $action }}" method="POST" style="display:inline">
    @csrf
    @method('DELETE')
    <x-button
        type="submit"
        variant="danger"
        onclick="return confirm({{ Illuminate\Support\Js::from($confirm) }})">
        {{ $slot }}
    </x-button>
</form>
```

`action` wajib diisi karena tidak memiliki default. `Js::from()` membuat nama product aman jika berisi tanda kutip.

Pada tabel index, gunakan:

```blade
<x-button-delete
    action="/products/{{ $product->id }}"
    :confirm="'Delete ' . $product->name . '?'">
    Delete
</x-button-delete>
```

Form mengirim DELETE ke ID product dan menjalankan soft delete. Data terhapus sementara ada di `/products/trash`. `is_active` adalah status yang terpisah dari `deleted_at`; delete tidak mengubah `is_active`. Jangan menambahkan UI force-delete atau restore.
