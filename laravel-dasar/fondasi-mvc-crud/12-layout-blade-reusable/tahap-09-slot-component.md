# Tahap 9 — Slot: Lubang yang Bisa Diisi Apa Saja

> Mentor: Laravel Dasar — Fondasi MVC, CRUD
> Topik: 12. Layout Blade Reusable
> Fokus tahap ini: **memahami slot** di component, kombinasi dengan props, dan kapan slot lebih cocok.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Menjelaskan apa itu **slot** dalam Blade component.
2. Menjelaskan perbedaan **slot vs props**.
3. Membuat component dengan **default slot**.
4. Membuat component dengan **named slot** (banyak slot).
5. Memutuskan kapan pakai **slot**, kapan pakai **props**.
6. Membuat component `<x-tombol>` dan `<x-kartu>` dengan slot.

Slot adalah senjata terakhir dalam template composition Blade. Setelah ini, kamu menguasai **empat alat Blade**: layout, partial, component, slot.

---

## 2. Analogi Sehari-Hari: Gelas Isi Bebas

Di tahap 8 kita pakai analogi **stempel bonus** untuk component + props. Stempel itu punya **ruang kosong** yang diisi nilai **spesifik** (tanggal, nama kasir).

Sekarang bayangkan **gelas**. Gelas itu **wadah** tetap. Tapi isinya **bebas**:

- Senin pagi: gelas diisi **kopi panas**.
- Siang: gelas diisi **es teh**.
- Malam: gelas diisi **air putih**.

Gelasnya **sama**, isinya **berubah-ubah, bebas apa saja**. Bahkan bisa diisi **benda** (koin, gula) bukan minuman.

Di Blade:

- **Gelas** = component dengan **slot**.
- **Isi** = apa pun yang kamu taruh **di antara** `<x-gelas>` dan `</x-gelas>`.

Bedanya dengan **stempel (props)**:

- Stempel: ruang kosong diisi **nilai** (teks tanggal, nama).
- Gelas (slot): ruang kosong diisi **konten apa pun** (teks, HTML, bahkan component lain).

> 📝 **Pesan mentor:**
> Inilah inti slot: **lubang yang isinya bebas**, bukan sekadar nilai primitif. Slot bisa diisi kode HTML kompleks, bahkan component lain di dalamnya.

---

## 3. Apa Itu Slot?

### Definisi sederhana

> **Slot** adalah **lubang** di dalam component yang diisi konten bebas dari luar, ditulis **di antara tag pembuka dan penutup** component.

### Sintaks dasar

**Di dalam component:**

```blade
{{ $slot }}
```

`$slot` adalah variabel **otomatis** di setiap component. Berisi konten yang ditaruh di antara `<x-nama>` dan `</x-nama>`.

**Di luar (pemanggilan):**

```blade
<x-tombol>
    Klik di sini          ← ini isi slot
</x-tombol>
```

Apa pun yang ditulis antara `<x-tombol>` dan `</x-tombol>` akan masuk ke `$slot` di dalam component.

### Analogi dalam HTML biasa

Mirip seperti `<button>Klik</button>` di HTML. Teks "Klik" adalah **isi** tombol. Kita bisa bayangkan `<button>` sebagai component dengan slot, dan teks di antaranya adalah isi slot.

---

## 4. Perbedaan Slot vs Props

Ini pertanyaan kunci tahap 9. Mari kita jelas pelan-pelan.

| Hal | Props | Slot |
|---|---|---|
| Cara kirim | lewat atribut: `<x-c :nama="$val" />` | lewat isi: `<x-c>KONTEN</x-c>` |
| Isi | nilai (string, angka, objek, boolean) | konten bebas (teks, HTML, component) |
| Deklarasi | `@props(['nama'])` | `{{ $slot }}` otomatis |
| Cocok untuk | **data** sederhana / objek | **tampilan** / struktur HTML |

### Contoh kasus: tombol

