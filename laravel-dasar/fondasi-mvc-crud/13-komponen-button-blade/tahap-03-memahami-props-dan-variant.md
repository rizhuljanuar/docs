# Tahap 3 — Memahami Props dan Variant Secara Mendalam

> Materi: 13. Komponen Button Blade
> Level: Pemula — Fondasi Laravel, MVC, CRUD
> Fokus tahap ini: **memahami apa itu variant**, kapan pakai variant yang mana, dan bagaimana mengatur logika warna di dalam komponen.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Menjelaskan apa itu **variant** dan hubungannya dengan **props**.
2. Menyebutkan **5 variant utama** dan kapan masing-masing dipakai.
3. Memetakan tombol-tombol di CRUD Produk ke variant yang tepat.
4. Memahami cara kerja logika `variant` di dalam komponen `button.blade.php`.
5. Menambahkan **validasi variant** agar komponen tidak menerima variant acak.
6. Mengatur **variant default** yang masuk akal.

Di tahap 2 kita sudah bikin komponen dasar, tapi kita belum bahas **kenapa** `primary` itu biru, **kapan** harus pakai `danger`, atau **apa bedanya** `secondary` dengan `warning`. Itu yang akan kita kupas habis sekarang.

---

## 2. Analogi: Seragam Pekerja di Rumah Sakit

Bayangkan kamu masuk ke rumah sakit. Kamu melihat banyak orang dengan **warna seragam berbeda**:

| Warna Seragam  | Pekerjaan          | Makna warna              |
| -------------- | ------------------ | ------------------------ |
| **Putih**      | Dokter             | "Saya otoritas medis"    |
| **Hijau**      | Perawat            | "Saya yang merawat"      |
| **Biru**       | Resepsionis        | "Saya bisa bantu info"   |
| **Merah**      | Petugas darurat    | "Hati-hati, ada situasi" |
| **Kuning**     | Petugas kebersihan | "Mohon geser, saya sapu" |

Tanpa bertanya, kamu **langsung tahu** siapa orang itu dari warna seragamnya.

### Hubungannya dengan variant tombol

Tombol di website itu seperti **pekerja di rumah sakit**. Warna tombol memberi tahu pengguna **apa peran** tombol itu:

| Warna Tombol | Variant       | Peran Tombol                 |
| ------------ | ------------- | ---------------------------- |
| **Biru**     | `primary`     | "Saya aksi utama"            |
| **Abu-abu**  | `secondary`   | "Saya aksi biasa"            |
| **Hijau**    | `success`     | "Saya tanda keberhasilan"    |
| **Merah**    | `danger`      | "Hati-hati, saya berbahaya"  |
| **Kuning**   | `warning`     | "Mohon perhatian"            |

Jadi **variant** itu bukan sekadar "warna", tapi **peran** tombol. Warna hanya cara untuk **menyampaikan peran** secara visual.

> **Pesan mentor:**
> Pemula sering keliru mengira variant cuma soal " warna kesukaan". Salah! Variant itu soal **komunikasi**. Pemakaiannya harus konsisten di seluruh aplikasi, supaya pengguna **tidak bingung**.

---

## 3. Apa Itu Variant Sebenarnya?

Di tahap 2, kita sudah pakai `variant="primary"` dan `variant="danger"` tanpa benar-benar memahaminya. Sekarang kita kupas habis.

### Definisi

**Variant** adalah **nilai props** yang menentukan **versi tampilan** sebuah komponen.

Ingat: `variant` hanyalah **salah satu props** (input) komponen. Namanya boleh apa saja (`warna`, `jenis`, `tipe`), tapi komunitas Laravel dan Bootstrap sepakat memakai nama **`variant`** untuk konsistensi.

### Variant vs Props

| Konsep   | Hubungannya                                          |
| -------- | ---------------------------------------------------- |
| **Props** | Istilah umum untuk **semua input** komponen         |
| **Variant** | Salah satu props khusus yang menentukan **gaya/tampilan** |

Analoginya: **Props** itu seperti "jenis kelamin" (umum kategori), sedangkan **Variant** itu seperti "laki-laki/perempuan" (nilai spesifik di dalamnya).

### Contoh

Komponen kita menerima **1 props** bernama `variant`:

