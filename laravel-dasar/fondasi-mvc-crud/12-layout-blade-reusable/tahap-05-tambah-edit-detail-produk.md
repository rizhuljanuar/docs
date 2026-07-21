# Tahap 5 — Mengubah Halaman Tambah, Edit, dan Detail Produk

> Mentor: Laravel Dasar — Fondasi MVC, CRUD
> Topik: 12. Layout Blade Reusable
> Fokus tahap ini: **memindahkan 3 halaman sekaligus** (`create`, `edit`, `show`) + kasus khusus `@push('css')` dan judul dinamis.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Mengubah `produk/create.blade.php` (form tambah) agar pakai layout.
2. Mengubah `produk/edit.blade.php` (form edit) agar pakai layout.
3. Mengubah `produk/show.blade.php` (detail produk) agar pakai layout.
4. Membuat **judul tab dinamis** seperti "Detail Produk: Sepatu Lari".
5. Menambahkan **CSS khusus** per halaman lewat `@push('css')`.
6. Memperbaiki controller untuk `create`, `edit`, `show` mengirim `$title`.

Di tahap 4 kamu sudah paham polanya. Tahap ini **mengulang pola yang sama** untuk 3 halaman lain, dengan **dua hal baru**: judul dinamis dari data, dan `@push` untuk CSS khusus.

---

## 2. Analogi Sehari-Hari: Pindah Keluarga ke-2, ke-3, ke-4

Di tahap 4, kamu pindahkan **keluarga pertama** (Daftar Produk) ke rumah bapak (layout).

Sekarang tinggal pindahkan keluarga lain:

- Keluarga ke-2: **Form Tambah Produk** → pindah ke kamar "tambah".
- Keluarga ke-3: **Form Edit Produk** → pindah ke kamar "edit".
- Keluarga ke-4: **Detail Produk** → pindah ke kamar "detail".

Karena pola pemindahannya **sama** dengan tahap 4, kita bisa cepat. Tinggal hapus duplikat, bungkus unik dengan `@section`.

> 📝 **Pesan mentor:**
> Kalau tahap 4 terasa berat, tahap 5 akan terasa **lebih ringan** karena polanya sudah terbiasa. Inilah manfaat belajar bertahap: otak kamu sudah "magnet" untuk pola `@extends` + `@section`.

---

## 3. Bagian A: Halaman Tambah Produk (`produk/create.blade.php`)

### Bentuk lama (sebelum layout)

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <title>Tambah Produk</title>
    <style>
        form { max-width: 500px; margin: 0 auto; }
        label { display: block; margin-top: 10px; }
        input, textarea { width: 100%; padding: 8px; }
    </style>
</head>
<body>

    <header>
        <h1>Toko Bukhari</h1>
        <nav>...</nav>
    </header>

    <!-- ===== KONTEN UNIK ===== -->
    <h2>Tambah Produk Baru</h2>

    <form action="{{ route('produk.store') }}" method="POST" enctype="multipart/form-data">
        @csrf
        <label>Nama Produk</label>
        <input type="text" name="nama" value="{{ old('nama') }}">

        <label>Harga</label>
        <input type="number" name="harga" value="{{ old('harga') }}">

        <label>Deskripsi</label>
        <textarea name="deskripsi">{{ old('deskripsi') }}</textarea>

        <label>Gambar</label>
        <input type="file" name="gambar">

        @error('nama')
            <p style="color:red;">{{ $message }}</p>
        @enderror

        <button type="submit">Simpan</button>
    </form>

    <footer>...</footer>
</body>
</html>
```

### Tandai duplikat vs unik

```
┌──────────────────────────────────────────┐
│ <html> ... <head> ... <style>            │ ❌ duplikat (head) | ⚠ CSS unik
│ <header>...</header>                     │ ❌ duplikat
├──────────────────────────────────────────┤
│ <h2>Tambah Produk Baru</h2>              │ ✅ unik
│ <form>...</form>                         │ ✅ unik
├──────────────────────────────────────────┤
│ <footer>...</footer>                     │ ❌ duplikat
│ </body></html>                           │ ❌ duplikat
└──────────────────────────────────────────┘
```

**Catatan:** CSS untuk form (`<style>...`) ini **unik** untuk halaman form. Jadi tidak duplikat, tapi perlu **dipindahkan** ke `@push('css')` (lihat section 4).

### Bentuk baru (setelah pakai layout)

```blade
@extends('layout.app', ['title' => 'Tambah Produk'])

