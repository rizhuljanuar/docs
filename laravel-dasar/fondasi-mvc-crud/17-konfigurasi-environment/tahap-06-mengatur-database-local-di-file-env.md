# Tahap 6 — Mengatur Database Local `laravel_local` di File `.env`

> Fokus: menyesuaikan koneksi database untuk aplikasi Laravel yang berjalan di komputer sendiri, dengan aman dan tanpa mengubah setting production.

## Melanjutkan dari tahap 5

Pada tahap 5, kita sudah mengenal enam setting database:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_local
DB_USERNAME=root
DB_PASSWORD=
```

Sekarang kita akan memakai pemahaman tersebut untuk menyesuaikan file `.env` pada **project Laravel CRUD Product di komputer local**.

Tujuannya sederhana: saat aplikasi, migration, atau seeder berjalan di komputer kamu, semuanya menggunakan database latihan bernama `laravel_local`.

## Sebelum mengubah `.env`, pastikan tiga hal ini

Jangan langsung mengedit `.env`. Periksa dulu hal berikut:

1. **Kamu membuka folder project Laravel yang benar.**
   File `.env` yang ingin diubah berada di root project Laravel, sejajar dengan file `artisan`.

2. **MySQL local sedang berjalan.**
   Jika memakai Laragon, XAMPP, atau layanan MySQL lain, pastikan layanan MySQL sudah dinyalakan.

3. **Database `laravel_local` sudah ada.**
   File `.env` hanya memberi tahu Laravel nama database yang akan digunakan. File ini tidak otomatis membuat database MySQL baru.

Analogi sederhananya:

- `.env` adalah alamat tujuan pada surat;
- MySQL adalah gedung penyimpanan;
- database `laravel_local` adalah ruangan di dalam gedung.

Menulis alamat ruangan di surat tidak otomatis membangun ruangannya. Database tersebut harus sudah dibuat di MySQL terlebih dahulu.

> Jangan melakukan langkah ini pada server production. Tahap ini khusus untuk project local di komputer kamu.

## Buka file `.env` pada project Laravel

Di editor kode, buka folder project Laravel CRUD Product kamu. Kemudian buka file:

```text
.env
```

Cari bagian yang diawali dengan `DB_`, misalnya:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

Nilai awal dapat berbeda, terutama pada `DB_DATABASE`, `DB_USERNAME`, dan `DB_PASSWORD`. Jangan menghapus baris lain yang belum kamu pahami.

## Atur nama database local

Karena contoh kasus kita memakai database local bernama `laravel_local`, ubah bagian `DB_DATABASE` menjadi:

```env
DB_DATABASE=laravel_local
```

Contoh konfigurasi lengkap untuk setup MySQL local yang umum:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_local
DB_USERNAME=root
DB_PASSWORD=
```

Arti konfigurasi tersebut:

| Baris | Makna pada komputer local |
| --- | --- |
| `DB_CONNECTION=mysql` | Laravel memakai MySQL |
| `DB_HOST=127.0.0.1` | MySQL berjalan di komputer yang sama |
| `DB_PORT=3306` | MySQL memakai port umum 3306 |
| `DB_DATABASE=laravel_local` | Laravel memakai database latihan `laravel_local` |
| `DB_USERNAME=root` | Laravel masuk memakai akun MySQL `root` |
| `DB_PASSWORD=` | Akun tersebut tidak memakai password pada contoh setup ini |

## Sesuaikan username dan password dengan komputer kamu

Bagian ini penting: `root` dan password kosong hanyalah contoh setup local yang umum. Nilai yang benar bergantung pada MySQL di komputer kamu.

Gunakan tabel ini sebagai panduan:

| Kondisi MySQL local kamu | Contoh `.env` |
| --- | --- |
| Akun `root` tanpa password | `DB_USERNAME=root` dan `DB_PASSWORD=` |
| Akun `root` memakai password | `DB_USERNAME=root` dan isi `DB_PASSWORD` dengan password tersebut |
| Memakai akun MySQL khusus | Isi `DB_USERNAME` dan `DB_PASSWORD` sesuai akun itu |

Jangan menebak password. Jika kamu tidak tahu akun atau password MySQL, periksa konfigurasi dari alat yang kamu gunakan, misalnya Laragon, XAMPP, phpMyAdmin, atau setup MySQL kamu.

> Jangan menaruh password asli di materi, screenshot publik, commit Git, atau GitHub. Password hanya ditulis pada file `.env` local milik kamu sendiri.

