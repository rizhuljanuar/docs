# Tahap 4 — Controller Produk Sebelum Sorting (Titik Mulai)

> Materi: Sorting Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi Sehari-hari: Sebelum Pasang Tombol Remote

Bayangkan kamu punya TV **tanpa remote**. Untuk ganti channel, kamu harus jalan ke TV, tekan tombol fisik di badan TV.

TV-nya jalan. Gambarnya bagus. Tapi **tiap kali mau ganti channel, kamu harus bangkit dari sofa**.

Lalu kamu beli remote. Sekarang, dari sofa, kamu bisa ganti channel tinggal tekan tombol.

Nah, di Laravel:

- **Controller tanpa sorting** = TV tanpa remote. Produk tetap tampil, tapi urutannya **ditentukan mati oleh kode**. User tidak bisa ubah.
- **Controller dengan sorting** = TV dengan remote. User tinggal klik, urutan berubah sesuai pilihannya.

Tahap 4 ini adalah: kita lihat dulu **"TV tanpa remote"** itu seperti apa. Supaya tahap 5 nanti, kamu paham **bagian mana yang kita ganti**.

---

## 2. Controller Produk yang Sudah Kamu Punya

Sejauh materi CRUD + pagination, controller kamu kemungkinan besar sudah seperti ini:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Produk;
use Illuminate\Http\Request;

class ProdukController extends Controller
{
    public function index()
    {
        $produk = Produk::latest()->paginate(10);

        return view('produk.index', compact('produk'));
    }

