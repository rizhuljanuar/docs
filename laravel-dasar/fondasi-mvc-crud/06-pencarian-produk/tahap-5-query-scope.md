# Tahap 5 — Query Scope: Memisahkan Logic Pencarian ke Model

> Materi Laravel Dasar · Topik 6: Pencarian Produk
> Mentor: penjelasan bertahap untuk pemula total
> Prasyarat: sudah baca **Tahap 1, 2, 3, dan 4**

---

## 1. Goal Tahap Ini

Di akhir Tahap 5 ini, kamu diharapkan:

- Pahami **apa itu Query Scope** dan kenapa berguna
- Beda **local scope** vs **global scope** (kita fokus local scope)
- Bikin method `scopeSearch()` di Model Produk
- Refactor Controller supaya lebih **bersih** (hanya 1 baris pemanggilan)
- Mengatasi **masalah `$keyword` kosong** secara rapi
- Pahami konvensi penamaan `scopeNama()` → dipanggil `->nama()`

---

## 2. Masalah yang Ingin Kita Selesaikan

### Ingat Controller kita di Tahap 3?

```php
public function index(Request $request)
{
    $keyword = $request->input('search');

    $produks = Produk::where('nama', 'like', '%' . $keyword . '%')->get();

    return view('produk.index', compact('produks'));
}
```

Sekarang bayangkan requirements-nya **bertambah**:

- Pencarian harus cari berdasarkan **nama ATAU deskripsi**
- Harus bisa **filter berdasarkan kategori**
- Harus **hanya tampilkan produk yang stok-nya > 0**
- Logic pencarian ini mau dipakai di **3 tempat**: Controller Produk admin, Controller API mobile, dan Controller halaman publik

Kalau semua itu kita tulis langsung di Controller, begini jadinya:

```php
public function index(Request $request)
{
    $keyword = $request->input('search');
    $kategori = $request->input('kategori');

    $produks = Produk::where('nama', 'like', '%' . $keyword . '%')
        ->orWhere('deskripsi', 'like', '%' . $keyword . '%')
        ->where('kategori', $kategori)
        ->where('stok', '>', 0)
        ->orderBy('nama')
        ->get();

    return view('produk.index', compact('produks'));
}

public function apiList(Request $request)
{
    // ❌ Duplicate code! Sama persis dengan index()
    $keyword = $request->input('search');
    $kategori = $request->input('kategori');

    $produks = Produk::where('nama', 'like', '%' . $keyword . '%')
        ->orWhere('deskripsi', 'like', '%' . $keyword . '%')
        ->where('kategori', $kategori)
        ->where('stok', '>', 0)
        ->orderBy('nama')
        ->get();

    return response()->json($produks);
}
```

### 3 Masalah Utama

1. **Duplicate code** → logika pencarian ditulis berulang di setiap method.
2. **Sulit diubah** → kalau ada perubahan logic (misal: tambah kolom baru), kita harus ubah **di semua tempat**. Lupa satu → bug.
3. **Controller berantakan** → Controller harusnya hanya **mengatur alur**, bukan menulis query panjang.

---

## 3. Analogi: Resep di Buku Resep

Bayangkan kamu bekerja di restoran. Ada 3 menu yang semuanya pakai **bumbu dasar yang sama** (bawang, cabai, terasi).

### Cara Buruk

Tiap kali kamu masak, kamu hafal bumbu dasar itu **di kepala**, lalu takar dari awal.
Hasilnya:

- Bisa salah takar di menu kedua.
- Repot kalau mau ubah resep (harus otak-atik 3 tempat).

### Cara Baik

Kamu tulis **bumbu dasar** di buku resep sebagai **satu resep terpisah**.
Tiap masak menu, kamu cukup tulis: *"pakai 1 takar Bumbu Dasar (lihat halaman 5)"*.

Sekarang:

- Konsisten, takarannya selalu sama.
- Kalau mau ubah rasa bumbu dasar, ubah **1 tempat saja** (di buku resep).

**Query Scope itu = "Bumbu Dasar" di buku resep Laravel.**

Sebuah method terpisah di Model yang bisa **dipanggil berulang** dari mana saja.

---

## 4. Apa Itu Query Scope?

> **Query Scope** = method di Model yang **merangkum** satu query (atau rangkaian query) supaya bisa dipakai ulang dengan nama yang deskriptif.

Dalam Laravel, scope dibuat dengan **menambahkan prefix `scope`** di nama method.

