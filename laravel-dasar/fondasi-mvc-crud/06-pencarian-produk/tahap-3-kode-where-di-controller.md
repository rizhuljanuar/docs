# Tahap 3 — Kode Pertama: `where('nama','like',...)` di Controller Produk

> Materi Laravel Dasar · Topik 6: Pencarian Produk
> Mentor: penjelasan bertahap untuk pemula total
> Prasyarat: sudah baca **Tahap 1** dan **Tahap 2**

---

## 1. Goal Tahap Ini

Di akhir Tahap 3 ini, kamu diharapkan:

- Melihat **bentuk Controller Produk** yang sudah ada (sebelum ada pencarian)
- Menambahkan **satu baris** `->where(...)` untuk menyaring produk
- Pahami **bagaimana kata kunci dari user** masuk ke Controller
- Pahami **method chaining** (`->...->...->...`) di Laravel
- Bisa **menguji** lewat URL (sebelum bikin form di Tahap 4)

**Belum bikin form HTML di tahap ini.** Kita uji lewat URL dulu supaya fokus ke Controller.

---

## 2. Asumsi Kondisi Awal

Sebelum mulai, ini asumsi kondisi project Laravel kamu (hasil dari materi sebelumnya: CRUD Produk):

### File: `app/Http/Controllers/ProdukController.php` (SEBELUM ada pencarian)

```php
<?php

namespace App\Http\Controllers;

use App\Models\Produk;
use Illuminate\Http\Request;

class ProdukController extends Controller
{
    public function index()
    {
        $produks = Produk::all();   // ambil SEMUA produk

        return view('produk.index', compact('produks'));
    }

    // ... method create(), store(), edit(), update(), destroy() ...
}
```

Penjelasan singkat tiap bagian:

| Bagian                          | Fungsi                                                    |
|---------------------------------|-----------------------------------------------------------|
| `use App\Models\Produk;`        | Import model `Produk` supaya bisa dipakai di controller.  |
| `use Illuminate\Http\Request;`  | Import class `Request` untuk membaca input dari user.     |
| `public function index()`       | Method yang dipanggil saat user buka halaman daftar produk. |
| `Produk::all()`                 | Ambil **semua** baris dari tabel `produk`.                |
| `$produks = ...`                | Simpan hasilnya ke variabel `$produks`.                   |
| `return view('produk.index', compact('produks'))` | Kirim variabel `$produks` ke view `produk/index.blade.php`. |

**Baris yang akan kita ubah hanya satu:** `Produk::all();`

---

## 3. Inti Perubahannya

Dari:

```php
$produks = Produk::all();
```

Menjadi:

```php
$produks = Produk::where('nama', 'like', '%' . $keyword . '%')->get();
```

Itu saja. **Satu baris.** Tapi banyak hal baru di dalamnya, jadi kita pecah pelan-pelan.

---

## 4. Langkah Kecil #1: Ambil Kata Kunci dari User

User akan mengetik kata kunci di kotak pencarian, misal `laptop`.
Kata kunci itu dikirim ke Controller lewat URL.

### Cara baca input user di Laravel

```php
$keyword = $request->input('search');
```

Penjelasan:

| Bagian              | Fungsi                                                            |
|---------------------|-------------------------------------------------------------------|
| `$request`          | Objek yang **menampung semua input** dari user (URL, form, dll).  |
| `->input('search')` | Ambil nilai input yang **namanya 'search'** dari URL/form.        |
| `$keyword = ...`    | Simpan nilainya ke variabel `$keyword`.                           |

Contoh: kalau URL-nya `produk?search=laptop`, maka `$keyword` berisi string `'laptop'`.

> Catatan: `'search'` di sini adalah **nama parameter** di URL. Bebas kamu kasih nama apa saja (`q`, `cari`, `keyword`). Yang penting konsisten dengan form di Tahap 4.

---

## 5. Langkah Kecil #2: Tambahkan `->where(...)`

Sekarang kita gabungkan dengan model `Produk`:

```php
Produk::where('nama', 'like', '%' . $keyword . '%')
```

Pecahan tiap bagian:

| Bagian            | Fungsi                                                             |
|-------------------|--------------------------------------------------------------------|
| `Produk::`        | Mulai query ke model `Produk` (mewakili tabel `produk`).           |
| `->where(...)`    | Tambahkan filter. Masih berupa **query builder**, belum dieksekusi. |
| `'nama'`          | Nama **kolom** yang diperiksa (kolom `nama` di tabel produk).      |
| `'like'`          | Operator: "mirip / mengandung sebagian".                           |
| `'%' . $keyword . '%'` | Gabungan: `%` + kata kunci user + `%`.                        |

### Kenapa ada titik (`.`)?

Di PHP, **titik = penggabung string** (concatenation).

```php
'%' . 'laptop' . '%'   →   '%laptop%'
```

Jadi kalau user ketik `laptop`, query yang terbentuk: `%laptop%` → sesuai konsep di Tahap 2.

---

## 6. Langkah Kecil #3: Tutup dengan `->get()`

```php
Produk::where('nama', 'like', '%' . $keyword . '%')->get();
```

Penjelasan `->get()`:

- Method `->where(...)` **belum benar-benar menjalankan query**. Dia hanya **menyusun** query (menyiapkan resep).
- `->get()` = "oke, **jalankan query-nya sekarang** dan berikan hasilnya ke saya."

**Analogi kasir restoran:**
- `->where(...)` = kamu menyusun pesanan di kertas ("nasi goreng, pedes, tanpa telur").
- `->get()`       = kamu kasih kertas itu ke kasir: *"tolong eksekusi!"*
- Hasilnya = makanan datang (data terambil).

### Yang sering bikin pemula bingung

