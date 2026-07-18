# Tahap 2 — Menambahkan Kolom `deleted_at` dan Trait `SoftDeletes` pada Model Product

> Materi: Soft Delete Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi: Menyiapkan "Tempat Sampah" Sebelum Bisa Membuang

Bayangkan kamu baru pindah ke rumah baru. Kamu mau punya kebiasaan: **sebelum membuang sesuatu, taruh dulu di tempat sampah**.

Pertanyaannya: apa yang harus kamu siapkan **lebih dulu**?

Jawabannya jelas: **tempat sampahnya itu sendiri**.

Tanpa tempat sampah, kamu tidak bisa melakukan kebiasaan "buang dulu, pertimbangkan nanti". Kamu hanya bisa langsung membuang ke luar (permanen).

Nah di Laravel, untuk bisa melakukan soft delete, kita harus **menyiapkan dua hal** dulu:

1. **Kolom `deleted_at`** di tabel database → inilah "tempat sampah"-nya.
2. **Trait `SoftDeletes`** di model `Product` → inilah yang bilang ke Laravel, "Tolong, model ini mau pakai sistem tempat sampah ya, bukan buang langsung."

Dua hal ini wajib ada **sebelum** kita menyentuh controller atau view. Makanya tahap 2 fokus ke dua ini dulu.

---

## 2. Apa Itu Migration di Laravel?

Sebelum bikin kolom `deleted_at`, kita perlu paham dulu cara Laravel mengubah struktur tabel.

Di Laravel, kita **tidak mengubah tabel database langsung lewat phpMyAdmin atau SQL**. Kita pakai yang namanya **migration**.

**Migration** = file PHP yang berisi instruksi "ubah struktur database". Laravel akan baca file ini, lalu menjalankan perintah ke database secara otomatis.

Cara cek migration yang sudah ada di projek kamu:

```bash
php artisan migrate:status
```

Nanti muncul daftar file migration yang sudah pernah dijalankan. Kamu akan lihat nama-nama seperti:

- `2014_10_12_000000_create_users_table.php`
- `xxxx_xx_xx_xxxxxx_create_products_table.php` ← ini migration bikin tabel produk

Setuktur file migration biasanya seperti ini:

```php
public function up()
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('nama');
        // ... kolom lainnya
        $table->timestamps();
    });
}
```

**Inti migration**: setiap kali kita mau ubah struktur tabel (tambah kolom, hapus kolom, ubah tipe), kita **buat file migration baru**. Tidak boleh edit migration lama yang sudah dijalankan.

---

## 3. Langkah 1: Tambah Kolom `deleted_at` ke Tabel Products

### a. Buat Migration Baru

Buka terminal di folder projek Laravel kamu, lalu jalankan:

```bash
php artisan make:migration add_deleted_at_to_products_table --table=products
```

Penjelasan perbagian:

| Bagian | Arti |
|---|---|
| `php artisan make:migration` | Perintah Laravel untuk bikin file migration baru |
| `add_deleted_at_to_products_table` | Nama migration (bebas, tapi deskriptif) |
| `--table=products` | Bilang ke Laravel: migration ini untuk **ubah tabel yang sudah ada**, yaitu `products` |

Laravel akan bikin file baru di `database/migrations/`, dengan nama seperti:

```
2026_07_18_103000_add_deleted_at_to_products_table.php
```

### b. Edit File Migration

Buka file itu. Akan ada dua fungsi: `up()` dan `down()`.

- `up()` = apa yang dilakukan saat migration dijalankan (tambah kolom).
- `down()` = apa yang dilakukan saat migration di-rollback (batal tambah kolom).