Misal kita mau bikin component tombol.

**Pakai props** (kurang cocok):

```blade
{{-- components/tombol.blade.php --}}
@props(['teks'])

<button class="btn">{{ $teks }}</button>
```

Pemakaian:

```blade
<x-tombol teks="Simpan" />
```

Ini works, **tapi** kalau kita mau tombol berisi **ikon + teks**:

```blade
<x-tombol teks="<i class='icon-save'></i> Simpan" />
{{-- ↑ nggak elegan, HTML di dalam string --}}
```

**Pakai slot** (lebih cocok):

```blade
{{-- components/tombol.blade.php --}}
@props([])

<button class="btn">
    {{ $slot }}
</button>
```

Pemakaian:

```blade
<x-tombol>
    <i class="icon-save"></i> Simpan
</x-tombol>

{{-- Atau cuma teks --}}
<x-tombol>Hapus</x-tombol>

{{-- Atau component lain di dalam --}}
<x-tombol>
    <x-ikon name="save" /> Simpan
</x-tombol>
```

Lihat? Dengan slot, isi tombol **bebas apa saja**: teks, HTML, ikon, bahkan component lain.

> 📝 **Pesan mentor:**
> Aturan praktis: kalau isinya **teks/angka/data** → props. Kalau isinya **HTML/struktur/komposisi elemen** → slot. Kalau ragu, tanya: "apakah isinya bisa berupa tag `<i>`, `<img>`, atau `<x-...>`?" Kalau ya → slot.

---

## 5. Langkah 1: Membuat Component `<x-tombol>` dengan Default Slot

### File `components/tombol.blade.php`

```blade
@props([
    'tipe' => 'primary',  // props biasa, tetap boleh digabung dengan slot
])

<button class="btn btn-{{ $tipe }}">
    {{ $slot }}
</button>
```

### Penjelasan

- `@props(['tipe' => 'primary'])` → component tetap terima props biasa (default "primary").
- `{{ $slot }}` → lubang yang akan diisi konten dari pemanggil.

> 📝 **Pesan mentor:**
> Component **boleh punya keduanya**: props untuk konfigurasi (tipe tombol), slot untuk isi (teks/ikon di dalamnya). Ini kombinasi umum.

### Pemakaian

```blade
<x-tombol>Simpan</x-tombol>

<x-tombol tipe="danger">Hapus</x-tombol>

<x-tombol tipe="success">
    <i class="icon-check"></i> Konfirmasi
</x-tombol>
```

Hasilnya:

```html
<button class="btn btn-primary">Simpan</button>

<button class="btn btn-danger">Hapus</button>

<button class="btn btn-success">
    <i class="icon-check"></i> Konfirmasi
</button>
```

Satu component, tiga variasi isi.

---

## 6. Langkah 2: Membuat Component `<x-kartu>` dengan Slot

Ini contoh lebih kompleks: kartu yang **kerangkanya tetap**, tapi **isi dalamnya bebas**.

### File `components/kartu.blade.php`

```blade
@props(['judul'])

<div class="kartu">
    <div class="kartu-header">
        <h3>{{ $judul }}</h3>
    </div>
    <div class="kartu-body">
        {{ $slot }}
    </div>
</div>
```

### Pemakaian

```blade
<x-kartu judul="Total Penjualan">
    <p>Total: <strong>Rp 5.000.000</strong></p>
    <p>Bulan: Juli 2026</p>
</x-kartu>

<x-kartu judul="Produk Terlaris">
    <ul>
        <li>Sepatu Lari — 100 terjual</li>
        <li>Tas Sekolah — 80 terjual</li>
    </ul>
</x-kartu>

<x-kartu judul="Grafik Pertumbuhan">
    <img src="/grafik.png" alt="Grafik">
    <p><a href="/laporan">Lihat detail laporan</a></p>
</x-kartu>
```

