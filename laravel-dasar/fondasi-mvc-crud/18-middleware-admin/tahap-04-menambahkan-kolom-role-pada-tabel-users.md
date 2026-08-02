# Tahap 4 — Menambahkan Kolom `role` pada Tabel `users`

> Fokus: menambahkan tempat penyimpanan role `admin` dan `user` pada database dengan migration Laravel 13+.

Pada tahap 3, kita sudah memahami bahwa Laravel memerlukan data role untuk menjawab pertanyaan berikut:

> “User yang sudah login ini boleh masuk ke halaman admin atau tidak?”

Saat ini tabel `users` umumnya mempunyai data dasar seperti nama, email, dan password. Kita akan menambahkan satu kolom baru bernama `role` agar setiap user juga mempunyai tanda sebagai `admin` atau `user`.

## Analogi menambahkan kolom pada buku daftar anggota

Bayangkan tabel `users` seperti buku daftar anggota toko.

Awalnya buku tersebut mempunyai kolom seperti ini:

| Nama | Email | Password |
| --- | --- | --- |
| Andi | andi@example.com | tersimpan aman dalam bentuk hash |
| Siti | siti@example.com | tersimpan aman dalam bentuk hash |

Agar penjaga dapat membedakan pelanggan dan pengelola, buku itu memerlukan kolom baru:

| Nama | Email | Role |
| --- | --- | --- |
| Andi | andi@example.com | `user` |
| Siti | siti@example.com | `admin` |

Di Laravel, cara yang aman dan rapi untuk mengubah struktur tabel database adalah memakai **migration**.

## Apa itu migration?

**Migration** adalah file PHP yang mencatat perubahan struktur database.

Anggap migration seperti instruksi tertulis:

> “Tambahkan kolom `role` pada tabel `users`.”

Laravel menjalankan instruksi ini dengan perintah Artisan. Dengan begitu, perubahan database tidak dilakukan secara manual dan dapat diikuti oleh developer lain yang memakai project yang sama.

Pada materi CRUD Product sebelumnya, migration digunakan untuk membuat atau mengubah tabel seperti Product dan Category. Sekarang kita menggunakannya untuk menambah kolom pada tabel `users` yang sudah ada.

## Mengapa tidak mengubah migration awal `create_users_table`?

Project Laravel baru biasanya memiliki migration awal bernama mirip seperti ini:

```text
database/migrations/0001_01_01_000000_create_users_table.php
```

File itu membuat tabel `users` pertama kali. Di dalamnya ada bagian seperti ini:

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->timestamp('email_verified_at')->nullable();
    $table->string('password');
    $table->rememberToken();
    $table->timestamps();
});
```

Jangan langsung menambahkan `$table->string('role')` ke migration awal jika migration tersebut **sudah pernah dijalankan** pada database kamu.

Mengapa? Laravel mencatat migration yang sudah dijalankan. Mengubah file lama tidak membuat Laravel otomatis menjalankan ulang perubahan baru di dalam file tersebut.

Cara yang benar adalah membuat migration baru, khusus untuk menambahkan kolom role.

## Sebelum menjalankan perintah database

Migration mengubah struktur database aktif. Karena itu, lakukan pemeriksaan dulu, terutama karena materi 17 menjelaskan bahwa database ditentukan oleh file `.env`.

1. Buka file `.env` di root project Laravel.
2. Pastikan kamu memakai database local untuk belajar, misalnya `APP_ENV=local`.
3. Pastikan `DB_DATABASE` menunjuk database latihan yang benar.
4. Dari root project Laravel, jalankan:

```bash
php artisan db:show
```

Perintah tersebut hanya memeriksa informasi database. Pastikan hasilnya menunjukkan database yang memang aman untuk diubah.

> Jangan menjalankan migration secara sembarangan pada database production. Migration dapat membuat atau mengubah tabel pada database yang sedang aktif.

## Langkah 1, buat migration baru

Dari root project Laravel, jalankan perintah ini:

```bash
php artisan make:migration add_role_to_users_table --table=users
```

Penjelasan setiap bagian:

- `php artisan` menjalankan perintah bawaan Laravel.
- `make:migration` meminta Laravel membuat file migration baru.
- `add_role_to_users_table` adalah nama yang menjelaskan tujuan migration.
- `--table=users` memberi tahu Laravel bahwa migration ini akan mengubah tabel `users` yang sudah ada.

Setelah perintah berhasil, Laravel membuat file baru di folder:

```text
database/migrations/
```

Nama lengkap file memiliki waktu di bagian depan, sehingga setiap komputer dapat berbeda. Contohnya:

```text
2026_08_02_120000_add_role_to_users_table.php
```

Jangan membuat nama waktu file secara manual. Biarkan Laravel membuatnya.

## Langkah 2, isi migration untuk kolom role

Buka file migration yang baru dibuat. Laravel akan menyiapkan method `up()` dan `down()`.

Ubah isi bagian migration menjadi seperti ini:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('role')->default('user');
        });
    }

    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('role');
        });
    }
};
```

Mari pahami setiap bagian pentingnya.

### `Schema::table('users', ...)`

```php
Schema::table('users', function (Blueprint $table) {
    // Perubahan kolom ditulis di sini.
});
```

- `Schema` adalah alat Laravel untuk mengatur struktur tabel database.
- `table('users', ...)` berarti kita mengubah tabel yang sudah ada, yaitu `users`.
- `$table` adalah alat untuk menambahkan atau menghapus kolom di dalam tabel tersebut.

