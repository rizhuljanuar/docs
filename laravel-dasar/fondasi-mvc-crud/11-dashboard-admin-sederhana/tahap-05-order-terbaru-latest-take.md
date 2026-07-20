# Tahap 5 — Mengambil Order Terbaru dengan `latest()` & `take()`

> Materi: Dashboard Admin Sederhana
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: Produk, User, Order

---

## 1. Pengantar Sederhana

Pada **Tahap 3 dan 4**, kita sudah belajar **aggregation query** untuk menghitung **angka ringkasan**:

- `count()` → jumlah baris.
- `sum()` → total nilai kolom.
- `where()` → filter baris.

Sekarang dashboard kita sudah bisa tampilkan **6 angka**:

- Total produk
- Total user
- Total order
- Total pendapatan
- Produk aktif
- Produk nonaktif

Di **Tahap 5** ini, kita akan belajar **jenis query berbeda**, yaitu mengambil **data (baris)**, bukan angka ringkasan.

Khususnya, kita akan ambil **5 order paling baru** untuk ditampilkan di dashboard.

### Analogi: Kotak Masuk WhatsApp

Bayangkan kamu buka WhatsApp. Di layar utama, kamu tidak melihat **semua pesan yang pernah masuk** (bisa ribuan). Yang kamu lihat adalah **daftar percakapan terbaru**, misalnya 10-20 teratas.

Kenapa? Karena:

1. Yang penting biasanya **yang terbaru**.
2. Kamu tidak mungkin scroll semua ribuan pesan tiap hari.
3. Cukup lihat daftar terbaru, **klik salah satu** kalau mau detail.

Dashboard admin juga sama. Admin tidak butuh **melihat semua 87 order** di dashboard. Yang admin butuhkan adalah **5 order paling baru**, untuk tahu:

- "Ada order baru apa aja?"
- "Siapa yang baru belanja?"
- "Ada order yang harus segera saya proses?"

Kalau mau lihat detail, admin tinggal klik order itu untuk ke halaman `/admin/orders/{id}`.

Itulah yang akan kita buat di tahap ini: ambil **5 order terbaru** dari database.

---

## 2. Bedanya Angka Ringkasan vs Daftar Data

Sebelum mulai, mari kita paham bedanya dua jenis query yang sudah kita pelajari.

### Jenis 1: Query Angka Ringkasan (Aggregation)

| Method | Mengembalikan | Contoh |
|---|---|---|
| `count()` | 1 angka integer | `Order::count()` → `87` |
| `sum('total')` | 1 angka | `Order::sum('total')` → `15300000` |

Hasilnya: **satu nilai**. Cocok untuk kartu statistik di dashboard.

### Jenis 2: Query Daftar Data

| Method | Mengembalikan | Contoh |
|---|---|---|
| `get()` | **Collection** berisi banyak baris | `Order::get()` → semua order |
| `take(5)->get()` | Collection berisi 5 baris | `Order::take(5)->get()` → 5 order pertama |

Hasilnya: **banyak baris** (kotak berisi banyak data). Cocok untuk **tabel** atau **daftar** di dashboard.

### Analogi: Mintalan ke Kasir

Bayangkan kamu tanya ke kasir toko:

| Pertanyaan | Jawaban Kasir | Jenis |
|---|---|---|
| "Berapa total transaksi hari ini?" | "87 transaksi, Bos." | **Angka** (count) |
| "Tolong sebut 5 transaksi terakhir!" | "Yang pertama, Andi, Rp 250rb. Kedua, Budi, Rp 180rb..." | **Daftar data** (get) |

Di tahap 5 ini, kita fokus ke **jenis kedua**: mengambil **daftar order**.

---

## 3. Apa Itu `latest()`?

### Pengertian Sederhana

**`latest()`** = method Laravel untuk **mengurutkan data dari yang paling baru**.

Secara default, `latest()` mengurutkan berdasarkan kolom `created_at` (timestamp saat data dibuat), dari yang **paling besar** (terbaru) ke yang **paling kecil** (terlama).

