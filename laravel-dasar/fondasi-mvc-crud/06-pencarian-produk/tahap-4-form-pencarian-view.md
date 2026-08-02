# Tahap 4: Form Pencarian pada View

## Tujuan

Pada Tahap 3 pencarian diuji dari URL. Sekarang pengguna dapat memasukkan kata kunci melalui form pada `resources/views/products/index.blade.php`.

## Mengapa Menggunakan GET?

Pencarian hanya membaca dan menyaring data, sehingga form memakai `method="GET"`. Hasilnya dapat dibagikan atau dibuka kembali:

```text
/products?search=laptop
```

Form GET **tidak memakai `@csrf`**. Token CSRF diperlukan untuk request yang mengubah data, seperti form hapus yang tetap memakai `POST` dan `@method('DELETE')`.

## Tambahkan Form

Letakkan form ini di atas tabel daftar produk:

```blade
<form action="/products" method="GET">
    <input
        type="text"
        name="search"
        value="{{ request('search') }}"
        maxlength="100"
        placeholder="Cari berdasarkan nama produk..."
    >

    <button type="submit">Cari</button>
    <a href="/products">Reset</a>
</form>
```

`name="search"` harus sama dengan input yang dibaca controller. `request('search')` mengisi kembali kotak pencarian setelah form dikirim.

## Contoh Bagian Daftar

Gunakan nama field, variabel, dan URL yang sudah ditetapkan materi sebelumnya:

```blade
@foreach ($products as $product)
    <tr>
        <td>{{ $product->name }}</td>
        <td>{{ $product->price }}</td>
        <td>{{ $product->stock }}</td>
        <td>{{ $product->category?->name ?? '-' }}</td>
        <td>
            <a href="/products/{{ $product->slug }}">Lihat</a>
            <a href="/products/{{ $product->id }}/edit">Edit</a>
            <form action="/products/{{ $product->id }}" method="POST" style="display: inline;">
                @csrf
                @method('DELETE')
                <button type="submit">Hapus</button>
            </form>
        </td>
    </tr>
@endforeach

{{ $products->links() }}
```

Form hapus merupakan form terpisah dan tetap menggunakan CSRF karena menghapus data.

## Uji

1. Buka `/products`.
2. Masukkan `laptop`, lalu klik **Cari**.
3. Pastikan URL menjadi `/products?search=laptop`.
4. Klik **Reset** untuk kembali ke `/products`.

## Inti Tahap 4

> Form pencarian memakai GET tanpa CSRF dan mengirim parameter `search`. Tahap berikutnya memindahkan query sederhana dari controller ke local scope pada model `Product`.
