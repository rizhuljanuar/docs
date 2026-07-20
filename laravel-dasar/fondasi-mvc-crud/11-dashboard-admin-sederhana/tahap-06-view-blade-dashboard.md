# Tahap 6 — Membuat View `dashboard.blade.php`: Tampilkan Semua Data dengan Rapi

> Materi: Dashboard Admin Sederhana
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: Produk, User, Order

---

## 1. Pengantar Sederhana

Pada **Tahap 5**, kita sudah punya **semua data** yang dibutuhkan dashboard:

- 6 angka ringkasan (produk, user, order, pendapatan, produk aktif, produk nonaktif).
- 5 order terbaru (sebagai collection).

Tapi data itu hanya bisa dilihat lewat `dd()`, yang cuma cocok untuk **debugging**. Tidak bisa dilihat admin sungguhan, apalagi bos.

Di **Tahap 6** ini, kita akan **ganti `dd()`** dengan **view blade**, sehingga dashboard terlihat **cantik** seperti dashboard sungguhan.

### Analogi: Restoran dengan Dapur vs Ruang Makan

Bayangkan kamu punya restoran. Di **dapur**, semua bahan mentah sudah disiapkan: sayuran sudah dipotong, daging sudah direndam, nasi sudah matang.

Tapi kalau kamu **sajikan bahan mentah itu langsung ke pelanggan** (di atas meja, tanpa piring, tanpa presentasi), pelanggan akan **kabur**.

Yang benar: bahan mentah di **olah** jadi **makanan cantik di piring**, lalu disajikan di **ruang makan yang nyaman**.

| Dunia Laravel | Analogi Restoran |
|---|---|
| Data hasil query (`count`, `sum`, `latest`, dll) | Bahan mentah di dapur |
| **View blade** | **Olahan cantik di piring + ruang makan** |
| `dd($variable)` | Menumpuk bahan mentah ke meja (jelek!) |
| `return view(...)` | Menyajikan makanan di piring (cantik) |

Di tahap ini, kita "sajikan" data dashboard ke **piring yang cantik**: sebuah file HTML dengan layout rapi.

---

## 2. Apa Itu View Blade? (Singkat: Ingatan Ulang)

Sebelum mulai, mari kita ingat lagi apa itu View dan Blade dari materi CRUD Produk.

### View

**View** adalah bagian Laravel yang berisi **tampilan** yang dilihat user di browser. Isinya **HTML** (+ sedikit kode khusus untuk menampilkan data).

Di Laravel, file view disimpan di folder:

```
resources/views/
```

### Blade

**Blade** adalah mesin template Laravel yang membuat HTML menjadi **pintar**. Dengan Blade, kita bisa:

- Tampilkan variable: `{{ $nama }}`.
- Buat kondisi: `@if`, `@else`.
- Buat loop: `@foreach`.
- Pakai layout/inheritance: `@extends`, `@yield`.

File blade selalu berakhiran **`.blade.php`** (bukan cuma `.php`).

### Konvensi Penamaan

| Hal | Konvensi |
|---|---|
| Lokasi file | `resources/views/...` |
| Ekstensi | `.blade.php` |
| Folder bertitik | `admin.dashboard` = `admin/dashboard.blade.php` |
| Folder bertitik (sub) | `admin.products.index` = `admin/products/index.blade.php` |

---

## 3. Struktur File yang Akan Kita Buat

Di tahap 6 ini, kita akan buat **1 file baru** dan **edit 1 file**:

```
resources/views/
└── admin/
    └── dashboard.blade.php     ← BARU

app/Http/Controllers/Admin/
└── DashboardController.php     ← EDIT (ganti dd() dengan return view)
```

Setelah selesai, buka `http://localhost:8000/admin/dashboard` akan tampil **halaman dashboard cantik** dengan:

- 6 kartu angka.
- 1 tabel order terbaru.

---

## 4. Langkah 1: Update Controller — Kirim Data ke View