### Analogi: Tumpukan Surat di Meja

Bayangkan kamu punya tumpukan surat di meja. Setiap hari ada surat baru datang, kamu taruh di atas tumpukan.

Saat kamu lihat tumpukan itu, **surat paling atas** adalah **surat yang paling baru datang**.

Nah, `latest()` itu seperti menyuruh Laravel: "Ambil surat paling atas dulu, lalu di bawahnya, lalu di bawahnya..."

### Syntax Dasar

```php
Order::latest()->get();
```

Mari kita bedah:

| Bagian | Arti |
|---|---|
| `Order::` | Mulai query ke tabel `orders`. |
| `->latest()` | Urutkan dari yang terbaru (descending, berdasarkan `created_at`). |
| `->get()` | Ambil datanya sebagai collection. |

Hasilnya: **collection** berisi semua order, urut dari yang paling baru.

### SQL yang Dihasilkan

```php
Order::latest()->get();
```

Laravel terjemahkan ke:

```sql
SELECT * FROM orders ORDER BY created_at DESC;
```

- `ORDER BY created_at` = urutkan berdasarkan kolom `created_at`.
- `DESC` = descending (dari besar ke kecil, jadi terbaru di atas).

### Bisa Pakai Kolom Lain?

Ya. Misalnya, kita mau urut berdasarkan kolom `id` (angka urut):

```php
Order::latest('id')->get();   // urut berdasarkan id terbesar
```

Atau berdasarkan kolom `updated_at`:

```php
Order::latest('updated_at')->get();
```

Tapi untuk dashboard, `latest()` (tanpa argumen) sudah cukup karena kita peduli dengan **kapan order dibuat**.

---

## 4. Apa Itu `take()`?

### Pengertian Sederhana

**`take(n)`** = method Laravel untuk **membatasi jumlah baris** yang diambil, hanya `n` baris pertama.

### Analogi: Antrian Loket

Bayangkan ada antrian orang di loket tiket. Kamu bilang ke satpam:

> "Saya cuma melayani 5 orang pertama, yang lain saya skip."

`take(5)` seperti perintah itu: "Ambil 5 baris pertama, sisanya tidak usah."

### Syntax Dasar

```php
Order::take(5)->get();
```

| Bagian | Arti |
|---|---|
| `Order::` | Mulai query ke tabel `orders`. |
| `->take(5)` | Batasi hanya 5 baris pertama. |
| `->get()` | Ambil datanya. |

Hasil: collection berisi maksimal 5 order (bisa kurang kalau di tabel cuma ada 3 order).

### SQL yang Dihasilkan

```php
Order::take(5)->get();
```

Laravel terjemahkan ke:

```sql
SELECT * FROM orders LIMIT 5;
```

`LIMIT 5` di SQL = ambil maksimal 5 baris.

---

## 5. Menggabungkan `latest()` + `take()` + `get()`

Sekarang kita gabungkan ketiganya untuk mengambil **5 order terbaru**.

### Query yang Kita Mau

> "Ambil 5 order yang paling baru dibuat."

### Kode Laravel-nya

```php
$orderTerbaru = Order::latest()->take(5)->get();
```

Baca dari kiri ke kanan:

1. `Order::` → mulai dari tabel `orders`.
2. `->latest()` → urutkan dari yang paling baru.
3. `->take(5)` → ambil 5 baris pertama saja.
4. `->get()` → eksekusi query, kembalikan data sebagai collection.

### SQL yang Dihasilkan

```sql
SELECT * FROM orders ORDER BY created_at DESC LIMIT 5;
```

Mari kita bedah:

| Bagian SQL | Arti |
|---|---|
| `SELECT *` | Ambil semua kolom. |
| `FROM orders` | Dari tabel orders. |
| `ORDER BY created_at DESC` | Urutkan berdasarkan created_at, descending (terbaru dulu). |
| `LIMIT 5` | Ambil maksimal 5 baris. |

