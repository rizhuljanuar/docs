# Tahap 12 — Ringkasan dan Checklist Aman Konfigurasi Environment Laravel

> Penutup materi 17: menggunakan file `.env` agar aplikasi Laravel CRUD Product dapat memakai konfigurasi yang tepat di local dan production tanpa membocorkan rahasia atau salah memilih database.

## Apa yang sudah dipelajari?

Materi ini dimulai dari masalah sederhana:

> Komputer local memakai database `laravel_local`, tetapi server production memakai database yang berbeda. Bagaimana satu aplikasi Laravel dapat bekerja di kedua tempat?

Jawabannya adalah **environment configuration**.

Laravel memisahkan source code aplikasi dari setting yang dapat berbeda di setiap tempat.

```text
Source code CRUD Product tetap sama
        ↓
File .env local memakai setting local
        ↓
File .env production memakai setting production
        ↓
Laravel terhubung ke database yang sesuai
```

Dengan cara ini, controller, model, route, Blade, migration, factory, dan seeder tidak perlu menulis nama database, alamat server, username, atau password secara langsung.

## Ringkasan 12 tahap

| Tahap | Yang dipelajari |
| --- | --- |
| 1 | Environment adalah tempat atau kondisi aplikasi berjalan, seperti local dan production. |
| 2 | File `.env` berada di root project dan berisi baris `NAMA_SETTING=nilai`. |
| 3 | `APP_NAME` adalah nama aplikasi, sedangkan `APP_ENV` menandai environment. |
| 4 | `APP_DEBUG=true` membantu di local, sedangkan `APP_DEBUG=false` melindungi detail error di production. |
| 5 | Enam setting `DB_...` menjelaskan cara Laravel masuk ke database. |
| 6 | Database local `laravel_local` diatur melalui file `.env` project Laravel. |
| 7 | `php artisan db:show` dipakai untuk memeriksa koneksi database sebelum menjalankan perintah yang mengubah data. |
| 8 | `.env` dibaca melalui file dalam folder `config/`, terutama `config/database.php`. |
| 9 | Konfigurasi database tidak boleh di-hardcode di source code. |
| 10 | `.env` tidak boleh diunggah ke GitHub, sedangkan `.env.example` adalah template yang aman dibagikan. |
| 11 | Local dan production memakai `.env` serta database yang terpisah, tetapi memakai source code yang sama. |
| 12 | Menyatukan prinsip aman sebelum mengubah konfigurasi atau menjalankan perintah database. |

## Alur konfigurasi yang perlu diingat

Gunakan gambar alur sederhana ini saat lupa hubungan setiap bagian:

```text
.env
APP_ENV=local
DB_DATABASE=laravel_local
        ↓
config/app.php dan config/database.php
        ↓
Konfigurasi Laravel
        ↓
Controller, Model, Migration, Factory, Seeder, dan Query
        ↓
Database laravel_local
```

Misalnya, controller Product dapat tetap sederhana:

```php
$products = Product::query()->latest()->get();
```

Controller tersebut tidak perlu tahu nama database. Laravel sudah membaca `.env` melalui konfigurasi.

Pada server production, kode yang sama dapat membaca database production karena file `.env` milik server mempunyai nilai `DB_DATABASE` berbeda.

## Konfigurasi local yang umum

Saat belajar dan membuat fitur di komputer sendiri, bentuk penting `.env` dapat seperti ini:

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

Nilai `DB_USERNAME=root` dan password kosong hanya contoh. Selalu sesuaikan username, password, host, dan port dengan MySQL di komputer kamu.

Sebelum menjalankan perintah yang mengubah database, periksa bahwa:

```env
APP_ENV=local
DB_DATABASE=laravel_local
```

Kemudian jalankan:

```bash
php artisan db:show
```

Pastikan hasilnya tidak error dan menunjukkan database `laravel_local`.

## Konfigurasi production yang aman

Di server asli, prinsip dasar yang perlu diingat adalah:

```env
APP_ENV=production
APP_DEBUG=false
```

Selain itu, database production harus memakai host, nama database, username, dan password milik server tersebut.

Jangan menyalin file `.env` local ke production. Jangan juga menyalin `.env` production ke komputer local.

