# Tahap 7 — Ringkasan Penuh Materi 10, Perbandingan dengan Soft Delete, dan Best Practice

> Materi: Status Produk Aktif/Nonaktif
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk
> Tahap: FINAL (terakhir dari 7 tahap)

---

## 1. Analogi Penutup: Dua Sistem yang Saling Melengkapi

Selamat! Kamu sudah menyelesaikan materi 10 dari awal sampai akhir. Sebelum masuk ke ringkasan teknis, mari kita tutup dengan analogi yang menyatukan semua yang sudah kamu pelajari.

Bayangkan kamu **pemilik toko online**. Di gudangmu ada ratusan produk. Untuk mengelola semuanya dengan aman, kamu punya **tiga sistem** yang saling melengkapi:

| Sistem | Analogi | Tujuan |
|---|---|---|
| **CRUD dasar** (materi 01-08) | Rak pajangan + meja kerja | Bikin, lihat, edit, hapus produk |
| **Soft delete** (materi 09) | Tempat sampah | Membuang produk tanpa benar-benar hilang, bisa diambil lagi |
| **Status aktif/nonaktif** (materi 10, ini) | Saklar lampu per produk | Mengontrol produk mana yang boleh dilihat customer |

Tiga sistem ini **bukan tandingan**, tapi **pelengkap**. Di projek nyata, ketiganya dipakai bersamaan. Produk aktif (saklar ON) dijual di rak depan. Produk nonaktif (saklar OFF) masih di gudang, belum siap. Produk yang sudah tidak relevan lagi di-soft-delete (masuk tong sampah) untuk berjaga-jaga. Produk yang benar-benar salah input atau melanggar di-force-delete (dibuang permanen).

Dengan kombinasi ini, kamu punya kontrol penuh atas visibility produk di toko online. Customer hanya melihat produk yang sudah siap, admin tetap punya akses ke semua produk untuk dikelola, dan tidak ada data yang hilang tanpa sengaja.

---

## 2. Peta Jalan Materi 10 (Recap Tahap 1 - 6)

Berikut ringkasan apa yang sudah kamu pelajari di tiap tahap:

| Tahap | Topik | Hasil |
|---|---|---|
| **1** | Konsep status aktif/nonaktif | Paham kenapa produk belum siap tidak boleh tampil di publik (gambar kosong, harga 0, stok 0, deskripsi ngawur) |
| **2** | Migration + kolom `is_active` | Tambah kolom `is_active` (boolean, default `false`) ke tabel `products` |
| **3** | Query scope `active()` | Bikin shortcut `Product::active()->get()` di model, supaya gampang ambil hanya produk aktif |
| **4** | Pakai scope di controller | Pisahkan halaman publik (`Product::active()`) vs halaman admin (`Product::all()`) |
| **5** | View admin dengan badge status | Halaman admin menampilkan semua produk dengan badge hijau "Aktif" / merah "Nonaktif" |
| **6** | Tombol aktifkan/nonaktifkan | Route + controller toggle `$product->is_active = !$product->is_active`, form PATCH |

### Alur Lengkap dari Sisi Admin

```
Admin input produk baru (create)
         ↓
Produk tersimpan dengan is_active = 0 (nonaktif, default)
         ↓
Admin melengkapi data: gambar, harga final, stok, deskripsi
         ↓
Admin buka halaman /admin/produk
         ↓
Admin klik tombol "Aktifkan" pada produk itu
         ↓
is_active berubah jadi 1 (aktif)
         ↓
Produk muncul di halaman publik /produk
         ↓
Suatu hari, produk stok habis / butuh revisi
         ↓
Admin klik tombol "Nonaktifkan"
         ↓
is_active berubah jadi 0 (nonaktif)
         ↓
Produk menghilang dari halaman publik
         ↓
... (bisa berulang, tanpa data hilang)
```

### Alur Lengkap dari Sisi Customer (Publik)

```
Customer buka /produk
         ↓
Hanya melihat produk dengan is_active = 1
         ↓
Customer klik salah satu produk
         ↓
Laravel cek: produk ini is_active = 1?
         ↓
   ┌──── YA ────┐                  ┌──── TIDAK ────┐
   ↓            ↓                  ↓                ↓
Tampilkan detail produk     Redirect 404 Not Found
(Customer happy)            (Customer tidak tahu produk itu ada)
```

---

## 3. Tabel Perbandingan Lengkap: Status Aktif/Nonaktif vs Soft Delete

