# Tahap 7 — Variant Tambahan dan Tipe Link

> Materi: 13. Komponen Button Blade
> Level: Pemula — Fondasi Laravel, MVC, CRUD
> Fokus tahap ini: **menyelesaikan masalah `href`** dan membuat komponen Button yang **otomatis jadi `<a>` atau `<button>`** sesuai kebutuhan.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Menjelaskan **kenapa** tombol kadang butuh jadi `<a>` dan kadang butuh jadi `<button>`.
2. Memodifikasi komponen `<x-button>` supaya **otomatis** jadi `<a>` kalau ada `href`.
3. Menambahkan **variant tambahan** seperti `info` dan `light`.
4. Membuat komponen `<x-button-link>` khusus untuk link.
5. Memahami perbedaan **tombol yang mengirim form** vs **tombol yang pindah halaman**.
6. Melakukan **refactor final** semua tombol dengan tipe yang benar.

Ini adalah tahap **penyempurnaan**. Setelah tahap ini, semua masalah dari tahap 5 dan 6 akan teratasi.

---

## 2. Analogi: Pintu Biasa vs Pintu Otomatis

Bayangkan kamu masuk ke **mall**. Ada dua jenis pintu:

### Pintu Biasa (Manual)

- Kamu **dorong** pintu untuk masuk.
- Pintu hanya terbuka kalau kamu **mendorong**.
- Cocok untuk: ruangan dengan sedikit orang.

### Pintu Otomatis (Sensor)

- Pintu **otomatis terbuka** saat kamu mendekat (sensor mendeteksi).
- Kamu tidak perlu dorong apa-apa.
- Cocok untuk: mall, rumah sakit, tempat ramai.

### Hubungannya dengan komponen Button

Saat ini, komponen `<x-button>` kita **selalu** menghasilkan tag `<button>`:

```blade
<x-button variant="secondary" href="/products">Batal</x-button>
```

Hasilnya:

```html
<button href="/products" class="btn btn-secondary">Batal</button>
```

Masalahnya: tag `<button>` **tidak bisa pindah halaman**. Atribut `href` diabaikan oleh browser. Tombolnya tampil, tapi **tidak berfungsi** sebagai link.

**Solusi:** bikin komponen yang **otomatis** menghasilkan `<a>` kalau ada `href`, seperti pintu otomatis yang mendeteksi kehadiran orang.

---

## 3. Review: Kapan Pakai `<a>` vs `<button>`?

Sebelum modifikasi komponen, mari kita **pastikan** pemahaman tentang kapan pakai `<a>` dan kapan pakai `<button>`.

### 3.1 Pakai `<a>` (Link) saat...

Tombol berfungsi untuk **pindah ke halaman lain** atau **navigasi**.

Contoh:

- Tombol **Tambah Produk** → pindah ke halaman form tambah.
- Tombol **Detail** → pindah ke halaman detail.
- Tombol **Edit** → pindah ke halaman form edit.
- Tombol **Batal** → pindah kembali ke halaman daftar.
- Tombol **Kembali** → pindah ke halaman sebelumnya.

Ciri-ciri:

- Tidak mengirim data ke server.
- Hanya **navigasi**.
- Bisa di-bookmark.
- Buka di tab baru dengan klik kanan.

### 3.2 Pakai `<button>` saat...

Tombol berfungsi untuk **mengirim form** atau **trigger aksi JavaScript**.

Contoh:

- Tombol **Simpan Produk** → mengirim data form ke server (POST).
- Tombol **Update Produk** → mengirim data form ke server (PUT).
- Tombol **Hapus Produk** → mengirim perintah hapus ke server (DELETE).
- Tombol **Toggle Modal** → membuka/menutup modal via JavaScript.

Ciri-ciri:

- Mengirim data atau trigger aksi.
- Berada di dalam `<form>`.
- Tidak bisa di-bookmark.
- Punya `type="submit"` atau `type="button"`.

### 3.3 Tabel Ringkas