```blade
@props(['variant' => 'primary'])
```

Nilai `variant` bisa berupa:

- `"primary"`
- `"secondary"`
- `"success"`
- `"danger"`
- `"warning"`
- `"info"` (opsional, untuk hal-hal netral seperti "Detail")

Masing-masing nilai akan **mengubah tampilan** tombol di browser.

---

## 4. Daftar Variant Utama (Tabel Wajib Hafal)

Berikut adalah **5 variant utama** yang HARUS kamu hafal, karena dipakai di **semua** aplikasi web:

### 4.1 `primary` — Aksi Utama

| Hal            | Detail                                          |
| -------------- | ----------------------------------------------- |
| **Warna**      | Biru                                            |
| **Makna**      | "Ini aksi yang paling penting di halaman ini"  |
| **Kapan dipakai** | Hanya **satu per halaman** (aksi utama)       |
| **Contoh**     | Tombol **Simpan Produk** di halaman tambah produk |

```blade
<x-button variant="primary">Simpan Produk</x-button>
```

> **Pesan mentor:**
> Aturan emas: **hanya satu tombol `primary` per halaman**. Kalau ada dua tombol primary, pengguna akan bingung mana aksi utamanya. Ini mirip dengan rumah: hanya ada **satu pintu depan** sebagai pintu utama.

### 4.2 `secondary` — Aksi Biasa / Netral

| Hal            | Detail                                          |
| -------------- | ----------------------------------------------- |
| **Warna**      | Abu-abu                                         |
| **Makna**      | "Ini aksi biasa, tidak penting tapi ada"        |
| **Kapan dipakai** | Untuk aksi sekunder yang **tidak berbahaya**  |
| **Contoh**     | Tombol **Batal**, tombol **Kembali**            |

```blade
<x-button variant="secondary">Batal</x-button>
<x-button variant="secondary">Kembali</x-button>
```

### 4.3 `success` — Tanda Keberhasilan

| Hal            | Detail                                          |
| -------------- | ----------------------------------------------- |
| **Warna**      | Hijau                                           |
| **Makna**      | "Aksi ini menandai keberhasilan"                |
| **Kapan dipakai** | Saat pengguna **menyelesaikan** sesuatu      |
| **Contoh**     | Tombol **Selesai**, tombol **Konfirmasi Pesanan** |

```blade
<x-button variant="success">Selesai</x-button>
```

> **Catatan:** Jangan pakai `success` untuk tombol "Simpan", karena Simpan adalah aksi utama (pakai `primary`). `success` lebih ke arah **konfirmasi** sesuatu yang sudah berhasil.

### 4.4 `danger` — Aksi Berbahaya

| Hal            | Detail                                          |
| -------------- | ----------------------------------------------- |
| **Warna**      | Merah                                           |
| **Makna**      | "Hati-hati, aksi ini tidak bisa dibatalkan"     |
| **Kapan dipakai** | Saat aksi akan **menghapus/menghancurkan** data |
| **Contoh**     | Tombol **Hapus Produk**, tombol **Hapus Akun**  |

```blade
<x-button variant="danger">Hapus Produk</x-button>
```

> **Pesan mentor:**
> `danger` itu seperti **tombol alarm**. Harus jarang dipakai, dan **hanya** untuk aksi yang merusak. Kalau kamu pakai danger untuk tombol "Lihat Detail", pengguna akan takut mengklik karena merah = bahaya.

### 4.5 `warning` — Perhatian, Tapi Tidak Berbahaya

| Hal            | Detail                                          |
| -------------- | ----------------------------------------------- |
| **Warna**      | Kuning / oranye                                 |
| **Makna**      | "Mohon perhatian, ada konsekuensi ringan"       |
| **Kapan dipakai** | Saat aksi **bukan menghapus**, tapi perlu perhatian |
| **Contoh**     | Tombol **Edit**, tombol **Nonaktifkan Produk**  |

```blade
<x-button variant="warning">Edit Produk</x-button>
<x-button variant="warning">Nonaktifkan</x-button>
```

> **Bedanya `warning` vs `danger`:**
> `danger` = aksi **menghapus** permanen (merah). `warning` = aksi **mengubah/nonaktifkan** (kuning). Hapus itu lebih berbahaya daripada edit, jadi hapus pakai merah.

