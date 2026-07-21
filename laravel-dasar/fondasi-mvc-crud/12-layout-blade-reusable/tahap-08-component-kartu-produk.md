# Tahap 8 — Component: Membuat Komponen Kartu Produk

> Mentor: Laravel Dasar — Fondasi MVC, CRUD
> Topik: 12. Layout Blade Reusable
> Fokus tahap ini: **memahami component** dan membuat kartu produk yang bisa dipakai berulang-ulang.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Menjelaskan apa itu **component** dalam Blade dan bedanya dengan partial.
2. Membuat **class-based component** atau **anonymous component**.
3. Mendefinisikan **props** (variabel input) di component.
4. Memakai component lewat tag `<x-kartu-produk :produk="$item" />`.
5. Mengubah halaman daftar produk agar pakai kartu produk berulang.
6. Menjelaskan kapan pakai **component vs partial**.

Component adalah konsep Blade yang lebih **modern** (Laravel 7+). Ini tools paling powerful untuk reusable UI.

---

## 2. Analogi Sehari-Hari: Stempel Bonus Kasir

Bayangkan kamu kasir toko. Setiap kali pelanggan belanja, kamu harus:

1. Tulis "Terima kasih!" di struk.
2. Tulis tanggal.
3. Tulis nama kasir.

Tiga hal ini diulang **setiap transaksi**. Capek dan rawan salah.

Solusi: bikin **stempel bonus**. Stempel itu sudah berisi teks tetap ("Terima kasih!"), dengan **ruang kosong** untuk tanggal dan nama kasir yang berubah setiap transaksi.

```
┌─────────────────────────────┐
│      Terima kasih!          │  ← teks tetap (dari stempel)
│  Tanggal: ____  Kasir: ____ │  ← ruang kosong (diisi tiap transaksi)
└─────────────────────────────┘
```

Saat transaksi, kamu tinggal **cap stempel + isi tanggal & nama kasir**.

Di Blade:

- **Stempel** = **component** (kode template siap pakai).
- **Teks tetap** = bagian HTML yang tidak berubah di setiap pemakaian.
- **Ruang kosong** = **props** (variabel input yang berubah tiap pemakaian).
- **Cap + isi** = pakai component: `<x-kartu-produk :produk="$item" />`.

> 📝 **Pesan mentor:**
- Di tahap 7 (partial) kita membuat **navbar** yang dipakai **sekali** per halaman. Sekarang di tahap 8 kita bikin **kartu produk** yang dipakai **berkali-kali** di halaman yang sama. Itulah inti component.

---

## 3. Apa Itu Component?

### Definisi sederhana

> **Component** adalah **potongan Blade reusable** yang punya **props** (input), mirip seperti **fungsi** di PHP. Component dipakai lewat tag khusus: `<x-nama-component />`.

### Ciri khas component

- Lokasi default: `resources/views/components/`.
- Dipanggil lewat tag HTML: `<x-kartu-produk />` (bukan `@include`).
- Bisa terima **props** lewat atribut: `<x-kartu-produk :produk="$item" />`.
- Bisa punya **slot** (akan dibahas tahap 9).
- Bisa versi **anonymous** (cuma file Blade) atau **class-based** (file Blade + file PHP).

### Perbedaan partial vs component

| Hal | Partial | Component |
|---|---|---|
| Cara pakai | `@include('partials.kartu')` | `<x-kartu />` |
| Input data | `@include(..., ['key' => $val])` | `<x-kartu :key="$val" />` atau `key="val"` |
| Sintaks | Seperti fungsi PHP | Seperti tag HTML |
| Cocok untuk | Bagian yang muncul **sekali** (navbar, footer) | Bagian yang muncul **berkali-kali** (kartu produk, tombol, badge) |
| Validasi input | Tidak ada | Bisa dideklarasikan lewat props (terstruktur) |
| Kesenian Laravel | Lebih lama (Laravel 4+) | Modern (Laravel 7+) |