### Analogi Conveyor Belt (Lanjutan Tahap 4)

Di tahap 4 kita pakai analogi **conveyor belt pabrik**. Mari kita pakai lagi:

1. `Order::` → keluarkan semua order dari gudang ke conveyor.
2. `->latest()` → lewati mesin pengurut: yang terbaru di depan.
3. `->take(5)` → di tengah conveyor, ambil 5 pertama, sisanya didistribusi balik ke gudang.
4. `->get()` → di akhir conveyor, kemas hasilnya jadi satu kotak (collection).

Hasil: satu kotak berisi 5 order paling baru.

---

## 6. Apa Itu "Collection"?

Saat kita pakai `->get()`, Laravel mengembalikan **collection**. Apa itu?

### Pengertian Sederhana

**Collection** adalah kotak khusus Laravel berisi **banyak data**. Bisa dibayangkan seperti **array pintar**.

Bayangkan array biasa di PHP:

```php
$angka = [10, 20, 30, 40, 50];
```

Collection itu mirip array, tapi **punya banyak method siap pakai** untuk memanipulasi data, misalnya:

- `->count()` → hitung jumlah elemen.
- `->first()` → ambil elemen pertama.
- `->last()` → ambil elemen terakhir.
- `->map(...)` → ubah tiap elemen.
- `->filter(...)` → saring elemen.

### Contoh Collection Order

Setelah `Order::latest()->take(5)->get()`, variable `$orderTerbaru` berisi kira-kira:

```php
Illuminate\Database\Eloquent\Collection {
    items: [
        0 => Order { id: 1024, user_id: 99, total: 75000, status: "selesai", created_at: "2026-07-19 16:00" },
        1 => Order { id: 1023, user_id: 56, total: 250000, status: "pending", created_at: "2026-07-19 15:30" },
        2 => Order { id: 1022, user_id: 12, total: 180000, status: "pending", created_at: "2026-07-19 14:10" },
        3 => Order { id: 1021, user_id: 78, total: 90000, status: "selesai", created_at: "2026-07-19 11:45" },
        4 => Order { id: 1020, user_id: 45, total: 120000, status: "selesai", created_at: "2026-07-19 10:23" },
    ]
}
```

Perhatikan:

- Isinya **5 object Order** (baris dari database).
- Diurutkan dari `created_at` terbesar (16:00, terbaru) ke terkecil (10:23, paling lama di antara 5 ini).

Untuk **mengakses data di dalamnya**, kita bisa pakai loop `foreach` (akan kita lakukan di **Tahap 6** saat bikin view).

### Jangan Pusing Dulu

Untuk tahap ini, cukup tahu:

- `get()` → kembalikan **collection**.
- Collection = **kotak berisi banyak baris data**.
- Kita bisa loop dengan `foreach`.

Detail method collection akan kita pelajari di materi lain. Sekarang fokus ke cara **mengambil** datanya.

---

## 7. Langkah 1: Update `DashboardController`

