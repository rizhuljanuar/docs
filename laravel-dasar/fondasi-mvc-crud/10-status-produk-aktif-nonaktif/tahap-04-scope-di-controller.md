# Tahap 4 — Pakai Scope `active()` di Controller: Pisahkan Halaman Publik vs Halaman Admin

> Materi: Status Produk Aktif/Nonaktif
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi: Toko dengan Dua Pintu

Di tahap 3, kita sudah bikin **filter air** (scope `active()`). Tapi filter itu **belum dipasang** di mana pun. Masih tersimpan di gudang (model).

Sekarang pertanyaannya:

> **Di mana kita harus pasang filter itu?**

Bayangkan kamu punya toko fisik dengan **dua pintu**:

| Pintu | Siapa yang lewat | Apa yang mereka lihat |
|---|---|---|
| **Pintu Depan** (ke mall) | Pengunjung umum (customer) | Hanya barang yang sudah siap dijual: ada harga, ada label, sudah disetrika |
| **Pintu Belakang** (ke gudang) | Hanya kamu dan staf | Semua barang, termasuk yang masih dalam kardus, belum disetrika, belum dilabeli |

Kamu **pasang filter** `active()` di **pintu depan** saja. Pengunjung tidak akan pernah melihat barang yang belum siap.

Tapi di **pintu belakang**, kamu **tidak pakai filter**. Karena kamu dan staf perlu melihat semua barang, termasuk yang sedang disiapkan, supaya bisa lanjut mengerjakan kapan saja.

Di Laravel, dua pintu itu = **dua halaman di controller**:

- **Halaman publik** (pintu depan) → pakai `Product::active()->get()`. User cuma lihat produk aktif.
- **Halaman admin** (pintu belakang) → pakai `Product::all()`. Admin lihat semua produk, aktif maupun nonaktif.

Di tahap 4 ini, kita kerjakan persis itu: pisahkan dua halaman, pasang scope di halaman yang tepat.

---

## 2. Recap: Apa Isi Controller Sekarang?

