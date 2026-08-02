# Tahap 2 — Mengenal File `.env` di Laravel

> Fokus: mengenal bentuk dasar file `.env` dan beberapa setting penting, tanpa mengubah konfigurasi aplikasi terlebih dahulu.

## Melanjutkan dari tahap 1

Pada tahap 1, kita sudah belajar bahwa aplikasi Laravel yang sama dapat berjalan di tempat berbeda.

Misalnya:

- di komputer local, aplikasi CRUD Product memakai database `laravel_local`;
- di server production, aplikasi yang sama dapat memakai database lain.

Laravel menyimpan setting yang berbeda itu di file **`.env`**. Pada tahap ini, kita akan mengenal cara membaca file tersebut. Kita belum akan mengubah nilai apa pun.

## Di mana file `.env` berada?

Di dalam project Laravel 13+, file `.env` berada di folder paling luar atau **root project**.

Contoh susunannya:

```text
crud-product/
├── app/
├── bootstrap/
├── config/
├── database/
├── public/
├── resources/
├── routes/
├── .env
├── .env.example
├── artisan
└── composer.json
```

File `.env` berada sejajar dengan file `artisan` dan `composer.json`, bukan di dalam folder `app`, `config`, atau `database`.

> Untuk materi ini, buka file `.env` dari project Laravel CRUD Product kamu sendiri. Folder dokumentasi yang sedang kita tulis bukan folder aplikasi Laravel, sehingga file `.env` aplikasi tidak berada di sini.

## Cara membaca satu baris di `.env`

Isi file `.env` terlihat seperti daftar baris `NAMA_SETTING=ISI_SETTING`.

Contoh:

```env
APP_ENV=local
```

Bagian di sebelah kiri tanda sama dengan (`=`) adalah **nama setting**. Bagian di sebelah kanan adalah **nilainya**.

| Bagian | Contoh | Arti sederhana |
| --- | --- | --- |
| Nama setting | `APP_ENV` | Nama informasi yang ingin dibaca Laravel |
| Tanda penghubung | `=` | Memisahkan nama dan nilai |
| Nilai setting | `local` | Isi informasi untuk setting tersebut |

Analogi sederhananya seperti label dan isi sebuah laci:

```text
Label laci: APP_ENV
Isi laci: local
```

Laravel membaca label untuk mengetahui informasi apa yang tersedia, lalu memakai isi yang sesuai.

## Beberapa setting penting yang akan sering kamu lihat

File `.env` bawaan Laravel berisi cukup banyak baris. Jangan khawatir, kamu tidak perlu memahami semuanya sekarang.

Untuk tahap ini, cukup kenali tiga kelompok berikut.

### 1. Setting aplikasi

Contoh:

```env
APP_NAME=Laravel
APP_ENV=local
APP_DEBUG=true
```

| Setting | Arti sederhana |
| --- | --- |
| `APP_NAME` | Nama aplikasi yang dapat digunakan Laravel pada beberapa tampilan atau informasi aplikasi |
| `APP_ENV` | Penanda tempat aplikasi berjalan, misalnya `local` atau `production` |
| `APP_DEBUG` | Menentukan apakah Laravel boleh menampilkan detail error untuk membantu developer |

Saat belajar di komputer sendiri, kamu biasanya melihat nilai seperti ini:

```env
APP_ENV=local
APP_DEBUG=true
```

Kita akan membahas `APP_ENV` dan `APP_DEBUG` secara khusus pada tahap berikutnya. Untuk sekarang, cukup ingat bahwa keduanya membantu Laravel membedakan kondisi local dan production.

### 2. Setting koneksi database