> 📝 **Pesan mentor:**
> Aturan praktis: kalau bagian muncul **sekali** per halaman → partial. Kalau muncul **berkali-kali** per halaman (seperti kartu produk dalam daftar) → component.

---

## 4. Kapan Component Lebih Cocok dari Partial?

Pikirkan halaman **daftar produk**. Saat ini (tahap 4) kamu pakai `@foreach`:

```blade
@section('konten')
    <h2>Daftar Produk</h2>
    <div class="grid">
        @foreach ($produk as $item)
            <div class="kartu">
                <img src="{{ asset('storage/' . $item->gambar) }}" alt="{{ $item->nama }}">
                <h3>{{ $item->nama }}</h3>
                <p>Rp {{ number_format($item->harga, 0, ',', '.') }}</p>
                <a href="{{ route('produk.show', $item->id) }}">Lihat Detail</a>
            </div>
        @endforeach
    </div>
@endsection
```

Sekarang bayangkan struktur kartu yang sama dipakai di **halaman lain**:

- Halaman **produk favorit** → kartu produk favorit.
- Halaman **rekomendasi** di dashboard → kartu produk terbaru.
- Halaman **kategori** → kartu produk dalam kategori.

Tanpa component, kamu **copy-paste** kode kartu ke setiap halaman. Duplikat lagi!

Solusi: bikin **component kartu produk** sekali, pakai di mana saja.

---

## 5. Langkah 1: Buat File Component (Anonymous Component)

Kita mulai dengan **anonymous component** (paling sederhana, tanpa class PHP). Cukup bikin satu file Blade.

### Lokasi file

```
resources/views/
├── components/                    ← folder khusus component
│   └── kartu-produk.blade.php     ← BARU
├── layout/
├── partials/
└── produk/
```

> 🪤 **Jebakan pemula:**
> Nama file pakai **kebab-case** (huruf kecil dengan tanda hubung): `kartu-produk.blade.php`. Bukan `KartuProduk`, bukan `kartu_produk`. Ini konvensi Laravel.

### Isi `components/kartu-produk.blade.php`

```blade
@props(['produk'])

<div class="kartu">
    <img src="{{ asset('storage/' . $produk->gambar) }}"
         alt="{{ $produk->nama }}"
         width="200">

    <h3>{{ $produk->nama }}</h3>

    <p class="harga">Rp {{ number_format($produk->harga, 0, ',', '.') }}</p>

    @if ($produk->deskripsi)
        <p class="deskripsi">{{ Str::limit($produk->deskripsi, 50) }}</p>
    @endif

    <a href="{{ route('produk.show', $produk->id) }}">Lihat Detail</a>
</div>
```

### Penjelasan baris per baris

**Baris 1:** `@props(['produk'])`

Ini **directive khusus component**. Fungsinya: mendeklarasikan **props** (input) yang diterima component.

- `'produk'` artinya component ini menerima 1 props bernama `produk`.
- Setelah ini, variabel `$produk` **otomatis tersedia** di dalam component.

**Baris 3-18:** struktur kartu

- `<div class="kartu">` → pembungkus kartu.
- `<img>` → gambar produk dari field `gambar`.
- `<h3>{{ $produk->nama }}</h3>` → nama produk.
- `<p class="harga">...</p>` → harga dengan format Rupiah.
- `@if ($produk->deskripsi)` → tampilkan deskripsi hanya kalau ada.
- `Str::limit($produk->deskripsi, 50)` → potong deskripsi jadi maksimal 50 karakter (helper Laravel).
- `<a href="...">Lihat Detail</a>` → link ke halaman detail produk.

> 📝 **Pesan mentor:**
> Perhatikan: di dalam component, kita **tidak** pakai `@foreach`. Component cuma **satu kartu**. Looping dilakukan di halaman pemanggil, bukan di dalam component. Ini membuat component **fokus** ke satu hal: merender satu kartu produk.

---