Buka file `app/Http/Controllers/ProductController.php`. Kira-kira isinya seperti ini (versi sederhana, alih-alih versi lengkap dari materi-materi sebelumnya):

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    // Halaman daftar produk
    public function index()
    {
        $products = Product::all();
        return view('products.index', compact('products'));
    }

    // Halaman detail satu produk
    public function show($slug)
    {
        $product = Product::where('slug', $slug)->firstOrFail();
        return view('products.show', compact('product'));
    }

    // Form tambah produk
    public function create()
    {
        return view('products.create');
    }

    // Simpan produk baru
    public function store(Request $request)
    {
        $data = $request->validate([
            'nama' => 'required',
            'harga' => 'required|integer',
            'stok' => 'required|integer',
            'deskripsi' => 'nullable',
            'kategori' => 'required',
            'slug' => 'required|unique:products',
            'gambar' => 'nullable|image',
        ]);

        Product::create($data);
        return redirect('/produk')->with('success', 'Produk berhasil ditambahkan.');
    }

    // ... method edit, update, destroy ada di bawahnya
}
```

**Bagian yang akan kita ubah di tahap 4**: hanya **method `index()`** dan **method `show()`**.

### Apa Masalah dengan `index()` Sekarang?

Lihat baris ini:

```php
$products = Product::all();
```

Artinya: "Ambil **semua** produk dari database."

Masalahnya: ini akan ambil **semua produk**, termasuk yang `is_active = 0`. Padahal di halaman publik, kita cuma mau tampilkan produk yang aktif. Produk nonaktif akan ikut muncul, dan user bisa lihat produk yang belum siap (gambar kosong, harga 0, deskripsi ngawur).

Itulah yang harus kita perbaiki.

---

## 3. Konsep: Halaman Publik vs Halaman Admin

Sebelum ubah kode, mari paham dulu **strateginya**. Kita akan punya **dua jenis halaman** di aplikasi:

### a. Halaman Publik (dilihat customer)

Ini halaman yang diakses pengunjung toko online. Contoh:

| URL | Fungsi |
|---|---|
| `/produk` | Daftar produk yang dijual |
| `/produk/{slug}` | Detail satu produk |

Di halaman ini, kita **hanya tampilkan produk aktif**. Caranya: pakai scope `active()`.

### b. Halaman Admin (dilihat admin toko)

Ini halaman khusus admin untuk mengelola produk. Contoh:

| URL | Fungsi |
|---|---|
| `/admin/produk` | Daftar semua produk (aktif + nonaktif) |
| `/admin/produk/create` | Form tambah produk |
| `/admin/produk/{id}/edit` | Form edit produk |

Di halaman ini, admin **bisa lihat semua produk**, termasuk yang masih draft (nonaktif). Karena admin perlu melanjutkan edit produk yang belum siap.

### Ringkasan Perbedaan

| Aspek | Halaman Publik | Halaman Admin |
|---|---|---|
| Yang boleh akses | Pengunjung umum | Hanya admin |
| Query di controller | `Product::active()->get()` | `Product::all()` atau `Product::paginate(10)` |
| Produk yang muncul | Hanya `is_active = 1` | Semua produk (aktif + nonaktif) |
| Tujuan | Customer beli produk | Admin kelola produk |

Di projek nyata, biasanya halaman admin ditaruh di route dengan prefix `/admin/...` dan diproteksi middleware (misal harus login sebagai admin). Tapi untuk materi ini, supaya fokus ke konsep status aktif, kita akan sederhanakan dulu: anggap saja satu controller `ProductController` saja, dan kita bikin **dua method berbeda** untuk dua halaman.

---

## 4. Langkah 1: Pisahkan Method `index()` untuk Halaman Publik dan Admin

Kita akan ubah controller supaya punya **dua method daftar produk**:

- `index()` → untuk halaman publik, pakai scope `active()`.
- `adminIndex()` → untuk halaman admin, tampilkan semua produk.

Edit `app/Http/Controllers/ProductController.php`:

```php
// Halaman publik: hanya produk aktif
public function index()
{
    $products = Product::active()->latest()->get();
    return view('products.index', compact('products'));
}