Pertama, kita ubah `DashboardController` agar **tidak lagi** pakai `dd()`, melainkan `return view(...)`.

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
        // Angka ringkasan
        $totalProduk = Product::count();
        $totalUser   = User::count();
        $totalOrder  = Order::count();
        $totalPendapatan = Order::sum('total');
        $produkAktif     = Product::where('is_active', 1)->count();
        $produkNonaktif  = Product::where('is_active', 0)->count();

        // Daftar order terbaru
        $orderTerbaru = Order::latest()->take(5)->get();

        // Kirim semua data ke view
        return view('admin.dashboard', compact(
            'totalProduk',
            'totalUser',
            'totalOrder',
            'totalPendapatan',
            'produkAktif',
            'produkNonaktif',
            'orderTerbaru'
        ));
    }
}
```

### Yang Berubah dari Versi Tahap 5

| Bagian | Tahap 5 | Tahap 6 |
|---|---|---|
| Akhir method `index()` | `dd($orderTerbaru);` | `return view(...)` |

7 baris variable tidak berubah. Yang berubah hanya **cara penyajian**.

---

## 5. Penjelasan: Cara Mengirim Data dari Controller ke View

Mari kita bedah bagian `return view(...)` baris per baris.

### Bagian 1: `return view('admin.dashboard', ...)`

```php
return view('admin.dashboard', ...);
```

- `view(...)` = fungsi Laravel untuk **memuat file view**.
- `'admin.dashboard'` = nama view, yang mengarah ke file `resources/views/admin/dashboard.blade.php`.
- Tanda titik (`.`) di `admin.dashboard` = pemisah folder. Jadi `admin.dashboard` = folder `admin` / file `dashboard.blade.php`.

Kenapa pakai titik, bukan slash (`/`)? Karena Laravel menggunakan konvensi **dot notation**. Lebih konsisten dan tidak masalah OS (Windows pakai `\`, Linux pakai `/`, titik aman di semua OS).

### Bagian 2: `compact(...)`

```php
compact(
    'totalProduk',
    'totalUser',
    ...
)
```

**`compact(...)`** adalah fungsi bawaan PHP (bukan Laravel) yang **membungkus beberapa variable menjadi 1 array asosiatif**.

#### Analogi: Packing Kado

Bayangkan kamu mau kirim **5 barang** ke teman lewat kurir. Kalau kamu kirim **satu-satu**, kamu butuh 5 paket.

Tapi kalau kamu **masukkan 5 barang ke 1 kotak kado**, kamu cukup kirim **1 paket**.

`compact(...)` itu seperti **kotak kado**. Ia masukkan semua variable yang kamu sebutkan jadi **satu kotak** (array), lalu kotak itu dikirim ke view.

#### Cara Kerja `compact()`

```php
$totalProduk = 120;
$totalUser   = 340;

compact('totalProduk', 'totalUser');
```

Akan menghasilkan array seperti ini:

```php
[
    'totalProduk' => 120,
    'totalUser'   => 340,
]
```

Perhatikan: **nama variable jadi key array**, dan **isi variable jadi value array**.

Array inilah yang dikirim ke view, lalu di view tiap variable bisa diakses langsung sebagai `{{ $totalProduk }}`, `{{ $totalUser }}`, dst.

### Alternatif: Pakai Array Manual

Selain `compact()`, kamu bisa tulis array manual:

```php
return view('admin.dashboard', [
    'totalProduk' => $totalProduk,
    'totalUser'   => $totalUser,
    // ... dst
]);
```

Hasilnya **sama**. `compact()` cuma jalan pintas yang lebih ringkas. Bebas mau pakai yang mana, tapi `compact()` lebih sering dipakai karena ** lebih ringkas dan tidak ada typo nama key**.

---

## 6. Langkah 2: Buat Folder dan File View

Sekarang buat folder dan file untuk view-nya.

### Buat Folder `admin/` di dalam `resources/views/`

Buka terminal, dari root project:

```bash
mkdir -p resources/views/admin
```

Atau pakai file explorer: buat folder `admin` di dalam `resources/views/`.

### Buat File `dashboard.blade.php`

Di dalam folder `resources/views/admin/`, buat file baru bernama:

```
dashboard.blade.php
```

(Jangan lupa ekstensi `.blade.php`, bukan `.php` biasa.)

---

## 7. Langkah 3: Isi `dashboard.blade.php` dengan Kode Sederhana

Kita akan mulai dari **versi paling sederhana**, lalu tambah kompleksitas bertahap. Supaya kamu **paham setiap bagian**, tidak sekadar niru kode panjang.

### Versi 1: Hanya Tampilkan 6 Angka (Tanpa CSS)

Salin kode berikut ke file `dashboard.blade.php`:

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Dashboard Admin</title>
</head>
<body>
    <h1>Dashboard Admin</h1>

    <h2>Statistik</h2>
    <ul>
        <li>Total Produk: {{ $totalProduk }}</li>
        <li>Total User: {{ $totalUser }}</li>
        <li>Total Order: {{ $totalOrder }}</li>
        <li>Total Pendapatan: Rp {{ number_format($totalPendapatan, 0, ',', '.') }}</li>
        <li>Produk Aktif: {{ $produkAktif }}</li>
        <li>Produk Nonaktif: {{ $produkNonaktif }}</li>
    </ul>
</body>
</html>
```