## 6. Langkah 2: Pakai Component di Halaman Daftar Produk

Sekarang ubah `produk/index.blade.php`:

```blade
@extends('layout.app', ['title' => 'Daftar Produk'])

@section('konten')
    <h2>Daftar Produk</h2>
    <a href="{{ route('produk.create') }}">+ Tambah Produk</a>

    <div class="grid">
        @foreach ($produk as $item)
            <x-kartu-produk :produk="$item" />
        @endforeach
    </div>

    {{-- tabel lama bisa dihapus, diganti kartu --}}
@endsection
```

### Penjelasan pemakaian

**`<x-kartu-produk :produk="$item" />`**

- `<x-` → prefix wajib untuk semua component.
- `kartu-produk` → nama component (sama dengan nama file `kartu-produk.blade.php`, tanpa `.blade.php`).
- `:produk="$item"` → kirim props `produk` dengan nilai `$item` dari loop.
- `/>` → penutup self-closing (component tanpa isi di dalam).

### Cara kerja

Saat Blade melihat `<x-kartu-produk :produk="$item" />`:

1. Cari file `components/kartu-produk.blade.php`.
2. Baca props `produk` → isi dengan `$item`.
3. Jalankan kode di dalam component (gambar, nama, harga, dll).
4. Sisipkan hasilnya ke lokasi tag `<x-...>`.

Karena ini ada di dalam `@foreach`, component dipanggil **sekali per produk**. Hasilnya: banyak kartu produk dirender berurutan.

---

## 7. Tanda Titik Dua (`:`) vs Tanpa Titik Dua

Penting untuk dipahami. Ada **dua cara** kirim props:

### Cara 1: `:props="$variabel"` (dengan titik dua)

```blade
<x-kartu-produk :produk="$item" />
```

- Titik dua di awal → **evaluasi sebagai PHP**.
- `$item` dievaluasi jadi nilai variabel `$item` (object Produk).
- Cocok untuk kirim **variabel** atau **ekspresi PHP**.

### Cara 2: `props="nilai"` (tanpa titik dua)

```blade
<x-tombol warna="merah" />
```

- Tanpa titik dua → **diperlakukan sebagai string**.
- `"merah"` dikirim sebagai teks literal "merah".
- Cocok untuk kirim **nilai tetap** (string, angka).

### Contoh kombinasi

```blade
<x-kartu-produk :produk="$item" ukuran="kecil" />
```

- `:produk="$item"` → kirim variabel `$item` sebagai props `produk`.
- `ukuran="kecil"` → kirim string literal "kecil" sebagai props `ukuran`.

Di component, tangkap keduanya di `@props`:

```blade
@props(['produk', 'ukuran' => 'sedang'])  // 'sedang' = nilai default

<div class="kartu kartu-{{ $ukuran }}">
    {{-- ... --}}
</div>
```

> 📝 **Pesan mentor:**
> Aturan praktis: **kirim variabel/ekspresi PHP → pakai `:`**. **Kirim string/angka tetap → tidak perlu `:`**. Kalau ragu, pakai `:` lebih aman.

---

## 8. Props dengan Nilai Default

Kadang component butuh **opsional props** dengan nilai default kalau tidak dikirim.

Contoh: kartu produk bisa ukuran "kecil", "sedang", "besar", defaultnya "sedang".

```blade
@props([
    'produk',
    'ukuran' => 'sedang',          // default
    'tampilkanDeskripsi' => true,  // default
])

<div class="kartu kartu-{{ $ukuran }}">
    <img src="{{ asset('storage/' . $produk->gambar) }}" alt="{{ $produk->nama }}">

    <h3>{{ $produk->nama }}</h3>
    <p>Rp {{ number_format($produk->harga) }}</p>

    @if ($tampilkanDeskripsi && $produk->deskripsi)
        <p>{{ Str::limit($produk->deskripsi, 50) }}</p>
    @endif
</div>
```

Saat dipanggil:

```blade
{{-- pakai semua default --}}
<x-kartu-produk :produk="$item" />

{{-- ganti ukuran jadi besar --}}
<x-kartu-produk :produk="$item" ukuran="besar" />

{{-- sembunyikan deskripsi --}}
<x-kartu-produk :produk="$item" :tampilkanDeskripsi="false" />
```

> 🪤 **Jebakan pemula:**
> `:tampilkanDeskripsi="false"` harus pakai `:` karena `false` adalah boolean PHP. Tanpa `:`, `"false"` akan jadi string "false" (truthy!).

---

## 9. Pakai Component di Halaman Lain

Setelah component jadi, kamu bisa pakai di **halaman apapun** tanpa duplikasi kode.

### Contoh: Halaman rekomendasi di dashboard

```blade
{{-- dashboard/index.blade.php --}}
@extends('layout.app', ['title' => 'Dashboard Admin'])

@section('konten')
    <h2>Dashboard</h2>

    <h3>Produk Rekomendasi</h3>
    <div class="grid">
        @foreach ($produkTerbaru as $item)
            <x-kartu-produk :produk="$item" ukuran="kecil" />
        @endforeach
    </div>
@endsection
```

### Contoh: Halaman detail produk (kartu terkait)

```blade
{{-- produk/show.blade.php --}}
@extends('layout.app')

@section('konten')
    {{-- ...detail produk utama... --}}

    <h3>Produk Terkait</h3>
    <div class="grid">
        @foreach ($produkTerkait as $item)
            <x-kartu-produk :produk="$item" :tampilkanDeskripsi="false" />
        @endforeach
    </div>
@endsection
```

Satu component, dipakai di tiga halaman berbeda. **Tulis sekali, pakai banyak.**

---

## 10. Diagram Alur Component

```
Halaman produk/index.blade.php
┌─────────────────────────────────────────────┐
│ @foreach ($produk as $item)                 │
│   <x-kartu-produk :produk="$item" /> ───┐   │
│ @endforeach                             │   │
└─────────────────────────────────────────┼───┘
                                          │
                                          ▼
            components/kartu-produk.blade.php
            ┌──────────────────────────────────┐
            │ @props(['produk'])               │ ← terima props
            │                                  │
            │ <div class="kartu">              │
            │   <img src="...gambar...">       │
            │   <h3>{{ $produk->nama }}</h3>   │ ← pakai $produk
            │   <p>Rp ...</p>                  │
            │   <a>Lihat Detail</a>            │
            │ </div>                           │
            └──────────────────────────────────┘
                          │
                          ▼
              output kartu HTML utuh
              disisipkan ke lokasi <x-...>
```

Setiap iterasi `@foreach` menghasilkan **satu kartu**. Lima produk → lima kartu.

---

## 11. Class-Based Component (Opsional, Lebih Lanjut)

Selain anonymous component, Laravel mendukung **class-based component** yang punya file PHP terpisah. Berguna kalau component butuh **logika kompleks**.

### Membuat dengan Artisan

```bash
php artisan make:component KartuProduk
```

Perintah ini membuat **dua file**:

```
app/View/Components/KartuProduk.php         ← class PHP
resources/views/components/kartu-produk.blade.php   ← view Blade
```

### Isi class PHP

```php
namespace App\View\Components;

use Illuminate\View\Component;

class KartuProduk extends Component
{
    public $produk;
    public $ukuran;

    public function __construct($produk, $ukuran = 'sedang')
    {
        $this->produk = $produk;
        $this->ukuran = $ukuran;
    }

    public function render()
    {
        return view('components.kartu-produk');
    }
}
```

### Kapan pakai class-based?

- Component butuh **logika kompleks** (kalkulasi, query ringan, helper khusus).
- Ingin **validasi props** ketat.
- Ingin method helper yang bisa dipanggil di view.

