# Tahap 3 — Membuat Migration Tabel Products

> **Tujuan tahap ini:** Membuat struktur tabel `products` di database menggunakan **migration** Laravel. **Belum membuat model, controller, route, atau view.** Belum mengisi data apa pun.

---

## 1. Pengantar Sederhana

Pada tahap sebelumnya, kita sudah membuat **database** kosong.
Database itu ibarat lemari arsip besar yang masih **kosong**, belum ada map apa pun di dalamnya.

Sebelum kita bisa menyimpan data produk, kita harus membuat **tempat penyimpanan khusus** untuk produk di dalam database.

### Analogi: Lemari Arsip

| Database     | Lemari arsip besar          | Tempat menyimpan semuanya                      |
| Tabel        | Map khusus di lemari         | Satu map khusus untuk satu jenis data (misal: produk) |
| Kolom        | Label isian pada formulir     | "Nama", "Harga", "Stok" - jenis info yang harus diisi |
| Baris data   | Satu lembar formulir terisi | Satu produk lengkap beserta atributnya          |

Jadi, di tahap ini kita akan **membuat satu map kosong** di lemari arsip,
yaitu map **produk**. Map ini punya formulir dengan label:
nama produk, deskripsi, harga, dan stok.

Nanti, setiap kali kita menambah produk baru, kita mengisi satu lembar formulir
baru dan menyimpannya di map produk tersebut.

> Penting: Tahap ini **hanya menyiapkan map dan formulirnya**.
> Belum mengisi produk apa pun. Pengisian data dilakukan di tahap CRUD.

---

## 2. Apa Itu Migration?

### Pengertian sederhana

**Migration** adalah **catatan instruksi** untuk **membuat atau mengubah**
struktur database, khususnya tabel, menggunakan **kode PHP** (bukan klik-klik manual di phpMyAdmin).

### Analogi: Instruksi Pembuatan Rak

Bayangkan kamu pesan rak ke tukang kayu. Kamu tidak datang langsung memahat kayu,
tapi kamu menyerahkan **kertas instruksi** seperti ini:

> "Buat rak dengan 5 laci. Laci pertama lebar 30cm. Laci kedua lebar 20cm..."

Tukang membaca instruksi itu, lalu membangun rak sesuai yang diminta.

**Migration itu seperti kertas instruksi untuk tukang kayu (database).**
Laravel membaca migration, lalu membuat tabel sesuai instruksi yang kita tulis.

### Mengapa kita pakai migration?

| Tanpa Migration                                  | Dengan Migration                                  |
| ------------------------------------------------ | ------------------------------------------------ |
| Klik-klik manual di phpMyAdmin setiap pindah komputer | Cukup jalankan satu perintah, tabel jadi otomatis |
| Sulit melacak apa yang sudah diubah              | Riwayat perubahan tersimpan rapi dalam file       |
| Susah bagi tim untuk samakan struktur database   | Semua anggota tim pakai file migration yang sama  |
| Sulit mengulang dari awal                         | Bisa rollback (batal) dan redo dengan mudah       |

Jadi migration itu seperti **buku catatan perubahan database**:
setiap kali ada tabel baru atau kolom baru, kita tulis di migration,
semua anggota tim tinggal menjalankan file yang sama.

---

## 3. Struktur Tabel `products`

Kita akan membuat satu tabel bernama **`products`** untuk menyimpan data produk.
Tabel ini akan punya kolom-kolom berikut:

| Kolom         | Fungsi                          | Contoh                           |
| ------------- | ------------------------------- | -------------------------------- |
| `id`          | Nomor unik produk                | 1                                |
| `name`        | Nama produk                      | Kaos Hitam                       |
| `description` | Penjelasan produk                | Kaos bahan cotton combed 30s     |
| `price`       | Harga produk (dalam angka)       | 100000                           |
| `stock`       | Jumlah stok yang tersedia        | 20                               |
| `created_at`  | Waktu data dibuat (otomatis)     | 2026-07-16 10:30:00              |
| `updated_at`  | Waktu data terakhir diubah (otomatis) | 2026-07-16 11:45:00              |

### Penjelasan tiap kolom

- **`id`**: Setiap produk harus punya ID unik, jadi tidak akan tertukar.
  ID ini diisi otomatis oleh database (1, 2, 3, ...).
