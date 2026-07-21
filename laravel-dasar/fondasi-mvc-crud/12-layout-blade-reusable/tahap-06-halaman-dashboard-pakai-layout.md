# Tahap 6 — Mengubah Halaman Dashboard Admin Agar Pakai Layout

> Mentor: Laravel Dasar — Fondasi MVC, CRUD
> Topik: 12. Layout Blade Reusable
> Fokus tahap ini: **memindahkan dashboard ke layout** + kasus khusus **sidebar** yang berbeda dari halaman produk.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Mengubah `dashboard/index.blade.php` agar memakai layout (sama seperti 4 halaman produk).
2. Menjelaskan **kasus khusus dashboard**: butuh sidebar yang tidak ada di halaman produk.
3. Menjelaskan **dua solusi** untuk sidebar berbeda (multi-layout vs `@yield` opsional).
4. Membuat **layout kedua** untuk admin (`layout/admin.blade.php`) saat dibutuhkan.
5. Mengirim data agregasi (count, sum) dari controller dashboard dengan `$title`.

Di tahap 4 dan 5 kamu sudah pindahkan 4 halaman produk. Sekarang giliran **dashboard**. Pola dasarnya sama, tapi ada **satu tantangan baru**: sidebar admin.

---

## 2. Analogi Sehari-Hari: Dua Jenis Kamar di Rumah Bapak

Rumah bapak punya banyak kamar. Sebagian **kamar tidur biasa** (tanpa meja kerja), sebagian **kamar kerja** (ada meja, kursi, lampu meja).

- Keluarga "Produk" tidur di kamar biasa → cukup berisi tempat tidur + lemari.
- Keluarga "Dashboard" butuh **meja kerja** → tidak cocok di kamar biasa.

Dua solusi untuk keluarga Dashboard:

**Solusi A — Buat kamar baru (layout kedua)**

Bapak bikin **kamar kerja** terpisah dengan meja bawaan. Keluarga Dashboard pindah ke kamar itu.

- Plus: rapi, semua kebutuhan kerja sudah disiapkan kamar.
- Minus: kalau header/footer sama, ada sedikit duplikat antar dua kamar.

**Solusi B — Tambah meja opsional di kamar biasa**

Kamar biasa disediakan **lubang opsional** untuk meja. Kalau keluarga butuh meja, mereka bawa sendiri. Kalau tidak, lubang itu kosong.

- Plus: satu kamar untuk semua, fleksibel.
- Minus: kalau beda halaman butuh banyak hal berbeda, lubang opsional bisa berantakan.

Di Laravel:

- **Solusi A** = bikin `layout/admin.blade.php` (layout kedua).
- **Solusi B** = tambah `@yield('sidebar')` opsional di `layout/app.blade.php`.

Kita akan bahas keduanya.

---

## 3. Lihat Dulu Bentuk Lama Dashboard

