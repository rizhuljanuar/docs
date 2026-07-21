# Tahap 2 — Membuat File Layout Utama Blade

> Mentor: Laravel Dasar — Fondasi MVC, CRUD
> Topik: 12. Layout Blade Reusable
> Fokus tahap ini: **membuat satu file layout utama** + memahami directive `@yield`.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Membuat **satu file layout utama** bernama `layout/app.blade.php`.
2. Menjelaskan apa itu **directive `@yield`** dan fungsinya.
3. Menjelaskan apa itu **directive `@stack`** untuk CSS dan JavaScript.
4. Menjelaskan apa itu variabel `{{ $title ?? 'Default' }}` di layout.
5. Menulis struktur HTML lengkap layout tanpa bingung.

Di tahap ini kita **belum** mengubah halaman produk apa pun. Kita hanya membuat **kerangka layout**-nya dulu. Halaman produk akan diubah di tahap 4 dan 5.

---

## 2. Analogi Sehari-Hari: Membuat Bingkai Foto

Di tahap 1 kita sudah bicara soal **pigura foto** (picture frame).

Sekarang, bayangkan kamu pergi ke toko kerajinan untuk **membuat satu pigura**.

Kamu beli:
- **kayu jati** untuk bagian pinggir (header, sidebar, footer)
- **kaca bening** untuk bagian tengah → ini akan menjadi lubang `@yield('konten')`

Lalu kamu bawa pulang pigura kosong itu. Belum ada foto di dalamnya. **Kosong.**

Besoknya, baru kamu masukkan foto yang berbeda-beda:
- Senin → foto keluarga
- Selasa → foto liburan
- Rabu → foto kucing

**Pigura-nya tetap satu**, yang ganti hanya foto di tengahnya.

Di Laravel:

- **pigura kosong** = file `layout/app.blade.php` (yang akan kita buat sekarang)
- **kaca lubang** = `@yield('konten')` (tempat foto ditukar)
- **foto** = isi halaman produk / dashboard (tahap berikutnya)

> 📝 **Pesan mentor:**
> Di tahap ini, kita hanya **membuat piguranya** saja. Belum memasukkan foto. Sabar ya, ini penting supaya kamu paham fondasinya.

---

## 3. Persiapan: Di Mana File Layout Diletakkan?

Sebelum menulis kode, kita perlu tahu **di mana** file layout ini harus disimpan.

Struktur folder `resources/views/` di proyek Laravel kamu saat ini kira-kira seperti ini:

```
resources/views/
├── produk/
│   ├── index.blade.php      (daftar produk)
│   ├── create.blade.php     (tambah produk)
│   ├── edit.blade.php       (edit produk)
│   └── show.blade.php       (detail produk)
├── dashboard/
│   └── index.blade.php      (dashboard admin)
```

Perhatikan: belum ada folder `layout/`. **Folder ini belum ada** karena kita memang belum punya layout.

Kita akan **membuat folder baru** bernama `layout/`, lalu di dalamnya membuat file `app.blade.php`. Jadi hasil akhirnya:

```
resources/views/
├── layout/
│   └── app.blade.php        ← BARU: file layout utama (pigura kosong)
├── produk/
│   ├── index.blade.php
│   ├── create.blade.php
│   ├── edit.blade.php
│   └── show.blade.php
├── dashboard/
│   └── index.blade.php
```

### Kenapa dinamai `layout/app.blade.php`?

- `layout/` → nama folder, menyiratkan "semua file layout ada di sini" (kelak bisa ada `layout/admin.blade.php`, `layout/publik.blade.php`, dll.)
- `app.blade.php` → nama konvensi Laravel untuk "layout utama aplikasi". Bisa juga dinamai `main.blade.php` atau `master.blade.php`, tapi `app` lebih umum dipakai.

> 📝 **Pesan mentor:**
> Konvensi (kesepakatan nama) penting di Laravel. Ikuti konvensi `layout/app.blade.php` dulu supaya tutorial yang kamu baca di internet mudah dipahami.

---

## 4. Kode Lengkap File `layout/app.blade.php`

Sekarang saatnya menulis kode. Tapi **jangan** copy-paste dulu. Kita akan **bahas per bagian** di section selanjutnya.

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ $title ?? 'Toko Bukhari' }}</title>

    <!-- CSS Tambahan dari Halaman -->
    @stack('css')
