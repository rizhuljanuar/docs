# Tahap 4 — Slot dan Isi Tombol Secara Mendalam

> Materi: 13. Komponen Button Blade
> Level: Pemula — Fondasi Laravel, MVC, CRUD
> Fokus tahap ini: **memahami apa itu slot**, bagaimana `$slot` bekerja di komponen Button, dan cara menambahkan isi fleksibel seperti ikon di dalam tombol.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Menjelaskan apa itu **slot** dan bedanya dengan **props**.
2. Menjelaskan cara kerja `{{ $slot }}` di dalam komponen.
3. Menambahkan **ikon** di dalam tombol menggunakan slot.
4. Memahami **default slot** dan apa yang terjadi kalau slot kosong.
5. Memahami kapan pakai **slot** dan kapan pakai **props**.
6. Membuat tombol dengan isi HTML (bukan sekadar teks).

Di tahap 2 dan 3 kita sudah pakai `{{ $slot }}` tapi belum dijelaskan mendalam. Sekarang kita kupas habis konsep **slot** ini, karena ini adalah **kekuatan terbesar** komponen Blade.

---

## 2. Analogi: Gelas Isi Bebas

Bayangkan kamu punya **gelas** di rumah.

Gelas itu **bentuknya tetap**:

- Tinggi 10 cm
- Mulut diameter 5 cm
- Bahan kaca bening

Tapi **isi gelas** bisa **berubah-ubah**:

- Pagi: diisi kopi hitam
- Siang: diisi air putih
- Sore: diisi es teh
- Malam: diisi susu hangat

**Gelasnya sama, isinya beda-beda.**

### Hubungannya dengan komponen Button

- **Gelas** = komponen `<x-button>` (bentuk tetap: tombol dengan padding, border, warna variant).
- **Isi gelas** = `$slot` (bisa diisi apa saja: teks, ikon, HTML).

Saat kamu menulis:

```blade
<x-button variant="primary">Simpan Produk</x-button>
```

- **Gelasnya** = komponen Button dengan variant primary (biru, ada padding).
- **Isinya** = teks `"Simpan Produk"`.

Saat kamu menulis:

```blade
<x-button variant="primary">💾 Simpan</x-button>
```

- **Gelasnya** = sama (komponen Button primary).
- **Isinya** = teks dengan emoji `"💾 Simpan"`.

**Gelasnya tidak berubah**, hanya isinya yang berbeda. Inilah inti dari slot.

---

## 3. Apa Itu Slot?

### Definisi sederhana

> **Slot** adalah **lubang kosong** di dalam komponen yang bisa **diisi apa saja** oleh pemanggil.

Di Blade, slot direpresentasikan dengan variabel **`$slot`** yang **otomatis tersedia** di dalam komponen.

### Cara kerja

Di **dalam komponen**, kita tulis `{{ $slot }}` untuk menandai "di sinilah lubang yang bisa diisi":

```blade
<!-- components/button.blade.php -->
@props(['variant' => 'primary'])

<button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
    {{ $slot }}
</button>
```

Di **pemanggil**, kita tulis isi di **antara tag pembuka dan penutup**:

```blade
<x-button variant="primary">
    Simpan Produk
    <!-- ↑↑↑↑↑↑↑↑↑↑↑
         ini yang akan masuk ke $slot -->
</x-button>
```

Apapun yang kamu tulis di antara `<x-button>` dan `</x-button>` akan **dimasukkan** ke `$slot` di dalam komponen.

---

## 4. Slot vs Props: Apa Bedanya?

Ini pertanyaan penting yang sering bingungkan pemula. Mari kita klarifikasi.

### Props itu seperti "FORMULIR"

Bayangkan kamu mengisi **formulir** di kantor:

- Nama: _____
- Alamat: _____
- Umur: _____

Formulir punya **field-field** yang harus diisi dengan **nilai tertentu** (teks, angka, tanggal). Tidak bisa diisi dengan paragraf panjang atau gambar.