### Uji Coba Versi 1

1. Pastikan server jalan (`php artisan serve`).
2. Buka `http://localhost:8000/admin/dashboard`.

Hasilnya: halaman putih dengan judul "Dashboard Admin", dan daftar 6 statistik.

---

## 8. Penjelasan Kode Blade Baris per Baris

Mari kita bedah kode di atas.

### Bagian 1: HTML Dasar

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Dashboard Admin</title>
</head>
<body>
    ...
</body>
</html>
```

Ini adalah **kerangka HTML standar** yang wajib ada di setiap halaman web. Materi ini bukan tentang HTML, jadi kita tidak bahas mendalam. Intinya: ini wadah halaman web.

### Bagian 2: `{{ $variable }}` — Echo di Blade

```html
<li>Total Produk: {{ $totalProduk }}</li>
```

Perhatikan **kurung kurawal ganda** `{{ ... }}`. Itu adalah sintaks **Blade untuk echo**.

| Sintaks Blade | Setara dengan PHP |
|---|---|
| `{{ $totalProduk }}` | `<?= htmlspecialchars($totalProduk) ?>` |

Jadi `{{ $totalProduk }}` akan menampilkan isi variable `$totalProduk` (yang dikirim dari controller).

Perlu diketahui: Blade secara otomatis **mengamankan** output dengan `htmlspecialchars`, supaya karakter seperti `<`, `>`, `"`, `'` tidak dieksekusi sebagai HTML. Ini melindungi dari **XSS (Cross-Site Scripting)**, serangan keamanan umum.

### Bagian 3: `number_format()` untuk Format Angka

```html
<li>Total Pendapatan: Rp {{ number_format($totalPendapatan, 0, ',', '.') }}</li>
```

Di Tahap 4, `$totalPendapatan` berisi angka mentah seperti `15300000`. Itu **jelek** dibaca.

Kita pakai fungsi PHP `number_format()` untuk **memformat angka** jadi lebih enak dibaca.

#### Sintaks `number_format()`

```php
number_format($angka, $jumlah_desimal, $pemisah_desimal, $pemisah_ribuan);
```

Contoh:

```php
number_format(15300000, 0, ',', '.');
// Hasil: "15.300.000"
```

Penjelasan argumen:

| Posisi | Nilai | Arti |
|---|---|---|
| 1 | `15300000` | Angka yang mau diformat. |
| 2 | `0` | Tidak ada desimal (angka bulat). |
| 3 | `','` | Pemisah desimal pakai koma (kalau ada desimal). |
| 4 | `'.'` | Pemisah ribuan pakai titik (konvensi Indonesia). |

Jadi `Rp 15.300.000` jauh lebih enak dibaca daripada `Rp 15300000`.

---

## 9. Langkah 4: Tambahkan Tabel Order Terbaru

Sekarang mari kita tambahkan tabel untuk 5 order terbaru.

