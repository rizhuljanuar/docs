# Tahap 4 — Mengubah Halaman Daftar Produk Agar Pakai Layout

> Mentor: Laravel Dasar — Fondasi MVC, CRUD
> Topik: 12. Layout Blade Reusable
> Fokus tahap ini: **mengubah `produk/index.blade.php` nyata** agar memakai layout + kirim `$title` dari controller.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. **Membongkar** file Blade produk lama menjadi dua bagian (buang duplikat, sisakan konten unik).
2. **Mengubah** `produk/index.blade.php` agar memakai layout.
3. **Mengubah** controller `ProdukController@index` untuk mengirim variabel `$title`.
4. **Menguji** hasil di browser dan memastikan header/footer datang dari layout.

Di tahap ini kita fokus ke **satu halaman dulu**: daftar produk. Halaman `create`, `edit`, `show` akan diubah di tahap 5. Halaman dashboard di tahap 6.

---

## 2. Analogi Sehari-Hari: Pindah ke Rumah Bapak

Di tahap 3, kamu sudah pindah ke **rumah bapak** lewat halaman latihan.

Sekarang, kamu mau **pindahkan keluarga pertama**: keluarga "Daftar Produk".

Langkahnya:

1. **Kemas barang berharga** (konten unik daftar produk: tabel, tombol tambah).
2. **Buang barang duplikat** (header, footer, `<html>` — sudah disediakan rumah bapak).
3. **Pindah** (tulis `@extends` dan `@section`).
4. **Kirim label kamar** (`$title = 'Daftar Produk'` supaya tab browser sesuai).

---

## 3. Lihat Dulu Bentuk Lama `produk/index.blade.php`

Sebelum mengubah, mari kita lihat **bentuk lama** file `produk/index.blade.php` (versi tanpa layout). Ini kira-kira isinya:

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Daftar Produk</title>
    <style>
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ccc; padding: 8px; }
    </style>
</head>
<body>

    <!-- ===== HEADER (DUPLIKAT) ===== -->
    <header>
        <h1>Toko Bukhari</h1>
        <nav>
            <a href="{{ route('produk.index') }}">Produk</a>
            <a href="{{ route('dashboard.index') }}">Dashboard</a>
        </nav>
    </header>

    <!-- ===== KONTEN UNIK (HANYA ADA DI HALAMAN INI) ===== -->
    <h2>Daftar Produk</h2>
    <a href="{{ route('produk.create') }}">+ Tambah Produk</a>

    <table>
        <thead>
            <tr>
                <th>No</th>
                <th>Nama Produk</th>
                <th>Harga</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
            @foreach ($produk as $item)
                <tr>
                    <td>{{ $loop->iteration }}</td>
                    <td>{{ $item->nama }}</td>
                    <td>Rp {{ number_format($item->harga, 0, ',', '.') }}</td>
                    <td>
                        <a href="{{ route('produk.show', $item->id) }}">Detail</a>
                        <a href="{{ route('produk.edit', $item->id) }}">Edit</a>
                        <form action="{{ route('produk.destroy', $item->id) }}" method="POST" style="display:inline;">
                            @csrf
                            @method('DELETE')
                            <button type="submit">Hapus</button>
                        </form>
                    </td>
                </tr>
            @endforeach
        </tbody>
    </table>

    <!-- ===== FOOTER (DUPLIKAT) ===== -->
    <footer>
        <p>&copy; {{ date('Y') }} Toko Bukhari. Semua hak dilindungi.</p>
    </footer>

</body>
</html>
```

### Tandai bagian yang mana

Sekarang mari **tandai** bagian mana yang duplikat vs unik:

```
┌──────────────────────────────────────────┐
│ <!DOCTYPE html> ... <body>               │ ❌ DUPLIKAT (sudah ada di layout)
├──────────────────────────────────────────┤
│ <header>...                              │ ❌ DUPLIKAT (sudah ada di layout)
├──────────────────────────────────────────┤
│                                          │
│   <h2>Daftar Produk</h2>                 │ ✅ UNIK (hanya ada di halaman ini)
│   <a href="...">+ Tambah Produk</a>      │ ✅ UNIK
│   <table>...daftar produk...</table>     │ ✅ UNIK
│                                          │
├──────────────────────────────────────────┤
│ <footer>...                              │ ❌ DUPLIKAT (sudah ada di layout)
├──────────────────────────────────────────┤
│ </body></html>                           │ ❌ DUPLIKAT
└──────────────────────────────────────────┘
```

Hanya **bagian tengah** (h2 + tombol tambah + tabel) yang **benar-benar unik** untuk halaman daftar produk. Sisanya, buang.

---

## 4. Langkah 1: Identifikasi Bagian Unik

**Bagian unik** yang harus disisakan di `produk/index.blade.php`:

```blade
<h2>Daftar Produk</h2>
<a href="{{ route('produk.create') }}">+ Tambah Produk</a>