### Contoh

Di Model `Produk`, kita bikin:

```php
public function scopeSearch($query, $keyword)
{
    return $query->where('nama', 'like', '%' . $keyword . '%');
}
```

Lalu di Controller atau di mana pun, kita panggil:

```php
Produk::search($keyword)->get();
//        ↑ panggil "search" (tanpa prefix "scope")
```

Laravel **otomatis** mengenali bahwa `scopeSearch()` di Model = method `search()` di query builder.

---

## 5. Local Scope vs Global Scope

Laravel punya 2 jenis scope. Sebagai pemula, kita **fokus ke local scope** dulu.

| Jenis          | Kapan dipakai                                      | Cara pakai                        |
|----------------|-----------------------------------------------------|-----------------------------------|
| **Local Scope**| Hanya saat **dipanggil eksplisit**                  | `Produk::search(...)->get()`      |
| **Global Scope**| Otomatis di **semua query**, tanpa dipanggil       | Lebih advanced, butuh `boot()`    |

**Contoh global scope (hanya contoh, JANGAN diikuti sekarang):**

```php
// Model Produk
protected static function booted()
{
    static::addGlobalScope('aktif', function ($query) {
        $query->where('is_aktif', 1);
    });
}

// Efeknya: setiap kali Produk::anything(), otomatis ditambah where is_aktif = 1
```

Global scope berguna untuk logic seperti **soft delete** atau **filter multi-tenant**, tapi **terlalu maju untuk sekarang**. Kita pakai local scope saja.

---

## 6. Konvensi Penamaan Scope

Ini kunci supaya tidak bingung:

```
Method di Model:   scopeSearch($query, $keyword)
                    ↑↑↑↑
                    prefix wajib "scope" + nama method

Dipanggil di query: ->search($keyword)
                     ↑↑↑↑
                     Hapus "scope", sisanya = nama method
```

### Aturan Praktis Penamaan

- Gunakan **kata kerja yang jelas**: `search`, `filterByKategori`, `aktif`, `murah`.
- Awali dengan `scope` di Model, **tanpa** `scope` saat memanggil.
- Gunakan **camelCase**: `scopeSearchByKategori()` → dipanggil `->searchByKategori()`.

---

## 7. Parameter `$query` — Apa Itu?

Coba lihat lagi:

```php
public function scopeSearch($query, $keyword)
{
    return $query->where('nama', 'like', '%' . $keyword . '%');
}
```

Parameter `$query` itu **otomatis diisi oleh Laravel**, kamu **tidak perlu** mengirimnya manual saat memanggil.

Saat kamu tulis:

```php
Produk::search('laptop')->get();
```

Yang terjadi di balik layar:

```php
// Laravel otomatis menyusun:
$query = Produk::query();              // mulai query builder
Produk::scopeSearch($query, 'laptop'); // panggil scope-mu, kirim $query
$query->get();                          // ambil hasilnya
```

Jadi `$query` = **query builder yang sedang berjalan**, dan kamu tinggal **menambahkan kondisi** kepadanya dengan `->where(...)`.

**Analogi:** `$query` itu seperti **mangkuk adonan** yang sudah disiapkan Laravel. Tugas kamu di scope = **menambah bahan** ke mangkuk itu (`->where(...)`), bukan bikin mangkuk baru.

---

## 8. Langkah Kecil #1: Bikin Scope `search()` di Model

Edit `app/Models/Produk.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Produk extends Model
{
    protected $fillable = [
        'nama', 'harga', 'stok', 'deskripsi', 'kategori', 'slug', 'gambar',
    ];

    /**
     * Scope: cari produk berdasarkan kata kunci di kolom nama.
     */
    public function scopeSearch($query, $keyword)
    {
        return $query->where('nama', 'like', '%' . $keyword . '%');
    }
}
```

Penjelasan:

| Bagian                          | Fungsi                                                          |
|---------------------------------|-----------------------------------------------------------------|
| `public function scopeSearch()` | Method scope dengan prefix `scope`.                              |
| `$query` (parameter 1)         | Query builder otomatis dari Laravel. Tidak dikirim manual.      |
| `$keyword` (parameter 2)       | Parameter yang **kamu** kirim saat memanggil.                   |
| `return $query->where(...)`     | Tambahkan kondisi filter ke `$query`, lalu kembalikan.          |

---

## 9. Langkah Kecil #2: Refactor Controller Pakai Scope

