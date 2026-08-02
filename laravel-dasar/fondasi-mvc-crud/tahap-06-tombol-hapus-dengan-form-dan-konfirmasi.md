# Tahap 6 — Delete Form dan Konfirmasi

Delete bukan link GET. Browser mengirim form `POST`, lalu `@method('DELETE')` memberi tahu Laravel bahwa tindakannya DELETE. `@csrf` melindungi form dari CSRF.

Buat component khusus berikut:

```text
resources/views/components/button-delete.blade.php
```

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

`action` tidak memiliki default sehingga setiap penggunaan wajib menyebut URL penghapusan. `Js::from()` membuat string konfirmasi aman apabila nama product memuat tanda kutip.

Pada tabel index, pertahankan kolom dan URL lain lalu gunakan:

```blade
<x-button-delete
    action="/products/{{ $product->id }}"
    :confirm="'Delete ' . $product->name . '?'">
    Delete
</x-button-delete>
```

Form ini mengirim DELETE ke ID product dan menjalankan soft delete. Data terhapus sementara muncul di `/products/trash`. `is_active` adalah status product yang terpisah dari `deleted_at`; tombol ini tidak mengubah `is_active`. Jangan menebak atau menambahkan UI force-delete maupun restore.