**Props** di komponen juga seperti itu:

```blade
<x-button variant="primary" type="submit">
```

- `variant="primary"` → field "variant" diisi "primary"
- `type="submit"` → field "type" diisi "submit"

Props cocok untuk data yang **terstruktur** (string, angka, boolean).

### Slot itu seperti "AMPLOP SURAT"

Bayangkan kamu dikasih **amplop** oleh pos. Amplopnya **bentuknya tetap** (putih, persegi, ada perangko). Tapi **isi amplop** bisa apa saja:

- Surat tulisan
- Foto
- Uang
- Undangan

**Slot** di komponen juga seperti itu:

```blade
<x-button variant="primary">
    <!-- isi bebas: teks, HTML, ikon, apa saja -->
    <i class="icon-save"></i> Simpan Produk
</x-button>
```

Slot cocok untuk **konten bebas** (teks, HTML, gambar, bahkan komponen lain).

### Tabel Perbandingan

| Hal              | Props                           | Slot                              |
| ---------------- | ------------------------------- | --------------------------------- |
| **Analogi**      | Formulir                        | Amplop surat                      |
| **Cara kirim**   | Atribut: `variant="primary"`    | Isi di antara tag: `<x>...</x>`   |
| **Tipe data**    | String, angka, boolean, object  | HTML, teks, komponen apa saja     |
| **Jumlah**       | Banyak props (sesuai kebutuhan) | Biasanya 1 slot utama             |
| **Cocok untuk**  | Data terstruktur (warna, tipe)  | Konten bebas (teks tombol, ikon)  |

### Contoh di komponen Button

Kita pakai **keduanya** sekaligus:

```blade
<x-button type="submit" variant="primary">
    <i class="fas fa-save"></i> Simpan Produk
</x-button>
```

- **Props**: `type="submit"` dan `variant="primary"` (seperti formulir).
- **Slot**: `<i class="fas fa-save"></i> Simpan Produk` (seperti amplop surat).

> **Pesan mentor:**
> Aturan praktis: **kirim data/konfigurasi → pakai props**. **Kirim konten/tampilan → pakai slot**. Variant itu konfigurasi (pakai props). Teks tombol itu konten (pakai slot).

---

## 5. Latihan: Eksperimen dengan `$slot`

Sekarang mari kita **bereksperimen** untuk memahami bagaimana `$slot` bekerja.

### 5.1 Eksperimen 1: Teks Biasa

```blade
<x-button variant="primary">Simpan Produk</x-button>
```

Hasilnya di HTML:

```html
<button class="btn btn-primary">Simpan Produk</button>
```

**Yang masuk ke `$slot`**: teks `"Simpan Produk"`.

### 5.2 Eksperimen 2: Teks dengan Emoji

```blade
<x-button variant="primary">💾 Simpan Produk</x-button>
```

Hasilnya di HTML:

```html
<button class="btn btn-primary">💾 Simpan Produk</button>
```

**Yang masuk ke `$slot`**: teks dengan emoji `"💾 Simpan Produk"`.

Emoji adalah karakter teks biasa, jadi bisa langsung masuk ke slot.

### 5.3 Eksperimen 3: Teks dengan Tag HTML

```blade
<x-button variant="primary">
    <strong>Simpan</strong> Produk
</x-button>
```

Hasilnya di HTML:

```html
<button class="btn btn-primary">
    <strong>Simpan</strong> Produk
</button>
```

**Yang masuk ke `$slot`**: HTML `<strong>Simpan</strong> Produk`.

Slot **bisa berisi HTML**, bukan hanya teks biasa. Ini kekuatan besar slot.

### 5.4 Eksperimen 4: Slot Kosong

```blade
<x-button variant="primary"></x-button>
```

Hasilnya di HTML:

```html
<button class="btn btn-primary"></button>
```

**Yang masuk ke `$slot`**: string kosong `""`.

