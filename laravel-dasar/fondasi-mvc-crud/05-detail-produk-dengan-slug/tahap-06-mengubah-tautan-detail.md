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
{slug}
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

Cari tautan **Detail** yang masih memakai ID. Di **Materi 4 (Tahap 10 dan 11)**,
tautan ini ditulis dengan URL biasa:

```blade
<a href="/products/{{ $product->id }}">Lihat</a>
```

## Langkah 2: Mengganti ID dengan Slug

Ubah tautan tersebut menjadi:

```blade
<a href="/products/{{ $product->slug }}">Lihat</a>
```

Sekarang nilai yang dikirim ke URL adalah:

```blade
$product->slug
```

Bukan lagi:

```blade
$product->id
```

> **Catatan:** Kita tetap memakai URL biasa (`/products/{{ ... }}`) seperti
> di **Materi 1 sampai 4**, bukan named route. Ini konsisten dengan pola
> yang sudah dipakai sepanjang materi.

## Contoh pada Kolom Aksi

Di **Materi 4 (Tahap 11)**, kolom aksi pada halaman daftar produk memiliki
tautan untuk filter kategori, lihat, edit, dan hapus. Kita hanya mengubah
tautan **Lihat/Detail** agar memakai slug.

```blade
<td>
    <a href="/products/{{ $product->slug }}">Lihat</a>
    <a href="/products/{{ $product->id }}/edit">Edit</a>
    <form action="/products/{{ $product->id }}" method="POST" style="display:inline;">
        @csrf
        @method('DELETE')
        <button type="submit" onclick="return confirm('Hapus produk ini?')">Hapus</button>
    </form>
</td>
```

Perhatikan:

- **Lihat/Detail** memakai slug.
- **Edit** tetap memakai ID.
- **Hapus** tetap memakai ID.

Ini sesuai dengan route yang kita buat pada Tahap 5.

## Jika Nama Produk Juga Menjadi Tautan

Beberapa halaman tidak memiliki tombol **Detail**, tetapi nama produknya bisa
diklik.

Ubah tautannya dengan pola yang sama:

```blade
<a href="/products/{{ $product->slug }}">
    {{ $product->name }}
</a>
```

Lakukan perubahan pada setiap tautan yang memang menuju halaman detail produk.

## Langkah 3: Menguji Tautan

1. Buka halaman daftar:

   ```text
   http://127.0.0.1:8000/products
   ```

2. Arahkan mouse ke tombol **Lihat** atau **Detail**.
3. Pastikan alamat yang terlihat memakai slug, bukan hanya ID.
4. Klik **Lihat**.
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
<a href="/products/{{ $product->id }}">Lihat</a>
```

Benar:

```blade
<a href="/products/{{ $product->slug }}">Lihat</a>
```

### Slug Kosong

Jika URL tidak lengkap (misalnya `/products/` tanpa slug), periksa kolom
`slug` pada tabel `products`. Pastikan langkah pengisian slug produk lama pada
Tahap 4 sudah dijalankan.

## Checklist Tahap 6

- [ ] Tautan detail tidak lagi memakai `$product->id`.
- [ ] Tautan detail memakai `$product->slug`.
- [ ] Tautan detail memakai URL biasa: `/products/{{ $product->slug }}`.
- [ ] Klik **Lihat** menghasilkan URL slug.
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