---

## 5. Variant Tambahan: `info`

Selain 5 variant utama, Bootstrap juga punya variant `info` yang berguna:

| Hal            | Detail                                          |
| -------------- | ----------------------------------------------- |
| **Warna**      | Biru muda / cyan                                |
| **Makna**      | "Aksi informatif, membuka info tambahan"        |
| **Kapan dipakai** | Untuk tombol yang **menampilkan informasi**   |
| **Contoh**     | Tombol **Detail Produk**, tombol **Lihat Info** |

```blade
<x-button variant="info">Detail Produk</x-button>
```

> **Kenapa Detail pakai `info`, bukan `primary`?**
> Karena `primary` itu untuk **aksi utama halaman**. Di halaman daftar produk, aksi utamanya adalah **Tambah Produk**, bukan Detail. Detail hanya aksi informatif, jadi pakai `info`.

---

## 6. Tabel Ringkas: Pemetaan Tombol CRUD Produk

Sekarang, mari kita **petakan** semua tombol di project kita ke variant yang tepat:

| Tombol           | Halaman Muncul                    | Variant       | Alasan                                |
| ---------------- | --------------------------------- | ------------- | ------------------------------------- |
| **Tambah Produk** | `produk/index.blade.php`         | `primary`     | Aksi utama di halaman daftar          |
| **Simpan Produk** | `produk/create.blade.php`        | `primary`     | Aksi utama di halaman tambah          |
| **Update Produk** | `produk/edit.blade.php`          | `primary`     | Aksi utama di halaman edit            |
| **Edit Produk**   | `produk/index.blade.php`, `show` | `warning`     | Aksi mengubah, butuh perhatian        |
| **Hapus Produk**  | `produk/index.blade.php`, `show` | `danger`      | Aksi menghapus, berbahaya             |
| **Detail Produk** | `produk/index.blade.php`         | `info`        | Aksi informatif, bukan utama          |
| **Batal**         | `produk/create.blade.php`, `edit`| `secondary`   | Aksi netral, tidak penting            |
| **Kembali**       | `produk/show.blade.php`          | `secondary`   | Aksi netral, kembali ke daftar        |

> **Pesan mentor:**
> Coba amati: **tidak ada** dua aksi utama dalam satu halaman. Di halaman `create`, hanya tombol Simpan yang `primary`, tombol Batal `secondary`. Ini konsisten dengan aturan "satu `primary` per halaman".

---

## 7. Cara Kerja Logika Variant di Komponen

Sekarang saatnya **bongkar** bagaimana komponen kita menerjemahkan `variant="primary"` menjadi tombol berwarna biru.

### 7.1 Kode komponen saat ini (dari tahap 2)

```blade
@props(['variant' => 'primary'])

<button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
    {{ $slot }}
</button>
```

### 7.2 Apa yang terjadi saat dipanggil?

Saat kamu menulis:

```blade
<x-button variant="danger">Hapus Produk</x-button>
```

Maka:

1. `$variant` di dalam komponen bernilai `"danger"` (dari pemanggil).
2. String `'btn btn-' . $variant` menjadi `'btn btn-danger'`.
3. Hasilnya: `<button class="btn btn-danger">Hapus Produk</button>`.

### 7.3 Tapi dari mana warnanya datang?

Warnanya **bukan** dari komponen kita. Warnanya datang dari **Bootstrap CSS**.

Bootstrap sudah punya **class-class siap pakai** untuk tombol:

| Class CSS (Bootstrap) | Warna yang Dihasilkan |
| --------------------- | --------------------- |
| `btn btn-primary`     | Biru                  |
| `btn btn-secondary`   | Abu-abu               |
| `btn btn-success`     | Hijau                 |
| `btn btn-danger`      | Merah                 |
| `btn btn-warning`     | Kuning                |
| `btn btn-info`        | Cyan / biru muda      |

Komponen kita **cukup** menghasilkan class `btn btn-{variant}`, lalu **Bootstrap yang mewarnai** tombolnya.

> **Pesan mentor:**
> Ini penting dipahami: komponen kita **hanya menghasilkan class CSS**. Yang mewarnai adalah Bootstrap. Jadi kalau kamu ganti dari Bootstrap ke Tailwind, yang berubah adalah **rumus class** di dalam komponen, bukan cara pemanggilan `<x-button variant="primary">`.

