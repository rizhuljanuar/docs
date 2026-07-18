# Tahap 2 — Menambahkan Kolom `is_active` pada Tabel Products lewat Migration

> Materi: Status Produk Aktif/Nonaktif
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi: Menyiapkan "Saklar Lampu" di Setiap Produk

Di tahap 1, kita sudah paham konsepnya: setiap produk butuh **status aktif/nonaktif**, ibarat saklar lampu ON/OFF.

Sekarang pertanyaannya:

> **Bagaimana cara kita menyiapkan "saklar" itu di setiap produk?**

Bayangkan kamu baru bangun rumah. Kamu mau tiap kamar punya saklar lampu sendiri. Apa yang harus kamu lakukan?

1. Kamu **tidak perlu** membongkar rumahnya.
2. Kamu **cukup** memasang saklar kecil di tiap kamar, lalu hubungkan ke lampunya.

Saklar itu sederhana: **hanya dua posisi, ON atau OFF**. Tidak perlu tombol kompleks, tidak perlu remote, tidak perlu aplikasi. Cukup saklar biasa.

Di Laravel, "memasang saklar" di setiap produk = **menambahkan satu kolom kecil di tabel `products`**, yaitu kolom **`is_active`**.

- Kolom ini sederhana: hanya bisa berisi `1` (ON / aktif) atau `0` (OFF / nonaktif).
- Kolom ini dipasang ke **setiap produk**, lewat perintah **migration** (yang sudah kamu pelajari di materi 09).

Di tahap 2 ini, kita fokus ke **dua hal** dulu:

1. **Membuat migration** untuk menambah kolom `is_active`.
2. **Memahami tipe data `boolean`** di database (kenapa isinya cuma `0` dan `1`).

Kita **belum** menyentuh model, controller, atau view dulu. Satu langkah demi satu langkah.

---

## 2. Kenapa Perlu Migration Lagi? Bukannya Tabel `products` Sudah Ada?

Iya, tabel `products` sudah ada sejak materi 01. Tapi di tabel itu **belum ada kolom `is_active`**. Coba bayangkan struktur tabel `products` saat ini:

| id | nama | harga | stok | deskripsi | kategori | slug | gambar | created_at | updated_at | deleted_at |
|---|---|---|---|---|---|---|---|---|---|---|

Perhatikan: **tidak ada kolom `is_active`**. Jadi Laravel belum punya tempat untuk menyimpan status aktif/nonaktif setiap produk.

Cara menambahkan kolom baru ke tabel yang sudah ada di Laravel: **buat migration baru**. Kita **TIDAK boleh** edit migration lama (yang bikin tabel `products`) karena migration itu sudah dijalankan. Mengeditnya bisa bikin database tidak konsisten.

Ini seperti memasang saklar lampu di kamar yang sudah jadi: **kamu tidak membongkar rumahnya**, kamu hanya menambah saklar di dinding yang sudah ada.

> **Aturan emas migration**: Setiap kali mau ubah struktur tabel yang sudah ada, **buat file migration baru**. Jangan edit migration lama yang sudah pernah dijalankan.

---

## 3. Apa Itu Tipe Data `boolean`? Kenapa Cuma 0 dan 1?

Sebelum bikin migration, kita perlu paham tipe data yang akan kita pakai untuk kolom `is_active`.

Database punya banyak tipe data: `VARCHAR` (teks), `INT` (angka bulat), `DATETIME` (tanggal-jam), dst. Tapi untuk kolom "status aktif/nonaktif", kita pakai tipe khusus yang namanya **`boolean`**.

### Apa itu Boolean?

**Boolean** = tipe data yang **hanya punya dua nilai**: **BENAR** atau **SALAH** (true / false).

Istilah "boolean" diambil dari nama seorang matematikawan Inggris bernama **George Boole**. Jadi kalau kamu baca "boolean", bayangkan saja: "dua pilihan: benar atau salah".