Sekarang Controller jadi **jauh lebih bersih**:

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

        $produks = Produk::search($keyword)->get();
        //                 ↑↑↑↑↑↑
        //                 panggil scope (tanpa prefix "scope")

        return view('produk.index', compact('produks'));
    }
}
```

### Sebelum vs Sesudah

```php
// ❌ SEBELUM (di Controller):
$produks = Produk::where('nama', 'like', '%' . $keyword . '%')->get();

// ✅ SESUDAH (di Controller):
$produks = Produk::search($keyword)->get();
```

Lebih **singkat**, lebih **jelas maksudnya** (baca: *"cari produk berdasarkan keyword"*), dan **reusable**.

---

## 10. Keuntungan Utama Pakai Scope

| Keuntungan           | Contoh                                                              |
|----------------------|---------------------------------------------------------------------|
| **Reusable**         | `Produk::search($kw)->get()` bisa dipanggil di Controller, Job, Artisan, dll. |
| **Mudah dibaca**     | `Produk::search('laptop')->aktif()->get()` = membaca seperti kalimat. |
| **Mudah diubah**     | Mau ubah logic? Cukup ubah **1 tempat** di Model.                   |
| **Testable**         | Scope bisa diuji terpisah tanpa Controller.                         |
| **Konsisten**        | Semua pemanggilan pencarian pakai logic yang sama.                  |

---

## 11. Langkah Kecil #3: Menangani `$keyword` Kosong

Masalah yang kita parkir di Tahap 3: kalau `$keyword` kosong (user belum ketik apa-apa), query jadi `%%` → semua produk tampil, tapi **secara tidak sengaja**, bukan karena disengaja.

### Solusi Rapi di Scope

Update scope:

```php
public function scopeSearch($query, $keyword)
{
    if (!empty($keyword)) {
        return $query->where('nama', 'like', '%' . $keyword . '%');
    }

    return $query;
}
```

Penjelasan:

| Bagian                       | Fungsi                                                            |
|------------------------------|-------------------------------------------------------------------|
| `if (!empty($keyword))`      | Cek apakah `$keyword` **tidak kosong** (bukan `''` atau `null`).  |
| `return $query->where(...)`  | Kalau ada keyword, tambahkan filter.                              |
| `return $query`              | Kalau kosong, kembalikan `$query` apa adanya (tanpa filter).      |

Sekarang:

- User buka `/produk` (tanpa `?search=...`) → `$keyword` = null → **tidak difilter** → semua produk tampil **dengan sengaja**.
- User ketik `laptop` → `$keyword` = `'laptop'` → difilter → hanya yang cocok.

### Kenapa ini lebih baik daripada `%%`?

| Skenario                | Sebelum (Tahap 3)        | Sesudah (Tahap 5)                  |
|-------------------------|---------------------------|-------------------------------------|
| Keyword kosong          | Query `LIKE '%%'` (samar) | Tidak ada filter (eksplisit)       |
| Keyword `null`          | Query `LIKE '%'` (error-prone) | Tidak ada filter (aman)        |
| Performa                | Database masih scan like | Database query lebih ringan        |
| Keterbacaan             | **Samr**, tidak jelas     | **Eksplisit**: "kalau kosong, lewati" |

---

## 12. Chaining Multiple Scope (Bonus Preview)

Salah satu kekuatan scope: bisa **dirantai** dengan scope lain.

Misal kamu bikin 2 scope:

```php
// Model Produk
public function scopeSearch($query, $keyword)
{
    if (!empty($keyword)) {
        return $query->where('nama', 'like', '%' . $keyword . '%');
    }
    return $query;
}

public function scopeAktif($query)
{
    return $query->where('stok', '>', 0);
}
```

Di Controller:

```php
$produks = Produk::search($keyword)->aktif()->orderBy('nama')->get();
```

Baca seperti kalimat:

> *"Ambil Produk, **cari** dengan keyword, yang **aktif** (stok > 0), urutkan by nama, lalu **ambil** hasilnya."*

**Ini akan kita perdalam di Tahap 6** (pencarian multi-field).

---

## 13. Diagram Alur dengan Scope

```
USER BROWSER
   │
   │  /produk?search=laptop
   ▼