Sekarang mari kita update `DashboardController` untuk menambahkan query order terbaru.

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

        // Hitungan dari tahap 4
        $totalPendapatan  = Order::sum('total');
        $produkAktif      = Product::where('is_active', 1)->count();
        $produkNonaktif   = Product::where('is_active', 0)->count();

        // Query baru tahap 5: 5 order terbaru
        $orderTerbaru = Order::latest()->take(5)->get();

        // Sementara kita dump untuk lihat isinya
        dd($orderTerbaru);
    }
}
```

### Yang Berubah dari Versi Tahap 4

| Bagian | Tahap 4 | Tahap 5 |
|---|---|---|
| 6 variable ringkasan | ada | tetap ada |
| Variable `$orderTerbaru` | tidak ada | **ditambah** |
| Isi `return` | teks 6 angka | **diganti `dd()`** sementara |

### Kenapa Pakai `dd()` di Tahap Ini?

Karena kita **belum buat view** (tabel HTML). Kalau kita `return` collection langsung ke browser, browser akan tampil error atau halaman kosong (karena collection bukan string).

`dd()` = **dump and die**. Artinya: "Tampilkan isi variable, lalu hentikan eksekusi."

Hasilnya di browser: tampilan khusus Laravel yang menunjukkan isi collection secara detail. Ini sangat berguna untuk **debugging**.

Nanti di **Tahap 6**, kita hapus `dd()` dan ganti dengan `return view(...)`.

---

## 8. Langkah 2: Uji Coba dengan `dd()`

Sekarang waktunya uji coba.

1. Pastikan server Laravel jalan (`php artisan serve`).
2. Buka browser ke:

```
http://localhost:8000/admin/dashboard
```

3. Seharusnya muncul **halaman khusus Laravel** dengan judul "Illuminate\Database\Eloquent\Collection".

Tampilan `dd()` kira-kira seperti ini (sederhananya):

```
Illuminate\Database\Eloquent\Collection {▼
    #items: array:5 [▼
        0 => App\Models\Order {▼
            id: 1024
            user_id: 99
            total: 75000
            status: "selesai"
            created_at: "2026-07-19 16:00:00"
            updated_at: "2026-07-19 16:00:00"
        }
        1 => App\Models\Order {▼
            id: 1023
            user_id: 56
            total: 250000
            status: "pending"
            created_at: "2026-07-19 15:30:00"
            ...
        }
        2 => App\Models\Order { ... }
        3 => App\Models\Order { ... }
        4 => App\Models\Order { ... }
    ]
}
```

### Yang Harus Diperhatikan

- ✅ Ada **5 object Order** (index 0 sampai 4).
- ✅ Diurutkan dari `created_at` terbaru (16:00) ke paling lama di antara 5 ini.
- ✅ Tiap object punya kolom `id`, `user_id`, `total`, `status`, `created_at`.

Kalau yang muncul **bukan** seperti di atas, cek error di bawah.

### Kalau Ada Error

**Error 1: "Call to a member function take() on null"**

Penyebab: Ada yang salah dengan pemanggilan method, misalnya typo `lastest()` atau `take`.

Solusi: Pastikan penulisan: `Order::latest()->take(5)->get();` (perhatikan titik dan kurung).

**Error 2: Collection Kosong (items: array: [])**

Penyebab: Tabel `orders` kosong (tidak ada data).

Solusi: Isi data order dulu, atau jalankan seeder/factory.

**Error 3: Hanya Muncul 3 Order Padahal Diminta 5**

Penyebab: Di tabel `orders` hanya ada 3 baris.

Solusi: Itu wajar. `take(5)` hanya batas maksimal. Kalau data kurang, hasilnya sesuai jumlah data.

**Error 4: Urutan Tidak Sesuai Harapan**

Penyebab: Kolom `created_at` tidak terisi dengan benar (misalnya semua sama).

Solusi: Pastikan saat membuat order, kolom `created_at` otomatis diisi (Laravel mengisi ini otomatis kalau model pakai timestamps, default-nya ON).

---

## 9. Eksperimen Tambahan (Opsional)

Supaya makin paham, coba eksperimen berikut sebelum lanjut.

### Eksperimen 1: Ubah Jumlah `take()`

Coba ubah `take(5)` jadi `take(3)` atau `take(10)`.

```php
$orderTerbaru = Order::latest()->take(3)->get();   // ambil 3
$orderTerbaru = Order::latest()->take(10)->get();  // ambil 10
```

Lihat bagaimana jumlah baris di `dd()` berubah.

### Eksperimen 2: Lihat SQL yang Dijalankan

Pakai cara yang sama seperti di Tahap 3: tambahkan `\DB::listen(...)` di `AppServiceProvider::boot()`.

Lalu buka dashboard, cek `storage/logs/laravel.log`. Kamu akan lihat:

```
select * from `orders` order by `created_at` desc limit 5
```

Itu SQL asli yang dijalankan Laravel.

### Eksperimen 3: Pakai `oldest()` (Kebalikan `latest()`)

```php
$orderTerlama = Order::oldest()->take(5)->get();
```

`oldest()` = kebalikan `latest()`, mengurutkan dari **yang paling lama**. Cocok kalau mau lihat order pertama yang pernah dibuat.

### Eksperimen 4: Gabungkan dengan `where()`

Coba ambil **5 order pending** terbaru:

```php
$orderPendingTerbaru = Order::where('status', 'pending')
                            ->latest()
                            ->take(5)
                            ->get();
