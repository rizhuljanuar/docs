# Tahap 3 — `@extends` dan `@section` untuk Memakai Layout

> Mentor: Laravel Dasar — Fondasi MVC, CRUD
> Topik: 12. Layout Blade Reusable
> Fokus tahap ini: **memahami cara halaman "mewarisi" layout dan mengisi lubang `@yield`**.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Menjelaskan apa itu **directive `@extends`** dan fungsinya.
2. Menjelaskan apa itu **directive `@section` ... `@endsection`**.
3. Menjelaskan hubungan **`@yield` (di layout) ↔ `@section` (di halaman)**.
4. Menulis halaman Blade sederhana yang memakai layout.
5. Mengirim **variabel judul** (`$title`) ke layout.

Di tahap ini kita **belum mengubah halaman produk atau dashboard**. Kita akan berlatih dengan **satu halaman latihan** dulu (`latihan.blade.php`) agar kamu paham pola `@extends` + `@section` sebelum menyentuh file asli.

---

## 2. Analogi Sehari-Hari: Anak Mewarisi Rumah Orang Tua

Di tahap 1 kita pakai analogi **pigura**. Sekarang ganti analogi.

Bayangkan:

- **Bapak** membangun rumah besar. Rumah ini punya: tembok, atap, lantai, kamar kosong.
- **Anak** tidak perlu membangun rumah dari nol. Anak cukup **pindah ke rumah bapak**, lalu **mengisi kamar kosong** dengan barangnya sendiri.

Di Laravel:

- **Bapak** = `layout/app.blade.php` → punya kerangka tetap + kamar kosong (`@yield('konten')`)
- **Anak** = `produk/index.blade.php`, `dashboard/index.blade.php`, dll. → "pindah" ke rumah bapak, lalu isi kamarnya

Cara anak "pindah ke rumah bapak" di Blade:

```blade
@extends('layout.app')
```

Cara anak "mengisi kamar kosong":

```blade
@section('konten')
    <!-- isi kamar di sini -->
@endsection
```

Itulah dua directive yang akan kita pelajari.

---

## 3. Apa Itu `@extends('layout.app')`?

### Bentuk umum

```blade
@extends('nama.layout')
```

### Arti dalam bahasa manusia

> "Halaman ini mau **mewarisi** (pake kerangka dari) layout bernama `nama.layout`."

### `layout.app` itu apa?

Ingat tahap 2: kita membuat file di:

```
resources/views/layout/app.blade.php
```

Di Blade, **titik (`.`)** artinya **slash (`/`)** dalam path folder.

Jadi:

- `layout.app` = `resources/views/layout/app.blade.php`
- `produk.index` = `resources/views/produk/index.blade.php`
- `dashboard.index` = `resources/views/dashboard/index.blade.php`

> 📝 **Pesan mentor:**
> Kamu sudah pakai konvensi ini di controller: `view('produk.index')`. Sama persis. Blade pakai **notasi titik**, bukan slash.

### Apa yang terjadi saat Blade melihat `@extends`?

Saat halaman A berisi `@extends('layout.app')`, Blade melakukan hal ini secara otomatis:

1. **Ambil** isi file `layout/app.blade.php`.
2. **Render** dulu halaman A, simpan hasilnya di memori.
3. Pasang hasil halaman A ke **lubang `@yield('konten')`** di layout.
4. Kirim **hasil akhir** (layout + isi) ke browser.

Jadi yang dikirim ke browser **bukan** isi halaman A mentah, tapi **layout yang sudah terisi** halaman A.

### Penting: `@extends` harus di baris pertama

Aturan wajib Blade:

```blade
@extends('layout.app')    ← WAJIB paling atas
                          ← boleh ada baris kosong
{{-- baru di bawahnya boleh @section, @push, dll --}}
```

**Jangan** tarik kode HTML dulu baru `@extends`. Blade akan bingung dan error.

---

## 4. Apa Itu `@section('konten') ... @endsection`?

### Bentuk umum

```blade
@section('nama_lubang')
    <!-- konten untuk mengisi lubang itu -->
@endsection
```

### Arti dalam bahasa manusia