| Kondisi | Nilai Boolean |
|---|---|
| Produk aktif | `true` (benar, iya, produk ini aktif) |
| Produk nonaktif | `false` (salah, tidak, produk ini tidak aktif) |

### Kenapa di Database Sering Ditulis `0` dan `1`?

Karena database menyimpan data dalam bentuk angka di belakang layar. Konvensinya:

| Nilai Logika | Angka di Database |
|---|---|
| `true` (benar / aktif) | `1` |
| `false` (salah / nonaktif) | `0` |

Beberapa database (seperti MySQL) tidak punya tipe `BOOLEAN` asli. Tapi Laravel tahu ini, jadi Laravel akan otomatis membuat kolom `boolean` sebagai `TINYINT(1)`, yaitu angka kecil yang hanya berisi `0` atau `1`.

**Yang perlu kamu ingat**: kolom `is_active` isinya cuma dua kemungkinan:

- `1` = aktif
- `0` = nonaktif

Tidak ada nilai lain seperti `2`, `3`, atau `"aktif"`, `"nonaktif"`. Cukup `0` dan `1`. Sederhana sekali.

### Kenapa Nama Kolomnya `is_active`?

Konvensi penamaan di Laravel (dan programmer umumnya):

- Awalan **`is_`** menandakan: "ini pertanyaan ya/tidak".
- **`is_active`** dibaca: "apakah (produk) ini aktif?" → iya (`1`) / tidak (`0`).

Contoh penamaan lain dengan pola yang sama:

| Nama kolom | Artinya |
|---|---|
| `is_active` | Apakah aktif? |
| `is_admin` | Apakah user ini admin? |
| `is_verified` | Apakah sudah diverifikasi? |
| `is_paid` | Apakah sudah dibayar? |

Jadi begitu kamu lihat kolom `is_active`, otak kamu langsung tahu: "ah, ini status ya/tidak".

---

## 4. Langkah 1: Buat Migration Baru untuk Menambah Kolom `is_active`

Buka terminal di folder projek Laravel kamu, lalu jalankan perintah ini:

```bash
php artisan make:migration add_is_active_to_products_table --table=products
```

Penjelasan tiap bagian perintah:

| Bagian | Arti |
|---|---|
| `php artisan make:migration` | Perintah Laravel untuk membuat file migration baru |
| `add_is_active_to_products_table` | Nama migration (bebas, tapi pilih yang deskriptif) |
| `--table=products` | Memberitahu Laravel: migration ini untuk **mengubah tabel yang sudah ada**, yaitu `products` |

Setelah perintah ini dijalankan, Laravel membuat file baru di folder `database/migrations/`, dengan nama seperti:

```
2026_07_18_104500_add_is_active_to_products_table.php
```

Angka depan itu timestamp (tanggal-jam pembuatan), supaya migration dijalankan sesuai urutan waktu.

> **Catatan penting**: kita pakai `--table=products` (bukan `--create`). Tanda sama dengan `=` menandakan "modifikasi tabel yang sudah ada", bukan "buat tabel baru". Ini berbeda dengan migration awal `create_products_table` yang memakai `Schema::create()`.

---

## 5. Langkah 2: Edit File Migration

Buka file migration yang baru dibuat. Isinya kira-kira seperti ini (versi default dari Laravel):

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
            //
        });
    }

    public function down(): void
    {
        Schema::table('products', function (Blueprint $table) {
            //
        });
    }
};
```

Ada dua fungsi penting di dalam file migration:

| Fungsi | Kapan dijalankan? |
|---|---|
| `up()` | Saat migration **dijalankan** (`php artisan migrate`). Di sinilah kita **tambah kolom**. |
| `down()` | Saat migration **dibatalkan** (`php artisan migrate:rollback`). Di sinilah kita **hapus kolom** yang barusan ditambah. |

Sekarang ubah isinya jadi seperti ini:

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
            // Tambah kolom is_active, tipe boolean, default false (0)
            $table->boolean('is_active')->default(false);
        });
    }

    public function down(): void
    {
        Schema::table('products', function (Blueprint $table) {
            // Hapus kolom is_active kalau migration dibatalkan
            $table->dropColumn('is_active');
        });
    }
};
```

