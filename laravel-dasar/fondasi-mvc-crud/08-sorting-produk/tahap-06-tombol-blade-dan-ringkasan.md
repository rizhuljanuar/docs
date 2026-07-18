# Tahap 6 — Tombol Sorting di Blade + Gabung dengan Pagination + Ringkasan

> Materi: Sorting Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi Sehari-hari: Pasang Remote di Sofa

Di tahap 5, "remote TV" sudah jadi — controller sudah bisa mengurutkan produk sesuai parameter `sort` di URL. Tapi user masih harus **mengetik URL manual** seperti:

```
/produk?sort=harga-termurah
```

Itu seperti punya remote, tapi **remote-nya disimpan di gudang**. User harus bangkit, ambil remote, ketik manual. Tidak nyaman.

Tahap 6 ini: kita **pasang tombol-tombol sorting di halaman**, supaya user tinggal klik — remote tersedia di sofa.

---

## 2. Tujuan Tampilan

Di halaman `/produk`, kita tambahkan **deretan link / tombol**:

```
[ Terbaru ] [ Terlama ] [ Termurah ] [ Termahal ] [ Nama A-Z ] [ Nama Z-A ]
```

Saat user klik salah satu, URL berubah, dan halaman reload dengan urutan baru. Tombol yang **sedang aktif** kita **sorot** (mis. warna background beda) supaya user tahu posisinya.

Pagination di bawah juga harus **ikut bawa parameter `sort`**, supaya pindah halaman tidak kehilangan urutan.

---

## 3. Trik Kunci: `request()->fullUrlWithQuery()`

Laravel punya helper yang sangat berguna untuk membuat URL dengan query string:

```php
request()->fullUrlWithQuery(['sort' => 'harga-termurah'])
```

Apa fungsinya?

- Ambil URL saat ini (`/produk`, atau `/produk?sort=terbaru&page=2`, dsb.).
- **Set** parameter `sort` ke nilai baru (`'harga-termurah'`).
- **Jaga** parameter lain (mis. `page` — tapi ini akan otomatis di-reset).

Hasilnya: URL siap pakai untuk dijadikan `href` di link tombol.

Jadi kita tidak perlu **menulis URL manual** seperti `/produk?sort=harga-termurah`. Laravel yang susun.

> Ponytail: `fullUrlWithQuery()` menangani semua edge case URL (ada `?` / tidak ada, ada parameter lain / tidak). Lebih ringan daripada ngubin string manual.

---

## 4. Kode Blade: Deretan Tombol Sorting

Buka file `resources/views/produk/index.blade.php`. Di bagian atas (sebelum loop produk), tambahkan:

```blade
<div class="sorting">
    @php
        $opsiSort = [
            'terbaru'         => 'Terbaru',
            'terlama'         => 'Terlama',
            'harga-termurah'  => 'Termurah',
            'harga-termahal'  => 'Termahal',
            'nama-az'         => 'Nama A-Z',
            'nama-za'         => 'Nama Z-A',
        ];
        $sortAktif = request('sort') ?? 'terbaru';
    @endphp

    @foreach ($opsiSort as $nilai => $label)
        @php
            $url = request()->fullUrlWithQuery(['sort' => $nilai]);
            $aktif = ($sortAktif === $nilai) ? 'btn-aktif' : '';
        @endphp

        <a href="{{ $url }}" class="btn {{ $aktif }}">{{ $label }}</a>
    @endforeach
</div>
```

Mari kita bedah.

---

## 5. Bedah Kode Blade

### a. `$opsiSort` — daftar label yang tampil

```php
$opsiSort = [
    'terbaru'         => 'Terbaru',
    'terlama'         => 'Terlama',
    ...
];
```

- **Key** = nilai `sort` yang dikirim ke URL.
- **Value** = teks tombol yang dilihat user.

Catatan: label-label ini **harus cocok** dengan key di `$allowed` controller (tahap 5). Kalau tidak cocok, sorting tidak akan jalan.

> Ponytail: sebenarnya `$opsiSort` dan `$allowed` bisa dibuat satu sumber (mis. di `config/sorting.php` atau `View::share`). Tapi untuk skala belajar, duplikasi 6 baris ini masih sepadan. Refactor jadi config kalau opsi sudah tumbuh di banyak halaman.

### b. `$sortAktif` — opsi yang sedang dipilih