Tombol tampil **tanpa teks**, yang tentunya tidak berguna. Inilah kenapa kamu **harus** mengisi slot dengan sesuatu.

> **Jebakan pemula:**
> Self-closing tag `<x-button variant="primary" />` sama dengan slot kosong. Tombol akan tampil tanpa teks. Selalu tulis dengan **isi di dalamnya**.

---

## 6. Menambahkan Ikon di Dalam Tombol

Sekarang mari kita pakai slot untuk hal yang berguna: **menambahkan ikon** di dalam tombol.

### 6.1 Kenapa pakai ikon?

Ikon membuat tombol **lebih jelas** dan **lebih menarik** secara visual:

- Tombol Simpan dengan ikon 💾 → pengguna langsung paham ini tombol simpan.
- Tombol Edit dengan ikon ✏️ → pengguna langsung paham ini tombol edit.
- Tombol Hapus dengan ikon 🗑️ → pengguna langsung paham ini tombol hapus.

### 6.2 Dua cara menambahkan ikon

#### Cara 1: Pakai emoji (paling mudah)

Emoji adalah karakter teks, jadi bisa langsung dimasukkan ke slot:

```blade
<x-button variant="primary">💾 Simpan Produk</x-button>
<x-button variant="warning">✏️ Edit Produk</x-button>
<x-button variant="danger">🗑️ Hapus Produk</x-button>
<x-button variant="info">👁️ Detail Produk</x-button>
<x-button variant="secondary">↩️ Batal</x-button>
<x-button variant="secondary">⬅️ Kembali</x-button>
```

Hasilnya di browser:

- 💾 Simpan Produk (biru)
- ✏️ Edit Produk (kuning)
- 🗑️ Hapus Produk (merah)
- 👁️ Detail Produk (cyan)
- ↩️ Batal (abu-abu)
- ⬅️ Kembali (abu-abu)

#### Cara 2: Pakai ikon library (Font Awesome / Bootstrap Icons)

Kalau kamu sudah load **Font Awesome** atau **Bootstrap Icons** di layout, kamu bisa pakai ikon yang lebih profesional:

```blade
<x-button variant="primary">
    <i class="fas fa-save"></i> Simpan Produk
</x-button>

<x-button variant="warning">
    <i class="fas fa-edit"></i> Edit Produk
</x-button>

<x-button variant="danger">
    <i class="fas fa-trash"></i> Hapus Produk
</x-button>

<x-button variant="info">
    <i class="fas fa-eye"></i> Detail Produk
</x-button>

<x-button variant="secondary">
    <i class="fas fa-times"></i> Batal
</x-button>

<x-button variant="secondary">
    <i class="fas fa-arrow-left"></i> Kembali
</x-button>
```

> **Pesan mentor:**
> Untuk pemula, emoji sudah cukup. Tapi kalau kamu bikin aplikasi **serius**, pertimbangkan pakai **Font Awesome** atau **Bootstrap Icons** karena ikonnya lebih konsisten di semua browser dan device. Emoji kadang tampil beda di Windows vs Mac vs Android.

---

## 7. Props vs Slot: Kapan Pakai Yang Mana?

Ini pertanyaan yang sering muncul. Mari kita lihat **kasus nyata** dan pilih pendekatan yang tepat.

### Kasus 1: Teks tombol

**Pendekatan A - Pakai props:**

```blade
<x-button variant="primary" teks="Simpan Produk" />
```

Di komponen:

```blade
<button>{{ $teks }}</button>
```

**Pendekatan B - Pakai slot:**

```blade
<x-button variant="primary">Simpan Produk</x-button>
```

Di komponen:

```blade
<button>{{ $slot }}</button>
```

**Mana yang lebih baik?**

**Pendekatan B (slot)** lebih baik karena:

- Bisa diisi **HTML** (ikon, bold, dll).
- Sintaks lebih **natural** (seperti HTML biasa).
- Lebih **fleksibel** untuk perubahan ke depan.

