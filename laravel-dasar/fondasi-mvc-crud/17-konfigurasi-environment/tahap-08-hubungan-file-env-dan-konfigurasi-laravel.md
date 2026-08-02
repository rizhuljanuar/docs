# Tahap 8 — Hubungan File `.env` dan File Konfigurasi Laravel

> Fokus: memahami bagaimana Laravel memakai nilai dari `.env` melalui file konfigurasi, sehingga database tidak perlu ditulis langsung di controller, model, atau file aplikasi lainnya.

## Melanjutkan dari tahap 7

Pada tahap 7, kita sudah memeriksa koneksi ke database local dengan:

```bash
php artisan db:show
```

Jika hasilnya menunjukkan `laravel_local`, Laravel berhasil membaca konfigurasi database dari `.env`.

Sekarang mungkin muncul pertanyaan:

> Jika controller Product atau seeder tidak menulis `laravel_local`, dari mana mereka tahu database yang harus dipakai?

Jawabannya: Laravel memakai **file konfigurasi** sebagai penghubung antara `.env` dan source code aplikasi.

## Tiga bagian yang bekerja bersama

Untuk memahami alurnya, ingat tiga bagian berikut:

1. **`.env`** menyimpan nilai yang bisa berbeda di setiap komputer atau server.
2. **file di folder `config/`** membaca nilai tersebut dan menyusunnya menjadi konfigurasi Laravel.
3. **kode aplikasi** memakai konfigurasi Laravel, bukan menulis database secara langsung.

Alurnya seperti ini:

```text
.env
DB_DATABASE=laravel_local
        ↓ dibaca oleh
config/database.php
        ↓ digunakan oleh
Model, Controller, Migration, Seeder, dan Query Laravel
        ↓
Database laravel_local
```

## Analogi: catatan alamat, petugas pengatur, dan pekerja toko

Bayangkan kamu memiliki toko dengan banyak pekerja.

- File `.env` adalah **catatan alamat khusus**. Isinya dapat berbeda antara toko latihan dan toko asli.
- File konfigurasi adalah **petugas pengatur**. Ia membaca catatan alamat dan memberi petunjuk yang benar kepada pekerja.
- Controller, model, migration, dan seeder adalah **pekerja toko**. Mereka cukup mengikuti petunjuk dari petugas pengatur.

Pekerja tidak perlu menghafal atau menulis alamat gudang sendiri.

Begitu pula dengan Laravel. Controller Product tidak perlu menulis nama `laravel_local`. Laravel sudah mengatur koneksi database melalui konfigurasi.

## File `.env`: tempat nilai khusus environment

Kita sudah mengenal contoh nilai berikut di `.env`:

```env
APP_NAME="CRUD Product"
APP_ENV=local
APP_DEBUG=true

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_local
DB_USERNAME=root
DB_PASSWORD=
```

Nilai tersebut cocok untuk contoh aplikasi di komputer local.

Pada server production, file `.env` di server dapat memiliki nilai database berbeda. Source code Laravel tetap sama karena source code tidak perlu tahu nama database secara langsung.

## Folder `config/`: tempat aturan konfigurasi Laravel

Pada root project Laravel, terdapat folder:

```text
config/
```

Folder ini berisi file konfigurasi untuk berbagai bagian aplikasi, misalnya:

```text
config/
├── app.php
├── database.php
├── cache.php
├── filesystems.php
├── mail.php
└── ...
```

Untuk database, file yang berkaitan adalah:

```text
config/database.php
```

Kamu tidak perlu mengubah file ini pada tahap ini. Kita hanya akan melihat bentuk sederhananya agar memahami alur konfigurasi.

## Bagaimana `config/database.php` membaca `.env`?

Di dalam konfigurasi database Laravel 13+, terdapat pola seperti ini untuk koneksi MySQL:

```php
'mysql' => [
    'driver' => env('DB_CONNECTION', 'mysql'),
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
],
```

Tidak perlu menghafalkan kode ini. Perhatikan polanya saja:

```php
env('NAMA_SETTING', 'nilai cadangan')
```

Artinya, Laravel mencoba membaca nilai dari `.env`.

Contoh:

```php
env('DB_DATABASE', 'laravel')
```

Maknanya:

1. Cari nilai `DB_DATABASE` di file `.env`.
2. Jika ditemukan, gunakan nilai tersebut, misalnya `laravel_local`.
3. Jika tidak ditemukan, gunakan nilai cadangan `laravel`.

Karena `.env` local kita berisi:

```env
DB_DATABASE=laravel_local
```

maka konfigurasi database Laravel akan memakai `laravel_local`.

> Nilai `laravel` pada contoh kode adalah nilai cadangan, bukan nama database yang wajib kamu gunakan.

## Mengapa `env()` berada di file konfigurasi?

Pada materi awal, kita belajar agar tidak menulis nama database langsung di source code.

File konfigurasi adalah tempat yang tepat bagi Laravel untuk membaca `env()` karena ia menjadi satu pintu pengaturan.

Controller, model, dan seeder tidak perlu melakukan ini:

```php
$databaseName = env('DB_DATABASE');
```

Dan tidak perlu melakukan ini:

```php
$databaseName = 'laravel_local';
```

Keduanya bukan cara yang perlu dipakai dalam controller atau model aplikasi CRUD Product.

Sebagai gantinya, kode aplikasi menggunakan fitur Laravel seperti Eloquent:

```php
$products = Product::query()->latest()->get();
```

