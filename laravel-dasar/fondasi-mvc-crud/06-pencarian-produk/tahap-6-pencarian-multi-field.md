# Tahap 6 — Pencarian Multi-Field (Nama + Deskripsi + Kategori)

> Materi Laravel Dasar · Topik 6: Pencarian Produk
> Mentor: penjelasan bertahap untuk pemula total
> Prasyarat: sudah baca **Tahap 1 sampai 5**

---

## 1. Goal Tahap Ini

Di akhir Tahap 6 ini, kamu diharapkan:

- Pahami beda `where()` dan `orWhere()` di SQL/Laravel
- Memahami **logika AND vs OR** dengan analogi sederhana
- Memperluas scope `search()` supaya cari di **nama ATAU deskripsi**
- Bikin scope kedua: `scopeFilterByKategori()`
- Merantai 2 scope: `Produk::search($kw)->filterByKategori($kat)->get()`
- Pahami **bahaya gruping `orWhere`** dan cara mengatasinya pakai **closure grouping**

---

## 2. Masalah di Tahap Sebelumnya

Ingat scope kita di Tahap 5:

```php
public function scopeSearch($query, $keyword)
{
    if (!empty($keyword)) {
        return $query->where('nama', 'like', '%' . $keyword . '%');
    }
    return $query;
}
```

Sekarang bayangkan user mencari kata **"gaming"**.
Di database, produknya begini:

| id | nama                | deskripsi                             |
|----|---------------------|---------------------------------------|
| 1  | Laptop Asus ROG     | Laptop untuk pekerja kreatif          |
| 2  | Laptop Acer         | Laptop **gaming** murah, RAM 16GB     |
| 3  | Headset JBL         | Headset **gaming** nyaman             |

Saat user cari "gaming", yang **harusnya** muncul: produk #2 dan #3 (kata "gaming" ada di deskripsi).
Tapi karena scope kita **hanya cari di kolom `nama`**, hasilnya: **tidak ada yang cocok** (tidak ada nama produk yang mengandung "gaming").

User kecewa. Padahal produknya ada.

**Solutions:** Kita perlu cari di **kolom kedua** (`deskripsi`) juga.

---

## 3. Konsep: Logika AND vs OR

Sebelum nulis kode, pahami dulu logika dasarnya.

### Logika AND (`where` berturut-turut)

> "Saya mau produk yang **namanya ada kata laptop** DAN **harganya di bawah 10 juta**."

Produk harus memenuhi **dua syarat sekaligus**. Kalau salah satu tidak terpenuhi, **gugur**.

### Logika OR (`orWhere`)

> "Saya mau produk yang **namanya ada kata laptop** ATAU **deskripsinya ada kata laptop**."

Produk cukup memenuhi **salah satu** syarat saja. Muncul.

### Analogi: Pintu dan Gerbang

**AND** = dua pintu berurutan. Kamu harus lewat **keduanya**:
```
[Masuk] → [Pintu 1: Nama cocok] → [Pintu 2: Harga cocok] → [Keluar]
```
Kalau salah satu pintu **terkunci**, kamu tidak bisa lewat.

**OR** = dua pintu paralel. Kamu cukup lewat **salah satu**:
```
       ┌→ [Pintu A: Nama cocok]        ┐
[Masuk]│                                ├→ [Keluar]
       └→ [Pintu B: Deskripsi cocok]   ┘
```
Selama **salah satu** pintu terbuka, kamu bisa keluar.

---

## 4. `where()` vs `orWhere()` di Laravel

| Method       | Logika | Cara Baca                                |
|--------------|--------|-------------------------------------------|
| `->where()`  | AND    | "DAN syarat ini juga harus terpenuhi"     |
| `->orWhere()`| OR     | "ATAU syarat ini sebagai alternatif"      |

### Contoh 1: dua `where()` (AND)

```php
Produk::where('nama', 'like', '%laptop%')
      ->where('harga', '<', 10000000)
      ->get();
```

SQL jadinya:

```sql
SELECT * FROM produk
WHERE nama LIKE '%laptop%'
  AND harga < 10000000;
```

Artinya: produk yang namanya **mengandung "laptop"** DAN harganya **di bawah 10 juta**.

### Contoh 2: `where()` + `orWhere()` (OR)

```php
Produk::where('nama', 'like', '%laptop%')
      ->orWhere('deskripsi', 'like', '%laptop%')
      ->get();
```

SQL jadinya:

```sql
SELECT * FROM produk
WHERE nama LIKE '%laptop%'
   OR deskripsi LIKE '%laptop%';
```

Artinya: produk yang **namanya mengandung "laptop"** ATAU **deskripsinya mengandung "laptop"**.

---