### Penjelasan Baris Per Baris

#### Baris: `$table->boolean('is_active')`

Ini perintah ke Laravel: **"Buatkan kolom baru bernama `is_active`, dengan tipe boolean."**

Laravel akan menerjemahkan ke SQL kira-kira seperti ini (di MySQL):

```sql
is_active TINYINT(1)
```

Artinya: kolom berisi angka kecil, hanya menerima `0` atau `1`.

#### Baris: `->default(false)`

Ini perintah lanjutan: **"Beri nilai default `false`."**

Artinya: setiap kali ada produk baru (atau produk lama), kalau tidak diisi manual, kolom `is_active` otomatis **bernilai `false` (0)**.

Mengapa default-nya `false`? Karena ini **pengaman**. Ingat di tahap 1: produk baru sebaiknya **nonaktif dulu**, sampai admin benar-benar yakin data sudah lengkap dan siap tampil di publik. Dengan default `false`, kita tidak perlu repot mengeset statusnya setiap kali. Laravel yang akan melakukannya untuk kita.

Di belakang layar, `false` akan diterjemahkan jadi `0` di database. Tapi di kode Laravel, kita tulis `false` (bukan `0`) supaya lebih jelas dibaca: "ini status boolean, bukan angka biasa."

#### Baris di `down()`: `$table->dropColumn('is_active')`

Ini kebalikannya: **"Hapus kolom `is_active`."** Dipakai saat migration ini dibatalkan (`rollback`). Tujuannya supaya struktur tabel kembali seperti sebelum migration dijalankan.

---

## 6. Langkah 3: Jalankan Migration

Sekarang waktunya benar-benar menerapkan perubahan ke database. Jalankan:

```bash
php artisan migrate
```

Kalau berhasil, outputnya kira-kira seperti ini:

```
2026_07_18_104500_add_is_active_to_products_table ............... DONE
```

Sekarang cek struktur tabel `products` kamu (lewat phpMyAdmin, TablePlus, atau `DESCRIBE products` di SQL). Strukturnya jadi seperti ini:

| Field | Type | Null | Default |
|---|---|---|---|
| id | bigint | No | (auto increment) |
| nama | varchar | No | — |
| harga | int | No | — |
| stok | int | No | — |
| deskripsi | text | Yes | NULL |
| kategori | varchar | No | — |
| slug | varchar | No | — |
| gambar | varchar | Yes | NULL |
| created_at | timestamp | Yes | NULL |
| updated_at | timestamp | Yes | NULL |
| deleted_at | timestamp | Yes | NULL |
| **is_active** | **tinyint(1)** | **No** | **0** |

Perhatikan baris terakhir: **kolom `is_active` sudah ada**, tipenya `tinyint(1)` (ini cara MySQL menyimpan boolean), dan **default-nya `0`** (false / nonaktif). Persis seperti yang kita inginkan.

---

## 7. Apa yang Terjadi pada Produk yang Sudah Ada?

Pertanyaan bagus: kalau sebelumnya kita sudah punya 3 produk di tabel, setelah kolom `is_active` ditambah, **produk-produk lama itu isinya apa?**

Jawabannya: semuanya **otomatis diberi `is_active = 0` (nonaktif)**.

Bayangkan tabel `products` sebelum migration:

| id | nama | harga |
|---|---|---|
| 1 | Kopi Susu Vanilla | 18000 |
| 2 | Teh Manis Hangat | 8000 |
| 3 | Roti Coklat | 12000 |

Setelah migration dijalankan (kolom `is_active` ditambah dengan default `0`):

