# Tahap 5 — Membuat View Admin dengan Badge Status & Tombol Aktifkan/Nonaktifkan

> Materi: Status Produk Aktif/Nonaktif
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi: Dasbor Admin yang Menampilkan Semua Kondisi Produk

Di tahap 4, kita sudah **memisahkan** halaman publik dan halaman admin di controller. Scope `active()` dipasang di pintu publik, dan pintu admin tampilkan semua produk.

Tapi ada satu masalah: **di halaman admin, admin tidak bisa tahu produk mana yang aktif dan mana yang nonaktif.** Semua produk tampil sama saja di tabel. Admin harus hafal id-nya, atau cek kolom `is_active` lewat database langsung. Ribet.

Bayangkan kamu **admin gudang** yang punya daftar semua barang di buku catatan. Kalau di catatan itu **tidak ada tanda** mana barang yang sudah siap (di rak depan) dan mana yang masih draft (di gudang belakang), kamu akan kewalahan. Setiap kali mau tahu status, kamu harus lari ke depan toko, lalu lari balik ke gudang, hanya untuk memastikan.

Solusinya: **di catatan admin, kasih tanda**. Misalnya:

- Baris dengan tanda centang hijau = barang sudah siap (aktif).
- Baris dengan tanda silang merah = barang masih draft (nonaktif).

Sekarang admin bisa lihat semua produk sekaligus, dan **sekilas** langsung tahu mana yang perlu dia tindak lanjuti.

Di Laravel, "tanda" itu kita wujudkan dengan:

1. **Badge status** di kolom tabel: hijau "Aktif", merah "Nonaktif".
2. **Tombol aksi**: "Aktifkan" atau "Nonaktifkan", sesuai status produk.

Di tahap 5 ini, kita fokus ke **tampilan** dulu. Tombolnya kita bikin, **tapi belum dihubungkan ke controller** (itu tahap 6). Sekarang, tujuannya adalah supaya admin **bisa melihat** status semua produk dalam satu halaman.

---

## 2. Rencana Halaman Admin: `/admin/produk`

Kita akan bikin halaman baru dengan URL:

```
/admin/produk
```

Halaman ini sudah punya route-nya dari tahap 4:

```php
Route::get('/admin/produk', [ProductController::class, 'adminIndex'])->name('admin.produk.index');
```

Yang belum kita bikin: **view-nya**. Controller `adminIndex()` memanggil `view('products.admin-index')`, jadi file view yang harus dibuat adalah:

```
resources/views/products/admin-index.blade.php
```

Tampilan halaman admin yang kita tuju:

| ID | Nama Produk | Harga | Stok | Kategori | Status | Aksi |
|---|---|---|---|---|---|---|
| 1 | Kopi Susu Vanilla | Rp 18.000 | 15 | Minuman | **[Aktif]** (badge hijau) | [Nonaktifkan] |
| 2 | Teh Manis Hangat | Rp 8.000 | 10 | Minuman | **[Aktif]** (badge hijau) | [Nonaktifkan] |
| 3 | Tumbler Limited | Rp 0 | 0 | Merchandise | **[Nonaktif]** (badge merah) | [Aktifkan] |
| 4 | Draft Produk Baru | Rp 0 | 0 | (kosong) | **[Nonaktif]** (badge merah) | [Aktifkan] |

Perhatikan:

- Kolom **Status** menampilkan badge warna kontras: hijau untuk aktif, merah untuk nonaktif.
- Kolom **Aksi** menampilkan tombol kebalikan: kalau produk aktif, tombolnya "Nonaktifkan". Kalau nonaktif, tombolnya "Aktifkan".

Logika tombol kebalikan ini penting: kalau produk sudah aktif, tidak perlu tombol "Aktifkan" lagi (sia-sia). Yang relevan adalah menonaktifkannya. Begitu juga sebaliknya.

---

## 3. Langkah 1: Buat File View Baru

Buka folder `resources/views/products/`. Buat file baru:

