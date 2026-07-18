# Tahap 3 — Membuat Query Scope `active()` di Model Product

> Materi: Status Produk Aktif/Nonaktif
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi: Filter Air di Galon

Bayangkan kamu punya galon air berisi **3 liter air campuran**: ada air bersih, ada air kotor, ada air sabun.

Sekarang kamu mau minum. Pertanyaannya:

> **Apakah kamu akan minum langsung dari galon itu?**

Jawaban waras: **TIDAK**. Kamu akan **saring dulu** airnya lewat filter, supaya yang keluar hanya air bersih. Air kotor dan sabun tetap di galon, tapi tidak masuk ke gelasmu.

Nah, di Laravel, **query scope** itu seperti **filter air**:

- **Galon** = tabel `products` (semua produk, aktif dan nonaktif).
- **Filter** = query scope `active()`.
- **Gelas yang terisi air bersih** = hasil query, hanya produk aktif.

Tanpa filter, kamu akan dapat semua air (semua produk). Dengan filter `active()`, kamu hanya dapat produk yang `is_active = 1`. Produk nonaktif tetap ada di database, tapi tidak ikut keluar.

Lebih keren lagi: kamu bisa pakai filter `active()` ini **kapan saja, di mana saja**, tanpa harus nulis ulang kode filternya. Cukup panggil namanya: `Product::active()->get()`.

---

## 2. Masalah yang Kita Ingin Selesaikan

Sebelum belajar scope, mari lihat **masalah** dulu supaya kamu paham kenapa scope ini berguna.

Di tahap 2, kita sudah punya kolom `is_active` di tabel `products`. Sekarang bayangkan kita mau **mengambil hanya produk yang aktif**. Tanpa scope, kodenya seperti ini:

```php
$products = Product::where('is_active', true)->get();
```

Baris itu artinya: "Ambil semua produk dari tabel, **tapi hanya yang** `is_active`-nya `true`."

Sekarang bayangkan kode ini ditulis di **banyak tempat**:

```php
// Di controller ProductController (halaman publik)
public function index()
{
    $products = Product::where('is_active', true)->get();
    return view('products.index', compact('products'));
}

// Di controller CartController (halaman keranjang)
public function index()
{
    $products = Product::where('is_active', true)->get();
    // ...
}

// Di controller SearchController (halaman pencarian)
public function search($keyword)
{
    $products = Product::where('is_active', true)
        ->where('nama', 'LIKE', "%{$keyword}%")
        ->get();
    // ...
}
```

### Masalahnya:

1. **Kode berulang**: `where('is_active', true)` ditulis berkali-kali di banyak tempat.
2. **Rawan salah**: kalau suatu hari kita mau ubah logika (misalnya: produk aktif = `is_active = 1` DAN `stok > 0`), kita harus **mengubah di semua tempat**. Kalau ada satu tempat lupa diubah, muncul bug.
3. **Sulit dibaca**: kode jadi panjang dan berisik. Maksud sebenarnya ("ambil produk aktif") tidak langsung kelihatan.

### Solusinya:

Bungkus logika "ambil produk aktif" jadi **satu nama pendek**, simpan di model `Product`, lalu panggil nama itu di mana saja. Nama pendek itu namanya **query scope**.

Setelah pakai scope, kode di atas berubah jadi:

```php
Product::active()->get();
```

Lebih pendek, lebih jelas, dan kalau logikanya berubah, cukup ubah di **satu tempat** (di definisi scope).

---

## 3. Apa Itu Query Scope di Laravel?

**Query scope** = **shortcut untuk query yang sering dipakai**.

Istilah teknisnya:

- **Query** = perintah ambil data dari database (`Product::where(...)->get()`).
- **Scope** = "ruang lingkup" atau "cakupan". Dalam konteks ini: "cakupan query yang sudah diberi nama".

Jadi **query scope** = query yang sudah dibungkus dengan nama, supaya bisa dipanggil ulang dengan mudah.

### Analogi Lagi: Tombol "Air Dingin" di Dispenser

Dispenser air rumahan biasanya punya dua tombol: **air biru** (dingin) dan **air merah** (panas).

Saat kamu tekan tombol biru, kamu tidak perlu tahu di dalamnya ada kompresor apa, ada pendingin apa, ada pipa bagaimana. Cukup tekan tombol biru, keluar air dingin.

**Query scope itu seperti tombol biru itu.** Di dalamnya ada logika rumit (filter, kondisi), tapi yang kamu lihat dari luar cuma nama pendek (`active()`). Kamu tinggal pakai.

### Cara Pakai Scope (dari Sudut Pandang User)

Setelah scope didefinisikan, kamu bisa pakai di controller, tinker, atau tempat lain:

