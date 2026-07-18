# Tahap 4 — Form Pencarian di View Blade

> Materi Laravel Dasar · Topik 6: Pencarian Produk
> Mentor: penjelasan bertahap untuk pemula total
> Prasyarat: sudah baca **Tahap 1, 2, dan 3**

---

## 1. Goal Tahap Ini

Di akhir Tahap 4 ini, kamu diharapkan:

- Pahami kenapa form pencarian pakai **`method="GET"`**, bukan `POST`
- Bikin **kotak pencarian** (input text + tombol submit) di file Blade
- Hubungkan form ke route `produk.index`
- Tampilkan **kembali** kata kunci yang user ketik (biar kotak tidak kosong setelah submit)
- Bisa uji langsung di browser: ketik → Enter → hasil tersaring muncul

**Tidak ada perubahan Controller di tahap ini.** Cukup edit file view saja.

---

## 2. Asumsi Kondisi Awal

### File: `resources/views/produk/index.blade.php` (SEBELUM ada pencarian)

```html
@extends('layouts.app')

@section('content')
    <h1>Daftar Produk</h1>

    <table border="1">
        <thead>
            <tr>
                <th>Nama</th>
                <th>Harga</th>
                <th>Stok</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
            @foreach($produks as $produk)
                <tr>
                    <td>{{ $produk->nama }}</td>
                    <td>{{ $produk->harga }}</td>
                    <td>{{ $produk->stok }}</td>
                    <td>
                        <a href="{{ route('produk.edit', $produk->id) }}">Edit</a>
                        <form action="{{ route('produk.destroy', $produk->id) }}" method="POST" style="display:inline">
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

Kita akan **menambahkan satu blok form** di atas tabel, tanpa mengubah struktur yang sudah ada.

---

## 3. Konsep Penting: `GET` vs `POST` untuk Pencarian

Banyak pemula bingung: form itu biasanya `method="POST"`, kenapa untuk pencarian pakai `GET`?

### Aturan Praktis

| Maksud Form            | Method | Kenapa                                   |
|------------------------|--------|-------------------------------------------|
| **Mencari / menyaring**| `GET`  | Hasil bisa di-**bookmark**, di-**share** |
| **Menyimpan data baru**| `POST` | Data sensitif, tidak tampil di URL       |
| **Mengubah data**      | `POST` | Data perlu dikirim tersembunyi           |
| **Menghapus data**     | `POST` | Sama, perlu tersembunyi                  |

### Analogi

- **GET** = kamu tulis pesanan di **papan tulisan** di depan kasir. Semua orang bisa baca, kasir tinggal lihat. Cocok untuk hal yang **tidak rahasia** (seperti kata kunci pencarian).
- **POST** = kamu sampaikan pesanan **bisik-bisik** ke kasir. Tidak ada yang dengar. Cocok untuk hal yang **rahasia** (seperti password).

### Ciri khas GET

Saat pakai `GET`, data form **muncul di URL**:

```
/produk?search=laptop
```

Ini bagus untuk pencarian karena:

1. **Bisa di-bookmark** → user simpan link hasil pencarian "laptop" untuk dipakai lain kali.
2. **Bisa di-share** → user kirim link ke teman: *"lihat, hasil pencarian laptop di toko ini."*
3. **Bisa di-refresh** tanpa peringatan "Confirm Form Resubmission" (yang muncul di POST).
4. **Bisa di-back** dengan tombol back browser tanpa error.

---

## 4. Struktur Dasar Form GET

```html
<form action="{{ route('produk.index') }}" method="GET">
    @csrf
    <input type="text" name="search" placeholder="Cari produk...">
    <button type="submit">Cari</button>
</form>
```

Penjelasan tiap bagian:

| Bagian                                   | Fungsi                                                                 |
|------------------------------------------|-------------------------------------------------------------------------|
| `<form action="...">`                    | Tujuan form dikirim. Isi: URL route `produk.index`.                    |
| `method="GET"`                           | Form dikirim sebagai GET (data muncul di URL).                          |
| `@csrf`                                  | Token keamanan Laravel (meski GET sebenarnya tidak wajib, tetap aman). |
| `<input type="text" name="search">`      | Kotak teks untuk user mengetik kata kunci.                             |
| `name="search"`                          | **Penting**: harus sama dengan `$request->input('search')` di Controller. |
| `placeholder="Cari produk..."`           | Teks samar di kotak (menghilang saat user mulai mengetik).             |
| `<button type="submit">Cari</button>`    | Tombol untuk mengirim form.                                            |

### Yang menghubungkan Form dan Controller

Satu hal kunci: **`name="search"`** di form HARUS sama dengan **`->input('search')`** di Controller.

```
Form:            name="search"
                     ↓