---

## 8. Masalah: Variant Asal-Akalan

Sekarang muncul masalah. Apa yang terjadi kalau pemanggil **salah mengetik** variant?

```blade
<x-button variant="prmary">Simpan</x-button>
<!--              ↑↑↑↑↑↑
                  salah ketik: prmary bukan primary -->
```

Apa yang terjadi?

1. `$variant` bernilai `"prmary"` (salah ketik).
2. Class yang dihasilkan: `btn btn-prmary`.
3. Bootstrap **tidak kenal** class `btn-prmary`.
4. Hasilnya: tombol tampil **tanpa warna** (putih polos).

Ini **berbahaya** karena:

- Tidak ada error, jadi kamu **tidak sadar** ada yang salah.
- Tombol tampil polos, pengguna bingung.
- Bug tersembunyi yang sulit dilacak.

### Solusi: Validasi Variant

Kita bisa tambahkan **validasi** di dalam komponen supaya hanya menerima variant yang sudah ditetapkan.

---

## 9. Langkah 1: Tambahkan Daftar Variant yang Diterima

Kita modifikasi komponen `button.blade.php`:

```blade
@props([
    'variant' => 'primary',
])

@php
    $variantYangDiterima = ['primary', 'secondary', 'success', 'danger', 'warning', 'info'];

    if (!in_array($variant, $variantYangDiterima)) {
        throw new \InvalidArgumentException(
            "Variant '{$variant}' tidak dikenal. Variant yang valid: " .
            implode(', ', $variantYangDiterima)
        );
    }
@endphp

<button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
    {{ $slot }}
</button>
```

### Penjelasan baris per baris

#### `@props(['variant' => 'primary'])`

Sama seperti sebelumnya. Deklarasi props dengan default `"primary"`.

#### `@php ... @endphp`

Ini directive untuk menulis **kode PHP** di dalam file Blade. Semua di antara `@php` dan `@endphp` akan dieksekusi sebagai PHP.

#### `$variantYangDiterima = [...]`

Ini array berisi **daftar variant yang valid**. Hanya 6 variant yang diterima:

- `primary`
- `secondary`
- `success`
- `danger`
- `warning`
- `info`

#### `if (!in_array($variant, $variantYangDiterima))`

Fungsi `in_array($jarum, $jerami)` di PHP berfungsi **mencari** apakah `$jarum` ada di dalam array `$jerami`.

- Kalau ada → kembalikan `true`
- Kalau tidak ada → kembalikan `false`

`!in_array(...)` artinya "jika variant **tidak ada** di daftar".

#### `throw new \InvalidArgumentException(...)`

Ini melempar **error** dengan pesan yang jelas, supaya developer langsung tahu salahnya di mana.

**Contoh pesan error:**

```
Variant 'prmary' tidak dikenal. Variant yang valid: primary, secondary, success, danger, warning, info
```

Sekarang kalau ada yang salah ketik `variant="prmary"`, halaman akan error dengan **pesan jelas**, bukan tombol putih polos tanpa keterangan.

> **Pesan mentor:**
> Validasi ini opsional untuk pemula, tapi **sangat dianjurkan** untuk aplikasi nyata. Tanpa validasi, typo variant akan jadi bug **siluman** yang sulit dilacak.

---

## 10. Latihan: Tebak Variant yang Tepat

Sebelum lanjut, mari uji pemahamanmu. Coba tebak **variant yang tepat** untuk setiap kasus berikut:

| Skenario                                                  | Variant?     |
| --------------------------------------------------------- | ------------ |
| 1. Tombol "Simpan Produk" di halaman tambah produk        | ?            |
| 2. Tombol "Hapus Produk" di halaman daftar produk         | ?            |
| 3. Tombol "Batal" di halaman tambah produk                | ?            |
| 4. Tombol "Detail Produk" di halaman daftar produk        | ?            |
| 5. Tombol "Kembali ke Daftar" di halaman detail produk    | ?            |
| 6. Tombol "Edit Produk" di halaman daftar produk          | ?            |
| 7. Tombol "Konfirmasi Pesanan" di halaman checkout        | ?            |
| 8. Tombol "Nonaktifkan Produk" di halaman admin           | ?            |

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