Contoh bentuk umum:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_local
DB_USERNAME=root
DB_PASSWORD=
```

Baris-baris tersebut adalah informasi yang Laravel perlukan untuk menemukan database.

| Setting | Arti sederhana |
| --- | --- |
| `DB_CONNECTION` | Jenis database yang dipakai, misalnya MySQL |
| `DB_HOST` | Alamat tempat database berjalan |
| `DB_PORT` | Nomor pintu koneksi database |
| `DB_DATABASE` | Nama database yang ingin digunakan Laravel |
| `DB_USERNAME` | Nama pengguna untuk masuk ke database |
| `DB_PASSWORD` | Password pengguna database |

Belum perlu menghafalkan semuanya. Yang paling dekat dengan contoh kasus kamu adalah:

```env
DB_DATABASE=laravel_local
```

Baris itu memberi tahu Laravel agar memakai database local bernama `laravel_local`.

Pada tahap selanjutnya, kita akan mempelajari setiap setting database itu satu per satu sebelum mengatur koneksi database local.

### 3. Setting layanan lain

Kamu mungkin juga menemukan baris dengan awalan seperti ini:

```env
CACHE_STORE=database
SESSION_DRIVER=database
MAIL_MAILER=log
```

Baris tersebut berhubungan dengan fitur lain, seperti cache, session login, dan pengiriman email.

Untuk saat ini, jangan mengubahnya. Fokus kita masih pada environment dan database untuk aplikasi CRUD Product.

## Nilai kosong bukan berarti barisnya tidak berguna

Perhatikan contoh berikut:

```env
DB_PASSWORD=
```

Nilai di sebelah kanan memang kosong. Artinya, Laravel tetap membaca setting `DB_PASSWORD`, tetapi password database saat ini tidak diisi.

Ini sering terjadi pada setup database local tertentu. Namun, jangan langsung menyalin contoh ini ke project kamu. Username dan password database local setiap komputer bisa berbeda.

> Jangan pernah menuliskan password database asli dalam materi, chat, screenshot publik, commit Git, atau GitHub.

## Mengapa setiap komputer dapat memiliki `.env` yang berbeda?

Bayangkan dua developer mengerjakan source code aplikasi yang sama.

- Komputer A memakai database `laravel_local_jack`.
- Komputer B memakai database `laravel_local_budi`.

Keduanya tetap dapat memakai controller, model Product, migration, dan Blade yang sama. Masing-masing cukup memakai file `.env` dengan nilai `DB_DATABASE` yang sesuai di komputernya.

```text
Source code Laravel sama
        ↓
File .env tiap komputer dapat berbeda
        ↓
Laravel terhubung ke database yang sesuai
```

Inilah alasan `.env` dipisahkan dari source code utama.

## Jangan mengubah `.env` secara sembarangan

File `.env` sangat penting karena dapat menentukan database yang dipakai aplikasi. Salah menulis satu nilai dapat membuat Laravel gagal terhubung atau, yang lebih berbahaya, terhubung ke database yang tidak kamu maksud.

Sebelum mengubah `.env`, biasakan untuk:

1. Memastikan kamu sedang membuka project Laravel yang benar.
2. Memeriksa nilai `APP_ENV` dan nama database yang sedang dipakai.
3. Tidak menyalin password dari contoh internet tanpa memahami asalnya.
4. Tidak menjalankan seeder atau migration yang mengubah data sebelum yakin database-nya benar.

Hal ini berkaitan langsung dengan materi 16. Perintah seperti `php artisan db:seed` akan memakai koneksi database dari `.env` yang aktif.

## Ringkasan tahap 2

- File `.env` berada di root project Laravel.
- Satu baris `.env` berbentuk `NAMA_SETTING=nilai`.
- `APP_...` berkaitan dengan aplikasi dan environment.
- `DB_...` berkaitan dengan koneksi database.
- Nilai `.env` dapat berbeda pada setiap komputer atau server.
- Jangan mengubah atau membagikan password database sembarangan.
- Kita belum mengubah `.env` pada tahap ini.

Tahap berikutnya akan membahas `APP_NAME` dan `APP_ENV`, agar kamu semakin memahami bagaimana Laravel mengenali aplikasi dan lingkungan tempat ia berjalan.

---

**Apakah kamu ingin lanjut ke tahap 3: memahami `APP_NAME` dan `APP_ENV` di Laravel?**