| Tombol       | Tipe    | Tag HTML   | Kenapa                                  |
| ------------ | ------- | ---------- | --------------------------------------- |
| Tambah Produk | Link   | `<a>`      | Pindah ke halaman form tambah           |
| Detail       | Link    | `<a>`      | Pindah ke halaman detail                |
| Edit         | Link    | `<a>`      | Pindah ke halaman form edit             |
| Simpan       | Submit  | `<button>` | Kirim data ke server                    |
| Update       | Submit  | `<button>` | Kirim data ke server                    |
| Hapus        | Submit  | `<button>` | Kirim perintah hapus (di dalam form)    |
| Batal        | Link    | `<a>`      | Pindah kembali ke daftar                |
| Kembali      | Link    | `<a>`      | Pindah ke halaman sebelumnya            |

> **Pesan mentor:**
> Aturan praktis: **kalau pindah halaman → pakai `<a>`**. **Kalau kirim data/aksi → pakai `<button>`**. Jangan tertukar.

---

## 4. Masalah Saat Ini: Komponen Selalu Jadi `<button>`

Saat ini, komponen kita **selalu** menghasilkan `<button>`, walaupun ada `href`:

```blade
<!-- Komponen kita saat ini -->
@props(['variant' => 'primary'])

<button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
    {{ $slot }}
</button>
```

Jadi kalau kita tulis:

```blade
<x-button variant="secondary" href="/products">Batal</x-button>
```

Browser menerima:

```html
<button href="/products" class="btn btn-secondary">Batal</button>
```

**Masalah:**

1. `href` di tag `<button>` **tidak berfungsi** (browser abaikan).
2. Tombol tampil, tapi **tidak bisa diklik** untuk pindah halaman.
3. Pengguna klik "Batal", **tidak terjadi apa-apa**.

---

## 5. Solusi: Komponen yang Otomatis Jadi `<a>` atau `<button>`

Kita akan **modifikasi komponen** supaya **pintar**:

- Kalau ada `href` → hasilkan `<a href="...">`.
- Kalau tidak ada `href` → hasilkan `<button>`.

Caranya: pakai **kondisional `@if`** di Blade.

### 5.1 Kode komponen yang sudah dimodifikasi

```blade
@props([
    'variant' => 'primary',
    'href' => null,
])

@php
    $variantYangDiterima = ['primary', 'secondary', 'success', 'danger', 'warning', 'info', 'light'];

    if (!in_array($variant, $variantYangDiterima)) {
        throw new \InvalidArgumentException(
            "Variant '{$variant}' tidak dikenal. Variant yang valid: " .
            implode(', ', $variantYangDiterima)
        );
    }

    $ikonPerVariant = [
        'primary'   => '💾',
        'secondary' => '↩️',
        'success'   => '✅',
        'danger'    => '🗑️',
        'warning'   => '✏️',
        'info'      => '👁️',
        'light'     => '',
    ];

    $ikon = $ikonPerVariant[$variant] ?? '';
@endphp

@if ($href)
    <a href="{{ $href }}" {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
        @if ($ikon)<span class="me-1">{{ $ikon }}</span>@endif
        {{ $slot }}
    </a>
@else
    <button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
        @if ($ikon)<span class="me-1">{{ $ikon }}</span>@endif
        {{ $slot }}
    </button>
@endif
```

### 5.2 Penjelasan perubahan

#### Props baru: `'href' => null`

```blade
@props([
    'variant' => 'primary',
    'href' => null,    ← BARU
])
```

Sekarang komponen menerima props opsional `href`. Defaultnya `null` artinya "tidak ada href".

#### Kondisional `@if ($href)`

```blade
@if ($href)
    <a href="{{ $href }}" ...>
        ...
    </a>
@else
    <button ...>
        ...
    </button>
@endif
```

Logikanya:

- Kalau `$href` **ada** (tidak null) → hasilkan `<a>`.
- Kalau `$href` **tidak ada** (null) → hasilkan `<button>`.

#### Atribut `href` di `<a>`