```php
$sortAktif = request('sort') ?? 'terbaru';
```

- Ambil `sort` dari URL. Kalau tidak ada, default `'terbaru'`.
- Ini dipakai untuk menentukan tombol mana yang **disorot**.

### c. Loop untuk bangun tombol

```blade
@foreach ($opsiSort as $nilai => $label)
    @php
        $url   = request()->fullUrlWithQuery(['sort' => $nilai]);
        $aktif = ($sortAktif === $nilai) ? 'btn-aktif' : '';
    @endphp

    <a href="{{ $url }}" class="btn {{ $aktif }}">{{ $label }}</a>
@endforeach
```

Penjelasan tiap baris di dalam loop:

- `$url = request()->fullUrlWithQuery(['sort' => $nilai]);`
  → URL baru dengan `sort` di-set ke `$nilai`. Mis. tombol "Termahal" → `/produk?sort=harga-termahal`.
- `$aktif = ($sortAktif === $nilai) ? 'btn-aktif' : '';`
  → Kalau opsi ini sedang aktif, kasih class CSS `btn-aktif`. Kalau tidak, kosong.
- `<a href="{{ $url }}" ...>`
  → Link HTML biasa, href-nya ke URL sorting.
- `class="btn {{ $aktif }}"`
  → Class `btn` selalu; class `btn-aktif` hanya kalau dipilih. CSS yang atur warnanya.

---

## 6. Contoh CSS Sederhana (Opsional)

Supaya tombol kelihatan beda saat aktif, tambahkan CSS ini (bisa di file CSS atau inline `<style>`):

```css
.btn {
    display: inline-block;
    padding: 6px 12px;
    margin-right: 6px;
    border: 1px solid #ddd;
    border-radius: 4px;
    text-decoration: none;
    color: #333;
    background: #f9f9f9;
}

.btn-aktif {
    background: #2563eb;   /* biru */
    color: #fff;
    border-color: #2563eb;
}
```

Saat user klik "Termahal", tombol itu berubah jadi biru putih. Tombol lain tetap abu-abu. User langsung tahu **sedang di opsi mana**.

---

## 7. Gabungkan dengan Pagination: `appends()`

Sekarang masalah penting. Kalau user sudah di `/produk?sort=harga-termurah` lalu klik **Halaman 2**, URL harus jadi:

```
/produk?sort=harga-termurah&page=2
```

Bukan:

```
/produk?page=2          ← sort hilang, urutan balik ke default
```

Laravel mempermudah ini dengan method `appends()` di link pagination. Di file Blade yang sama, di bagian bawah (setelah loop produk):

```blade
{{ $produk->appends(['sort' => request('sort')])->links() }}
```

Penjelasan:

- `$produk->links()` → tampilkan tombol pagination bawaan Laravel.
- `->appends(['sort' => request('sort')])` → **sisipkan** parameter `sort` ke setiap link pagination.
- Hasilnya: setiap link page membawa `sort` saat ini, jadi urutan **tidak hilang** saat pindah halaman.

> Ponytail: hanya `sort` yang perlu di-`appends`. Parameter lain yang sudah otomatis (`page`) ditangani Laravel sendiri.

---

## 8. File Blade Lengkap (Untuk Konteks)

Berikut kerangka `produk/index.blade.php` setelah sorting dipasang:

```blade
@extends('layouts.app')

@section('content')

    {{-- Tombol Sorting --}}
    <div class="sorting">
        @php
            $opsiSort = [
                'terbaru'         => 'Terbaru',
                'terlama'         => 'Terlama',
                'harga-termurah'  => 'Termurah',
                'harga-termahal'  => 'Termahal',
                'nama-az'         => 'Nama A-Z',
                'nama-za'         => 'Nama Z-A',
            ];
            $sortAktif = request('sort') ?? 'terbaru';
        @endphp

        @foreach ($opsiSort as $nilai => $label)
            @php
                $url   = request()->fullUrlWithQuery(['sort' => $nilai]);
                $aktif = ($sortAktif === $nilai) ? 'btn-aktif' : '';
            @endphp

            <a href="{{ $url }}" class="btn {{ $aktif }}">{{ $label }}</a>
        @endforeach
    </div>

    {{-- Daftar Produk --}}
    @foreach ($produk as $p)
        <div class="produk-card">
            <h3>{{ $p->nama }}</h3>
            <p>Rp {{ number_format($p->harga, 0, ',', '.') }}</p>
            <p>Kategori: {{ $p->kategori }}</p>
            {{-- ... field lain ... --}}
        </div>
    @endforeach

    {{-- Pagination (membawa sort saat pindah halaman) --}}
    {{ $produk->appends(['sort' => request('sort')])->links() }}

@endsection
```