- **`name`**: Wajib diisi. Berisi nama produk seperti "Kaos Hitam".
- **`description`**: Boleh dikosongkan (opsional). Penjelasan lebih detail tentang produk.
- **`price`**: Harga dalam bentuk angka (integer), contoh `100000` untuk Rp 100.000.
- **`stock`**: Jumlah stok. Default-nya 0, jadi jika tidak diisi, dianggap stok nol.
- **`created_at`**: Diisi otomatis oleh Laravel saat produk pertama kali dibuat.
- **`updated_at`**: Diisi otomatis oleh Laravel setiap kali produk diubah.

> Catatan: Kolom `created_at` dan `updated_at` akan diurus oleh Laravel otomatis
> lewat perintah `$table->timestamps()`. Kita tidak perlu mengisinya secara manual.

---

## 4. Membuat File Migration

### Perintahnya

Buka terminal, pastikan kamu berada **di dalam folder project** (`toko-produk`),
lalu jalankan perintah berikut:

```bash
php artisan make:migration create_products_table
```

### Arti perintah tersebut

| Bagian                  | Arti                                                      |
| ----------------------- | --------------------------------------------------------- |
| `php`                   | Menjalankan PHP                                           |
| `artisan`               | Alat bantu perintah bawaan Laravel (seperti pisau serbaguna) |
| `make:migration`        | Perintah untuk membuat file migration baru                |
| `create_products_table` | Nama migration. Konvensi: `create_<namatabel>_table`      |

### Hasilnya

Laravel akan membuat file migration baru di folder:

```
database/migrations/
```

Nama file akan terlihat seperti:

```
2026_07_16_103000_create_products_table.php
```

Polanya: `<tanggal>_<jam>_<nama_migration>.php`.
Tanggal dan jam otomatis ditambahkan agar urutan migration jelas.

### Bukka file tersebut

Buka folder `database/migrations/` di editor (VS Code), lalu buka file
`..._create_products_table.php`. Kita akan mengisinya di langkah berikutnya.

---

## 5. Mengisi File Migration

Saat pertama dibuka, file migration bawaan Laravel berisi kerangka kosong.
Kita perlu mengisinya dengan kolom-kolom tabel `products`.

### Kode lengkap migration

Ganti isi file `..._create_products_table.php` menjadi seperti ini:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     * Fungsi ini dijalankan saat migration dieksekusi (membuat tabel).
     */
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description')->nullable();
            $table->integer('price');
            $table->integer('stock')->default(0);
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     * Fungsi ini dijalankan saat migration dibatalkan (rollback).
     */
    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