Satu component kartu, tiga isinya **sama sekali berbeda** (teks, list, gambar). Inilah kekuatan slot.

---

## 7. Default Slot vs Named Slot

Sampai sini, kita pakai **default slot** (`{{ $slot }}`). Tapi kadang satu component butuh **banyak lubang** dengan posisi berbeda.

Contoh: kartu dengan **header bebas** + **body bebas** + **footer bebas**.

### Named slot

Untuk bikin banyak slot, pakai directive `@slot('nama') ... @endslot` di pemanggilan.

### File `components/kartu-lengkap.blade.php`

```blade
@props([])

<div class="kartu">
    <div class="kartu-header">
        {{ $header }}              ← named slot "header"
    </div>
    <div class="kartu-body">
        {{ $slot }}                ← default slot (isi utama)
    </div>
    <div class="kartu-footer">
        {{ $footer }}              ← named slot "footer"
    </div>
</div>
```

> Catatan: di Laravel modern (10+), `$header` dan `$footer` sudah otomatis tersedia sebagai variabel. Untuk versi lebih lama, pakai `{{ $header ?? '' }}` untuk safety.

### Pemakaian

```blade
<x-kartu-lengkap>
    <x-slot:header>
        <h3>Total Penjualan</h3>
        <small>Juli 2026</small>
    </x-slot:header>

    <p>Isi utama kartu di sini.</p>
    <p>Bisa banyak paragraf.</p>
    <img src="/grafik.png" alt="grafik">

    <x-slot:footer>
        <a href="/laporan">Lihat detail</a>
    </x-slot:footer>
</x-kartu-lengkap>
```

### Penjelasan

**`<x-slot:header> ... </x-slot:header>`**

Menandai bahwa konten di dalamnya akan masuk ke named slot `header` (variabel `$header` di component).

**Konten di luar `<x-slot:...>`** (paragraf + gambar)

Masuk ke **default slot** (`$slot`).

> 📝 **Pesan mentor:**
> Default slot cocok kalau isinya **satu bagian**. Named slot cocok kalau isinya **tersebar di banyak area** (header, body, footer terpisah).

---

## 8. Diagram Slot vs Props

```
PROPS (data masuk lewat atribut)
─────────────────────────────────
Pemanggilan:
  <x-tombol teks="Simpan" />

Di component:
  @props(['teks'])
  <button>{{ $teks }}</button>
        ↑ variabel dari props


SLOT (konten masuk lewat isi tag)
─────────────────────────────────
Pemanggilan:
  <x-tombol>
      Simpan           ← ini isi slot
  </x-tombol>

Di component:
  <button>{{ $slot }}</button>
              ↑ variabel otomatis dari isi tag
```

---

## 9. Diagram Named Slot dalam Component Kartu

```
Pemanggilan:
<x-kartu-lengkap>
    <x-slot:header>              ← named slot "header"
        <h3>Total</h3>
    </x-slot:header>

    <p>Isi utama...</p>          ← default slot ($slot)
    <img src="...">

    <x-slot:footer>              ← named slot "footer"
        <a href="#">Detail</a>
    </x-slot:footer>
</x-kartu-lengkap>

            │
            ▼

Component kartu-lengkap.blade.php:
┌─────────────────────────────┐
│  {{ $header }}              │ ← diisi dari <x-slot:header>
├─────────────────────────────┤
│                             │
│  {{ $slot }}                │ ← diisi dari isi tag (default slot)
│    <p>Isi utama...</p>      │
│    <img src="...">          │
│                             │
├─────────────────────────────┤
│  {{ $footer }}              │ ← diisi dari <x-slot:footer>
└─────────────────────────────┘
```

---

## 10. Kombinasi: Kartu Produk dengan Props + Slot

Sekarang gabungkan tahap 8 (props) dan tahap 9 (slot) ke satu component kartu produk yang **fleksibel**.

### File `components/kartu-produk-slot.blade.php`