</head>
<body>

    <!-- ===== HEADER ===== -->
    <header>
        <h1>Toko Bukhari</h1>
        <nav>
            <a href="{{ route('produk.index') }}">Produk</a>
            <a href="{{ route('dashboard.index') }}">Dashboard</a>
        </nav>
    </header>

    <!-- ===== KONTEN UTAMA ===== -->
    <main>
        @yield('konten')
    </main>

    <!-- ===== FOOTER ===== -->
    <footer>
        <p>&copy; {{ date('Y') }} Toko Bukhari. Semua hak dilindungi.</p>
    </footer>

    <!-- JavaScript Tambahan dari Halaman -->
    @stack('js')

</body>
</html>
```

Sekarang mari kita bedah **satu per satu** bagian kode di atas dengan bahasa sederhana.

---

## 5. Penjelasan Bagian 1: Kerangka HTML Dasar

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ $title ?? 'Toko Bukhari' }}</title>
    @stack('css')
</head>
<body>
    ...
</body>
</html>
```

Ini adalah **kerangka HTML wajib** yang harus ada di setiap halaman web. Tanpa ini, browser tidak tahu kalau file ini adalah halaman HTML.

Penjelasan tiap baris:

| Baris | Fungsi |
|---|---|
| `<!DOCTYPE html>` | Memberi tahu browser: "ini halaman HTML5". |
| `<html lang="id">` | Membuka elemen html. `lang="id"` artinya bahasa Indonesia (bagus untuk SEO dan aksesibilitas). |
| `<meta charset="UTF-8">` | Encoding teks agar huruf bahasa Indonesia (seperti "é", "ñ") tampil benar. |
| `<meta name="viewport" ...>` | Agar halaman tampil rapi di HP dan tablet (responsive). |
| `<title>...</title>` | Judul tab browser. Ini **dinamis**, lihat section 6. |
| `@stack('css')` | Lubang untuk CSS tambahan dari halaman (lihat section 8). |

> 📝 **Pesan mentor:**
> Bagian ini sama persis dengan yang sudah kamu tulis di setiap file Blade produk. Bedanya sekarang **ditulis sekali saja** di layout.

---

## 6. Penjelasan Bagian 2: Judul Dinamis `{{ $title ?? '...' }}`

Perhatikan baris ini:

```blade
<title>{{ $title ?? 'Toko Bukhari' }}</title>
```

Ini bagian penting. Mari kita bedah.

### `{{ $title }}`

Dua kurung kurawal `{{ ... }}` di Blade artinya **tampilkan nilai variabel**. Sama seperti `echo` di PHP biasa.

Jadi `{{ $title }}` artinya: "tampilkan isi variabel `$title`".

### `?? 'Toko Bukhari'`

Operator `??` di PHP disebut **null coalescing**. Artinya:

> "Kalau variabel di kiri **ada dan tidak null**, pakai nilai itu.
> Tapi kalau **tidak ada** (atau null), pakai nilai cadangan di kanan."

Jadi `{{ $title ?? 'Toko Bukhari' }}` artinya:

> "Kalau halaman mengirim variabel `$title`, pakai itu sebagai judul tab.
> Tapi kalau tidak dikirim, pakai 'Toko Bukhari' sebagai judul default."

### Contoh konkret

Nanti di tahap 4, halaman daftar produk akan mengirim title begini:

```blade
@extends('layout.app', ['title' => 'Daftar Produk'])
```

Maka judul tab browser jadi: **"Daftar Produk"**.

Tapi kalau ada halaman yang **lupa** mengirim title, judul tab tidak kosong, melainkan otomatis: **"Toko Bukhari"**. Ini *default* yang aman.

> 🪤 **Jebakan pemula:**
> Kalau kamu menulis `{{ $title }}` saja tanpa `?? '...'`, dan halaman tidak mengirim title, Laravel akan error `"Undefined variable $title"`. Selalu pakai `?? '...'` untuk variabel yang bersifat opsional.

---

## 7. Penjelasan Bagian 3: `@yield('konten')` (Bintang Utama)

Inilah bagian **paling penting** di seluruh layout:

```blade
<main>
    @yield('konten')
</main>
```

### Apa itu `@yield`?

`@yield` adalah **directive Blade** (perintah khusus Blade, dimulai dengan `@`).

Fungsinya: **membuat "lubang" bernama `konten`** di layout.