## Contoh aman dan tidak aman

### Aman: konfigurasi hanya di `.env`

```env
DB_DATABASE=laravel_local
```

Nilai ini khusus untuk komputer local kamu dan dipisahkan dari source code aplikasi.

### Tidak aman: menulis nama database langsung di source code

```php
$databaseName = 'laravel_local';
```

Kode seperti ini membuat aplikasi sulit dipindahkan ke server lain. Selain itu, password database tidak pernah boleh ditulis langsung di controller, model, route, Blade, migration, factory, atau seeder.

Aplikasi CRUD Product tetap menggunakan model, controller, Blade, migration, factory, dan seeder yang telah dibuat sebelumnya. Yang berubah hanya arah koneksi database dari file `.env`.

## Pastikan setting aplikasi masih local

Sebelum menyimpan perubahan, periksa juga bagian aplikasi berikut:

```env
APP_ENV=local
APP_DEBUG=true
```

Kombinasi ini sesuai untuk belajar dan mengembangkan aplikasi di komputer sendiri.

Jika disatukan, bagian penting `.env` local kamu dapat berbentuk seperti ini:

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

Nilai `DB_USERNAME` dan `DB_PASSWORD` pada contoh ini tetap harus disesuaikan dengan MySQL local kamu.

## Simpan, tetapi jangan jalankan seeder dulu

Setelah mengubah `.env`, simpan file tersebut.

Pada tahap ini, kita **belum menjalankan** perintah berikut:

```bash
php artisan migrate
php artisan db:seed
```

Alasannya, kita perlu memeriksa lebih dahulu apakah Laravel benar-benar dapat memakai koneksi database local yang baru diatur. Itu akan menjadi fokus tahap berikutnya.

Hal ini penting karena:

- `php artisan migrate` dapat membuat atau mengubah struktur tabel;
- `php artisan db:seed` dari materi 16 dapat menambahkan Category dan Product dummy;
- kedua perintah harus dipastikan menuju database `laravel_local`, bukan database lain.

## Jika Laravel masih membaca nilai lama

Saat belajar di local, Laravel biasanya membaca perubahan `.env` pada request atau perintah Artisan berikutnya.

Namun, Laravel 13+ dapat memakai configuration cache. Jika suatu saat `.env` sudah diubah tetapi Laravel tetap membaca nilai lama, cache konfigurasi mungkin perlu dibersihkan atau dibuat ulang. Kita akan membahas tindakan tersebut saat diperlukan, setelah kamu memahami cara mengecek koneksi.

Jangan mengganti banyak nilai secara acak untuk mengatasi masalah. Periksa enam setting `DB_...` satu per satu.

## Checklist tahap 6

- [ ] Saya membuka `.env` dari root project Laravel CRUD Product, bukan dari folder dokumentasi.
- [ ] MySQL local sedang berjalan.
- [ ] Database `laravel_local` sudah dibuat di MySQL.
- [ ] `APP_ENV=local` dan `APP_DEBUG=true` masih sesuai untuk belajar di komputer local.
- [ ] `DB_CONNECTION` sesuai dengan database yang dipakai, yaitu `mysql` pada materi ini.
- [ ] `DB_HOST` dan `DB_PORT` sesuai setup MySQL local saya.
- [ ] `DB_DATABASE=laravel_local` sudah ditulis dengan benar.
- [ ] `DB_USERNAME` dan `DB_PASSWORD` sesuai akun MySQL local saya.
- [ ] Password asli tidak ditulis pada source code atau dibagikan secara publik.
- [ ] Saya belum menjalankan migration atau seeder sebelum koneksi diperiksa.

## Ringkasan tahap 6

- Konfigurasi database local diubah pada file `.env` project Laravel.
- `DB_DATABASE=laravel_local` membuat Laravel menargetkan database latihan tersebut.
- File `.env` tidak membuat database otomatis, database `laravel_local` harus sudah ada di MySQL.
- Username dan password MySQL setiap komputer dapat berbeda.
- Jangan mengubah file `.env` production saat melakukan latihan local.
- Jangan menjalankan migration atau seeder sebelum yakin Laravel terhubung ke database local yang tepat.

Tahap berikutnya akan memeriksa koneksi database dengan aman sebelum melanjutkan ke migration atau seeder dari materi sebelumnya.

---

**Apakah kamu ingin lanjut ke tahap 7: memeriksa koneksi Laravel ke database `laravel_local`?**
