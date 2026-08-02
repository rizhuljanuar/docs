# Tahap 6 — Form Status dengan State Eksplisit

Tambahkan form status di `resources/views/products/index.blade.php`, di samping kontrol edit dan hapus produk. Jangan gunakan link GET untuk mutasi.

```blade
@if ($product->is_active)
    <form action="/products/{{ $product->id }}/status" method="POST">
        @csrf
        @method('PATCH')
        <input type="hidden" name="is_active" value="0">
        <button type="submit">Nonaktifkan</button>
    </form>
@else
    <form action="/products/{{ $product->id }}/status" method="POST">
        @csrf
        @method('PATCH')
        <input type="hidden" name="is_active" value="1">
        <button type="submit">Aktifkan</button>
    </form>
@endif
```

`@csrf` melindungi request perubahan data. HTML form mengirim `POST`, sedangkan `@method('PATCH')` membuat Laravel memperlakukannya sebagai PATCH sesuai route. Konfirmasi JavaScript boleh ditambahkan, tetapi bukan pengganti validasi server atau authorization.

## Mengapa memakai state eksplisit?

Setiap form mengirim target status yang eksplisit: `0` untuk nonaktif atau `1` untuk aktif. Controller memvalidasi lalu menjalankan:

```php
$product->update($validated);
```

Cara ini idempoten: mengirim permintaan “set nonaktif” dua kali tetap menghasilkan nonaktif. Ini lebih aman terhadap double submit daripada membalik nilai dengan negasi boolean. Jangan membalik nilai status dari nilai yang tersimpan.

## Batas soft delete

Endpoint memakai `Product::findOrFail($id)`, sehingga produk trashed tidak bisa diubah statusnya dan mendapat 404. Restore dilakukan melalui route/view tong sampah yang sudah ada; setelah direstore, produk mempertahankan nilai `is_active` sebelum dihapus. Dengan kata lain, query normal SoftDeletes berarti “bukan trashed”, sedangkan `is_active` berarti “status publikasi”.