| id | nama | harga | is_active |
|---|---|---|---|
| 1 | Kopi Susu Vanilla | 18000 | **0** |
| 2 | Teh Manis Hangat | 8000 | **0** |
| 3 | Roti Coklat | 12000 | **0** |

Semua produk lama jadi **nonaktif**. Ini efek dari `->default(false)`.

### Tunggu, Berarti Semua Produk Saya Menghilang dari Halaman Publik?

**Belum.** Karena di tahap 2 ini, kita **belum** mengubah cara query di controller. Semua produk masih ditampilkan di halaman publik seperti biasa, walaupun `is_active = 0`.

Query "hanya tampilkan produk aktif" akan kita pelajari di tahap selanjutnya (query scope `active()`).

Untuk sekarang, di tahap 2, kamu cukup paham: **kolomnya sudah siap, semua produk sudah diberi default nonaktif, tinggal nanti kita atur query dan kontrolernya.**

---

## 8. Cara Manual Mengubah Status Produk (Sekadar Tes)

Sebelum lanjut, ayo coba ubah status satu produk secara manual lewat **Tinker**, supaya kamu lihat efeknya.

Buka terminal, jalankan:

```bash
php artisan tinker
```

Di dalam Tinker, ketik:

```php
$product = Product::find(1);
$product->is_active = true;
$product->save();
```

Penjelasan:

| Baris | Apa yang terjadi |
|---|---|
| `Product::find(1)` | Ambil produk dengan id = 1 dari database |
| `$product->is_active = true` | Set property `is_active` jadi `true` (diubah jadi aktif) |
| `$product->save()` | Simpan perubahan ke database |

Sekarang cek lagi tabel `products`:

| id | nama | harga | is_active |
|---|---|---|---|
| 1 | Kopi Susu Vanilla | 18000 | **1** |
| 2 | Teh Manis Hangat | 8000 | 0 |
| 3 | Roti Coklat | 12000 | 0 |

Produk id=1 sekarang **aktif**. Dua lainnya masih **nonaktif**.

### Catatan Kecil: Model `Product` belum tahu soal `is_active`?

Sebenarnya Laravel cukup pintar. Walaupun di model `Product` belum kita sebutkan `is_active` di `$fillable`, kamu masih bisa **membaca** kolom itu (`$product->is_active`) dan **mengubahnya** lewat `$product->save()`.

**TAPI**, supaya nanti bisa pakai `Product::create([...])` atau `$product->update([...])` dengan kolom `is_active`, kita perlu **menambahkan `is_active` ke `$fillable`** di model. Itu akan kita lakukan di tahap selanjutnya, bersamaan dengan menyiapkan query scope.

Untuk sekarang, di tahap 2, kita cukup main dengan `$product->is_active = true; $product->save();`. Cara ini langsung mengubah property di object, lalu menyimpannya. Cara ini **selalu bisa**, tidak peduli apa yang ada di `$fillable`.

---

## 9. Hal-hal Penting yang Sering Bikin Pemula Bingung

### a. Migration Harus Dijalankan Setelah Edit File

Setelah kamu mengedit file migration, kamu **wajib** menjalankan `php artisan migrate`. Kalau tidak, kolom `is_active` tidak akan benar-benar ada di database. Di file migration, instruksi sudah ditulis, tapi belum dieksekusi sampai `php artisan migrate` dipanggil.

### b. Default `false` vs Default `true`

Di migration kita pakai `->default(false)`. Ini penting:

- `->default(false)` = produk baru otomatis **nonaktif** (aman, tidak langsung tampil).
- `->default(true)` = produk baru otomatis **aktif** (langsung tampil ke publik).

Untuk kasus toko online, **default `false`** lebih aman. Bayangkan admin setengah input lalu tersimpan: kalau default-nya `true`, produk setengah jadi langsung tampil ke publik. Bahaya.

### c. `boolean` di Kode vs di Database