```blade
<a href="{{ $href }}" {{ $attributes->merge([...]) }}>
```

`href` ditempatkan secara eksplisit di tag `<a>`, supaya browser mengenalinya sebagai link.

> **Catatan teknis:**
> Kenapa `href` ditulis eksplisit, bukan andalkan `$attributes`? Karena `$attributes` berisi **semua atribut** termasuk `href`, tapi di tag `<a>` kita perlu memastikan `href` tampil di posisi yang benar. Dengan menulis `href="{{ $href }}"` secara eksplisit, kita **pastikan** nilainya benar. Untuk atribut lain (class, id, onclick), tetap pakai `$attributes->merge(...)`.

#### Variant tambahan `info` dan `light`

Perhatikan array validasi sekarang **menyertakan** `info` dan `light`:

```blade
$variantYangDiterima = ['primary', 'secondary', 'success', 'danger', 'warning', 'info', 'light'];
```

- `info` → untuk tombol Detail (cyan).
- `light` → untuk tombol netral dengan latar putih (opsional).

---

## 6. Cara Kerja Komponen yang Baru

Sekarang mari kita lihat bagaimana komponen yang baru **bereaksi** terhadap pemanggilan berbeda.

### 6.1 Pemanggilan 1: Tanpa `href` (Button)

```blade
<x-button variant="primary" type="submit">
    Simpan Produk
</x-button>
```

**Yang terjadi:**

1. `$href` = `null` (tidak dikirim).
2. Kondisi `@if ($href)` → **false**.
3. Jalankan `@else` → hasilkan `<button>`.
4. Hasil HTML:

```html
<button type="submit" class="btn btn-primary">
    <span class="me-1">💾</span>
    Simpan Produk
</button>
```

### 6.2 Pemanggilan 2: Dengan `href` (Link)

```blade
<x-button variant="secondary" href="/products">
    Batal
</x-button>
```

**Yang terjadi:**

1. `$href` = `"/products"` (dikirim pemanggil).
2. Kondisi `@if ($href)` → **true**.
3. Hasilkan `<a>`.
4. Hasil HTML:

```html
<a href="/products" class="btn btn-secondary">
    <span class="me-1">↩️</span>
    Batal
</a>
```

Sekarang **Batal berfungsi** sebagai link. Pengguna klik, pindah ke `/products`. Masalah selesai!

### 6.3 Pemanggilan 3: Hapus (Button di dalam form)

```blade
<form action="/products/1" method="POST">
    @csrf
    @method('DELETE')
    <x-button variant="danger" type="submit"
              onclick="return confirm('Yakin?')">
        Hapus
    </x-button>
</form>
```

**Yang terjadi:**

1. `$href` = `null` (tidak dikirim, meskipun ada form action).
2. Kondisi `@if ($href)` → **false**.
3. Hasilkan `<button>`.
4. Atribut `type="submit"` dan `onclick="..."` masuk lewat `$attributes`.
5. Hasil HTML:

```html
<button type="submit"
        onclick="return confirm('Yakin?')"
        class="btn btn-danger">
    <span class="me-1">🗑️</span>
    Hapus
</button>
```

---

## 7. Diagram: Cara Kerja Komponen yang Otomatis

```
Pemanggil kirim <x-button ...>
         │
         ▼
    Ada href?
         │
    ┌────┴────┐
    │         │
   YA        TIDAK
    │         │
    ▼         ▼
 <a href>  <button>
    │         │
    └────┬────┘
         │
         ▼
   Output HTML
```

**Contoh konkret:**

| Pemanggil                                      | `$href` | Hasil   |
| ---------------------------------------------- | ------- | ------- |
| `<x-button variant="primary" type="submit">`   | `null`  | `<button>` |
| `<x-button variant="secondary" href="/products">` | `"/products"` | `<a>` |
| `<x-button variant="info" href="/products/1">` | `"/products/1"` | `<a>` |
| `<x-button variant="danger" type="submit">`    | `null`  | `<button>` |

---