---

## 9. Alur Lengkap Fitur (End-to-End)

Sekarang kamu sudah punya gambaran utuh. Ini alurnya saat user memakai fitur:

1. User buka `/produk`. Controller ambil `$sort = null` → default `terbaru`.
2. Controller kirim produk terbaru ke Blade. Tombol "Terbaru" disorot.
3. User klik tombol **Termahal**. URL jadi `/produk?sort=harga-termahal`.
4. Controller baca `$sort = 'harga-termahal'` → `$kolom='harga', $arah='desc'`.
5. Controller kirim produk urutan termahal ke Blade. Tombol "Termahal" disorot.
6. User klik **Halaman 2**. URL jadi `/produk?sort=harga-termahal&page=2` (berkat `appends`).
7. Controller tetap baca `sort=harga-termahal` → lanjut 10 produk termahal berikutnya.

Semua mulus, aman, dan rapi.

---

## 10. Ringkasan Seluruh Materi Sorting Produk

| Tahap | Topik | Inti |
|---|---|---|
| 1 | Konsep sorting | Kenapa butuh, analogi, ASC/DESC |
| 2 | Parameter `sort` + dynamic query | `request('sort')`, query yang berubah sesuai input |
| 3 | Validasi + whitelist | Bahaya input user, whitelist + map |
| 4 | Controller sebelum sorting | `latest()`, identifikasi baris yang diubah |
| 5 | Implementasi dynamic query | `orderBy($kolom, $arah)`, 11 baris di controller |
| 6 | Tombol Blade + pagination | `fullUrlWithQuery`, `appends`, highlight aktif |

### Inti yang Harus Kamu Ingat

1. **Sorting = orderBy dinamis**. Kolom dan arah berdasarkan input user.
2. **Jangan percaya input user**. Selalu validasi pakai **whitelist**.
3. **Whitelist + map = satu array**. Daftar sah sekaligus peta ke `[kolom, arah]`.
4. **`?? default`** menangani nilai kosong / tidak sah.
5. **`appends`** menjaga parameter `sort` saat pindah halaman pagination.
6. **`fullUrlWithQuery`** membuat URL tombol tanpa ngubin string manual.

### Bahasa Kunci

- **Whitelist**: daftar nilai yang diizinkan.
- **Dynamic query**: query yang bentuknya berubah sesuai input.
- **ASC / DESC**: arah urutan naik / turun.
- **Destructuring**: `[$a, $b] = $array` memecah array jadi variabel.
- **Query parameter**: pasangan `?key=value` di URL.

---

## Selamat!

Kamu sudah menyelesaikan materi **Sorting Produk** dari nol sampai bisa. Sekarang kamu bisa:

- Menjelaskan konsep sorting dan kenapa penting.
- Memahami parameter `sort` di URL.
- Membangun dynamic query yang **aman** dengan whitelist.
- Mengganti `latest()` statis dengan `orderBy($kolom, $arah)` dinamis.
- Membuat tombol sorting di Blade dengan highlight aktif.
- Menjaga urutan saat pindah halaman pagination.

### Latihan untuk Menguatkan Pemahaman

1. Tambah opsi sort baru: **stok-terbanyak** (`stok DESC`) dan **stok-tersedikit** (`stok ASC`).
2. Kombinasikan dengan fitur **pencarian** (materi 06): `/produk?cari=laptop&sort=harga-termurah`.
3. Ubah tombol link jadi `<form>` dengan `<select>` dropdown (lebih kompak di mobile).
4. Simpan opsi sort terakhir di **session**, supaya user tidak perlu pilih ulang saat kembali ke halaman.

---

## Pertanyaan Akhir untuk Kamu

> **Materi Sorting Produk sudah selesai. Apakah ada bagian yang ingin kamu perdalam lebih lanjut, atau kamu siap lanjut ke materi nomor 9 (Filter Produk by Kategori)?**

Kalau ada yang masih ragu, tanyakan. Kalau sudah siap, kita bisa lanjut ke topik berikutnya. Sampai jumpa di materi selanjutnya.