| Di kode Laravel | Di database MySQL |
|---|---|
| `true` | `1` |
| `false` | `0` |

Keduanya sama, hanya cara penulisan berbeda. Di kode PHP/Laravel, kita pakai `true`/`false` (lebih jelas dibaca). Di database, disimpan sebagai `1`/`0` (lebih hemat tempat). Laravel akan otomatis melakukan konversi bolak-balik untuk kita.

### d. Jangan Campur `is_active` dengan `deleted_at`

Di materi 09 kita sudah punya `deleted_at` untuk soft delete. Sekarang kita tambah `is_active`. **Dua-duanya berbeda fungsi:**

| Kolom | Tujuan | Kapan diisi |
|---|---|---|
| `deleted_at` | Tandai produk yang **dihapus sementara** | Saat admin klik "Hapus" |
| `is_active` | Tandai produk yang **siap / belum siap tampil di publik** | Saat admin membuat/edit produk |

Produk bisa dalam kondisi seperti ini:

| Kondisi | `is_active` | `deleted_at` |
|---|---|---|
| Aktif dan dijual | `1` | NULL |
| Nonaktif (draft, belum siap) | `0` | NULL |
| Sudah dihapus (soft delete) | (bebas) | terisi tanggal |

Dua kolom ini **saling melengkapi**, bukan bertabrakan.

### e. Bolehkah Saya Hapus Kolom `deleted_at` Sekarang?

**Boleh**, kalau projek kamu memang tidak butuh soft delete. Tapi sebaiknya **dijaga saja**. Soft delete dan status aktif/nonaktif sering dipakai bersamaan di aplikasi nyata. Menghapus salah satunya tidak perlu.

---

## Ringkasan Tahap 2

| Hal | Isi |
|---|---|
| Tujuan | Menambahkan "saklar" status ke setiap produk |
| Tipe data | `boolean` (cuma `0` / `1`, atau `false` / `true`) |
| Perintah migration | `php artisan make:migration add_is_active_to_products_table --table=products` |
| Isi `up()` | `$table->boolean('is_active')->default(false);` |
| Isi `down()` | `$table->dropColumn('is_active');` |
| Jalankan migration | `php artisan migrate` |
| Default kolom | `false` (0) supaya produk baru otomatis nonaktif |
| Efek pada produk lama | Semua produk lama otomatis dapat `is_active = 0` |
| Beda dengan `deleted_at` | `deleted_at` = hapus sementara; `is_active` = kontrol tampil/tidak |

---

## Apa yang Sudah Bisa Kamu Lakukan Setelah Tahap 2?

Setelah menjalankan tahap 2, kamu bisa:

1. **Mengecek** lewat phpMyAdmin atau Tinker bahwa kolom `is_active` sudah ada di tabel `products`.
2. **Mengubah status** produk satu per satu lewat Tinker:
   ```php
   $product = Product::find(1);
   $product->is_active = true;
   $product->save();
   ```
3. **Melihat** isi kolom `is_active` tiap produk.

**Tapi**, di halaman web `/produk` kamu, **belum ada perubahan tampilan**. Semua produk masih tampil seperti biasa, walaupun ada yang `is_active = 0`. Karena query di controller belum memfilter berdasarkan `is_active`.

Itu yang akan kita kerjakan di tahap 3.

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: membuat query scope `active()` di model Product, supaya mudah mengambil hanya produk yang aktif?**

Kalau iya, tahap 3 kita akan:

1. Tambahkan `is_active` ke `$fillable` di model `Product`.
2. Pelajari apa itu **query scope** (cara Laravel menyimpan query yang sering dipakai supaya bisa dipanggil dengan satu kata).
3. Bikin scope `active()` di model `Product`: `Product::active()->get()` hanya ambil produk aktif.
4. Lihat bagaimana scope ini bikin kode di controller jadi lebih rapi dan gampang dibaca.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