```

### Penjelasan tiap bagian

#### Baris `use ...` (bagian atas)

Ini adalah perintah untuk "memanggil" file pustaka Laravel yang dibutuhkan.
Seperti meminjam alat dari kotak perkakas: `Migration`, `Blueprint`, `Schema`.
Kamu tidak perlu menghafalnya, biarkan saja.

#### `return new class extends Migration`

Ini adalah cara Laravel mendefinisikan sebuah migration.
Anggap saja seperti "KTP migration" - identitas bahwa file ini adalah migration.

#### Fungsi `up()`

Fungsi `up()` dijalankan **saat migration dieksekusi** (tabel dibuat).
Di sinilah kita menulis **struktur tabel** yang ingin dibuat.

#### Fungsi `down()`

Fungsi `down()` dijalankan **saat migration dibatalkan (rollback)**.
Biasanya isinya kebalikan dari `up()`. Karena `up()` membuat tabel `products`,
maka `down()` menghapus tabel `products`.

#### Bagian dalam `up()`

```php
Schema::create('products', function (Blueprint $table) {
    // ... definisi kolom di sini
});
```

Artinya: "Buat tabel baru bernama `products`, dengan kolom-kolom sebagai berikut..."

#### Penjelasan tiap kolom

| Kode                                     | Arti                                                                 |
| ---------------------------------------- | -------------------------------------------------------------------- |
| `$table->id();`                          | Kolom `id` otomatis (primary key, auto-increment)                    |
| `$table->string('name');`               | Kolom `name` tipe VARCHAR (teks pendek, maksimal 255 karakter)       |
| `$table->text('description')->nullable();` | Kolom `description` tipe TEXT (teks panjang), boleh dikosongkan    |
| `$table->integer('price');`             | Kolom `price` tipe INT (bilangan bulat)                              |
| `$table->integer('stock')->default(0);` | Kolom `stock` tipe INT, default 0 jika tidak diisi                   |
| `$table->timestamps();`                  | Otomatis membuat kolom `created_at` dan `updated_at`                 |

### Analogi: Formulir Pendaftaran

Bayangkan kamu membuat formulir pendaftaran untuk karyawan baru:

- `id()` = Nomor induk karyawan (otomatis).
- `string('name')` = Isian "Nama Lengkap" (wajib diisi).
- `text('description')->nullable()` = Isian "Catatan tambahan" (boleh kosong).
- `integer('price')` = Isian "Gaji" (harus angka).
- `integer('stock')->default(0)` = Isian "Cuti tersisa", kalau kosong dianggap 0.
- `timestamps()` = Stempel tanggal otomatis di setiap formulir.

---

## 6. Menjalankan Migration

Setelah file migration diisi, kita perlu **menjalankannya** supaya tabel benar-benar dibuat di database.

### Perintahnya

Di terminal (di dalam folder project), jalankan:

```bash
php artisan migrate
```

Artinya: "Tolong jalankan semua migration yang belum dijalankan."

### Hasil yang diharapkan

Kamu akan melihat output seperti ini:

```
2026_07_16_103000_create_products_table .............................. DONE
```

Artinya: migration berhasil dijalankan, tabel `products` sudah dibuat di database.

### Memeriksa hasil di phpMyAdmin (opsional)

1. Buka http://localhost/phpmyadmin
2. Pilih database `toko_produk`
3. Kamu akan melihat tabel `products` beserta kolom-kolomnya.
4. Kamu juga akan melihat tabel bernama `migrations` - ini dipakai Laravel
   untuk melacak migration mana yang sudah dijalankan.

> Catatan: Tabel `products` **masih kosong**, belum ada data produk.
> Di tahap CRUD selanjutnya, kita akan mengisinya.

---

## 7. Menjalankan dan Membatalkan Migration (Opsional)

### Melihat daftar migration

```bash
php artisan migrate:status
```

Akan menampilkan daftar migration dan apakah sudah dijalankan (Ran = yes) atau belum (Ran = no).

### Membatalkan migration terakhir (rollback)

Jika kamu melakukan kesalahan pada migration terakhir, bisa dibatalkan:

```bash
php artisan migrate:rollback
```

Artinya: "Batalkan migration terakhir." Fungsi `down()` akan dijalankan,
tabel `products` akan dihapus.

Setelah memperbaiki file migration, jalankan ulang:

```bash
php artisan migrate
```

### Menghapus semua dan ulangi (fresh)

Untuk belajar, kadang kita ingin mulai bersih:

```bash
php artisan migrate:fresh
```

Artinya: "Hapus semua tabel lalu jalankan semua migration dari awal."

> Peringatan: `migrate:fresh` akan **menghapus semua data** di database.
> Jangan dipakai di produksi.

---

## 8. Ringkasan Alur Tahap Ini

```
1. php artisan make:migration create_products_table
        |
        v
2. Buka file di database/migrations/..._create_products_table.php
        |
        v
3. Isi dengan struktur tabel products (id, name, description, price, stock, timestamps)
        |
        v
4. php artisan migrate
        |
        v
5. Tabel products dibuat di database (masih kosong)
```

---

## 9. Checklist Keberhasilan Tahap Ini

Centang (`x`) jika sudah selesai:

- [ ] Saya paham bahwa migration adalah instruksi pembuatan tabel di database.
- [ ] Saya sudah membuat file migration dengan `php artisan make:migration create_products_table`.
- [ ] File migration berisi kolom: `id`, `name`, `description`, `price`, `stock`, `created_at`, `updated_at`.
- [ ] Saya sudah menjalankan `php artisan migrate` tanpa error.
- [ ] Tabel `products` muncul di phpMyAdmin / database.
- [ ] Saya mengerti cara rollback jika ada kesalahan.

Jika semua sudah tercentang, struktur tabel produk sudah siap.

---

## 10. Penutup

Pada tahap ini kamu sudah:

- Membuat **migration** pertama.
- Mendefinisikan **struktur tabel `products`** lewat kode PHP.
- Menjalankan migration sehingga tabel benar-benar ada di database.

Tabel `products` saat ini masih **kosong**, belum berisi produk apa pun.
Di tahap selanjutnya, kita akan mulai belajar tentang **Model** - yaitu
jembatan antara kode Laravel dengan tabel di database.

### Jika sudah paham tahap ini, ketik:

```
lanjut
```

Jangan lanjut sebelum kamu benar-benar siap. Pelan-pelan, asal paham.