```
resources/views/products/admin-index.blade.php
```

Kita akan isi file ini secara **bertahap** per bagian supaya kamu paham tiap potongannya. Versi lengkap ada di bagian akhir.

### Catatan tentang Layout

Contoh ini pakai `layouts.app` (template induk) dan **Tailwind CSS** untuk styling. Projek kamu mungkin pakai Bootstrap atau CSS biasa, jadi sesuaikan class-nya. Konsep logikanya tetap sama.

---

## 4. Bagian A: Header Halaman

```blade
@extends('layouts.app')

@section('title', 'Kelola Produk (Admin)')

@section('content')
<div class="container mx-auto p-4">

    <h1 class="text-2xl font-bold mb-2">Kelola Produk (Admin)</h1>
    <p class="text-gray-600 mb-4">
        Daftar semua produk, baik yang aktif maupun yang masih dalam persiapan (nonaktif).
    </p>

    <div class="mb-4">
        <a href="{{ route('produk.index') }}" class="text-blue-600 underline">
            &larr; Lihat Halaman Publik
        </a>
    </div>
```

Penjelasan per baris:

| Baris | Fungsi |
|---|---|
| `@extends('layouts.app')` | Pakai layout induk projek (header, footer, dll). |
| `@section('title', ...)` | Set judul tab browser jadi "Kelola Produk (Admin)". |
| `@section('content')` | Awal blok konten utama. |
| `<h1>` | Judul halaman. |
| `<p>` | Deskripsi singkat supaya admin paham konteks halaman ini. |
| `<a href="{{ route('produk.index') }}">` | Link balik ke halaman publik `/produk`. Admin kadang mau preview tampilan customer. |
| `&larr;` | Karakter panah kiri (←) di HTML. |

---

## 5. Bagian B: Tabel Daftar Produk (dengan Badge & Tombol)

Ini bagian inti. Perhatikan baik-baik dua kolom baru: **Status** dan **Aksi**.

```blade
    @if($products->count() > 0)
        <table class="w-full border border-gray-300">
            <thead class="bg-gray-100">
                <tr>
                    <th class="border p-2 text-left">ID</th>
                    <th class="border p-2 text-left">Nama Produk</th>
                    <th class="border p-2 text-left">Harga</th>
                    <th class="border p-2 text-left">Stok</th>
                    <th class="border p-2 text-left">Kategori</th>
                    <th class="border p-2 text-left">Status</th>
                    <th class="border p-2 text-left">Aksi</th>
                </tr>
            </thead>
            <tbody>
                @foreach($products as $product)
                    <tr>
                        <td class="border p-2">{{ $product->id }}</td>
                        <td class="border p-2">{{ $product->nama }}</td>
                        <td class="border p-2">Rp {{ number_format($product->harga, 0, ',', '.') }}</td>
                        <td class="border p-2">{{ $product->stok }}</td>
                        <td class="border p-2">{{ $product->kategori }}</td>

                        {{-- Kolom Status: badge warna --}}
                        <td class="border p-2">
                            @if($product->is_active)
                                <span class="bg-green-100 text-green-800 text-xs px-2 py-1 rounded-full">
                                    Aktif
                                </span>
                            @else
                                <span class="bg-red-100 text-red-800 text-xs px-2 py-1 rounded-full">
                                    Nonaktif
                                </span>
                            @endif
                        </td>

                        {{-- Kolom Aksi: tombol kebalikan dari status --}}
                        <td class="border p-2">
                            @if($product->is_active)
                                <span class="text-gray-400 text-xs">(tombol nonaktifkan di tahap 6)</span>
                            @else
                                <span class="text-gray-400 text-xs">(tombol aktifkan di tahap 6)</span>
                            @endif
                        </td>
                    </tr>
                @endforeach
            </tbody>
        </table>
```

Penjelasan per bagian penting:

### `@if($products->count() > 0)`

Cek dulu apakah ada produk sama sekali. Kalau tabel kosong, kita tampilkan pesan ramah (di Bagian C).