> 📝 **Pesan mentor:**
> Untuk **pemula**, anonymous component (cuma file Blade) **cukup**. Class-based dibutuhkan kalau component sudah kompleks. Jangan over-engineer.
>
> <!-- ponytail: anonymous component cukup untuk MVP. Pakai class-based saat ada logika non-trivial. -->

---

## 12. Perbandingan Lengkap: Partial vs Component vs Layout

Sekarang kamu sudah kenal ketiganya. Rangkuman:

| Konsep | Cara pakai | Cocok untuk | Input data |
|---|---|---|---|
| **Layout** | `@extends('layout.app')` | kerangka halaman (1 per halaman) | `@section`, `@yield` |
| **Partial** | `@include('partials.x')` | bagian yang muncul **sekali** (navbar, footer) | array di `@include` |
| **Component** | `<x-nama />` | bagian yang muncul **berkali-kali** (kartu, tombol) | props & slot |

### Analogi akhir

- **Layout** = cetakan kue besar (bikin 1 loyang = 1 halaman).
- **Partial** = cetakan kecil yang dipakai 1 kali per loyang (hiasan atas).
- **Component** = cetakan kecil yang dipakai berkali-kali per loyang (motif bunga yang berulang).

---

## 13. Troubleshooting

### Error 1: `Unable to find component [kartu-produk]`

**Penyebab:**
- Folder `components/` belum dibuat.
- File bernama salah (mis: `KartuProduk.blade.php` — harus kebab-case).

**Solusi:** Cek path: `resources/views/components/kartu-produk.blade.php`.

### Error 2: `Undefined variable $produk` di dalam component

**Penyebab:** Lupa mendeklarasikan `@props(['produk'])` di awal component, atau nama props salah.

**Solusi:** Tambah `@props(['produk'])` di baris pertama component. Pastikan nama cocok dengan atribut yang dikirim (`:produk="..."`).

### Error 3: String "false" diperlakukan sebagai true

**Penyebab:** Pakai `tampilkanDeskripsi="false"` (tanpa `:`) — `"false"` adalah string truthy.

**Solusi:** Pakai `:tampilkanDeskripsi="false"` (dengan `:`) supaya dievaluasi sebagai boolean PHP.

### Error 4: Component tampil tapi datanya kosong

**Penyebab:** Variabel yang dikirim lewat `:produk="$item"` ternyata null di iterasi tertentu.

**Solusi:** Cek data dari controller. Tambah `@if ($produk)` di dalam component untuk handle null.

---

## 14. Latihan Mandiri

**Latihan F:**

Buat component `<x-badge-status :aktif="$produk->aktif" />` yang menampilkan badge hijau "Aktif" kalau `$produk->aktif == true`, atau badge merah "Nonaktif" kalau false.

<details>
<summary><strong>Lihat jawaban Latihan F</strong></summary>

**File `components/badge-status.blade.php`:**

```blade
@props(['aktif'])

@if ($aktif)
    <span style="background:green; color:white; padding:2px 8px; border-radius:4px;">
        Aktif
    </span>
@else
    <span style="background:red; color:white; padding:2px 8px; border-radius:4px;">
        Nonaktif
    </span>
@endif
```

**Pemakaian:**

```blade
<p>Status: <x-badge-status :aktif="$produk->aktif" /></p>
```

</details>

---

## 15. Istilah Kunci Tahap Ini

| Istilah | Arti sederhana |
|---|---|
| **Component** | potongan Blade reusable dengan props, dipakai via `<x-...>` |
| **Props** | input/variabel yang diterima component |
| **`@props([...])`** | directive untuk deklarasi props di anonymous component |
| **`:` (titik dua)** | penanda atribut dievaluasi sebagai PHP |
| **kebab-case** | konvensi nama file component: huruf kecil + tanda hubung |
| **Anonymous component** | component tanpa class PHP (cuma file Blade) |
| **Class-based component** | component dengan class PHP terpisah |

---

## 16. Rangkuman Tahap 8