> **Aturan praktis:** Untuk **konten tampilan** (teks, ikon, HTML), selalu pakai **slot**. Untuk **konfigurasi** (variant, type, href), pakai **props**.

### Kasus 2: Variant warna

**Pendekatan A - Pakai slot (SALAH):**

```blade
<x-button>primary</x-button>
<!-- lalu di komponen somehow parse teksnya -->
```

Ini **salah besar** karena:

- Variant adalah **konfigurasi**, bukan konten.
- Tidak bisa divalidasi dengan mudah.
- Tidak natural.

**Pendekatan B - Pakai props (BENAR):**

```blade
<x-button variant="primary">Simpan Produk</x-button>
```

Ini **benar** karena:

- Variant adalah konfigurasi (data terstruktur).
- Bisa divalidasi (seperti di tahap 3).
- Bisa diberi nilai default.

---

## 8. Default Slot: Apa Kalau Slot Kosong?

 Kadang kamu lupa mengisi slot, atau sengaja dikosongkan. Apa yang terjadi?

### 8.1 Slot kosong = tombol kosong

```blade
<x-button variant="primary"></x-button>
```

Hasilnya:

```html
<button class="btn btn-primary"></button>
```

Tombol tampil **tanpa teks**. Tidak berguna dan membingungkan.

### 8.2 Solusi: Beri teks default di komponen

Kita bisa modifikasi komponen supaya kalau slot kosong, tampilkan **teks default**:

```blade
@props(['variant' => 'primary'])

<button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
    {{ $slot->default('Klik di sini') }}
</button>
```

Perhatikan baris `{{ $slot->default('Klik di sini') }}`:

- `->default('...')` adalah method yang menentukan **teks default** kalau slot kosong.
- Jika pemanggil **mengisi slot** dengan teks, teks itu yang dipakai.
- Jika pemanggil **tidak mengisi slot**, teks default `"Klik di sini"` yang dipakai.

### 8.3 Cara penggunaan default slot

**Pemanggil 1 - Isi slot:**

```blade
<x-button variant="primary">Simpan Produk</x-button>
```

Hasil: tombol bertuliskan **"Simpan Produk"**.

**Pemanggil 2 - Slot kosong:**

```blade
<x-button variant="primary"></x-button>
```

Hasil: tombol bertuliskan **"Klik di sini"** (default).

**Pemanggil 3 - Self-closing (slot kosong):**

```blade
<x-button variant="primary" />
```

Hasil: tombol bertuliskan **"Klik di sini"** (default).

> **Pesan mentor:**
> Default slot itu **opsional**. Tapi sangat berguna untuk mencegah tombol kosong yang membingungkan. Pilih teks default yang **netral** dan **umum**, seperti "Klik di sini" atau "Tombol".

---

## 9. Komponen Button dengan Ikon Bawaan (Opsional)

Ini ide menarik: bagaimana kalau komponen Button **otomatis** menampilkan ikon berdasarkan variant?

Misalnya:

- `variant="primary"` → otomatis muncul ikon 💾 sebelum teks
- `variant="danger"` → otomatis muncul ikon 🗑️ sebelum teks
- `variant="warning"` → otomatis muncul ikon ✏️ sebelum teks

### 9.1 Modifikasi komponen `button.blade.php`

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

    // Ikon default per variant
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

### 9.2 Penjelasan kode baru

#### `$ikonPerVariant = [...]`

Ini array yang memetakan **setiap variant ke ikon default**:

- `primary` → `💾`
- `secondary` → `↩️`
- `success` → `✅`
- `danger` → `🗑️`
- `warning` → `✏️`
- `info` → `👁️`

#### `$ikon = $ikonPerVariant[$variant] ?? ''`

Ini mengambil ikon dari array berdasarkan variant yang dipilih. Operator `??` adalah **null coalescing**, artinya: "ambil nilai dari array, kalau tidak ada, pakai string kosong".

#### `@if ($ikon) ... @endif`