PRODUK CONTROLLER
   │
   │  $keyword = $request->input('search')   →  "laptop"
   │
   │  Produk::search($keyword)->get();
   │           │
   │           ▼
   │     MODEL PRODUK
   │     method scopeSearch($query, "laptop")
   │           │
   │           ▼
   │       $query->where('nama', 'like', '%laptop%')
   │           │
   │           ▼
   │       DATABASE MySQL
   │       SELECT * FROM produk WHERE nama LIKE '%laptop%'
   │           │
   │           ▼
   │       kembalikan 2 baris
   │
   ▼
VIEW  →  tampilkan 2 produk
```

---

## 14. Kesalahan Umum Pemula di Scope

### A. Lupa prefix `scope`

```php
// ❌ SALAH
public function search($query, $keyword) { ... }

// Dipanggil:
Produk::search('laptop')->get();
// Error: Method search tidak dikenali sebagai scope.
```

```php
// ✅ BENAR
public function scopeSearch($query, $keyword) { ... }
```

### B. Lupa `return $query`

```php
// ❌ SALAH (tidak dikembalikan, chaining putus)
public function scopeSearch($query, $keyword)
{
    $query->where('nama', 'like', '%' . $keyword . '%');
}

// Dipanggil:
Produk::search('laptop')->aktif()->get();
// Error: ->aktif() dipanggil pada null.
```

```php
// ✅ BENAR
public function scopeSearch($query, $keyword)
{
    return $query->where('nama', 'like', '%' . $keyword . '%');
}
```

### C. Mengirim `$query` secara manual saat memanggil

```php
// ❌ SALAH
Produk::scopeSearch(Produk::query(), 'laptop')->get();
```

```php
// ✅ BENAR
Produk::search('laptop')->get();
// Laravel otomatis kirim $query untukmu.
```

### D. Nama scope yang ambigu

```php
// ❌ KURANG JELAS
public function scopeCari($query, $keyword) { ... }
public function scopeFilter($query, $keyword) { ... }
// Filter apa? Cari di mana?
```

```php
// ✅ JELAS
public function scopeSearchByNama($query, $keyword) { ... }
public function scopeFilterByKategori($query, $kategori) { ... }
```

---

## 15. Uji Coba

Lakukan tes yang sama seperti Tahap 4, sekarang dengan scope:

| # | Aksi                                            | Hasil Diharapkan                                     |
|---|-------------------------------------------------|------------------------------------------------------|
| 1 | Buka `/produk`                                  | Semua produk tampil (scope dilewati, keyword kosong). |
| 2 | Ketik `laptop`, klik Cari                       | Hanya produk mengandung "laptop".                    |
| 3 | Ketik kata tidak ada, misal `xyz`               | Tabel kosong.                                        |
| 4 | Kosongkan kotak, klik Cari                      | Semua produk tampil lagi (tidak error).              |
| 5 | Buka URL tanpa `?search=`, misal via route name | Semua produk tampil, tidak ada query like aneh.      |

Kalau semua berhasil, **scope kamu bekerja sempurna**.

---

## 16. Kesimpulan Tahap 5

- **Query Scope** = method di Model untuk merangkum query supaya **reusable**.
- Konvensi: method `scopeNama()` di Model → dipanggil `->nama()` di query.
- Parameter `$query` **otomatis diisi Laravel**, tidak perlu dikirim manual.
- Controller jadi **lebih bersih**: dari `Produk::where(...)->get()` jadi `Produk::search($keyword)->get()`.
- Scope bisa **dirantai** dengan scope lain (`->search()->aktif()->get()`).
- Gunakan scope untuk **menangani `$keyword` kosong** secara rapi (`if (!empty(...))`).
- **Local scope** cukup untuk pemula; **global scope** untuk kasus advanced (soft delete, multi-tenant).

Sekarang kode kamu sudah **rapi** dan **siap dipakai ulang**. Tapi pencarian kita masih **hanya di kolom `nama`**. Bagaimana kalau user cari kata yang ada di `deskripsi`? Itu topik Tahap 6.

---

## Pertanyaan Berikutnya

**Apakah kamu ingin lanjut ke langkah berikutnya: Pencarian Multi-Field (nama + deskripsi + kategori)?**

Pada Tahap 6 kita akan belajar:

- `orWhere()` untuk mencari di **kolom kedua**
- Pencarian gabungan nama + deskripsi dalam 1 scope
- Filter tambahan berdasarkan **kategori** (pakai scope kedua)
- Chaining 2 scope: `Produk::search($kw)->kategori($kat)->get()`

— **Mentor Laravel**