> "Ini isi untuk **lubang bernama `nama_lubang`** yang ada di layout."

### Hubungan dengan `@yield`

Di tahap 2, layout menulis:

```blade
@yield('konten')
```

Di halaman (anak), kita tulis:

```blade
@section('konten')
    Halo, ini isi konten saya.
@endsection
```

Apa yang Blade lakukan?

Blade **mencocokkan nama**: `'konten'` == `'konten'`. Lalu mengganti `@yield('konten')` di layout dengan isi `@section('konten')` di halaman.

### Analogi "kunci dan gembok"

Bayangkan:

- `@yield('konten')` di layout = **gembok** dengan label "konten".
- `@section('konten')` di halaman = **kunci** dengan label "konten".

Blade memasangkan kunci dan gembok berdasarkan **namanya**. Nama harus **sama persis**.

```
Layout:    @yield('konten')       → gembok "konten" (kosong, menunggu)
Halaman:   @section('konten')     → kunci "konten" (membawa isi)
                                    ───────────────
                                    Match! Kunci cocok gembok.
                                    Gembok dibuka, isi ditampilkan.
```

### Yang terjadi kalau nama tidak cocok

Misal layout punya `@yield('konten')` tapi halaman menulis `@section('isi')`:

```blade
@section('isi')      ← nama 'isi', tidak cocok dengan 'konten'
    Halo!
@endsection
```

Hasil: **lubang `@yield('konten')` tetap kosong**, dan halaman tidak menampilkan apa-apa (atau error warning). Nama harus **sama persis**.

> 🪤 **Jebakan pemula:**
> Salah ketik nama section paling sering terjadi. Cek 3x: `konten` di layout harus persis sama dengan `konten` di halaman. Hindari spasi, huruf besar/kecil beda.

---

## 5. Kode Lengkap Halaman Latihan

Sekarang mari kita berlatih. Buat **file baru** untuk latihan (bukan file produk dulu):

```
resources/views/latihan.blade.php
```

Isi file:

```blade
@extends('layout.app')

@section('konten')
    <h2>Selamat Datang di Halaman Latihan</h2>
    <p>Ini adalah halaman latihan pertama yang memakai layout.</p>
    <p>Jika kamu melihat header di atas dan footer di bawah,
       berarti layout berhasil dipakai.</p>
@endsection
```

### Penjelasan baris per baris

**Baris 1:** `@extends('layout.app')`

Artinya: "Halaman ini mewarisi layout `layout/app.blade.php`. Tolong pakai kerangka dari sana."

**Baris 3:** `@section('konten')`

Artinya: "Saya mau mengisi lubang bernama `konten` yang ada di layout."

**Baris 4-7:** konten HTML

Ini adalah isi yang akan dimasukkan ke lubang `@yield('konten')`. Bebas apa saja: teks, gambar, tabel, form, dll.

**Baris 8:** `@endforeach` ... eh salah, `@endsection`

`@endsection` menandai **akhir** dari section. Wajib ada. Kalau lupa, Blade error "section not closed".

### Hasil yang dilihat di browser

Setelah route diatur (lihat section 6), saat kamu buka halaman latihan di browser, Blade akan merangkai begini:

```html
<!-- dari layout/app.blade.php -->
<!DOCTYPE html>
<html lang="id">
<head>
    <title>Toko Bukhari</title>           <!-- default, karena halaman belum kirim $title -->
</head>
<body>

    <header>
        <h1>Toko Bukhari</h1>
        <nav>...menu...</nav>
    </header>

    <main>
        <!-- INI HASIL @section('konten') dari latihan.blade.php -->
        <h2>Selamat Datang di Halaman Latihan</h2>
        <p>Ini adalah halaman latihan pertama yang memakai layout.</p>
        <p>Jika kamu melihat header di atas dan footer di bawah,
           berarti layout berhasil dipakai.</p>
    </main>

    <footer>
        <p>&copy; 2026 Toko Bukhari...</p>
    </footer>

</body>
</html>
```

Perhatikan:

- Header, footer, `<html>`, `<head>` **datang dari layout**.
- Konten `<h2>Selamat Datang...</h2>` **datang dari halaman latihan**.
- Halaman latihan **tidak perlu** menulis `<html>`, `<head>`, `<header>`, `<footer>`. Semua itu disumbang layout.

---

## 6. Cara Menampilkan Halaman Latihan (Route)

Agar halaman latihan bisa dibuka di browser, kita perlu **route** sementara.

Buka `routes/web.php`, tambahkan:

```php
use Illuminate\Support\Facades\Route;

Route::get('/latihan', function () {
    return view('latihan');
});
```

Penjelasan:

- `Route::get('/latihan', ...)` → saat user mengakses URL `/latihan`, jalankan fungsi di dalamnya.
- `function () { return view('latihan'); }` → kembalikan view bernama `latihan` (yaitu file `latihan.blade.php`).
- Closure (fungsi tanpa nama) dipakai karena halaman ini **statis**, tidak butuh controller.

### Coba di browser

Buka:

```
http://localhost:8000/latihan
```

Kamu **harus** melihat:

1. Header "Toko Bukhari" di atas (dari layout).
2. Konten "Selamat Datang di Halaman Latihan" di tengah (dari halaman).
3. Footer "© 2026 Toko Bukhari..." di bawah (dari layout).

Kalau tiga itu muncul, **selamat!** Layout kamu bekerja sempurna.

> 🪤 **Jebakan pemula:**
> Kalau error `View [layout.app] not found`, periksa:
> 1. Folder `layout/` ada di `resources/views/`?
> 2. File di dalamnya bernama `app.blade.php` (titik, bukan underscore)?
> 3. Penulisan `@extends('layout.app')` benar (titik, bukan slash)?

---

## 7. Mengirim Variabel Judul ke Layout

Di tahap 2, layout menulis:

```blade
<title>{{ $title ?? 'Toko Bukhari' }}</title>
```

Sekarang kita mau halaman latihan **mengirim** variabel `$title` sendiri supaya judul tab berubah.

### Cara 1: Lewat `@extends` (paling umum)

Ubah halaman `latihan.blade.php` jadi:

```blade
@extends('layout.app', ['title' => 'Halaman Latihan'])

@section('konten')
    <h2>Selamat Datang di Halaman Latihan</h2>
    <p>...</p>
@endsection
```

Penjelasan:

- `@extends('layout.app', ['title' => 'Halaman Latihan'])` → mewarisi layout **dan** mengirim array variabel ke layout.
- `'title' => 'Halaman Latihan'` → kunci `title` jadi variabel `$title` di layout, nilainya `'Halaman Latihan'`.

Sekarang judul tab browser akan jadi: **"Halaman Latihan"** (bukan default "Toko Bukhari").

### Cara 2: Lewat controller (akan dipakai di tahap 4)

Nanti di controller, kamu bisa kirim title lewat `view()`:

```php
public function index()
{
    $produk = Produk::all();
    return view('produk.index', [
        'produk' => $produk,
        'title'  => 'Daftar Produk',    // ← ini akan jadi $title di layout
    ]);
}
```

Variabel yang dikirim controller akan **otomatis tersedia** di layout juga. Jadi judul tab berubah jadi "Daftar Produk" saat halaman daftar produk dibuka.

### Beda Cara 1 dan Cara 2

| Cara | Lokasi pengiriman | Cocok untuk |
|---|---|---|
| `@extends(..., [...])` | di file Blade | judul tetap per halaman (mis: "Tentang Kami") |
| `view(..., [...])` | di controller | judul dinamis (mis: "Detail Produk: $produk->nama") |

Keduanya sah. Pilih sesuai situasi.

---

## 8. Diagram Alur Kerja `@extends` + `@section`