@push('css')
    <style>
        form { max-width: 500px; margin: 0 auto; }
        label { display:block; margin-top: 10px; }
        input, textarea { width: 100%; padding: 8px; }
    </style>
@endpush

@section('konten')
    <h2>Tambah Produk Baru</h2>

    <form action="{{ route('produk.store') }}" method="POST" enctype="multipart/form-data">
        @csrf
        <label>Nama Produk</label>
        <input type="text" name="nama" value="{{ old('nama') }}">

        <label>Harga</label>
        <input type="number" name="harga" value="{{ old('harga') }}">

        <label>Deskripsi</label>
        <textarea name="deskripsi">{{ old('deskripsi') }}</textarea>

        <label>Gambar</label>
        <input type="file" name="gambar">

        @error('nama')
            <p style="color:red;">{{ $message }}</p>
        @enderror

        <button type="submit">Simpan</button>
    </form>
@endsection
```

### Yang berubah

| Bagian lama | Bagian baru |
|---|---|
| `<!DOCTYPE html> ... <body>` | `@extends('layout.app', ...)` |
| `<header>...</header>` | **dihapus** (disumbang layout) |
| `<style>...</style>` di `<head>` | dipindah ke blok `@push('css') ... @endpush` |
| `<footer>...</footer>` | **dihapus** (disumbang layout) |
| `</body></html>` | **dihapus** |

### Controller `ProdukController@create` (opsional, kirim title)

```php
public function create()
{
    return view('produk.create', [
        'title' => 'Tambah Produk',
    ]);
}
```

Kalau sudah kirim `$title` dari controller, hapus `['title' => ...]` di `@extends`:

```blade
@extends('layout.app')
```

---

## 4. Penjelasan Baru: `@push('css') ... @endpush`

Inilah bagian baru di tahap 5. Di tahap 2, layout sudah menyiapkan:

```blade
<head>
    ...
    @stack('css')
</head>
```

Sekarang halaman form bisa **menambah (push)** CSS khusus ke tumpukan itu:

```blade
@push('css')
    <style>
        form { max-width: 500px; margin: 0 auto; }
        ...
    </style>
@endpush
```

### Cara kerja `@push` vs `@section`

| Directive | Sifat | Cocok untuk |
|---|---|---|
| `@section('konten')` | **menimpa** (overwrite) | konten utama (satu per halaman) |
| `@push('css')` | **menumpuk** (append) | CSS/JS tambahan (boleh banyak) |

### Analogi `@push` vs `@section`

- `@section('konten')` = **kotak pasir**. Kalau halaman mengisi sekali, sudah penuh. Isi lagi = timpa.
- `@push('css')` = **tumpukan buku**. Halaman A tambah 1 buku, halaman B tambah 1 buku, dst. Semua masuk ke tumpukan yang sama.

### Boleh ada banyak `@push`

Halaman bisa menulis **beberapa** `@push`:

```blade
@push('css')
    <style>...</style>
@endpush

@push('js')
    <script>...</script>
@endpush
```

CSS masuk ke `@stack('css')` di layout, JS masuk ke `@stack('js')`. Tidak bercampur.

> 🪤 **Jebakan pemula:**
> Lupa `@endpush`. Sama seperti `@endsection`, harus selalu ditutup. Kalau lupa, Blade error "push stack not closed".

---

## 5. Bagian B: Halaman Edit Produk (`produk/edit.blade.php`)

Polanya **sama persis** dengan halaman tambah. Bedanya: form sudah terisi data produk yang diedit.

### Bentuk baru (langsung)

```blade
@extends('layout.app', ['title' => 'Edit Produk'])

@push('css')
    <style>
        form { max-width: 500px; margin: 0 auto; }
        label { display: block; margin-top: 10px; }
        input, textarea { width: 100%; padding: 8px; }
    </style>
@endpush