URL jadi:        /produk?search=laptop
                     ↓
Controller:      $request->input('search')   →  "laptop"
```

Kalau `name="q"`, maka Controller juga harus `->input('q')`. Konsisten.

---

## 5. Langkah Kecil #1: Tambahkan Form di Atas Tabel

Edit `resources/views/produk/index.blade.php`, tambahkan blok form **sebelum** `<table>`:

```html
@extends('layouts.app')

@section('content')
    <h1>Daftar Produk</h1>

    {{-- BLOK PENCARIAN --}}
    <form action="{{ route('produk.index') }}" method="GET">
        @csrf
        <input type="text" name="search" placeholder="Cari produk...">
        <button type="submit">Cari</button>
    </form>
    {{-- END BLOK PENCARIAN --}}

    <table border="1">
        {{-- ... struktur tabel tetap sama ... --}}
    </table>
@endsection
```

**Sekarang tes di browser:**

1. Buka `/produk`.
2. Ketik `laptop` di kotak pencarian.
3. Tekan Enter atau klik tombol "Cari".
4. URL berubah jadi `/produk?search=laptop`.
5. Tabel seharusnya hanya menampilkan produk yang namanya mengandung "laptop".

Kalau berhasil, **selamat**, fitur pencarian dasar sudah jalan.

---

## 6. Masalah #1: Kotak Pencarian Kosong Setelah Submit

Coba lakukan ini:

1. Ketik `laptop` di kotak.
2. Tekan Enter.
3. Perhatikan kotaknya → **kosong**, padahal hasil pencarian "laptop" masih tampil di tabel.

Kenapa? Karena kita tidak **mengisi kembali** nilai kotak dengan kata kunci yang user ketik tadi.

### Solusi: Pakai `old()` atau nilai dari request

Di Blade, kita bisa akses kata kunci lewat:

```php
request('search')
```

Ini cara cepat Laravel untuk membaca nilai `?search=...` dari URL langsung di view.

Update input:

```html
<input type="text" name="search"
       value="{{ request('search') }}"
       placeholder="Cari produk...">
```

Sekarang setelah submit, kotak akan **tetap berisi** "laptop". Lebih ramah user.

---

## 7. Masalah #2: Tombol "Reset" / "Lihat Semua"

Setelah user mencari, dia mungkin ingin **melihat semua produk lagi** (tanpa filter).
Kalau tidak ada tombol reset, user harus:

1. Hapus teks di kotak.
2. Tekan Enter lagi.

Ribet. Kita tambahkan tombol reset yang **sederhana** berupa link:

```html
<a href="{{ route('produk.index') }}">Reset</a>
```

Ini bukan tombol form, hanya link biasa ke `/produk` (tanpa `?search=`), jadi ** semua produk tampil lagi**.

---

## 8. Kode Lengkap View Setelah Pencarian

```html
@extends('layouts.app')

@section('content')
    <h1>Daftar Produk</h1>

    {{-- FORM PENCARIAN --}}
    <form action="{{ route('produk.index') }}" method="GET">
        @csrf
        <input type="text"
               name="search"
               value="{{ request('search') }}"
               placeholder="Cari produk...">
        <button type="submit">Cari</button>
        <a href="{{ route('produk.index') }}">Reset</a>
    </form>
    <br>

    {{-- TABEL PRODUK --}}
    <table border="1">
        <thead>
            <tr>
                <th>Nama</th>
                <th>Harga</th>
                <th>Stok</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
            @foreach($produks as $produk)
                <tr>
                    <td>{{ $produk->nama }}</td>
                    <td>{{ $produk->harga }}</td>
                    <td>{{ $produk->stok }}</td>
                    <td>
                        <a href="{{ route('produk.edit', $produk->id) }}">Edit</a>
                        <form action="{{ route('produk.destroy', $produk->id) }}"
                              method="POST" style="display:inline">
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

---

## 9. Hal Kecil yang Sering Bikin Pemula Bingung

### A. Kenapa pakai `@csrf` di form GET?

Di Laravel, `@csrf` sebenarnya **wajib untuk POST**, tapi untuk GET biasanya tidak diwajibkan.
Boleh kamu **hapus** `@csrf` di form GET. Tapi kalau dibiarkan, juga tidak salah (hanya extra field `_token` di URL).

### B. Kenapa tombol "Reset" pakai `<a>`, bukan `<button>`?

Karena reset = **pindah ke URL baru** (tanpa `?search=`), itu lebih cocok sebagai link (`<a href>`), bukan submit form.

Kalau pakai `<button>` biasa (tanpa `type="submit"` dan tanpa JavaScript), tidak akan melakukan apa-apa.