## 5. Langkah Kecil #1: Update Scope `search()` dengan `orWhere`

Edit `app/Models/Produk.php`:

```php
public function scopeSearch($query, $keyword)
{
    if (!empty($keyword)) {
        return $query->where('nama', 'like', '%' . $keyword . '%')
                     ->orWhere('deskripsi', 'like', '%' . $keyword . '%');
    }
    return $query;
}
```

Yang ditambah: **satu baris** `->orWhere(...)`.

Sekarang saat user cari **"gaming"**:

| id | nama              | deskripsi                       | Cocok?  |
|----|-------------------|---------------------------------|---------|
| 1  | Laptop Asus ROG   | Laptop untuk pekerja kreatif    | ❌       |
| 2  | Laptop Acer       | Laptop **gaming** murah         | ✅ (deskripsi) |
| 3  | Headset JBL       | Headset **gaming** nyaman       | ✅ (deskripsi) |

Hasil: produk #2 dan #3 muncul. **Mantap.**

---

## 6. Langkah Kecil #2: Bikin Scope Kedua untuk Kategori

Sekarang kita bikin scope **terpisah** untuk filter kategori.

Di Model `Produk`:

```php
public function scopeFilterByKategori($query, $kategori)
{
    if (!empty($kategori)) {
        return $query->where('kategori', $kategori);
    }
    return $query;
}
```

Penjelasan:

| Bagian                           | Fungsi                                                    |
|----------------------------------|------------------------------------------------------------|
| `scopeFilterByKategori()`        | Method scope untuk filter berdasarkan kolom `kategori`.    |
| `$query`                         | Query builder otomatis dari Laravel.                       |
| `$kategori`                      | Parameter yang kamu kirim (misal `'Elektronik'`).          |
| `->where('kategori', $kategori)` | Filter: kolom `kategori` harus **sama persis** dengan `$kategori`. |
| `if (!empty($kategori))`         | Kalau `$kategori` kosong, lewati (tidak difilter).         |

### Kenapa kategori pakai `where`, bukan `like`?

Karena user **memilih** dari dropdown, nilainya **pasti persis** (misal `Elektronik`, `Fashion`, `Olahraga`).
Tidak perlu pakai `like` (cari sebagian).

Aturan praktis:

- Input **text bebas** (user ketik) → pakai **`like`**
- Input **dropdown/select** (user pilih dari opsi) → pakai **`where` = **

---

## 7. Langkah Kecil #3: Update Controller untuk Kirim 2 Input

Sekarang Controller baca **dua** input dari user:

```php
public function index(Request $request)
{
    $keyword = $request->input('search');
    $kategori = $request->input('kategori');

    $produks = Produk::search($keyword)
                     ->filterByKategori($kategori)
                     ->get();

    return view('produk.index', compact('produks'));
}
```

Baca seperti kalimat:

> *"Ambil Produk, **cari** dengan keyword (di nama/deskripsi), **filter berdasarkan kategori**, lalu ambil hasilnya."*

---

## 8. Langkah Kecil #4: Update Form di View

Tambahkan **dropdown kategori** di form pencarian (`resources/views/produk/index.blade.php`):

```html
<form action="{{ route('produk.index') }}" method="GET">
    @csrf

    <input type="text"
           name="search"
           value="{{ request('search') }}"
           placeholder="Cari produk...">

    <select name="kategori">
        <option value="">-- Semua Kategori --</option>
        <option value="Elektronik"  @selected(request('kategori') === 'Elektronik')>Elektronik</option>
        <option value="Fashion"     @selected(request('kategori') === 'Fashion')>Fashion</option>
        <option value="Olahraga"    @selected(request('kategori') === 'Olahraga')>Olahraga</option>
        <option value="Buku"        @selected(request('kategori') === 'Buku')>Buku</option>
    </select>

    <button type="submit">Cari</button>
    <a href="{{ route('produk.index') }}">Reset</a>
</form>
```

Penjelasan bagian baru:

| Bagian                                  | Fungsi                                                        |
|-----------------------------------------|---------------------------------------------------------------|
| `<select name="kategori">`              | Dropdown untuk pilih kategori.                                |
| `<option value="">-- Semua --</option>` | Opsi default: **tidak difilter** (value kosong).              |
| `value="Elektronik"` dst.               | Nilai yang dikirim kalau opsi ini dipilih.                    |
| `@selected(request('kategori') === '...')` | Blade directive: tambahkan `selected` kalau opsi ini sedang aktif. |

### Cara baca URL hasil

Setelah submit, URL jadi begini:

```
/produk?search=gaming&kategori=Elektronik
```

Artinya: cari produk yang **nama/deskripsi mengandung "gaming"** DAN **kategorinya Elektronik**.

---