```php
// Ambil semua produk aktif
$products = Product::active()->get();

// Ambil produk aktif yang harga di bawah 50000
$products = Product::active()->where('harga', '<', 50000)->get();

// Hitung jumlah produk aktif
$count = Product::active()->count();

// Ambil produk aktif, urutkan berdasarkan nama
$products = Product::active()->orderBy('nama')->get();
```

Perhatikan pola **ranting method** (method chaining). Setelah `Product::active()`, kita bisa sambung dengan method query lain (`where`, `orderBy`, `count`, dst). Scope tidak menghalangi ranting method. Justru scope adalah bagian dari rantai.

### Aturan Nama Scope

Laravel punya **konvensi** (aturan tidak tertulis tapi disepakati) untuk nama scope:

- Nama method di model **harus** diawali kata `scope`.
- Saat dipanggil, kata `scope` **dihapus**, sisanya jadi nama scope yang dipakai.

Contoh:

| Nama method di model | Cara panggil di luar |
|---|---|
| `scopeActive` | `Product::active()` |
| `scopePopular` | `Product::popular()` |
| `scopeCheap` | `Product::cheap()` |
| `scopeInStock` | `Product::inStock()` |

Konvensi `scope` ini wajib, karena Laravel memakainya untuk membedakan: mana method biasa, mana yang bisa dipakai sebagai scope di query.

---

## 4. Langkah 1: Tambahkan `is_active` ke `$fillable`

Sebelum bikin scope, kita perlu menyelesaikan satu hal kecil dari tahap 2 yang sengaja saya tunda.

Buka file `app/Models/Product.php`. Saat ini isinya kira-kira seperti ini:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

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
    ];
}
```

Sekarang **tambahkan `'is_active'`** ke daftar `$fillable`:

```php
protected $fillable = [
    'nama',
    'harga',
    'stok',
    'deskripsi',
    'kategori',
    'slug',
    'gambar',
    'is_active',  // ← tambahan
];
```

### Kenapa Perlu Ditambahkan ke `$fillable`?

Di Laravel, **mass assignment** (mengisi banyak field sekaligus lewat array) hanya boleh dilakukan untuk field yang ada di `$fillable`. Contoh mass assignment:

```php
Product::create([
    'nama' => 'Kopi Susu Vanilla',
    'harga' => 18000,
    'is_active' => true,
]);
```

Kalau `is_active` tidak ada di `$fillable`, Laravel akan **mengabaikan** field itu saat mass assignment demi keamanan. Jadi walaupun kamu tulis `'is_active' => true` di atas, produk tetap dibuat dengan `is_active = 0` (default database).

**Catatan keamanan**: menambahkan `is_active` ke `$fillable` berarti **user bisa mengisi field ini lewat form**. Nanti di tahap 5/6, kita akan pastikan form yang dikirim user **benar-benar dimaksudkan** untuk ubah status, bukan baju musang yang menyusup dari form lain. Tapi untuk sekarang, kita izinkan dulu supaya CRUD bisa jalan.

---

## 5. Langkah 2: Buat Query Scope `active()` di Model Product

Sekarang waktunya bikin scope-nya. Di file `app/Models/Product.php` yang sama, tambahkan method `scopeActive` di dalam class:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Builder;  // ← tambahan (untuk tipe hint)

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

    // Scope: ambil hanya produk yang aktif
    public function scopeActive(Builder $query): Builder
    {
        return $query->where('is_active', true);
    }
}
```

### Penjelasan Baris Per Baris

#### `use Illuminate\Database\Eloquent\Builder;`

Baris ini **mengimpor** class `Builder`. Builder itu seperti "pembangun query" di Laravel, objek yang menyimpan kondisi query sebelum akhirnya dieksekusi dengan `->get()` atau `->count()`.

Kita pakai `Builder` di sini sebagai **tipe parameter** di method `scopeActive`. Ini cuma anotasi (penanda) supaya IDE dan pembaca kode jelas bahwa parameter `$query` itu adalah objek Builder. Laravel yang akan otomatis mengirim objek Builder ke scope kita.

#### `public function scopeActive(Builder $query): Builder`

Ini deklarasi method-nya. Pecahan tiap bagian:

| Bagian | Arti |
|---|---|
| `public` | Method bisa dipanggil dari luar class |
| `function` | Ini function / method biasa di PHP |
| `scopeActive` | Nama method. **WAJIB diawali `scope`** supaya Laravel mengenalinya sebagai scope |
| `(Builder $query)` | Satu parameter, yaitu objek query builder Laravel |
| `: Builder` | Tipe kembalian method, yaitu Builder juga (karena kita kembalikan query yang dimodifikasi) |

**Intinya**: nama method harus `scope` + NamaScopePascalCase. Untuk `scopeActive`, saat dipanggil jadi `::active()`.

#### `return $query->where('is_active', true);`

Ini isi scope-nya, bagian terpenting. Pecahannya:

- `$query` = objek query builder yang Laravel kirim otomatis ke scope kita.
- `->where('is_active', true)` = tambahkan kondisi WHERE ke query: "kolom `is_active` harus `true`".
- `return` = kembalikan query yang sudah dimodifikasi.

**Yang terjadi di belakang layar**:

Saat kamu panggil `Product::active()`, Laravel **tidak langsung** menjalankan query ke database. Yang dilakukan Laravel adalah:

1. Buat objek query builder baru untuk model `Product`.
2. Masukkan objek itu ke parameter `$query` di scope `scopeActive`.
3. Di scope, kita tambahkan kondisi `WHERE is_active = true` ke query builder itu.
4. Kembalikan objek query builder yang sudah diberi kondisi.
5. Query baru akan **dieksekusi** saat kamu panggil `->get()`, `->count()`, `->first()`, dst.

Jadi `Product::active()` **belum menjalankan query**. Yang berikut baru menjalankan:

```php
Product::active()->get();    // menjalankan query, kembalikan koleksi produk
Product::active()->count();  // menjalankan query, kembalikan angka
Product::active()->first();  // menjalankan query, kembalikan 1 produk pertama
```

Inilah kenapa scope bisa disambung (chain) dengan method lain seperti `where`, `orderBy`, dll.

---

## 6. Cara Pakai Scope (Banyak Contoh)

Sekarang scope sudah dibuat. Ayo kita coba pakai lewat **Tinker** untuk memastikan semuanya jalan.

Buka terminal, jalankan:

```bash
php artisan tinker
```

### Contoh 1: Ambil Semua Produk Aktif

```php
$products = Product::active()->get();
```

Akan return koleksi produk **HANYA yang `is_active = 1`**. Produk dengan `is_active = 0` tidak ikut.

### Contoh 2: Hitung Jumlah Produk Aktif

```php
$count = Product::active()->count();
```

Akan return angka, misalnya `1` (kalau di tabel kamu cuma 1 produk yang aktif).

### Contoh 3: Sambung dengan `where` Lain

```php
$cheapActiveProducts = Product::active()
    ->where('harga', '<', 15000)
    ->get();
```

Scope `active()` **bisa disambung** dengan method query lain. Kode ini artinya: "Ambil produk aktif, lalu filter lagi hanya yang harga di bawah 15000."

### Contoh 4: Sambung dengan `orderBy`

```php
$products = Product::active()
    ->orderBy('nama')
    ->get();
```

Artinya: "Ambil produk aktif, urutkan berdasarkan nama A-Z."

### Contoh 5: Ambil 1 Produk Aktif Pertama

```php
$product = Product::active()->first();
```

Akan return objek produk aktif pertama (atau `null` kalau tidak ada produk aktif sama sekali).

### Contoh 6: Cari Produk Aktif Berdasarkan Slug

```php
$product = Product::active()
    ->where('slug', 'kopi-susu-vanilla-250ml')
    ->first();
```

Artinya: "Ambil produk aktif dengan slug tertentu." Ini contoh yang sangat berguna untuk halaman detail produk di halaman publik, supaya user tidak bisa akses produk nonaktif lewat URL.

---

## 7. Apa yang Terjadi di Belakang Layar? (Lihat SQL-nya)

Laravel punya cara mudah untuk melihat SQL apa yang sebenarnya dijalankan. Di Tinker, coba ini:

```php
DB::enableQueryLog();

Product::active()->get();

DB::getQueryLog();
```

Akan muncul array berisi query yang dijalankan. Kira-kira isinya:

```sql
select * from `products` where `is_active` = 1 and `products`.`deleted_at` is null
```

Perhatikan dua kondisi `WHERE`:

1. `` `is_active` = 1 `` → ini dari scope `active()` yang kita bikin.
2. `` `products`.`deleted_at` is null `` → ini otomatis ditambah Laravel karena model `Product` pakai trait `SoftDeletes` (dari materi 09).

Jadi dua sistem (soft delete + status aktif) **bekerja bersama** dengan mulus. Canggih.

---

## 8. Lebih Jauh: Scope Bisa Terima Parameter

Sebagai bonus, scope juga bisa **terima parameter**. Misalnya kita mau bikin scope `status($value)` yang fleksibel:

```php
public function scopeStatus(Builder $query, bool $value): Builder
{
    return $query->where('is_active', $value);
}
```

Cara pakai:

```php
$active = Product::status(true)->get();    // ambil yang aktif
$inactive = Product::status(false)->get(); // ambil yang nonaktif
```

**Tapi untuk pemula, saya sarankan cukup bikin scope `active()` saja dulu.** Karena di halaman publik, hampir selalu kita cuma butuh produk aktif. Kalau butuh produk nonaktif, biasanya itu di halaman admin, dan admin bisa pakai `Product::where('is_active', false)->get()` saja tanpa scope.