## 8. Refactor Final: Semua Halaman dengan Tipe yang Benar

Sekarang komponen sudah **otomatis** memilih `<a>` atau `<button>`. Mari kita **refactor final** semua halaman dengan tipe yang **benar**.

### 8.1 Halaman `produk/index.blade.php` (Daftar Produk)

```blade
@extends('layout.app', ['title' => 'Daftar Produk'])

@section('konten')
    <h2>Daftar Produk</h2>

    <div class="mb-3">
        {{-- Tambah Produk: LINK (pindah ke form) --}}
        <x-button variant="primary" href="{{ route('produk.create') }}">
            Tambah Produk
        </x-button>
    </div>

    <table class="table">
        <thead>
            <tr>
                <th>Nama</th>
                <th>Harga</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
            @foreach ($produk as $item)
                <tr>
                    <td>{{ $item->nama }}</td>
                    <td>Rp {{ number_format($item->harga, 0, ',', '.') }}</td>
                    <td>
                        {{-- Detail: LINK --}}
                        <x-button variant="info" href="{{ route('produk.show', $item->id) }}">
                            Detail
                        </x-button>

                        {{-- Edit: LINK --}}
                        <x-button variant="warning" href="{{ route('produk.edit', $item->id) }}">
                            Edit
                        </x-button>

                        {{-- Hapus: BUTTON (pakai komponen khusus) --}}
                        <x-button-delete
                            action="{{ route('produk.destroy', $item->id) }}"
                            confirm="Yakin ingin menghapus {{ $item->nama }}?">
                            Hapus
                        </x-button-delete>
                    </td>
                </tr>
            @endforeach
        </tbody>
    </table>
@endsection
```

**Perhatikan:**

- Tambah Produk, Detail, Edit → semua pakai `href="..."` (jadi otomatis `<a>`).
- Hapus → pakai komponen `<x-button-delete>` (yang di dalamnya pakai `<x-button>` tanpa `href`, jadi `<button>`).

### 8.2 Halaman `produk/create.blade.php` (Tambah Produk)

```blade
@extends('layout.app', ['title' => 'Tambah Produk'])

@section('konten')
    <h2>Tambah Produk</h2>

    <form action="{{ route('produk.store') }}" method="POST">
        @csrf

        <div class="mb-3">
            <label>Nama Produk</label>
            <input type="text" name="nama" class="form-control" required>
        </div>

        <div class="mb-3">
            <label>Harga</label>
            <input type="number" name="harga" class="form-control" required>
        </div>

        {{-- Simpan: BUTTON (mengirim form) --}}
        <x-button variant="primary" type="submit">
            Simpan Produk
        </x-button>

        {{-- Batal: LINK (pindah ke daftar) --}}
        <x-button variant="secondary" href="{{ route('produk.index') }}">
            Batal
        </x-button>
    </form>
@endsection
```

**Perhatikan:**

- Simpan Produk → `type="submit"` **tanpa** `href` (jadi `<button>`).
- Batal → `href="..."` **tanpa** `type` (jadi `<a>`).

### 8.3 Halaman `produk/edit.blade.php` (Edit Produk)

```blade
@extends('layout.app', ['title' => 'Edit Produk'])

@section('konten')
    <h2>Edit Produk</h2>

    <form action="{{ route('produk.update', $produk->id) }}" method="POST">
        @csrf
        @method('PUT')

        <div class="mb-3">
            <label>Nama Produk</label>
            <input type="text" name="nama" class="form-control"
                   value="{{ $produk->nama }}" required>
        </div>

        <div class="mb-3">
            <label>Harga</label>
            <input type="number" name="harga" class="form-control"
                   value="{{ $produk->harga }}" required>
        </div>

        {{-- Update: BUTTON --}}
        <x-button variant="primary" type="submit">
            Update Produk
        </x-button>

        {{-- Batal: LINK --}}
        <x-button variant="secondary" href="{{ route('produk.index') }}">
            Batal
        </x-button>
    </form>
@endsection
```

