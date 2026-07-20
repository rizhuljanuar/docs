# Tahap 3 — Aggregation Query `count()`: Menghitung Jumlah Data

> Materi: Dashboard Admin Sederhana
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: Produk, User, Order

---

## 1. Pengantar Sederhana

Pada **Tahap 2**, kita sudah:

- Membuat `DashboardController`.
- Menambah route `/admin/dashboard`.
- Method `index()` masih hanya `return` teks "Halo, ini halaman Dashboard Admin."

Di **Tahap 3** ini, kita akan ganti teks itu dengan **angka asli** dari database.

Tapi **belum semua angka**. Kita fokus ke **satu jenis query dulu**, yaitu **`count()`** untuk menghitung jumlah data.

Kita akan hitung **3 hal di tahap ini**:

1. Jumlah produk.
2. Jumlah user.
3. Jumlah order.

Total pendapatan, produk aktif/nonaktif, dan order terbaru akan menyusul di tahap-tahap berikutnya (Tahap 4 dan 5).

### Analogi: Menghitung Jumlah Buku di Rak

Bayangkan kamu kerja di toko buku. Bos meminta:

> "Hitung berapa banyak buku di rak nomor 3!"

Cara menghitungnya:

- Kamu **berjalan ke rak nomor 3**.
- Lalu kamu **hitung satu per satu**: 1, 2, 3, 4, 5, ..., 24.
- Lalu kamu lapor: "Rak nomor 3 ada **24 buku**, Bos!"

Di Laravel, operasi "hitung berapa banyak data di tabel" itu **sudah otomatis**. Kita tidak perlu menghitung manual satu per satu. Cukup panggil satu method: **`count()`**.

Itulah yang akan kita pelajari sekarang.

---

## 2. Apa Itu Aggregation Query?

Sebelum masuk kode, kita harus paham dulu konsep **aggregation query** (query agregasi), supaya tidak sekadar "niru kode tanpa ngerti".

### Pengertian Sederhana

**Aggregation Query** adalah jenis query yang **bukan mengambil data mentah**, tapi **menghitung hasil ringkasan** dari banyak baris data.

Lebih gampangnya:

| Jenis Query | Tujuan | Contoh |
|---|---|---|
| **Query Biasa** | Ambil data mentah (baris per baris) | `SELECT * FROM products` → kembalikan semua produk. |
| **Aggregation Query** | Hitung **ringkasan** dari banyak baris | `SELECT COUNT(*) FROM products` → kembalikan **satu angka** (jumlah baris). |

### Analogi: Daftar Absen vs Jumlah Murid

Bayangkan kamu seorang guru. Kamu punya **daftar absen** berisi nama 30 murid.

| Pertanyaan | Cara Mencari Jawaban | Jenis |
|---|---|---|
| "Siapa saja murid di kelas ini?" | Baca daftar absen satu per satu, tunjukkan semua nama. | **Query Biasa** (ambil data mentah) |
| "Berapa jumlah murid di kelas ini?" | Lihat angka di akhir daftar: **30 murid**. Tidak perlu sebut nama satu per satu. | **Aggregation Query** (cuma hasil ringkasan) |

Sama di Laravel:

- `Product::all()` → **query biasa**, ambil semua produk (berat kalau datanya banyak).
- `Product::count()` → **aggregation query**, ambil **satu angka** saja (ringan).

### Kenapa Aggregation Query Itu Penting untuk Dashboard?

Karena dashboard **tidak butuh data mentah**. Dashboard **cuma butuh angka ringkasan**:

- "Total produk?" → 24.
- "Total user?" → 340.
- "Total order?" → 87.

Kalau kita pakai `Product::all()` lalu hitung manual di PHP, Laravel akan **mengambil semua data produk dari database ke memori** dulu, baru menghitung jumlahnya. Itu **boros banget** (bayangkan kalau ada 100.000 produk).

Sedangkan `Product::count()` langsung menyuruh **database yang menghitung**, lalu database balas dengan satu angka. **Cepat dan ringan.**

---

## 3. Jenis-Jenis Aggregation Query di Laravel

Laravel punya banyak method aggregation. Yang paling sering dipakai:

| Method | Fungsi | Contoh SQL yang Dihasilkan |
|---|---|---|
| `count()` | Hitung jumlah baris | `SELECT COUNT(*) FROM products` |
| `sum('kolom')` | Jumlahkan nilai di kolom tertentu | `SELECT SUM(total) FROM orders` |
| `avg('kolom')` | Rata-rata nilai kolom | `SELECT AVG(harga) FROM products` |
| `min('kolom')` | Nilai terkecil kolom | `SELECT MIN(harga) FROM products` |
| `max('kolom')` | Nilai terbesar kolom | `SELECT MAX(harga) FROM products` |

Di **Tahap 3 ini**, kita fokus ke **`count()`** dulu. Yang lain menyusul.

---

## 4. Cara Pakai `count()` di Laravel

### Bentuk Paling Sederhana

```php
Product::count();
```

Satu baris itu. Selesai. Laravel akan:
1. Menghubungi database.
2. Menjalankan SQL: `SELECT COUNT(*) FROM products`.
3. Mengembalikan **satu angka** (integer), misalnya `24`.

Mari kita bedah:

| Bagian | Arti |
|---|---|
| `Product` | Nama model (mewakili tabel `products`). |
| `::` | Operator "panggil method langsung di class", tanpa perlu `new Product()`. |
| `count()` | Method aggregation untuk hitung jumlah baris. |

### Prasyarat: Ada Model-nya

Sebelum bisa panggil `Product::count()`, kamu harus **pastikan dulu** model `Product` sudah ada. Begitu juga `User` dan `Order`.

Cek di folder:

```
app/Models/
├── Product.php    ← harus ada
├── User.php       ← harus ada (default Laravel)
└── Order.php      ← harus ada
```

Kalau belum ada, kamu bisa buat dengan:

```bash
php artisan make:model Product
php artisan make:model Order
```

(Model `User` biasanya sudah ada bawaan Laravel.)

### Catatan: Untuk Belajar, Data Tabel Bisa Pakai Seeder / Factory

Agar dashboard nanti menunjukkan angka yang masuk akal, tabel `products`, `users`, dan `orders` di database kamu harus **berisi data**.

Kalau tabel masih kosong, hasil `count()` akan `0`. Wajar.

Untuk belajar, kamu bisa isi data manual lewat form CRUD yang sudah dibuat di materi sebelumnya. Atau pakai **seeder** / **factory** (kalau sudah diajarkan). Tapi itu **di luar cakupan tahap ini**. Yang penting kamu **paham konsepnya dulu**.

---

## 5. Langkah 1: Ubah Method `index()` di DashboardController

Sekarang, mari kita ubah method `index()` di `DashboardController`.

Buka file:

```
app/Http/Controllers/Admin/DashboardController.php
```

Ubah isinya menjadi seperti ini:

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

        return 'Total Produk: ' . $totalProduk
              . ' | Total User: ' . $totalUser
              . ' | Total Order: ' . $totalOrder;
    }
}
```

### Yang Berubah dari Versi Tahap 2

Mari kita lihat **apa saja yang berubah** dibanding versi tahap 2:

| Bagian | Tahap 2 | Tahap 3 |
|---|---|---|
| `use App\Models\Product;` | tidak ada | **ditambahkan** (untuk impor model Product) |
| `use App\Models\User;` | tidak ada | **ditambahkan** (untuk impor model User) |
| `use App\Models\Order;` | tidak ada | **ditambahkan** (untuk impor model Order) |
| `use Illuminate\Http\Request;` | ada | **dihapus** (karena tidak dipakai) |
| Isi `index()` | `return 'Halo, ini halaman Dashboard Admin.';` | Hitung `count()` lalu kembalikan teks berisi angka |

---

## 6. Penjelasan Kode Baris per Baris

Sekarang mari kita bedah **bagian-bagian baru** dengan tenang.

### Bagian 1: Statement `use` untuk Impor Model

```php
use App\Models\Product;
use App\Models\User;
use App\Models\Order;
```

Fungsinya: **mengimpor** class model `Product`, `User`, dan `Order` supaya bisa dipakai di dalam file controller ini.

Bayangkan seperti kamu mau **meminjam buku** dari perpustakaan. Kamu tidak bisa langsung pakai buku itu; kamu harus **daftar/panggil dulu** bukunya dari rak.

`use App\Models\Product;` artinya: "Tolong ambilkan class `Product` dari folder `App\Models`."

Tanpa baris ini, saat kamu tulis `Product::count()`, Laravel akan bingung: "Maksudmu `Product` yang mana? Saya tidak kenal."

### Bagian 2: Variable `$totalProduk`, `$totalUser`, `$totalOrder`

```php
$totalProduk = Product::count();
$totalUser   = User::count();
$totalOrder  = Order::count();
```

Mari kita bedah satu baris sebagai contoh:

```php
$totalProduk = Product::count();
```

- `$totalProduk` = nama variable (kotak untuk menyimpan angka).
- `=` = simpan hasil ke variable.
- `Product::count()` = hitung jumlah baris di tabel `products`.

Setelah baris ini jalan, variable `$totalProduk` akan berisi **angka integer**, misalnya `24`.

Kenapa variable namanya `$totalProduk` (pakai Bahasa Indonesia)? Sebenarnya bebas, mau `$totalProducts`, `$jumlahProduk`, atau `$tp`. Tapi untuk belajar, **Bahasa Indonesia lebih mudah dibaca**. Nanti kalau sudah mahir, bisa ikut konvensi Inggris.

### Bagian 3: Penggabungan String dengan Titik (`.`)

```php
return 'Total Produk: ' . $totalProduk
      . ' | Total User: ' . $totalUser
      . ' | Total Order: ' . $totalOrder;