Mengikuti prinsip **YAGNI** (You Aren't Gonna Need It): bikin yang simple dulu. Kalau nanti butuh scope `status()` atau scope lain, baru kita tambahkan.

---

## 9. Hal-hal Penting yang Sering Bikin Pemula Bingung

### a. Scope Tidak Mengeksekusi Query

`Product::active()` **belum menjalankan query**. Yang dilakukan scope hanyalah **menambahkan kondisi** ke query builder. Query baru dijalankan saat kamu panggil method eksekutor: `->get()`, `->first()`, `->count()`, `->paginate()`, dst.

Jadi kalau kamu hanya nulis `Product::active();` tanpa `->get()`, **tidak terjadi apa-apa** di database.

### b. Nama Method vs Nama Scope

| Method di model | Cara panggil |
|---|---|
| `scopeActive` | `Product::active()` |
| `scopeActive` (huruf besar A) | `Product::Active()` juga bisa (PHP tidak kasensitive ke nama method), tapi konvensi: panggil dengan huruf kecil |

**Konvensi**: nama method pakai PascalCase (`scopeActive`, `scopeInStock`), cara panggil pakai camelCase kecil (`active`, `inStock`).

### c. Scope Harus `return $query->...`

Di dalam scope, **wajib** mengembalikan query builder yang sudah dimodifikasi. Kalau kamu lupa menulis `return`:

```php
// SALAH - tidak return
public function scopeActive(Builder $query): Builder
{
    $query->where('is_active', true);  // tidak return
}
```

Maka saat dipanggil `Product::active()->get()`, akan error karena `active()` mengembalikan `null`, bukan objek Builder. **Selalu ingat `return`.**

### d. Scope Bisa Dirantai dengan Scope Lain

Kalau kamu punya beberapa scope, semuanya bisa disambung:

```php
Product::active()
    ->inStock()         // (misal) ambil yang stok > 0
    ->popular()         // (misal) ambil yang banyak dilihat
    ->get();
```

Setiap scope akan **menambahkan kondisi WHERE** baru ke query. Kondisi-kondisi itu digabung dengan `AND`.

### e. Scope Tidak Mengubah Data

Scope hanya **membaca** data (SELECT). Scope tidak bisa dipakai untuk mengubah `is_active` produk. Kalau mau ubah status, gunakan cara biasa: `$product->is_active = true; $product->save();`.

### f. Scope Bukan Method Biasa di Model

Walaupun `scopeActive` ditulis seperti method biasa di model, **kamu tidak memanggilnya langsung** sebagai `$product->scopeActive()`. Selalu panggil lewat query builder: `Product::active()->...`. Ini kontrak khusus Laravel, bukan aturan PHP.

---

## Ringkasan Tahap 3

| Hal | Isi |
|---|---|
| Tujuan | Bikin shortcut untuk query "ambil produk aktif" |
| Masalah sebelum scope | `Product::where('is_active', true)` ditulis berulang di banyak tempat |
| Solusi | Bungkus logika jadi satu method dengan nama pendek |
| Nama method di model | `scopeActive(Builder $query)` |
| Cara panggil | `Product::active()->get()` |
| Aturan nama | Harus diawali `scope`, saat dipanggil hapus `scope` |
| Yang terjadi di SQL | `WHERE is_active = 1` |
| Persiapan tambahan | Tambah `'is_active'` ke `$fillable` di model |

---

## Apa yang Sudah Bisa Kamu Lakukan Setelah Tahap 3?

Setelah mengikuti tahap 3, kamu bisa:

1. **Mengambil hanya produk aktif** lewat Tinker:
   ```php
   Product::active()->get();
   ```
2. **Menggabung scope** dengan method query lain:
   ```php
   Product::active()->where('harga', '<', 15000)->get();
   ```
3. **Melihat SQL** yang dijalankan oleh scope:
   ```php
   DB::enableQueryLog();
   Product::active()->get();
   DB::getQueryLog();
   ```

**Tapi**, di halaman web `/produk` kamu, **belum ada perubahan**. Controller masih memakai `Product::all()` atau query lama. Scope sudah ada di model, tapi belum dipakai.

Itu yang akan kita kerjakan di tahap 4.

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: memakai scope `active()` di controller, supaya halaman publik hanya menampilkan produk aktif?**

Kalau iya, tahap 4 kita akan:

1. Buka controller `ProductController`.
2. Bedakan dua halaman: **halaman publik** (hanya produk aktif) dan **halaman admin** (semua produk, termasuk nonaktif).
3. Pakai `Product::active()->get()` di halaman publik, dan `Product::all()` di halaman admin.
4. Tes halaman web: pastikan produk dengan `is_active = 0` tidak muncul di halaman publik, tapi tetap muncul di halaman admin.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
