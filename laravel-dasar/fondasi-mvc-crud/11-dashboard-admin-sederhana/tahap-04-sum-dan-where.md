# Tahap 4 — Aggregation Query `sum()` & `where()`: Total Pendapatan + Produk Aktif/Nonaktif

> Materi: Dashboard Admin Sederhana
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: Produk, User, Order

---

## 1. Pengantar Sederhana

Pada **Tahap 3**, kita sudah belajar **`count()`** untuk menghitung jumlah baris. Sekarang dashboard kita sudah bisa tampilkan 3 angka:

- Total produk
- Total user
- Total order

Di **Tahap 4** ini, kita akan tambah **3 angka lagi**:

- **Total pendapatan** (gabungan nilai semua order).
- **Produk aktif** (jumlah produk dengan `is_active = 1`).
- **Produk nonaktif** (jumlah produk dengan `is_active = 0`).

Untuk itu, kita butuh 2 "senjata" baru:

1. **`sum('kolom')`** untuk **menjumlahkan** nilai di sebuah kolom.
2. **`where('kolom', nilai)`** untuk **memfilter** baris sebelum dihitung.

### Analogi: Kasir Toko

Bayangkan kamu seorang **kasir di toko**. Di akhir hari, kamu punya **tumpukan struk** (struk = order).

| Pertanyaan Bos | Cara Hitung | Disebut |
|---|---|---|
| "Berapa banyak struk hari ini?" | Hitung jumlah struk: **87 struk** | `count()` (Tahap 3) |
| "Berapa total uang dari semua struk?" | Jumlahkan angka di tiap struk: 120rb + 90rb + 180rb + ... = **Rp 15.300.000** | `sum()` (Tahap 4) |
| "Berapa struk yang statusnya selesai?" | Filter struk "selesai", lalu hitung: **85 struk** | `where() + count()` (Tahap 4) |
| "Berapa struk yang masih pending?" | Filter struk "pending", lalu hitung: **2 struk** | `where() + count()` (Tahap 4) |

Nah, di tahap ini kita akan terapkan logika yang sama ke tabel `orders` dan `products`.

---

## 2. Apa Itu `sum()`?

### Pengertian Sederhana

**`sum('kolom')`** = method Laravel untuk **menjumlahkan** semua nilai di sebuah kolom tertentu.

Kalau `count()` menghitung **berapa baris**, maka `sum()` menghitung **berapa total nilai** di kolom tertentu.

### Analogi: Menabung di Celengan

Bayangkan kamu punya **celengan**. Setiap hari, kamu masukkan uang:

| Hari | Uang Dimasukkan |
|---|---|
| Senin | Rp 5.000 |
| Selasa | Rp 10.000 |
| Rabu | Rp 7.000 |
| Kamis | Rp 12.000 |
| Jumat | Rp 8.000 |

Kalau kamu tanya:

- "Berapa banyak hari saya menabung?" → **5 hari** (ini `count()`).
- "Berapa total uang di celengan?" → **Rp 42.000** (ini `sum('uang')`).

Nah, `sum()` menjumlahkan **nilai** di kolom, bukan menghitung jumlah baris.

### Contoh di Tabel `orders`

Misal tabel `orders` berisi:

| id | user_id | total | status | created_at |
|----|---------|-------|--------|------------|
| 1020 | 45 | 120000 | selesai | 2026-07-19 |
| 1021 | 78 | 90000 | selesai | 2026-07-19 |
| 1022 | 12 | 180000 | pending | 2026-07-19 |
| 1023 | 56 | 250000 | pending | 2026-07-19 |
| 1024 | 99 | 75000 | selesai | 2026-07-19 |

Kalau kita `count()`:

```php
Order::count();   // hasil: 5 (ada 5 baris)
```

Kalau kita `sum('total')`:

```php
Order::sum('total');   // hasil: 715000 (120000 + 90000 + 180000 + 250000 + 75000)
```

Beda kan? `count()` hitung jumlah baris (5), `sum('total')` jumlahkan kolom `total` (715.000).

### Syntax Dasar

```php
Model::sum('nama_kolom');
```

Contoh:

```php
Order::sum('total');          // total semua order
Order::sum('ongkir');         // total ongkir semua order
Order::sum('jumlah_item');    // total item yang pernah diorder
```