```
┌────────────────────────────────┐
│  latihan.blade.php             │
│                                │
│  @extends('layout.app')        │ ───┐
│                                │    │ "saya mau pakai layout itu"
│  @section('konten')            │    │ + "ini isi untuk lubang 'konten'"
│    <h2>Selamat Datang...</h2>  │    │
│  @endsection                   │    │
└────────────────────────────────┘    │
                                      ▼
┌──────────────────────────────────────────────────────┐
│  layout/app.blade.php                                │
│                                                      │
│  <html>...                                           │
│    <title>{{ $title ?? 'Toko Bukhari' }}</title>     │ ← diisi dari variabel yang dikirim
│    <header>...</header>                              │
│    <main>                                            │
│      @yield('konten')   ◄── diisi oleh @section      │ ← diisi dari halaman
│    </main>                                           │
│    <footer>...</footer>                              │
│  </html>                                             │
└──────────────────────────────────────────────────────┘
                                      │
                                      ▼
                          ┌─────────────────────┐
                          │  Browser user       │
                          │                     │
                          │  Header (layout)    │
                          │  -----              │
                          │  Konten (halaman)   │
                          │  -----              │
                          │  Footer (layout)    │
                          └─────────────────────┘
```

---

## 9. Hal yang Sering Bikin Error (Troubleshooting)

Berikut error paling sering dialami pemula saat belajar `@extends`:

### Error 1: `View [layout.app] not found`

**Penyebab:**
- Folder `layout/` belum dibuat, atau
- File bernama `app.blade.php` tapi salah ketik (mis: `App.blade.php`, `app.php`).

**Solusi:** Cek path file persis `resources/views/layout/app.blade.php`.

### Error 2: Header/footer tidak muncul

**Penyebab:**
- Lupa menulis `@extends('layout.app')` di baris pertama.
- Atau menulis `@extends` setelah kode HTML lain.

**Solusi:** `@extends` wajib di **baris paling atas** file.

### Error 3: Konten halaman tidak muncul

**Penyebab:**
- Nama section tidak cocok (mis: `@section('isi')` padahal layout `@yield('konten')`).
- Lupa `@endsection` di akhir.

**Solusi:** Cek nama section **persis sama** dengan nama yield di layout, dan pastikan `@endsection` ada.

### Error 4: `Undefined variable $title`

**Penyebab:**
- Layout menulis `{{ $title }}` tanpa `?? '...'`.
- Halaman tidak mengirim `$title`.

**Solusi:** Selalu pakai `{{ $title ?? 'Toko Bukhari' }}` (dengan nilai cadangan).

> 📝 **Pesan mentor:**
> Kalau layout bekerja tapi "aneh", 90% masalahnya adalah salah ketik nama. Cek ejaan `konten`, `layout.app`, `@endsection` dengan teliti.

---

## 10. Latihan Mandiri (Coba Sendiri)

Sebelum lanjut ke tahap 4, coba kerjakan latihan ini:

### Latihan A

Buat halaman baru bernama `tentang.blade.php` di `resources/views/` dengan isi:

1. Memakai layout `layout.app`.
2. Judul tab: "Tentang Kami".
3. Konten: judul `<h2>Tentang Toko Bukhari</h2>` + paragraf singkat tentang toko.

<details>
<summary><strong>Lihat jawaban Latihan A</strong></summary>

```blade
@extends('layout.app', ['title' => 'Tentang Kami'])

@section('konten')
    <h2>Tentang Toko Bukhari</h2>
    <p>Toko Bukhari berdiri sejak 2020, menjual berbagai
       kebutuhan rumah tangga dengan harga terjangkau.</p>
@endsection
```

Jangan lupa tambahkan route di `routes/web.php`:

```php
Route::get('/tentang', function () {
    return view('tentang');
});
```

Lalu buka `http://localhost:8000/tentang`.

</details>

---

## 11. Rangkuman Directive yang Dipelajari

| Directive | Di mana ditulis | Fungsi |
|---|---|---|
| `@extends('layout.app')` | halaman (anak), **baris pertama** | mewarisi layout |
| `@section('nama') ... @endsection` | halaman (anak) | mengisi lubang `@yield('nama')` di layout |
| `@yield('nama')` | layout | membuat lubang kosong (dipelajari tahap 2) |
| `@extends('layout.app', ['title' => '...'])` | halaman | mewarisi + kirim variabel ke layout |

---

## 12. Istilah Kunci Tahap Ini