<table>
    <thead>
        <tr>
            <th>No</th>
            <th>Nama Produk</th>
            <th>Harga</th>
            <th>Aksi</th>
        </tr>
    </thead>
    <tbody>
        @foreach ($produk as $item)
            <tr>
                <td>{{ $loop->iteration }}</td>
                <td>{{ $item->nama }}</td>
                <td>Rp {{ number_format($item->harga, 0, ',', '.') }}</td>
                <td>
                    <a href="{{ route('produk.show', $item->id) }}">Detail</a>
                    <a href="{{ route('produk.edit', $item->id) }}">Edit</a>
                    <form action="{{ route('produk.destroy', $item->id) }}" method="POST" style="display:inline;">
                        @csrf
                        @method('DELETE')
                        <button type="submit">Hapus</button>
                    </form>
                </td>
            </tr>
        @endforeach
    </tbody>
</table>
```

**Hanya ini.** Tidak ada `<html>`, `<head>`, `<header>`, `<footer>`. Bagian itu disumbang layout.

---

## 5. Langkah 2: Bungkus Bagian Unik dengan `@extends` dan `@section`

Sekarang bungkus bagian unik itu dengan dua directive yang sudah kamu pelajari:

```blade
@extends('layout.app', ['title' => 'Daftar Produk'])

@section('konten')
    <h2>Daftar Produk</h2>
    <a href="{{ route('produk.create') }}">+ Tambah Produk</a>

    <table>
        <thead>
            <tr>
                <th>No</th>
                <th>Nama Produk</th>
                <th>Harga</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
            @foreach ($produk as $item)
                <tr>
                    <td>{{ $loop->iteration }}</td>
                    <td>{{ $item->nama }}</td>
                    <td>Rp {{ number_format($item->harga, 0, ',', '.') }}</td>
                    <td>
                        <a href="{{ route('produk.show', $item->id) }}">Detail</a>
                        <a href="{{ route('produk.edit', $item->id) }}">Edit</a>
                        <form action="{{ route('produk.destroy', $item->id) }}" method="POST" style="display:inline;">
                            @csrf
                            @method('DELETE')
                            <button type="submit">Hapus</button>
                        </form>
                    </td>
                </tr>
            @endforeach
        </tbody>
    </table>