Yang penting: **kolom yang di-sum harus berupa angka** (integer, decimal, float). Kalau kamu `sum()` kolom teks seperti `nama`, hasilnya tidak masuk akal.

### SQL yang Dihasilkan

Saat kamu tulis `Order::sum('total')`, Laravel menerjemahkan ke:

```sql
SELECT SUM(total) FROM orders;
```

Database akan menjumlahkan kolom `total` dari semua baris dan mengembalikan **satu angka**.

---

## 3. Apa Itu `where()`?

Sebelum lanjut ke produk aktif/nonaktif, kita harus paham **`where()`** dulu.

### Pengertian Sederhana

**`where('kolom', nilai)`** = method Laravel untuk **memfilter baris** berdasarkan kondisi tertentu.

Bisa dibaca: "Ambil hanya baris yang **kolomnya sama dengan** nilai tertentu."

### Analogi: Saring Pasir di Pantai

Bayangkan kamu di pantai, mau kumpulkan **batu merah** saja dari tumpukan pasir + batu campuran.

Cara manual: ambil semua, lalu periksa satu-satu, buang yang bukan merah.

Cara pintar: pakai **saringan khusus** yang hanya luluskan batu merah.

`where()` itu seperti **saringan** di query database. Database hanya mengembalikan baris yang cocok dengan kondisi yang kamu tentukan.

### Contoh di Tabel `products`

Misal tabel `products` berisi (kolom `is_active` dijelaskan di materi nomor 10):

| id | nama | is_active |
|----|------|-----------|
| 1 | Kopi Susu Vanilla | 1 |
| 2 | Tumbler Limited | 0 |
| 3 | Gula Aren 500g | 1 |
| 4 | Kaos Tiedye | 0 |
| 5 | Topi Bucket | 1 |

Kalau kita tulis:

```php
Product::where('is_active', 1)->get();
```

Maka Laravel hanya akan ambil baris dengan `is_active = 1`:

- Produk 1 (Kopi Susu Vanilla)
- Produk 3 (Gula Aren 500g)
- Produk 5 (Topi Bucket)

Baris dengan `is_active = 0` (Tumbler, Kaos) **tidak ikut**.

### Syntax Dasar

```php
Model::where('kolom', nilai)->...;
```

Beberapa contoh:

```php
Product::where('is_active', 1)->get();         // produk aktif
Product::where('is_active', 0)->get();         // produk nonaktif
Product::where('kategori', 'Minuman')->get();  // produk kategori Minuman
Order::where('status', 'pending')->get();      // order pending
User::where('role', 'admin')->get();           // user dengan role admin
```

`where()` selalu butuh **ditutup** dengan method akhir seperti `->get()` (ambil data), `->count()` (hitung), `->sum()` (jumlahkan), `->first()` (ambil 1), dll. Tanpa penutup, query tidak akan dijalankan.

### SQL yang Dihasilkan

```php
Product::where('is_active', 1)->count();
```

Laravel terjemahkan ke:

```sql
SELECT COUNT(*) FROM products WHERE is_active = 1;
```

Bisa kamu lihat, `where()` di Laravel = `WHERE` di SQL. Persis sama.

---

## 4. Menggabungkan `where()` + `count()`

Sekarang kita gabungkan dua method ini untuk menghitung **produk aktif** dan **produk nonaktif**.

### Produk Aktif

```php
Product::where('is_active', 1)->count();
```

Baca: "Hitung jumlah produk yang `is_active`-nya `1`."

Hasil: angka integer (misal: `3`).

### Produk Nonaktif

```php
Product::where('is_active', 0)->count();
```

Baca: "Hitung jumlah produk yang `is_active`-nya `0`."

Hasil: angka integer (misal: `2`).

### Analogi: Hitung Siswa Lulus vs Tidak Lulus

Bayangkan kamu guru. Daftar nilai siswa:

| Nama | Lulus? |
|---|---|
| Andi | Ya |
| Budi | Tidak |
| Citra | Ya |
| Dini | Tidak |
| Eka | Ya |

Kalau bos tanya:

- "Berapa siswa yang lulus?" → Filter yang "Ya", hitung: **3**.
- "Berapa siswa yang tidak lulus?" → Filter yang "Tidak", hitung: **2**.