File `.env` production harus dibuat langsung di server atau dikelola oleh sistem secret yang aman, bukan dikirim melalui GitHub.

## Best practice utama

### 1. Ubah konfigurasi environment di `.env`, bukan di source code

Untuk mengganti database local, ubah:

```env
DB_DATABASE=laravel_local
```

Jangan menulis seperti ini pada controller, model, migration, factory, seeder, route, atau Blade:

```php
$databaseName = 'laravel_local';
```

### 2. Gunakan `env()` hanya di file konfigurasi

File seperti `config/database.php` memakai `env()` untuk membaca `.env`.

Di controller, model, seeder, command, atau Blade, jangan memanggil `env()` langsung. Jika perlu membaca konfigurasi aplikasi, gunakan `config()`.

Contoh:

```blade
{{ config('app.name') }}
```

Untuk CRUD Product yang sudah dibuat, kamu tidak perlu menambahkan `config()` baru jika memang tidak ada kebutuhan.

### 3. Jangan membagikan `.env`

`.env` dapat memuat password database, `APP_KEY`, API key, token, dan setting server.

Pastikan:

- `.env` ada di `.gitignore`;
- `.env` tidak muncul di `git status --short` sebelum commit;
- password dan token tidak dimasukkan ke dokumentasi, screenshot, chat publik, atau GitHub.

### 4. Bagikan `.env.example`, bukan `.env`

`.env.example` menunjukkan setting yang diperlukan tanpa membagikan nilai rahasia.

Saat project dipakai di komputer baru:

```bash
cp .env.example .env
php artisan key:generate
```

Setelah itu, pemilik komputer mengisi database local miliknya sendiri.

### 5. Periksa koneksi sebelum mengubah database

Sebelum menjalankan migration, seeder, atau perintah database lain:

```text
Periksa file .env
    ↓
Pastikan MySQL berjalan
    ↓
Jalankan php artisan db:show
    ↓
Pastikan nama database benar
    ↓
Baru jalankan perintah yang mengubah database
```

Jika Laravel masih membaca nilai lama setelah `.env` diubah, kamu dapat memakai:

```bash
php artisan config:clear
```

Kemudian periksa ulang dengan:

```bash
php artisan db:show
```

### 6. Pahami dampak perintah database

| Perintah | Fungsi | Dampak pada database aktif |
| --- | --- | --- |
| `php artisan db:show` | Memeriksa informasi database | Membaca saja, tidak mengubah data |
| `php artisan config:clear` | Menghapus cache konfigurasi | Tidak mengubah tabel atau data aplikasi |
| `php artisan migrate` | Menjalankan migration | Dapat membuat atau mengubah tabel |
| `php artisan db:seed` | Menjalankan seeder | Dapat menambah atau memperbarui data sesuai seeder |
| `php artisan migrate:fresh --seed` | Menghapus tabel, menjalankan migration, lalu seeder | Menghapus seluruh tabel pada database aktif |

Perintah terakhir sangat berbahaya jika database yang aktif adalah production. Gunakan hanya pada database local atau development yang memang aman untuk dihapus.

### 7. Data dummy hanya untuk tempat yang aman

Pada materi 16, kita membuat Category dan Product dummy dengan factory dan seeder.

Sebelum menjalankan:

```bash
php artisan db:seed
```

pastikan `.env` menunjuk ke database latihan seperti `laravel_local`.

Jangan memakai seeder data dummy pada production tanpa kebutuhan yang jelas dan tanpa memahami dampaknya.

### 8. Jangan menganggap `APP_ENV=production` sebagai tombol pengaman

`APP_ENV=production` hanya menandai environment. Nilai itu tidak otomatis menghentikan semua perintah berbahaya.

Tetap baca perintah dengan teliti, periksa database aktif, dan jangan menjalankan tindakan yang mengubah data production tanpa tanggung jawab serta persiapan yang sesuai.

## Checklist sebelum mengubah `.env`

- [ ] Saya membuka `.env` dari root project Laravel, bukan dari folder dokumentasi.
- [ ] Saya tahu apakah saya sedang bekerja di local atau production.
- [ ] Saya tidak menyalin setting dari internet tanpa memahami fungsinya.
- [ ] Saya mengubah nilai yang diperlukan saja.
- [ ] Saya tidak menulis password atau nama database langsung di source code.
- [ ] Jika setelah diubah Laravel membaca nilai lama, saya mempertimbangkan `php artisan config:clear`.