Lubang ini **kosong** di file layout. Nanti di tahap 4, tiap halaman (daftar produk, tambah produk, dashboard, dll.) akan **mengisi** lubang ini dengan konten mereka sendiri.

### Analogi

Bayangkan layout ini adalah sebuah **formulir kosong** dengan satu kotak bertuliskan:

```
┌──────────────────────────────┐
│  ISI KONTEN DI SINI:         │
│  _______________________     │
│  _______________________     │
│  _______________________     │
└──────────────────────────────┘
```

`@yield('konten')` = **kotak kosong** itu. Siap diisi apa saja oleh halaman yang memakai layout.

### Kenapa namanya `'konten'`?

Nama `'konten'` **bukan kata wajib**. Kamu bebas menamai apa saja, misalnya:

- `@yield('isi')`
- `@yield('content')`
- `@yield('body')`

Tapi konsisten saja pakai `'konten'` agar mudah diingat dan seragam di seluruh proyek.

> 📝 **Pesan mentor:**
> **Satu layout bisa punya banyak `@yield`.** Misalnya nanti bisa ditambah `@yield('sidebar')`, `@yield('judul_halaman')`, dll. Tapi untuk sekarang, cukup **satu `@yield('konten')`** dulu agar tidak bingung.

### Tag `<main>` di sekitarnya

`<main>` adalah tag HTML5 yang menandai "isi utama halaman". Ini bagus untuk SEO dan aksesibilitas (pembaca layar bagi tunanetra tahu di mana konten utama berada).

---

## 8. Penjelasan Bagian 4: `@stack('css')` dan `@stack('js')`

Lihat dua baris ini (tersebar di kode):

```blade
@stack('css')    <!-- di dalam <head> -->
@stack('js')     <!-- sebelum </body> -->
```

### Apa itu `@stack`?

`@stack` artinya **"tumpukan"**. Fungsinya: membuat **tumpukan kosong** tempat halaman bisa menambah CSS atau JavaScript khusus.

### Kenapa perlu `@stack`?

Pikirkan situasi ini:

- Halaman **daftar produk** butuh CSS khusus untuk tabel produk.
- Halaman **tambah produk** butuh CSS khusus untuk form.
- Halaman **dashboard** butuh CSS khusus untuk grafik.

Setiap halaman punya **CSS/JS yang berbeda-beda**. Kalau semua CSS ditumpuk di layout, jadi berantakan.

Solusinya: layout sediakan **lubang stack**, tiap halaman bisa **menambah (push)** CSS/JS sendiri ke lubang itu.

### Cara pakai (di tahap berikutnya)

Nanti di halaman produk, kita bisa tulis:

```blade
@push('css')
    <style>
        table { border-collapse: collapse; }
    </style>
@endpush
```

Maka CSS itu otomatis disisipkan di lokasi `@stack('css')` di layout. Rapi!

### Beda `@yield` dan `@stack`

| Directive | Cara mengisi | Sifat |
|---|---|---|
| `@yield('konten')` | **menimpa** (`@section ... @endsection`) | satu lubang diisi satu halaman |
| `@stack('css')` | **menumpuk** (`@push ... @endpush`) | banyak halaman bisa tambah ke tumpukan |

> 📝 **Pesan mentor:**
> Untuk pemula, `@stack` **bisa diabaikan dulu**. Kamu tetap bisa bikin layout tanpa `@stack`. Saya tampilkan di sini agar kamu tahu ini ada dan tidak kaget saat baca tutorial lain. Kalau bingung, **hapus saja baris `@stack`** dulu, layout tetap berfungsi.
>
> <!-- ponytail: @stack bisa dihapus untuk MVP layout. Tambah saat halaman butuh CSS/JS khusus. -->

---

## 9. Penjelasan Bagian 5: Header dan Footer

```blade
<!-- ===== HEADER ===== -->
<header>
    <h1>Toko Bukhari</h1>
    <nav>
        <a href="{{ route('produk.index') }}">Produk</a>
        <a href="{{ route('dashboard.index') }}">Dashboard</a>
    </nav>
</header>
```

Header berisi:
- `<h1>Toko Bukhari</h1>` → judul/logo toko.
- `<nav>` → menu navigasi.
- `{{ route('produk.index') }}` → helper Laravel untuk menghasilkan URL berdasarkan **nama route**. Lebih aman dan rapi daripada menulis `/produk` manual. Akan otomatis berubah kalau route berubah.

Footer:

```blade
<footer>
    <p>&copy; {{ date('Y') }} Toko Bukhari. Semua hak dilindungi.</p>
</footer>
```

- `&copy;` → simbol © dalam HTML.
- `{{ date('Y') }}` → tahun saat ini (misal 2026), otomatis update tiap tahun.

> 📝 **Pesan mentor:**
> Inilah inti layout: header dan footer **ditulis sekali** di sini, tidak ditulis ulang di setiap halaman produk. Saat mau ubah "Toko Bukhari" jadi "Toko Berkah", cukup ubah **1 file ini**.

---

## 10. Cara Membuat File di VS Code / Editor

Langkah konkret:

1. Buka folder `resources/views/` di editor.
2. Klik kanan → **New Folder** → namai `layout`.
3. Masuk ke folder `layout/`.
4. Klik kanan → **New File** → namai `app.blade.php`.
5. Copy kode lengkap dari section 4 di atas.
6. Paste dan save (`Ctrl + S` / `Cmd + S`).

Hasilnya:

```
resources/views/layout/app.blade.php    ← FILE INI
```

### Cek: file kosong belum bisa dilihat di browser

Kalau kamu langsung buka `http://localhost:8000/produk`, halaman itu masih pakai Blade lama (yang belum pakai layout). Layout baru ini **belum terlihat efeknya** karena belum ada halaman yang memakainya. **Ini normal.**

Efek layout akan terlihat di **tahap 4**, setelah kita ubah halaman produk agar memakai layout ini.

> 📝 **Pesan mentor:**
> Jangan panik kalau "tidak terlihat perubahan". Layout itu seperti pipa air yang baru dipasang tapi belum dihubungkan ke keran. Kita sambungkan di tahap berikutnya.

---

## 11. Diagram: Struktur Layout dengan Lubang `@yield`

Visualisasi file `layout/app.blade.php`:

```
┌────────────────────────────────────────────────┐
│  <html>                                        │ ← tetap
│    <head>                                      │ ← tetap
│      <title>{{ $title ?? '...' }}</title>      │ ← dinamis (judul)
│      @stack('css')                  ◄── lubang CSS
│    </head>                                     │
│    <body>                                      │
│      ┌──────────────────────────────────────┐  │
│      │ <header>                             │  │ ← tetap
│      │   Toko Bukhari | Produk | Dashboard  │  │
│      │ </header>                            │  │
│      └──────────────────────────────────────┘  │
│      ┌──────────────────────────────────────┐  │
│      │ <main>                               │  │
│      │   @yield('konten')      ◄── LUBANG   │  │ ← diisi tiap halaman
│      │ </main>                              │  │
│      └──────────────────────────────────────┘  │
│      ┌──────────────────────────────────────┐  │
│      │ <footer>                             │  │ ← tetap
│      │   © 2026 Toko Bukhari                │  │
│      │ </footer>                            │  │
│      └──────────────────────────────────────┘  │
│      @stack('js')                  ◄── lubang JS
│    </body>                                     │
│  </html>                                       │
└────────────────────────────────────────────────┘
```

Bagian **tetap** (kuning): `<html>`, `<head>`, `<header>`, `<footer>`.
Bagian **dinamis** (hijau): `<title>`, `@yield('konten')`, `@stack('css')`, `@stack('js')`.

---

## 12. Rangkuman Directive Baru yang Dipelajari

| Directive | Fungsi | Di mana |
|---|---|---|
| `@yield('konten')` | Membuat lubang konten utama (diisi tiap halaman) | layout |
| `@stack('css')` | Lubang tumpukan CSS (bisa diisi banyak halaman) | layout |
| `@stack('js')` | Lubang tumpukan JavaScript | layout |
| `{{ $title ?? '...' }}` | Variabel dinamis dengan nilai cadangan | layout |

Di **tahap 3** kamu akan belajar directive lawan dari `@yield`, yaitu `@extends` dan `@section` — keduanya digunakan **di halaman produk** (bukan di layout) untuk **mengisi lubang** `@yield`.

---

## 13. Istilah Kunci Tahap Ini

| Istilah | Arti sederhana |
|---|---|
| **Layout** | kerangka tetap halaman dengan lubang `@yield` |
| **Directive** | perintah khusus Blade, dimulai dengan tanda `@` (seperti `@yield`, `@stack`) |
| **`@yield`** | membuat lubang (kosong) di layout untuk diisi halaman lain |
| **`@stack`** | membuat tumpukan kosong untuk CSS/JS tambahan |
| **Konvensi** | kesepakatan nama (mis: `layout/app.blade.php`) |