Sama persis dengan `Product::where('is_active', 1)->count()` (aktif) dan `Product::where('is_active', 0)->count()` (nonaktif).

### Operator Lain di `where()`

Selain `=`, `where()` bisa pakai operator lain:

```php
Product::where('harga', '>', 100000)->count();          // produk di atas 100rb
Product::where('stok', '<=', 5)->count();               // produk stok menipis
Product::where('nama', 'like', '%kopi%')->count();      // produk mengandung "kopi"
```

Format umum:

```php
Model::where('kolom', 'operator', 'nilai')->...;
```

Tapi di tahap ini kita cuma butuh yang sederhana: `where('is_active', 1)` (operator `=` tidak perlu ditulis, Laravel asumsikan `=` kalau operator tidak diberikan).

---

## 5. Langkah 1: Update `DashboardController`

Sekarang mari kita update `DashboardController` untuk menambahkan 3 angka baru.

Buka file:

```
app/Http/Controllers/Admin/DashboardController.php
```

Ubah isinya menjadi:

```php
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use App\Models\Product;
use App\Models\User;
use App\Models\Order;

class DashboardController extends Controller
{
    public function index()
    {
        // Hitungan dari tahap 3
        $totalProduk = Product::count();
        $totalUser   = User::count();
        $totalOrder  = Order::count();

        // Hitungan baru di tahap 4
        $totalPendapatan  = Order::sum('total');
        $produkAktif      = Product::where('is_active', 1)->count();
        $produkNonaktif   = Product::where('is_active', 0)->count();

        return 'Total Produk: ' . $totalProduk
              . ' | Total User: ' . $totalUser
              . ' | Total Order: ' . $totalOrder
              . ' | Total Pendapatan: ' . $totalPendapatan
              . ' | Produk Aktif: ' . $produkAktif
              . ' | Produk Nonaktif: ' . $produkNonaktif;
    }
}
```

### Yang Berubah dari Versi Tahap 3

| Bagian | Tahap 3 | Tahap 4 |
|---|---|---|
| 3 baris variable | `$totalProduk`, `$totalUser`, `$totalOrder` | tetap ada |
| 3 baris variable baru | tidak ada | **ditambah**: `$totalPendapatan`, `$produkAktif`, `$produkNonaktif` |
| Isi `return` | 3 angka | **6 angka** |

---

## 6. Penjelasan Kode Baru Baris per Baris

Mari kita bedah 3 baris baru dengan tenang.

### Bagian 1: `$totalPendapatan = Order::sum('total');`

```php
$totalPendapatan = Order::sum('total');
```

- `Order` = nama model (mewakili tabel `orders`).
- `::sum('total')` = jumlahkan nilai kolom `total` di semua baris tabel `orders`.
- Hasilnya disimpan di variable `$totalPendapatan`.

Setelah baris ini jalan, `$totalPendapatan` berisi angka (misal: `15300000`).

### Bagian 2: `$produkAktif = Product::where('is_active', 1)->count();`

```php
$produkAktif = Product::where('is_active', 1)->count();
```

Mari kita bedah dari kiri ke kanan:

- `Product::` = mulai query ke tabel `products`.
- `->where('is_active', 1)` = tambahkan kondisi: "kolom `is_active` harus `1`".
- `->count()` = hitung jumlah baris yang lulus filter.

Hasilnya: jumlah produk aktif (misal: `98`).

### Bagian 3: `$produkNonaktif = Product::where('is_active', 0)->count();`

```php
$produkNonaktif = Product::where('is_active', 0)->count();
```

Sama seperti di atas, tapi filter-nya `is_active = 0`. Hasilnya: jumlah produk nonaktif (misal: `22`).

### Mengapa Bisa Dihubungkan dengan `->`?

Di Laravel, konsep ini disebut **method chaining** (merangkai method dengan panah `->`).

Bayangkan seperti **pabrik dengan conveyor belt**:

1. `Product::` → keluarkan produk dari gudang ke conveyor.
2. `->where('is_active', 1)` → lewati saringan: hanya produk aktif yang lolos.
3. `->count()` → di akhir conveyor, hitung berapa produk yang lolos.

Setiap method menerima hasil method sebelumnya, lalu **memproses lebih lanjut**.

Contoh lain chaining:

```php
Product::where('is_active', 1)->where('stok', '>', 0)->count();
```

Ini berarti: hitung produk yang **aktif** DAN **stoknya lebih dari 0**.

---

## 7. Langkah 2: Uji Coba di Browser

Sekarang mari uji coba.

1. Pastikan server jalan (`php artisan serve`).
2. Buka browser ke:

```
http://localhost:8000/admin/dashboard
```

3. Seharusnya muncul teks seperti:

```
Total Produk: 120 | Total User: 340 | Total Order: 87 | Total Pendapatan: 15300000 | Produk Aktif: 98 | Produk Nonaktif: 22
```

### Catatan tentang Penulisan Angka

Perhatikan bahwa `Total Pendapatan: 15300000` tanpa pemisah ribuan. **Jelek** ya? Di Tahap 6 (view blade) kita akan format jadi `Rp 15.300.000`. Tapi untuk sekarang, biarkan dulu angka mentahnya, karena fokus kita ke **query**, bukan tampilan.

### Kalau Ada Error

**Error 1: "Column not found: 1054 Unknown column 'is_active'"**

Penyebab: Kolom `is_active` belum ada di tabel `products`.

Solusi: Buat migration untuk tambah kolom `is_active` (lihat materi nomor 10 tentang Status Produk Aktif/Nonaktif).

```bash
php artisan make:migration add_is_active_to_products_table --table=products
```

Lalu isi migration dengan kode tambah kolom `is_active`, jalankan `php artisan migrate`.

**Error 2: "Column not found: 1054 Unknown column 'total'"**

Penyebab: Kolom `total` belum ada di tabel `orders`.

Solusi: Pastikan migration `orders` punya kolom `total`. Misalnya:

```php
$table->integer('total');   // atau decimal, bigInteger, dll
```

**Error 3: Hasil `sum()` null atau 0 padahal ada data**

Penyebab: Nama kolom salah (typo), atau tabel kosong.

Solusi: Cek nama kolom di database dengan tool seperti phpMyAdmin, DBeaver, atau jalankan `DESCRIBE orders;` di MySQL.

**Error 4: Hasil `sum()` mengembalikan float, bukan integer**

Penyebab: Kolom di database bertipe `decimal` atau `float`.

Solusi: Itu wajar, tidak masalah. Nanti di view kita bisa format dengan `number_format()`.

---

## 8. Memahami Apa yang Terjadi di Belakang Layar

Mari kita lihat **SQL yang sebenarnya dijalankan** untuk 6 query di dashboard kita sekarang.

| Kode Laravel | SQL yang Dijalankan |
|---|---|
| `Product::count()` | `SELECT COUNT(*) FROM products` |
| `User::count()` | `SELECT COUNT(*) FROM users` |
| `Order::count()` | `SELECT COUNT(*) FROM orders` |
| `Order::sum('total')` | `SELECT SUM(total) FROM orders` |
| `Product::where('is_active', 1)->count()` | `SELECT COUNT(*) FROM products WHERE is_active = 1` |
| `Product::where('is_active', 0)->count()` | `SELECT COUNT(*) FROM products WHERE is_active = 0` |

Perhatikan: **6 query terpisah**. Itu artinya dashboard kita saat ini **menghubungi database 6 kali** untuk menampilkan satu halaman.

### Apakah 6 Query Itu Masalah?

Untuk aplikasi **belajar** atau **data kecil**, **tidak masalah**. 6 query cepat selesai (mungkin 20-30ms).

Tapi kalau datanya sudah **besar** (misal jutaan produk, jutaan order), 6 query bisa menjadi **lambat**. Di **Tahap 7 nanti**, kita akan belajar **cache** untuk mengatasi ini.

Untuk sekarang, **tidak perlu pusing**. Fokus kita adalah paham **cara pakai** `sum()` dan `where()`.

---

## 9. Validasi Logika: Hitung Manual

Mari kita validasi logika dengan **contoh data kecil**, supaya kamu yakin query kita benar.

### Skenario Data Tabel `products`

| id | nama | is_active |
|----|------|-----------|
| 1 | Kopi Susu Vanilla | 1 |
| 2 | Tumbler Limited | 0 |
| 3 | Gula Aren 500g | 1 |
| 4 | Kaos Tiedye | 0 |
| 5 | Topi Bucket | 1 |
| 6 | Mug Keramik | 1 |

