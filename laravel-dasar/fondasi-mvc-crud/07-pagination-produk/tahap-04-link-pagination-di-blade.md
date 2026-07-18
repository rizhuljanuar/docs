# Tahap 4 — Tampilkan Link Pagination di Halaman Blade

> Materi: Pagination Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Prasyarat: Selesai Tahap 3 (controller sudah pakai `paginate(10)`)

---

## 1. Tujuan Tahap Ini

Controller sudah pakai `paginate(10)`, tapi **di halaman Blade belum ada tombol pindah halaman**. User belum bisa klik "halaman 2", "halaman 3", dst.

Di tahap ini kita tambahkan **satu baris kode** saja di file Blade, lalu tombol pagination akan muncul otomatis.

---

## 2. Lokasi File Blade Produk

Biasanya ada di:

```
resources/views/produk/index.blade.php
```

> Catatan: Lokasi dan nama file kamu mungkin sedikit berbeda. Yang penting ini adalah file tampilan yang menampilkan daftar produk.

---

## 3. Isi Blade Versi Lama (Belum Ada Pagination)

Contoh sederhana tampilan daftar produk:

```php
<h1>Daftar Produk</h1>

<ul>
    @foreach ($produks as $produk)
        <li>
            {{ $produk->nama }} -
            Rp {{ $produk->harga }} -
            Stok: {{ $produk->stok }}
        </li>
    @endforeach
</ul>
```

Saat ini halaman hanya menampilkan produk yang dikirim controller (10 produk per halaman), **tanpa cara untuk pindah halaman**.

---

## 4. Tambahkan SATU Baris Ini

Di bagian bawah daftar produk (setelah `</ul>` atau setelah tabel produk), tambahkan:

```php
{{ $produks->links() }}
```

Jadi file lengkapnya menjadi:

```php
<h1>Daftar Produk</h1>

<ul>
    @foreach ($produks as $produk)
        <li>
            {{ $produk->nama }} -
            Rp {{ $produk->harga }} -
            Stok: {{ $produk->stok }}
        </li>
    @endforeach
</ul>

{{ $produks->links() }}
```

**Hanya 1 baris tambahan.** Itu saja.

---

## 5. Arti `{{ $produks->links() }}` Per Kata

| Bagian | Arti |
|---|---|
| `{{ ... }}` | "Tampilkan hasil ini di layar" (echo di Blade) |
| `$produks` | Variabel hasil `paginate(10)` dari controller |
| `->links()` | Method Laravel: buat tombol-tombol pagination |

Satu kalimatnya:

> "Tampilkan tombol pindah halaman berdasarkan data `$produks`."

---

## 6. Apa yang Muncul di Browser?

Setelah ditambahkan, di bawah daftar produk akan muncul otomatis seperti ini:

```
« 1  2  3  4  5  ...  100 »
```

- `«` → ke halaman pertama
- Angka `1` (tercetak tebal) → halaman saat ini
- Angka lain → pindah ke halaman itu
- `...` → halaman yang disembunyikan (karena terlalu banyak)
- `100` → halaman terakhir
- `»` → ke halaman terakhir

Saat user klik salah satu angka, Laravel **otomatis** membuka URL seperti `/produk?page=2` dan menampilkan produk halaman tersebut.

---

## 7. Kenapa Ini Sangat Mudah?

Biasanya membuat pagination itu rumit:
- Harus hitung total data.
- Harus hitung jumlah halaman.
- Harus buat tombol satu per satu.
- Harus atur kondisi "tombol aktif", "tombol disabled", dll.

Laravel sudah **mengerjakan semua itu** lewat method `links()`. Kamu tinggal panggil satu baris, selesai.

---

## 8. Analogi Sederhana

Bayangkan kamu menulis buku:

- **Tanpa daftar isi & nomor halaman** = pembaca bingung mau lompat ke bab mana.
- **Dengan daftar isi & nomor halaman** = pembaca tinggal lihat nomor, langsung lompat.

`{{ $produks->links() }}` itu seperti "cetak nomor halaman" secara otomatis di bukumu. Laravel yang hitung, kamu tinggal tampilkan.

---

## 9. Catatan: Tampilan Default vs Bootstrap/Tailwind

Secara default, Laravel pakai **CSS Tailwind** untuk gaya tombol pagination.

Kalau kamu pakai Bootstrap (framework CSS lama yang masih populer), kamu perlu menulis sedikit berbeda:

```php
{{ $produks->links('pagination::bootstrap-5') }}
```

Tapi untuk pemula, **cukup pakai `{{ $produks->links() }}`** dulu. Kerja dulu, percantik nanti.

---

## 10. Ringkasan Tahap 4

| Hal | Isi |
|---|---|
| Lokasi file | `resources/views/produk/index.blade.php` |
| Yang ditambahkan | `{{ $produks->links() }}` di bawah daftar produk |
| Efek | Tombol pagination muncul otomatis |
| URL otomatis | Klik halaman 2 → `/produk?page=2` |
| Framework CSS | Default Tailwind (atau bisa pakai Bootstrap) |

---

## 11. Cek Pemahaman Kamu

Jawab singkat di kepala:

1. Method apa yang dipakai di Blade untuk menampilkan tombol pagination?
2. Di mana posisi kode itu biasanya diletakkan?
3. Saat user klik tombol halaman 2, URL jadi seperti apa?
4. Kenapa pagination di Laravel sangat mudah dibanding bikin manual?

Kalau sudah bisa jawab, kamu siap lanjut.

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: customisasi pagination dan ringkasan akhir?**

Kalau iya, tahap 5 (tahap terakhir) kita akan:
1. Ubah jumlah produk per halaman (misal jadi 5 atau 20).
2. Tampilkan info "Menampilkan produk 11-20 dari 1.000".
3. Tangani kasus halaman kosong (tidak ada produk sama sekali).
4. Ringkasan lengkap semua tahap.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