Kita memakai `Schema::table()`, bukan `Schema::create()`, karena tabel `users` sudah ada.

### `$table->string('role')->default('user');`

```php
$table->string('role')->default('user');
```

Baris ini melakukan dua hal:

- `string('role')` menambahkan kolom teks bernama `role`.
- `default('user')` menjadikan `user` sebagai nilai awal jika tidak ada role yang diberikan.

Setelah migration dijalankan, user lama dan user baru akan aman memiliki nilai awal:

```text
user
```

Ini penting karena user biasa adalah kondisi paling aman secara default. Tidak ada akun yang otomatis menjadi admin hanya karena kolom role baru dibuat.

### `up()`

```php
public function up(): void
```

Method `up()` berisi perubahan yang diterapkan saat kamu menjalankan:

```bash
php artisan migrate
```

Dalam materi ini, `up()` menambahkan kolom `role`.

### `down()`

```php
public function down(): void
```

Method `down()` berisi kebalikan dari `up()`. Jika migration terakhir perlu dibatalkan pada database local, Laravel tahu cara menghapus kolom `role`:

```php
$table->dropColumn('role');
```

Kita menulis `down()` agar migration dapat dikembalikan dengan rapi saat latihan. Jangan memakai rollback pada database penting tanpa memahami migration lain yang juga dapat ikut dibatalkan.

## Langkah 3, jalankan migration

Setelah memastikan `.env` dan file migration benar, jalankan:

```bash
php artisan migrate
```

Perintah ini menjalankan migration yang belum pernah dicatat Laravel sebagai sudah dijalankan.

Pada tahap ini, hasil yang diharapkan adalah:

```text
tabel users
    ↓
tambah kolom role
    ↓
nilai default untuk user lama dan baru: user
```

Sekarang gambaran data user menjadi:

| id | name | email | role |
| --- | --- | --- | --- |
| 1 | Andi | andi@example.com | `user` |
| 2 | Siti | siti@example.com | `user` |

Siti masih memiliki role `user` untuk sementara. Pada tahap berikutnya, kita akan menentukan bagaimana user baru otomatis memakai role `user` dan bagaimana satu akun latihan dapat diberi role `admin` dengan sengaja.

## Cara memeriksa hasil migration

Setelah migration selesai, kamu dapat memeriksa struktur tabel melalui database tool yang kamu gunakan, misalnya phpMyAdmin, TablePlus, atau aplikasi database lain.

Cari tabel `users`, lalu pastikan ada kolom:

```text
role
```

Nilai awal pada akun yang sudah ada seharusnya adalah:

```text
user
```

Kamu juga dapat menjalankan kembali:

```bash
php artisan db:show
```

Perintah itu membantu memastikan koneksi database tetap benar, tetapi tampilan kolom yang tersedia bergantung pada database driver dan tool yang kamu gunakan. Pemeriksaan paling jelas untuk kolom adalah melihat struktur tabel dengan tool database kamu.

## Catatan penting tentang model `User`

Pada tahap ini, kita baru menambahkan kolom ke database. Untuk **membaca** role dengan kode seperti ini:

```php
auth()->user()->role
```

Laravel dapat mengambil nilai kolom `role` dari model `User`.

Nanti saat kamu membuat atau memperbarui data user menggunakan mass assignment, misalnya `User::create([...])`, kolom `role` perlu diizinkan di model `User` melalui `$fillable`. Kita akan membahas kebutuhan itu pada tahap yang memang membuat atau memperbarui data role.

Jadi, jangan terburu-buru mengubah model `User` jika pada tahap ini kamu hanya mengikuti migration.

## Kesalahan umum yang perlu dihindari

### 1. Menjalankan `migrate:fresh` untuk menambah satu kolom

Jangan memakai:

```bash
php artisan migrate:fresh
```

hanya untuk mencoba migration role. Perintah tersebut menghapus semua tabel pada database aktif sebelum menjalankan ulang migration. Untuk tahap ini, gunakan perintah yang lebih aman:

```bash
php artisan migrate
```

### 2. Memberikan default `admin`

Jangan menulis:

```php
$table->string('role')->default('admin');
```

Akibatnya, semua akun yang belum diatur bisa menjadi admin. Gunakan default `user`.

### 3. Menyimpan role pada kolom nama

Jangan mengubah nama user menjadi `admin` untuk menandai pengelola. Buat kolom `role` khusus agar data nama dan hak akses tidak tercampur.

### 4. Mengubah `.env` agar migration berjalan

`.env` hanya menentukan database yang digunakan. Perubahan struktur tabel harus ditulis dalam migration, bukan dalam `.env`.

## Yang perlu diingat pada tahap ini

1. Role `admin` dan `user` harus disimpan sebagai data pada tabel `users`.
2. Migration adalah cara Laravel mencatat perubahan struktur database.
3. Buat migration baru dengan `php artisan make:migration`, jangan mengubah migration lama yang sudah dijalankan.
4. Kolom `role` memakai nilai default `user`, karena akun baru atau lama tidak boleh otomatis mendapat hak admin.
5. Selalu periksa `.env` dan database aktif sebelum menjalankan `php artisan migrate`.
6. Kita belum membuat middleware admin. Saat ini kita baru menyiapkan data yang akan diperiksa middleware nanti.

Tahap berikutnya akan membahas cara menjaga role default `user` ketika akun dibuat, termasuk hubungan migration, model `User`, dan proses pendaftaran akun.

---

**Apakah kamu ingin lanjut ke tahap 5: memastikan user baru otomatis memiliki role `user`?**