Update `dashboard.blade.php` menjadi:

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Dashboard Admin</title>
</head>
<body>
    <h1>Dashboard Admin</h1>

    <h2>Statistik</h2>
    <ul>
        <li>Total Produk: {{ $totalProduk }}</li>
        <li>Total User: {{ $totalUser }}</li>
        <li>Total Order: {{ $totalOrder }}</li>
        <li>Total Pendapatan: Rp {{ number_format($totalPendapatan, 0, ',', '.') }}</li>
        <li>Produk Aktif: {{ $produkAktif }}</li>
        <li>Produk Nonaktif: {{ $produkNonaktif }}</li>
    </ul>

    <h2>Order Terbaru</h2>
    @if ($orderTerbaru->count() > 0)
        <table border="1" cellpadding="8">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>User ID</th>
                    <th>Total</th>
                    <th>Status</th>
                    <th>Tanggal</th>
                </tr>
            </thead>
            <tbody>
                @foreach ($orderTerbaru as $order)
                    <tr>
                        <td>#{{ $order->id }}</td>
                        <td>{{ $order->user_id }}</td>
                        <td>Rp {{ number_format($order->total, 0, ',', '.') }}</td>
                        <td>{{ $order->status }}</td>
                        <td>{{ $order->created_at->format('d-m-Y H:i') }}</td>
                    </tr>
                @endforeach
            </tbody>
        </table>
    @else
        <p>Belum ada order.</p>
    @endif
</body>
</html>
```

### Uji Coba dengan Tabel

Refresh browser. Sekarang di bawah daftar statistik, akan muncul **tabel order terbaru** dengan 5 baris.

---

## 10. Penjelasan Kode Baru: `@if`, `@foreach`, Method Object

Mari kita bedah bagian-bagian baru.

### Bagian 1: `@if ... @else ... @endif`

```html
@if ($orderTerbaru->count() > 0)
    <!-- tampilkan tabel -->
@else
    <p>Belum ada order.</p>
@endif
```

Ini adalah **kondisi di Blade**, mirip `if` di PHP biasa.

Logikanya: "Kalau di collection `$orderTerbaru` ada **lebih dari 0** order, tampilkan tabel. Kalau tidak (0 order), tampilkan pesan 'Belum ada order.'"

Mengapa perlu `@if` ini? Supaya kalau tabel `orders` di database kosong, halaman tidak menampilkan **tabel kosong yang membingungkan**.

**Kenapa pakai `$orderTerbaru->count()`, bukan `Order::count()`?** Karena kita cuma mau tahu berapa order di collection `$orderTerbaru` (hasil `take(5)`), bukan total semua order di database. Lihat bahasan lengkapnya di Tahap 5.

### Bagian 2: `@foreach ... @endforeach`

```html
@foreach ($orderTerbaru as $order)
    <tr>
        <td>#{{ $order->id }}</td>
        ...
    </tr>