Sketsa `dashboard/index.blade.php` versi lama (tanpa layout):

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <title>Dashboard Admin</title>
    <style>
        .dashboard-grid { display: grid; grid-template-columns: 200px 1fr; }
        .sidebar { background: #f4f4f4; padding: 15px; }
        .card { background: white; padding: 20px; border: 1px solid #ddd; }
    </style>
</head>
<body>

    <header>
        <h1>Toko Bukhari</h1>
        <nav>...</nav>
    </header>

    <!-- ===== LAYOUT KHUSUS DASHBOARD: ada sidebar ===== -->
    <div class="dashboard-grid">

        <!-- SIDEBAR (unik untuk dashboard) -->
        <aside class="sidebar">
            <h3>Menu Admin</h3>
            <ul>
                <li><a href="{{ route('produk.index') }}">Produk</a></li>
                <li><a href="{{ route('dashboard.index') }}">Dashboard</a></li>
                <li><a href="/logout">Logout</a></li>
            </ul>
        </aside>

        <!-- KONTEN UTAMA (unik) -->
        <main>
            <h2>Dashboard</h2>

            <div class="card">
                <h3>Total Produk</h3>
                <p>{{ $totalProduk }}</p>
            </div>

            <div class="card">
                <h3>Total Pendapatan</h3>
                <p>Rp {{ number_format($totalPendapatan, 0, ',', '.') }}</p>
            </div>

            <h3>Produk Terbaru</h3>
            <ul>
                @foreach ($produkTerbaru as $item)
                    <li>{{ $item->nama }} — Rp {{ number_format($item->harga) }}</li>
                @endforeach
            </ul>
        </main>

    </div>

    <footer>...</footer>
</body>
</html>
```

### Tandai bagian

```
┌──────────────────────────────────────────┐
│ <html> ... <head> ... <style>            │ ❌ duplikat | ⚠ CSS unik
│ <header>...</header>                     │ ❌ duplikat
├──────────────────────────────────────────┤
│ <div class="dashboard-grid">             │ ⚠ wrapper unik
│   <aside class="sidebar">...</aside>     │ ⚠ SIDEBAR unik
│   <main>...dashboard content...</main>   │ ✅ konten unik
│ </div>                                   │ ⚠ wrapper unik
├──────────────────────────────────────────┤
│ <footer>...</footer>                     │ ❌ duplikat
│ </body></html>                           │ ❌ duplikat
└──────────────────────────────────────────┘
```

Perhatikan: dashboard punya **sidebar yang tidak ada di halaman produk**. Inilah masalah baru.

---

## 4. Solusi A: Tambah `@yield('sidebar')` Opsional di Layout Utama

Ini solusi paling sederhana untuk pemula. Kita **modifikasi layout utama** supaya mendukung sidebar opsional.

### Langkah 1: Ubah `layout/app.blade.php`

Tambahkan blok **opsional** untuk sidebar. Kita pakai directive `@hasSection` untuk mengecek apakah halaman mengirim sidebar atau tidak.

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

    <header>
        <h1>Toko Bukhari</h1>
        <nav>
            <a href="{{ route('produk.index') }}">Produk</a>
            <a href="{{ route('dashboard.index') }}">Dashboard</a>
        </nav>
    </header>

    <div style="display:flex;">
        {{-- SIDEBAR OPSIONAL: hanya muncul kalau halaman mengirim @section('sidebar') --}}
        @hasSection('sidebar')
            <aside style="width:200px; background:#f4f4f4; padding:15px;">
                @yield('sidebar')
            </aside>
        @endif

        <main style="flex:1;">
            @yield('konten')
        </main>
    </div>

    <footer>
        <p>&copy; {{ date('Y') }} Toko Bukhari. Semua hak dilindungi.</p>
    </footer>

    @stack('js')

</body>
</html>
```

### Penjelasan bagian baru

**`@hasSection('sidebar') ... @endif`**

Directive ini **mengecek** apakah halaman menyediakan section bernama `sidebar`. Jika ya, tampilkan blok `<aside>`. Jika tidak, **skip** (tidak error).

**`@yield('sidebar')`**

Lubang untuk sidebar. Sama seperti `@yield('konten')`, tapi **hanya diisi** kalau halaman menulis `@section('sidebar')`. Kalau tidak, kosong.

### Langkah 2: Ubah `dashboard/index.blade.php`

```blade
@extends('layout.app', ['title' => 'Dashboard Admin'])

@push('css')
    <style>
        .card { background: white; padding: 20px; border: 1px solid #ddd; }
    </style>
@endpush

@section('sidebar')
    <h3>Menu Admin</h3>
    <ul>
        <li><a href="{{ route('produk.index') }}">Produk</a></li>
        <li><a href="{{ route('dashboard.index') }}">Dashboard</a></li>
        <li><a href="/logout">Logout</a></li>
    </ul>
@endsection

@section('konten')
    <h2>Dashboard</h2>

    <div class="card">
        <h3>Total Produk</h3>
        <p>{{ $totalProduk }}</p>
    </div>

    <div class="card">
        <h3>Total Pendapatan</h3>
        <p>Rp {{ number_format($totalPendapatan, 0, ',', '.') }}</p>
    </div>

    <h3>Produk Terbaru</h3>
    <ul>
        @foreach ($produkTerbaru as $item)
            <li>{{ $item->nama }} — Rp {{ number_format($item->harga) }}</li>
        @endforeach
    </ul>
@endsection
```

### Penjelasan

Dashboard sekarang mengisi **dua lubang**:

1. `@section('sidebar')` → menu admin di kiri.
2. `@section('konten')` → isi dashboard di kanan.

Halaman produk **tidak** menulis `@section('sidebar')`, jadi `@hasSection('sidebar')` bernilai false, dan `<aside>` tidak dirender. Halaman produk tetap tampil tanpa sidebar, seperti biasa.

### Controller `DashboardController@index`

```php
public function index()
{
    return view('dashboard.index', [
        'totalProduk'     => Produk::count(),
        'totalPendapatan' => Produk::sum('harga'),
        'produkTerbaru'   => Produk::latest()->take(5)->get(),
        'title'           => 'Dashboard Admin',
    ]);
}
```

> 📝 **Pesan mentor:**
> Solusi A cocok kalau **hanya dashboard** yang butuh sidebar. Kalau semua halaman admin (produk + dashboard + kategori + user management) butuh sidebar yang sama, gunakan **Solusi B** (layout kedua).

---

## 5. Solusi B: Layout Kedua untuk Admin

Solusi B dipakai kalau banyak halaman admin butuh **struktur berbeda** dari halaman publik. Misalnya:

- Halaman **publik**: tanpa sidebar, lebar penuh.
- Halaman **admin** (dashboard, kelola produk, kelola user): semua butuh sidebar yang sama.

Daripada tiap halaman admin nulis `@section('sidebar')` berulang, lebih baik bikin **layout kedua** khusus admin.

### Struktur file

```
resources/views/layout/
├── app.blade.php        ← layout publik (header + footer, tanpa sidebar)
└── admin.blade.php      ← layout admin (header + footer + sidebar tetap)
```

### Isi `layout/admin.blade.php`

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ $title ?? 'Admin - Toko Bukhari' }}</title>
    @stack('css')