```blade
@props(['produk'])

<div class="kartu-produk">
    <img src="{{ asset('storage/' . $produk->gambar) }}" alt="{{ $produk->nama }}">

    <h3>{{ $produk->nama }}</h3>
    <p>Rp {{ number_format($produk->harga, 0, ',', '.') }}</p>

    {{-- Slot opsional untuk badge diskon / label khusus --}}
    @isset($slot)
        <div class="badge-area">
            {{ $slot }}
        </div>
    @endisset

    <a href="{{ route('produk.show', $produk->id) }}">Lihat Detail</a>
</div>
```

### Penjelasan

- `@props(['produk'])` → kartu tetap butuh data produk.
- `@isset($slot)` → hanya tampilkan blok badge **kalau pemanggil mengisi slot**. Kalau tidak diisi, blok itu hilang.

### Pemakaian

```blade
{{-- Tanpa slot: kartu sederhana --}}
<x-kartu-produk-slot :produk="$item" />

{{-- Dengan slot: ada badge "Promo 20%" --}}
<x-kartu-produk-slot :produk="$item">
    <span class="badge-promo">Promo 20%</span>
</x-kartu-produk-slot>

{{-- Slot kompleks: badge + hitung hemat --}}
<x-kartu-produk-slot :produk="$item">
    <span class="badge-promo">Promo 20%</span>
    <small>Hemat Rp {{ number_format($item->harga * 0.2) }}</small>
</x-kartu-produk-slot>
```

Satu component, fleksibel: bisa tanpa badge, bisa badge sederhana, bisa badge kompleks. Inilah kombinasi **props (data tetap) + slot (konten bebas)**.

---

## 11. Kapan Pakai Slot?

| Skenario | Pakai slot? |
|---|---|
| Tombol dengan isi bebas (teks, ikon, keduanya) | ✅ Ya |
| Kartu dengan body bebas (paragraf, list, gambar) | ✅ Ya |
| Layout halaman (header + konten + footer) | ✅ Ya, walau `@yield` lebih cocok |
| Data sederhana (nama, harga, angka) | ❌ Tidak, pakai props |
| Object/Model (Produk, User) | ❌ Tidak, pakai props |
| Boolean flag (tampilkan/sembunyikan) | ❌ Tidak, pakai props |
| Komposisi HTML kompleks di dalam component | ✅ Ya |

### Aturan praktis cepat

> **"Apakah isinya bisa berupa tag `<i>`, `<img>`, atau `<x-...>`?"**
> - **Ya** → pakai **slot**.
> - **Tidak (cuma teks/angka)** → pakai **props**.

---

## 12. Hubungan dengan Layout (`@yield`)

Di tahap 2, kamu sudah kenal `@yield('konten')` di layout. Sebenarnya `@yield` **mirip slot**, tapi untuk **layout**, bukan component.

| Hal | `@yield` (layout) | `$slot` (component) |
|---|---|---|
| Tempat | layout (file `layout/app.blade.php`) | component (file `components/x.blade.php`) |
| Cara pakai | `@section('konten') ... @endsection` | `<x-nama> ISI </x-nama>` |
| Named variant | `@yield('sidebar')` + `@section('sidebar')` | `<x-slot:header>` di dalam `<x-nama>` |
| Konsep | sama: lubang diisi dari luar | sama: lubang diisi dari luar |

Intinya sama: **membuat lubang yang diisi konten dari luar**. Hanya beda konteks (layout vs component) dan beda sintaks (`@yield` vs `$slot`).

---

## 13. Rangkuman Empat Alat Blade

Sekarang kamu sudah mengenal **empat alat** template composition Blade:

| # | Alat | Cara pakai | Untuk |
|---|---|---|---|
| 1 | **Layout** | `@extends` + `@yield` | kerangka halaman |
| 2 | **Partial** | `@include` | bagian sekali pakai (navbar) |
| 3 | **Component** | `<x-...>` + props | bagian berulang dengan data (kartu) |
| 4 | **Slot** | `<x-...> ISI </x-...>` | isi bebas di dalam component |