// Halaman admin: semua produk (aktif + nonaktif)
public function adminIndex()
{
    $products = Product::latest()->get();
    return view('products.admin-index', compact('products'));
}
```

### Penjelasan Baris Per Baris

#### `Product::active()->latest()->get()`

Ini baris baru di `index()`. Pecahannya:

| Bagian | Arti |
|---|---|
| `Product::active()` | Pakai scope yang kita bikin di tahap 3. Tambah kondisi `WHERE is_active = 1`. |
| `->latest()` | Urutkan berdasarkan tanggal terbaru (`created_at DESC`). Produk baru muncul paling atas. |
| `->get()` | Eksekusi query, kembalikan koleksi produk. |

**SQL yang dijalankan**:

```sql
SELECT * FROM products WHERE is_active = 1 AND deleted_at IS NULL ORDER BY created_at DESC;
```

Dua kondisi `WHERE` muncul otomatis: satu dari scope `active()`, satu dari trait `SoftDeletes`. Canggih.

#### `Product::latest()->get()`

Di `adminIndex()`, kita **tidak** pakai scope `active()`. Jadi semua produk ditampilkan, baik aktif maupun nonaktif.

**SQL yang dijalankan**:

```sql
SELECT * FROM products WHERE deleted_at IS NULL ORDER BY created_at DESC;
```

Hanya ada filter soft delete (yang otomatis dari trait). Produk nonaktif tetap muncul di halaman admin.

#### `latest()` Apa?

`latest()` adalah method query builder Laravel untuk **mengurutkan berdasarkan kolom `created_at` menurun** (DESC). Produk paling baru muncul paling atas.

Kebalikannya: `oldest()` = urutkan dari paling lama dulu.

Sebenarnya `latest()` sama saja dengan `orderBy('created_at', 'desc')`, tapi lebih pendek dan jelas maksudnya. Ini contoh baik dari Laravel: menyediakan shortcut untuk hal yang sering dipakai.

#### `return view('products.index', compact('products'))`

Kirim data `$products` ke view `resources/views/products/index.blade.php`. Ini standar, tidak ada perubahan dari materi sebelumnya.

#### `compact('products')` Apa?

`compact('products')` adalah fungsi PHP untuk membuat array asosiatif `['products' => $products]`. Sama saja dengan kalau kita tulis manual:

```php
return view('products.index', [
    'products' => $products,
]);
```

`compact()` cuma shortcut. Laravel sangat sering pakai ini.

---

## 5. Langkah 2: Ubah Method `show()` juga

Halaman detail produk (`/produk/{slug}`) juga **halaman publik**. Jadi kita harus pastikan user tidak bisa akses produk nonaktif lewat URL.

Bayangkan skenario: produk **"Tumbler Limited"** masih nonaktif (`is_active = 0`). Admin sudah input slug `tumbler-limited-edition`. Kalau user iseng ketik URL:

```
https://toko-kamu.test/produk/tumbler-limited-edition
```

Tanpa filter, user akan bisa lihat detail produk yang belum siap itu. Kebocoran!

Caranya: tambahkan scope `active()` di method `show()`.

```php
// Halaman detail produk (publik): hanya produk aktif
public function show($slug)
{
    $product = Product::active()
        ->where('slug', $slug)
        ->firstOrFail();

    return view('products.show', compact('product'));
}
```

### Penjelasan Perubahan

Sebelumnya:

```php
$product = Product::where('slug', $slug)->firstOrFail();
```

Sekarang:

```php
$product = Product::active()->where('slug', $slug)->firstOrFail();
```

Yang ditambahkan: **`Product::active()`** di awal query ranting.

**SQL yang dijalankan**:

```sql
SELECT * FROM products
WHERE is_active = 1
  AND deleted_at IS NULL
  AND slug = 'tumbler-limited-edition'
LIMIT 1;
```

Ada tiga kondisi sekarang: aktif, tidak dihapus, dan slug cocok.

### Apa yang Terjadi Kalau User Coba Akses Produk Nonaktif?

Misal user ketik URL produk yang `is_active = 0`:

1. Laravel menjalankan query di atas.
2. Karena `is_active = 0`, produk tidak masuk hasil query.
3. `->firstOrFail()` tidak menemukan produk.
4. Laravel otomatis lempar halaman **404 Not Found**.

User melihat halaman 404, menganggap produk itu tidak ada. Padahal di database produknya ada, hanya saja "disembunyikan" dari publik. Inilah cara yang benar.

---

## 6. Langkah 3: Daftarkan Route Baru

Sekarang kita punya dua method di controller (`index` dan `adminIndex`). Kita perlu daftarkan dua route di `routes/web.php`.

Buka file `routes/web.php`. Tambahkan route untuk halaman admin:

```php
use App\Http\Controllers\ProductController;

// Halaman publik (customer)
Route::get('/produk', [ProductController::class, 'index'])->name('produk.index');
Route::get('/produk/{slug}', [ProductController::class, 'show'])->name('produk.show');