### C. Form di dalam form itu dilarang di HTML

Di kode atas, ada form hapus produk **di dalam tabel** (baris per produk).
Itu form terpisah, **tidak boleh bersarang di dalam form pencarian**.

Struktur HTML yang benar:

```
<form pencarian>  ← di luar
    ...
</form>

<table>
    @foreach(...)
        <form hapus>  ← form terpisah, di dalam <td>
        </form>
    @endforeach
</table>
```

Jangan pernah **menumpuk** `<form>` di dalam `<form>`.

### D. `request('search')` vs `Request $request`

Di Controller kita pakai:

```php
public function index(Request $request)
{
    $keyword = $request->input('search');
}
```

Di Blade kita pakai:

```php
request('search')
```

Bedanya:

- `Request $request` (Controller) → versi **injeksi class**, lebih cocok untuk logic kompleks.
- `request(...)` (Blade) → versi **helper global**, lebih singkat untuk dipakai di view.

Keduanya membaca **sumber yang sama** (URL / form), jadi hasilnya sama.

---

## 10. Uji Coba Lengkap

Lakukan skenario berikut untuk memastikan fitur pencarian jalan:

| # | Aksi                                              | Hasil Diharapkan                                |
|---|---------------------------------------------------|--------------------------------------------------|
| 1 | Buka `/produk`                                    | Semua produk tampil, kotak pencarian kosong.     |
| 2 | Ketik `laptop`, klik "Cari"                       | Tabel hanya berisi produk mengandung "laptop".   |
| 3 | Periksa kotak pencarian setelah submit            | Kotak **tetap berisi** "laptop".                  |
| 4 | Lihat URL                                         | Berubah jadi `/produk?search=laptop`.            |
| 5 | Hapus teks di kotak, klik "Cari"                  | Semua produk tampil lagi (kata kunci kosong).    |
| 6 | Klik "Reset"                                      | Sama seperti no. 5, URL jadi `/produk` bersih.   |
| 7 | Bookmark URL `/produk?search=laptop`              | Bisa dibuka lagi nanti, hasil pencarian sama.    |
| 8 | Ketik kata yang tidak ada, misal `xyz`            | Tabel kosong (tidak ada produk cocok).            |

Kalau semua berhasil, **fitur pencarian dasar kamu sudah lengkap dan user-friendly.**

---

## 11. Apa yang Masih Kurang?

Coba lakukan ini: di Controller kamu, edit method `index()` jadi begini (versi saat ini dari Tahap 3):

```php
public function index(Request $request)
{
    $keyword = $request->input('search');

    $produks = Produk::where('nama', 'like', '%' . $keyword . '%')->get();

    return view('produk.index', compact('produks'));
}
```

Sekarang bayangkan kebutuhan baru:

- Pencarian harus **bisa berdasarkan nama ATAU deskripsi**
- Harus bisa **filter berdasarkan kategori** juga (misal: "Laptop + kategori Elektronik")
- Logic pencarian ini mau dipakai di **halaman admin** dan **halaman publik** (duplicate code!)

Kalau semua itu kita tulis langsung di Controller, kode akan jadi **panjang, berantakan, dan sulit dipakai ulang**.

Inilah masalah yang akan kita selesaikan di **Tahap 5** dengan **Query Scope**.

---

## 12. Kesimpulan Tahap 4

- Form pencarian pakai **`method="GET"`** supaya kata kunci muncul di URL (bookmark-able, share-able).
- Atribut **`name="search"`** di form HARUS cocok dengan `->input('search')` di Controller.
- `value="{{ request('search') }}"` membuat kotak pencarian **tetap berisi** kata kunci setelah submit.
- Tombol "Reset" cukup pakai `<a href="{{ route('produk.index') }}">` (link ke URL tanpa query).
- Jangan menumpuk `<form>` di dalam `<form>`.
- `request(...)` di Blade = helper cepat, sama dengan `$request->input(...)` di Controller.

Sekarang fitur pencarian dasar kamu sudah **berfungsi penuh**. Tapi kode Controller masih bisa lebih rapi dan reusable. Itu topik Tahap 5.

---

## Pertanyaan Berikutnya

**Apakah kamu ingin lanjut ke langkah berikutnya: memisahkan logic filter ke Method Query Scope?**

Pada Tahap 5 kita akan belajar:

- Apa itu **Query Scope** di Laravel (dan kenapa berguna)
- Bikin method terpisah di **Model Produk** untuk menangani pencarian
- Refactor Controller supaya lebih bersih: hanya 1 baris pemanggilan
- Mengatasi masalah `$keyword` kosong dengan rapi

— **Mentor Laravel**
