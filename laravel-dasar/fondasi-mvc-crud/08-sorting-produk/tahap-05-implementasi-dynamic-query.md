# Tahap 5 — Implementasi Dynamic Query Sorting di Controller

> Materi: Sorting Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi Sehari-hari: Remote TV dengan 6 Tombol

Di tahap 4 kita bilang controller lama itu seperti **TV tanpa remote**. Sekarang kita pasang remotenya.

Remote-nya punya **6 tombol**:

- Terbaru
- Terlama
- Termurah
- Termahal
- Nama A-Z
- Nama Z-A

Di balik layar, tiap tombol punya **dua informasi** yang sudah dipetakan:

- tombol mana → **kolom** database mana.
- tombol mana → **arah** (ASC atau DESC).

Tombol-tombol itu **sudah ada di whitelist** (tahap 3). Sekarang kita **hubungkan remote ke TV**: whitelist → query.

---

## 2. Kode Akhir Controller (Yang Akan Kita Bangun)

Sebelum pecah pelan-pelan, ini hasil akhir method `index`:

```php
public function index()
{
    $sort = request('sort');

    $allowed = [
        'terbaru'         => ['created_at', 'desc'],
        'terlama'         => ['created_at', 'asc'],
        'harga-termurah'  => ['harga', 'asc'],
        'harga-termahal'  => ['harga', 'desc'],
        'nama-az'         => ['nama', 'asc'],
        'nama-za'         => ['nama', 'desc'],
    ];

    [$kolom, $arah] = $allowed[$sort] ?? $allowed['terbaru'];

    $produk = Produk::orderBy($kolom, $arah)->paginate(10);

    return view('produk.index', compact('produk'));
}
```

Total **11 baris**. Cukup itu untuk menghidupkan fitur sorting lengkap.

Sekarang kita bedah pelan-pelan.

---

## 3. Bedah Pelan-Pelan

### a. Ambil parameter dari URL

```php
$sort = request('sort');
```

- `request('sort')` → ambil nilai parameter `sort` dari URL.
- Hasilnya bisa: `'terbaru'`, `'nama-az'`, atau `null` kalau user tidak kirim.

Ini **jembatan pertama** antara user dan controller.

### b. Definisikan whitelist + map

```php
$allowed = [
    'terbaru'         => ['created_at', 'desc'],
    'terlama'         => ['created_at', 'asc'],
    'harga-termurah'  => ['harga', 'asc'],
    'harga-termahal'  => ['harga', 'desc'],
    'nama-az'         => ['nama', 'asc'],
    'nama-za'         => ['nama', 'desc'],
];
```

- Kiri (`'terbaru'`, dst.) = nilai `sort` yang **diizinkan**.
- Kanan (`['created_at', 'desc']`, dst.) = **artinya**: kolom database + arah.

Ingat tahap 3: array ini **sekaligus whitelist dan peta**. Inilah "remote" kita.

### c. Ambil kolom & arah, dengan default

```php
[$kolom, $arah] = $allowed[$sort] ?? $allowed['terbaru'];
```

Ini baris **paling penting**. Mari pecah:

- `$allowed[$sort]` → kalau `$sort` ada di whitelist, ambil pasangannya (mis. `['harga', 'asc']`).
- `?? $allowed['terbaru']` → kalau **tidak ada** (null), pakai default `terbaru` → `['created_at', 'desc']`.
- `[$kolom, $arah] = ...` → **destructuring**: array `[a, b]` dipecah jadi dua variabel `$kolom = a` dan `$arah = b`.

Setelah baris ini:

- `$kolom` berisi nama kolom (mis. `'harga'`).
- `$arah` berisi `'asc'` atau `'desc'`.

Dua variabel ini yang **dinamis** — bisa beda tiap request, tergantung user.

### d. Bangun query dengan `orderBy`

```php
$produk = Produk::orderBy($kolom, $arah)->paginate(10);
```

- `Produk::orderBy($kolom, $arah)` → urutkan produk berdasarkan kolom `$kolom`, arah `$arah`.
- `->paginate(10)` → ambil 10 per halaman.

Karena `$kolom` dan `$arah` **datang dari whitelist**, kita **pastikan aman**. User tidak bisa menyisipkan kolom liar, karena semua nilai sudah dipetakan.

> Ponytail: kami **tidak** memakai `when()` berantai atau closure panjang. Array lookup + destructuring sudah cukup. Tambah opsi sorting? Cukup sebaris di `$allowed`.

---

## 4. Skenario: Apa yang Terjadi Saat User Buka URL?

Mari kita uji dengan beberapa URL:

#### URL 1: `/produk?sort=harga-termurah`

1. `$sort = 'harga-termurah'`
2. Ada di `$allowed`? **Ya** → ambil `['harga', 'asc']`.
3. `$kolom = 'harga'`, `$arah = 'asc'`.
4. Query: `Produk::orderBy('harga', 'asc')->paginate(10)`.
5. Hasil: produk ditampilkan **dari termurah ke termahal**, 10 per halaman.

#### URL 2: `/produk?sort=nama-za`

1. `$sort = 'nama-za'`
2. Ada di `$allowed`? **Ya** → `['nama', 'desc']`.
3. `$kolom = 'nama'`, `$arah = 'desc'`.
4. Query: `Produk::orderBy('nama', 'desc')->paginate(10)`.
5. Hasil: produk **Z ke A**.

