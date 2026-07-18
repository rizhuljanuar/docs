# Tahap 5 — Customisasi Pagination & Ringkasan Akhir (Tahap Terakhir)

> Materi: Pagination Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Prasyarat: Selesai Tahap 4 (link pagination sudah muncul di Blade)

---

## 1. Tujuan Tahap Ini (Tahap Terakhir)

Pagination dasar sudah jalan. Sekarang kita pelajari beberapa hal kecil berguna:

1. Mengubah jumlah produk per halaman.
2. Menampilkan info "Menampilkan produk 11-20 dari 1.000".
3. Menangani kasus halaman kosong (tidak ada produk sama sekali).

Di akhir ada **ringkasan lengkap** semua tahap, jadi kamu bisa lihat gambaran utuhnya.

---

## 2. Mengubah Jumlah Produk Per Halaman

Mudah saja: ubah angka di dalam `paginate()`.

```php
// 5 produk per halaman
$produks = Produk::paginate(5);

// 10 produk per halaman (default contoh kita)
$produks = Produk::paginate(10);

// 20 produk per halaman
$produks = Produk::paginate(20);
```

### Bagaimana Memilih Angka yang Tepat?

| Jumlah | Cocok kalau... |
|---|---|
| 5–8 | Produknya besar / ada gambar besar (misal kartu produk lebar) |
| 10–12 | Paling umum dipakai, seimbang |
| 20–25 | Produknya berbentuk baris tabel yang ringkas |

Tips: **jangan terlalu banyak** (balik ke masalah tahap 1), dan **jangan terlalu sedikit** (user capek klik tombol terus).

---

## 3. Menampilkan Info "Menampilkan X dari Y Produk"

User senang tahu posisi mereka. Contoh info yang bagus:

```
Menampilkan produk 11-20 dari 1.000 produk.
```

Di Blade, tambahkan kode ini (di atas atau bawah daftar produk):

```php
<p>
    Menampilkan produk
    {{ $produks->firstItem() }}-{{ $produks->lastItem() }}
    dari {{ $produks->total() }} produk.
</p>
```

### Arti Methodnya

| Method | Arti |
|---|---|
| `$produks->firstItem()` | Nomor urut produk pertama di halaman ini (misal: 11) |
| `$produks->lastItem()` | Nomor urut produk terakhir di halaman ini (misal: 20) |
| `$produks->total()` | Total semua produk di database (misal: 1.000) |

Jadi kalau user sedang di halaman 2 dengan 10 produk per halaman:
- `firstItem()` → 11
- `lastItem()` → 20
- `total()` → 1.000

Hasilnya: **"Menampilkan produk 11-20 dari 1.000 produk."**

---

## 4. Menangani Kasus Halaman Kosong

Apa yang terjadi kalau **tidak ada produk sama sekali** di database?

Tanpa ditangani, halaman akan menampilkan daftar kosong tanpa pesan. User bingung: "Apakah loading error? Apakah produk habis?"

Solusinya: pakai `@forelse` (versi pintar dari `@foreach`) dan `@empty`:

```php
<h1>Daftar Produk</h1>

@forelse ($produks as $produk)
    <div>
        {{ $produk->nama }} -
        Rp {{ $produk->harga }} -
        Stok: {{ $produk->stok }}
    </div>
@empty
    <p>Belum ada produk. Silakan tambahkan produk terlebih dahulu.</p>
@endforelse

{{ $produks->links() }}
```

### Perbedaan `@foreach` vs `@forelse`

| Sintaks | Kalau Data Ada | Kalau Data Kosong |
|---|---|---|
| `@foreach` | Loop seperti biasa | Tidak menampilkan apa-apa (kosong) |
| `@forelse` + `@empty` | Loop seperti biasa | Tampilkan pesan di bagian `@empty` |

Jadi `@forelse` lebih ramah user karena selalu memberi pesan jelas.

---

## 5. Menggunakan Field Produk Lengkap

Sebagai recap, berikut contoh Blade yang menampilkan semua field produk dengan rapi:

```php
<h1>Daftar Produk</h1>

<p>
    Menampilkan produk
    {{ $produks->firstItem() }}-{{ $produks->lastItem() }}
    dari {{ $produks->total() }} produk.
</p>

@forelse ($produks as $produk)
    <div style="border: 1px solid #ddd; margin-bottom: 10px; padding: 10px;">
        <h3>{{ $produk->nama }}</h3>
        <img src="{{ asset('storage/' . $produk->gambar) }}" alt="{{ $produk->nama }}" width="100">
        <p>Harga: Rp {{ $produk->harga }}</p>
        <p>Stok: {{ $produk->stok }}</p>
        <p>Kategori: {{ $produk->kategori }}</p>
        <p>Deskripsi: {{ $produk->deskripsi }}</p>
        <a href="/produk/{{ $produk->slug }}">Lihat Detail</a>
    </div>
@empty
    <p>Belum ada produk.</p>
@endforelse

{{ $produks->links() }}
```

