# Tahap 7 — Ringkasan dan Best Practice Flash Message

> Penutup materi 14: memberi user informasi singkat tanpa mengubah alur CRUD Product yang sudah selesai.

## Apa yang sudah dibuat

Flash message adalah pesan sementara setelah user melakukan tindakan. Pada materi ini, satu layout membaca pesan flash dan controller mengirimnya setelah mutasi berhasil.

Di `resources/views/layouts/app.blade.php`, blok yang sudah dibuat membaca dua jenis pesan:

```blade
@if (session('success'))
    <div role="status">
        {{ session('success') }}
    </div>
@endif

@if (session('error'))
    <div role="alert">
        {{ session('error') }}
    </div>
@endif
```

- `success` untuk tindakan yang berhasil.
- `error` untuk kegagalan umum yang sudah ditangani.
- Error validasi tetap tampil dekat field form menggunakan `@error`, bukan sebagai pesan sukses.

## Pola controller yang dipakai

Setelah perubahan Product benar-benar berhasil, gunakan urutan ini:

```php
// 1. Simpan, update, hapus, atau restore berhasil.
// 2. Hapus cache dashboard.
Cache::forget('dashboard.products.summary');

// 3. Redirect sambil membawa pesan sementara.
return redirect('/products')->with('success', 'Data berhasil disimpan');
```

Flash message dibuat pada redirect, bukan disimpan sebagai kolom di database. Laravel menampilkan pesan itu pada request berikutnya, lalu pesan hilang setelah dipakai.

## Pesan untuk setiap tindakan

| Method controller | Setelah berhasil | Halaman tujuan | Pesan |
| --- | --- | --- | --- |
| `store()` | Product dibuat, cache dihapus | `/products` | Data berhasil disimpan |
| `update()` | Product diperbarui, cache dihapus | `/products` | Data berhasil diperbarui |
| `destroy()` | Soft delete, cache dihapus | `/products` | Data berhasil dihapus |
| `restore()` | Product dipulihkan, cache dihapus | `/products/trash` | Product berhasil dikembalikan |

Contoh lengkap redirect setiap aksi:

```php
return redirect('/products')->with('success', 'Data berhasil disimpan');
return redirect('/products')->with('success', 'Data berhasil diperbarui');
return redirect('/products')->with('success', 'Data berhasil dihapus');
return redirect('/products/trash')->with('success', 'Product berhasil dikembalikan');
```

Gunakan hanya satu baris redirect yang sesuai di akhir masing-masing method, bukan semua baris sekaligus dalam satu method.

## Validasi gagal bukan success

Validasi berjalan sebelum `Product::create(...)` atau `$product->update(...)`.

```php
$validated = $request->validate([
    'name' => ['required', 'string'],
    'price' => ['required', 'numeric'],
    'stock' => ['required', 'integer'],
]);
```

Jika validasi gagal, Laravel kembali ke form dengan input lama dan error per field. Contoh Blade:

```blade
<input name="name" value="{{ old('name') }}">

@error('name')
    <p role="alert">{{ $message }}</p>
@enderror
```

Tidak ada Product yang dibuat, tidak ada cache dashboard yang dihapus, dan tidak ada pesan **Data berhasil disimpan**.

Jika kegagalan umum sudah ditangani dan user perlu diarahkan kembali, gunakan key `error`:

```php
return redirect('/products/create')->with('error', 'Terjadi kesalahan, silakan coba lagi');
```

Jangan memakai pesan umum itu untuk menggantikan error validasi per field.

## Checklist akhir

- [ ] Layout tunggal `layouts.app` memeriksa `session('success')` sebelum `@yield('content')`.
- [ ] Layout hanya menampilkan `session('error')` bila key tersebut ada.
- [ ] `store()` mengirim **Data berhasil disimpan** hanya setelah Product tersimpan dan cache dashboard dihapus.
- [ ] `update()` mengirim **Data berhasil diperbarui** hanya setelah perubahan berhasil dan cache dashboard dihapus.
- [ ] `destroy()` mengirim **Data berhasil dihapus** setelah soft delete dan invalidasi cache.
- [ ] `restore()` memakai `onlyTrashed()`, mengirim **Product berhasil dikembalikan**, dan menghapus cache sesudah restore berhasil.
- [ ] Error validasi memakai `@error` dan `old()`, bukan flash success.
- [ ] Pesan hanya menjelaskan hasil aksi, tidak menjalankan query atau mengubah data.

## Kontrak aplikasi tetap sama

Flash message hanya menambah informasi untuk user. Ia tidak mengubah kontrak aplikasi sebelumnya:

- Semua child view tetap memakai `@extends('layouts.app')` dan `@section('content')`.
- Daftar `/products` tetap memakai `$products` paginator, search, category filter, sort, pagination, dan query string.
- Detail memakai `/products/{{ $product->slug }}`. Edit, update, delete, dan restore memakai ID.
- Soft delete memakai `deleted_at`; status publikasi memakai `is_active`. Keduanya tetap berbeda.
- Dashboard tetap memakai `DashboardController@index` dengan cache `dashboard.products.summary`, TTL lima menit, dan query Product terbaru yang telah dibuat pada materi sebelumnya.

## Uji manual singkat

1. Buat product valid, lalu pastikan pesan **Data berhasil disimpan** muncul sekali di `/products`.
2. Edit product, lalu pastikan pesan **Data berhasil diperbarui** muncul sekali.
3. Delete product, lalu pastikan pesan **Data berhasil dihapus** muncul sekali dan product masuk trash.
4. Restore product dari `/products/trash`, lalu pastikan pesan **Product berhasil dikembalikan** muncul sekali.
5. Kirim form tanpa `name`, lalu pastikan error field muncul dan tidak ada pesan success.
6. Muat ulang halaman setelah setiap flash message. Pesan seharusnya sudah hilang.

Dengan pola ini, user selalu mengetahui hasil tindakan CRUD tanpa duplikasi kode notifikasi pada setiap view.