Empat alat ini **bekerja sama** dalam proyek Laravel nyata:

```
Halaman (view) ─┬─ @extends ──> Layout
                ├─ @include ──> Partial (navbar, footer)
                └─ <x-...> ───> Component
                                    └─ $slot ──> Isi bebas
```

---

## 14. Troubleshooting

### Error 1: `$slot` undefined di component

**Penyebab:**
- Versi Laravel terlalu lama (slot anonymous component hadir Laravel 7+).
- Atau salah ketik `$slots` (jamak), harusnya `$slot`.

**Solusi:** Pastikan Laravel 7+ dan tulis `{{ $slot }}` (tunggal).

### Error 2: Named slot tidak muncul

**Penyebab:**
- Pemanggilan pakai `@slot('header')` (directive) padahal seharusnya `<x-slot:header>` (tag).
- Atau nama slot tidak cocok.

**Solusi:** Untuk **anonymous component**, pakai sintaks tag `<x-slot:header> ... </x-slot:header>`. Cek nama slot cocok dengan variabel di component (`$header`).

### Error 3: Default slot tampil dua kali

**Penyebab:** Pemanggil menulis konten di luar `<x-slot:...>`, dan **juga** ada konten di default slot. Sebenarnya ini wajar — default slot terisi dari konten di luar named slot.

**Solusi:** Pahami: konten di **luar** `<x-slot:...>` masuk ke **default slot**. Jangan tulis dua kali.

### Error 4: Slot kosong tampil sebagai blok kosong

**Penyebab:** Pemanggil tidak mengisi slot, tapi component tetap merender `<div class="badge-area">` kosong.

**Solusi:** Pakai `@isset($slot) ... @endisset` di component untuk menyembunyikan blok kalau slot tidak diisi.

---

## 15. Latihan Mandiri

**Latihan G:**

Buat component `<x-pesan>` (alert/notifikasi) dengan:

- Props `tipe` (default "info", nilai lain: "success", "danger", "warning").
- Slot untuk isi pesan bebas.

Contoh pemakaian:

```blade
<x-pesan tipe="success">
    <strong>Berhasil!</strong> Produk telah ditambahkan.
</x-pesan>
```

<details>
<summary><strong>Lihat jawaban Latihan G</strong></summary>

**File `components/pesan.blade.php`:**

```blade
@props(['tipe' => 'info'])

<div class="alert alert-{{ $tipe }}" style="padding:10px; border:1px solid #ccc;">
    {{ $slot }}
</div>
```

**Pemakaian:**

```blade
<x-pesan tipe="success">
    <strong>Berhasil!</strong> Produk telah ditambahkan.
</x-pesan>

<x-pesan tipe="danger">
    <strong>Gagal!</strong> Harga harus berupa angka.
</x-pesan>

<x-pesan>
    Info: Klik tombol di atas untuk menambah produk.
</x-pesan>
```

</details>

---

## 16. Istilah Kunci Tahap Ini

| Istilah | Arti sederhana |
|---|---|
| **Slot** | lubang di component, diisi konten bebas dari luar |
| **Default slot** | lubang utama `$slot`, diisi dari isi tag component |
| **Named slot** | lubang bernama (mis: `$header`), diisi via `<x-slot:nama>` |
| **`{{ $slot }}`** | variabel otomatis untuk default slot di component |
| **`<x-slot:nama>`** | tag untuk mengisi named slot di pemanggilan |

---

## 17. Rangkuman Tahap 9