Ubah isinya jadi seperti ini:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('products', function (Blueprint $table) {
            // Tambah kolom deleted_at, tipe nullable timestamp
            $table->softDeletes();
        });
    }

    public function down(): void
    {
        Schema::table('products', function (Blueprint $table) {
            // Hapus kolom deleted_at kalau migration di-rollback
            $table->dropSoftDeletes();
        });
    }
};
```

Penjelasan per baris penting:

**`$table->softDeletes();`**

Ini shortcut Laravel. Fungsinya: tambahkan kolom bernama `deleted_at` dengan tipe `TIMESTAMP` yang **boleh NULL**. Kolom inilah "tempat sampah" yang akan kita pakai.

Default-nya, kolom ini bernama `deleted_at`. Tapi kamu bisa kasih nama lain kalau mau:

```php
$table->softDeletes('dihapus_pada');  // kolom jadi namanya "dihapus_pada"
```

**Untuk pemula: pakai nama default `deleted_at` saja.** Biar konsisten dengan dokumentasi Laravel dan tutorial lain.

**`$table->dropSoftDeletes();`**

Ini kebalikannya: hapus kolom `deleted_at`. Dipakai di fungsi `down()` supaya kalau migration dibatalkan (`rollback`), tabel balik seperti semula.

### c. Jalankan Migration

Sekarang jalankan migration-nya:

```bash
php artisan migrate
```

Kalau berhasil, kamu akan lihat output seperti:

```
2026_07_18_103000_add_deleted_at_to_products_table ............... DONE
```

Sekarang tabel `products` kamu punya kolom baru: **`deleted_at`**.

Isi kolom itu default-nya **NULL** untuk semua produk yang sudah ada. Artinya: **semua produk yang sudah ada dianggap "belum dihapus"** (masih aktif). Mantap.

---

## 4. Langkah 2: Pakai Trait `SoftDeletes` di Model Product

Sekarang kita kasih tahu Laravel: "Model `Product` ini ingin pakai sistem soft delete ya."

Buka file `app/Models/Product.php`. Kira-kira isinya seperti ini:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
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

Ubah jadi seperti ini:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;  // ← tambahan 1

class Product extends Model
{
    use SoftDeletes;  // ← tambahan 2

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

Penjelasan:

### `use Illuminate\Database\Eloquent\SoftDeletes;`

Ini **mengimpor** (memanggil) fitur SoftDeletes dari Laravel. Mirip seperti `use Illuminate\Database\Eloquent\Model;` yang sudah kamu pakai.

Istilah teknisnya: ini mengimpor **trait**. Trait itu seperti "campuran bumbu siap pakai" yang bisa kita tambahkan ke model supaya model punya kemampuan baru. SoftDeletes adalah trait yang bikin model paham konsep "hapus sementara".

### `use SoftDeletes;`

Di dalam class `Product`, kita tulis `use SoftDeletes;`. Artinya: **"Model Product ini pakai trait SoftDeletes."**

Setelah baris ini ditambahkan, secara otomatis perilaku model `Product` berubah:

1. **Query biasa** (`Product::all()`, `Product::find($id)`, dst) akan **otomatis menyembunyikan** produk yang `deleted_at`-nya tidak NULL.
2. Saat kamu panggil `$product->delete()`, Laravel **tidak menjalankan `DELETE FROM`**. Yang dilakukan adalah **mengisi `deleted_at` dengan timestamp sekarang**.
3. Muncul method-method baru: `restore()`, `forceDelete()`, `withTrashed()`, `onlyTrashed()`, dst. (akan kita bahas di tahap-tahap berikutnya).

Dua baris kecil ini saja sudah **mengubah cara kerja model Product secara fundamental**. Tanpa menyentuh controller, tanpa menyentuh view. Ini kekuatan trait di Laravel.

---

## 5. Apa yang Terjadi di Database Sekarang?

Sebelum kita paham soft delete di kode, mari lihat dulu apa isi tabel `products` sekarang.

Bayangkan kamu punya 3 produk di tabel:

| id | nama | harga | deleted_at |
|---|---|---|---|
| 1 | Kopi Susu Vanilla | 18000 | NULL |
| 2 | Teh Manis Hangat | 8000 | NULL |
| 3 | Roti Coklat | 12000 | NULL |

Semua produk `deleted_at`-nya **NULL**. Artinya semua produk **aktif** dan **belum dihapus**.

Sekarang, kalau kita jalankan kode ini di controller atau tinker:

```php
$product = Product::find(2);
$product->delete();
```

Apa yang terjadi?

**SEBELUM pakai SoftDeletes:**
```sql
DELETE FROM products WHERE id = 2;
```
Baris produk id=2 hilang dari tabel selamanya.

**SESUDAH pakai SoftDeletes:**
```sql
UPDATE products SET deleted_at = '2026-07-18 10:35:42' WHERE id = 2;
```
Barisnya masih ada. Cuma `deleted_at`-nya jadi terisi tanggal sekarang.

Isi tabel setelah di-soft-delete:

| id | nama | harga | deleted_at |
|---|---|---|---|
| 1 | Kopi Susu Vanilla | 18000 | NULL |
| 2 | Teh Manis Hangat | 8000 | **2026-07-18 10:35:42** |
| 3 | Roti Coklat | 12000 | NULL |

Produk id=2 **masih ada di tabel**. Tapi `deleted_at`-nya terisi tanggal penghapusan.

---

## 6. Efek pada Query: Produk yang Dihapus "Menghilang" Secara Ajaib

Inilah bagian yang seru. Setelah model pakai `SoftDeletes`, Laravel **secara otomatis** memfilter produk yang sudah dihapus di setiap query.

Coba lihat:

### Query 1: Ambil Semua Produk Aktif

```php
$products = Product::all();
```

Yang diambil **HANYA** produk dengan `deleted_at = NULL`. Produk id=2 (yang sudah di-soft-delete) **tidak muncul**, padahal datanya masih ada di database.

### Query 2: Cari Produk Berdasarkan ID

```php
$product = Product::find(2);  // produk yang sudah di-soft-delete
```

Hasilnya: **`null`**. Laravel bilang "tidak ketemu", padahal sebenarnya ada di tabel.

### Query 3: Hitung Jumlah Produk Aktif

```php
$count = Product::count();
```

Hasilnya: **2** (produk id=1 dan id=3). Produk id=2 **tidak dihitung**.

**Kesimpulan**: dari sudut pandang kode aplikasi kita, produk id=2 seakan-akan sudah tidak ada. Tapi dari sudut pandang database, produk itu masih lengkap dengan semua datanya. Inilah magic soft delete.

---

## 7. Cara Melihat Produk yang Sudah Di-Soft-Delete

Kalau produk id=2 "menghilang" dari semua query biasa, bagaimana kita melihatnya lagi?

Laravel menyediakan dua method khusus:

### a. `withTrashed()` — Tampilkan Semua, Termasuk yang Dihapus

```php
$allProducts = Product::withTrashed()->get();
```

Akan return **3 produk**: id=1, id=2, dan id=3. Termasuk yang `deleted_at`-nya terisi.

### b. `onlyTrashed()` — Tampilkan HANYA yang Sudah Dihapus

```php
$trashedProducts = Product::onlyTrashed()->get();
```

Akan return **1 produk**: id=2 saja. Hanya yang ada di "tempat sampah".

Dua method ini yang akan jadi dasar membuat halaman **"Tong Sampah Produk"** di tahap 4. Untuk sekarang, kamu cukup tahu mereka ada.

---

## 8. Hal-hal Penting yang Sering Bikin Pemula Bingung

### a. Migration Harus Dijalankan Dulu

Setelah edit file migration, kamu **wajib** jalankan `php artisan migrate`. Kalau tidak, kolom `deleted_at` tidak akan benar-benar ada di database. Model `Product` akan error saat coba soft-delete karena kolom tidak ketemu.

### b. Trait `SoftDeletes` Wajib Ada di Model

Kalau kolom `deleted_at` ada di tabel tapi kamu **lupa** pakai `use SoftDeletes;` di model, maka:

- Query biasa **TIDAK** akan menyembunyikan produk yang `deleted_at`-nya terisi.
- `$product->delete()` tetap akan `DELETE FROM` permanen.

Trait dan kolom harus **berdua** ada supaya soft delete bekerja.

### c. Kolom `deleted_at` Bukan Kolom Biasa

Kolom `deleted_at` itu **timestamp nullable**. Kamu tidak pernah **harus** mengisinya secara manual. Laravel akan otomatis mengisi saat `delete()` dipanggil, dan otomatis mengosongkan saat `restore()` dipanggil.

### d. `deleted_at` Tidak Sama dengan `updated_at` atau `created_at`

Laravel punya tiga timestamp otomatis:

| Kolom | Kapan diisi |
|---|---|
| `created_at` | Saat produk pertama kali dibuat (insert) |
| `updated_at` | Saat produk diedit (update) |
| `deleted_at` | Saat produk di-soft-delete (bukan saat di-restore permanen) |

Tiga-tiganya beda fungsi. Jangan tertukar.

### e. Field `deleted_at` Biasanya Tidak Dimasukkan `$fillable`

Di model `Product`, `$fillable` berisi field-field yang boleh diisi massal lewat `Product::create([...])`:

```php
protected $fillable = [
    'nama',
    'harga',
    'stok',
    'deskripsi',
    'kategori',
    'slug',
    'gambar',
];
```

**Perhatikan: `deleted_at` TIDAK ada di `$fillable`**, dan itu **benar**. Karena `deleted_at` tidak boleh diisi manual oleh user lewat form. Laravel yang akan mengisi kolom itu secara otomatis lewat method `delete()`.

---

## Ringkasan Tahap 2

| Hal | Isi |
|---|---|
| Tujuan | Siapkan "tempat sampah" supaya bisa soft delete |
| Langkah 1 | Tambah kolom `deleted_at` lewat migration (`$table->softDeletes()`) |
| Langkah 2 | Pakai trait `SoftDeletes` di model Product (`use SoftDeletes;`) |
| Perubahan otomatis | Query biasa exclude produk dengan `deleted_at` terisi |
| Method baru (pengenal) | `withTrashed()`, `onlyTrashed()`, `restore()`, `forceDelete()` |
| Catatan | `deleted_at` tidak masuk `$fillable`, diisi otomatis Laravel |
| Syarat | Migration **dan** trait harus berdua ada |

---

## Apa yang Sudah Bisa Kamu Lakukan Setelah Tahap 2?

Setelah mengikuti tahap 2, kalau kamu buka **Tinker** (`php artisan tinker`) dan jalankan:

```php
$product = Product::first();
$product->delete();
```

Maka:

- Produk itu akan **menghilang** dari `Product::all()`.
- Tapi **masih ada** di database dengan `deleted_at` terisi.

**Tapi**, di halaman web `/produk` kamu, **belum ada perubahan tampilan**. Karena controller dan view belum kita ubah. Itu akan kita kerjakan di tahap 3.

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: memodifikasi controller `destroy()` supaya melakukan soft delete, dan melihat efeknya di halaman produk?**

Kalau iya, tahap 3 kita akan:
1. Buka controller `ProductController`.
2. Lihat method `destroy()` yang saat ini masih hapus permanen.
3. Ubah perilakunya jadi soft delete (sebenarnya tidak banyak berubah, karena trait yang melakukan pekerjaan).
4. Tes klik tombol Hapus di halaman `/produk`, dan lihat produk menghilang tanpa benar-benar hilang dari database.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
