# Tahap 7 — Error Umum, Tips Performa, dan Rangkuman Akhir

> Materi Laravel Dasar · Topik 6: Pencarian Produk
> Mentor: penjelasan bertahap untuk pemula total
> Prasyarat: sudah baca **Tahap 1 sampai 6**

---

## 1. Goal Tahap Akhir Ini

Di akhir Tahap 7 ini, kamu diharapkan:

- Pahami **error umum** yang sering muncul saat bikin pencarian + cara mengatasinya
- Tahu **tips performa** untuk pencarian di tabel dengan ribuan baris
- Mengerti kapan harus pakai **pagination** untuk hasil pencarian
- Punya **rangkuman lengkap** dari seluruh materi Tahap 1-6
- Punya **checklist** siap dipakai saat bikin fitur pencarian di project nyata

---

## 2. Error Umum Saat Bikin Pencarian

### Error #1: "Call to a member function get() on null"

**Gejala:** Setelah panggil `Produk::search(...)->get()`, muncul error ini.

**Penyebab:** Di scope kamu **lupa `return $query`** saat kondisi kosong.

```php
// ❌ SALAH
public function scopeSearch($query, $keyword)
{
    if (!empty($keyword)) {
        return $query->where('nama', 'like', '%' . $keyword . '%');
    }
    // Tidak ada return di sini → return null implicitly
}
```

**Solusi:** Selalu `return $query` di semua cabang.

```php
// ✅ BENAR
public function scopeSearch($query, $keyword)
{
    if (!empty($keyword)) {
        return $query->where('nama', 'like', '%' . $keyword . '%');
    }
    return $query;
}
```

---

### Error #2: Hasil pencarian tidak sesuai harapan (semua produk muncul, atau tidak ada)

**Gejala:** User cari `laptop`, tapi semua produk tampil. Atau sebaliknya, tidak ada yang tampil.

**Penyebab #1:** Lupa tambahkan `%` di sekitar keyword.

```php
// ❌ SALAH
$query->where('nama', 'like', $keyword);
// Mencari yang namanya PERSIS "$keyword", bukan mengandung.
```

```php
// ✅ BENAR
$query->where('nama', 'like', '%' . $keyword . '%');
```

**Penyebab #2:** Nama parameter form **tidak cocok** dengan `$request->input(...)` di Controller.

```html
<!-- Form: name="q" -->
<input type="text" name="q">
```

```php
// Controller: ambil 'search' → tidak cocok, hasilnya null
$keyword = $request->input('search');
```

**Solusi:** Konsisten. Cek `name="..."` di form vs `->input('...')` di Controller.

---

### Error #3: Filter `orWhere` bocor ke filter lain (bug logika)

**Gejala:** Saat user cari `gaming` + kategori `Elektronik`, produk non-Elektronik juga muncul.

**Penyebab:** Sudah dijelaskan di Tahap 6. SQL mengevaluasi `AND` lebih dulu dari `OR`.

**Solusi:** Grup `orWhere` dengan closure:

```php
$query->where(function ($q) use ($keyword) {
    $q->where('nama', 'like', '%' . $keyword . '%')
      ->orWhere('deskripsi', 'like', '%' . $keyword . '%');
});
```

---

### Error #4: Kotak pencarian kosong setelah submit

**Gejala:** User ketik `laptop`, klik Cari, hasil tampil tapi **kotak kosong** kembali.

**Penyebab:** Lupa tambahkan `value="{{ request('search') }}"` di input.

**Solusi:**

```html
<input type="text" name="search" value="{{ request('search') }}" placeholder="Cari...">
```

---

### Error #5: SQL error "SQLSTATE syntax error" saat keyword ada tanda kutip

**Gejala:** User ketik `laptop's`, muncul error SQL.

**Penyebab:** Seharusnya **tidak terjadi** di Laravel karena query builder otomatis **escape** karakter berbahaya via PDO binding.

Kalau kamu melihat error ini, kemungkinan kamu **memakai string concatenation** untuk membangun SQL manual:

```php
// ❌ SALAH (rentan SQL injection)
DB::select("SELECT * FROM produk WHERE nama LIKE '%" . $keyword . "%'");
```

**Solusi:** Selalu pakai **query builder** atau **Eloquent**, JANGAN rangkai SQL manual.

```php
// ✅ BENAR (aman, otomatis di-escape)
Produk::where('nama', 'like', '%' . $keyword . '%')->get();
```

---

### Error #6: "Method search does not exist"

**Gejala:** Saat panggil `Produk::search(...)`, error method tidak ditemukan.

**Penyebab #1:** Lupa prefix `scope` di nama method Model.

```php
// ❌ SALAH
public function search($query, $keyword) { ... }
```

```php
// ✅ BENAR
public function scopeSearch($query, $keyword) { ... }
```

**Penyebab #2:** Method tidak ada di Model yang benar. Cek apakah kamu edit `app/Models/Produk.php`, bukan file lain.

---

## 3. Tips Performa untuk Pencarian

Saat tabel produk sudah **ribuan atau puluhan ribu baris**, pencarian bisa **lambat**. Berikut tipsnya:

### Tip #1: Tambahkan Index di Kolom yang Sering Dicari

Index = **daftar isi** untuk kolom. Membuat pencarian jauh lebih cepat.

**Migration:**

```php
Schema::table('produk', function (Blueprint $table) {
    $table->index('nama');
    $table->index('kategori');
});
```

**Analogi:** Mencari kata di buku 1000 halaman tanpa daftar isi = scan halaman demi halaman (lambat). Dengan daftar isi = langsung ke halaman yang dituju (cepat).

**Catatan:** Index bikin pencarian cepat, tapi **insert/update sedikit melambat** karena index juga harus di-update. Jadi index hanya di kolom yang sering dicari.

---

### Tip #2: Gunakan Pagination, Jangan Tampilkan Semua Hasil

Kalau hasil pencarian **masih ratusan baris**, jangan tampilkan semua. Pakai **pagination** (halaman).

**Controller:**

```php
// ❌ Ambil semua (boros memori)
$produks = Produk::search($keyword)->get();

// ✅ Ambil per halaman (misal 10 per halaman)
$produks = Produk::search($keyword)->paginate(10);
```

**View (Blade) - tambahkan link pagination:**

```html
<table>
    {{-- ... rows ... --}}
</table>

{{ $produks->links() }}
{{-- Ini otomatis bikin link "Halaman 1 2 3 ... 10" --}}
```

**Catatan:** Method `paginate()` mengembalikan objek berbeda dari `get()`. Kalau kamu pakai `paginate()`, jangan pakai `get()` juga.

---

### Tip #3: Batasi Jumlah Karakter Keyword

Jangan biarkan user ketik keyword sepanjang 10.000 karakter.

```php
$keyword = substr($request->input('search'), 0, 100);
// Ambil maksimal 100 karakter pertama
```

Atau tambahkan `maxlength` di HTML:

```html
<input type="text" name="search" maxlength="100" ...>
```

---

### Tip #4: Debounce di JavaScript (Opsional untuk Pemula)

Kalau kamu mau pencarian **realtime** (hasil muncul saat user mengetik, tanpa klik tombol), tambahkan **debounce** di JavaScript supaya tidak request ke server setiap huruf ditekan.

```html
<input type="text" name="search" id="search-input" ...>

<script>
let timeout;
document.getElementById('search-input').addEventListener('input', function() {
    clearTimeout(timeout);
    timeout = setTimeout(() => {
        this.form.submit();
    }, 500); // tunggu 500ms setelah user berhenti mengetik
});
</script>
```

**Catatan:** Ini **opsional**. Untuk pemula, form dengan tombol Cari sudah cukup. Debounce adalah topik intermediate.

---

## 4. Checklist Membangun Fitur Pencarian

Saat kamu bikin fitur pencarian di project nyata, ikuti urutan ini:

- [ ] **1. Tentukan kolom yang akan dicari** (misal: `nama`, `deskripsi`)
- [ ] **2. Tentukan filter tambahan** (misal: `kategori`, `harga`)
- [ ] **3. Bikin scope di Model** untuk tiap filter (`scopeSearch`, `scopeFilterByKategori`)
- [ ] **4. Tangani keyword kosong** di scope (`if (!empty(...))`)
- [ ] **5. Pakai closure grouping** kalau ada `orWhere`
- [ ] **6. Controller baca input** dari `$request->input(...)`, panggil scope berantai
- [ ] **7. Form di view** pakai `method="GET"`, `name="..."` konsisten
- [ ] **8. Tampilkan kembali keyword** dengan `value="{{ request('...') }}"`
- [ ] **9. Tombol Reset** untuk reset pencarian
- [ ] **10. Uji dengan berbagai skenario** (lihat daftar di Tahap 6 bagian 13)
- [ ] **11. Tambahkan pagination** kalau hasil bisa banyak
- [ ] **12. Tambahkan index** di kolom yang sering dicari

---

## 5. Rangkuman Akhir Materi Pencarian Produk

### Tahap 1: Konsep Pencarian

- Pencarian = fitur untuk **menyaring** daftar data supaya hanya yang cocok yang tampil.
- Wajib ada kalau data sudah banyak (puluhan, ratusan, ribuan baris).
- Tanpa pencarian, user susah menemukan barang, toko kehilangan pembeli.

### Tahap 2: Query, Filtering, `where`, `like`, `%`

- **Query** = permintaan ke database.
- **Filtering** = menyaring data dengan syarat (`WHERE`).
- **`=`** = sama persis. **`LIKE`** = mirip / mengandung sebagian.
- **`%`** = wildcard (teks apa pun di posisi itu).
- **`%laptop%`** = mengandung "laptop" di mana pun posisinya.

### Tahap 3: Kode `where` di Controller

- Cukup ubah **1 baris**: `Produk::all()` → `Produk::where('nama','like','%'.$kw.'%')->get()`.
- `$request->input('search')` untuk baca keyword dari URL.
- Selalu tutup dengan `->get()` untuk eksekusi query.

### Tahap 4: Form Pencarian di View

