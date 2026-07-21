# Tahap 7 — Partial: Memecah Navbar dan Footer

> Mentor: Laravel Dasar — Fondasi MVC, CRUD
> Topik: 12. Layout Blade Reusable
> Fokus tahap ini: **memahami partial** dan memecah navbar/footer menjadi file kecil yang bisa di-`@include`.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Menjelaskan apa itu **partial** dalam Blade.
2. Menjelaskan perbedaan **partial vs layout vs component**.
3. Memecah `<header>` (navbar) dari layout menjadi `partials/navbar.blade.php`.
4. Memecah `<footer>` dari layout menjadi `partials/footer.blade.php`.
5. Meng-include partial lewat directive `@include('partials.navbar')`.
6. Mengirim **data** ke partial lewat `@include('partials.navbar', ['menu' => 'produk'])`.
7. Menjelaskan kapan partial bermanfaat (terutama saat ada **multi-layout**).

Di tahap 1-6 kamu sudah menguasai layout. Sekarang kita **meningkatkan** keterbacaan layout dengan memecah bagian-bagian kecil.

---

## 2. Analogi Sehari-Hari: Modul Dapur

Bayangkan dapur punya **satu lemari besar** berisi: piring, gelas, sendok, garpu, mangkuk, panci.

Saat kamu mau ambil **sendok**, kamu harus buka lemari, mencari di antara semua barang, baru ketemu sendok di pojok.

Capek, kan?

Solusi: pisahkan jadi **laci-laci kecil**:

- Laci 1: hanya piring
- Laci 2: hanya sendok & garpu
- Laci 3: hanya gelas

Sekarang saat butuh sendok, kamu **langsung buka laci 2**. Cepat dan jelas.

Di Blade:

- **Lemari besar** = file layout (`layout/app.blade.php`) yang berisi semua: html, head, header, footer, dll.
- **Laci kecil** = file **partial** (`partials/navbar.blade.php`, `partials/footer.blade.php`).
- **Buka laci** = directive `@include('partials.navbar')`.

Partial = **potongan kecil** layout yang dipisah jadi file sendiri agar mudah dikelola.

---

## 3. Apa Itu Partial?

### Definisi sederhana

> **Partial** adalah **potongan kecil** Blade yang berisi bagian tertentu dari halaman (mis: navbar, footer, satu kartu produk), lalu disimpan di file sendiri dan di-include saat dibutuhkan.

### Ciri khas partial

- Lokasi: biasanya di folder `resources/views/partials/`.
- Nama file: deskriptif, contoh `partials/navbar.blade.php`, `partials/footer.blade.php`.
- Dipanggil lewat `@include('partials.navbar')`.
- Tidak punya `@extends` (bukan halaman utuh, hanya potongan).
- Tidak punya `<html>` atau `<body>` (cuma bagian kecil).

### Perbedaan partial vs layout vs component

| Konsep | Ukuran | Cara pakai | Contoh |
|---|---|---|---|
| **Layout** | Besar (kerangka halaman) | `@extends('layout.app')` | `layout/app.blade.php` |
| **Partial** | Sedang (bagian halaman) | `@include('partials.navbar')` | `partials/navbar.blade.php` |
| **Component** | Kecil & reusable (bisa berulang) | `<x-kartu-produk />` | `components/kartu-produk.blade.php` (tahap 8) |

> 📝 **Pesan mentor:**
> Aturan praktis: **layout** untuk kerangka, **partial** untuk bagian yang muncul **sekali** di halaman (navbar, footer), **component** untuk bagian yang muncul **berkali-kali** (kartu produk, tombol). Component akan dibahas di tahap 8.

---

## 4. Kenapa Perlu Partial?

Di tahap 6 kita punya **dua layout** (kalau pakai Solusi B):

- `layout/app.blade.php` (publik)
- `layout/admin.blade.php` (admin)

Keduanya punya **navbar** dan **footer** yang **sama persis**. Itu artinya navbar ditulis **dua kali** di dua file layout. Duplikat lagi!

Lalu kalau kamu ubah navbar (mis: tambah menu "Promo"), kamu harus ubah **dua file layout**.

Solusinya: pindahkan navbar ke **partial**, lalu dua layout tinggal `@include`. Ubah partial sekali → dua layout ikut berubah.

### Manfaat partial