1. **Slot** = lubang di component yang isinya **bebas apa pun** (teks, HTML, component lain).
2. Default slot pakai variabel `{{ $slot }}`, diisi dari konten **di antara tag** `<x-nama> ISI </x-nama>`.
3. Named slot pakai variabel seperti `{{ $header }}`, diisi via `<x-slot:header> ... </x-slot:header>`.
4. **Props** untuk **data sederhana** (teks, objek). **Slot** untuk **konten bebas / HTML kompleks**.
5. Component bisa **kombinasi** props + slot: props untuk data, slot untuk isi fleksibel.
6. `@yield` di layout **mirip** slot di component — sama-sama lubang yang diisi dari luar.
7. Empat alat Blade sudah lengkap: **layout, partial, component, slot**.

---

## 18. Cek Pemahaman

1. Apa perbedaan **props** dan **slot**?
2. Bagaimana cara mendefinisikan default slot di dalam component?
3. Bagaimana cara mengisi default slot dari pemanggilan?
4. Bagaimana cara membuat dan mengisi **named slot**?
5. Kapan harus pakai slot, bukan props? Berikan satu contoh.
6. Apa hubungan `@yield` (di layout) dengan `$slot` (di component)?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. **Props** menerima **data sederhana** (string, angka, objek) lewat atribut: `:nama="$var"`. **Slot** menerima **konten bebas** (teks, HTML, component) lewat isi tag antara pembuka dan penutup.
2. Cukup tulis `{{ $slot }}` di posisi yang diinginkan. `$slot` adalah variabel otomatis yang sudah ada di setiap component.
3. Tulis konten di antara tag pembuka dan penutup component: `<x-tombol>KLIK</x-tombol>`. Teks "KLIK" jadi isi `$slot`.
4. Definisi: di component tulis `{{ $header }}`. Pemanggilan: `<x-slot:header> ISI </x-slot:header>` di dalam tag component.
5. Saat isi component **bukan data sederhana**, tapi **struktur HTML**. Contoh: tombol berisi ikon + teks `<x-tombol><i class="icon"></i> Simpan</x-tombol>`. Tidak bisa pakai props karena isinya HTML, bukan teks.
6. **Konsepnya sama**: keduanya lubang yang diisi konten dari luar. Beda **konteks**: `@yield` untuk layout (`@section` pengisi), `$slot` untuk component (isi tag pengisi). Beda **sintaks**, tapi ide dasar sama.

</details>

---

## 19. Apakah Kamu Ingin Lanjut?

Di tahap 9 ini kamu sudah **menguasai empat alat Blade**: layout, partial, component, slot. Semua konsep template composition sudah lengkap.

Langkah terakhir:

> ### "Apakah kamu ingin lanjut ke langkah terakhir: ringkasan dan best practice?"
>
> Di tahap terakhir kita akan:
>
> - menyatukan **semua konsep** (layout + partial + component + slot)
> - tabel ringkasan kapan pakai masing-masing
> - **best practice** Blade untuk proyek nyata
> - **kesalahan umum** yang harus dihindari
> - **checklist refactor** untuk menilai apakah Blade sudah rapi
> - penutup dan saran belajar lanjutan

Jawab: **"Ya, lanjut"** untuk ke tahap 10 (terakhir),
atau **"Ulangi tahap 9"** kalau ada bagian yang masih perlu diperdalam.

---

> 📚 **Daftar Tahap (12. Layout Blade Reusable):**
> - ✅ Tahap 1 — Apa itu layout Blade reusable
> - ✅ Tahap 2 — Membuat file layout utama Blade
> - ✅ Tahap 3 — `@extends` dan `@section` untuk memakai layout
> - ✅ Tahap 4 — Mengubah halaman daftar produk agar pakai layout
> - ✅ Tahap 5 — Mengubah halaman tambah, edit, detail produk
> - ✅ Tahap 6 — Mengubah halaman dashboard admin
> - ✅ Tahap 7 — Partial: memecah navbar dan footer
> - ✅ Tahap 8 — Component: membuat komponen kartu produk
> - ✅ Tahap 9 — Slot: lubang yang bisa diisi apa saja (kamu di sini)
> - ⏳ Tahap 10 — Ringkasan dan best practice