Ini tabel referensi yang sering dicari pemula. Kapan pakai yang mana? Apa bedanya? Simpan tabel ini baik-baik.

| Aspek | Status Aktif/Nonaktif (materi 10) | Soft Delete (materi 09) |
|---|---|---|
| **Kolom database** | `is_active` (boolean: 0/1) | `deleted_at` (timestamp nullable) |
| **Trait Laravel** | Tidak ada (cuma query scope) | `use SoftDeletes;` |
| **Kapan dipakai?** | Saat admin bikin/edit produk | Saat admin klik "Hapus" |
| **Tujuan utama** | Kontrol produk mana yang tampil di publik | Cegah produk benar-benar hilang saat dihapus |
| **Analogi** | Saklar lampu ON/OFF | Tempat sampah |
| **Default produk baru** | `is_active = 0` (nonaktif, aman) | `deleted_at = NULL` (tidak dihapus) |
| **Produk muncul di `Product::all()`?** | **Ya**, semua produk muncul | **Hanya** yang `deleted_at`-nya NULL |
| **Produk muncul di `Product::active()->get()`?** | **Hanya** yang `is_active = 1` | **Ya**, semua produk yang belum dihapus |
| **Produk bisa diakses lewat URL langsung?** | Tidak (kalau nonaktif, 404) | Tidak (kalau sudah dihapus, 404) |
| **Bisa dibalik?** | Ya, tinggal klik tombol kebalikan | Ya, lewat `restore()` |
| **Cara membalik** | Set `is_active = !$product->is_active` lalu `save()` | `$product->restore()` |
| **Method query khusus** | Scope `active()` bikin sendiri | `withTrashed()`, `onlyTrashed()` bawaan trait |
| **Butuh migration?** | Ya, untuk tambah kolom `is_active` | Ya, untuk tambah kolom `deleted_at` |

### Kapan Pakai Yang Mana?

| Skenario | Pakai |
|---|---|
| Produk baru, belum siap dijual | **Status** (nonaktifkan, bukan hapus) |
| Produk stok habis, mau dijual lagi nanti | **Status** (nonaktifkan sementara) |
| Produk seasonal (kayak kue Lebaran, kue Natal) | **Status** (nonaktifkan di luar musim) |
| Produk sedang direvisi harganya | **Status** (nonaktifkan sampai harga final) |
| Produk tidak relevan lagi (gambar buruk, kualitas turun) | **Soft delete** (hapus dulu, jaga-jaga) |
| Produk recalled (cacat, bahaya) | **Soft delete** + mungkin force delete setelah yakin |
| Produk dibuat salah (input dobel, typo parah) | **Soft delete**, lalu force delete setelah pasti |
| Produk dummy / data testing | **Force delete** (hapus permanen, tidak perlu restore) |

### Saran Praktis

- **Mulai dari status aktif/nonaktif** untuk masalah "belum siap" atau "sementara tidak dijual". Ini paling umum.
- **Pakai soft delete** kalau produk sudah tidak relevan lagi tapi admin masih ragu (mungkin perlu audit, mungkin salah klik).
- **Force delete** cuma kalau 100% yakin tidak butuh data itu lagi. Setelah force delete, tidak ada jalan kembali.

---

## 4. Kode Lengkap (Cheat Sheet)

Untuk referensi cepat, berikut semua kode yang sudah kamu tulis sepanjang materi 10. Bisa kamu pakai sebagai cheat sheet saat mengerjakan projek nyata.

### a. Migration

```php
// database/migrations/xxxx_add_is_active_to_products_table.php

public function up(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->boolean('is_active')->default(false);
    });
}

public function down(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->dropColumn('is_active');
    });
}
```

### b. Model `Product`

```php
// app/Models/Product.php

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Builder;

class Product extends Model
{
    use SoftDeletes;

    protected $fillable = [
        'nama',
        'harga',
        'stok',
        'deskripsi',
        'kategori',
        'slug',
        'gambar',
        'is_active',
    ];

    public function scopeActive(Builder $query): Builder
    {
        return $query->where('is_active', true);
    }
}
```

### c. Controller `ProductController` (bagian relevan)