| # | Manfaat | Efek |
|---|---|---|
| 1 | **Layout lebih pendek & fokus** | layout hanya berisi kerangka, bukan detail navbar |
| 2 | **Bagian reusable lintas layout** | navbar bisa dipakai di `layout.app` & `layout.admin` tanpa duplikat |
| 3 | **Ubah sekali, update di mana-mana** | ubah `partials/navbar.blade.php` → semua layout yang pakai ikut update |
| 4 | **Kolaborasi tim lebih mudah** | orang A kerja navbar, orang B kerja footer, tidak bentrok di file layout |

---

## 5. Langkah 1: Buat Folder `partials/`

Buat folder baru di dalam `resources/views/`:

```
resources/views/
├── layout/
│   └── app.blade.php
├── partials/              ← BARU
│   ├── navbar.blade.php   ← BARU
│   └── footer.blade.php   ← BARU
├── produk/
└── dashboard/
```

> 🪤 **Jebakan pemula:**
> Nama folder **harus** `partials/` (huruf kecil, jamak). Bukan `partial/`, bukan `Partial/`. Konvensi Laravel.

---

## 6. Langkah 2: Buat File `partials/navbar.blade.php`

Pindahkan blok `<header>...</header>` dari layout ke file baru:

```blade
<header>
    <h1>Toko Bukhari</h1>
    <nav>
        <a href="{{ route('produk.index') }}">Produk</a>
        <a href="{{ route('dashboard.index') }}">Dashboard</a>
        @if (auth()->check())
            <a href="/logout">Logout ({{ auth()->user()->name }})</a>
        @else
            <a href="/login">Login</a>
        @endif
    </nav>
</header>
```

### Penjelasan

- File partial **tidak** punya `<!DOCTYPE html>`, `<html>`, `<body>`. Hanya blok `<header>`.
- `auth()->check()` → cek apakah user sudah login. Kalau ya, tampilkan tombol Logout. Kalau tidak, tampilkan Login.
- `auth()->user()->name` → nama user yang sedang login (kalau ada).

> 📝 **Pesan mentor:**
> `auth()->check()` adalah contoh **logika di partial**. Partial bisa berisi logika PHP ringan. Tapi kalau logikanya berat (mis: query database), sebaiknya kirim data dari controller, jangan query di partial.

---

## 7. Langkah 3: Buat File `partials/footer.blade.php`

Pindahkan blok `<footer>...</footer>`:

```blade
<footer>
    <p>&copy; {{ date('Y') }} Toko Bukhari. Semua hak dilindungi.</p>
    <nav>
        <a href="/tentang">Tentang Kami</a>
        |
        <a href="/kontak">Kontak</a>
        |
        <a href="/privacy">Kebijakan Privasi</a>
    </nav>
</footer>
```

### Penjelasan

- Footer berisi copyright + link-link halaman statis.
- `{{ date('Y') }}` → tahun saat ini, otomatis update tiap tahun.

---

## 8. Langkah 4: Ubah `layout/app.blade.php` Pakai `@include`

Sekarang ganti blok `<header>` dan `<footer>` di layout dengan `@include`:

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

    {{-- Ganti <header> dengan @include --}}
    @include('partials.navbar')

    <div style="display:flex;">
        @hasSection('sidebar')
            <aside style="width:200px; background:#f4f4f4; padding:15px;">
                @yield('sidebar')
            </aside>
        @endif

        <main style="flex:1;">
            @yield('konten')
        </main>
    </div>

    {{-- Ganti <footer> dengan @include --}}
    @include('partials.footer')

    @stack('js')

</body>
</html>
```

### Yang berubah

| Bagian lama | Bagian baru |
|---|---|
| `<header>...20 baris...</header>` | `@include('partials.navbar')` (1 baris) |
| `<footer>...5 baris...</footer>` | `@include('partials.footer')` (1 baris) |

Layout sekarang jauh lebih **ringkas** dan fokus pada kerangka.

---

## 9. Directive `@include` — Penjelasan Detail

### Bentuk umum

```blade
@include('nama.partial')
```

### Arti

> "Sisipkan isi file `nama.partial` ke posisi ini."

### Cara kerja

Saat Blade melihat `@include('partials.navbar')`, ia melakukan:

1. Cari file `resources/views/partials/navbar.blade.php`.
2. Baca isinya.
3. Sisipkan ke lokasi `@include`.
4. Jalankan kode Blade di dalamnya (seperti `{{ }}`, `@if`, dll).
5. Kirim hasilnya ke output.

### Notasi titik sama seperti biasa

`partials.navbar` = `resources/views/partials/navbar.blade.php`. Titik = slash.

---

## 10. Mengirim Data ke Partial

Partial bisa menerima **data** dari tempat yang meng-include-nya. Sintaksnya:

```blade
@include('partials.navbar', ['menuAktif' => 'produk'])
```

Sekarang di dalam `partials/navbar.blade.php`, variabel `$menuAktif` tersedia. Bisa dipakai untuk menyorot menu aktif:

```blade
<header>
    <h1>Toko Bukhari</h1>
    <nav>
        <a href="{{ route('produk.index') }}"
           style="{{ $menuAktif ?? '' === 'produk' ? 'font-weight:bold;' : '' }}">
            Produk
        </a>
        <a href="{{ route('dashboard.index') }}"
           style="{{ ($menuAktif ?? '') === 'dashboard' ? 'font-weight:bold;' : '' }}">
            Dashboard
        </a>
    </nav>