Jika ikon ada (tidak kosong), tampilkan ikon di dalam `<span class="me-1">`. Class `me-1` adalah class Bootstrap untuk memberi **margin kanan** (margin-end) sebesar 1 unit, supaya ikon tidak menempel ke teks.

#### `{{ $slot }}`

Setelah ikon, tampilkan isi slot (teks tombol dari pemanggil).

### 9.3 Hasil pemakaian

Sekarang setiap kali kamu menulis:

```blade
<x-button variant="primary">Simpan Produk</x-button>
```

Yang tampil di tombol adalah:

```
💾 Simpan Produk
```

Ikon **otomatis** muncul berdasarkan variant, tanpa kamu harus manual nulis ikon di setiap pemanggilan.

```blade
<x-button variant="danger">Hapus Produk</x-button>
```

Yang tampil:

```
🗑️ Hapus Produk
```

> **Pesan mentor:**
> Ini contoh bagus bagaimana komponen bisa **menyembunyikan kompleksitas**. Pemanggil cukup sebut variant, dan komponen yang urus ikonnya. Tapi ingat: ini **opsional**. Kalau kamu mau tombol tanpa ikon, jangan tambahkan fitur ini.

---

## 10. Slot untuk HTML Kompleks

Slot tidak terbatas pada teks dan ikon. Kamu bisa masukkan **HTML kompleks** bahkan **komponen lain** ke dalam slot.

### 10.1 Contoh: Tombol dengan badge

Bayangkan kamu mau tombol "Keranjang" dengan badge jumlah item:

```blade
<x-button variant="primary">
    🛒 Keranjang
    <span class="badge bg-danger ms-1">3</span>
</x-button>
```

Hasilnya:

```html
<button class="btn btn-primary">
    🛒 Keranjang
    <span class="badge bg-danger ms-1">3</span>
</button>
```

Tombol akan tampil dengan badge merah bertuliskan "3" di sebelah teks "Keranjang".

### 10.2 Contoh: Tombol dengan struktur kompleks

```blade
<x-button variant="success">
    <i class="fas fa-check"></i>
    <span class="ms-1">Konfirmasi Pesanan</span>
    <small class="ms-2 text-muted">(Gratis ongkir)</small>
</x-button>
```

Semua HTML di atas akan masuk ke `$slot` dan ditampilkan apa adanya di dalam tombol.

> **Pesan mentor:**
> Inilah kekuatan slot: kamu bisa masukkan **apa saja** ke dalam tombol tanpa harus mengubah komponennya. Komponen tetap sederhana, fleksibilitas ada di tangan pemanggil.

---

## 11. Diagram: Cara Kerja Slot

```
Pemanggil:
┌───────────────────────────────────────────┐
│ <x-button variant="primary">              │
│     <i class="fas fa-save"></i> Simpan    │  ← ini yang masuk ke $slot
│ </x-button>                               │
└─────────────────────┬─────────────────────┘
                      │
                      │  Blade mengambil isi
                      ▼
Komponen: components/button.blade.php
┌───────────────────────────────────────────┐
│ @props(['variant' => 'primary'])          │
│                                           │
│ <button {{ $attributes->merge([...]) }}>  │
│     {{ $slot }}                ←─ disisipkan di sini
│ </button>                                 │
└───────────────────────────────────────────┘
                      │
                      │  Hasil render
                      ▼
Output HTML:
┌───────────────────────────────────────────┐
│ <button class="btn btn-primary">          │
│     <i class="fas fa-save"></i> Simpan    │
│ </button>                                 │
└───────────────────────────────────────────┘
```

---

## 12. Troubleshooting

### Error 1: Tombol tampil tanpa teks

**Penyebab:**

- Menulis komponen sebagai **self-closing**: `<x-button variant="primary" />`.
- Atau menulis tag penutup tanpa isi: `<x-button variant="primary"></x-button>`.

**Solusi:**

Selalu isi slot dengan teks:

```blade
<x-button variant="primary">Simpan Produk</x-button>
```

