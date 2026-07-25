# Tahap 5 — Menerapkan `<x-button>` di Semua Halaman Produk

> Materi: 13. Komponen Button Blade
> Level: Pemula — Fondasi Laravel, MVC, CRUD
> Fokus tahap ini: **merefactor semua halaman CRUD Produk** supaya pakai komponen `<x-button>` secara konsisten.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Mengganti semua tombol manual di **4 halaman CRUD Produk** dengan `<x-button>`.
2. Memilih variant yang tepat untuk setiap tombol berdasarkan perannya.
3. Menjelaskan manfaat konsistensi setelah refactor selesai.
4. Membandingkan kode **sebelum** dan **sesudah** refactor.
5. Memverifikasi bahwa semua tombol tampil konsisten di browser.

Ini adalah tahap **praktik terbanyak** di seluruh materi. Kita akan ubah **4 halaman sekaligus**. Siapkan kopimu.

---

## 2. Analogi: Renovasi Rumah Setapak

Bayangkan kamu punya **rumah lama** yang setiap kamarnya pakai **pintu beda-beda**:

- Pintu kamar tidur: pintu kayu cokelat dengan handle kuningan.
- Pintu kamar mandi: pintu putih dengan handle plastik.
- Pintu dapur: pintu geser kaca.
- Pintu belakang: pintu besi.

Setiap kali ada yang rusak, kamu harus **cari pengrajin berbeda** untuk masing-masing pintu.

**Renovasi:** kamu ganti semua pintu dengan **pintu standar** yang sama:

- Semua pakai pintu kayu warna putih.
- Semua pakai handle krom.
- Hanya **warna catnya** yang beda (kayu natural untuk kamar, putih untuk kamar mandi, dll).

Sekarang:

- Tampilan jadi **seragam** dan **rapi**.
- Kalau handle rusak, kamu cukup beli **satu jenis handle** untuk semua pintu.
- Kalau mau ganti warna, cukup cat ulang (tidak ganti pintunya).

### Hubungannya dengan refactor tombol

- **Pintu lama** = tombol manual dengan class CSS acak di setiap halaman.
- **Pintu baru** = komponen `<x-button>` yang seragam.
- **Warna cat** = variant (primary, danger, warning, dll).
- **Handle krom** = class Bootstrap `btn` yang sama di semua tombol.

Di tahap ini, kita **renovasi** semua tombol di 4 halaman CRUD Produk supaya pakai **pintu standar** yang sama.

---

## 3. Daftar Halaman yang Akan Direfactor

Kita akan ubah **4 halaman** sekaligus:

| Halaman                  | File                           | Tombol yang Ada                              |
| ------------------------ | ------------------------------ | -------------------------------------------- |
| Daftar Produk            | `produk/index.blade.php`       | Tambah Produk, Edit, Hapus, Detail           |
| Tambah Produk            | `produk/create.blade.php`      | Simpan Produk, Batal                         |
| Edit Produk              | `produk/edit.blade.php`        | Update Produk, Batal                         |
| Detail Produk            | `produk/show.blade.php`        | Edit Produk, Hapus Produk, Kembali           |

Total: **sekitar 10-12 tombol** akan diubah.

> **Pesan mentor:**
> Sebelum mulai, **backing dulu** semua file ini (copy ke folder backup atau commit ke git). Kalau ada yang salah, kamu bisa kembali ke versi lama dengan mudah.

---

## 4. Komponen Button Final (Versi yang Kita Pakai)

Sebelum mulai refactor, pastikan komponen `button.blade.php` sudah berisi kode lengkap dari tahap 3 dan 4:

### File: `resources/views/components/button.blade.php`

```blade
@props(['variant' => 'primary'])

@php
    $variantYangDiterima = ['primary', 'secondary', 'success', 'danger', 'warning', 'info'];

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
    ];

    $ikon = $ikonPerVariant[$variant] ?? '';
@endphp

<button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
    @if ($ikon)
        <span class="me-1">{{ $ikon }}</span>
    @endif
    {{ $slot }}
</button>
```