## Checklist sebelum migration atau seeder

- [ ] MySQL sedang berjalan.
- [ ] Saya memeriksa `APP_ENV` pada `.env`.
- [ ] Saya memeriksa `DB_DATABASE` pada `.env`.
- [ ] Saya menjalankan `php artisan db:show` dari root project Laravel.
- [ ] Hasil `db:show` menunjukkan database yang memang ingin saya pakai.
- [ ] Saya memahami dampak perintah yang akan dijalankan.
- [ ] Saya tidak menjalankan `migrate:fresh --seed` pada database penting atau production.
- [ ] Untuk seeder materi 16, saya yakin database aktif adalah database latihan yang aman.

## Checklist sebelum commit atau membagikan project

- [ ] File `.env` tidak ikut di-commit.
- [ ] `.env` tercantum pada `.gitignore`.
- [ ] Saya memeriksa `git status --short` sebelum commit.
- [ ] `.env.example` tidak berisi `APP_KEY`, password, token, atau secret asli.
- [ ] Source code tidak mengandung password, alamat database production, atau token asli.
- [ ] Developer lain dapat membuat `.env` mereka sendiri dari `.env.example`.

## Kesalahan umum dan cara memperbaikinya

| Kesalahan | Dampak | Cara memperbaiki |
| --- | --- | --- |
| Mengubah `config/database.php` untuk nama database local | Konfigurasi menjadi khusus satu komputer | Ubah `DB_DATABASE` pada `.env` local |
| Menulis password di controller atau seeder | Password dapat bocor ke Git | Pindahkan ke `.env`, lalu ganti password jika pernah dibagikan |
| Menjalankan seeder tanpa memeriksa database aktif | Data dummy dapat masuk database yang salah | Jalankan `php artisan db:show` terlebih dahulu |
| Menganggap `APP_DEBUG=false` menghapus error | Error tetap ada, tetapi detail disembunyikan | Perbaiki penyebab error melalui log dan konfigurasi yang benar |
| Menyalin `.env` local ke production | Production memakai setting local yang salah | Buat `.env` production sendiri di server |
| Mengunggah `.env` ke GitHub | Secret dapat bocor | Ganti semua secret terkait dan pastikan `.env` diabaikan Git |
| Menjalankan `migrate:fresh --seed` di production | Semua tabel aktif dapat terhapus | Jangan jalankan, gunakan prosedur deployment yang aman |

## Hubungan akhir dengan CRUD Product

Konfigurasi environment tidak mengubah cara fitur CRUD Product dibuat. Ia memastikan semua fitur tersebut bekerja pada database yang benar.

| Fitur dari materi sebelumnya | Hubungan dengan environment |
| --- | --- |
| Create dan edit Product | Menyimpan perubahan ke database yang dipilih `.env` |
| Validasi form dan error message | Error detail membantu developer di local, tetapi tidak ditampilkan di production |
| Upload gambar Product | Konfigurasi aplikasi dapat berbeda antar environment tanpa mengubah controller upload |
| Category dan relasi Product | Membaca serta menyimpan relasi pada database aktif |
| Search, pagination, dan sorting | Query berjalan pada koneksi database dari `.env` |
| Soft delete dan status Product | Mengubah data di database aktif, sehingga environment harus benar |
| Dashboard admin | Menghitung data dari database aktif |
| Factory dan seeder materi 16 | Membuat data dummy hanya pada database local atau development yang aman |

Kalimat yang perlu kamu ingat:

> **Source code menjelaskan apa yang aplikasi lakukan. File `.env` menjelaskan tempat dan setting yang dipakai aplikasi saat melakukannya.**

## Penutup

Materi **17. Konfigurasi Environment** selesai.

Sekarang kamu sudah memahami mengapa Laravel menyimpan konfigurasi database seperti `laravel_local` di file `.env`, bukan di controller atau source code aplikasi.

Dengan kebiasaan memeriksa `.env`, menjalankan `php artisan db:show` sebelum perintah database, menjaga `.env` agar tidak masuk GitHub, dan memisahkan database local dari production, kamu dapat mengembangkan aplikasi Laravel dengan lebih aman.