Atau tambahkan default slot di komponen:

```blade
{{ $slot->default('Klik di sini') }}
```

### Error 2: Ikon tidak muncul

**Penyebab:**

- Font Awesome / Bootstrap Icons belum di-load di layout.
- Class ikon salah (mis: `fas fa-savee` bukan `fas fa-save`).

**Solusi:**

Cek di `layout/app.blade.php` apakah CSS ikon sudah di-load:

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
```

### Error 3: Ikon dan teks menempel

**Penyebab:**

- Tidak ada spasi antara ikon dan teks.
- Tidak ada margin di ikon.

**Solusi:**

Tambahkan class Bootstrap `me-1` (margin-end) atau `me-2` di ikon:

```blade
<x-button variant="primary">
    <i class="fas fa-save me-1"></i> Simpan Produk
</x-button>
```

Atau pakai `<span class="me-1">` pembungkus ikon.

### Error 4: HTML di slot tampil sebagai teks

**Penyebab:**

Kamu menggunakan kurung kurawal tunggal `{!! ... !!}` atau meng-escape HTML.

**Solusi:**

Pastikan pakai `{{ $slot }}` (kurung kurawal ganda), bukan `{!! $slot !!}`. Blade secara otomatis merender HTML di dalam `$slot` tanpa escaping.

> **Catatan:** Sebenarnya `$slot` adalah instance objek, bukan string biasa, jadi `{{ $slot }}` akan merender HTML dengan benar.

---

## 13. Latihan Mandiri

**Latihan C:**

Coba tambahkan **ikon** ke semua tombol di halaman `produk/coba-tombol.blade.php` (dari tahap 3). Pilih ikon yang sesuai untuk setiap variant.

<details>
<summary><strong>Lihat jawaban Latihan C</strong></summary>

```blade
@extends('layout.app', ['title' => 'Coba Tombol'])

@section('konten')
    <h2>Coba Semua Variant dengan Ikon</h2>

    <div style="display: flex; gap: 10px; flex-wrap: wrap; margin-top: 20px;">
        <x-button variant="primary">
            <i class="fas fa-save me-1"></i> Simpan
        </x-button>

        <x-button variant="secondary">
            <i class="fas fa-times me-1"></i> Batal
        </x-button>

        <x-button variant="success">
            <i class="fas fa-check me-1"></i> Selesai
        </x-button>

        <x-button variant="danger">
            <i class="fas fa-trash me-1"></i> Hapus
        </x-button>

        <x-button variant="warning">
            <i class="fas fa-edit me-1"></i> Edit
        </x-button>

        <x-button variant="info">
            <i class="fas fa-eye me-1"></i> Detail
        </x-button>
    </div>