#### URL 3: `/produk` (tanpa `?sort=`)

1. `$sort = null`.
2. `$allowed[null]` tidak ada → pakai default `$allowed['terbaru']` → `['created_at', 'desc']`.
3. `$kolom = 'created_at'`, `$arah = 'desc'`.
4. Query: `Produk::orderBy('created_at', 'desc')->paginate(10)`.
5. Hasil: produk **terbaru dulu** (default aman).

#### URL 4: `/produk?sort=password` (nilai jahat)

1. `$sort = 'password'`.
2. Ada di `$allowed`? **Tidak** → pakai default.
3. Hasil: sama seperti URL 3 → **terbaru**.
4. **Tidak ada error, tidak ada bocor data.** Whitelist bekerja.

Inilah hasil kerja whitelist + map + destructuring.

---

## 5. Penjelasan: Kenapa Tidak Pakai `when()` atau `if` Berantai?

Mungkin kamu pernah lihat tutorial lain pakai pola begini:

```php
$produk = Produk::query();

if ($sort === 'terbaru') {
    $produk->latest();
} elseif ($sort === 'harga-termurah') {
    $produk->orderBy('harga', 'asc');
} elseif (...) {
    ...
}

$produk = $produk->paginate(10);
```

Ini **bisa jalan**, tapi:

- Lebih panjang.
- Repetitif.
- Mudah lupa satu cabang → bug.
- Mau tambah sort baru? Harus tambah `elseif` lagi.

Dengan **whitelist + map**, kita **menulis logika sekali**. Tambah opsi sort = tambah **satu baris di array**. Lebih kering, lebih mudah dirawat.

> Ponytail: satu sumber kebenaran (`$allowed`) lebih baik daripada banyak cabang `if`. Ubah = ubah array. Tidak ada logika tersebar.

---

## 6. File Controller Lengkap (Untuk Konteks)

Supaya kamu pahami posisinya, ini controller lengkap dengan method `index` baru:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Produk;
use Illuminate\Http\Request;

class ProdukController extends Controller
{
    public function index()
    {
        $sort = request('sort');

        $allowed = [
            'terbaru'         => ['created_at', 'desc'],
            'terlama'         => ['created_at', 'asc'],
            'harga-termurah'  => ['harga', 'asc'],
            'harga-termahal'  => ['harga', 'desc'],
            'nama-az'         => ['nama', 'asc'],
            'nama-za'         => ['nama', 'desc'],
        ];

        [$kolom, $arah] = $allowed[$sort] ?? $allowed['terbaru'];

        $produk = Produk::orderBy($kolom, $arah)->paginate(10);

        return view('produk.index', compact('produk'));
    }

    // method lain tetap seperti biasa: create, store, show, edit, update, destroy
}
```

Hanya method `index` yang **berubah**. Selebihnya tetap.

---

## 7. Test Manual (Tanpa Blade Dulu)

Sebelum membuat tombol di Blade (tahap 6), kamu sudah bisa **menguji** lewat browser. Pastikan ada beberapa data produk di database (paling tidak 3-5 produk dengan harga dan tanggal berbeda), lalu buka:

| URL yang kamu ketik di browser | Hasil yang harus terlihat |
|---|---|
| `http://localhost:8000/produk` | Produk urutan **terbaru** |
| `http://localhost:8000/produk?sort=terbaru` | Sama, **terbaru** |
| `http://localhost:8000/produk?sort=terlama` | **Terlama** |
| `http://localhost:8000/produk?sort=harga-termurah` | **Termurah** |
| `http://localhost:8000/produk?sort=harga-termahal` | **Termahal** |
| `http://localhost:8000/produk?sort=nama-az` | **A-Z** |
| `http://localhost:8000/produk?sort=nama-za` | **Z-A** |
| `http://localhost:8000/produk?sort=aneh` | Default (**terbaru**) |

Kalau semua hasil sesuai, **fitur sorting kamu sudah berfungsi**. Tinggal pasang tombol di view biar user tidak perlu mengetik URL manual.

---

## Ringkasan Tahap 5

| Hal | Isi |
|---|---|
| Total kode | 11 baris di method `index` |
| Inti | Whitelist + map + destructuring → `orderBy($kolom, $arah)` |
| `$sort` | `request('sort')` dari URL |
| `$allowed` | Daftar 6 opsi → pasangan `[kolom, arah]` |
| Default | `$allowed['terbaru']` (via `??`) |
| Hasil dinamis | `$kolom` & `$arah` berubah tiap request |
| Aman | User tidak bisa sisipkan kolom liar |
| Tambahan opsional | Tidak pakai `when()`/`elseif` — array lookup sudah cukup |

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: membuat tombol sorting di Blade, dan menggabungkannya dengan pagination?**

Kalau iya, tahap 6 kita akan:

1. Buat **dropdown / link** sorting di `produk/index.blade.php`.
2. Pakai **query string** dengan `request()->fullUrlWithQuery()`.
3. Pakai `appends()` supaya **link pagination ikut membawa `sort`**.
4. Tunjukkan **highlight tombol aktif** (mis. tombol "Termurah" disorot saat dipilih).
5. Ringkasan akhir seluruh materi sorting.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