### 8.4 Halaman `produk/show.blade.php` (Detail Produk)

```blade
@extends('layout.app', ['title' => 'Detail Produk'])

@section('konten')
    <h2>{{ $produk->nama }}</h2>

    <div class="card">
        <div class="card-body">
            <p>Harga: Rp {{ number_format($produk->harga, 0, ',', '.') }}</p>
            <p>Deskripsi: {{ $produk->deskripsi }}</p>
        </div>
    </div>

    <div class="mt-3">
        {{-- Edit: LINK --}}
        <x-button variant="warning" href="{{ route('produk.edit', $produk->id) }}">
            Edit Produk
        </x-button>

        {{-- Hapus: BUTTON (pakai komponen khusus) --}}
        <x-button-delete
            action="{{ route('produk.destroy', $produk->id) }}"
            confirm="Yakin ingin menghapus {{ $produk->nama }}?">
            Hapus Produk
        </x-button-delete>

        {{-- Kembali: LINK --}}
        <x-button variant="secondary" href="{{ route('produk.index') }}">
            Kembali
        </x-button>
    </div>
@endsection
```

---

## 9. Tabel Final: Semua Tombol dengan Tipe yang Benar

Setelah refactor final, ini adalah **daftar lengkap** semua tombol di CRUD Produk:

| Halaman      | Tombol         | Tipe      | Tag HTML   | Variant     |
| ------------ | -------------- | --------- | ---------- | ----------- |
| `index`      | Tambah Produk  | Link      | `<a>`      | `primary`   |
| `index`      | Detail         | Link      | `<a>`      | `info`      |
| `index`      | Edit           | Link      | `<a>`      | `warning`   |
| `index`      | Hapus          | Submit    | `<button>` | `danger`    |
| `create`     | Simpan Produk  | Submit    | `<button>` | `primary`   |
| `create`     | Batal          | Link      | `<a>`      | `secondary` |
| `edit`       | Update Produk  | Submit    | `<button>` | `primary`   |
| `edit`       | Batal          | Link      | `<a>`      | `secondary` |
| `show`       | Edit Produk    | Link      | `<a>`      | `warning`   |
| `show`       | Hapus Produk   | Submit    | `<button>` | `danger`    |
| `show`       | Kembali        | Link      | `<a>`      | `secondary` |

### Pola yang muncul

1. **Semua aksi navigasi** (pindah halaman) → `<a>` dengan `href`.
2. **Semua aksi submit** (kirim form) → `<button>` dengan `type="submit"`.
3. **Variant konsisten berdasarkan peran**, bukan berdasarkan halaman.

---

## 10. Variant Tambahan: `info` dan `light`

Di tahap 3 kita sudah bahas 5 variant utama + `info`. Sekarang kita tambah `light` sebagai variant opsional.

### 10.1 Variant `info` (Cyan)

| Hal            | Detail                            |
| -------------- | --------------------------------- |
| **Warna**      | Cyan / biru muda                  |
| **Makna**      | Aksi informatif                   |
| **Kapan dipakai** | Tombol yang **menampilkan info** |
| **Contoh**     | Detail Produk, Lihat Info         |

```blade
<x-button variant="info" href="...">
    Detail Produk
</x-button>
```

### 10.2 Variant `light` (Putih) - Opsional

| Hal            | Detail                            |
| -------------- | --------------------------------- |
| **Warna**      | Putih dengan teks gelap           |
| **Makna**      | Aksi sangat netral, hampir tak terlihat |
| **Kapan dipakai** | Di latar gelap, atau untuk aksi "sekunder" yang sangat tidak penting |
| **Contoh**     | Tutup, Batal di modal gelap       |

```blade
<x-button variant="light">
    Tutup
</x-button>
```

> **Pesan mentor:**
> `light` itu opsional dan **jarang dipakai**. Kalau ragu, pakai `secondary` saja yang lebih jelas terlihat.

### 10.3 Daftar variant lengkap