> Kalau komponenmu belum sekompleks ini, tidak masalah. Versi sederhana dari tahap 2 juga cukup. Tapi versi di atas akan menghasilkan tombol **lebih kaya** dengan ikon otomatis.

---

## 5. Refactor Halaman 1: `produk/index.blade.php` (Daftar Produk)

### 5.1 Kode lama (sebelum refactor)

```blade
@extends('layout.app', ['title' => 'Daftar Produk'])

@section('konten')
    <h2>Daftar Produk</h2>

    <a href="{{ route('produk.create') }}" class="btn btn-primary mb-3">
        + Tambah Produk
    </a>

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
                        <a href="{{ route('produk.show', $item->id) }}"
                           class="btn btn-info btn-sm">
                            Detail
                        </a>
                        <a href="{{ route('produk.edit', $item->id) }}"
                           class="btn btn-warning btn-sm">
                            Edit
                        </a>
                        <form action="{{ route('produk.destroy', $item->id) }}"
                              method="POST"
                              style="display:inline">
                            @csrf
                            @method('DELETE')
                            <button type="submit"
                                    class="btn btn-danger btn-sm"
                                    onclick="return confirm('Hapus produk ini?')">
                                Hapus
                            </button>
                        </form>
                    </td>
                </tr>
            @endforeach
        </tbody>
    </table>
@endsection
```

### 5.2 Analisis tombol yang ada

Di halaman ini ada **4 jenis tombol**:

| Tombol          | Variant yang Tepat | Alasan                               |
| --------------- | ------------------ | ------------------------------------ |
| Tambah Produk   | `primary`          | Aksi utama di halaman daftar         |
| Detail          | `info`             | Aksi informatif                      |
| Edit            | `warning`          | Aksi mengubah                        |
| Hapus           | `danger`           | Aksi berbahaya (menghapus data)      |

### 5.3 Kode baru (sesudah refactor)

```blade
@extends('layout.app', ['title' => 'Daftar Produk'])

@section('konten')
    <h2>Daftar Produk</h2>

    <div class="mb-3">
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
                        <x-button variant="info" href="{{ route('produk.show', $item->id) }}">
                            Detail
                        </x-button>

                        <x-button variant="warning" href="{{ route('produk.edit', $item->id) }}">
                            Edit
                        </x-button>

                        <form action="{{ route('produk.destroy', $item->id) }}"
                              method="POST"
                              style="display:inline">
                            @csrf
                            @method('DELETE')
                            <x-button variant="danger" type="submit"
                                      onclick="return confirm('Hapus produk ini?')">
                                Hapus
                            </x-button>
                        </form>
                    </td>
                </tr>
            @endforeach
        </tbody>
    </table>
@endsection
```

### 5.4 Perhatikan perubahan pentingnya

#### Tombol Tambah Produk

**Sebelum:**

```blade
<a href="..." class="btn btn-primary mb-3">+ Tambah Produk</a>
```

**Sesudah:**

```blade
<x-button variant="primary" href="{{ route('produk.create') }}">
    Tambah Produk
</x-button>
```

> Tunggu, kenapa kita kirim `href` padahal komponen kita pakai `<button>`, bukan `<a>`?
>
> Pertanyaan bagus! Untuk sekarang, `href` akan jadi atribut tambahan di `<button>`, yang **tidak akan membuat tombol jadi link**. Kita akan bahas solusinya di **tahap 7** (Tipe Link). Untuk sekarang, kita fokus dulu ke **struktur komponen** dan **variant**.

#### Tombol Hapus (di dalam form)

**Sebelum:**

```blade
<button type="submit" class="btn btn-danger btn-sm"
        onclick="return confirm('Hapus produk ini?')">
    Hapus
</button>
```

**Sesudah:**

```blade
<x-button variant="danger" type="submit"
          onclick="return confirm('Hapus produk ini?')">
    Hapus
</x-button>
```