Kode ini sudah lengkap dan siap pakai sebagai contoh belajar.

---

## 6. Ringkasan Lengkap Materi Pagination Produk

### Gambaran Besar

Pagination = membagi daftar panjang menjadi halaman-halaman kecil supaya ringan dan rapi.

### Alur Kerja Pagination di Laravel

```
1. User buka /produk
2. Controller ambil 10 produk per halaman dengan paginate(10)
3. Controller kirim data $produks ke Blade
4. Blade tampilkan 10 produk dengan @foreach
5. Blade tampilkan tombol pagination dengan $produks->links()
6. User klik halaman 2 --> URL jadi /produk?page=2
7. Controller ambil produk halaman 2 (produk 11-20)
8. ... ulang
```

### Kode Penting yang Wajib Diingat

| Lokasi | Kode | Fungsi |
|---|---|---|
| Controller | `Produk::paginate(10)` | Ambil 10 produk per halaman |
| Blade | `{{ $produks->links() }}` | Tampilkan tombol pagination |
| Blade | `{{ $produks->total() }}` | Tampilkan total produk |
| Blade | `{{ $produks->firstItem() }}` | Nomor produk pertama di halaman ini |
| Blade | `{{ $produks->lastItem() }}` | Nomor produk terakhir di halaman ini |
| Blade | `@forelse ... @empty ... @endforelse` | Loop dengan pesan kalau data kosong |

### Manfaat Pagination

**Untuk User:**
- Halaman cepat terbuka.
- Mudah mencari produk.
- Tampilan rapi.

**Untuk Performa:**
- Server hanya ambil 10 produk, bukan semua.
- Database ringan.
- Loading cepat.
- Bisa melayani banyak user.

---

## 7. Kesalahan Umum yang Perlu Dihindari

| Kesalahan | Akibat | Solusi |
|---|---|---|
| Masih pakai `all()` di controller | Tetap berat, tidak ada pagination | Ganti dengan `paginate(10)` |
| Lupa menambah `links()` di Blade | Produk tampil 10 tapi tidak bisa pindah halaman | Tambah `{{ $produks->links() }}` |
| Pakai `@foreach` padahal data bisa kosong | User bingung kalau tidak ada produk | Pakai `@forelse` + `@empty` |
| Angka `paginate()` terlalu besar (misal 100) | Pagination jadi tidak berguna | Pakai 5–25 sesuai kebutuhan |

---

## 8. Apa Selanjutnya Setelah Pagination?

Sekarang kamu sudah menguasai dasar pagination. Selanjutnya bisa dipelajari (di materi lain):

- **Pencarian produk + pagination** (gabung dengan fitur search).
- **Pagination dengan filtering** (per kategori).
- **Pagination custom** (ganti tampilan tombol dengan design sendiri).
- **Pagination dengan AJAX** (pindah halaman tanpa reload).

Tapi untuk sekarang, **selamat!** Kamu sudah menyelesaikan materi Pagination Produk.

---

## 9. Cek Pemahaman Akhir

Jawab singkat untuk memastikan kamu paham:

1. Method apa di controller untuk mengambil data per halaman?
2. Method apa di Blade untuk menampilkan tombol pagination?
3. Bagaimana cara mengubah jumlah produk per halaman?
4. Method apa untuk menampilkan total produk?
5. Apa beda `@foreach` dan `@forelse`?
6. Apa URL yang muncul saat user klik halaman 2?

Kalau kamu bisa jawab semua, kamu sudah paham dasar pagination di Laravel.

---

## Selesai!

Materi **Pagination Produk** sudah lengkap dari tahap 1 sampai tahap 5.

Ringkasan 5 tahap:

1. **Konsep pagination** — kenapa tidak boleh tampilkan semua sekaligus.
2. **Controller lama** — lihat `Produk::all()` dan pahami masalahnya.
3. **Gunakan paginate** — ganti ke `Produk::paginate(10)`.
4. **Link pagination di Blade** — tambah `{{ $produks->links() }}`.
5. **Customisasi & ringkasan** — info total, halaman kosong, best practice.

Selamat belajar! Kalau ada bagian yang masih membingungkan, ulangi tahap itu pelan-pelan. Laravel itu soal latihan dan kebiasaan.
