# Tahap 6: Mengubah Tautan Detail agar Memakai Slug

## Tujuan Tahap Ini

Pada Tahap 5, halaman detail sudah bisa dibuka melalui URL slug:

```text
/products/kaos-hitam-7
```

Namun, tombol **Detail** pada halaman daftar mungkin masih membuat URL dari ID:

```text
/products/7
```

Sekarang kita akan mengubah tautan tersebut agar mengirim slug.

## Analogi Sederhana: Memperbarui Petunjuk Arah

Bayangkan sebuah toko sudah pindah ke alamat baru, tetapi papan petunjuk masih
menunjukkan alamat lama.

Route slug adalah alamat baru. Tautan **Detail** adalah papan petunjuknya.
Keduanya harus memakai alamat yang sama agar pengunjung sampai ke halaman yang
benar.

## Kenapa Tautan ID Tidak Lagi Bekerja?

Route detail sekarang memakai:

```php
{product:slug}
```

Artinya, Laravel menganggap bagian terakhir URL sebagai slug.

Jika tautan mengirim:

```text
/products/7
```

Laravel akan mencari produk dengan:

```text
slug = 7
```

Slug tersebut tidak ada, sehingga Laravel menampilkan halaman 404.

## Langkah 1: Membuka Halaman Daftar Produk

Buka:

```text
resources/views/products/index.blade.php
```

Cari tautan **Detail** yang masih memakai ID:

```blade
<a href="/products/{{ $product->id }}">Detail</a>
```

## Langkah 2: Mengganti ID dengan Slug

Ubah tautan tersebut menjadi:

```blade
<a href="{{ route('products.show', ['product' => $product->slug]) }}">
    Detail
</a>
```

Sekarang nilai yang dikirim ke parameter `product` adalah:

```blade
$product->slug
```

Bukan lagi:

```blade
$product->id
```

## Memahami `route()`

Kode berikut:

```blade
route('products.show', ['product' => $product->slug])
```

memiliki arti:

| Bagian                    | Arti                                      |
|---------------------------|-------------------------------------------|
| `products.show`           | Nama route detail dari Tahap 5            |
| `product`                 | Nama parameter pada `{product:slug}`      |
| `$product->slug`          | Nilai slug yang dimasukkan ke dalam URL   |

Jika nilai slug adalah:

```text
kaos-hitam-7
```

Laravel menghasilkan URL:

```text
http://127.0.0.1:8000/products/kaos-hitam-7
```

Kita memakai nama route agar URL tidak ditulis manual di banyak tempat.

## Contoh pada Kolom Aksi

Jika halaman daftar memiliki tombol **Detail**, **Edit**, dan **Hapus**, bagian
aksinya dapat terlihat seperti ini:

```blade
<td>
    <a href="{{ route('products.show', ['product' => $product->slug]) }}">
        Detail
    </a>

    <a href="/products/{{ $product->id }}/edit">
        Edit
    </a>

    <form action="/products/{{ $product->id }}" method="POST">
        @csrf
        @method('DELETE')
        <button type="submit">Hapus</button>
    </form>
</td>
```

Perhatikan:

- **Detail** memakai slug.
- **Edit** tetap memakai ID.
- **Hapus** tetap memakai ID.

Ini sesuai dengan route yang kita buat pada Tahap 5.

## Jika Nama Produk Juga Menjadi Tautan

Beberapa halaman tidak memiliki tombol **Detail**, tetapi nama produknya bisa
diklik.

Ubah tautannya dengan pola yang sama:

```blade
<a href="{{ route('products.show', ['product' => $product->slug]) }}">
    {{ $product->name }}
</a>
```

Lakukan perubahan pada setiap tautan yang memang menuju halaman detail produk.

## Kenapa Tidak Mengirim `$product` Langsung?

Kode berikut terlihat lebih pendek:

```blade
route('products.show', $product)
```

Namun, model `Product` kita masih memakai ID sebagai route key bawaan. Kode
tersebut dapat menghasilkan URL dengan ID.

Karena route detail secara khusus memakai slug, kirim slug secara jelas:

```blade
route('products.show', ['product' => $product->slug])
```

## Langkah 3: Menguji Tautan

1. Buka halaman daftar:

   ```text
   http://127.0.0.1:8000/products
   ```

2. Arahkan mouse ke tombol **Detail**.
3. Pastikan alamat yang terlihat memakai slug, bukan hanya ID.
4. Klik **Detail**.
5. Pastikan halaman produk yang benar tampil.

Contoh hasil:

```text
http://127.0.0.1:8000/products/kaos-hitam-7
```

Uji juga tombol **Edit** dan **Hapus** untuk memastikan keduanya masih bekerja
dengan ID.

## Kesalahan yang Sering Terjadi

### Masih Mengirim ID

Salah:

```blade
route('products.show', ['product' => $product->id])
```

Benar:

```blade
route('products.show', ['product' => $product->slug])
```

### Nama Route Tidak Ditemukan

Jika muncul error:

```text
Route [products.show] not defined.
```

Pastikan route detail pada `routes/web.php` memiliki:

```php
->name('products.show');
```

### Slug Kosong

Jika URL tidak lengkap, periksa kolom `slug` pada tabel `products`. Pastikan
langkah pengisian slug produk lama pada Tahap 4 sudah dijalankan.

## Checklist Tahap 6

- [ ] Tautan detail tidak lagi memakai `$product->id`.
- [ ] Tautan detail memakai `$product->slug`.
- [ ] Nama route yang dipakai adalah `products.show`.
- [ ] Klik **Detail** menghasilkan URL slug.
- [ ] Halaman menampilkan produk yang benar.
- [ ] Tautan edit dan hapus tetap bekerja dengan ID.

## Inti Tahap 6

> Route sudah mencari berdasarkan slug, jadi tautan detail juga harus
> mengirimkan slug.

Sekarang pengguna dapat membuka halaman detail dari daftar produk melalui URL
yang lebih jelas dan mudah dibaca.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke **Tahap 7: pengujian akhir dan rangkuman alur
slug**?

Ketik **"lanjut"** jika sudah siap.