```php
// app/Http/Controllers/ProductController.php

use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    // Halaman publik: hanya produk aktif
    public function index()
    {
        $products = Product::active()->latest()->get();
        return view('products.index', compact('products'));
    }

    // Halaman detail produk publik: hanya produk aktif
    public function show($slug)
    {
        $product = Product::active()
            ->where('slug', $slug)
            ->firstOrFail();

        return view('products.show', compact('product'));
    }

    // Halaman admin: semua produk
    public function adminIndex()
    {
        $products = Product::latest()->get();
        return view('products.admin-index', compact('products'));
    }

    // Toggle status aktif/nonaktif
    public function updateStatus(Request $request, $id)
    {
        $product = Product::findOrFail($id);
        $product->is_active = !$product->is_active;
        $product->save();

        $status = $product->is_active ? 'aktif' : 'nonaktif';

        return redirect()->route('admin.produk.index')
            ->with('success', "Produk \"{$product->nama}\" sekarang {$status}.");
    }
}
```

### d. Routes

```php
// routes/web.php

use App\Http\Controllers\ProductController;

// Halaman publik
Route::get('/produk', [ProductController::class, 'index'])->name('produk.index');
Route::get('/produk/{slug}', [ProductController::class, 'show'])->name('produk.show');

// Halaman admin
Route::get('/admin/produk', [ProductController::class, 'adminIndex'])->name('admin.produk.index');

// Aksi admin: toggle status
Route::patch('/admin/produk/{id}/status', [ProductController::class, 'updateStatus'])
    ->name('admin.produk.status');
```

### e. View `admin-index.blade.php` (inti-nya)

```blade
{{-- Badge status --}}
@if($product->is_active)
    <span class="bg-green-100 text-green-800 ...">Aktif</span>
@else
    <span class="bg-red-100 text-red-800 ...">Nonaktif</span>
@endif

{{-- Tombol toggle --}}
<form action="{{ route('admin.produk.status', $product->id) }}" method="POST" style="display:inline;">
    @csrf
    @method('PATCH')
    @if($product->is_active)
        <button type="submit" onclick="return confirm('Yakin nonaktifkan?')">Nonaktifkan</button>
    @else
        <button type="submit" onclick="return confirm('Yakin aktifkan?')">Aktifkan</button>
    @endif
</form>
```

---

## 5. Best Practice di Projek Nyata

Sekarang kamu sudah paham dasarnya. Berikut beberapa **praktik yang umum dipakai di projek nyata**, sebagai referensi saat kamu membuat aplikasi sungguhan.

### a. Default `is_active = false` saat Create Form

Saat admin membuka form tambah produk, kasih opsi **"Simpan sebagai Draft"** (nonaktif) dan **"Simpan & Tampilkan"** (aktif). Jangan langsung aktif secara default. Ini mencegah produk setengah jadi bocor ke publik.

```blade
<form action="/admin/produk" method="POST">
    @csrf
    {{-- field-field produk lainnya --}}

    <label>
        <input type="checkbox" name="is_active" value="1">
        Tampilkan produk ini ke publik
    </label>

    <button type="submit">Simpan</button>
</form>
```

Di controller:

```php
public function store(Request $request)
{
    $data = $request->validate([
        'nama' => 'required',
        // ... validasi field lainnya
        'is_active' => 'boolean',  // terima true/false, default false
    ]);

    Product::create($data);
    return redirect()->route('admin.produk.index');
}
```

Karena kita sudah set `->default(false)` di migration, kalau checkbox tidak dicentang, `is_active` otomatis `false`. Aman.

### b. Tambah Indeks di Database untuk Kolom `is_active`

Kalau tabel `products` punya ribuan baris, query `WHERE is_active = 1` bisa lambat tanpa indeks. Tambahkan indeks di migration:

```php
$table->boolean('is_active')->default(false)->index();
```

Atau bikin migration baru kalau tabel sudah ada:

```php
Schema::table('products', function (Blueprint $table) {
    $table->index('is_active');
});
```

Indeks bikin pencarian produk aktif jadi **jauh lebih cepat** di tabel besar. Untuk projek belajar, tidak terasa efeknya. Tapi di produksi, ini penting.

### c. Pakai Enum atau Cast di Model (Opsional)

Beberapa developer suka pakai **enum** atau **accessor/mutator** supaya status lebih ekspresif. Contoh:

```php
// app/Models/Product.php

protected $casts = [
    'is_active' => 'boolean',
];
```

Dengan `$casts`, Laravel otomatis mengkonversi kolom `is_active` jadi boolean PHP saat dibaca, dan jadi integer `0/1` saat ditulis ke database. Jadi kamu bisa selalu pakai `true`/`false` di kode, tanpa peduli tipenya di database.

Setelah ditambah `$casts`, kode berikut selalu aman:

```php
if ($product->is_active) {  // selalu boolean true/false, bukan 0/1
    // ...
}
```

### d. Scope Tambahan untuk Kasus Lain

Setelah `scopeActive`, kamu bisa bikin scope lain sesuai kebutuhan:

```php
// Produk yang aktif DAN ada stoknya
public function scopeAvailable(Builder $query): Builder
{
    return $query->where('is_active', true)->where('stok', '>', 0);
}

// Produk yang aktif DAN murah (di bawah threshold)
public function scopeCheap(Builder $query, int $maxPrice = 50000): Builder
{
    return $query->where('is_active', true)->where('harga', '<', $maxPrice);
}
```

Cara pakai:

```php
Product::available()->get();           // produk aktif + ada stok
Product::cheap(10000)->get();          // produk aktif + di bawah Rp 10.000
Product::available()->cheap()->get();  // gabungan dua scope
```

Tapi ingat **YAGNI**: jangan bikin scope-spekulatif yang belum dipakai. Bikin scope **saat kamu menemukan diri kamu menulis query yang sama di tiga tempat berbeda**. Itu saatnya ekstrak jadi scope.

### e. Halaman Admin Wajib Diproteksi Middleware

Di materi ini, halaman `/admin/produk` bisa diakses siapa saja. **Di projek nyata, ini bahaya.** Tambahkan middleware autentikasi:

```php
// routes/web.php
Route::middleware(['auth', 'admin'])->group(function () {
    Route::get('/admin/produk', [ProductController::class, 'adminIndex'])->name('admin.produk.index');
    Route::patch('/admin/produk/{id}/status', [ProductController::class, 'updateStatus'])->name('admin.produk.status');
});
```

Middleware `auth` memastikan user sudah login. Middleware `admin` (custom) memastikan user yang login punya role admin. Pelajari ini di materi "Autentikasi & Otorisasi" Laravel.

### f. Pakai Activity Log untuk Audit

Di toko online nyata, setiap perubahan status produk sebaiknya **dicatat** (siapa yang ubah, kapan, dari status apa ke apa). Ini penting untuk audit. Pakai package seperti `spatie/laravel-activitylog` untuk ini. Di luar lingkup materi 10, tapi patut diingat untuk tahap berikutnya.

### g. Test Fitur Toggle

Sebelum deploy ke produksi, tulis test otomatis. Contoh test sederhana:

```php
// tests/Feature/ProductStatusTest.php

public function test_admin_can_activate_product()
{
    $product = Product::factory()->create(['is_active' => false]);

    $this->patch(route('admin.produk.status', $product->id));

    $this->assertTrue($product->fresh()->is_active);
}
```

Pelajari testing di materi "Testing Laravel". Buat pemula, fokus dulu ke fiturnya. Tapi begitu projek mulai serius, testing itu investasi.

---

## 6. Studi Kasus: Kombinasi Soft Delete + Status Aktif

Mari kita lihat satu skenario nyata yang menggabungkan kedua sistem. Misal kamu jualan **"Kopi Susu Vanilla"**:

### Bulan 1: Produk Baru Dibuat

- Admin input produk, **belum upload gambar**, **deskripsi masih kosong**.
- Default: `is_active = 0` (nonaktif), `deleted_at = NULL` (tidak dihapus).
- Produk **tidak tampil** di halaman publik. Customer tidak bisa lihat.
- Admin tetap bisa lihat produk ini di halaman admin dengan badge merah "Nonaktif".

### Bulan 1 (minggu kedua): Produk Siap Dijual

- Admin sudah upload gambar, lengkapi deskripsi, set harga final.
- Admin klik **"Aktifkan"** di halaman admin.
- Sekarang: `is_active = 1` (aktif), `deleted_at = NULL`.
- Produk **muncul** di halaman publik. Customer bisa lihat dan beli.

### Bulan 3: Stok Habis Sementara

- Supplier telat kirim barang. Stok habis.
- Admin klik **"Nonaktifkan"** supaya customer tidak order sesuatu yang tidak bisa dikirim.
- Sekarang: `is_active = 0` (nonaktif), `deleted_at = NULL`.
- Produk **menghilang** dari halaman publik. Customer tidak bisa lihat.
- Data produk tetap aman di database. Begitu stok datang, tinggal aktifkan lagi.

### Bulan 6: Produk Tidak Lagi Dijual