// Halaman admin
Route::get('/admin/produk', [ProductController::class, 'adminIndex'])->name('admin.produk.index');
```

### Penjelasan

| Route | Method Controller | Tujuan |
|---|---|---|
| `GET /produk` | `index` | Halaman publik, hanya produk aktif |
| `GET /produk/{slug}` | `show` | Detail produk publik, hanya produk aktif |
| `GET /admin/produk` | `adminIndex` | Halaman admin, semua produk |

### Kenapa Pakai `->name(...)`?

Method `->name(...)` memberi **nama** ke route, supaya di view kita bisa panggil `route('produk.index')` alih-alih hardcode URL `/produk`. Ini praktik baik di Laravel, bikin kode fleksibel: kalau suatu hari URL berubah, kita tidak perlu ubah semua view.

### Catatan: Route `/produk/{slug}` Harus di Bawah `/produk` yang Statis

Pastikan urutan route seperti di atas:

```php
Route::get('/produk', ...);          // statis
Route::get('/produk/{slug}', ...);   // dinamis
```

**Jangan** dibalik. Kalau dibalik, Laravel akan mengira `/produk` adalah nilai `{slug}`, padahal `/produk` tidak punya slug. Akan error.

Untungnya Laravel cukup pintar mengenali route statis dulu, tapi selalu aman kalau kamu letakkan statis di atas dinamis.

---

## 7. Tes Alurnya: Eksperimen dengan Data

Sekarang ayo kita lihat efeknya dengan data nyata. Pastikan di tabel `products` kamu ada kombinasi produk aktif dan nonaktif. Misal:

| id | nama | slug | is_active |
|---|---|---|---|
| 1 | Kopi Susu Vanilla | kopi-susu-vanilla | **1** (aktif) |
| 2 | Teh Manis Hangat | teh-manis-hangat | **1** (aktif) |
| 3 | Tumbler Limited | tumbler-limited-edition | **0** (nonaktif) |
| 4 | Draft Produk Baru | draft-produk | **0** (nonaktif) |

Kalau belum punya kombinasi seperti ini, ubah dulu lewat Tinker:

```bash
php artisan tinker
```

```php
Product::find(1)->update(['is_active' => true]);
Product::find(2)->update(['is_active' => true]);
Product::find(3)->update(['is_active' => false]);
Product::find(4)->update(['is_active' => false]);
exit;
```

### Tes 1: Buka Halaman Publik

Buka browser, akses:

```
http://localhost:8000/produk
```

Yang muncul **hanya 2 produk**: Kopi Susu Vanilla dan Teh Manis Hangat. Tumbler dan Draft **tidak muncul**.

### Tes 2: Coba Akses Produk Nonaktif via URL

Sekarang coba ketik langsung URL produk nonaktif:

```
http://localhost:8000/produk/tumbler-limited-edition
```

Hasilnya: **halaman 404 Not Found**. User tidak bisa "mengakali" dengan mengetik URL. Mantap.

### Tes 3: Buka Halaman Admin

Sekarang buka halaman admin:

```
http://localhost:8000/admin/produk
```

Yang muncul **semua 4 produk** (asumsi view-nya sudah kamu bikin, kita akan bahas di tahap 5). Admin bisa lihat semua produk, termasuk yang nonaktif.

> **Catatan**: kalau view `products.admin-index` belum kamu bikin, akan muncul error "view not found". Itu wajar, kita akan bikin view-nya di tahap 5. Untuk sekarang, fokus dulu ke konsep pemisahan controller.

---

## 8. Hal-hal Penting yang Sering Bikin Pemula Bingung

### a. Jangan Pakai Scope di Halaman Admin

Salah umum: pemula pakai `Product::active()->get()` di **semua** halaman, termasuk admin. Akibatnya admin **tidak bisa lihat** produk yang nonaktif, padahal itu yang mau dia kelola.

**Aturan**: scope `active()` hanya untuk halaman yang dilihat customer. Halaman admin pakai `Product::all()` atau `Product::paginate(...)` tanpa scope.

### b. `firstOrFail()` vs `first()` di `show()`

Pakai `firstOrFail()`, bukan `first()`.

```php
$product = Product::active()->where('slug', $slug)->firstOrFail();  // BENAR
```

Kenapa? Karena kalau produk tidak ketemu (misal user akses produk nonaktif), `first()` akan return `null`. Lalu view error karena coba akses property `$product->nama` di null.

`firstOrFail()` akan otomatis lempar **404 Not Found** kalau tidak ketemu. Halaman error yang lebih ramah, dan tidak membocorkan info apakah produk ada tapi nonaktif.

### c. Scope Tidak Menghapus Data

Sering keliru: pemula mengira `Product::active()` akan **mengubah** produk jadi aktif. Bukan. Scope hanya **menyaring** query. Scope `active()` artinya "ambil yang aktif", bukan "jadikan aktif".

Untuk **mengubah** status produk, gunakan:

```php
$product->is_active = true;
$product->save();
```

Itu akan kita lakukan di tahap 6 (form update status).

### d. Pakai Scope untuk Detail, Bukan Hanya Index

Jangan lupa pakai scope di method `show()` juga (halaman detail). Bukan cuma `index()`. Kalau tidak, user bisa "akali" dengan mengetik URL langsung ke produk nonaktif.

### e. Halaman Admin Belum Diproteksi Login

Di tahap 4 ini, halaman `/admin/produk` bisa diakses siapa saja, termasuk customer. Ini belum aman. Di projek nyata, kita harus pakai **middleware** (misal `auth` dan `admin`) supaya hanya admin yang login bisa akses. Tapi materi keamanan itu di luar lingkup materi 10 ini. Untuk sekarang, kita fokus ke konsep status aktif dulu.

### f. `latest()` Itu Shortcut `orderBy`

`Product::latest()->get()` sama dengan `Product::orderBy('created_at', 'desc')->get()`. Lebih pendek, lebih jelas maksudnya. Tapi kalau mau urutkan berdasarkan kolom lain, pakai `orderBy`:

```php
Product::orderBy('nama')->get();           // urut nama A-Z
Product::orderBy('harga', 'asc')->get();   // urut harga termurah
```

---

## Ringkasan Tahap 4

| Hal | Isi |
|---|---|
| Tujuan | Pisahkan halaman publik (produk aktif) dan admin (semua produk) |
| Method `index()` | `Product::active()->latest()->get()` — untuk halaman publik |
| Method `adminIndex()` | `Product::latest()->get()` — untuk halaman admin |
| Method `show()` | Tambah `Product::active()->where('slug', $slug)->firstOrFail()` |
| Route baru | `GET /admin/produk` → `adminIndex()` |
| Efek | Produk nonaktif menghilang dari halaman publik (termasuk direct URL) |
| Catatan | Halaman admin belum diproteksi login (di luar lingkup materi 10) |

---

## Apa yang Sudah Bisa Kamu Lakukan Setelah Tahap 4?

Setelah tahap 4:

1. **Halaman `/produk`** hanya menampilkan produk aktif. Produk nonaktif tidak muncul.
2. **URL `/produk/{slug}` produk nonaktif** langsung 404. User tidak bisa "akali".
3. **Halaman `/admin/produk`** menampilkan semua produk (aktif + nonaktif), supaya admin bisa lihat produk yang masih draft.
4. **Controller** sudah punya pemisahan yang jelas antara konteks publik dan konteks admin.

**Yang belum bisa kamu lakukan sampai tahap ini:**

- Belum ada **tampilan visual** yang menunjukkan mana produk aktif dan mana yang nonaktif (belum ada badge di view).
- Belum ada **tombol** "Aktifkan" / "Nonaktifkan" untuk mengubah status produk dari halaman admin.

Itu yang akan kita kerjakan di tahap 5.

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: membuat view dengan badge status dan tombol aktifkan/nonaktifkan?**

Kalau iya, tahap 5 kita akan:

1. Bikin view `products/admin-index.blade.php`.
2. Tampilkan tabel semua produk dengan kolom **status** (badge hijau "Aktif" / merah "Nonaktif").
3. Tambahkan tombol **"Aktifkan"** dan **"Nonaktifkan"** di samping tiap produk.
4. Belum jalankan tombolnya dulu (itu tahap 6) — fokusnya dulu ke tampilan dan badge.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