---

## 14. Rangkuman Tahap 2

1. Layout utama disimpan di `resources/views/layout/app.blade.php` (folder dan file baru).
2. Layout berisi: kerangka HTML tetap + header + footer + satu lubang `@yield('konten')`.
3. `@yield('konten')` = lubang kosong yang akan diisi tiap halaman (produk, dashboard, dll.).
4. `{{ $title ?? 'Toko Bukhari' }}` = judul tab dinamis dengan nilai cadangan.
5. `@stack('css')` dan `@stack('js')` = lubang untuk CSS/JS tambahan per halaman (opsional untuk pemula).
6. **File layout belum terlihat efeknya** sampai ada halaman yang memakainya (akan dilakukan di tahap 4).

---

## 15. Cek Pemahaman (Jawab Sendiri Dulu)

1. Di folder mana file `app.blade.php` disimpan?
2. Apa fungsi `@yield('konten')`?
3. Apa arti `??` di `{{ $title ?? 'Toko Bukhari' }}`?
4. Kenapa kita pakai `{{ route('produk.index') }}` alih-alih `/produk` langsung?
5. Apa beda `@yield` dan `@stack`?
6. Kalau file layout sudah dibuat, apakah halaman produk otomatis berubah? Kenapa?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. Di `resources/views/layout/app.blade.php` (folder `layout/` baru dibuat).
2. Membuat lubang kosong di layout, nanti diisi konten utama oleh tiap halaman (produk, dashboard).
3. Operator **null coalescing**: "jika `$title` ada, pakai itu. Jika tidak, pakai 'Toko Bukhari' sebagai default."
4. Karena `route('produk.index')` menghasilkan URL berdasarkan **nama route**. Jika URL berubah (misal `/produk` → `/barang`), kode Blade tidak perlu diubah, asal nama route tetap sama. Lebih aman dan rapi.
5. `@yield` diisi dengan **menimpa** (`@section`), satu lubang satu halaman. `@stack` diisi dengan **menumpuk** (`@push`), banyak halaman bisa tambah ke tumpukan.
6. **Tidak otomatis**. Layout hanya "pigura kosong". Halaman produk harus **secara eksplisit memakai layout** itu dengan `@extends`. Ini akan dipelajari di tahap 3 dan dipraktikkan di tahap 4.

</details>

---

## 16. Apakah Kamu Ingin Lanjut?

Di tahap 2 ini kita sudah **membuat file layout utama**, tapi belum menggunakannya di halaman mana pun. Layarnya masih seperti biasa karena belum ada yang memakai layout ini.

Langkah berikutnya:

> ### "Apakah kamu ingin lanjut ke langkah berikutnya: belajar `@extends` dan `@section` untuk memakai layout?"
>
> Di tahap berikutnya kita akan:
>
> - memahami directive `@extends('layout.app')` (cara halaman "mewarisi" layout)
> - memahami directive `@section('konten') ... @endsection` (cara mengisi lubang `@yield`)
> - lihat contoh **satu halaman sederhana** (bukan produk dulu) yang memakai layout
> - **belum** mengubah halaman produk apa pun (itu tahap 4)

Jawab: **"Ya, lanjut"** untuk ke tahap 3,
atau **"Ulangi tahap 2"** kalau ada bagian yang masih perlu diperdalam.

---

> 📚 **Daftar Tahap (12. Layout Blade Reusable):**
> - ✅ Tahap 1 — Apa itu layout Blade reusable
> - ✅ Tahap 2 — Membuat file layout utama Blade (kamu di sini)
> - ⏳ Tahap 3 — `@extends` dan `@section` untuk memakai layout
> - ⏳ Tahap 4 — Mengubah halaman daftar produk agar pakai layout
> - ⏳ Tahap 5 — Mengubah halaman tambah, edit, detail produk
> - ⏳ Tahap 6 — Mengubah halaman dashboard admin
> - ⏳ Tahap 7 — Partial: memecah navbar dan footer
> - ⏳ Tahap 8 — Component: membuat komponen kartu produk
> - ⏳ Tahap 9 — Slot: lubang yang bisa diisi apa saja
> - ⏳ Tahap 10 — Ringkasan dan best practice