| Variant       | Warna       | Peran                        | Contoh             |
| ------------- | ----------- | ---------------------------- | ------------------ |
| `primary`     | Biru        | Aksi utama                   | Simpan, Tambah     |
| `secondary`   | Abu-abu     | Aksi netral                  | Batal, Kembali     |
| `success`     | Hijau       | Konfirmasi keberhasilan      | Selesai, Konfirmasi|
| `danger`      | Merah       | Aksi berbahaya               | Hapus              |
| `warning`     | Kuning      | Perlu perhatian              | Edit, Nonaktifkan  |
| `info`        | Cyan        | Aksi informatif              | Detail             |
| `light`       | Putih       | Netral di latar gelap        | Tutup (opsional)   |

---

## 11. Komponen `<x-button-link>` Alternatif (Opsional)

Kalau kamu mau **pemisahan eksplisit** antara tombol-link dan tombol-submit, kamu bisa bikin komponen terpisah `<x-button-link>`.

### 11.1 File `components/button-link.blade.php`

```blade
@props([
    'variant' => 'primary',
    'href' => null,
])

<a href="{{ $href }}" {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
    {{ $slot }}
</a>
```

### 11.2 Cara pakai

```blade
<x-button-link variant="secondary" href="/products">
    Batal
</x-button-link>
```

### 11.3 Kelebihan dan kekurangan

| Pendekatan | Kelebihan | Kekurangan |
| ---------- | --------- | ---------- |
| **Komponen otomatis** (`<x-button>` dengan deteksi `href`) | Satu komponen untuk semua, sintaks konsisten | Logika kondisional di komponen |
| **Komponen terpisah** (`<x-button>` + `<x-button-link>`) | Pemisahan jelas, tidak ada kondisional | Dua komponen untuk diingat |

> **Pesan mentor:**
> Untuk **pemula**, komponen otomatis (satu `<x-button>` yang pintar) sudah **cukup**. Komponen terpisah cocok kalau kamu mau **kontrol ketat** atas kapan pakai link vs button. Pilih sesuai selera, yang penting **konsisten**.

---

## 12. Verifikasi di Browser

Sekarang save semua file dan coba di browser.

### 12.1 Checklist verifikasi

| Halaman      | Yang Diversiifikasi                           |
| ------------ | --------------------------------------------- |
| `index`      | Tambah Produk **bisa diklik** → pindah ke form |
| `index`      | Detail **bisa diklik** → pindah ke halaman detail |
| `index`      | Edit **bisa diklik** → pindah ke form edit    |
| `index`      | Hapus **bisa diklik** → tampil konfirmasi → hapus |
| `create`     | Simpan **bisa diklik** → produk tersimpan     |
| `create`     | Batal **bisa diklik** → kembali ke daftar     |
| `edit`       | Update **bisa diklik** → produk terupdate     |
| `edit`       | Batal **bisa diklik** → kembali ke daftar     |
| `show`       | Edit **bisa diklik** → pindah ke form edit    |
| `show`       | Hapus **bisa diklik** → tampil konfirmasi     |
| `show`       | Kembali **bisa diklik** → kembali ke daftar   |

### 12.2 Cara test

1. Buka `http://localhost:8000/products`.
2. Klik **Tambah Produk** → harus pindah ke form tambah.
3. Isi form, klik **Simpan** → harus kembali ke daftar dengan produk baru.
4. Klik **Detail** produk → harus pindah ke halaman detail.
5. Klik **Edit** produk → harus pindah ke form edit.
6. Klik **Hapus** → harus tampil konfirmasi, klik OK → produk terhapus.
7. Klik **Batal** / **Kembali** → harus kembali ke daftar.

Kalau semua berfungsi, **selamat!** Semua tombol sekarang **konsisten** dan **berfungsi dengan benar**.

---

## 13. Troubleshooting

### Error 1: Tombol Link tidak bisa diklik

**Penyebab:**

- Komponen belum dimodifikasi untuk deteksi `href`.
- Masih pakai versi lama yang selalu hasilkan `<button>`.

**Solusi:**