## 9. Bahaya Tersembunyi `orWhere`

Sekarang saatnya **warning penting**. Ini sering bikin bug nyata.

### Skenario Bermasalah

User cari: `search=gaming`, `kategori=Elektronik`.

Controller kita jalanin:

```php
Produk::search('gaming')->filterByKategori('Elektronik')->get();
```

Yang terjadi di query builder:

```php
Produk::where('nama', 'like', '%gaming%')
      ->orWhere('deskripsi', 'like', '%gaming%')   // dari scopeSearch
      ->where('kategori', 'Elektronik')             // dari scopeFilterByKategori
      ->get();
```

SQL jadinya:

```sql
SELECT * FROM produk
WHERE nama LIKE '%gaming%'
   OR deskripsi LIKE '%gaming%'
  AND kategori = 'Elektronik';
```

### Tunggu, ini salah!

Di SQL, **`AND` diutamakan daripada `OR`** (seperti kali dibagi lebih kuat dari tambah/kurang di matematika).

SQL di atas sebenarnya dievaluasi sebagai:

```sql
WHERE nama LIKE '%gaming%'
   OR (deskripsi LIKE '%gaming%' AND kategori = 'Elektronik')
```

Artinya produk yang muncul:

- Yang namanya ada "gaming" → **semua kategori** (tidak difilter!)
- ATAU yang deskripsinya ada "gaming" DAN kategori Elektronik

**Ini bukan yang user mau.** User mau:

- (nama ada "gaming" **ATAU** deskripsi ada "gaming") **DAN** kategori Elektronik

---

## 10. Solusi: Grouping dengan Closure

Kita perlu **mengurung** bagian OR dalam tanda kurung, supaya dievaluasi sebagai **satu kesatuan**.

Dalam SQL:

```sql
WHERE (nama LIKE '%gaming%' OR deskripsi LIKE '%gaming%')
  AND kategori = 'Elektronik';
```

Dalam Laravel, pakai **closure** (fungsi anonim):

```php
$query->where(function ($q) use ($keyword) {
    $q->where('nama', 'like', '%' . $keyword . '%')
      ->orWhere('deskripsi', 'like', '%' . $keyword . '%');
});
```

### Penjelasan Closure

| Bagian                       | Fungsi                                                     |
|------------------------------|-------------------------------------------------------------|
| `where(function ($q) { ... })` | Bikin **grup** di dalam `WHERE`, semua kondisi di dalamnya diurung dengan kurung. |
| `$q`                         | Query builder **lokal** di dalam grup.                     |
| `use ($keyword)`             | Mengizinkan closure "melihat" variabel `$keyword` dari luar. |

### Analogi Kurung

Mirip matematika:

- `2 + 3 x 4` = 14 (perkalian dulu)
- `(2 + 3) x 4` = 20 (yang di kurung dulu)

Di SQL juga sama:

- `A OR B AND C` → `A OR (B AND C)`  (AND dulu)
- `(A OR B) AND C` → yang di kurung dulu

---

## 11. Update Scope dengan Closure

Update `scopeSearch()` di Model:

```php
public function scopeSearch($query, $keyword)
{
    if (!empty($keyword)) {
        return $query->where(function ($q) use ($keyword) {
            $q->where('nama', 'like', '%' . $keyword . '%')
              ->orWhere('deskripsi', 'like', '%' . $keyword . '%');
        });
    }
    return $query;
}
```

Sekarang query yang dihasilkan saat user cari `gaming` + `Elektronik`:

```sql
SELECT * FROM produk
WHERE (nama LIKE '%gaming%' OR deskripsi LIKE '%gaming%')
  AND kategori = 'Elektronik';
```

**Benar!** Inilah yang user mau.

---

## 12. Kode Lengkap Setelah Tahap 6

### `app/Models/Produk.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Produk extends Model
{
    protected $fillable = [
        'nama', 'harga', 'stok', 'deskripsi', 'kategori', 'slug', 'gambar',
    ];

    public function scopeSearch($query, $keyword)
    {
        if (!empty($keyword)) {
            return $query->where(function ($q) use ($keyword) {
                $q->where('nama', 'like', '%' . $keyword . '%')
                  ->orWhere('deskripsi', 'like', '%' . $keyword . '%');
            });
        }
        return $query;
    }

    public function scopeFilterByKategori($query, $kategori)
    {
        if (!empty($kategori)) {
            return $query->where('kategori', $kategori);
        }
        return $query;
    }
}
```

### `app/Http/Controllers/ProdukController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Produk;
use Illuminate\Http\Request;

class ProdukController extends Controller
{
    public function index(Request $request)
    {
        $keyword = $request->input('search');
        $kategori = $request->input('kategori');

        $produks = Produk::search($keyword)
                         ->filterByKategori($kategori)
                         ->get();

        return view('produk.index', compact('produks'));
    }