Mari kita hitung manual dan bandingkan dengan query:

| Query | Hitung Manual | Hasil yang Benar |
|---|---|---|
| `Product::count()` | Hitung semua baris | **6** |
| `Product::where('is_active', 1)->count()` | Hitung baris dengan is_active=1: id 1, 3, 5, 6 | **4** |
| `Product::where('is_active', 0)->count()` | Hitung baris dengan is_active=0: id 2, 4 | **2** |

Cek: 4 + 2 = 6 = total produk. ✅ (Logika konsisten.)

### Skenario Data Tabel `orders`

| id | total | status |
|----|-------|--------|
| 1020 | 120000 | selesai |
| 1021 | 90000 | selesai |
| 1022 | 180000 | pending |
| 1023 | 250000 | pending |
| 1024 | 75000 | selesai |

| Query | Hitung Manual | Hasil yang Benar |
|---|---|---|
| `Order::count()` | Hitung semua baris | **5** |
| `Order::sum('total')` | 120000 + 90000 + 180000 + 250000 + 75000 | **715000** |
| `Order::where('status', 'pending')->count()` | Hitung baris pending: id 1022, 1023 | **2** |

Cek: jumlah baris sama dengan `count()`, total nilai sama dengan `sum()`. ✅

---

## 10. Kesalahan Umum Pemula

### Kesalahan 1: Lupa `->count()` setelah `where()`

```php
// ❌ TIDAK AKAN JALAN (tidak ada method penutup)
$produkAktif = Product::where('is_active', 1);
```

Hasilnya bukan angka, melainkan **object query builder**. Error saat dicoba di-echo.

Yang benar:

```php
// ✅ BENAR
$produkAktif = Product::where('is_active', 1)->count();
```

### Kesalahan 2: Pakai `get()->count()` (Buruk)

```php
// ❌ BURUK (mengambil semua data ke memori dulu)
$produkAktif = Product::where('is_active', 1)->get()->count();
```

Sama seperti kesalahan di Tahap 3: `get()` mengambil **semua data** ke memori PHP dulu, baru dihitung. Boros dan lambat.

Yang benar:

```php
// ✅ BAIK (database yang menghitung)
$produkAktif = Product::where('is_active', 1)->count();
```

### Kesalahan 3: `sum()` Tanpa Argumen

```php
// ❌ ERROR
$total = Order::sum();   // ArgumentCountError
```

`sum()` **wajib** diberi nama kolom. Yang benar:

```php
// ✅ BENAR
$total = Order::sum('total');
```

### Kesalahan 4: `sum()` Kolom Non-Angka

```php
// ❌ HASIL TIDAK MASUK AKAL
Order::sum('status');   // status adalah string ("pending", "selesai")
```

`sum()` hanya cocok untuk kolom **angka** (integer, decimal, float). Kalau dipaksa ke kolom string, hasilnya 0 atau error.

### Kesalahan 5: Salah Urut Chaining

```php
// ❌ ERROR (count() dulu, baru where)
Product::count()->where('is_active', 1);
```

`count()` mengembalikan **angka integer**, bukan query builder. Setelah `count()`, kamu tidak bisa lagi merangkai `where()`.

Yang benar: `where()` dulu, baru `count()`:

```php
// ✅ BENAR
Product::where('is_active', 1)->count();
```

---

## 11. Eksperimen Tambahan (Opsional)

### Eksperimen 1: Filter dengan Kondisi Lain

Coba tambahkan query lain di `index()` sementara untuk belajar:

```php
// Produk dengan stok menipis (<= 5)
$produkStokMenipis = Product::where('stok', '<=', 5)->count();

// Order dengan status pending
$orderPending = Order::where('status', 'pending')->count();

// Order dengan status selesai
$orderSelesai = Order::where('status', 'selesai')->count();

// User yang terdaftar sebagai admin
$jumlahAdmin = User::where('role', 'admin')->count();
```

Tiap query di atas ** sah dan sering dipakai di dashboard sungguhan**.

### Eksperimen 2: Gabungkan Dua Kondisi dengan `where()`

```php
// Produk aktif DAN stok lebih dari 0
$produkAktifBersi = Product::where('is_active', 1)
                            ->where('stok', '>', 0)
                            ->count();
```