Saat query itu berjalan, Laravel memakai koneksi database yang sudah diatur melalui `config/database.php` dan `.env`.

Dengan begitu, controller tetap fokus mengambil Product. Controller tidak perlu mengurus nama database, alamat MySQL, username, atau password.

## Hubungan dengan kode dari materi sebelumnya

Berikut contoh bagaimana fitur yang sudah dibuat memakai konfigurasi database secara tidak langsung:

| Kode atau fitur | Yang dilakukan | Koneksi yang dipakai |
| --- | --- | --- |
| `Product::query()` | Membaca atau mencari Product | Konfigurasi database dari `.env` |
| `Category::query()` | Membaca Category | Konfigurasi database dari `.env` |
| Migration | Membuat atau mengubah tabel | Konfigurasi database dari `.env` |
| `Product::factory()` | Membuat data Product dummy | Konfigurasi database dari `.env` saat `create()` dipanggil |
| `php artisan db:seed` | Menjalankan seeder Category dan Product | Konfigurasi database dari `.env` |
| `php artisan db:show` | Memeriksa database aktif | Konfigurasi database dari `.env` |

Semua tetap menuju `laravel_local` selama `.env` local berisi `DB_DATABASE=laravel_local` dan setting koneksi lain benar.

## Menggunakan `config()` di source code

Saat source code perlu membaca konfigurasi Laravel, gunakan helper `config()`.

Contoh dari tahap 3:

```blade
{{ config('app.name') }}
```

Kode ini mengambil nama aplikasi dari konfigurasi Laravel. Nilai `APP_NAME` pada `.env` dibaca lebih dulu oleh `config/app.php`, kemudian dapat diakses melalui `config('app.name')`.

Cara berpikirnya:

```text
APP_NAME di .env
        ↓
config/app.php
        ↓
config('app.name') di Blade atau PHP
```

Untuk materi ini, kamu tidak perlu menambahkan `config()` ke controller Product atau Blade. Contoh tersebut hanya menunjukkan cara Laravel memakai konfigurasi secara rapi.

## Mengapa jangan memakai `env()` langsung di controller atau Blade?

Laravel dapat menyimpan konfigurasi menjadi cache agar aplikasi lebih cepat, terutama di production.

Karena itu, penggunaan `env()` sebaiknya berada di file konfigurasi dalam folder `config/`. Di controller, model, command, seeder, atau Blade, gunakan nilai dari `config()` atau fitur Laravel yang sudah tersedia.

Contoh yang tidak perlu dibuat pada controller:

```php
$environment = env('APP_ENV');
```

Contoh yang lebih sesuai jika benar-benar diperlukan:

```php
$environment = config('app.env');
```

Namun, untuk aplikasi CRUD Product yang telah dibuat, kamu belum membutuhkan pengecekan environment di controller. Jangan menambah kode hanya karena mengetahui helper ini.

## Jangan mengubah `config/database.php` untuk mengganti database local

Untuk mengganti database local dari `laravel` ke `laravel_local`, ubah nilai ini di `.env`:

```env
DB_DATABASE=laravel_local
```

Jangan mengganti nilai cadangan di `config/database.php` menjadi nama database pribadi kamu.

Jika kamu menulis `laravel_local` langsung di `config/database.php`, kode konfigurasi menjadi khusus untuk komputer kamu dan sulit dipakai di production atau komputer developer lain.

Ringkasnya:

| Tujuan | Tempat yang tepat |
| --- | --- |
| Mengubah nama database untuk komputer local | `.env` local |
| Menentukan pola konfigurasi database Laravel | `config/database.php` |
| Mengambil nilai konfigurasi di source code jika diperlukan | `config()` |
| Menulis password atau nama database langsung di controller/model | Jangan dilakukan |

## Catatan tentang configuration cache

Pada tahap 7, kita sudah mengenal perintah:

```bash
php artisan config:clear
```

Perintah itu berguna jika Laravel masih memakai konfigurasi lama setelah `.env` diubah.

Alurnya sederhana:

```text
Ubah .env
    ↓
Laravel masih membaca nilai lama?
    ↓ ya
Jalankan php artisan config:clear
    ↓
Periksa lagi dengan php artisan db:show
```

Kamu tidak perlu menjalankan `config:clear` setiap kali mengubah `.env`. Gunakan hanya saat memang Laravel masih membaca nilai lama atau saat materi berikutnya memintanya.

## Ringkasan tahap 8

- `.env` menyimpan nilai yang berbeda untuk local dan production.
- File dalam folder `config/` membaca nilai `.env` dan mengubahnya menjadi konfigurasi Laravel.
- `config/database.php` mengatur pola koneksi database dan membaca `DB_...` dari `.env`.
- Controller, model, migration, dan seeder memakai konfigurasi tersebut secara otomatis.
- Gunakan `config()` jika source code perlu membaca konfigurasi aplikasi.
- Jangan memakai `env()` langsung di controller, model, seeder, command, atau Blade.
- Untuk mengganti database local, ubah `.env`, bukan menulis nama database di `config/database.php`.
- `php artisan config:clear` dapat dipakai jika Laravel masih membaca konfigurasi lama.

Tahap berikutnya akan membahas lebih dalam alasan keamanan dan perawatan kode, terutama mengapa konfigurasi database tidak boleh ditulis langsung di source code.

---

**Apakah kamu ingin lanjut ke tahap 9: memahami risiko hardcode konfigurasi database di source code?**