    // ... method lain: create, store, show, edit, update, destroy
}
```

Mari kita bedah bagian penting untuk sorting.

---

## 3. Bedah Baris Per Baris

### a. `public function index()`

Method `index` adalah method yang **menampilkan daftar produk**. Ini adalah pintu masuk halaman `/produk`.

Laravel akan memanggil method ini **setiap kali user membuka halaman daftar produk**.

### b. `$produk = Produk::latest()->paginate(10);`

Ini baris **kunci**. Mari kita pecah:

| Bagian | Arti |
|---|---|
| `Produk::` | Model `Produk` (yang mewakili tabel `produks` di database). |
| `latest()` | Shortcut Laravel: urutkan dari yang **terbaru** (`created_at DESC`). |
| `paginate(10)` | Ambil **10 produk per halaman**. |

Jadi baris ini bilang ke database:

> "Ambil produk, urutkan dari yang terbaru, bagi jadi halaman 10-10."

**Inilah "remote yang mati"**: urutan sudah ditulis mati sebagai `latest()`. User tidak bisa minta urutan lain. Semua user lihat hasil yang sama persis.

### c. `return view('produk.index', compact('produk'));`

Baris ini **mengirim variabel `$produk` ke file view** (tampilan Blade) bernama `produk.index`. Di view, data ini akan ditampilkan sebagai kartu-kartu produk.

Bagian ini **tidak perlu diubah** untuk sorting. Yang kita ubah hanya **cara kita ambil data**, bukan cara kita mengirimnya.

---

## 4. Apa Itu `latest()` Sebenarnya?

`latest()` itu adalah **jalan pintas** Laravel untuk:

```php
Produk::orderBy('created_at', 'desc')
```

Artinya:

- `orderBy('created_at', 'desc')` → urutkan kolom `created_at` secara **DESC** (dari baru ke lama).

Jadi `latest()` itu cuma **gaya penulisan singkat**. Di belakangnya, tetap `orderBy('created_at', 'desc')`.

Untuk keperluan sorting, kita akan **mengganti `latest()`** dengan bentuk eksplisit `orderBy(...)`, supaya kolom dan arahnya bisa **dinamis** (bisa diganti sesuai pilihan user).

> Ponytail: kalau kamu tidak butuh sorting dinamis, `latest()` sudah cukup dan lebih singkat. `orderBy(...)` eksplisit hanya dipakai kalau urutan bisa berubah-ubah sesuai input user.

---

## 5. Identifikasi: Apa yang Akan Kita Ubah?

Kita hanya ubah **satu baris**:

```php
// SEBELUM
$produk = Produk::latest()->paginate(10);
```

Menjadi **bentuk dinamis** (kerangka):

```php
// SESUDAH (kerangka, masih akan diisi tahap 5)
$produk = Produk::orderBy($kolom, $arah)->paginate(10);
```

Perhatikan:

- `Produk::` → tetap.
- `latest()` → diganti `orderBy($kolom, $arah)`. Kolom dan arah **datang dari whitelist** (yang sudah kita bangun di tahap 3).
- `paginate(10)` → tetap (kita ingin sorting dan pagination jalan bersamaan).

Itu saja. **Satu baris berubah.** Selebihnya, kode tahap 3 (whitelist) disisipkan di atasnya.

---

## 6. Bagaimana Sorting + Pagination Bekerja Bersama?

Pertanyaan bagus: "Kalau saya sortir berdasarkan harga termurah, lalu pindah ke halaman 2, apakah urutan tetap?"

**Jawabannya: iya, kalau kita lewatkan parameter `sort` ke link pagination.**

Bayangkannya begini:

- User buka: `/produk?sort=harga-termurah` → halaman 1 menampilkan 10 produk termurah.
- User klik "Halaman 2" → URL harus jadi `/produk?sort=harga-termurah&page=2` → lanjut 10 produk berikutnya, tetap urut termurah.

Kalau link pagination tidak membawa `sort`, halaman 2 akan kembali ke default (urutan terbaru). User bingung.

Untungnya, Laravel punya trik kecil untuk ini di Blade, yaitu method `appends()`. Kita akan pakai di tahap 6.

Untuk sekarang, cukup pahami: **sorting dan pagination bisa jalan bareng**, asalkan parameter `sort` dilewatkan ke setiap link pagination.

---

## 7. Apa yang Tidak Berubah?

Supaya kamu tenang, ini bagian yang **tetap sama**:

| Hal | Apakah berubah? |
|---|---|
| Model `Produk` | Tidak |
| Migration / struktur tabel | Tidak |
| Method lain (`create`, `store`, `edit`, dll.) | Tidak |
| Cara kirim data ke view (`compact`) | Tidak |
| `paginate(10)` | Tidak |
| File Blade utama (struktur) | Tidak, hanya tambah tombol sorting |

Jadi sorting itu **fitur yang sangat lokal**. Perubahannya kecil: satu baris query di controller + beberapa tombol di Blade.

---

## Ringkasan Tahap 4

| Hal | Isi |
|---|---|
| Titik mulai | `Produk::latest()->paginate(10)` |
| `latest()` | Shortcut untuk `orderBy('created_at', 'desc')` |
| Yang diubah | Satu baris: ganti `latest()` jadi `orderBy($kolom, $arah)` |
| `$kolom` & `$arah` | Datang dari whitelist (tahap 3) |
| Yang tetap | Model, migration, method lain, `paginate(10)`, struktur Blade |
| Catatan | Sorting + pagination bisa hidup berdampingan, tapi harus `appends()` `sort` ke link page |

---

## Kode Sejauh Ini (Sebelum Perubahan)

```php
public function index()
{
    $produk = Produk::latest()->paginate(10);

    return view('produk.index', compact('produk'));
}
```

Inilah **kerangka statis** yang akan kita hidupkan di tahap 5.

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: menerapkan dynamic query sorting di controller produk?**

Kalau iya, tahap 5 kita akan:

1. Gabungkan whitelist (tahap 3) ke method `index`.
2. Ambil `$kolom` dan `$arah` dari whitelist.
3. Ganti `latest()` jadi `orderBy($kolom, $arah)`.
4. Lihat hasil akhir controller dengan sorting dinamis.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