Perhatikan: `type="submit"` dan `onclick="..."` dikirim sebagai **atribut tambahan**. Komponen akan otomatis menyertakan atribut ini di tag `<button>` berkat `$attributes->merge(...)`.

---

## 6. Refactor Halaman 2: `produk/create.blade.php` (Tambah Produk)

### 6.1 Kode lama

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

        <button type="submit" class="btn btn-primary">
            Simpan Produk
        </button>

        <a href="{{ route('produk.index') }}" class="btn btn-light">
            Batal
        </a>
    </form>
@endsection
```

### 6.2 Analisis tombol yang ada

| Tombol         | Variant yang Tepat | Alasan                           |
| -------------- | ------------------ | -------------------------------- |
| Simpan Produk  | `primary`          | Aksi utama di halaman tambah     |
| Batal          | `secondary`        | Aksi netral, kembali ke daftar   |

### 6.3 Kode baru

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

        <x-button type="submit" variant="primary">
            Simpan Produk
        </x-button>

        <x-button variant="secondary" href="{{ route('produk.index') }}">
            Batal
        </x-button>
    </form>
@endsection
```

### 6.4 Yang berubah

- Tombol **Simpan Produk**: dari `<button class="btn btn-primary">` → `<x-button variant="primary" type="submit">`.
- Tombol **Batal**: dari `<a class="btn btn-light">` → `<x-button variant="secondary" href="...">`.
- Sekarang tombol Simpan dan Batal **sudah konsisten**, sama-sama pakai komponen `<x-button>`.

---

## 7. Refactor Halaman 3: `produk/edit.blade.php` (Edit Produk)

### 7.1 Kode lama

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

        <button type="submit" class="btn btn-success px-4">
            Update Produk
        </button>

        <a href="{{ route('produk.index') }}" class="btn btn-secondary">
            Batal
        </a>
    </form>
@endsection
```

### 7.2 Analisis tombol yang ada

| Tombol          | Variant yang Tepat | Alasan                                    |
| --------------- | ------------------ | ----------------------------------------- |
| Update Produk   | `primary`          | Aksi utama di halaman edit                |
| Batal           | `secondary`        | Aksi netral                               |

> **Catatan penting:**
> Di kode lama, tombol Update pakai `btn btn-success` (hijau). Tapi **variant yang tepat** untuk Update adalah `primary` (biru), karena Update adalah **aksi utama** di halaman edit, bukan konfirmasi keberhasilan. Ini adalah **perbaikan kualitas** yang kita lakukan saat refactor.

### 7.3 Kode baru

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

        <x-button type="submit" variant="primary">
            Update Produk
        </x-button>

        <x-button variant="secondary" href="{{ route('produk.index') }}">
            Batal
        </x-button>
    </form>
@endsection
```

### 7.4 Perhatikan konsistensinya

Sekarang bandingkan halaman `create.blade.php` dan `edit.blade.php`:

| Tombol        | Halaman Create             | Halaman Edit                |
| ------------- | -------------------------- | --------------------------- |
| Aksi utama    | `<x-button variant="primary" type="submit">Simpan</x-button>` | `<x-button variant="primary" type="submit">Update</x-button>` |
| Batal         | `<x-button variant="secondary" href="...">Batal</x-button>`   | `<x-button variant="secondary" href="...">Batal</x-button>`   |

**Polanya sama persis!** Inilah manfaat konsistensi: sekali kamu paham pola di satu halaman, kamu langsung paham pola di halaman lain.

---

## 8. Refactor Halaman 4: `produk/show.blade.php` (Detail Produk)

### 8.1 Kode lama

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
        <a href="{{ route('produk.edit', $produk->id) }}" class="btn btn-info">
            Edit Produk
        </a>

        <form action="{{ route('produk.destroy', $produk->id) }}"
              method="POST"
              style="display:inline">
            @csrf
            @method('DELETE')
            <button type="submit" class="btn btn-danger"
                    onclick="return confirm('Hapus produk ini?')">
                Hapus Produk
            </button>
        </form>

        <a href="{{ route('produk.index') }}" class="btn btn-outline-secondary">
            Kembali
        </a>
    </div>