- Form pakai **`method="GET"`** supaya keyword muncul di URL (bookmark-able, share-able).
- `name="search"` di form **harus cocok** dengan `->input('search')` di Controller.
- `value="{{ request('search') }}"` supaya kotak tetap berisi keyword setelah submit.
- Tombol Reset pakai `<a href="{{ route('produk.index') }}">`.

### Tahap 5: Query Scope

- **Scope** = method di Model untuk merangkum query, supaya reusable.
- Konvensi: `scopeNama()` → dipanggil `->nama()`.
- Parameter `$query` otomatis dari Laravel.
- Controller jadi bersih: `Produk::search($kw)->get()`.
- Tangani keyword kosong dengan `if (!empty(...))`.

### Tahap 6: Pencarian Multi-Field

- `where()` = **AND**, `orWhere()` = **OR**.
- Cari di nama **ATAU** deskripsi: `where(...)->orWhere(...)`.
- Filter dropdown (kategori) pakai `where` biasa, bukan `like`.
- **Bahaya `orWhere`**: logika bisa bocor saat digabung filter lain.
- **Solusi**: grup dengan closure `where(function ($q) use ($kw) { ... })`.

### Tahap 7: Error Umum & Tips

- **Lupa `return $query`** → error "Call to member function on null".
- **Lupa `%`** → hasil tidak sesuai.
- **Nama form tidak cocok** dengan Controller → keyword null.
- **`orWhere` tanpa grup** → filter bocor.
- **Index kolom** untuk performa.
- **Pagination** untuk hasil banyak.

---

## 6. Kode Final Lengkap

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
                         ->paginate(10);

        return view('produk.index', compact('produks'));
    }

    // ... method create(), store(), edit(), update(), destroy() ...
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
               placeholder="Cari produk..."
               maxlength="100">

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

    {{-- Link pagination --}}
    {{ $produks->links() }}
@endsection
```

### `routes/web.php`

```php
Route::resource('produk', ProdukController::class);
```

---

## 7. Visualisasi Alur Lengkap

```
USER
  │
  │  Buka: /produk?search=laptop&kategori=Elektronik
  │
  ▼
ROUTES (web.php)
  │  Route::resource('produk', ...)
  │
  ▼
CONTROLLER (ProdukController@index)
  │
  │  $keyword  = $request->input('search')     → "laptop"
  │  $kategori = $request->input('kategori')   → "Elektronik"
  │
  │  $produks = Produk::search('laptop')
  │                     ->filterByKategori('Elektronik')
  │                     ->paginate(10);
  │            │
  │            ▼
  │       MODEL (Produk)
  │            │
  │            │  scopeSearch($query, 'laptop'):
  │            │    where(function ($q) {
  │            │      $q->where('nama', 'like', '%laptop%')
  │            │        ->orWhere('deskripsi', 'like', '%laptop%');
  │            │    })
  │            │
  │            │  scopeFilterByKategori($query, 'Elektronik'):
  │            │    where('kategori', 'Elektronik')
  │            │
  │            ▼
  │       DATABASE
  │            SELECT * FROM produk
  │            WHERE (nama LIKE '%laptop%' OR deskripsi LIKE '%laptop%')
  │              AND kategori = 'Elektronik'
  │            LIMIT 10 OFFSET 0;
  │            │
  │            ▼ kembalikan hasil (paginated)
  │
  ▼
VIEW (produk/index.blade.php)
  │  - Form pencarian dengan value dari request()
  │  - Tabel hasil pencarian
  │  - Link pagination
  │
  ▼
USER BROWSER → lihat hasil
```

---

## 8. Selamat!

Kamu telah menyelesaikan **materi Pencarian Produk** secara bertahap. Sekarang kamu bisa:

- Menjelaskan konsep pencarian ke orang lain
- Menulis query `where`, `like`, `orWhere`
- Bikin scope reusable di Model
- Menangani multi-field search dengan aman (closure grouping)
- Mengatasi error umum
- Menambahkan pagination dan index untuk performa

### Saran Latihan Berikutnya

Untuk memperdalam pemahaman, coba latihan ini:

1. **Tambahkan filter harga minimum** (misal: produk dengan harga di atas 100.000).
2. **Buat scope `scopeFilterByHarga($query, $min, $max)`** untuk range harga.
3. **Gabungkan 3 scope** dalam 1 query: `search + filterByKategori + filterByHarga`.
4. **Tambahkan sorting** (urutkan by harga ascending/descending) dengan parameter lain.
5. **Coba jalankan** di project Laravel sungguhan dan uji semua skenario.

---

## Penutup

> Selamat! kamu sudah lulus dari topik **6. Pencarian Produk** di Level Dasar Laravel.
>
> Topik berikutnya yang bisa kamu pelajari berikutnya:
> - **7. Pagination** (lebih dalam)
> - **8. Relasi Database** (One-to-Many: Produk → Kategori)
> - **9. Middleware dan Autentikasi**
>
> Tetap semangat belajar. Pencarian adalah salah satu fitur yang akan kamu bikin di **hampir setiap project Laravel**. Sekarang kamu punya bekal yang kuat.

— **Mentor Laravel**