</header>
```

### Cara pakai di layout

Layout tidak tahu menu mana yang aktif (tergantung halaman). Jadi layout bisa pakai **nilai default**:

```blade
@include('partials.navbar', ['menuAktif' => $menuAktif ?? ''])
```

Atau tiap halaman mengirim `$menuAktif` ke layout lewat controller:

```php
// ProdukController@index
return view('produk.index', [
    'produk'     => $produk,
    'title'      => 'Daftar Produk',
    'menuAktif'  => 'produk',     // ← diteruskan ke partial via layout
]);
```

### Sifat variabel di `@include`

| Asal variabel | Tersedia di partial? |
|---|---|
| Dikirim controller ke view utama | ✅ otomatis tersedia |
| Dikirim lewat `@include(..., [...])` | ✅ tersedia |
| Variabel lokal di file yang meng-include | ✅ otomatis tersedia (kecuali dipisah) |

> 📝 **Pesan mentor:**
> Secara default, partial **mewarisi semua variabel** dari view yang meng-include. Tapi best practice: **kirim eksplisit** lewat `@include(..., [...])` supaya jelas dependensi partial. Kalau partial butuh variabel X, kirim X secara eksplisit.

---

## 11. Diagram Alur `@include`

```
layout/app.blade.php
┌───────────────────────────────────────────────┐
│ <html>                                        │
│ <head>...</head>                              │
│ <body>                                        │
│                                               │
│   @include('partials.navbar')  ◄───────────┐  │
│       │                                    │  │
│       └─→ partials/navbar.blade.php        │  │
│           ┌──────────────────────────┐      │  │
│           │ <header>                 │      │  │
│           │   <nav>Produk | Dash</nav>│     │  │
│           │ </header>                │      │  │
│           └──────────────────────────┘      │  │
│               isi disisipkan ke sini ───────┘  │
│                                               │
│   <main>@yield('konten')</main>               │
│                                               │
│   @include('partials.footer')  ◄──────────┐   │
│       │                                    │   │
│       └─→ partials/footer.blade.php        │   │
│           ┌──────────────────────────┐     │   │
│           │ <footer>                 │     │   │
│           │   © 2026 Toko Bukhari    │     │   │
│           │ </footer>                │     │   │
│           └──────────────────────────┘     │   │
│               isi disisipkan ke sini ──────┘   │
│ </body>                                       │
└───────────────────────────────────────────────┘
```

---

## 12. Manfaat Nyata Saat ada Multi-Layout

Sekarang bayangkan kamu pakai **Solusi B** (tahap 6): dua layout, `layout/app.blade.php` dan `layout/admin.blade.php`.

Tanpa partial, navbar ditulis di kedua layout:

```
layout/app.blade.php:
    <header>Toko Bukhari | Produk | Dashboard</header>

layout/admin.blade.php:
    <header>Toko Bukhari | Produk | Dashboard</header>  ← DUPLIKAT
```

Dengan partial, cukup `@include('partials.navbar')` di kedua layout:

```
layout/app.blade.php:
    @include('partials.navbar')

layout/admin.blade.php:
    @include('partials.navbar')

partials/navbar.blade.php:
    <header>Toko Bukhari | Produk | Dashboard</header>  ← ditulis sekali
