# Tahap 11 — Membandingkan Konfigurasi Local dan Production Laravel

> Fokus: memahami bahwa satu source code Laravel dapat memakai file `.env` yang berbeda di komputer local dan server production, tanpa melakukan deploy atau mengubah server.

## Melanjutkan dari tahap 10

Pada tahap 10, kita sudah belajar bahwa setiap komputer dan server memiliki file `.env` masing-masing.

- `.env` tidak dibagikan melalui Git atau GitHub.
- `.env.example` hanya menjadi template aman.
- Setiap developer membuat `.env` local miliknya sendiri.
- Server production juga memiliki `.env` sendiri.

Sekarang kita akan membandingkan isi konfigurasi **local** dan **production** agar perbedaan environment menjadi lebih jelas.

## Satu aplikasi, dua tempat yang berbeda

Aplikasi CRUD Product yang dibuat pada materi sebelumnya memiliki source code yang sama:

- model `Product` dan `Category`,
- controller CRUD Product dan Category,
- route,
- Blade,
- migration,
- factory dan seeder dari materi 16.

Namun, aplikasi itu dapat berjalan di dua tempat yang berbeda:

| Tempat | Tujuan | Contoh database |
| --- | --- | --- |
| Komputer local | Belajar, membuat fitur, dan menguji aplikasi | `laravel_local` |
| Server production | Menjalankan aplikasi untuk pengguna sungguhan | `laravel_produk_production` |

Kodenya tidak perlu diubah hanya karena aplikasi berpindah tempat. Yang disesuaikan adalah konfigurasi pada file `.env` di tempat tersebut.

## Analogi: resep sama, dapur berbeda

Bayangkan kamu membuat resep makanan yang sama di dua dapur:

- dapur latihan di rumah,
- dapur restoran yang melayani pelanggan.

Resepnya tetap sama. Namun, alamat dapur, bahan yang tersedia, kunci pintu, dan aturan keamanan dapat berbeda.

Dalam Laravel:

| Analogi | Laravel |
| --- | --- |
| Resep makanan | Source code aplikasi CRUD Product |
| Dapur latihan | Environment `local` |
| Dapur restoran | Environment `production` |
| Catatan alamat, kunci, dan aturan dapur | File `.env` |

Jadi, `.env` bukan mengubah cara CRUD Product bekerja. `.env` memberi tahu Laravel tempat dan aturan yang sesuai untuk menjalankan aplikasi tersebut.

## Contoh konfigurasi local

Berikut contoh bentuk `.env` untuk komputer local:

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

Arti pentingnya:

- `APP_ENV=local` menandakan aplikasi dipakai untuk belajar atau pengembangan;
- `APP_DEBUG=true` membantu developer melihat detail error;
- `DB_HOST=127.0.0.1` biasanya berarti MySQL berjalan di komputer yang sama;
- `DB_DATABASE=laravel_local` menunjuk database latihan;
- username dan password mengikuti setup MySQL local kamu.

Nilai `root` dan password kosong hanya contoh. Setup MySQL local setiap orang bisa berbeda.

## Contoh bentuk konfigurasi production

Server production memakai file `.env` miliknya sendiri. Bentuknya dapat seperti ini:

```env
APP_NAME="CRUD Product"
APP_ENV=production
APP_DEBUG=false

DB_CONNECTION=mysql
DB_HOST=alamat-database-production
DB_PORT=3306
DB_DATABASE=laravel_produk_production
DB_USERNAME=akun-database-production
DB_PASSWORD=password-database-production
```

Contoh di atas menggunakan nilai penjelas, bukan data server asli. Jangan menyalinnya ke `.env` local kamu dan jangan menuliskan password production dalam dokumentasi atau GitHub.

Perbedaan pentingnya:

- `APP_ENV=production` menandakan aplikasi berada di server asli;
- `APP_DEBUG=false` menyembunyikan detail internal error dari pengguna;
- host, database, username, dan password mengikuti server database production.

## Perbandingan local dan production

| Setting | Local | Production | Mengapa dapat berbeda? |
| --- | --- | --- | --- |
| `APP_NAME` | `CRUD Product` | `CRUD Product` | Nama aplikasi boleh tetap sama |
| `APP_ENV` | `local` | `production` | Menandai tempat aplikasi berjalan |
| `APP_DEBUG` | `true` | `false` | Local membantu developer, production melindungi detail internal |
| `DB_CONNECTION` | `mysql` | `mysql` atau sesuai server | Jenis database dapat sama atau berbeda |
| `DB_HOST` | `127.0.0.1` | alamat database production | Local dan server adalah mesin berbeda |
| `DB_PORT` | `3306` | sesuai server | Port mengikuti setup database |
| `DB_DATABASE` | `laravel_local` | `laravel_produk_production` | Database latihan dan database asli dipisahkan |
| `DB_USERNAME` | akun MySQL local | akun database production | Akun akses tiap tempat berbeda |
| `DB_PASSWORD` | password local | password production | Setiap akun memiliki password sendiri |