@section('konten')
    <h2>Edit Produk: {{ $produk->nama }}</h2>

    <form action="{{ route('produk.update', $produk->id) }}" method="POST" enctype="multipart/form-data">
        @csrf
        @method('PUT')

        <label>Nama Produk</label>
        <input type="text" name="nama" value="{{ old('nama', $produk->nama) }}">

        <label>Harga</label>
        <input type="number" name="harga" value="{{ old('harga', $produk->harga) }}">

        <label>Deskripsi</label>
        <textarea name="deskripsi">{{ old('deskripsi', $produk->deskripsi) }}</textarea>

        <label>Gambar (kosongkan jika tidak diubah)</label>
        <input type="file" name="gambar">

        @if ($produk->gambar)
            <img src="{{ asset('storage/' . $produk->gambar) }}" width="100">
        @endif

        <button type="submit">Update</button>
    </form>
@endsection
```

### Yang berbeda dari halaman tambah

| Halaman tambah | Halaman edit |
|---|---|
| `route('produk.store')` | `route('produk.update', $produk->id)` |
| (tanpa `@method`) | `@method('PUT')` |
| `value="{{ old('nama') }}"` | `value="{{ old('nama', $produk->nama) }}"` |
| Judul "Tambah Produk Baru" | "Edit Produk: {nama produk}" |

### Controller `ProdukController@edit`

```php
public function edit($id)
{
    $produk = Produk::findOrFail($id);
    return view('produk.edit', [
        'produk' => $produk,
        'title'  => 'Edit Produk',
    ]);
}
```

---

## 6. Bagian C: Halaman Detail Produk (`produk/show.blade.php`)

Halaman detail menampilkan **satu produk** lengkap. Di sini kita pelajari **judul tab dinamis** dengan data produk.

### Bentuk baru

```blade
@extends('layout.app')

@section('konten')
    <h2>{{ $produk->nama }}</h2>

    @if ($produk->gambar)
        <img src="{{ asset('storage/' . $produk->gambar) }}" alt="{{ $produk->nama }}" width="300">
    @endif

    <p><strong>Harga:</strong> Rp {{ number_format($produk->harga, 0, ',', '.') }}</p>
    <p><strong>Deskripsi:</strong></p>
    <p>{{ $produk->deskripsi }}</p>

    <a href="{{ route('produk.index') }}">&larr; Kembali ke Daftar</a>
@endsection
```

### Kasus khusus: judul tab dinamis

Di halaman detail, judul tab sebaiknya **bukan** "Detail Produk" generik. Lebih baik: "Detail Produk: Sepatu Lari" (pakai nama produk).

**Kenapa?** Agar user bisa membedakan tab browser saat membuka banyak produk sekaligus.

**Cara:** Kirim `$title` dinamis dari controller:

```php
public function show($id)
{
    $produk = Produk::findOrFail($id);
    return view('produk.show', [
        'produk' => $produk,
        'title'  => 'Detail Produk: ' . $produk->nama,    // ← dinamis!
    ]);
}
```

Sekarang judul tab jadi **"Detail Produk: Sepatu Lari"** (atau apa pun nama produknya).

### Kenapa ini harus dari controller, bukan `@extends`?

Karena di controller kita punya akses ke variabel `$produk`. Di `@extends` Blade, kita tidak punya data itu langsung ( Blade tidak query database).

Inilah alasan di tahap 3 saya bilang: "judul dinamis butuh controller, judul tetap cukup `@extends`".

---

## 7. Ringkasan: 4 Pola Judul Tab yang Dipelajari

| Halaman | Pola judul | Lokasi kirim |
|---|---|---|
| Daftar Produk | tetap ("Daftar Produk") | controller atau `@extends` |
| Tambah Produk | tetap ("Tambah Produk") | controller atau `@extends` |
| Edit Produk | tetap ("Edit Produk") | controller atau `@extends` |
| Detail Produk | **dinamis** ("Detail Produk: {nama}") | **wajib controller** |

> 📝 **Pesan mentor:**
> Aturan praktis: kalau judul butuh **data dari database** → kirim dari controller. Kalau judul **tetap** per halaman → boleh dari `@extends` atau controller, sama saja.

---

## 8. Diagram: Tiga Halaman Baru Memakai Satu Layout

```
                ┌─────────────────────────────┐
                │  layout/app.blade.php       │
                │  (satu file untuk semua)    │
                │  ┌───────────────────────┐  │
                │  │ <header>              │  │
                │  │ <main>                │  │
                │  │   @yield('konten')    │  │ ← lubang diisi tiap halaman
                │  │ </main>               │  │
                │  │ <footer>              │  │
                │  └───────────────────────┘  │
                └─────────────────────────────┘
                          ▲
            ┌─────────────┼─────────────┐
            │             │             │
   ┌────────┴───────┐ ┌───┴────────┐ ┌──┴──────────┐
   │ create.blade   │ │ edit.blade │ │ show.blade  │
   │                │ │            │ │             │
   │ @extends       │ │ @extends   │ │ @extends    │
   │ @push('css')   │ │ @push('css')│ │             │
   │ @section       │ │ @section   │ │ @section    │
   │   <form        │ │   <form    │ │   <h2>nama  │
   │    tambah>     │ │    edit>   │ │   <img>     │
   │ @endsection    │ │ @endsection│ │ @endsection │
   └────────────────┘ └────────────┘ └─────────────┘