Pastikan komponen `button.blade.php` sudah punya kondisional `@if ($href)` seperti di bagian 5.1.

### Error 2: Tampil error "href tidak dikenal"

**Penyebab:**

- Props `href` belum dideklarasikan di `@props([...])`.

**Solusi:**

Tambahkan `'href' => null` di deklarasi props:

```blade
@props([
    'variant' => 'primary',
    'href' => null,
])
```

### Error 3: Semua tombol jadi `<a>` padahal seharusnya `<button>`

**Penyebab:**

- Kamu salah mengirim `href` ke tombol submit.
- Atau kondisi `@if ($href)` salah logika.

**Solusi:**

Pastikan tombol submit **tidak** punya `href`:

```blade
<!-- Benar: tombol submit, tidak ada href -->
<x-button variant="primary" type="submit">Simpan</x-button>

<!-- Salah: tombol submit, tapi ada href -->
<x-button variant="primary" type="submit" href="...">Simpan</x-button>
```

### Error 4: Atribut `href` muncul dua kali di HTML

**Penyebab:**

- `href` ditulis eksplisit di `<a>`, tapi juga masuk ke `$attributes`.

**Solusi:**

Ini sebenarnya tidak masalah karena `$attributes->merge` akan menimpa duplikat. Tapi kalau mau bersih, bisa pakai `$attributes->except(['href'])`:

```blade
<a href="{{ $href }}" {{ $attributes->except(['href'])->merge(['class' => 'btn btn-' . $variant]) }}>
```

Tapi untuk pemula, tidak perlu ribet. Biarkan saja.

---

## 14. Latihan Mandiri

**Latihan F:**

Refactor **halaman dashboard admin** supaya semua tombol pakai `href` dengan benar.

**Kode lama (masalah):**

```blade
<a href="{{ route('produk.index') }}" class="btn btn-primary">
    Kelola Produk
</a>
<a href="{{ route('produk.create') }}" class="btn btn-success">
    Tambah Produk Baru
</a>
<a href="/logout" class="btn btn-danger">
    Logout
</a>
```

<details>
<summary><strong>Lihat jawaban Latihan F</strong></summary>

```blade
<x-button variant="primary" href="{{ route('produk.index') }}">
    Kelola Produk
</x-button>

<x-button variant="info" href="{{ route('produk.create') }}">
    Tambah Produk Baru
</x-button>

<x-button variant="danger" href="/logout">
    Logout
</x-button>
```

**Analisis:**

- Semua tombol di dashboard adalah **link** (pindah halaman), jadi semua pakai `href`.
- Kelola Produk → `primary` (aksi utama dashboard).
- Tambah Produk Baru → `info` (dashboard sudah punya primary, jadi tambah jadi info).
- Logout → `danger` (aksi yang mengakhiri sesi).

</details>

---

## 15. Istilah Kunci Tahap Ini

| Istilah              | Arti sederhana                                         |
| -------------------- | ------------------------------------------------------ |
| **Tipe link**        | Tombol yang berfungsi untuk navigasi (pakai `<a>`)     |
| **Tipe submit**      | Tombol yang berfungsi untuk mengirim form (pakai `<button>`) |
| **`href`**           | Atribut yang menentukan URL tujuan link                |
| **`type="submit"`**  | Atribut yang membuat tombol mengirim form              |
| **Komponen otomatis**| Komponen yang mendeteksi props dan menyesuaikan output |
| **Kondisional**      | Logika `@if ... @else` di Blade                        |

---

## 16. Ringkasan Tahap 7

1. **Tombol ada dua tipe:**
   - **Link** (`<a>`) → untuk navigasi/pindah halaman.
   - **Submit** (`<button>`) → untuk kirim form/trigger aksi.
2. **Komponen `<x-button>` dimodifikasi** supaya otomatis jadi `<a>` kalau ada `href`, dan `<button>` kalau tidak ada.
3. **Logika kondisional** `@if ($href)` di Blade menentukan tag mana yang dihasilkan.
4. **Variant tambahan**: `info` (cyan) untuk tombol Detail, `light` (putih) opsional.
5. **Refactor final** semua halaman CRUD Produk dengan tipe yang benar:
   - Navigasi → `<a href="...">`.
   - Submit form → `<button type="submit">`.
