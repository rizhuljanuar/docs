# Tahap 3 — Ubah `Produk::all()` Menjadi `Produk::paginate(10)`

> Materi: Pagination Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Prasyarat: Selesai Tahap 2 (paham controller lama)

---

## 1. Tujuan Tahap Ini

Kita akan **mengubah satu kata** di controller produk:

```php
// Versi lama (ambil semua)
$produks = Produk::all();

// Versi baru (ambil 10 per halaman)
$produks = Produk::paginate(10);
```

Cuma ganti `all()` → `paginate(10)`. Tapi efeknya besar. Di tahap ini kita pahami **apa yang sebenarnya terjadi** di belakang layar.

---

## 2. Controller Produk Setelah Diubah

Ini tampilan lengkap controller setelah perubahan:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Produk;
use Illuminate\Http\Request;

class ProdukController extends Controller
{
    // Menampilkan daftar produk
    public function index()
    {
        // Ambil 10 produk per halaman
        $produks = Produk::paginate(10);

        // Kirim data produk ke halaman tampilan
        return view('produk.index', compact('produks'));
    }
}
```

Perhatikan: hanya baris ke-12 yang berubah. Sisanya tetap sama.

---

## 3. Arti `Produk::paginate(10)` Per Kata

```php
$produks = Produk::paginate(10);
```

| Bagian | Arti |
|---|---|
| `$produks` | Variabel (kotak) untuk menyimpan data |
| `Produk` | Model produk (mewakili tabel `produks`) |
| `::paginate(10)` | Method Laravel: ambil data **10 per halaman** |
| Angka `10` | Jumlah produk yang ditampilkan per halaman |

Satu kalimatnya:

> "Ambil produk dari database, tampilkan 10 produk per halaman."

Angka `10` bisa kamu ubah sesuka hati:
- `paginate(5)` → 5 produk per halaman.
- `paginate(20)` → 20 produk per halaman.
- `paginate(10)` → yang umum dipakai.

---

## 4. Apa yang Terjadi di Belakang Layar?

Saat kamu pakai `paginate(10)`, Laravel sebenarnya melakukan banyak hal otomatis:

### a. Laravel Cek Halaman Saat Ini dari URL

Kalau user membuka:

```
http://localhost:8000/produk         --> Laravel tahu: halaman 1
http://localhost:8000/produk?page=2  --> Laravel tahu: halaman 2
http://localhost:8000/produk?page=3  --> Laravel tahu: halaman 3
```

Parameter `?page=` itu ditambahkan **otomatis** oleh Laravel saat user klik tombol pagination nanti.

### b. Laravel Hanya Ambil 10 Produk Sesuai Halaman

Kalau di database ada 1.000 produk:

| URL | Data yang diambil |
|---|---|
| `/produk` | produk nomor 1–10 |
| `/produk?page=2` | produk nomor 11–20 |
| `/produk?page=3` | produk nomor 21–30 |
| ... | ... |
| `/produk?page=100` | produk nomor 991–1000 |

Jadi database **tidak pernah** mengirim 1.000 produk sekaligus. Hanya 10. Ini sangat ringan.

### c. Laravel Siapkan Info Tambahan

Variabel `$produks` yang dikirim ke Blade **bukan sekadar daftar produk**. Laravel menambahkan info penting seperti:

| Info | Arti |
|---|---|
| `total` | Total semua produk (misal 1.000) |
| `perPage` | Jumlah per halaman (10) |
| `currentPage` | Halaman saat ini |
| `lastPage` | Halaman terakhir (misal 100) |
| `links()` | Tombol-tombol pindah halaman |

Info tambahan ini akan dipakai di tahap 4 saat kita menampilkan tombol pagination di Blade.

---

## 5. Analogi Sederhana

Bayangkan kamu di perpustakaan:

- **`all()`** = pustakawan membawa **semua 1.000 buku** ke mejamu sekaligus. Meja jebol, kamu bingung.
- **`paginate(10)`** = pustakawan membawa **10 buku saja dulu**, lalu bilang: "Kalau mau lihat buku berikutnya, panggil saya dan sebut nomor halamannya."

Pustakawan Laravel sangat pintar — dia sudah tahu buku nomor berapa yang harus dia ambil berdasarkan nomor halaman yang kamu sebut.

---

## 6. Perbandingan Sebelum vs Sesudah

| Hal | `Produk::all()` | `Produk::paginate(10)` |
|---|---|---|
| Jumlah data diambil | Semua produk | Hanya 10 per halaman |
| Berat database | Berat kalau data banyak | Selalu ringan |
| Halaman loading | Lama | Cepat |
| Ada info halaman | Tidak ada | Ada (total, currentPage, dll) |
| Bisa pindah halaman | Tidak | Bisa (akan diatur di tahap 4) |

---

## 7. Catatan Penting: Blade Masih Bisa Pakai `@foreach`

Walaupun `$produks` sekarang adalah hasil pagination (bukan array biasa), di file Blade kamu **tetap bisa pakai** `@foreach` seperti biasa:

```php
@foreach ($produks as $produk)
    {{ $produk->nama }} - {{ $produk->harga }}
@endforeach
```

Laravel pintar — hasil `paginate()` bisa dipakai seperti biasa untuk looping. Jadi kamu tidak perlu mengubah kode tampilan yang sudah ada.

Yang **belum ada** di Blade saat ini: tombol untuk pindah halaman. Itu yang akan kita tambahkan di tahap 4.

---

## 8. Ringkasan Tahap 3

| Hal | Isi |
|---|---|
| Yang diubah | `$produks = Produk::all();` → `$produks = Produk::paginate(10);` |
| Efek | Hanya 10 produk diambil per halaman |
| URL otomatis | `/produk?page=2`, `/produk?page=3`, dst |
| Bonus dari Laravel | Info total produk, halaman saat ini, tombol pagination |
| Blade tetap sama | `@foreach` masih bisa dipakai seperti biasa |

---

## 9. Cek Pemahaman Kamu

Jawab singkat di kepala:

1. Method apa yang menggantikan `all()` agar data tampil per halaman?
2. Angka `10` di dalam `paginate(10)` artinya apa?
3. Bagaimana Laravel tahu user sedang di halaman berapa?
4. Apakah di Blade kamu perlu mengubah kode `@foreach` yang sudah ada?

Kalau sudah bisa jawab, kamu siap lanjut.

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: menampilkan link pagination di halaman Blade?**

Kalau iya, tahap 4 kita akan:
1. Tambah satu baris `{{ $produks->links() }}` di file `produk/index.blade.php`.
2. Lihat hasilnya: tombol nomor halaman muncul otomatis.
3. Pahami apa yang dilakukan `links()` dan kenapa begitu mudah.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