```

Ini contoh **chaining panjang**: filter status, urutkan terbaru, ambil 5. Di dashboard sungguhan, query seperti ini sering muncul.

### Eksperimen 5: Pakai Method Collection

Setelah dapat `$orderTerbaru`, coba tambahkan ini:

```php
dd(
    $orderTerbaru->count(),   // hitung jumlah (5)
    $orderTerbaru->first(),   // ambil order pertama
    $orderTerbaru->last(),    // ambil order terakhir
);
```

Ini membuktikan bahwa collection punya **method pintar** yang bisa dipakai langsung.

---

## 10. Beda `count()` di Query vs `count()` di Collection

Sekarang mari kita bahas hal yang sering bikin bingung pemula.

### `count()` di Query (Aggregation)

```php
Order::count();
Order::where('status', 'pending')->count();
```

Ini adalah **aggregation query**. Database yang menghitung, kembalikan 1 angka. Efisien.

### `count()` di Collection

```php
$orderTerbaru = Order::latest()->take(5)->get();
$orderTerbaru->count();   // ← count di collection
```

Ini berbeda! Yang dihitung adalah **jumlah elemen di collection**, bukan query ke database.

Karena `$orderTerbaru` isinya 5 order (hasil `take(5)`), maka `$orderTerbaru->count()` akan kembalikan **5**.

### Analogi

Bayangkan kamu pesan **5 kopi** ke barista.

- `Order::count()` = kamu tanya barista: "Berapa total kopi yang pernah kamu buat hari ini?" (database yang hitung)
- `$orderTerbaru->count()` = kamu hitung sendiri di meja: "Saya pesan 5 kopi, ya 5 ini." (di memori PHP)

**Kesimpulan**: Untuk dashboard, pakai `Order::count()` (aggregation) untuk hitung total semua order. Pakai `$orderTerbaru->count()` hanya kalau mau tahu berapa data yang ada di collection (misal untuk validasi "apakah ada order terbaru?").

### Jangan Tertukar

| Mau... | Pakai |
|---|---|
| Hitung total semua order | `Order::count()` |
| Hitung total order pending | `Order::where('status', 'pending')->count()` |
| Hitung jumlah order di collection `$orderTerbaru` | `$orderTerbaru->count()` |

---

## 11. Kesalahan Umum Pemula

### Kesalahan 1: Lupa `->get()`

```php
// ❌ TIDAK JALAN (kembalikan query builder, bukan data)
$orderTerbaru = Order::latest()->take(5);
```

Tanpa `->get()`, variable berisi **object query builder**, bukan data. Kalau di-`dd()`, akan muncul rumit-rumit, bukan daftar order.

Yang benar:

```php
// ✅ BENAR
$orderTerbaru = Order::latest()->take(5)->get();
```

### Kesalahan 2: `take()` Setelah `get()`

```php
// ❌ SALAH URUTAN
$orderTerbaru = Order::latest()->get()->take(5);
```

Ini sebenarnya **masih jalan**, tapi **tidak efisien**. `get()` dulu akan **mengambil semua order dari database** (bisa ribuan), baru `take(5)` mengambil 5 di memori PHP. Boros!

Yang benar: `take()` **sebelum** `get()`, supaya database yang membatasi:

```php
// ✅ EFISIEN
$orderTerbaru = Order::latest()->take(5)->get();
```

SQL yang dihasilkan jadi `... LIMIT 5`, jadi database hanya kirim 5 baris ke PHP.

### Kesalahan 3: `latest()` Ditulis `lastest()`

```php
// ❌ TYPO
$orderTerbaru = Order::lastest()->take(5)->get();
```

Typo sangat umum. Yang benar: `latest()` (tanpa "s" setelah "la").

### Kesalahan 4: `take()` Tanpa Argumen

```php
// ❌ ERROR
$orderTerbaru = Order::latest()->take()->get();
```

`take()` **wajib** diberi angka. Yang benar:

```php
// ✅ BENAR
$orderTerbaru = Order::latest()->take(5)->get();
```

### Kesalahan 5: Bingung `get()` vs `first()`

| Method | Mengembalikan |
|---|---|
| `->get()` | **Collection** (banyak baris, kotak) |
| `->first()` | **Object tunggal** (satu baris) |

Untuk dashboard, kita mau **5 baris**, jadi pakai `->get()`.

Kalau mau ambil **hanya 1 baris** (order paling baru), pakai `->first()`:

```php
$orderPalingBaru = Order::latest()->first();   // 1 object Order
```

---

## 12. Validasi Logika dengan Contoh Data

Mari validasi dengan data kecil.

### Skenario Data Tabel `orders`

| id | user_id | total | status | created_at |
|----|---------|-------|--------|------------|
| 1020 | 45 | 120000 | selesai | 2026-07-19 10:23 |
| 1021 | 78 | 90000 | selesai | 2026-07-19 11:45 |
| 1022 | 12 | 180000 | pending | 2026-07-19 14:10 |
| 1023 | 56 | 250000 | pending | 2026-07-19 15:30 |
| 1024 | 99 | 75000 | selesai | 2026-07-19 16:00 |
| 1025 | 34 | 60000 | selesai | 2026-07-20 09:00 |
| 1026 | 88 | 220000 | pending | 2026-07-20 10:30 |

Mari kita jalankan query `Order::latest()->take(5)->get();` dan prediksi hasilnya.

### Langkah 1: Urutkan dari Terbaru (`latest()`)

Berdasarkan `created_at` descending:

1. 1026 - 2026-07-20 10:30 ← paling baru
2. 1025 - 2026-07-20 09:00
3. 1024 - 2026-07-19 16:00
4. 1023 - 2026-07-19 15:30
5. 1022 - 2026-07-19 14:10
6. 1021 - 2026-07-19 11:45
7. 1020 - 2026-07-19 10:23 ← paling lama

### Langkah 2: Ambil 5 Pertama (`take(5)`)

Hasil `$orderTerbaru`:

1. Order 1026 (user 88, 220000, pending, 2026-07-20 10:30)
2. Order 1025 (user 34, 60000, selesai, 2026-07-20 09:00)
3. Order 1024 (user 99, 75000, selesai, 2026-07-19 16:00)
4. Order 1023 (user 56, 250000, pending, 2026-07-19 15:30)
5. Order 1022 (user 12, 180000, pending, 2026-07-19 14:10)

Order 1021 dan 1020 **tidak ikut** karena kalah baru.

Kalau kamu `dd($orderTerbaru)`, seharusnya tampil 5 baris dengan ID 1026, 1025, 1024, 1023, 1022.

---

## 13. Ringkasan Tahap 5

Mari kita rangkum apa yang sudah kita pelajari:

| Konsep | Penjelasan Singkat |
|---|---|
| **Jenis query angka vs data** | Angka = `count`/`sum`. Data = `get`. |
| **`latest()`** | Urutkan dari yang paling baru (descending, by `created_at`). |
| **`take(n)`** | Batasi `n` baris pertama. |
| **`get()`** | Eksekusi query, kembalikan **collection**. |
| **Collection** | Kotak berisi banyak data, punya method pintar (`count`, `first`, dll). |
| **Urutan chaining** | `latest()` → `take()` → `get()`. |
| **Jangan `get()` dulu** | Pakai `take()` sebelum `get()` supaya efisien. |
| **`count()` di query vs collection** | Query = database hitung. Collection = PHP hitung elemen yang ada. |

### Hasil Akhir Kode Tahap 5

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
        // Angka ringkasan
        $totalProduk = Product::count();
        $totalUser   = User::count();
        $totalOrder  = Order::count();
        $totalPendapatan  = Order::sum('total');
        $produkAktif      = Product::where('is_active', 1)->count();
        $produkNonaktif   = Product::where('is_active', 0)->count();

        // Daftar order terbaru
        $orderTerbaru = Order::latest()->take(5)->get();

        // Sementara: dump untuk lihat isinya
        dd($orderTerbaru);
    }
}
```