- Putusan bisnis: produk ini **diskontinyu** (tidak dijual lagi selamanya).
- Admin klik tombol **"Hapus"** (soft delete, dari materi 09).
- Sekarang: `is_active = 0` (nonaktif), `deleted_at = 2026-12-15 10:00:00` (terisi).
- Produk **menghilang** dari halaman admin utama, muncul di halaman tong sampah.
- Tapi data tetap aman, untuk jaga-jaga ada audit atau laporan historis.

### Bulan 12: Benar-benar Dibuang

- Setelah setahun, produk itu tidak pernah dibutuhkan lagi. Mau benar-benar dihapus untuk menghemat space database.
- Admin buka halaman tong sampah, klik **"Hapus Permanen"** (force delete dari materi 09).
- Sekarang: baris produk **hilang dari database selamanya**.
- Tidak bisa di-restore. Data benar-benar pergi.

### Pelajaran dari Studi Kasus

Di projek nyata, **siklus hidup produk melibatkan banyak status dan keputusan**. Tidak ada sistem tunggal yang cukup. Kombinasi:

- **CRUD dasar** untuk create/edit.
- **Status aktif/nonaktif** untuk kontrol publikasi.
- **Soft delete** untuk penghapusan sementara yang aman.
- **Force delete** untuk pembersihan permanen.

Dengan kombinasi ini, kamu punya **fleksibilitas penuh** mengelola produk di berbagai tahap dan situasi.

---

## 7. Istilah Penting yang Sudah Kamu Pelajari

Berikut glossary istilah yang muncul di sepanjang materi 10:

| Istilah | Arti Singkat |
|---|---|
| **Status produk** | Kondisi produk: aktif / nonaktif |
| **State sederhana** | Keadaan dengan pilihan minimal (cuma dua) |
| **Boolean** | Tipe data dengan dua nilai: true/false (atau 1/0 di database) |
| **`is_active`** | Nama kolom status aktif produk. Konvensi prefix `is_` = "apakah..." |
| **Default value** | Nilai otomatis kalau tidak diisi manual (misal `default(false)` = default nonaktif) |
| **Query scope** | Shortcut untuk query yang sering dipakai. Method di model diawali `scope` |
| **Method chaining** | Rantai method (`Product::active()->latest()->get()`) |
| **Halaman publik** | Halaman yang dilihat customer, hanya produk aktif |
| **Halaman admin** | Halaman yang dilihat admin, semua produk |
| **Badge** | Label kecil berwarna di UI yang menunjukkan status (hijau aktif, merah nonaktif) |
| **Toggle** | Membalikkan nilai boolean (`!$product->is_active`) |
| **Operator `!`** | Operator NOT di PHP, membalik boolean |
| **Operator ternary `? :`** | Versi singkat `if-else`: `$x ? 'a' : 'b'` |
| **Mass assignment** | Mengisi banyak field sekaligus lewat array (`Product::create([...])`) |
| **`$fillable`** | Daftar field yang boleh di-mass-assign di model |
| **`$casts`** | Konversi tipe otomatis di model (misal `'is_active' => 'boolean'`) |
| **Method HTTP PATCH** | Method untuk update sebagian field (semantik update) |
| **`@method('PATCH')`** | Directive Blade untuk override method HTML form jadi PATCH |
| **Flash session** | Data session sekali baca, otomatis hilang (`->with('success', ...)`) |
| **Route-model binding** | Laravel otomatis inject objek model berdasarkan ID di URL |

---

## 8. Referensi Lanjutan

Setelah materi 10, kamu sudah punya fondasi yang kuat tentang MVC, CRUD, soft delete, dan status aktif/nonaktif. Berikut beberapa topik yang patut dipelajari berikutnya:

| Topik | Kenapa Penting |
|---|---|
| **Eloquent Relationships** | Hubungan antar tabel (produk punya banyak review, kategori punya banyak produk) |
| **Middleware & Authentication** | Proteksi halaman admin, login user |
| **Authorization & Policies** | Siapa boleh lakukan apa (admin vs customer vs guest) |
| **Form Request Validation** | Validasi terpisah dari controller (lebih rapi) |
| **Database Indexing** | Optimasi query untuk tabel besar |
| **Laravel Testing** | Test otomatis supaya yakin fitur tetap jalan setelah refactor |
| **Activity Log** | Catat siapa melakukan apa (audit trail) |
| **Eloquent Events & Observers** | Trigger aksi otomatis saat produk dibuat/diedit/dihapus |
| **API Resources** | Kalau mau bikin API (bukan cuma halaman web) |
| **Queue & Jobs** | Pemrosesan background (misal kirim email saat produk baru diaktifkan) |