```

Di PHP, **titik (`.`)** adalah operator untuk **menggabungkan string** (disebut "concatenation").

Contoh sederhana:

```php
'Hello' . ' ' . 'World'   // hasilnya: "Hello World"
```

Kalau string digabung dengan **integer**, PHP akan otomatis mengubah integer itu jadi string:

```php
'Umur saya: ' . 25   // hasilnya: "Umur saya: 25"
```

Jadi baris di atas akan menghasilkan teks seperti:

```
Total Produk: 24 | Total User: 340 | Total Order: 87
```

Kenapa kita **bagi jadi 3 baris** dengan titik di depan? Supaya **mudah dibaca manusia**. Kita bisa juga tulis dalam 1 baris panjang, tapi kode jadi jelek. Pemecahan baris hanya soal gaya, hasil akhirnya sama.

### Alternatif: Pakai Curly Braces `{$var}` di String

Selain pakai titik, PHP punya cara lain: **string interpolation** dengan kurung kurawal:

```php
return "Total Produk: {$totalProduk} | Total User: {$totalUser} | Total Order: {$totalOrder}";
```

Perhatikan:
- Pakai **tanda kutip ganda** `"..."`, bukan kutip tunggal `'...'` (karena interpolation cuma jalan di kutip ganda).
- Variable ditulis langsung di dalam string, dibungkus `{$namaVariable}`.

Keduanya **benar**. Pilih yang menurutmu lebih enak dibaca. Di tahap ini, kita pakai **gabungan titik** dulu karena lebih jelas untuk pemula (kelihatan mana teks, mana variable).

---

## 7. Langkah 2: Uji Coba di Browser

Sekarang waktunya uji coba.

1. Pastikan server Laravel jalan (`php artisan serve`).
2. Buka browser ke:

```
http://localhost:8000/admin/dashboard
```

3. Seharusnya muncul teks seperti:

```
Total Produk: 24 | Total User: 340 | Total Order: 87
```

(Angkanya tergantung isi database kamu. Kalau tabel kosong, akan muncul `0`.)

### Kalau Ada Error

**Error 1: "Class 'App\Models\Product' not found"**

Penyebab: Model `Product` belum dibuat.

Solusi:

```bash
php artisan make:model Product
```

**Error 2: "Class 'App\Models\Order' not found"**

Penyebab: Model `Order` belum dibuat.

Solusi:

```bash
php artisan make:model Order
```

**Error 3: "SQLSTATE[42S02]: Base table or view not found: Table 'products' doesn't exist"**

Penyebab: Tabel `products` belum ada di database (migration belum dijalankan).

Solusi:

```bash
php artisan migrate
```

**Error 4: Angka selalu 0**

Penyebab: Tabel ada, tapi **kosong** (tidak ada data).

Solusi: Isi data manual lewat form CRUD, atau pakai seeder/factory (di luar cakupan tahap ini).

**Error 5: Method Illuminate\Database\Eloquent\Builder::count does not exist.**

Penyebab: Ada typo di penulisan `count()`. Misalnya `conut()` atau `Count()`.

Solusi: Pastikan persis `count()` (huruf kecil semua, ejaan benar).

---

## 8. Apa yang Sebenarnya Terjadi di Belakang Layar?

Saat kamu tulis `Product::count()`, Laravel akan **menerjemahkan** ke SQL dan menjalankannya di database.

### Alurnya

1. Browser membuka `http://localhost:8000/admin/dashboard`.
2. Laravel cari route dengan URL `/admin/dashboard`, ketemu: `[DashboardController::class, 'index']`.
3. Laravel panggil method `index()` di `DashboardController`.
4. Di dalam method, ada `Product::count()`.
5. Laravel **menerjemahkan** ke SQL:

```sql
SELECT COUNT(*) FROM products;
```

6. Laravel kirim SQL ini ke database (MySQL/PostgreSQL/SQLite).
7. Database eksekusi SQL, kembalikan **satu angka** (misal: `24`).
8. Laravel terima angka itu, simpan di variable `$totalProduk`.
9. Method `index()` gabung jadi teks, `return` ke browser.
10. Browser tampilkan teks ke user.

### Kenapa Ini Efisien?

Perhatikan bahwa database yang **menghitung** jumlah baris, bukan PHP. Database itu spesialis hal seperti ini, jauh lebih cepat dari PHP.

Bayangkan tabel `products` berisi **1 juta produk**.

- Kalau kita pakai `Product::all()->count()` (buruk!): PHP akan **ambil 1 juta baris** dari database ke memori, baru hitung. Lambat, boros memori.
- Kalau kita pakai `Product::count()` (baik): Database langsung hitung sendiri, kirim **1 angka** ke PHP. Cepat, irit.

Kesimpulan: **Selalu pakai `count()` langsung**, bukan `all()->count()` atau `get()->count()`. Ini best practice yang wajib diingat.

---

## 9. Eksperimen Tambahan (Opsional)

Supaya makin paham, cobalah **eksperimen kecil** berikut sebelum lanjut ke tahap 4.

### Eksperimen 1: Lihat SQL yang Dijalankan Laravel

Buka file:

```
app/Providers/AppServiceProvider.php
```

Di method `boot()`, tambahkan (sementara untuk belajar):

```php
public function boot()
{
    \DB::listen(function ($query) {
        logger($query->sql . ' | ' . json_encode($query->bindings));
    });
}
```

Sekarang, setiap kali Laravel menjalankan query, SQL-nya akan **ditulis ke log** di:

```
storage/logs/laravel.log
```

Buka dashboard di browser, lalu cek `laravel.log`. Kamu akan melihat baris seperti:

```
[2026-07-20 10:30:45] local.DEBUG: select count(*) as aggregate from `products` | []
[2026-07-20 10:30:45] local.DEBUG: select count(*) as aggregate from `users` | []
[2026-07-20 10:30:45] local.DEBUG: select count(*) as aggregate from `orders` | []
```

Itulah SQL asli yang Laravel kirim ke database. Bagus untuk **belajar debug**, tapi **jangan lupa hapus** kodenya setelah selesai belajar (sebelum production).

### Eksperimen 2: Pakai `dd()` untuk Lihat Variable

Coba ubah method `index()` sementara jadi:

```php
public function index()
{
    $totalProduk = Product::count();
    $totalUser   = User::count();
    $totalOrder  = Order::count();

    dd($totalProduk, $totalUser, $totalOrder);
}
```

`dd()` = **dump and die**. Laravel akan menampilkan isi variable di layar, lalu **menghentikan eksekusi**. Berguna untuk debugging.

Hasilnya kira-kira:

```
24    // ini isi $totalProduk
340   // ini isi $totalUser
87    // ini isi $totalOrder
```

Selesai eksperimen, **kembalikan kode ke versi normal** (hapus `dd()`).

---

## 10. Kesalahan Umum Pemula

Sebelum lanjut, mari kita lihat beberapa kesalahan yang sering dilakukan pemula saat belajar `count()`.

### Kesalahan 1: Pakai `all()->count()` (Buruk)

```php
// ❌ BURUK
$totalProduk = Product::all()->count();
```

Ini mengambil **semua data produk** dari database ke memori PHP dulu, baru menghitung jumlahnya. Boros dan lambat.

Yang benar:

```php
// ✅ BAIK
$totalProduk = Product::count();
```

### Kesalahan 2: Lupa `use App\Models\...`

