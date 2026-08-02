# Tahap 3 — Menampilkan Flash Success di Blade

> Fokus: membaca pesan `success` dari session flash dan menampilkannya setelah user kembali ke `/products`.

## Pesan sudah dibawa, tetapi belum dibaca

Pada tahap 2, `ProductController@store()` sudah mengirim pesan ini:

```php
return redirect('/products')->with('success', 'Data berhasil disimpan');
```

Bayangkan controller menitipkan secarik catatan kepada Laravel:

```text
success: Data berhasil disimpan
```

Browser sudah sampai di `/products`, tetapi halaman Blade belum tahu bahwa ia harus membaca dan menampilkan catatan itu. Sekarang kita menambahkan satu pemeriksaan kecil di layout.

## Mengapa menaruhnya di layout?

Semua halaman yang sudah dibuat memakai layout yang sama:

```text
resources/views/layouts/app.blade.php
```

Layout memiliki `<main>` dan `@yield('content')`. Jika pesan diletakkan di layout, setiap halaman yang menerima flash message dapat menampilkannya di tempat yang konsisten. Kita tidak perlu menyalin kode notifikasi ke setiap view Product.

## Tambahkan pemeriksaan `success`

Buka `resources/views/layouts/app.blade.php`. Di dalam `<main>`, letakkan kode berikut **sebelum** `@yield('content')`:

```blade
<main>
    @if (session('success'))
        <div role="status">
            {{ session('success') }}
        </div>
    @endif

    @yield('content')
</main>
```

Jangan menghapus `@yield('content')`. Baris itu tetap diperlukan untuk menampilkan isi halaman seperti daftar products, form add, detail, dan dashboard.

## Membaca kode pelan-pelan

| Kode | Arti sederhana |
| --- | --- |
| `session('success')` | Minta Laravel membaca data session flash dengan nama `success`. |
| `@if (...)` | Tampilkan kotak pesan hanya jika pesan `success` memang ada. |
| `{{ session('success') }}` | Tampilkan isi pesannya dengan aman di halaman. |
| `role="status"` | Memberi tahu teknologi bantu bahwa area ini adalah pesan status. |
| `@endif` | Menutup pemeriksaan `@if`. |

Nama `success` harus sama persis dengan nama dari controller:

```php
->with('success', 'Data berhasil disimpan')
```

Jika controller mengirim `success`, tetapi Blade membaca `status` atau `message`, Blade tidak akan menemukan pesannya.

## Alur lengkap setelah tahap 3

```text
User menekan Save product
        |
        v
ProductController@store menyimpan Product
        |
        v
redirect('/products')->with('success', 'Data berhasil disimpan')
        |
        v
Layout membaca session('success')
        |
        v
User melihat: Data berhasil disimpan
```

Saat halaman `/products` dimuat lagi setelah pesan tersebut dipakai, flash message akan hilang. Itulah alasan `@if` diperlukan: pada kunjungan biasa ke `/products`, tidak ada pesan yang perlu ditampilkan dan tidak ada kotak notifikasi kosong.

## Coba sendiri

1. Tambahkan blok Blade di layout, tepat sebelum `@yield('content')`.
2. Buat product baru dari `/products/create`.
3. Tekan **Save product**.
4. Setelah kembali ke `/products`, pesan **Data berhasil disimpan** harus terlihat.
5. Muat ulang halaman `/products` sekali lagi. Pesan seharusnya sudah tidak tampil.

Jika pesan tidak muncul, periksa dua hal dulu:

- `store()` menggunakan `->with('success', 'Data berhasil disimpan')`.
- Blade juga membaca `session('success')`, bukan nama lain.

## Yang tidak berubah

- Layout tetap satu, yaitu `layouts.app`, dan child view tetap memakai `@extends('layouts.app')` serta `@section('content')`.
- Daftar `/products` tetap memakai `$products` paginator, pencarian, category filter, sorting, dan pagination.
- Tidak ada perubahan pada Product fields, detail berbasis slug, SoftDeletes, `is_active`, trash, maupun cache dashboard.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: menambahkan flash message setelah mengubah dan menghapus product?**