6. **Komponen alternatif** `<x-button-link>` bisa dibuat kalau mau pemisahan eksplisit (opsional).
7. **Semua masalah dari tahap 5 dan 6 sekarang teratasi.**

---

## 17. Cek Pemahaman

1. Kapan tombol sebaiknya jadi `<a>`, dan kapan jadi `<button>`?
2. Bagaimana cara membuat komponen `<x-button>` otomatis memilih `<a>` atau `<button>`?
3. Apa fungsi props `href` di komponen yang sudah dimodifikasi?
4. Variant apa yang tepat untuk tombol "Detail Produk"?
5. Kenapa tombol "Simpan Produk" tidak boleh punya `href`?
6. Apa keuntungan pakai komponen otomatis vs komponen terpisah (`<x-button-link>`)?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. **`<a>`** untuk navigasi/pindah halaman (tidak kirim data). **`<button>`** untuk mengirim form atau trigger aksi.
2. Dengan menambahkan props `href` dan kondisional `@if ($href)` di komponen: kalau ada `href`, hasilkan `<a>`; kalau tidak, hasilkan `<button>`.
3. Menentukan URL tujuan link. Kalau `href` dikirim, komponen otomatis jadi `<a href="...">`.
4. `variant="info"` (cyan), karena Detail adalah aksi informatif.
5. Karena Simpan adalah **submit form**, bukan navigasi. Kalau ada `href`, komponen akan jadi `<a>` dan form tidak akan terkirim.
6. **Komponen otomatis**: satu komponen, sintaks konsisten, lebih sedikit file. **Komponen terpisah**: pemisahan jelas, kontrol ketat. Untuk pemula, komponen otomatis sudah cukup.

</details>

---

## 18. Apakah Kamu Ingin Lanjut?

Di tahap 7 ini kamu sudah:

- Menyelesaikan masalah `href` yang belum berfungsi.
- Membuat komponen yang **otomatis** jadi `<a>` atau `<button>`.
- Menambahkan variant `info` dan `light`.
- Melakukan **refactor final** semua halaman dengan tipe yang benar.

Semua tombol sekarang **konsisten**, **berfungsi**, dan **mudah dirawat**.

Di tahap terakhir, kita akan **rangkum** semua yang sudah dipelajari dan membahas **best practice** untuk dikembangkan ke depan.

> ### Pertanyaan:
>
> Apakah kamu ingin lanjut ke langkah berikutnya: **Ringkasan dan Best Practice**?
>
> Di tahap 8 (terakhir) kita akan bahas:
>
> 1. Rangkuman semua konsep dari tahap 1-7
> 2. **Best practice** membuat komponen Blade
> 3. Checklist sebelum deploy
> 4. Kesalahan umum yang harus dihindari
> 5. Ide pengembangan komponen ke depan (size, outline, disabled)
> 6. Daftar lengkap semua file dan struktur akhir
>
> Ketik **"lanjut"** untuk ke tahap 8 (terakhir),
> atau tanyakan jika ada bagian tahap 7 yang masih bingung.

---

> **Daftar Tahap (13. Komponen Button Blade):**
> - [x] Tahap 1 — Kenapa Tombol Harus Konsisten
> - [x] Tahap 2 — Membuat File Komponen `<x-button>` Sederhana
> - [x] Tahap 3 — Memahami Props dan Variant
> - [x] Tahap 4 — Slot dan Isi Tombol
> - [x] Tahap 5 — Menerapkan `<x-button>` di Halaman Produk
> - [x] Tahap 6 — Tombol Hapus dengan Form dan Konfirmasi
> - [x] Tahap 7 — Variant Tambahan dan Tipe Link (kamu di sini)
> - [ ] Tahap 8 — Ringkasan dan Best Practice