@endsection
```

### 8.2 Analisis tombol yang ada

| Tombol          | Variant yang Tepat | Alasan                              |
| --------------- | ------------------ | ----------------------------------- |
| Edit Produk     | `warning`          | Aksi mengubah                       |
| Hapus Produk    | `danger`           | Aksi berbahaya                      |
| Kembali         | `secondary`        | Aksi netral, kembali ke daftar      |

### 8.3 Kode baru

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
        <x-button variant="warning" href="{{ route('produk.edit', $produk->id) }}">
            Edit Produk
        </x-button>

        <form action="{{ route('produk.destroy', $produk->id) }}"
              method="POST"
              style="display:inline">
            @csrf
            @method('DELETE')
            <x-button variant="danger" type="submit"
                      onclick="return confirm('Hapus produk ini?')">
                Hapus Produk
            </x-button>
        </form>

        <x-button variant="secondary" href="{{ route('produk.index') }}">
            Kembali
        </x-button>
    </div>
@endsection
```

### 8.4 Yang berubah

- **Edit Produk**: dari `btn btn-info` → `variant="warning"` (info diganti warning karena Edit = mengubah).
- **Hapus Produk**: dari `<button class="btn btn-danger">` → `<x-button variant="danger" type="submit">`.
- **Kembali**: dari `<a class="btn btn-outline-secondary">` → `<x-button variant="secondary" href="...">`.

---

## 9. Tabel Konsistensi: Semua Tombol Setelah Refactor

Sekarang semua halaman sudah direfactor. Mari kita lihat **tabel konsistensi** lengkap:

| Halaman         | Tombol          | Kode                                            |
| --------------- | --------------- | ----------------------------------------------- |
| `index.blade`   | Tambah Produk   | `<x-button variant="primary">`                  |
| `index.blade`   | Detail          | `<x-button variant="info">`                     |
| `index.blade`   | Edit            | `<x-button variant="warning">`                  |
| `index.blade`   | Hapus           | `<x-button variant="danger" type="submit">`     |
| `create.blade`  | Simpan Produk   | `<x-button variant="primary" type="submit">`    |
| `create.blade`  | Batal           | `<x-button variant="secondary">`                |
| `edit.blade`    | Update Produk   | `<x-button variant="primary" type="submit">`    |
| `edit.blade`    | Batal           | `<x-button variant="secondary">`                |
| `show.blade`    | Edit Produk     | `<x-button variant="warning">`                  |
| `show.blade`    | Hapus Produk    | `<x-button variant="danger" type="submit">`     |
| `show.blade`    | Kembali         | `<x-button variant="secondary">`                |

### Pola yang muncul

Setelah refactor, pola **sangat jelas**:

1. **Aksi utama** (Simpan, Update, Tambah) → selalu `variant="primary"`.
2. **Aksi mengubah** (Edit) → selalu `variant="warning"`.
3. **Aksi menghapus** (Hapus) → selalu `variant="danger"`.
4. **Aksi informatif** (Detail) → selalu `variant="info"`.
5. **Aksi netral** (Batal, Kembali) → selalu `variant="secondary"`.

Tidak ada lagi tombol dengan class acak. Semua konsisten.

---

## 10. Coba di Browser

Sekarang save semua file yang sudah direfactor, lalu cek di browser.

### 10.1 Checklist verifikasi

| Halaman                  | URL                                          | Yang Harus Tampil                              |
| ------------------------ | -------------------------------------------- | ---------------------------------------------- |
| Daftar Produk            | `http://localhost:8000/products`             | Tombol Tambah (biru), Detail (cyan), Edit (kuning), Hapus (merah) |
| Tambah Produk            | `http://localhost:8000/products/create`      | Tombol Simpan (biru), Batal (abu-abu)          |
| Edit Produk              | `http://localhost:8000/products/1/edit`      | Tombol Update (biru), Batal (abu-abu)          |
| Detail Produk            | `http://localhost:8000/products/1`           | Tombol Edit (kuning), Hapus (merah), Kembali (abu-abu) |