| Istilah | Arti sederhana |
|---|---|
| **`@extends`** | "halaman ini mau mewarisi layout X" |
| **`@section`** | "ini isi untuk lubang bernama X di layout" |
| **`@endsection`** | penutup `@section`, wajib ada |
| **Notasi titik** | `layout.app` = `layout/app.blade.php` (titik = slash) |
| **Closure** | fungsi tanpa nama, dipakai untuk route statis |

---

## 13. Rangkuman Tahap 3

1. `@extends('layout.app')` → halaman mewarisi layout, **wajib di baris pertama**.
2. `@section('konten') ... @endsection` → mengisi lubang `@yield('konten')` di layout.
3. Nama section **harus persis sama** dengan nama yield.
4. Notasi titik: `layout.app` = `resources/views/layout/app.blade.php`.
5. Variabel bisa dikirim ke layout lewat `@extends('layout.app', ['title' => '...'])` atau lewat controller.
6. Kita sudah membuat halaman latihan `/latihan` sebagai bukti layout bekerja.
7. **Halaman produk belum diubah** — itu di tahap 4.

---

## 14. Cek Pemahaman (Jawab Sendiri Dulu)

1. Di baris ke berapa `@extends` harus ditulis?
2. Apa beda `@yield` dan `@section`?
3. Apa arti notasi `produk.index` di Blade?
4. Bagaimana cara mengirim variabel `$title` dari halaman ke layout?
5. Apa yang terjadi kalau nama `@section('isi')` tidak cocok dengan `@yield('konten')` di layout?
6. Apa fungsi `@endsection`?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. **Di baris pertama** file Blade. Tidak boleh ada kode HTML lain sebelumnya.
2. `@yield` ditulis di **layout** (membuat lubang kosong). `@section` ditulis di **halaman** (mengisi lubang tersebut). Keduanya berpasangan via nama yang sama.
3. Notasi titik `produk.index` artinya path file `resources/views/produk/index.blade.php`. Titik menggantikan slash.
4. Dua cara: (a) lewat Blade `@extends('layout.app', ['title' => '...'])`, atau (b) lewat controller `view('...', ['title' => '...'])`.
5. Lubang `@yield('konten')` akan **tetap kosong**. Isi `@section('isi')` tidak akan ditampilkan. Nama harus persis sama.
6. Menandai **akhir** dari sebuah `@section`. Wajib ada, jika tidak Blade error "section not closed".

</details>

---

## 15. Apakah Kamu Ingin Lanjut?

Di tahap 3 ini kita sudah **paham pola `@extends` + `@section`** lewat halaman latihan, tapi **belum menyentuh file produk atau dashboard asli**.

Langkah berikutnya:

> ### "Apakah kamu ingin lanjut ke langkah berikutnya: mengubah halaman daftar produk agar pakai layout?"
>
> Di tahap berikutnya kita akan:
>
> - membuka file `produk/index.blade.php` yang sudah ada
> - **memecah** isinya jadi dua: hapus header/footer duplikat, sisakan konten unik
> - menambahkan `@extends('layout.app')` dan `@section('konten')`
> - mengubah controller `ProdukController@index` untuk mengirim `$title`
> - **melihat efeknya langsung** di browser

Jawab: **"Ya, lanjut"** untuk ke tahap 4,
atau **"Ulangi tahap 3"** kalau ada bagian yang masih perlu diperdalam.

---

> 📚 **Daftar Tahap (12. Layout Blade Reusable):**
> - ✅ Tahap 1 — Apa itu layout Blade reusable
> - ✅ Tahap 2 — Membuat file layout utama Blade
> - ✅ Tahap 3 — `@extends` dan `@section` untuk memakai layout (kamu di sini)
> - ⏳ Tahap 4 — Mengubah halaman daftar produk agar pakai layout
> - ⏳ Tahap 5 — Mengubah halaman tambah, edit, detail produk
> - ⏳ Tahap 6 — Mengubah halaman dashboard admin
> - ⏳ Tahap 7 — Partial: memecah navbar dan footer
> - ⏳ Tahap 8 — Component: membuat komponen kartu produk
> - ⏳ Tahap 9 — Slot: lubang yang bisa diisi apa saja
> - ⏳ Tahap 10 — Ringkasan dan best practice