### `@foreach($products as $product)`

Looping tiap produk di koleksi `$products` (yang dikirim controller `adminIndex`). Variabel `$product` di dalam loop adalah satu objek produk pada satu waktu.

### Format Harga: `number_format($product->harga, 0, ',', '.')`

Fungsi PHP untuk format angka ribuan. Penjelasan argumen:

| Argumen | Isi | Arti |
|---|---|---|
| Argumen 1 | `$product->harga` | Angka asli dari database, misalnya `18000` |
| Argumen 2 | `0` | Jumlah angka desimal (0 = tidak ada koma desimal) |
| Argumen 3 | `','` | Pemisah desimal (tidak dipakai karena argumen 2 = 0) |
| Argumen 4 | `'.'` | Pemisah ribuan: titik. Jadi `18000` jadi `18.000` |

Hasil akhir: "Rp 18.000". Lebih enak dibaca daripada "Rp 18000".

### Logika Badge Status

```blade
@if($product->is_active)
    <span class="bg-green-100 text-green-800 ...">Aktif</span>
@else
    <span class="bg-red-100 text-red-800 ...">Nonaktif</span>
@endif
```

Penjelasan:

- `$product->is_active` adalah akses kolom `is_active` di database. Karena tipe boolean, isinya `true` (1) atau `false` (0).
- `@if($product->is_active)` = "kalau produk ini aktif...".
- Tampilkan badge hijau dengan tulisan **"Aktif"**.
- `@else` = "...kalau tidak (nonaktif)...".
- Tampilkan badge merah dengan tulisan **"Nonaktif"**.

**Trik penting**: di Blade, `@if($product->is_active)` bekerja karena Laravel otomatis mengkonversi nilai `1` jadi `true` dan `0` jadi `false` saat dipakai di kondisi. Kamu tidak perlu nulis `@if($product->is_active === true)` atau `@if($product->is_active == 1)`. Cukup `@if($product->is_active)`.

### Kelas Tailwind untuk Badge

| Kelas | Arti |
|---|---|
| `bg-green-100` | Background hijau muda (kalau aktif) |
| `bg-red-100` | Background merah muda (kalau nonaktif) |
| `text-green-800` | Teks hijau tua |
| `text-red-800` | Teks merah tua |
| `text-xs` | Ukuran teks extra small |
| `px-2 py-1` | Padding horizontal 2, vertikal 1 |
| `rounded-full` | Sudut membulat penuh (bikin badge pill) |

Kombinasi kelas-kelas ini menghasilkan badge kecil berwarna yang langsung mencolok di mata admin. Kalau projekmu pakai Bootstrap, ganti dengan kelas seperti `badge badge-success` / `badge badge-danger`.

### Kolom Aksi: Placeholder

```blade
@if($product->is_active)
    <span class="text-gray-400 text-xs">(tombol nonaktifkan di tahap 6)</span>
@else
    <span class="text-gray-400 text-xs">(tombol aktifkan di tahap 6)</span>
@endif
```

Di tahap 5 ini, kolom aksi masih **placeholder** (teks abu-abu). Tujuannya: **memberi sinyal visual** apa yang akan muncul di tahap 6.

Di tahap 6 nanti, placeholder ini akan diganti dengan form yang sebenarnya:

```blade
<form action="/admin/produk/{{ $product->id }}/status" method="POST">
    @csrf
    @method('PATCH')
    <button type="submit">Nonaktifkan</button>
</form>
```

Tapi untuk sekarang, kita pakai placeholder supaya fokus ke **tampilan** dulu, sesuai janji di akhir tahap 4.

---

## 6. Bagian C: Pesan Kalau Tidak Ada Produk

```blade
    @else
        <div class="bg-yellow-100 border border-yellow-300 p-4 rounded">
            Belum ada produk. Tambahkan produk baru lewat halaman create.
        </div>
    @endif}
```

Penjelasan:

- `@else` berlaku kalau `$products->count()` == 0 (tidak ada produk sama sekali).
- Tampilkan pesan ramah berwarna kuning supaya admin tahu tidak ada error, hanya memang belum ada data.

---

## 7. File View Lengkap (Bisa Kamu Copy-Paste)

```blade
@extends('layouts.app')

@section('title', 'Kelola Produk (Admin)')

@section('content')
<div class="container mx-auto p-4">

    <h1 class="text-2xl font-bold mb-2">Kelola Produk (Admin)</h1>
    <p class="text-gray-600 mb-4">
        Daftar semua produk, baik yang aktif maupun yang masih dalam persiapan (nonaktif).
    </p>

    <div class="mb-4">
        <a href="{{ route('produk.index') }}" class="text-blue-600 underline">
            &larr; Lihat Halaman Publik
        </a>
    </div>

    @if($products->count() > 0)
        <table class="w-full border border-gray-300">
            <thead class="bg-gray-100">
                <tr>
                    <th class="border p-2 text-left">ID</th>
                    <th class="border p-2 text-left">Nama Produk</th>
                    <th class="border p-2 text-left">Harga</th>
                    <th class="border p-2 text-left">Stok</th>
                    <th class="border p-2 text-left">Kategori</th>
                    <th class="border p-2 text-left">Status</th>
                    <th class="border p-2 text-left">Aksi</th>
                </tr>
            </thead>
            <tbody>
                @foreach($products as $product)
                    <tr>
                        <td class="border p-2">{{ $product->id }}</td>
                        <td class="border p-2">{{ $product->nama }}</td>
                        <td class="border p-2">Rp {{ number_format($product->harga, 0, ',', '.') }}</td>
                        <td class="border p-2">{{ $product->stok }}</td>
                        <td class="border p-2">{{ $product->kategori }}</td>

                        <td class="border p-2">
                            @if($product->is_active)
                                <span class="bg-green-100 text-green-800 text-xs px-2 py-1 rounded-full">
                                    Aktif
                                </span>
                            @else
                                <span class="bg-red-100 text-red-800 text-xs px-2 py-1 rounded-full">
                                    Nonaktif
                                </span>
                            @endif
                        </td>

                        <td class="border p-2">
                            @if($product->is_active)
                                <span class="text-gray-400 text-xs">(tombol nonaktifkan di tahap 6)</span>
                            @else
                                <span class="text-gray-400 text-xs">(tombol aktifkan di tahap 6)</span>
                            @endif
                        </td>
                    </tr>
                @endforeach
            </tbody>
        </table>
    @else
        <div class="bg-yellow-100 border border-yellow-300 p-4 rounded">
            Belum ada produk. Tambahkan produk baru lewat halaman create.
        </div>
    @endif

</div>
@endsection
```

---

## 8. Bonus: Tambah Ringkasan Jumlah di Atas Tabel

Untuk membuat halaman admin lebih informatif, kita bisa tambahkan **ringkasan jumlah produk aktif dan nonaktif** di atas tabel. Ini opsional, tapi sangat membantu admin melihat situasi sekilas.

Taruh kode ini **di atas tabel**, di bawah link "Lihat Halaman Publik":

```blade
    <div class="flex gap-4 mb-4">
        <div class="bg-green-50 border border-green-200 px-4 py-2 rounded">
            <span class="text-green-700 font-bold">
                {{ $products->where('is_active', true)->count() }}
            </span>
            <span class="text-green-700 text-sm">Aktif</span>
        </div>

        <div class="bg-red-50 border border-red-200 px-4 py-2 rounded">
            <span class="text-red-700 font-bold">
                {{ $products->where('is_active', false)->count() }}
            </span>
            <span class="text-red-700 text-sm">Nonaktif</span>
        </div>
    </div>
```

Penjelasan:

- `$products->where('is_active', true)->count()` = filter koleksi `$products` di sisi PHP, ambil yang aktif, hitung jumlahnya.
- Ini **collection method**, bukan query database. Karena `$products` sudah di-`get()` di controller, kita bisa memfilter lagi di sisi PHP tanpa query tambahan.
- Hasilnya: dua kotak kecil hijau dan merah dengan angka. Admin langsung tahu "Oh, 2 produk aktif, 2 produk nonaktif."

Ini contoh pola yang sering dipakai di dashboard admin. Ringkasan dulu di atas, detail di bawah.

---

## 9. Uji Coba: Buka Halaman Admin

### Langkah A: Pastikan Ada Data Variatif

Cek di Tinker bahwa di tabel `products` kamu ada campuran produk aktif dan nonaktif:

```bash
php artisan tinker
```

```php
Product::select('id', 'nama', 'is_active')->get();
exit
```

Pastikan ada produk dengan `is_active = 1` dan `is_active = 0`. Kalau belum variatif, ubah dulu:

```php
Product::find(1)->update(['is_active' => true]);
Product::find(2)->update(['is_active' => true]);
Product::find(3)->update(['is_active' => false]);
exit
```

### Langkah B: Buka Halaman Admin

Akses URL:

```
http://localhost:8000/admin/produk
```

(Ganti port kalau dev server kamu pakai port lain.)

### Langkah C: Periksa Tampilan

Pastikan:

- Tabel muncul dengan kolom **Status** (badge hijau "Aktif" / merah "Nonaktif").
- Kolom **Aksi** menampilkan placeholder abu-abu sesuai status produk.
- Kalau kamu pakai bonus ringkasan, dua kotak hijau/merah muncul di atas tabel dengan angka yang sesuai.

### Langkah D: Bandingkan dengan Halaman Publik

Buka juga:

```
http://localhost:8000/produk
```

Bandingkan: halaman publik **hanya menampilkan produk aktif**. Halaman admin **menampilkan semua produk** dengan badge status yang jelas. Inilah perbedaan fungsi dua halaman yang kita rancang di tahap 4.

---

## 10. Hal-hal Penting yang Sering Bikin Pemula Bingung

### a. `@if($product->is_active)` Tanpa `== true`

Di Blade, kamu tidak perlu nulis:

```blade
@if($product->is_active == true)   // terlalu panjang
@if($product->is_active === 1)     // terlalu ketat, rawan masalah tipe data
```

Cukup:

```blade
@if($product->is_active)
```

Laravel otomatis tahu: kalau `is_active` bernilai `1` (atau `true`), kondisi benar. Kalau `0` (atau `false`), kondisi salah. Ini disebut **truthy/falsy check** di PHP, dan Blade mendukungnya.

### b. Kebalikan: `@unless` sebagai Alternatif

Kalau mau cek "jika produk tidak aktif", kamu bisa pakai:

```blade
@if(!$product->is_active)
```

Atau pakai directive `@unless` (yang lebih jelas dibaca):

```blade
@unless($product->is_active)
    ... produk nonaktif ...
@endunless
```

Dua-duanya sama. Pilih yang menurutmu lebih enak dibaca.

### c. Jangan Gunakan `==` untuk Membandingkan Boolean dengan Angka

Hindari:

```blade
@if($product->is_active == 1)   // bisa jadi sumber bug halus
```

Kenapa? Karena di PHP, `1 == true` dan `"1" == true` dan `"1" == 1` semua bernilai `true`. PHP sangat longgar dalam membandingkan tipe. Ini bisa menyebabkan bug yang sulit dilacak.

Lebih aman: pakai truthy check (`@if($product->is_active)`) atau pakai `===` (strict comparison).

### d. Badge Tidak Muncul / Salah Warna

Kalau badge tidak muncul sama sekali, kemungkinan:

1. Kamu lupa cek di Tinker bahwa kolom `is_active` benar-benar ada di database (mungkin migration belum dijalankan).
2. Ada typo di nama kolom (`is_Active` bukan `is_active`). Nama kolom di MySQL Linux **case-sensitive**, jadi hati-hati.