@endsection
```

### Apa yang berubah?

| Bagian lama | Bagian baru |
|---|---|
| `<!DOCTYPE html> ... <body>` (7 baris) | `@extends('layout.app', ['title' => 'Daftar Produk'])` (1 baris) |
| `<header>...</header>` (8 baris) | **dihapus** (disumbang layout) |
| Konten unik (h2 + tombol + tabel) | **dibungkus** `@section('konten') ... @endsection` |
| `<footer>...</footer>` (3 baris) | **dihapus** (disumbang layout) |
| `</body></html>` | **dihapus** |

Total: dari ~50 baris menjadi ~30 baris. **Lebih pendek, lebih fokus.**

### Penjelasan baris penting

**Baris 1:** `@extends('layout.app', ['title' => 'Daftar Produk'])`

- Mewarisi layout `layout/app.blade.php`.
- Mengirim variabel `$title` dengan nilai `'Daftar Produk'`, supaya judul tab browser jadi "Daftar Produk".

**Baris 3:** `@section('konten')`

- Memulai blok konten yang akan dimasukkan ke lubang `@yield('konten')` di layout.

**Baris 4-32:** konten unik

- Bebas. Ini yang akan tampil di `<main>` layout.

**Baris 33:** `@endsection`

- Menutup section. Jangan lupa.

---

## 6. Langkah 3: Update Controller untuk Kirim `$title`

Sebenarnya di langkah 2 kita **sudah** mengirim `$title` lewat `@extends(...)`, jadi ini **opsional**. Tapi cara yang **lebih umum** di Laravel adalah kirim `$title` dari controller.

### Kenapa dari controller lebih baik?

Karena di controller kita bisa pakai **data dinamis**, misalnya:

```php
'title' => 'Detail Produk: ' . $produk->nama
```

Tidak bisa begini di `@extends` karena Blade tidak punya akses langsung ke logika controller.

### Buka `ProdukController.php`

```bash
app/Http/Controllers/ProdukController.php
```

Cari method `index()`:

```php
public function index()
{
    $produk = Produk::all();
    return view('produk.index', compact('produk'));
}
```

Ubah jadi:

```php
public function index()
{
    $produk = Produk::all();
    return view('produk.index', [
        'produk' => $produk,
        'title'  => 'Daftar Produk',
    ]);
}
```

### Penjelasan perubahan

| Kode lama | Kode baru |
|---|---|
| `compact('produk')` | array eksplisit `['produk' => $produk, 'title' => 'Daftar Produk']` |

- `compact('produk')` = cara singkat untuk `['produk' => $produk]`.
- Sekarang kita butuh **dua variabel**, jadi lebih jelas pakai array eksplisit.

### Kalau sudah kirim `$title` dari controller, hapus dari Blade

Karena `$title` sekarang dikirim controller, **boleh hapus** dari `@extends`:

```blade
@extends('layout.app')   ← tanpa ['title' => ...] lagi

@section('konten')
    ...
@endsection
```

### Kesimpulan: pilih salah satu

| Cara | Lokasi |
|---|---|
| **A:** `@extends('layout.app', ['title' => 'Daftar Produk'])` | di Blade saja |
| **B:** kirim dari controller + hapus `['title' => ...]` di Blade | di controller saja |

**Jangan keduanya** untuk variabel yang sama. Pilih satu. Rekomendasi: **cara B** (controller) karena lebih konsisten dengan variabel lain seperti `$produk`.

---

## 7. Langkah 4: Hapus Duplikat dari File Lama

Saat mengubah `produk/index.blade.php`, **pastikan** kamu:

1. ✅ **Menghapus** `<!DOCTYPE html>`, `<html>`, `<head>`, `<body>`.
2. ✅ **Menghapus** `<header>...</header>` (sudah ada di layout).
3. ✅ **Menghapus** `<footer>...</footer>` (sudah ada di layout).
4. ✅ **Menghapus** `</body></html>` di akhir.
5. ✅ **Menyisakan** hanya konten unik yang dibungkus `@section('konten')`.

> 🪤 **Jebakan pemula:**
> Lupa menghapus `<header>` lama. Akibatnya halaman punya **dua header** (satu dari layout, satu dari file lama). Tampilan jadi aneh dengan dua logo "Toko Bukhari".

---

## 8. Langkah 5: Uji di Browser

Saatnya melihat hasilnya.

1. Pastikan server Laravel jalan: `php artisan serve`.
2. Buka browser ke: `http://localhost:8000/produk`.

### Yang harus terlihat

| Elemen | Sumber | Status |
|---|---|---|
| Judul tab browser: "Daftar Produk" | dari `$title` ke layout | ✅ |
| Header "Toko Bukhari" + menu di atas | dari layout | ✅ |
| Judul halaman "Daftar Produk" (`<h2>`) | dari halaman produk | ✅ |
| Tombol "+ Tambah Produk" | dari halaman produk | ✅ |
| Tabel daftar produk | dari halaman produk | ✅ |
| Footer "© 2026 Toko Bukhari..." | dari layout | ✅ |

Kalau **semuanya muncul**, selamat! Halaman daftar produk berhasil dipindahkan ke layout.

### Uji konsistensi (kekuatan layout)

Sekarang bukti terbesar manfaat layout:

1. Buka file `layout/app.blade.php`.
2. Ubah tulisan "Toko Bukhari" di `<h1>` jadi "Toko Berkah Jaya".
3. Save.
4. **Refresh** halaman `/produk` di browser (tanpa ubah `produk/index.blade.php`).