1. **Component** = potongan Blade reusable yang punya **props**, dipakai lewat `<x-nama />`.
2. Cocok untuk bagian yang muncul **berkali-kali** (kartu produk, tombol, badge).
3. Anonymous component: cukup file Blade dengan `@props([...])`.
4. Props dikirim lewat atribut: `:variabel="$val"` atau `tetap="nilai"`.
5. Tanda `:` berarti evaluasi PHP, tanpa `:` berarti string literal.
6. Props bisa punya **nilai default**: `@props(['ukuran' => 'sedang'])`.
7. Component bisa dipakai di banyak halaman → **tulis sekali, pakai banyak**.
8. Class-based component untuk logika kompleks (opsional untuk pemula).

---

## 17. Cek Pemahaman

1. Apa perbedaan **partial** dan **component**?
2. Bagaimana cara membuat component anonymous bernama `kartu-produk`?
3. Apa fungsi directive `@props(['produk'])`?
4. Apa beda `:nama="$var"` dan `nama="var"`?
5. Bagaimana cara memberi nilai default untuk props `ukuran`?
6. Kapan harus pakai `:tampilkan="false"` daripada `tampilkan="false"`?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. **Partial** dipakai untuk bagian yang muncul **sekali** per halaman (navbar, footer), lewat `@include`. **Component** dipakai untuk bagian yang muncul **berkali-kali** per halaman (kartu produk, tombol), lewat `<x-...>`. Component punya props yang lebih terstruktur.
2. Buat file `resources/views/components/kartu-produk.blade.php` (kebab-case) dan tulis `@props([...])` di atas, diikuti struktur Blade.
3. Mendeklarasikan bahwa component menerima props bernama `produk`. Setelah ini, variabel `$produk` otomatis tersedia di dalam component.
4. `:nama="$var"` mengevaluasi `$var` sebagai PHP (kirim nilai variabel). `nama="var"` mengirim string literal "var" (tidak dievaluasi).
5. Di `@props`: `@props(['ukuran' => 'sedang'])`. Kalau pemanggil tidak kirim props `ukuran`, otomatis pakai "sedang".
6. Saat kirim **boolean/angka/variabel PHP**. Tanpa `:`, "false" akan jadi string truthy, bukan boolean false.

</details>

---

## 18. Apakah Kamu Ingin Lanjut?

Di tahap 8 ini kamu sudah bisa membuat component reusable. Tapi component punya satu **kekuatan tersembunyi** yang belum dibahas: **slot**.

Langkah berikutnya:

> ### "Apakah kamu ingin lanjut ke langkah berikutnya: belajar Slot di dalam component?"
>
> Di tahap berikutnya kita akan:
>
> - memahami apa itu **slot** dan bedanya dengan props
> - membuat component kartu produk dengan **slot isi bebas**
> - memahami **default slot** dan **named slot**
> - kasus nyata: tombol `<x-tombol>` yang isinya bisa teks bebas atau ikon
> - contoh: kartu produk dengan slot untuk badge custom

Jawab: **"Ya, lanjut"** untuk ke tahap 9,
atau **"Ulangi tahap 8"** kalau ada bagian yang masih perlu diperdalam.

---

> 📚 **Daftar Tahap (12. Layout Blade Reusable):**
> - ✅ Tahap 1 — Apa itu layout Blade reusable
> - ✅ Tahap 2 — Membuat file layout utama Blade
> - ✅ Tahap 3 — `@extends` dan `@section` untuk memakai layout
> - ✅ Tahap 4 — Mengubah halaman daftar produk agar pakai layout
> - ✅ Tahap 5 — Mengubah halaman tambah, edit, detail produk
> - ✅ Tahap 6 — Mengubah halaman dashboard admin
> - ✅ Tahap 7 — Partial: memecah navbar dan footer
> - ✅ Tahap 8 — Component: membuat komponen kartu produk (kamu di sini)
> - ⏳ Tahap 9 — Slot: lubang yang bisa diisi apa saja
> - ⏳ Tahap 10 — Ringkasan dan best practice