```php
// ❌ ERROR
$totalProduk = Product::count();   // "Class 'Product' not found"
```

Penyebab: Lupa import model `Product`.

Yang benar:

```php
// ✅ BENAR
use App\Models\Product;            // ← di bagian atas file

$totalProduk = Product::count();
```

### Kesalahan 3: Bingung `Product` vs `products`

Pemula sering tanya: "Kenapa saya tulis `Product` (tunggal, kapital), padahal tabelnya namanya `products` (jamak, huruf kecil)?"

Jawaban: **Itu konvensi Laravel.**

- **Model** (di kode PHP): `Product` (tunggal, huruf kapital di awal).
- **Tabel** (di database): `products` (jamak, huruf kecil semua).

Laravel **otomatis** menghubungkan keduanya berdasarkan aturan penamaan. Jadi di kode kamu **selalu pakai `Product`** (nama model), bukan `products`.

### Kesalahan 4: Membungkus `count()` di Kurung yang Salah

```php
// ❌ SALAH (ini method call, bukan count)
Product::count;
```

`count` adalah **method**, jadi harus pakai kurung:

```php
// ✅ BENAR
Product::count();
```

---

## 11. Ringkasan Tahap 3

Mari kita rangkum apa yang sudah kita pelajari:

| Konsep | Penjelasan Singkat |
|---|---|
| **Aggregation Query** | Query yang **menghitung ringkasan** (bukan ambil data mentah). Contoh: jumlah baris, total nilai, rata-rata. |
| **`count()`** | Method Laravel untuk **menghitung jumlah baris** di tabel. |
| **Efisiensi** | `Product::count()` membuat **database yang menghitung**, bukan PHP. Cepat dan ringan. |
| **Syntax dasar** | `Product::count()` → kembalikan integer. |
| **Jangan lupa `use`** | Selalu import model: `use App\Models\Product;`. |
| **Jangan pakai `all()->count()`** | Boros memori, lambat. Pakai langsung `count()`. |

### Hasil Akhir Kode Tahap 3

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

        return 'Total Produk: ' . $totalProduk
              . ' | Total User: ' . $totalUser
              . ' | Total Order: ' . $totalOrder;
    }
}
```

### Apa yang Sudah Bisa Dashboard Kita Lakukan?

Saat ini dashboard sudah bisa **menghitung jumlah**:

- ✅ Total produk.
- ✅ Total user.
- ✅ Total order.

Yang **belum** bisa:

- ❌ Total pendapatan (butuh `sum()`).
- ❌ Produk aktif / nonaktif (butuh `count()` dengan `where()`).
- ❌ Order terbaru (butuh `latest()` + `take()`).

Itulah yang akan kita pelajari di **Tahap 4**.

---

## 12. Apa Selanjutnya?

Di **Tahap 4**, kita akan belajar dua aggregation query baru:

1. **`sum('kolom')`** untuk menghitung **total pendapatan** (menjumlahkan kolom `total` di tabel `orders`).
2. **`count()` dengan `where()`** untuk menghitung **produk aktif** dan **produk nonaktif** (filter berdasarkan kolom `is_active`).

Setelah Tahap 4, dashboard kita akan bisa menampilkan **6 angka**:

- Total produk
- Total user
- Total order
- **Total pendapatan** ← baru
- **Produk aktif** ← baru
- **Produk nonaktif** ← baru

Tapi tetap **tampilannya masih teks biasa** di browser. Kita baru akan buat tampilan cantik dengan kartu-kartu di **Tahap 6** (view blade).

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: belajar `sum()` untuk total pendapatan dan `where()` untuk produk aktif/nonaktif?**

Kalau **ya**, kita akan pelajari:

- Apa itu `sum()` dan cara kerjanya.
- Cara pakai `Order::sum('total')` untuk total pendapatan.
- Cara pakai `Product::where('is_active', 1)->count()` untuk produk aktif.
- Cara pakai `Product::where('is_active', 0)->count()` untuk produk nonaktif.

Kalau **belum**, kita bisa:

- Ulang penjelasan tentang aggregation query.
- Praktik ulang `count()` dengan berbagai tabel.
- Diskusi lebih dalam tentang konvensi Laravel (Model vs tabel).
- Coba eksperimen tambahan dengan `dd()` atau query log.

Tinggal bilang saja.