| Skenario                               | Variant       | Alasan                              |
| -------------------------------------- | ------------- | ----------------------------------- |
| 1. Simpan Produk (aksi utama)          | `primary`     | Aksi paling penting di halaman      |
| 2. Hapus Produk                        | `danger`      | Aksi merusak data permanen          |
| 3. Batal                               | `secondary`   | Aksi netral, tidak penting          |
| 4. Detail Produk                       | `info`        | Aksi informatif                     |
| 5. Kembali ke Daftar                   | `secondary`   | Aksi netral                         |
| 6. Edit Produk                         | `warning`     | Aksi mengubah, butuh perhatian      |
| 7. Konfirmasi Pesanan                  | `success`     | Tanda keberhasilan menyelesaikan   |
| 8. Nonaktifkan Produk                  | `warning`     | Aksi mengubah status                |

</details>

---

## 11. Eksperimen: Coba Semua Variant di Satu Halaman

Sekarang mari kita **lihat semua variant** secara visual. Buat satu halaman sementara untuk mencoba semua variant.

### 11.1 Buat file `produk/coba-tombol.blade.php`

```blade
@extends('layout.app', ['title' => 'Coba Tombol'])

@section('konten')
    <h2>Coba Semua Variant</h2>

    <div style="display: flex; gap: 10px; flex-wrap: wrap; margin-top: 20px;">
        <x-button variant="primary">Primary</x-button>
        <x-button variant="secondary">Secondary</x-button>
        <x-button variant="success">Success</x-button>
        <x-button variant="danger">Danger</x-button>
        <x-button variant="warning">Warning</x-button>
        <x-button variant="info">Info</x-button>
    </div>

    <h3 style="margin-top: 30px;">Contoh Pemakaian Nyata</h3>

    <div style="display: flex; gap: 10px; flex-wrap: wrap; margin-top: 10px;">
        <x-button variant="primary">Simpan Produk</x-button>
        <x-button variant="warning">Edit Produk</x-button>
        <x-button variant="danger">Hapus Produk</x-button>
        <x-button variant="info">Detail Produk</x-button>
        <x-button variant="secondary">Batal</x-button>
        <x-button variant="secondary">Kembali</x-button>
    </div>
@endsection
```

### 11.2 Tambahkan route sementara

Di `routes/web.php`, tambahkan:

```php
Route::get('/coba-tombol', function () {
    return view('produk.coba-tombol');
});
```

### 11.3 Lihat hasilnya di browser

Buka:

```
http://localhost:8000/coba-tombol
```

Kamu harusnya melihat **6 tombol berwarna** berjejer: biru, abu-abu, hijau, merah, kuning, cyan.

**Coba amati:**

- Semua tombol punya **ukuran dan padding yang sama** (karena sama-sama pakai class `btn`).
- Hanya **warnanya** yang beda, karena variant-nya beda.
- Semua tombol terlihat **konsisten** karena dibuat dari **satu komponen yang sama**.

> **Pesan mentor:**
> Halaman coba-tombol ini hanya untuk **pembelajaran**. Setelah selesai belajar, hapus file dan route-nya supaya tidak mengotori aplikasi.

---

## 12. Apa yang Sudah Kita Capai di Tahap Ini?

| Konsep                                          | Status |
| ----------------------------------------------- | ------ |
| Memahami 5 variant utama + info                 | ✅      |
| Memetakan tombol CRUD ke variant yang tepat     | ✅      |
| Memahami cara kerja logika `variant` di komponen | ✅     |
| Validasi variant untuk cegah typo               | ✅      |
| Mencoba semua variant secara visual             | ✅      |

### Penting untuk dicerna

1. **Variant bukan sekadar warna**, tapi **peran** tombol.
2. **Primary hanya untuk satu aksi utama** per halaman.
3. **Danger hanya untuk aksi merusak** (hapus).
4. **Warning untuk aksi mengubah** (edit, nonaktifkan).
5. **Secondary untuk aksi netral** (batal, kembali).
6. **Info untuk aksi informatif** (detail).
7. **Success untuk konfirmasi keberhasilan** (bukan simpan biasa).

---

## 13. Troubleshooting

### Error 1: `InvalidArgumentException: Variant 'prmary' tidak dikenal...`