```

Semua tiga halaman kirim ke **satu layout yang sama**. Header/footer dari layout, konten unik dari halaman.

---

## 9. Troubleshooting Khusus Tahap 5

### Error 1: CSS form tidak muncul

**Penyebab:**
- Lupa `@push('css')`, atau
- Nama stack salah (`@push('style')` padahal layout `@stack('css')`), atau
- Layout belum punya `@stack('css')`.

**Solusi:** Cek nama stack **persis** `'css'` di kedua sisi (layout & halaman).

### Error 2: Form edit menampilkan field kosong

**Penyebab:** `value="{{ old('nama') }}"` tanpa fallback ke data produk.

**Solusi:** Pakai `old('nama', $produk->nama)` — artinya "pakai input lama jika ada, kalau tidak pakai data produk".

### Error 3: Judul tab di halaman detail tidak berubah

**Penyebab:** Controller `show()` belum kirim `$title`, atau `@extends` menimpa dengan nilai tetap.

**Solusi:** Pastikan controller `show()` kirim `'title' => 'Detail Produk: ' . $produk->nama`.

### Error 4: Method PUT tidak dikenali

**Penyebab:** Lupa `@method('PUT')` di form edit.

**Solusi:** Tambahkan `@method('PUT')` setelah `@csrf`. HTML form hanya support GET/POST; PUT/DELETE harus disimulasikan lewat `@method`.

### Error 5: `Undefined variable $produk`

**Penyebab:** Controller `edit()` atau `show()` belum kirim `$produk` ke view.

**Solusi:** Pastikan ada `'produk' => $produk` di array yang dikirim `view(...)`.

---

## 10. Latihan Mandiri

**Latihan C — Hapus duplikat:**

Berikut potongan kode `produk/show.blade.php` versi lama. Tandai bagian mana yang **harus dihapus** dan mana yang **disisakan**.

```blade
<!DOCTYPE html>
<html>
<head><title>Detail Produk</title></head>
<body>
    <header>
        <h1>Toko Bukhari</h1>
    </header>

    <h2>{{ $produk->nama }}</h2>
    <img src="{{ asset('storage/' . $produk->gambar) }}">
    <p>Harga: Rp {{ number_format($produk->harga) }}</p>

    <footer>&copy; 2026</footer>
</body>
</html>
```

<details>
<summary><strong>Lihat jawaban Latihan C</strong></summary>

**Hapus:**
- `<!DOCTYPE html>`, `<html>`, `<head>`, `<body>`
- `<header>...</header>`
- `<footer>...</footer>`
- `</body></html>`

**Sisakan (dibungkus `@section`):**
```blade
@extends('layout.app')

@section('konten')
    <h2>{{ $produk->nama }}</h2>
    <img src="{{ asset('storage/' . $produk->gambar) }}">
    <p>Harga: Rp {{ number_format($produk->harga) }}</p>