Ini contoh **multiple where**. Kamu bisa rangkai `->where(...)` beberapa kali untuk filter bertingkat (logikanya: AND).

### Eksperimen 3: Pakai Operator Lain

```php
Product::where('harga', '>=', 100000)->count();        // produk di atas atau sama dengan 100rb
Product::where('harga', '<', 50000)->count();           // produk di bawah 50rb
Product::where('nama', 'like', '%kopi%')->count();      // produk yang namanya ada "kopi"
```

Eksperimen ini opsional. Yang penting kamu paham **dasarnya** dulu.

---

## 12. Ringkasan Tahap 4

Mari kita rangkum:

| Konsep | Penjelasan Singkat |
|---|---|
| **`sum('kolom')`** | Menjumlahkan **nilai** di sebuah kolom angka. |
| **`where('kolom', nilai)`** | Memfilter baris berdasarkan kondisi. |
| **Chaining** | Merangkai method dengan `->`. Contoh: `where()->count()`. |
| **`sum()` butuh nama kolom** | Wajib kasih argumen: `sum('total')`. |
| **`where()` butuh penutup** | Setelah `where()`, harus ada `->get()`, `->count()`, `->sum()`, dll. |
| **Urutan chaining** | Filter dulu (`where()`), baru akhiri (`count()`/`sum()`). |
| **Jangan `get()->count()`** | Boros memori. Pakai langsung `->count()`. |

### Hasil Akhir Kode Tahap 4

```php
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use App\Models\Product;
use App\Models\User;
use App\Models\Order;

class DashboardController extends Controller
{
    public function index()
    {
        $totalProduk = Product::count();
        $totalUser   = User::count();
        $totalOrder  = Order::count();

        $totalPendapatan  = Order::sum('total');
        $produkAktif      = Product::where('is_active', 1)->count();
        $produkNonaktif   = Product::where('is_active', 0)->count();

        return 'Total Produk: ' . $totalProduk
              . ' | Total User: ' . $totalUser
              . ' | Total Order: ' . $totalOrder
              . ' | Total Pendapatan: ' . $totalPendapatan
              . ' | Produk Aktif: ' . $produkAktif
              . ' | Produk Nonaktif: ' . $produkNonaktif;
    }
}
```

### Apa yang Sudah Bisa Dashboard Kita Lakukan?

Sekarang dashboard sudah bisa tampilkan **6 angka**:

- ✅ Total produk
- ✅ Total user
- ✅ Total order
- ✅ **Total pendapatan** ← baru
- ✅ **Produk aktif** ← baru
- ✅ **Produk nonaktif** ← baru

Yang **belum**:

- ❌ Order terbaru (butuh `latest()` + `take()`).

Itulah yang akan kita pelajari di **Tahap 5**.

---

## 13. Apa Selanjutnya?

Di **Tahap 5**, kita akan belajar mengambil **data order terbaru** untuk ditampilkan di dashboard. Bukan angka ringkasan, tapi **daftar order** (5 order paling baru).

Yang akan kita pelajari:

- **`latest()`** untuk urutkan data dari yang paling baru.
- **`take(n)`** untuk ambil hanya sejumlah `n` baris.
- **`get()`** untuk mengambil data (bukan angka).
- Cara kirim data ini nantinya ke view (Tahap 6).

Setelah Tahap 5, kita punya **semua data** yang dibutuhkan dashboard:

- 6 angka ringkasan.
- Daftar order terbaru.

Baru di **Tahap 6**, kita buat **tampilan cantik** dengan view blade (kartu angka + tabel order terbaru).

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: belajar `latest()` dan `take()` untuk mengambil order terbaru?**

Kalau **ya**, kita akan pelajari:

- Bedanya ambil **angka** (count/sum) vs ambil **data** (list order).
- Cara pakai `Order::latest()->take(5)->get()`.
- Apa itu **collection** di Laravel (singkat, tidak dalam).
- Bagaimana nanti data ini ditampilkan di view (Tahap 6).

Kalau **belum**, kita bisa:

- Ulang penjelasan tentang `sum()`.
- Ulang penjelasan tentang `where()`.
- Praktik dengan tabel dan kolom lain.
- Diskusi tentang SQL yang dihasilkan Laravel.

Tinggal bilang saja.