@endsection
```

**Amati:**

- Semua tombol punya ikon di kiri teks.
- Ikon dipilih sesuai **peran** variant (save, times, check, trash, edit, eye).
- Semua tombol tetap konsisten karena pakai komponen yang sama.

</details>

---

## 14. Istilah Kunci Tahap Ini

| Istilah              | Arti sederhana                                            |
| -------------------- | --------------------------------------------------------- |
| **Slot**             | Lubang kosong di komponen yang bisa diisi konten apa saja |
| **`$slot`**          | Variabel otomatis berisi isi dari pemanggil               |
| **Default slot**     | Teks yang muncul kalau slot tidak diisi                   |
| **`$slot->default()`** | Method untuk menentukan teks default slot               |
| **Props**            | Input terstruktur (string, angka, boolean)                |
| **Self-closing**     | Tag yang ditutup sendiri: `<x-button />` (slot kosong)   |

---

## 15. Ringkasan Tahap 4

1. **Slot** = lubang kosong di komponen yang bisa diisi **konten bebas** (teks, HTML, ikon).
2. **`{{ $slot }}`** di komponen menandakan lokasi lubang.
3. **Isi slot** ditulis di antara tag pembuka dan penutup: `<x-button>isi</x-button>`.
4. **Props vs Slot:**
   - Props = formulir (data terstruktur: variant, type).
   - Slot = amplop surat (konten bebas: teks, HTML, ikon).
5. **Slot bisa berisi:** teks biasa, emoji, tag HTML, ikon Font Awesome, bahkan komponen lain.
6. **Default slot** dengan `$slot->default('...')` mencegah tombol kosong.
7. **Ikon bisa otomatis** berdasarkan variant (lewat array mapping di komponen).
8. **Self-closing tag** `<x-button />` menghasilkan slot kosong.

---

## 16. Cek Pemahaman

1. Apa itu slot dan apa bedanya dengan props?
2. Bagaimana cara menambahkan ikon di dalam tombol?
3. Apa yang terjadi kalau kamu menulis `<x-button variant="primary" />` (self-closing)?
4. Bagaimana cara memberi teks default kalau slot kosong?
5. Apa bisa slot berisi HTML? Berikan contoh.
6. Kenapa untuk teks tombol lebih baik pakai slot, bukan props?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. **Slot** = lubang untuk konten bebas (teks, HTML). **Props** = input terstruktur (variant, type). Slot ditulis di antara tag, props ditulis sebagai atribut.
2. Isi slot dengan tag ikon: `<x-button><i class="fas fa-save"></i> Simpan</x-button>`. Atau pakai emoji: `<x-button>💾 Simpan</x-button>`.
3. Tombol tampil **tanpa teks** (slot kosong), kecuali komponen punya default slot.
4. Di komponen, pakai `{{ $slot->default('Klik di sini') }}`. Kalau slot tidak diisi, teks default yang dipakai.
5. **Bisa**. Contoh: `<x-button><strong>Simpan</strong> Produk</x-button>` akan menghasilkan tombol dengan teks "Simpan" dicetak tebal.
6. Karena slot **fleksibel** (bisa diisi HTML, ikon, dll), sementara props hanya untuk **data terstruktur**. Teks tombol adalah konten, bukan konfigurasi.

</details>

---

## 17. Apakah Kamu Ingin Lanjut?

Di tahap 4 ini kamu sudah memahami **slot secara mendalam**:

- Apa itu slot dan bedanya dengan props
- Cara menambahkan ikon
- Default slot untuk cegah tombol kosong
- Slot untuk HTML kompleks

Sekarang kamu sudah punya **semua konsep dasar** komponen Button. Di tahap berikutnya, kita akan **menerapkan** komponen ini ke **semua halaman** CRUD Produk secara nyata:

> ### Pertanyaan:
>
> Apakah kamu ingin lanjut ke langkah berikutnya: **menerapkan `<x-button>` di semua halaman Produk**?
>
> Di tahap 5 kita akan:
>
> 1. Refactor halaman `index.blade.php` (daftar produk)
> 2. Refactor halaman `create.blade.php` (tambah produk)
> 3. Refactor halaman `edit.blade.php` (edit produk)
> 4. Refactor halaman `show.blade.php` (detail produk)
> 5. Melihat semua tombol konsisten di seluruh aplikasi
>
> Ketik **"lanjut"** untuk ke tahap 5,
> atau tanyakan jika ada bagian tahap 4 yang masih bingung.

---

> **Daftar Tahap (13. Komponen Button Blade):**
> - [x] Tahap 1 — Kenapa Tombol Harus Konsisten
> - [x] Tahap 2 — Membuat File Komponen `<x-button>` Sederhana
> - [x] Tahap 3 — Memahami Props dan Variant
> - [x] Tahap 4 — Slot dan Isi Tombol (kamu di sini)
> - [ ] Tahap 5 — Menerapkan `<x-button>` di Halaman Produk
> - [ ] Tahap 6 — Tombol Hapus dengan Form dan Konfirmasi
> - [ ] Tahap 7 — Variant Tambahan dan Tipe Link
> - [ ] Tahap 8 — Ringkasan dan Best Practice