Header harus berubah jadi "Toko Berkah Jaya". **Satu file diubah → semua halaman ikut berubah.** Itulah kekuatan layout.

> 📝 **Pesan mentor:**
> Inilah "moment aha" pertama kamu. Sekarang setiap kali kita ubah layout, halaman produk (dan nanti dashboard, tambah, edit, detail) **otomatis** ikut berubah tanpa diutak-atik satu per satu.

---

## 9. Troubleshooting 5 Error Paling Sering

### Error 1: Halaman tampil, tapi ada **dua header**

**Penyebab:** `<header>` lama belum dihapus dari `produk/index.blade.php`.

**Solusi:** Hapus semua blok `<header>...</header>` dari file produk. Cukup pakai header dari layout.

### Error 2: Konten tabel produk **tidak muncul**

**Penyebab:**
- Lupa menulis `@section('konten')`, atau
- Nama section salah (`@section('isi')` padahal layout `@yield('konten')`), atau
- Lupa `@endsection`.

**Solusi:** Cek nama `@section` **persis** `'konten'` dan pastikan `@endsection` menutup blok.

### Error 3: Header/footer **hilang**

**Penyebab:** Lupa `@extends('layout.app')` di baris pertama.

**Solusi:** Tambahkan `@extends('layout.app')` di **baris paling atas**.

### Error 4: `Undefined variable $title`

**Penyebab:**
- Layout menulis `{{ $title }}` tanpa `?? '...'`, dan
- Controller **belum** kirim `$title`, dan
- `@extends` tidak kirim `$title` juga.

**Solusi:** Pilih salah satu: (a) tambah `?? 'Toko Bukhari'` di layout, (b) kirim `$title` dari controller, atau (c) kirim lewat `@extends`.

### Error 5: `Undefined variable $produk`

**Penyebab:** Controller tidak mengirim variabel `$produk` ke view.

**Solusi:** Pastikan `ProdukController@index` menulis `view('produk.index', ['produk' => $produk, ...])`.

---

## 10. Diagram Sebelum vs Sesudah

**SEBELUM** layout:

```
produk/index.blade.php (50 baris)
├── <!DOCTYPE html>          ❌ duplikat
├── <head>                   ❌ duplikat
├── <header>                 ❌ duplikat
├── <h2>Daftar Produk</h2>   ✅ unik
├── <table>...</table>       ✅ unik
├── <footer>                 ❌ duplikat
└── </body></html>           ❌ duplikat
```

**SESUDAH** layout:

```
produk/index.blade.php (30 baris)
├── @extends('layout.app')
└── @section('konten')
       ├── <h2>Daftar Produk</h2>   ✅ unik
       ├── <a>+ Tambah Produk</a>   ✅ unik
       └── <table>...</table>       ✅ unik
    @endsection
```

Plus layout sendiri:

```
layout/app.blade.php (1 file untuk semua halaman)
├── <html><head>
├── <header>
├── @yield('konten')   ← diisi tiap halaman
├── <footer>
└── </html>
```

---

## 11. Latihan Mandiri (Sebelum Lanjut)

**Latihan B:**

Sebelum lanjut ke tahap 5, coba uji pemahaman dengan latihan ringan ini.

Misalkan kamu punya halaman "Daftar Kategori" di `kategori/index.blade.php`. Sketsa saja (tidak perlu benar-benar dibuat):

1. Bagaimana tulisan `@extends` untuk halaman itu dengan judul "Daftar Kategori"?
2. Bagaimana bungkus konten tabel kategori dengan `@section`?
3. Bagaimana controller `KategoriController@index` mengirim `$title`?

<details>
<summary><strong>Lihat jawaban Latihan B</strong></summary>

**Blade** (`kategori/index.blade.php`):

```blade
@extends('layout.app')

@section('konten')
    <h2>Daftar Kategori</h2>
    <table>
        <!-- tabel kategori -->
    </table>
@endsection
```

**Controller** (`KategoriController.php`):

```php
public function index()
{
    $kategori = Kategori::all();
    return view('kategori.index', [
        'kategori' => $kategori,
        'title'    => 'Daftar Kategori',
    ]);
}
```

</details>

---

## 12. Istilah Kunci Tahap Ini