@endsection
```

</details>

---

## 11. Istilah Kunci Tahap Ini

| Istilah | Arti sederhana |
|---|---|
| **`@push('css')`** | menambah CSS ke tumpukan CSS di layout |
| **`@endpush`** | penutup `@push`, wajib ada |
| **`@stack('css')`** | lokasi tumpukan di layout (dipelajari tahap 2) |
| **Judul dinamis** | judul tab yang mengandung data dari database |
| **`old('nama', $default)`** | helper: pakai input lama, kalau tidak ada pakai nilai default |

---

## 12. Rangkuman Tahap 5

1. Tiga halaman (`create`, `edit`, `show`) dipindahkan ke layout dengan **pola yang sama** seperti `index`.
2. CSS khusus per halaman dipindah ke blok `@push('css') ... @endpush`.
3. `@push` bersifat **menumpuk** (append), beda dengan `@section` yang **menimpa** (overwrite).
4. Halaman edit pakai `old('field', $produk->field)` untuk fallback input lama → data produk.
5. Halaman detail butuh **judul dinamis** ("Detail Produk: {nama}") → **wajib** dikirim dari controller.
6. Setelah tahap ini, semua 4 halaman CRUD Produk sudah konsisten memakai layout.

---

## 13. Cek Pemahaman

1. Apa beda `@section('konten')` dan `@push('css')`?
2. Kenapa CSS form tidak ditulis langsung di layout, melainkan lewat `@push`?
3. Bagaimana cara membuat judul tab "Detail Produk: Sepatu Lari"?
4. Apa fungsi `old('nama', $produk->nama)` di form edit?
5. Apa yang terjadi kalau `@endpush` lupa ditulis?
6. Halaman mana yang **wajib** kirim `$title` dari controller (tidak bisa lewat `@extends`)?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. `@section` **menimpa** (overwrite) — satu isi per lubang. `@push` **menumpuk** (append) — banyak isi ke tumpukan yang sama.
2. Karena CSS form hanya dibutuhkan di halaman form, bukan di semua halaman. Kalau ditulis di layout, halaman lain (seperti daftar produk) ikut memuat CSS yang tidak dipakai — boros.
3. Kirim dari controller: `'title' => 'Detail Produk: ' . $produk->nama`. Tidak bisa lewat `@extends` karena Blade tidak punya akses data produk langsung.
4. Artinya: "pakai input user sebelumnya (saat validasi gagal), kalau tidak ada, pakai data dari `$produk->nama`".
5. Blade error "push stack not closed". Sama seperti lupa `@endsection`.
6. Halaman **detail produk** (`show`), karena judulnya dinamis (mengandung nama produk dari database).

</details>

---

## 14. Apakah Kamu Ingin Lanjut?

Di tahap 5 ini semua **4 halaman CRUD Produk** sudah memakai layout. Tersisa halaman dashboard.

Langkah berikutnya:

> ### "Apakah kamu ingin lanjut ke langkah berikutnya: mengubah halaman dashboard admin agar pakai layout?"
>
> Di tahap berikutnya kita akan:
>
> - mengubah `dashboard/index.blade.php`
> - membahas **kasus khusus**: dashboard sering punya **sidebar** yang berbeda dari halaman produk
> - solusi: **layout kedua** untuk admin (mis: `layout/admin.blade.php`) atau **slot opsional** di layout utama
> - mengirim data agregasi (count, sum) dari controller dashboard

Jawab: **"Ya, lanjut"** untuk ke tahap 6,
atau **"Ulangi tahap 5"** kalau ada bagian yang masih perlu diperdalam.

---

> 📚 **Daftar Tahap (12. Layout Blade Reusable):**
> - ✅ Tahap 1 — Apa itu layout Blade reusable
> - ✅ Tahap 2 — Membuat file layout utama Blade
> - ✅ Tahap 3 — `@extends` dan `@section` untuk memakai layout
> - ✅ Tahap 4 — Mengubah halaman daftar produk agar pakai layout
> - ✅ Tahap 5 — Mengubah halaman tambah, edit, detail produk (kamu di sini)
> - ⏳ Tahap 6 — Mengubah halaman dashboard admin
> - ⏳ Tahap 7 — Partial: memecah navbar dan footer
> - ⏳ Tahap 8 — Component: membuat komponen kartu produk
> - ⏳ Tahap 9 — Slot: lubang yang bisa diisi apa saja
> - ⏳ Tahap 10 — Ringkasan dan best practice