### 10.2 Apa yang harusnya terlihat?

Coba buka **semua 4 halaman** secara berurutan. Amati:

1. **Semua tombol punya ukuran dan padding yang sama** (berkat class `btn`).
2. **Warna tombol konsisten** di semua halaman (primary selalu biru, danger selalu merah).
3. **Ikon otomatis** muncul sesuai variant (jika kamu pakai fitur ikon dari tahap 4).
4. **Tidak ada lagi tombol dengan style acak** seperti `btn-light` atau `btn-outline-secondary`.

> Kalau semua halaman tampil benar, **selamat!** Kamu sudah berhasil merefaktor seluruh CRUD Produk dengan komponen Button.

---

## 11. Perbandingan Sebelum vs Sesudah Refactor

### 11.1 Kode tombol di halaman `create.blade.php`

**Sebelum:**

```blade
<button type="submit" class="btn btn-primary">Simpan Produk</button>
<a href="..." class="btn btn-light">Batal</a>
```

**Sesudah:**

```blade
<x-button type="submit" variant="primary">Simpan Produk</x-button>
<x-button variant="secondary" href="...">Batal</x-button>
```

### 11.2 Jumlah duplikasi class CSS

| Hal                     | Sebelum Refactor      | Sesudah Refactor             |
| ----------------------- | --------------------- | ---------------------------- |
| Class `btn btn-*`       | Ditulis manual        | Otomatis dari komponen       |
| Konsistensi Batal       | `btn-light` vs `btn-secondary` (tidak konsisten) | Selalu `variant="secondary"` (konsisten) |
| Kemudahan ubah style    | Cari semua file satu-satu | Ubah di `button.blade.php`, efek ke semua |
| Risiko typo class       | Tinggi (`btn-prmary`) | Rendah (divalidasi komponen) |

### 11.3 Keuntungan jangka panjang

Bayangkan **6 bulan ke depan** kamu ingin:

**Skenario A:** Ganti semua tombol primary dari biru ke ungu.

- **Sebelum refactor:** Cari semua file yang punya `btn-primary`, ganti satu-satu. Mungkin **5-10 file**.
- **Sesudah refactor:** Cukup ubah **satu baris** di `button.blade.php`.

**Skenario B:** Tambahkan ikon ke semua tombol Hapus.

- **Sebelum refactor:** Tambah `<i class="fas fa-trash"></i>` di **setiap tombol Hapus** di semua halaman.
- **Sesudah refactor:** Tambah ikon di array `$ikonPerVariant` di komponen, **selesai**.

---

## 12. Troubleshooting

### Error 1: Tombol tampil sebagai `<button>` padahal seharusnya link

**Penyebab:**

Kamu kirim `href="..."` ke komponen, tapi komponen menghasilkan `<button href="...">` yang **bukan link**.

**Solusi sementara:**

Atribut `href` akan diabaikan browser di tag `<button>`. Tombol tampil, tapi **tidak bisa diklik untuk pindah halaman**. Untuk sekarang, bungkus dengan `<a>` manual:

```blade
<a href="{{ route('produk.index') }}" class="text-decoration-none">
    <x-button variant="secondary">Batal</x-button>
</a>
```

> **Solusi permanen:** Ini akan dibahas di **tahap 7** (Variant Tambahan dan Tipe Link), di mana kita akan modifikasi komponen supaya bisa **otomatis** jadi `<a>` kalau ada `href`.

### Error 2: Form Hapus tidak berfungsi

**Penyebab:**

Tombol Hapus tidak punya `type="submit"`, atau form tidak punya `@method('DELETE')`.

**Solusi:**

Pastikan:

1. Tombol Hapus di komponen pakai `type="submit"`.
2. Form punya `@csrf` dan `@method('DELETE')`.