**Penyebab:**

Salah ketik nama variant saat memanggil komponen.

**Solusi:**

Cek daftar variant yang valid di pesan error, lalu perbaiki typo.

### Error 2: Tombol tampil putih polos tanpa warna

**Penyebab:**

Kemungkinan besar kamu **tidak memakai Bootstrap**, atau class `btn btn-{variant}` tidak ada di CSS yang di-load.

**Solusi:**

Cek di `layout/app.blade.php` apakah Bootstrap CSS sudah di-load. Kalau tidak, tambahkan:

```html
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
```

### Error 3: Halaman blank / error 500

**Penyebab:**

Mungkin ada **salah sintaks** di kode PHP `@php ... @endphp`. Cek koma, kurung, dan titik koma.

**Solusi:**

Bandingkan dengan kode di tahap ini. Pastikan tidak ada typo di `in_array`, `implode`, atau `throw new`.

### Error 4: Tombol ada, tapi teksnya tidak muncul

**Penyebab:**

Lupa menulis teks di dalam tag, atau menulis komponen sebagai self-closing.

**Solusi:**

Pastikan menulis teks di antara tag pembuka dan penutup:

```blade
<!-- Benar -->
<x-button variant="primary">Simpan</x-button>

<!-- Salah (self-closing) -->
<x-button variant="primary" />
```

---

## 14. Latihan Mandiri

**Latihan B:**

Coba **refactor** (perbaiki) halaman `produk/edit.blade.php` yang sebelumnya pakai kode seperti ini:

```blade
<form action="/products/{{ $product->id }}" method="POST">
    @csrf
    @method('PUT')

    <!-- ...input form... -->

    <button type="submit" class="btn btn-success px-4">
        Update Produk
    </button>
    <a href="/products" class="btn btn-secondary">
        Batal
    </a>
</form>
```

Ubah kedua tombol di atas pakai `<x-button>` dengan **variant yang tepat**.

<details>
<summary><strong>Lihat jawaban Latihan B</strong></summary>

**Analisis:**

- Tombol **Update Produk** = aksi utama di halaman edit → `primary` (bukan `success`!)
- Tombol **Batal** = aksi netral → `secondary`

**Hasil refactor:**

```blade
<form action="/products/{{ $product->id }}" method="POST">
    @csrf
    @method('PUT')

    <!-- ...input form... -->

    <x-button type="submit" variant="primary">
        Update Produk
    </x-button>

    <x-button variant="secondary" href="/products">
        Batal
    </x-button>
</form>
```

**Perhatikan:**

- Kita ganti `btn btn-success` → `variant="primary"`. Kenapa? Karena Update Produk adalah **aksi utama** di halaman edit, bukan aksi keberhasilan. `success` adalah untuk konfirmasi keberhasilan, bukan untuk menyimpan.
- Sekarang tombol Update Produk berwarna **biru**, bukan hijau. Ini lebih konsisten dengan tombol Simpan di halaman create (yang juga biru).

</details>

---

## 15. Istilah Kunci Tahap Ini

| Istilah                      | Arti sederhana                                          |
| ---------------------------- | ------------------------------------------------------- |
| **Variant**                  | Nilai props yang menentukan versi tampilan komponen     |
| **Primary**                  | Variant untuk aksi utama (biru)                         |
| **Secondary**                | Variant untuk aksi netral (abu-abu)                     |
| **Success**                  | Variant untuk konfirmasi keberhasilan (hijau)           |
| **Danger**                   | Variant untuk aksi berbahaya (merah)                    |
| **Warning**                  | Variant untuk aksi yang perlu perhatian (kuning)        |
| **Info**                     | Variant untuk aksi informatif (cyan)                    |
| **Validasi variant**         | Mencegah variant asal-asalan dengan `in_array`          |
| **`@php ... @endphp`**       | Directive untuk menulis kode PHP di dalam Blade         |

---

## 16. Ringkasan Tahap 3

1. **Variant** = props yang menentukan **peran** tombol, bukan sekadar warna.
2. **5 variant utama**:
   - `primary` = aksi utama (biru) — **hanya satu per halaman**
   - `secondary` = aksi netral (abu-abu) — batal, kembali
   - `success` = konfirmasi keberhasilan (hijau) — selesai
   - `danger` = aksi berbahaya (merah) — hapus
   - `warning` = perlu perhatian (kuning) — edit