### Apa yang Sudah Bisa Dashboard Kita Lakukan?

Sekarang dashboard sudah punya **semua data** yang dibutuhkan:

- ✅ 6 angka ringkasan (produk, user, order, pendapatan, produk aktif, produk nonaktif).
- ✅ **5 order terbaru** (sebagai collection).

Tapi masih ada **satu masalah**: semua data itu belum **ditampilkan cantik**. Saat ini kita hanya bisa lihat dengan `dd()`, yang cuma cocok untuk debugging.

Itulah yang akan kita selesaikan di **Tahap 6**.

---

## 14. Apa Selanjutnya?

Di **Tahap 6**, kita akan:

1. **Hapus `dd()`** dan ganti dengan `return view('admin.dashboard', ...)`.
2. Buat file view `resources/views/admin/dashboard.blade.php`.
3. Kirim semua data (6 angka + 5 order terbaru) ke view.
4. Tampilkan dengan HTML + Blade:
   - **6 kartu angka** (Total Produk, Total User, dst).
   - **1 tabel** berisi 5 order terbaru.

Setelah Tahap 6, dashboard kita akan terlihat **seperti dashboard sungguhan**, bukan teks mentah atau `dd()`.

### Preview Tampilan Tahap 6

Nanti kira-kira tampilannya seperti ini:

```
┌─────────────────────────────────────────────────────────────┐
│                      DASHBOARD ADMIN                         │
├─────────────────────────────────────────────────────────────┤
│  Total Produk: 120    Total User: 340    Total Order: 87     │
│  Total Pendapatan: Rp 15.300.000                             │
│  Produk Aktif: 98     Produk Nonaktif: 22                    │
├─────────────────────────────────────────────────────────────┤
│  Order Terbaru                                               │
│  ┌──────────┬───────┬───────────┬─────────┬───────────┐     │
│  │ ID       │ User  │ Total     │ Status  │ Tanggal   │     │
│  ├──────────┼───────┼───────────┼─────────┼───────────┤     │
│  │ #1026    │ User8 │ Rp 220rb  │ pending │ 20-07     │     │
│  │ #1025    │ User3 │ Rp 60rb   │ selesai │ 20-07     │     │
│  │ ...      │ ...   │ ...       │ ...     │ ...       │     │
│  └──────────┴───────┴───────────┴─────────┴───────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### Setelah Tahap 6

Tinggal **Tahap 7 terakhir**: tambah **cache** supaya dashboard tidak lemot saat data sudah besar. Materi penutup.

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: membuat view `dashboard.blade.php` untuk menampilkan semua data dengan rapi?**

Kalau **ya**, kita akan pelajari:

- Cara kirim data dari controller ke view (`return view(..., compact(...))`).
- Sintaks Blade dasar: `{{ $variable }}`, `@foreach`, dll.
- Membuat layout HTML sederhana dengan Bootstrap atau CSS biasa.
- Menampilkan 6 kartu angka.
- Menampilkan tabel order terbaru.

Kalau **belum**, kita bisa:

- Ulang penjelasan tentang `latest()` dan `take()`.
- Ulang penjelasan tentang collection.
- Praktik lebih banyak eksperimen dengan query data.
- Diskusi tentang pagination (kalau kamu tertarik).

Tinggal bilang saja.