Tidak perlu belajar semua sekaligus. Pilih satu yang paling relevan dengan projek yang sedang kamu kerjakan.

### Dokumentasi Resmi Laravel

- Eloquent ORM: https://laravel.com/docs/eloquent
- Query Scopes: https://laravel.com/docs/eloquent#query-scopes
- Migrations: https://laravel.com/docs/migrations
- Soft Deleting: https://laravel.com/docs/eloquent#soft-deleting
- Blade Templates: https://laravel.com/docs/blade

---

## 9. Apa yang Bisa Kamu Banggakan di Akhir Materi 10?

Setelah menyelesaikan materi ini, kamu sekarang **bisa**:

1. **Menjelaskan konsep** status aktif/nonaktif dengan analogi yang jelas ke orang lain.
2. **Mendesain struktur database** dengan kolom `is_active` dan default yang aman.
3. **Membuat query scope sendiri** di model Laravel, bukan cuma pakai bawaan.
4. **Menggunakan method chaining** dengan lancar (`Product::active()->latest()->get()`).
5. **Memisahkan halaman publik dan admin** dengan logika yang berbeda di controller.
6. **Membuat view dengan badge dinamis** yang warnanya berdasarkan data.
7. **Membuat form toggle** dengan method HTTP PATCH dan `@method` directive.
8. **Men-debug** masalah umum seperti 405 Method Not Allowed, 404 di produk nonaktif, badge tidak muncul.
9. **Membedakan** kapan pakai status aktif/nonaktif vs soft delete vs force delete.
10. **Memahami** bagaimana sistem ini berpadu dengan soft delete dari materi 09.

Itu banyak sekali konsep yang sudah kamu kuasai. Kamu sudah naik level dari pemula Laravel ke pemula Laravel yang **paham fondasi**.

---

## 10. Pesan Penutup

Sekarang kamu sudah punya dua sistem pengelolaan produk yang lengkap:

- **Status aktif/nonaktif** (materi 10) untuk kontrol publikasi.
- **Soft delete** (materi 09) untuk penghapusan yang aman.

Keduanya adalah **fondasi** yang akan kamu pakai terus-menerus di projek Laravel berikutnya. Konsepnya tidak hanya berlaku untuk produk. Di projek nyata, kamu akan menemui:

- Status aktif/nonaktif untuk **user**, **artikel**, **kategori**, **promo**.
- Soft delete untuk **transaksi**, **review**, **komentar**, **file upload**.

Pola berpikirnya sama. Kolom status boolean + scope. Kolom `deleted_at` + trait. Method chaining. Form PATCH dengan `@method`. Semua sudah kamu pelajari.

**Saran terakhir**: jangan cuma baca materi ini. **Praktikkan di projek kamu sendiri.** Buat projek kecil (toko sederhana, blog pribadi, daftar tugas). Terapkan status aktif/nonaktif di sana. Temukan sendiri masalah yang muncul, dan pecahkan. Belajar Laravel paling efektif adalah **dengan membangun sesuatu yang kamu peduli**.

Selamat menyelesaikan materi 10, dan selamat melanjutkan ke topik Laravel berikutnya.

---

## Ringkasan Akhir Materi 10

| Hal | Isi |
|---|---|
| Tujuan materi | Mengontrol produk mana yang tampil di halaman publik |
| Konsep inti | State sederhana: aktif / nonaktif (boolean `is_active`) |
| Komponen yang dipelajari | Migration, model + scope, controller (publik vs admin), view + badge, form toggle |
| File yang dibuat/diubah | Migration baru, `app/Models/Product.php`, `app/Http/Controllers/ProductController.php`, `routes/web.php`, `resources/views/products/admin-index.blade.php` |
| Kompatibel dengan | Soft delete (materi 09). Dua sistem saling melengkapi |
| Kunci sukses | Default `false` (aman), scope di halaman publik, badge visual di admin, toggle lewat form PATCH |
| Selanjutnya | Eloquent relationships, authentication, testing, dan banyak lagi |

---

**Materi 10: Status Produk Aktif/Nonaktif - SELESAI.**

> Selamat! Kamu sudah menyelesaikan 10 dari topik-topik fondasi Laravel. Lanjutkan ke materi berikutnya sesuai silabus, atau ulik sendiri topik yang menarik minatmu. Semoga perjalanan belajar Laravel kamu menyenangkan.
