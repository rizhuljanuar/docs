# Tahap 2 — Lihat Controller Produk Sebelum Pagination (Pahami Dulu Sebelum Ubah)

> Materi: Pagination Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Prasyarat: Selesai Tahap 1 (konsep pagination)

---

## 1. Tujuan Tahap Ini

Sebelum kita mengubah kode, kita harus **melihat dulu** bagaimana cara controller produk saat ini mengambil data dari database.

Pada tahap ini kita **belum mengubah apa-apa**. Kita hanya:
- Lihat file controller produk.
- Pahami baris mana yang mengambil semua produk.
- Mengerti kenapa baris itu yang nanti harus kita ganti.

---

## 2. Ingat: Apa Itu Controller?

Controller adalah **tempat logika aplikasi**. Tugasnya: menerima permintaan dari user, mengambil data dari database, lalu mengirim data itu ke halaman tampilan (Blade).

Alur sederhananya:

```
User buka halaman  ->  Controller  ->  Database
                                          |
                   Controller ambil data <-+
                            |
                            v
                   Kirim data ke Blade (tampilan)
```

Jadi controller adalah **jembatan** antara user dan database.

---

## 3. Lokasi File Controller Produk

Di Laravel, biasanya file controller produk ada di:

```
app/Http/Controllers/ProdukController.php
```

> Catatan: Nama controller kamu mungkin sedikit berbeda, misalnya `ProductController.php`. Yang penting isinya sama — tempat mengatur data produk.

---

## 4. Isi Controller Produk (Versi Lama — Belum Pagination)

Ini contoh controller produk yang **masih mengambil semua data sekaligus**:

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
        // Ambil SEMUA produk dari database
        $produks = Produk::all();

        // Kirim data produk ke halaman tampilan
        return view('produk.index', compact('produks'));
    }
}
```

Tidak perlu dihafal. Kita fokus ke **satu baris** saja di bawah ini.

---

## 5. Fokus ke Satu Baris Ini

```php
$produks = Produk::all();
```

Arti per kata:

| Bagian | Arti |
|---|---|
| `$produks` | Nama variabel (kotak) untuk menyimpan data produk |
| `=` | Isi variabel dengan hasil di sebelah kanan |
| `Produk` | Nama model produk (mewakili tabel `produks` di database) |
| `::all()` | Method bawaan Laravel: **ambil semua baris** dari tabel |

Jadi satu kalimatnya:

> "Ambil semua produk dari database, lalu simpan ke variabel `$produks`."

---

## 6. Kenapa Baris Ini Jadi Masalah?

`Produk::all()` mengambil **semua produk sekaligus**.

- Kalau di database ada 10 produk → tidak masalah.
- Kalau ada 1.000 produk → **semua 1.000 diambil sekaligus**.
- Kalau ada 10.000 produk → server bisa tertekan, halaman lambat.

Inilah yang nanti kita perbaiki di tahap berikutnya.

---

## 7. Apa Itu `compact('produks')`?

Baris terakhir:

```php
return view('produk.index', compact('produks'));
```

Artinya:

- `view('produk.index')` → tampilkan file tampilan `produk/index.blade.php`.
- `compact('produks')` → kirim variabel `$produks` ke file Blade itu supaya bisa dipakai di tampilan.

> Catatan: `compact()` hanya cara singkat menulis `['produks' => $produks]`. Tidak perlu dibahas dalam sekarang, yang penting kamu tahu fungsinya mengoper data ke Blade.

---

## 8. Ringkasan Tahap 2

| Hal | Isi |
|---|---|
| Lokasi file | `app/Http/Controllers/ProdukController.php` |
| Method penting | `index()` — untuk menampilkan daftar produk |
| Baris yang akan diubah | `$produks = Produk::all();` |
| Masalah | `all()` ambil semua produk sekaligus |
| Tujuan tahap 3 | Ganti `all()` dengan `paginate(10)` |

---

## 9. Cek Pemahaman Kamu

Sebelum lanjut, jawab singkat di kepala:

1. Di controller produk, method apa yang bertugas menampilkan daftar produk?
2. Method `Produk::all()` itu melakukan apa?
3. Kenapa `Produk::all()` jadi masalah kalau datanya banyak?

Kalau kamu sudah bisa jawab ketiganya, kamu siap lanjut.

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: mengubah `Produk::all()` menjadi `Produk::paginate(10)` di controller?**

Kalau iya, tahap 3 kita akan:
1. Lihat perubahan kode (cuma 1 kata yang berubah).
2. Pahami apa yang dilakukan `paginate(10)` di belakang layar.
3. Lihat bagaimana Laravel secara otomatis tahu halaman saat ini (lewat URL `?page=2`, dll).

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