</head>
<body>

    <header>
        <h1>Admin Toko Bukhari</h1>
        <nav>
            <a href="{{ route('dashboard.index') }}">Dashboard</a>
            <a href="{{ route('produk.index') }}">Produk</a>
            <a href="/logout">Logout</a>
        </nav>
    </header>

    <div style="display:flex;">
        {{-- SIDEBAR TETAP, tidak opsional --}}
        <aside style="width:200px; background:#f4f4f4; padding:15px;">
            <h3>Menu Admin</h3>
            <ul>
                <li><a href="{{ route('dashboard.index') }}">Dashboard</a></li>
                <li><a href="{{ route('produk.index') }}">Kelola Produk</a></li>
                <li><a href="/admin/user">Kelola User</a></li>
                <li><a href="/logout">Logout</a></li>
            </ul>
        </aside>

        <main style="flex:1;">
            @yield('konten')
        </main>
    </div>

    <footer>
        <p>&copy; {{ date('Y') }} Admin Toko Bukhari.</p>
    </footer>

    @stack('js')

</body>
</html>
```

### Halaman dashboard pakai `layout.admin`

```blade
@extends('layout.admin', ['title' => 'Dashboard Admin'])

@push('css')
    <style>
        .card { background: white; padding: 20px; border: 1px solid #ddd; }
    </style>
@endpush

@section('konten')
    <h2>Dashboard</h2>

    <div class="card">
        <h3>Total Produk</h3>
        <p>{{ $totalProduk }}</p>
    </div>

    {{-- dst --}}
@endsection
```

### Yang berbeda dari Solusi A

| Hal | Solusi A (yield opsional) | Solusi B (layout kedua) |
|---|---|---|
| Jumlah layout | 1 (`layout.app`) | 2 (`layout.app` + `layout.admin`) |
| Sidebar | opsional lewat `@hasSection` | tetap di `layout.admin` |
| Halaman dashboard | tulis `@section('sidebar')` + `@section('konten')` | cukup `@section('konten')` |
| Cocok untuk | 1-2 halaman butuh sidebar | banyak halaman admin butuh sidebar sama |

---

## 6. Kapan Pakai Solusi A vs Solusi B?

Decision tree sederhana:

```
Apakah lebih dari 2-3 halaman butuh sidebar yang sama?
├── YA  → Solusi B (layout kedua)
└── TIDAK (hanya dashboard) → Solusi A (yield opsional)
```

### Contoh kasus nyata

| Skenario | Pilih |
|---|---|
| Hanya dashboard yang punya sidebar | Solusi A |
| Dashboard + kelola produk + kelola user, semua butuh sidebar sama | Solusi B |
| Berbeda halaman butuh sidebar berbeda | Solusi A (section berbeda per halaman) |
| Sidebar selalu sama di semua halaman admin | Solusi B (lebih DRY) |

> 📝 **Pesan mentor:**
> Untuk proyek belajar yang sekarang, **Solusi A sudah cukup**. Kalau nanti proyek tumbuh jadi panel admin yang kompleks, baru pertimbangkan Solusi B. Jangan over-engineer di awal.
>
> <!-- ponytail: Solusi A cukup untuk MVP. Pindah ke Solusi B saat ada 3+ halaman admin. -->

---

## 7. Diagram Solusi A: Satu Layout dengan Sidebar Opsional

```
layout/app.blade.php
┌────────────────────────────────────────────────┐
│ <header>Toko Bukhari | Produk | Dashboard</header>│
├────────────┬───────────────────────────────────┤
│            │                                   │
│ @yield(    │ @yield('konten')                  │
│  'sidebar')│                                   │
│            │                                   │
│  ↑ OPSIONAL│  ↑ WAJIB                          │
│  (kalau    │                                   │
│  halaman   │                                   │
│  kirim)    │                                   │
│            │                                   │
├────────────┴───────────────────────────────────┤
│ <footer>...</footer>                           │
└────────────────────────────────────────────────┘

Halaman PRODUK (tidak kirim sidebar):
  @section('konten') ... @endsection
  → sidebar kosong, halaman penuh lebar

Halaman DASHBOARD (kirim sidebar):
  @section('sidebar') ... @endsection   ← kiri terisi
  @section('konten')   ... @endsection   ← kanan terisi
```

## 8. Diagram Solusi B: Dua Layout Terpisah

```
layout/app.blade.php          layout/admin.blade.php
┌─────────────────────┐      ┌─────────────────────┐
│ <header>            │      │ <header>Admin       │
├─────────────────────┤      ├─────────┬───────────┤
│                     │      │ SIDEBAR │           │
│ @yield('konten')    │      │ TETAP   │ @yield    │
│                     │      │ (sama   │ ('konten')│
│                     │      │  untuk  │           │
│                     │      │  semua  │           │
│                     │      │  halaman│           │
│                     │      │  admin) │           │
├─────────────────────┤      ├─────────┴───────────┤
│ <footer>            │      │ <footer>Admin       │
└─────────────────────┘      └─────────────────────┘
        ▲                              ▲
        │                              │
   halaman publik                halaman admin
   (produk, kontak)              (dashboard, kelola user)
```

---

## 9. Troubleshooting

### Error 1: Sidebar muncul di halaman produk

**Penyebab (Solusi A):** Halaman produk tidak sengaja menulis `@section('sidebar')`.

**Solusi:** Hapus `@section('sidebar')` dari halaman yang tidak butuh sidebar. Atau cek `@hasSection('sidebar')` di layout sudah benar.

### Error 2: Sidebar tidak muncul di dashboard

**Penyebab (Solusi A):**
- Lupa menulis `@section('sidebar')` di dashboard, atau
- Lupa `@hasSection('sidebar') ... @endif` di layout, atau
- Nama section tidak cocok.

**Solusi:** Cek nama section `'sidebar'` konsisten di layout dan halaman.

### Error 3 (Solusi B): Dashboard masih tampil tanpa sidebar

**Penyebab:** Dashboard masih pakai `@extends('layout.app')` (layout publik), bukan `@extends('layout.admin')`.

**Solusi:** Ubah jadi `@extends('layout.admin')`.

### Error 4: Header berbeda antara layout publik dan admin

**Penyebab (Solusi B):** Wajar, karena dua layout sengaja dibuat berbeda.

**Solusi:** Ini **bukan error**, ini fitur. Tapi kalau header mau sama, pindahkan header ke **partial** (akan dipelajari di tahap 7).

---

## 10. Latihan Mandiri

**Latihan D:**

Misalkan kamu punya halaman "Kelola User" (`admin/user/index.blade.php`) yang juga butuh sidebar seperti dashboard. Manakah solusi yang lebih cocok: A atau B? Tuliskan `@extends` yang dipakai.

<details>
<summary><strong>Lihat jawaban Latihan D</strong></summary>

Karena sekarang ada **2 halaman admin** (dashboard + kelola user) yang butuh sidebar sama, **Solusi B lebih cocok**. Pakai `@extends('layout.admin')`:

```blade
@extends('layout.admin', ['title' => 'Kelola User'])

@section('konten')
    <h2>Daftar User</h2>
    <table>...</table>
@endsection
```

</details>

---

## 11. Istilah Kunci Tahap Ini

| Istilah | Arti sederhana |
|---|---|
| **Sidebar** | menu samping, biasanya khas halaman admin |
| **`@hasSection`** | cek apakah halaman mengirim section tertentu (untuk kondisional) |
| **Layout kedua** | file layout terpisah untuk tipe halaman berbeda (admin vs publik) |
| **Over-engineering** | membuat terlalu rumit untuk kebutuhan yang belum ada (jangan!) |

---

## 12. Rangkuman Tahap 6

1. Halaman dashboard bisa dipindahkan ke layout dengan pola yang sama seperti halaman produk.
2. Tantangan baru: dashboard butuh **sidebar** yang tidak ada di halaman produk.
3. **Solusi A**: tambah `@yield('sidebar')` + `@hasSection('sidebar')` di layout utama. Cocok untuk 1-2 halaman.
4. **Solusi B**: bikin `layout/admin.blade.php` terpisah dengan sidebar tetap. Cocok untuk 3+ halaman admin.
5. Decision tree: lebih dari 2-3 halaman butuh sidebar sama → Solusi B; kalau hanya dashboard → Solusi A.
6. Jangan **over-engineer** di awal. Solusi A cukup untuk proyek belajar.

---

## 13. Cek Pemahaman

1. Apa **kasus khusus** dashboard yang tidak ada di halaman produk?
2. Apa fungsi directive `@hasSection('sidebar')`?
3. Kapan harus pakai **Solusi B** (layout kedua)?
4. Apa risiko pakai Solusi B terlalu dini (saat hanya 1 halaman butuh sidebar)?
5. Di Solusi A, apa yang terjadi kalau halaman produk **tidak** menulis `@section('sidebar')`?
6. Apa perbedaan `@yield('konten')` (wajib) vs `@yield('sidebar')` (opsional di Solusi A)?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. Dashboard butuh **sidebar** (menu admin di samping) yang tidak ada di halaman produk biasa.
2. Mengecek apakah halaman mengirim section bernama `'sidebar'`. Jika ya, blok di dalamnya dirender. Jika tidak, dilewati. Berguna untuk sidebar opsional.
3. Saat **lebih dari 2-3 halaman** butuh sidebar yang sama persis. Bikin layout kedua menghindari duplikasi `@section('sidebar')` di banyak file.
4. **Over-engineering**. Layout kedua berarti duplikasi header/footer di dua file layout. Kalau hanya 1 halaman butuh sidebar, Solusi A jauh lebih sederhana.
5. `@hasSection('sidebar')` bernilai **false**, jadi blok `<aside>` tidak dirender. Halaman produk tampil tanpa sidebar, lebar penuh. Tidak error.
6. `@yield('konten')` wajib diisi (setiap halaman punya konten utama). `@yield('sidebar')` **opsional** — kalau tidak diisi, sidebar tidak muncul, halaman tetap berfungsi.

</details>

---

## 14. Apakah Kamu Ingin Lanjut?

Di tahap 6 ini **semua 5 halaman** (4 produk + dashboard) sudah memakai layout. Kerangka Blade sudah reusable.

Langkah berikutnya, kita masuk ke topik lanjutan Blade:

> ### "Apakah kamu ingin lanjut ke langkah berikutnya: belajar Partial untuk memecah navbar dan footer?"
>
> Di tahap berikutnya kita akan:
>
> - memahami apa itu **partial** dan kapan dipakai
> - memecah navbar dari layout menjadi file `partials/navbar.blade.php`
> - memecah footer menjadi `partials/footer.blade.php`
> - meng-include partial lewat `@include('partials.navbar')`
> - manfaat: kalau dua layout (app & admin) butuh navbar sama, partial menghindari duplikasi

Jawab: **"Ya, lanjut"** untuk ke tahap 7,
atau **"Ulangi tahap 6"** kalau ada bagian yang masih perlu diperdalam.

---

> 📚 **Daftar Tahap (12. Layout Blade Reusable):**
> - ✅ Tahap 1 — Apa itu layout Blade reusable
> - ✅ Tahap 2 — Membuat file layout utama Blade
> - ✅ Tahap 3 — `@extends` dan `@section` untuk memakai layout
> - ✅ Tahap 4 — Mengubah halaman daftar produk agar pakai layout
> - ✅ Tahap 5 — Mengubah halaman tambah, edit, detail produk
> - ✅ Tahap 6 — Mengubah halaman dashboard admin (kamu di sini)
> - ⏳ Tahap 7 — Partial: memecah navbar dan footer
> - ⏳ Tahap 8 — Component: membuat komponen kartu produk
> - ⏳ Tahap 9 — Slot: lubang yang bisa diisi apa saja
> - ⏳ Tahap 10 — Ringkasan dan best practice