```php
$produks = Produk::where('nama', 'like', '%' . $keyword . '%');
// ❌ Tanpa ->get()
// $produks masih berupa "query builder", BUKAN data produk.
// View akan error kalau di-foreach.
```

**Selalu tutup dengan `->get()`** untuk mengambil data. (Pengecualian: `->first()` untuk ambil 1 baris, tapi itu beda kasus.)

---

## 7. Method Chaining: Rantai Panah

Kode `Produk::...->...->get()` ini namanya **method chaining** (rantai method).

Bayangkan **sushi conveyor belt** (jalana sushi):

```
Produk::                  ← mulai dari sini (piring kosong)
  ->where('nama', ...)    ← tambahkan filter 1
  ->where('stok', '>', 0) ← tambahkan filter 2 (opsional, contoh)
  ->orderBy('nama')       ← urutkan berdasarkan nama
  ->get();                ← ambil hasilnya, selesai
```

Setiap panah `->` = "lalu lakukan ini juga". Hasilnya jadi **query yang makin spesifik**.

---

## 8. Kode Lengkap Method `index()` Setelah Pencarian

Sekarang kita rangkai semuanya:

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

        $produks = Produk::where('nama', 'like', '%' . $keyword . '%')->get();

        return view('produk.index', compact('produks'));
    }

    // ... method lainnya tetap sama ...
}
```

### Yang berubah dari versi sebelumnya:

1. `public function index(Request $request)` → sebelumnya tidak ada parameter `Request $request`. Sekarang kita **baca input user**, jadi butuh ini.
2. `$keyword = $request->input('search');` → ambil kata kunci dari URL.
3. `Produk::where(...)->get();` → ganti `Produk::all()`.

---

## 9. Uji Coba Lewat URL (Tanpa Form Dulu)

Karena kita belum bikin form HTML (Tahap 4), kita bisa uji langsung lewat URL.

Asumsi route kamu (di `routes/web.php`) seperti ini:

```php
Route::get('/produk', [ProdukController::class, 'index'])->name('produk.index');
```

Maka buka browser dan ketik:

| URL                                                | Hasil yang diharapkan                          |
|----------------------------------------------------|------------------------------------------------|
| `http://localhost:8000/produk`                     | **ERROR**: `$keyword` kosong, query jadi `%%`  |
| `http://localhost:8000/produk?search=laptop`       | Tampil semua produk yang namanya mengandung "laptop" |
| `http://localhost:8000/produk?search=Sepatu`       | Tampil produk "Sepatu Running Nike", dll.      |
| `http://localhost:8000/produk?search=Buku`         | Tampil produk "Buku Laravel Pemula", dll.      |

### ⚠️ Masalah saat user belum ketik apa-apa

URL pertama (`/produk` tanpa `?search=...`) akan bikin `$keyword = null`, lalu query jadi:

```php
Produk::where('nama', 'like', '%' . null . '%')->get();
// sama dengan:  Produk::where('nama', 'like', '%')->get();
```

`%%` sebenarnya cocok dengan **semua produk**, jadi mungkin **tidak error** di beberapa database. Tapi ini **tidak rapi** dan bisa jadi masalah di Tahap 5 nanti.

Kita akan perbaiki di **Tahap 5** (Query Scope) supaya rapi. Untuk sekarang, cukup pahami idenya dulu.

---

## 10. Contoh Visual Alur Data

```
USER BROWSER
   │
   │  buka: /produk?search=laptop
   │
   ▼
ROUTE WEB.PHP
   │  Route::get('/produk', [ProdukController::class, 'index'])
   │
   ▼
PRODUK CONTROLLER → index(Request $request)
   │
   │  1. $keyword = $request->input('search')   →  "laptop"
   │
   │  2. Produk::where('nama','like','%laptop%')->get()
   │     │
   │     ▼
   │   DATABASE MySQL
   │     SELECT * FROM produk WHERE nama LIKE '%laptop%'
   │     │
   │     ▼ kembalikan 2 baris (Laptop Asus, Laptop Acer)
   │
   │  3. $produks = [Produk{id:1}, Produk{id:2}]
   │
   │  4. return view('produk.index', compact('produks'))
   │
   ▼
VIEW produk/index.blade.php
   │  @foreach($produks as $p)
   │    <tr>{{ $p->nama }}</tr>
   │  @endforeach
   │
   ▼
USER BROWSER → lihat 2 produk di tabel
```

---

## 11. Kesimpulan Tahap 3

- Untuk menambah pencarian, **cukup ubah 1 baris** `Produk::all()` menjadi `Produk::where(...)->get()`.
- `$request->input('search')` untuk membaca kata kunci dari user.
- `->where('nama', 'like', '%' . $keyword . '%')` = filter berdasarkan kolom `nama`.
- `->get()` = eksekusi query dan ambil hasilnya. **Jangan lupa** `->get()`.
- Method chaining (`->...->...`) seperti sushi conveyor belt: tiap panah menambah kondisi.
- Saat ini belum ada form HTML. User uji lewat URL `?search=...`.
- Ada kelemahan: kalau `$keyword` kosong, query jadi `%%` (semua tampil). Akan diperbaiki di Tahap 5.

---

## Pertanyaan Berikutnya

**Apakah kamu ingin lanjut ke langkah berikutnya: membuat Form Pencarian di view Blade?**

Pada Tahap 4 kita akan:

- Bikin **kotak pencarian** (input text + tombol) di file `produk/index.blade.php`
- Pakai **`method="GET"`** di form (bukan POST) supaya kata kunci muncul di URL (penting untuk shareable link)
- Hubungkan form ke route `produk.index`
- Biarkan user cukup **ketik di kotak** dan tekan Enter, tidak perlu ketik URL manual lagi

— **Mentor Laravel**