```blade
<form action="..." method="POST" style="display:inline">
    @csrf
    @method('DELETE')
    <x-button variant="danger" type="submit">Hapus</x-button>
</form>
```

### Error 3: `MethodNotAllowedHttpException`

**Penyebab:**

Lupa pakai `@method('DELETE')` di form Hapus, atau lupa `@method('PUT')` di form Edit.

**Solusi:**

Tambah directive `@method` yang sesuai:

- Form Hapus: `@method('DELETE')`
- Form Edit: `@method('PUT')` (atau `@method('PATCH')`)

### Error 4: Tombol tidak tampil sama sekali

**Penyebab:**

- Lupa membuat file komponen `button.blade.php`.
- Nama file salah (misal: `Button.blade.php`).
- Folder komponen salah.

**Solusi:**

Cek struktur:

```
resources/views/components/button.blade.php
```

Pastikan nama file **kebab-case** dan ada di folder `components/`.

---

## 13. Latihan Mandiri

**Latihan D:**

Coba refactor **halaman dashboard admin** (`dashboard/index.blade.php`) yang masih pakai tombol manual. Ganti semua tombol dengan `<x-button>`.

**Kode lama (contoh):**

```blade
@extends('layout.app', ['title' => 'Dashboard Admin'])

@section('konten')
    <h2>Dashboard Admin</h2>

    <div class="mt-3">
        <a href="{{ route('produk.index') }}" class="btn btn-primary">
            Kelola Produk
        </a>
        <a href="{{ route('produk.create') }}" class="btn btn-success">
            Tambah Produk Baru
        </a>
        <a href="/logout" class="btn btn-danger">
            Logout
        </a>
    </div>
@endsection
```

<details>
<summary><strong>Lihat jawaban Latihan D</strong></summary>

**Analisis variant:**

- **Kelola Produk** = aksi utama dashboard → `primary`
- **Tambah Produk Baru** = aksi mengarah ke form tambah → `primary` (atau `info` kalau dianggap informatif)
- **Logout** = aksi keluar (bukan menghapus data) → `secondary` atau `danger` (tergantung preferensi)

**Kode baru:**

```blade
@extends('layout.app', ['title' => 'Dashboard Admin'])

@section('konten')
    <h2>Dashboard Admin</h2>

    <div class="mt-3 d-flex gap-2">
        <x-button variant="primary" href="{{ route('produk.index') }}">
            Kelola Produk
        </x-button>

        <x-button variant="info" href="{{ route('produk.create') }}">
            Tambah Produk Baru
        </x-button>

        <x-button variant="danger" href="/logout">
            Logout
        </x-button>
    </div>
@endsection
```

**Catatan:**

- Kelola Produk → `primary` karena ini aksi utama dashboard.
- Tambah Produk Baru → `info` karena dashboard sudah punya `primary` (Kelola Produk), jadi tambah produk jadi `info` (informatif).
- Logout → `danger` karena logout adalah aksi yang **mengakhiri sesi** (agak berbahaya, pengguna harus login lagi).

</details>

---

## 14. Istilah Kunci Tahap Ini

| Istilah            | Arti sederhana                                          |
| ------------------ | ------------------------------------------------------- |
| **Refactor**       | Memperbaiki struktur kode tanpa mengubah fungsinya       |
| **Konsistensi**    | Keseragaman tampilan dan pola di seluruh aplikasi       |
| **Pola tombol**    | Aturan tetap yang menentukan variant untuk setiap aksi  |
| **Verifikasi**     | Memastikan semua halaman tampil benar setelah refactor   |

---

## 15. Ringkasan Tahap 5

1. Kita telah **merefactor 4 halaman** CRUD Produk: `index`, `create`, `edit`, `show`.
2. **Pola variant yang konsisten** muncul setelah refactor:
   - Aksi utama → `primary`
   - Edit → `warning`
   - Hapus → `danger`
   - Detail → `info`
   - Batal/Kembali → `secondary`