3. **Variant tambahan**: `info` untuk aksi informatif (cyan) — detail
4. **Cara kerja**: komponen menghasilkan class CSS `btn btn-{variant}`, lalu **Bootstrap** yang mewarnai.
5. **Validasi variant** mencegah typo dengan `in_array`.
6. **Pemetaan tombol CRUD Produk** ke variant yang tepat sudah dirangkum di tabel.
7. **Aturan emas**: hanya satu tombol `primary` per halaman.

---

## 17. Cek Pemahaman

1. Sebutkan 5 variant utama dan warnanya.
2. Kapan kamu pakai `warning`, dan kapan pakai `danger`? Apa bedanya?
3. Kenapa tombol "Detail Produk" sebaiknya pakai `info`, bukan `primary`?
4. Apa yang terjadi kalau kamu menulis `variant="hijau"` (variant asal-asalan)?
5. Apa fungsi `@php ... @endphp` di dalam komponen?
6. Di halaman `create.blade.php`, variant apa untuk tombol "Simpan"? Kenapa?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. **Primary** (biru), **secondary** (abu-abu), **success** (hijau), **danger** (merah), **warning** (kuning).
2. `warning` untuk aksi **mengubah** data (edit, nonaktifkan) — kuning. `danger` untuk aksi **merusak permanen** (hapus) — merah. Bedanya: danger lebih berbahaya karena tidak bisa dibatalkan.
3. Karena `primary` itu untuk **aksi utama halaman**. Di halaman daftar produk, aksi utamanya adalah "Tambah Produk", bukan "Detail". Detail hanya aksi informatif, jadi pakai `info`.
4. Tanpa validasi: tombol tampil **putih polos** karena class `btn btn-hijau` tidak ada di Bootstrap. Dengan validasi: halaman **error** dengan pesan jelas.
5. Untuk menulis **kode PHP** di dalam file Blade, seperti `in_array`, `throw`, logika kondisional, dll.
6. `primary`, karena "Simpan Produk" adalah **aksi utama** di halaman tambah produk. Hanya boleh ada satu primary per halaman.

</details>

---

## 18. Apakah Kamu Ingin Lanjut?

Di tahap 3 ini kamu sudah memahami **konsep variant** dan **pemetaan tombol** CRUD Produk ke variant yang tepat.

Tapi kita belum membahas **slot** secara mendalam. Di tahap 2 kita singgung sedikit soal `$slot`, tapi masih banyak yang bisa dilakukan dengan slot:

- Menambahkan **ikon** di dalam tombol (mis: ikon save sebelum teks "Simpan")
- Menambahkan **badge** (mis: jumlah item di keranjang)
- Slot **named slot** untuk bagian khusus

> ### Pertanyaan:
>
> Apakah kamu ingin lanjut ke langkah berikutnya: **memahami Slot dan isi tombol secara mendalam**?
>
> Di tahap 4 kita akan bahas:
>
> 1. Apa itu slot dan bedanya dengan props
> 2. Cara kerja `$slot` di dalam komponen Button
> 3. Menambahkan ikon di dalam tombol dengan slot
> 4. Slot untuk teks bebas (HTML di dalam tombol)
> 5. Default slot: apa yang terjadi kalau slot kosong
>
> Ketik **"lanjut"** untuk ke tahap 4,
> atau tanyakan jika ada bagian tahap 3 yang masih bingung.

---

> **Daftar Tahap (13. Komponen Button Blade):**
> - [x] Tahap 1 — Kenapa Tombol Harus Konsisten
> - [x] Tahap 2 — Membuat File Komponen `<x-button>` Sederhana
> - [x] Tahap 3 — Memahami Props dan Variant (kamu di sini)
> - [ ] Tahap 4 — Slot dan Isi Tombol
> - [ ] Tahap 5 — Menerapkan `<x-button>` di Halaman Produk
> - [ ] Tahap 6 — Tombol Hapus dengan Form dan Konfirmasi
> - [ ] Tahap 7 — Variant Tambahan dan Tipe Link
> - [ ] Tahap 8 — Ringkasan dan Best Practice