Kalau badge muncul tapi warnanya salah (misalnya selalu merah padahal produk aktif), kemungkinan:

1. Nilai di database bukan `1` tapi `"1"` (string). Walau PHP longgar, beberapa kasus bisa menyebabkan masalah. Pastikan migration pakai `$table->boolean('is_active')` (bukan `$table->string('is_active')`).

### e. Kalau Projek Tidak Pakai Tailwind

Contoh di file ini pakai Tailwind CSS. Kalau projekmu pakai Bootstrap, ganti kelas Tailwind dengan kelas Bootstrap. Misalnya:

| Tailwind | Bootstrap setara |
|---|---|
| `bg-green-100 text-green-800 text-xs px-2 py-1 rounded-full` | `badge badge-success` |
| `bg-red-100 text-red-800 text-xs px-2 py-1 rounded-full` | `badge badge-danger` |
| `text-gray-400 text-xs` | `text-muted small` |

Logika Blade-nya tetap sama, hanya kelas CSS yang beda.

### f. Looping `$products` Bisa Filter di Sisi PHP

Bonus di Bagian 8 (`$products->where('is_active', true)->count()`) memanfaatkan **collection method** di Laravel. Setelah `->get()` atau `->paginate()`, hasilnya adalah **Collection**, objek array serba bisa yang punya banyak method termasuk `where`, `count`, `sum`, `avg`, dll.

Ini berbeda dengan `where` di query builder: yang itu terjadi di SQL. Tapi setelah data keluar dari database, kita masih bisa memfilter lagi di sisi PHP.

Hanya saja: jangan berlebihan. Kalau bisa di-filter di SQL, lakukan di SQL (lebih hemat). Filter collection dipakai kalau data sudah tidak bisa dihindari untuk di-`get()` semua.

---

## 11. Apa yang Sudah Bisa Kamu Lakukan Setelah Tahap 5?

Setelah tahap 5, admin bisa:

1. Membuka halaman `/admin/produk`.
2. Melihat **semua produk** dalam satu tabel, lengkap dengan kolom Status.
3. **Sekilas melihat** mana produk aktif (badge hijau) dan mana yang nonaktif (badge merah).
4. Melihat ringkasan jumlah produk aktif dan nonaktif (kalau ikut bonus Bagian 8).
5. Klik link balik ke halaman publik untuk preview.

**Yang belum bisa admin lakukan**:

- **Mengubah status** produk dari halaman admin (tombol aktifkan/nonaktifkan masih placeholder abu-abu).

Itu yang akan kita kerjakan di tahap 6.

---

## Ringkasan Tahap 5

| Hal | Isi |
|---|---|
| Tujuan | Tampilan halaman admin dengan badge status dan placeholder tombol |
| File view baru | `resources/views/products/admin-index.blade.php` |
| Kolom baru di tabel | Status (badge), Aksi (placeholder) |
| Logika badge | `@if($product->is_active)` → badge hijau "Aktif", `@else` → badge merah "Nonaktif" |
| Format harga | `number_format($product->harga, 0, ',', '.')` |
| Bonus ringkasan | `$products->where('is_active', true)->count()` di collection |
| Styling | Pakai Tailwind class (sesuaikan kalau pakai Bootstrap) |
| Yang belum jalan | Tombol aksi masih placeholder (akan dihubungkan di tahap 6) |

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: menghubungkan tombol aktifkan/nonaktifkan ke route dan controller, supaya admin benar-benar bisa mengubah status produk?**

Kalau iya, tahap 6 kita akan:

1. Tambah route baru `Route::patch('/admin/produk/{id}/status', ...)`.
2. Tambah method `updateStatus()` di controller.
3. Ganti placeholder di view dengan form yang sebenarnya (`@method('PATCH')`).
4. Pakai `$product->update(['is_active' => !$product->is_active])` untuk toggle status.
5. Tes klik tombol, refresh halaman, lihat badge berubah warna.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