Yang paling penting: **jangan menyalin file `.env` local ke server production, dan jangan menyalin `.env` production ke komputer local.**

## Mengapa database local dan production harus dipisahkan?

Database local adalah tempat aman untuk belajar dan mencoba fitur. Database production berisi data yang harus dijaga.

Jika keduanya memakai database yang sama, tindakan latihan dapat memengaruhi data asli.

Contohnya dari materi sebelumnya:

| Tindakan | Aman pada database local? | Aman langsung pada database production? |
| --- | --- | --- |
| Mengubah Product contoh | Ya | Tidak selalu, karena bisa mengubah data asli |
| Menjalankan `php artisan db:seed` | Ya, jika memang ingin data dummy | Tidak untuk data dummy tanpa rencana yang jelas |
| Menjalankan `php artisan migrate` | Ya, setelah memahami migration | Perlu perencanaan dan backup yang tepat |
| Menjalankan `php artisan migrate:fresh --seed` | Hanya jika database latihan boleh dihapus | Tidak, karena menghapus seluruh tabel aktif |

Pada materi 16, seeder membuat Category dan Product dummy. Data seperti itu cocok untuk `laravel_local`, bukan untuk database production yang dipakai pengguna.

## `APP_ENV=production` bukan perlindungan otomatis

Ini perlu diingat:

```env
APP_ENV=production
```

bukan tombol ajaib yang otomatis membuat semua tindakan berbahaya berhenti.

Laravel tetap dapat menjalankan banyak perintah jika kamu memiliki akses ke server. Karena itu, developer tetap harus berhati-hati membaca perintah dan memastikan database yang aktif.

Sebelum menjalankan perintah yang berhubungan dengan database pada environment apa pun, biasakan melakukan urutan dari tahap 7:

```text
Periksa .env
    ↓
Pastikan database yang dipilih benar
    ↓
Jalankan php artisan db:show
    ↓
Baru pertimbangkan perintah yang mengubah database
```

Di production, tindakan database sebaiknya dilakukan hanya oleh orang yang bertanggung jawab atas server dan data aplikasi.

## Apa yang dilakukan saat deploy, secara sederhana?

Saat aplikasi Laravel dipasang ke server, biasanya alurnya seperti ini:

```text
Source code aplikasi diambil dari repository
        ↓
File .env production dibuat atau diisi langsung di server
        ↓
Server memakai database production
        ↓
Aplikasi berjalan untuk pengguna
```

File `.env` production dibuat langsung di server atau diberikan melalui sistem pengelolaan secret yang aman. File tersebut tidak ikut berasal dari GitHub.

Untuk materi ini, kamu belum perlu melakukan deploy. Cukup pahami bahwa server membutuhkan konfigurasi production sendiri.

## Hubungan dengan kode CRUD Product

Controller tetap dapat memakai kode yang sama:

```php
$products = Product::query()->latest()->get();
```

Di komputer local, query ini membaca Product dari `laravel_local`.

Di production, query yang sama membaca Product dari database production.

Perbedaannya ditentukan oleh `.env`, bukan dengan membuat dua versi `ProductController`.

Hal yang sama berlaku untuk:

```bash
php artisan db:show
```

Perintah ini akan memeriksa database berdasarkan `.env` tempat perintah dijalankan. Karena itu, jangan menjalankan perintah server dari komputer local atau sebaliknya tanpa memahami environment yang sedang digunakan.

## Checklist tahap 11

- [ ] Saya memahami bahwa source code Laravel sama, tetapi `.env` local dan production berbeda.
- [ ] `.env` local saya memakai `APP_ENV=local` dan `APP_DEBUG=true`.
- [ ] `.env` production harus memakai `APP_ENV=production` dan `APP_DEBUG=false`.
- [ ] Database local dan production memakai nama serta akun yang terpisah.
- [ ] Saya tidak akan menyalin `.env` local ke production atau sebaliknya.
- [ ] Saya memahami data dummy materi 16 hanya untuk database local atau development yang aman.
- [ ] Saya akan menjalankan `php artisan db:show` untuk memeriksa database aktif sebelum perintah yang mengubah data.
- [ ] Saya tidak menjalankan `migrate:fresh --seed` pada database production.

## Ringkasan tahap 11

- Satu aplikasi CRUD Product dapat memakai source code yang sama di local dan production.
- File `.env` menentukan konfigurasi yang berbeda pada setiap tempat.
- Local biasanya memakai `APP_ENV=local` dan `APP_DEBUG=true`.
- Production harus memakai `APP_ENV=production` dan `APP_DEBUG=false`.
- Database local dan production harus terpisah untuk melindungi data asli.
- Seeder data dummy dari materi 16 tidak dijalankan sembarangan di production.
- Jangan pernah menyalin file `.env` antara local dan production.

Tahap berikutnya akan merangkum konfigurasi environment Laravel dan membentuk checklist aman sebelum mengubah `.env`, menjalankan migration, seeder, atau menyiapkan aplikasi untuk production.

---

**Apakah kamu ingin lanjut ke tahap 12: ringkasan dan checklist aman konfigurasi environment Laravel?**