```

Saat navbar berubah, ubah **1 file** (`partials/navbar.blade.php`) → kedua layout ikut berubah.

> 📝 **Pesan mentor:**
> Inilah **skenario terbaik partial**: saat ada **multi-layout** yang butuh bagian sama. Untuk proyek dengan 1 layout saja, partial **opsional** — boleh dipakai untuk kerapian, boleh tidak.

---

## 13. Aturan Praktis: Kapan Pakai Partial?

| Skenario | Pakai partial? |
|---|---|
| Hanya 1 layout, navbar/footer pendek (5-10 baris) | **Tidak perlu**, biarkan di layout |
| Hanya 1 layout, navbar/footer panjang (20+ baris) | **Boleh**, untuk kerapian |
| Multi-layout yang butuh navbar sama | **Wajib**, menghindari duplikasi |
| Kartu produk yang muncul berkali-kali di 1 halaman | **Bukan partial**, gunakan component (tahap 8) |
| Form input tunggal yang dipakai di banyak form | **Partial cocok** (`partials/form-input.blade.php`) |

> 📝 **Pesan mentor:**
> Jangan **over-engineer**. Kalau navbar hanya 3 baris dan hanya 1 layout yang pakai, biarkan di layout. Partial bukan kewajiban, tapi alat bantu saat dibutuhkan.
>
> <!-- ponytail: partial untuk multi-layout atau navbar/footer panjang. Kalau cuma 1 layout & navbar pendek, biarkan di layout. -->

---

## 14. Contoh Lain Partial yang Umum

Selain navbar dan footer, partial sering dipakai untuk:

### Contoh 1: `partials/sidebar.blade.php`

Menu admin yang sama di banyak halaman admin:

```blade
<aside class="sidebar">
    <h3>Menu Admin</h3>
    <ul>
        <li><a href="{{ route('dashboard.index') }}">Dashboard</a></li>
        <li><a href="{{ route('produk.index') }}">Produk</a></li>
        <li><a href="/admin/user">User</a></li>
    </ul>
</aside>
```

Di layout admin:

```blade
@include('partials.sidebar')
```

### Contoh 2: `partials/pagination.blade.php`

Navigasi pagination yang dipakai di banyak halaman daftar:

```blade
<div class="pagination">
    {{ $items->links() }}
</div>
```

### Contoh 3: `partials/flash-message.blade.php`

Notifikasi flash (sukses/error) yang muncul setelah submit form:

```blade
@if (session('sukses'))
    <div class="alert alert-success">
        {{ session('sukses') }}
    </div>
@endif
```

Di layout:

```blade
@include('partials.flash-message')
```

---

## 15. Troubleshooting

### Error 1: `View [partials.navbar] not found`

**Penyebab:**
- Folder `partials/` belum dibuat, atau
- File bernama salah (mis: `Navbar.blade.php` — huruf besar).

**Solusi:** Cek path persis: `resources/views/partials/navbar.blade.php` (huruf kecil semua).

### Error 2: Variabel `$menuAktif` undefined di partial

**Penyebab:** Partial butuh variabel tapi tidak dikirim.

**Solusi:** Kirim lewat `@include('partials.navbar', ['menuAktif' => 'produk'])`. Atau pakai default di partial: `{{ $menuAktif ?? '' }}`.

### Error 3: Partial tampil dua kali

**Penyebab:** `@include` ditulis dua kali di layout, atau partial lama tidak dihapus dari layout saat dipindah.

**Solusi:** Hapus blok `<header>` / `<footer>` yang lama dari layout, biarkan hanya `@include`.

### Error 4: Looping di partial tidak jalan

**Penyebab:** Variabel loop tidak dikirim. Misal partial butuh `$produk` tapi controller tidak kirim.

**Solusi:** Kirim data dari controller, atau lewat `@include('partials.kartu-produk', ['produk' => $item])`.

---

## 16. Latihan Mandiri

**Latihan E:**

Buat partial `partials/sidebar.blade.php` yang berisi menu admin (Dashboard, Produk, User, Logout). Lalu tulis `@include` untuk memasangnya di `layout/admin.blade.php`.

<details>
<summary><strong>Lihat jawaban Latihan E</strong></summary>

**File `partials/sidebar.blade.php`:**

```blade
<aside style="width:200px; background:#f4f4f4; padding:15px;">
    <h3>Menu Admin</h3>
    <ul>
        <li><a href="{{ route('dashboard.index') }}">Dashboard</a></li>
        <li><a href="{{ route('produk.index') }}">Produk</a></li>
        <li><a href="/admin/user">User</a></li>
        <li>
            <form action="/logout" method="POST">
                @csrf
                <button type="submit">Logout</button>
            </form>
        </li>
    </ul>