@endforeach
```

Ini adalah **loop di Blade**. Logikanya sama dengan `foreach` di PHP biasa:

```php
foreach ($orderTerbaru as $order) {
    echo $order->id;
}
```

**Cara baca**: "Untuk setiap order di `$orderTerbaru`, buat 1 baris tabel `<tr>`. Di tiap iterasi, variable `$order` berisi **object Order** yang sedang diproses."

Kalau `$orderTerbaru` berisi 5 order, maka loop jalan **5 kali**, dan hasilnya **5 baris `<tr>`** di tabel HTML.

### Bagian 3: Akses Property Object `$order->id`

```html
<td>#{{ $order->id }}</td>
<td>{{ $order->user_id }}</td>
<td>{{ $order->status }}</td>
```

Perhatikan `$order->id`. Tanda panah `->` di PHP adalah cara **mengakses property object**.

Bayangkan `$order` adalah **object Order** (satu baris dari tabel `orders`). Object ini punya beberapa "property" (atribut) sesuai kolom tabel:

| Property | Isi |
|---|---|
| `$order->id` | 1026 |
| `$order->user_id` | 88 |
| `$order->total` | 220000 |
| `$order->status` | "pending" |
| `$order->created_at` | (object Carbon tanggal) |

Jadi `$order->id` = akses property `id` dari object order yang sedang di-loop.

### Bagian 4: Method `format()` di Tanggal

```html
<td>{{ $order->created_at->format('d-m-Y H:i') }}</td>
```

Di Laravel, kolom `created_at` **otomatis** dikonversi menjadi **object Carbon** (library tanggal pintar). Object Carbon punya method `format(...)` untuk memformat tanggal.

`'d-m-Y H:i'` adalah format PHP standar:

| Huruf | Arti | Contoh |
|---|---|---|
| `d` | Tanggal 2 digit | 20 |
| `m` | Bulan 2 digit | 07 |
| `Y` | Tahun 4 digit | 2026 |
| `H` | Jam 24 jam | 10 |
| `i` | Menit | 30 |

Hasil: `20-07-2026 10:30`.

Tanpa `format()`, `$order->created_at` akan tampil sebagai `2026-07-20 10:30:00` (default SQL), yang agak kaku.

---

## 11. Langkah 5: Percantik dengan CSS (Opsional tapi Direkomendasikan)

Sekarang dashboard kita sudah **fungsi**, tapi tampilannya masih **polos** (background putih, teks hitam, tabel garis hitam).

Mari kita tambahkan **CSS sederhana** untuk membuatnya **lebih enak dilihat**.

### Versi 3: Tambahkan CSS Inline

Update `dashboard.blade.php`:

```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Dashboard Admin</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #f5f5f5;
            margin: 20px;
        }
        h1 {
            color: #333;
        }
        .stat-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
            margin-bottom: 30px;
        }
        .stat-card {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
        }
        .stat-label {
            color: #666;
            font-size: 14px;
            margin-bottom: 5px;
        }
        .stat-value {
            color: #2c3e50;
            font-size: 24px;
            font-weight: bold;
        }
        table {
            width: 100%;
            background: white;
            border-collapse: collapse;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
            border-radius: 8px;
            overflow: hidden;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #eee;
        }
        th {
            background: #2c3e50;
            color: white;
        }
        .status-pending {
            color: #e67e22;
            font-weight: bold;
        }
        .status-selesai {
            color: #27ae60;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1>Dashboard Admin</h1>

    <div class="stat-grid">
        <div class="stat-card">
            <div class="stat-label">Total Produk</div>
            <div class="stat-value">{{ $totalProduk }}</div>
        </div>
        <div class="stat-card">
            <div class="stat-label">Total User</div>
            <div class="stat-value">{{ $totalUser }}</div>
        </div>
        <div class="stat-card">
            <div class="stat-label">Total Order</div>
            <div class="stat-value">{{ $totalOrder }}</div>
        </div>
        <div class="stat-card">
            <div class="stat-label">Total Pendapatan</div>
            <div class="stat-value">Rp {{ number_format($totalPendapatan, 0, ',', '.') }}</div>
        </div>
        <div class="stat-card">
            <div class="stat-label">Produk Aktif</div>
            <div class="stat-value">{{ $produkAktif }}</div>
        </div>
        <div class="stat-card">
            <div class="stat-label">Produk Nonaktif</div>
            <div class="stat-value">{{ $produkNonaktif }}</div>
        </div>
    </div>

    <h2>Order Terbaru</h2>
    @if ($orderTerbaru->count() > 0)
        <table>
            <thead>
                <tr>
                    <th>ID</th>
                    <th>User ID</th>
                    <th>Total</th>
                    <th>Status</th>
                    <th>Tanggal</th>
                </tr>
            </thead>
            <tbody>
                @foreach ($orderTerbaru as $order)
                    <tr>
                        <td>#{{ $order->id }}</td>
                        <td>{{ $order->user_id }}</td>
                        <td>Rp {{ number_format($order->total, 0, ',', '.') }}</td>
                        <td>
                            @if ($order->status === 'pending')
                                <span class="status-pending">{{ $order->status }}</span>
                            @else
                                <span class="status-selesai">{{ $order->status }}</span>
                            @endif
                        </td>
                        <td>{{ $order->created_at->format('d-m-Y H:i') }}</td>
                    </tr>
                @endforeach
            </tbody>
        </table>
    @else
        <p>Belum ada order.</p>
    @endif
</body>
</html>
```

### Uji Coba Versi 3

Refresh browser. Sekarang dashboard akan tampil **cantik**:

- 6 kartu statistik berwarna putih dengan bayangan halus, dalam grid 3 kolom.
- Tabel order terbaru dengan header gelap, status berwarna (orange untuk pending, hijau untuk selesai).

### Catatan tentang CSS

CSS di atas adalah **CSS murni** (tanpa framework). Saya tidak pakai Bootstrap/Tailwind di sini supaya kamu **paham dasarnya dulu**. Di project sungguhan, kamu bisa pakai framework CSS favoritmu.

Beberapa teknik CSS yang dipakai:

- `display: grid` → layout grid untuk kartu statistik.
- `border-radius` → sudut membulat.
- `box-shadow` → bayangan halus.
- `border-collapse: collapse` → tabel tanpa garis ganda.

---

## 12. Uji Coba Akhir dan Verifikasi

Sekarang mari pastikan semua jalan dengan baik.

### Checklist Uji Coba

Buka `http://localhost:8000/admin/dashboard`. Pastikan:

| Hal yang Dicek | Ekspektasi |
|---|---|
| Halaman tampil tanpa error | ✅ Tidak ada pesan error Laravel |
| Judul "Dashboard Admin" | ✅ Muncul di atas |
| 6 kartu statistik muncul | ✅ Total Produk, User, Order, Pendapatan, Aktif, Nonaktif |
| Angka di kartu benar | ✅ Sesuai data di database (cek via phpMyAdmin / tinker) |
| Pendapatan diformat ribuan | ✅ Tampil `Rp 15.300.000` bukan `15300000` |
| Tabel order terbaru muncul | ✅ Maksimal 5 baris, urut dari terbaru |
| Tanggal diformat cantik | ✅ Tampil `20-07-2026 10:30` bukan `2026-07-20 10:30:00` |
| Status berwarna | ✅ Pending = orange, Selesai = hijau |
| Kalau tabel kosong, ada pesan | ✅ "Belum ada order." |

### Kalau Ada Error

**Error 1: "View [admin.dashboard] not found."**

Penyebab: File view tidak ada atau salah nama.

Solusi:
- Cek file ada di `resources/views/admin/dashboard.blade.php`.
- Pastikan ekstensi `.blade.php`, bukan `.php`.
- Jalankan `php artisan view:clear` untuk hapus cache view.

**Error 2: "Undefined variable: totalProduk"**

Penyebab: Variable tidak dikirim dari controller ke view.

Solusi:
- Cek `compact(...)` di controller, pastikan semua variable disebut.
- Hati-hati typo: `'totalProduk'` (besar K) vs `'totalproduk'` (kecil).

**Error 3: "Property [id] does not exist on this collection instance."**

Penyebab: Kamu akses `$orderTerbaru->id` di luar loop (mengira `$orderTerbaru` adalah satu order).

Solusi: Pakai `@foreach ($orderTerbaru as $order)` dulu, lalu di dalam loop akses `$order->id`.

**Error 4: "Method Illuminate\Database\Eloquent\Collection::format does not exist."**

Penyebab: Kamu menulis `$orderTerbaru->created_at->format(...)` (mengira collection punya `created_at`).

Solusi: Pakai `$order->created_at->format(...)` di dalam `@foreach`.

**Error 5: Tampilan masih polos meski sudah edit CSS**

Penyebab: Browser cache, atau salah edit file.

Solusi:
- Hard refresh: `Ctrl + Shift + R` (Windows/Linux) atau `Cmd + Shift + R` (Mac).
- Jalankan `php artisan view:clear`.

---

## 13. Kesalahan Umum Pemula

### Kesalahan 1: Lupa Ekstensi `.blade.php`

Kalau file hanya `dashboard.php` (bukan `dashboard.blade.php`):
- `{{ $var }}` tidak akan dikenali (muncul apa adanya di halaman).
- `@if`, `@foreach` tidak jalan.

Selalu pakai `.blade.php` untuk view Laravel.

### Kesalahan 2: `{{ }}` Pakai Satu Kurung

```html
<!-- ❌ SALAH (cuma 1 kurung) -->
<li>Total Produk: { $totalProduk }</li>
```

Hasilnya: literal `{ 120 }` di halaman (tidak diparsing).

Yang benar:

```html
<!-- ✅ BENAR (2 kurung) -->
<li>Total Produk: {{ $totalProduk }}</li>
```

### Kesalahan 3: Lupa `@endforeach` atau `@endif`

Blade **wajib** ditutup. Kalau lupa, akan error "Unexpected end of file".

Selalu cek pasangan:

- `@if` → `@endif`
- `@foreach` → `@endforeach`
- `@for` → `@endfor`

### Kesalahan 4: Salah Naruh Kode di Luar `@foreach`

```html
<!-- ❌ SALAH: $order tidak dikenal di luar foreach -->
<p>Total order terbaru: {{ $order->total }}</p>
@foreach ($orderTerbaru as $order)
    ...
@endforeach
```

Variable `$order` hanya **ada di dalam** `@foreach`. Di luar, variable itu tidak terdefinisi.

Yang benar (kalau mau akses order pertama):

```html
<!-- ✅ BENAR: pakai method ->first() -->
<p>Order paling baru: {{ $orderTerbaru->first()->total }}</p>
```

### Kesalahan 5: Kirim Data Tapi Tidak Dipakai di View

Di controller:

```php
return view('admin.dashboard', compact('totalProduk'));
```

Tapi di view tidak ada `{{ $totalProduk }}`. Tidak error, tapi **boros**. Pastikan semua data yang dikirim **dipakai di view**.

### Kesalahan 6: HTML Table Tidak Berstruktur

```html
<!-- ❌ SALAH: tr langsung di luar table -->
<tr>
    <td>...</td>
</tr>
```

Browser akan bingung. Selalu bungkus:

```html
<!-- ✅ BENAR -->
<table>
    <thead>
        <tr>...</tr>
    </thead>
    <tbody>
        <tr>...</tr>
    </tbody>
</table>
```

---

## 14. Eksperimen Tambahan (Opsional)

### Eksperimen 1: Tambah Kartu Statistik Baru

Coba tambah kartu **"Rata-rata Nilai Order"**. Di controller:

```php
$rataOrder = Order::avg('total');   // average = rata-rata
```

Kirim ke view, tampilkan sebagai kartu ke-7.

### Eksperimen 2: Link ke Detail Order

Tambahkan kolom "Aksi" di tabel:

```html
<tr>
    <td>#{{ $order->id }}</td>
    ...
    <td>
        <a href="/admin/orders/{{ $order->id }}">Lihat</a>
    </td>
</tr>
```

(Jangan lupa bikin route `/admin/orders/{id}` juga.)

### Eksperimen 3: Tampilkan Nama User, Bukan ID

Sekarang tabel hanya tampilkan `$order->user_id` (angka). Kurang informatif.

Pelajari **Eloquent Relationship** (Relasi) di materi lain, lalu tampilkan `$order->user->name`. Ini pakai relasi `belongsTo`.

### Eksperimen 4: Pakai `@forelse` (Lebih Keren dari `@foreach`)

Blade punya `@forelse` yang seperti `@foreach`, tapi dengan fallback `@empty`:

```html
@forelse ($orderTerbaru as $order)
    <tr>...</tr>
@empty
    <tr>
        <td colspan="5">Belum ada order.</td>
    </tr>
@endforelse
```

Lebih ringkas daripada `@if + @foreach + @else`.

### Eksperimen 5: Pisah CSS ke File Terpisah

Buat file `public/css/dashboard.css`, isi CSS-nya. Lalu di view:

```html
<link rel="stylesheet" href="{{ asset('css/dashboard.css') }}">
```

Lebih rapi untuk project besar.

---

## 15. Validasi: Apakah Dashboard Sudah Sesuai Tujuan?

Mari kita cek dengan **masalah awal** yang sudah kita bahas di Tahap 1:

| Masalah Tanpa Dashboard | Solusi Dashboard Kita |
|---|---|
| Cek jumlah produk manual | ✅ Kartu "Total Produk" |
| Cek jumlah user manual | ✅ Kartu "Total User" |
| Cek jumlah order manual | ✅ Kartu "Total Order" |
| Hitung total pendapatan manual | ✅ Kartu "Total Pendapatan" |
| Filter produk aktif manual | ✅ Kartu "Produk Aktif" |
| Filter produk nonaktif manual | ✅ Kartu "Produk Nonaktif" |
| Cek order terbaru satu-satu | ✅ Tabel "Order Terbaru" (5 terbaru) |

Semua **7 masalah awal** sudah teratasi dengan **satu halaman dashboard**.

Admin sekarang bisa tahu kondisi toko dalam **5 detik**, bukan 20 menit.

---

## 16. Ringkasan Tahap 6

Mari kita rangkum:

| Konsep | Penjelasan Singkat |
|---|---|
| **View** | Tempat menulis HTML yang dilihat user. File di `resources/views/`. |
| **Blade** | Mesin template Laravel: `{{ }}`, `@if`, `@foreach`. |
| **`view('nama')`** | Fungsi controller untuk memuat file view. |
| **`compact(...)`** | Bungkus beberapa variable jadi 1 array untuk dikirim ke view. |
| **`{{ $var }}`** | Tampilkan isi variable di Blade (echo aman dari XSS). |
| **`number_format()`** | Format angka ribuan (ex: 15300000 → 15.300.000). |
| **`@foreach`** | Loop untuk iterasi collection di view. |
| **`@if / @else / @endif`** | Kondisi di view. |
| **`->` (arrow)** | Akses property object (ex: `$order->id`). |
| **Carbon `format()`** | Format tanggal dari kolom `created_at`. |

### Hasil Akhir Kode Controller

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
        $totalPendapatan = Order::sum('total');
        $produkAktif     = Product::where('is_active', 1)->count();
        $produkNonaktif  = Product::where('is_active', 0)->count();

        $orderTerbaru = Order::latest()->take(5)->get();

        return view('admin.dashboard', compact(
            'totalProduk',
            'totalUser',
            'totalOrder',
            'totalPendapatan',
            'produkAktif',
            'produkNonaktif',
            'orderTerbaru'
        ));
    }
}
```

### Hasil Akhir Kode View (`dashboard.blade.php`)

Lihat **Versi 3** di Bagian 11 (versi dengan CSS dan tampilan cantik). Itu adalah versi final yang kita bangun di tahap ini.

### Apa yang Sudah Bisa Dashboard Kita Lakukan?

Sekarang dashboard **fungsional penuh** dari sisi fitur:

- ✅ 6 kartu statistik (angka ringkasan).
- ✅ Tabel 5 order terbaru.
- ✅ Tampilan cantik dengan CSS.
- ✅ Format angka ribuan.
- ✅ Format tanggal Carbon.
- ✅ Status berwarna.

Yang **tersisa** adalah **optimasi performa** untuk project skala besar. Itulah topik Tahap 7.

---

## 17. Apa Selanjutnya?

Di **Tahap 7 (terakhir)**, kita akan belajar **cache** untuk dashboard.

### Kenapa Cache Penting?

Bayangkan dashboard dibuka **100 kali per hari** oleh admin. Setiap kali dibuka, Laravel menjalankan **7 query** (6 aggregation + 1 data order terbaru). Itu artinya **700 query per hari** hanya untuk dashboard.

Kalau data sudah besar (ribuan produk, jutaan order), tiap query bisa makan waktu 100-500ms. Dashboard bisa **lambat** (2-3 detik setiap dibuka).

Solusinya: **cache**. Simpan hasil query selama beberapa menit, lalu pakai ulang. Dashboard jadi **cepat** (0.05 detik).

Yang akan kita pelajari di Tahap 7:

- Apa itu cache (dengan analogi sederhana).
- Cara pakai `Cache::remember(...)` di Laravel.
- Berapa lama cache sebaiknya disimpan (TTL).
- Kapan harus **invalidate** (hapus cache).
- Ringkasan keseluruhan materi Dashboard Admin + best practice.

Tahap 7 akan jadi **penutup** materi ini. Setelah itu, kamu sudah mahir membuat dashboard admin sederhana di Laravel.

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah terakhir: belajar cache untuk dashboard yang cepat + ringkasan keseluruhan materi?**

Kalau **ya**, kita akan pelajari:

- Konsep cache dengan analogi "lemari arsip pribadi".
- Cara pakai `Cache::remember('key', 60, function () {...})`.
- Berapa lama cache yang ideal untuk dashboard.
- Ringkasan semua tahap (1 sampai 7) sebagai cheat sheet.
- Tips dan best practice untuk production.

Kalau **belum**, kita bisa:

- Ulang penjelasan tentang Blade.
- Ulang penjelasan tentang `compact()`.
- Praktik menambah kartu statistik baru.
- Diskusi tentang layout inheritance (`@extends`, `@yield`).

Tinggal bilang saja.