3. **Manfaat refactor:**
   - Semua tombol **konsisten** di semua halaman.
   - **Mudah dirawat** (ubah di satu tempat, efek ke semua).
   - **Hemat kode** (tidak perlu tulis class manual).
   - **Anti typo** (variant divalidasi).
4. **Verifikasi di browser**: semua halaman tampil dengan style yang konsisten.
5. **Perbaikan kualitas**: tombol Update dari `success` jadi `primary` (lebih tepat perannya).

---

## 16. Cek Pemahaman

1. Sebutkan 4 halaman yang direfactor di tahap ini.
2. Variant apa yang dipakai untuk tombol "Simpan Produk"? Kenapa?
3. Kenapa tombol "Update Produk" diubah dari `success` ke `primary`?
4. Apa yang harus dilakukan kalau mau menambahkan ikon ke semua tombol Hapus setelah refactor?
5. Apa pola variant yang konsisten untuk aksi "Batal" di semua halaman?
6. Bagaimana cara memverifikasi bahwa refactor berhasil?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. `produk/index.blade.php`, `produk/create.blade.php`, `produk/edit.blade.php`, dan `produk/show.blade.php`.
2. `variant="primary"`, karena Simpan Produk adalah **aksi utama** di halaman tambah produk.
3. Karena Update adalah **aksi utama** di halaman edit, bukan konfirmasi keberhasilan. `success` untuk konfirmasi keberhasilan, `primary` untuk aksi utama.
4. Cukup tambahkan ikon di array `$ikonPerVariant['danger']` di komponen `button.blade.php`. Semua tombol Hapus di seluruh aplikasi akan otomatis dapat ikon.
5. Selalu `variant="secondary"`, baik di halaman `create` maupun `edit`.
6. Buka semua 4 halaman di browser, cek apakah tombol tampil dengan warna yang sesuai variant dan tidak ada error.

</details>

---

## 17. Apakah Kamu Ingin Lanjut?

Di tahap 5 ini kamu sudah **merefactor seluruh CRUD Produk** dengan komponen `<x-button>`. Semua tombol sekarang konsisten.

Tapi masih ada **satu masalah** yang belum diselesaikan: tombol dengan `href` (seperti Batal, Kembali, Edit) saat ini **tidak berfungsi sebagai link** karena komponen kita menghasilkan `<button>`, bukan `<a>`.

Di tahap berikutnya, kita akan **bongkar kasus khusus**: **tombol Hapus yang butuh form + konfirmasi**, dan bagaimana menanganinya dengan rapi.

> ### Pertanyaan:
>
> Apakah kamu ingin lanjut ke langkah berikutnya: **Tombol Hapus dengan Form dan Konfirmasi**?
>
> Di tahap 6 kita akan bahas:
>
> 1. Kenapa tombol Hapus butuh `<form>` (karena method DELETE)
> 2. Cara bungkus `<x-button>` di dalam form
> 3. Tambahkan konfirmasi `onclick="return confirm(...)"`
> 4. Buat komponen turunan `<x-button-delete>` khusus untuk hapus
> 5. Alternatif: pakai form inline dengan styling minimal
>
> Ketik **"lanjut"** untuk ke tahap 6,
> atau tanyakan jika ada bagian tahap 5 yang masih bingung.

---

> **Daftar Tahap (13. Komponen Button Blade):**
> - [x] Tahap 1 — Kenapa Tombol Harus Konsisten
> - [x] Tahap 2 — Membuat File Komponen `<x-button>` Sederhana
> - [x] Tahap 3 — Memahami Props dan Variant
> - [x] Tahap 4 — Slot dan Isi Tombol
> - [x] Tahap 5 — Menerapkan `<x-button>` di Halaman Produk (kamu di sini)
> - [ ] Tahap 6 — Tombol Hapus dengan Form dan Konfirmasi
> - [ ] Tahap 7 — Variant Tambahan dan Tipe Link
> - [ ] Tahap 8 — Ringkasan dan Best Practice