| Istilah | Arti sederhana |
|---|---|
| **Duplikat** | bagian kode yang sama di banyak file (header, footer) |
| **Bagian unik** | konten khas halaman tertentu (tabel produk, form, dll.) |
| **Pemetaan** | menandai bagian mana yang dibuang, mana yang disisakan |
| **`compact()`** | helper PHP untuk membuat array dari nama variabel (mis: `compact('produk')`) |

---

## 13. Rangkuman Tahap 4

1. **Identifikasi** bagian unik vs duplikat di file Blade lama.
2. **Buang** duplikat (`<html>`, `<head>`, `<header>`, `<footer>`).
3. **Bungkus** bagian unik dengan `@extends('layout.app')` + `@section('konten') ... @endsection`.
4. **Kirim `$title`** dari controller (atau lewat `@extends`).
5. **Uji** di browser → header, konten, footer harus muncul konsisten.
6. **Buktikan kekuatan layout**: ubah layout → semua halaman ikut berubah.

---

## 14. Cek Pemahaman (Jawab Sendiri Dulu)

1. Bagian mana dari `produk/index.blade.php` yang **harus dihapus** saat beralih ke layout?
2. Bagian mana yang **harus disisakan**?
3. Apa fungsi `@extends('layout.app')` di baris pertama file produk?
4. Bagaimana cara mengirim `$title` dari controller?
5. Apa risiko kalau `<header>` lama lupa dihapus dari file produk?
6. Bagaimana cara membuktikan bahwa layout bekerja untuk banyak halaman?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. Hapus: `<!DOCTYPE html>`, `<html>`, `<head>`, `<body>`, `<header>`, `<footer>`, `</body></html>`. Semua itu disumbang layout.
2. Sisakan: konten unik saja (h2 "Daftar Produk", tombol "+ Tambah Produk", tabel produk).
3. Memberitahu Blade bahwa halaman ini **mewarisi layout** `layout/app.blade.php`. Blade akan otomatis menyisipkan header, footer, dll.
4. Tambahkan key `'title' => 'Daftar Produk'` di array yang dikirim via `view('produk.index', [...])` di controller.
5. Halaman akan menampilkan **dua header**: satu dari layout, satu dari file produk. Tampilan jadi aneh dan duplikat.
6. Ubah satu hal di layout (misalnya teks header), lalu refresh semua halaman yang memakai layout. Semua harus ikut berubah tanpa diutak-atik satu per satu.

</details>

---

## 15. Apakah Kamu Ingin Lanjut?

Di tahap 4 ini **satu halaman** (daftar produk) sudah berhasil dipindahkan ke layout. Saatnya lanjut ke halaman lain.

Langkah berikutnya:

> ### "Apakah kamu ingin lanjut ke langkah berikutnya: mengubah halaman tambah, edit, dan detail produk?"
>
> Di tahap berikutnya kita akan:
>
> - mengubah `produk/create.blade.php` (form tambah)
> - mengubah `produk/edit.blade.php` (form edit)
> - mengubah `produk/show.blade.php` (detail produk)
> - membahas **kasus khusus**: detail produk dengan judul dinamis (`'Detail Produk: ' . $produk->nama`)
> - membahas **`@push('css')`** untuk menambah CSS khusus form

Jawab: **"Ya, lanjut"** untuk ke tahap 5,
atau **"Ulangi tahap 4"** kalau ada bagian yang masih perlu diperdalam.

---

> 📚 **Daftar Tahap (12. Layout Blade Reusable):**
> - ✅ Tahap 1 — Apa itu layout Blade reusable
> - ✅ Tahap 2 — Membuat file layout utama Blade
> - ✅ Tahap 3 — `@extends` dan `@section` untuk memakai layout
> - ✅ Tahap 4 — Mengubah halaman daftar produk agar pakai layout (kamu di sini)
> - ⏳ Tahap 5 — Mengubah halaman tambah, edit, detail produk
> - ⏳ Tahap 6 — Mengubah halaman dashboard admin
> - ⏳ Tahap 7 — Partial: memecah navbar dan footer
> - ⏳ Tahap 8 — Component: membuat komponen kartu produk
> - ⏳ Tahap 9 — Slot: lubang yang bisa diisi apa saja
> - ⏳ Tahap 10 — Ringkasan dan best practice