</aside>
```

**Di `layout/admin.blade.php`** (ganti blok `<aside>` lama):

```blade
<div style="display:flex;">
    @include('partials.sidebar')

    <main style="flex:1;">
        @yield('konten')
    </main>
</div>
```

</details>

---

## 17. Istilah Kunci Tahap Ini

| Istilah | Arti sederhana |
|---|---|
| **Partial** | potongan kecil Blade di file sendiri (navbar, footer) |
| **`@include`** | directive untuk menyisipkan partial |
| **`partials/`** | folder konvensi untuk menyimpan partial |
| **Multi-layout** | lebih dari satu layout (app + admin) |
| **Menu aktif** | menu yang sedang disorot di navbar (butuh kirim `$menuAktif`) |

---

## 18. Rangkuman Tahap 7

1. **Partial** = potongan kecil Blade (navbar, footer, sidebar) di file sendiri.
2. Dipanggil lewat `@include('partials.navbar')`.
3. Dipakai saat: navbar/footer panjang, atau saat **multi-layout** butuh bagian sama.
4. Partial bisa terima data lewat `@include(..., ['key' => 'value'])`.
5. Secara default, partial mewarisi semua variabel dari view pemanggil.
6. Manfaat: layout lebih pendek, bagian reusable lintas layout, kolaborasi tim lebih mudah.
7. Jangan over-engineer: kalau navbar pendek dan hanya 1 layout, partial opsional.

---

## 19. Cek Pemahaman

1. Apa perbedaan **partial** dan **layout**?
2. Di folder mana partial biasanya disimpan?
3. Bagaimana cara menyisipkan partial `partials/navbar.blade.php` ke layout?
4. Bagaimana cara mengirim variabel `$menuAktif` ke partial?
5. Kapan partial **wajib** dipakai (bukan opsional)?
6. Apakah partial bisa berisi logika PHP seperti `@if` dan `@foreach`?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. **Layout** = kerangka utuh halaman (dipakai via `@extends`). **Partial** = potongan kecil (navbar, footer) yang disisipkan via `@include`. Partial tidak punya `<html>` atau `@extends`.
2. Di `resources/views/partials/` (huruf kecil, jamak).
3. `@include('partials.navbar')` — notasi titik, titik = slash.
4. `@include('partials.navbar', ['menuAktif' => 'produk'])` — kirim array asosiatif sebagai argumen kedua.
5. Saat ada **multi-layout** yang butuh navbar/footer/sidebar yang sama. Tanpa partial, harus duplikat di setiap layout.
6. **Ya**, partial bisa berisi logika Blade apa pun (`@if`, `@foreach`, `{{ }}`, dll). Tapi hindari query database di partial — kirim data dari controller.

</details>

---

## 20. Apakah Kamu Ingin Lanjut?

Di tahap 7 ini layout kamu sudah **lebih rapi** berkat partial. Sekarang navbar dan footer punya rumah sendiri.

Langkah berikutnya, kita masuk konsep yang lebih **powerful**:

> ### "Apakah kamu ingin lanjut ke langkah berikutnya: belajar Component untuk membuat kartu produk yang reusable?"
>
> Di tahap berikutnya kita akan:
>
> - memahami apa itu **component** dan bedanya dengan partial
> - membuat `components/kartu-produk.blade.php`
> - memakainya lewat `<x-kartu-produk :produk="$item" />`
> - belajar **props** (variabel input) dan kenapa component lebih terstruktur dari partial
> - menampilkan daftar produk dengan banyak kartu dari satu component

Jawab: **"Ya, lanjut"** untuk ke tahap 8,
atau **"Ulangi tahap 7"** kalau ada bagian yang masih perlu diperdalam.

---

> 📚 **Daftar Tahap (12. Layout Blade Reusable):**
> - ✅ Tahap 1 — Apa itu layout Blade reusable
> - ✅ Tahap 2 — Membuat file layout utama Blade
> - ✅ Tahap 3 — `@extends` dan `@section` untuk memakai layout
> - ✅ Tahap 4 — Mengubah halaman daftar produk agar pakai layout
> - ✅ Tahap 5 — Mengubah halaman tambah, edit, detail produk
> - ✅ Tahap 6 — Mengubah halaman dashboard admin
> - ✅ Tahap 7 — Partial: memecah navbar dan footer (kamu di sini)
> - ⏳ Tahap 8 — Component: membuat komponen kartu produk
> - ⏳ Tahap 9 — Slot: lubang yang bisa diisi apa saja
> - ⏳ Tahap 10 — Ringkasan dan best practice