    // ... method lainnya ...
}
```

### `resources/views/produk/index.blade.php`

```html
@extends('layouts.app')

@section('content')
    <h1>Daftar Produk</h1>

    <form action="{{ route('produk.index') }}" method="GET">
        @csrf

        <input type="text"
               name="search"
               value="{{ request('search') }}"
               placeholder="Cari produk...">

        <select name="kategori">
            <option value="">-- Semua Kategori --</option>
            <option value="Elektronik"  @selected(request('kategori') === 'Elektronik')>Elektronik</option>
            <option value="Fashion"     @selected(request('kategori') === 'Fashion')>Fashion</option>
            <option value="Olahraga"    @selected(request('kategori') === 'Olahraga')>Olahraga</option>
            <option value="Buku"        @selected(request('kategori') === 'Buku')>Buku</option>
        </select>

        <button type="submit">Cari</button>
        <a href="{{ route('produk.index') }}">Reset</a>
    </form>
    <br>

    <table border="1">
        <thead>
            <tr>
                <th>Nama</th>
                <th>Harga</th>
                <th>Stok</th>
                <th>Kategori</th>
                <th>Aksi</th>
            </tr>
        </thead>
        <tbody>
            @foreach($produks as $produk)
                <tr>
                    <td>{{ $produk->nama }}</td>
                    <td>{{ $produk->harga }}</td>
                    <td>{{ $produk->stok }}</td>
                    <td>{{ $produk->kategori }}</td>
                    <td>
                        <a href="{{ route('produk.edit', $produk->id) }}">Edit</a>
                        <form action="{{ route('produk.destroy', $produk->id) }}"
                              method="POST" style="display:inline">
                            @csrf
                            @method('DELETE')
                            <button type="submit">Hapus</button>
                        </form>
                    </td>
                </tr>
            @endforeach
        </tbody>
    </table>
@endsection
```

---

## 13. Uji Coba Lengkap

| # | Input                                                       | Hasil Diharapkan                                       |
|---|-------------------------------------------------------------|--------------------------------------------------------|
| 1 | Tanpa input (buka `/produk`)                                | Semua produk tampil.                                   |
| 2 | `search=laptop`                                             | Semua produk yang nama/deskripsinya mengandung "laptop".|
| 3 | `search=gaming`                                             | Produk yang deskripsinya ada "gaming" muncul.          |
| 4 | `kategori=Elektronik` saja                                  | Hanya produk Elektronik.                               |
| 5 | `search=laptop` + `kategori=Elektronik`                     | Laptop yang kategori Elektronik saja.                  |
| 6 | `search=gaming` + `kategori=Elektronik`                     | Produk dengan kata "gaming" di nama/deskripsi, **DAN** Elektronik. |
| 7 | `search=xyz` (tidak ada)                                    | Tabel kosong.                                          |
| 8 | Klik Reset                                                  | Kembali ke semua produk.                               |

**Khusus tes #6:** ini adalah tes yang **akan gagal kalau kamu tidak pakai closure grouping** di scope. Coba hapus closure-nya, lalu lihat produk non-Elektronik juga akan muncul. Itu bug yang dijelaskan di bagian 9.

---

## 14. Kesimpulan Tahap 6

- `where()` = **AND** (dua syarat harus terpenuhi).
- `orWhere()` = **OR** (cukup salah satu syarat).
- Untuk pencarian multi-field (nama **ATAU** deskripsi), pakai `where()` + `orWhere()`.
- Untuk filter kategori (pilihan dropdown), pakai `where('kategori', $val)` biasa, bukan `like`.
- Pisahkan tiap filter ke **scope tersendiri** supaya reusable: `scopeSearch()`, `scopeFilterByKategori()`.
- **Bahaya `orWhere`**: saat digabung dengan `where` lain, SQL mengevaluasi `AND` dulu → bug logika.
- **Solusi**: grup `orWhere` dalam closure: `where(function ($q) use ($kw) { ... })`.
- Sekarang fitur pencarian kamu **fleksibel** (cari di banyak kolom + filter).

---

## Pertanyaan Berikutnya

**Apakah kamu ingin lanjut ke langkah terakhir (Tahap 7): pengujian akhir, error umum, dan rangkuman?**

Pada Tahap 7 kita akan:

- Daftar **error umum** yang sering muncul saat bikin pencarian + cara mengatasinya
- Tips **performa** untuk pencarian di tabel besar
- Best practice: **pagination**, **debounce** (JavaScript ringan), dan **index database**
- **Rangkuman akhir** seluruh materi Tahap 1-6

— **Mentor Laravel**
