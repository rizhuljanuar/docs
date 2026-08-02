# Tahap 5 — Mengenal Konfigurasi Koneksi Database Laravel

> Fokus: memahami enam setting `DB_...` yang diperlukan Laravel untuk terhubung ke database, tanpa mengubah konfigurasi atau menjalankan perintah database terlebih dahulu.

## Melanjutkan dari tahap 4

Sampai tahap 4, kita sudah memahami:

- `.env` menyimpan setting yang dapat berbeda antara local dan production;
- `APP_ENV` menandai lingkungan aplikasi;
- `APP_DEBUG` mengatur detail error yang ditampilkan;
- aplikasi CRUD Product local kita memakai contoh database `laravel_local`.

Sekarang pertanyaannya: bagaimana Laravel menemukan database `laravel_local` itu?

Laravel tidak cukup hanya mengetahui nama database. Laravel juga perlu mengetahui jenis database, lokasi database, serta akun yang boleh masuk ke database tersebut.

Informasi itu biasanya ditulis dengan enam setting `DB_...` di file `.env`.

## Analogi: Laravel ingin masuk ke gedung database

Bayangkan database adalah sebuah gedung penyimpanan data.

Di dalam gedung itu tersimpan tabel seperti:

- `products`,
- `categories`,
- tabel migration,
- data Category dan Product dummy dari materi 16.

Agar Laravel dapat masuk ke gedung yang benar, Laravel memerlukan enam informasi:

| Informasi yang dibutuhkan | Setting Laravel | Analogi |
| --- | --- | --- |
| Jenis gedung | `DB_CONNECTION` | Jenis sistem pintu yang digunakan |
| Lokasi gedung | `DB_HOST` | Alamat gedung |
| Pintu masuk | `DB_PORT` | Nomor pintu gedung |
| Nama ruang data | `DB_DATABASE` | Nama ruang atau gudang yang dituju |
| Nama petugas | `DB_USERNAME` | Nama akun untuk masuk |
| Kunci petugas | `DB_PASSWORD` | Password akun tersebut |

Jika salah satu informasi penting tidak cocok, Laravel mungkin tidak dapat terhubung ke database.

## Bentuk konfigurasi database di `.env`

Untuk database MySQL yang berjalan di komputer local, bentuknya sering seperti ini:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_local
DB_USERNAME=root
DB_PASSWORD=
```

Contoh ini hanya membantu membaca arti tiap baris. Jangan langsung menyalin seluruhnya ke `.env` kamu, karena username, password, atau setup database local setiap komputer dapat berbeda.

Sekarang mari mengenal setiap barisnya.

## 1. `DB_CONNECTION`: jenis database

```env
DB_CONNECTION=mysql
```

`DB_CONNECTION` memberi tahu Laravel jenis database yang digunakan.

Pada aplikasi CRUD Product dari materi sebelumnya, kita menggunakan **MySQL**, jadi nilainya adalah:

```env
DB_CONNECTION=mysql
```

Laravel mendukung beberapa jenis database, tetapi kita tidak perlu membahas semuanya sekarang. Yang perlu kamu ingat:

> Jika project kamu memakai MySQL, gunakan `DB_CONNECTION=mysql`.

Jangan mengganti nilai ini ke jenis lain hanya karena melihat contoh dari project lain. Jenis koneksi harus sesuai dengan database yang benar-benar kamu pasang dan gunakan.

## 2. `DB_HOST`: alamat database

```env
DB_HOST=127.0.0.1
```

`DB_HOST` adalah alamat tempat database berjalan.

Nilai `127.0.0.1` berarti database berjalan di **komputer yang sama** dengan aplikasi Laravel. Ini umum untuk belajar di local.

Analogi sederhananya:

- `127.0.0.1` berarti gedung database berada di halaman rumah sendiri.
- alamat server lain berarti gedung database berada di tempat lain dan perlu alamat khusus untuk mencapainya.

Pada production, `DB_HOST` dapat berbeda karena database mungkin berada di server yang berbeda. Kita belum perlu mengatur hal itu sekarang.

## 3. `DB_PORT`: nomor pintu database

```env
DB_PORT=3306
```

`DB_PORT` adalah nomor pintu yang dipakai Laravel untuk berkomunikasi dengan database.

Untuk MySQL, `3306` adalah port yang umum dipakai. Jadi, pada setup local MySQL biasa, nilainya sering:

```env
DB_PORT=3306
```

Port bukan nama database. Port juga bukan password. Ia hanya membantu Laravel memilih jalur komunikasi yang tepat menuju layanan MySQL.

Jika MySQL kamu memakai port lain, gunakan port yang sesuai setup kamu. Jangan mengubahnya tanpa alasan, karena Laravel bisa gagal menemukan MySQL.

## 4. `DB_DATABASE`: nama database

```env
DB_DATABASE=laravel_local
```

`DB_DATABASE` adalah nama database yang akan dipakai oleh aplikasi Laravel.

Pada contoh materi ini, kita memakai:

```env
DB_DATABASE=laravel_local
```

Artinya, model Product dan Category, migration, query, serta seeder akan bekerja pada database bernama `laravel_local`.

Ingat hubungan dengan materi 16:

```bash
php artisan db:seed
```

Perintah tersebut akan menambahkan data dummy ke database yang ditunjuk oleh `DB_DATABASE`. Jika nilainya `laravel_local`, data dummy masuk ke `laravel_local`.

Karena itu, selalu periksa nama ini sebelum menjalankan migration atau seeder.

## 5. `DB_USERNAME`: nama akun database

```env
DB_USERNAME=root
```

`DB_USERNAME` adalah nama akun yang digunakan Laravel untuk masuk ke MySQL.

Pada beberapa setup local, akun bawaan MySQL adalah `root`. Namun, itu tidak selalu sama di semua komputer.

Contoh:

```env
DB_USERNAME=root
```

atau akun lain yang memang dibuat untuk project kamu:

```env
DB_USERNAME=crud_product_user
```

Gunakan nama akun yang benar-benar tersedia di MySQL kamu. Laravel tidak dapat masuk ke database jika akun ini salah, walaupun nama database sudah benar.

## 6. `DB_PASSWORD`: password akun database

```env
DB_PASSWORD=
```

`DB_PASSWORD` adalah password untuk akun yang ditulis pada `DB_USERNAME`.

Pada sebagian setup local, akun `root` belum memiliki password sehingga nilainya dapat kosong. Pada setup lain, password wajib diisi.

Contoh nilai kosong:

```env
DB_PASSWORD=
```

Contoh nilai terisi, tanpa memakai password asli:

```env
DB_PASSWORD=password_database_kamu
```

Jangan menuliskan password asli di dokumentasi, chat, screenshot publik, source code, atau GitHub.

> Password adalah rahasia akun database. File `.env` membantu menyimpannya terpisah dari source code aplikasi.

## Enam setting bekerja sebagai satu paket

Keenam setting berikut saling melengkapi:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_local
DB_USERNAME=root
DB_PASSWORD=
```

Laravel memakai semuanya saat mencoba membuka koneksi database.

Misalnya:

- `DB_DATABASE` benar, tetapi `DB_PASSWORD` salah, koneksi tetap gagal.
- `DB_USERNAME` benar, tetapi `DB_HOST` salah, koneksi tetap gagal.
- `DB_HOST` dan `DB_PORT` benar, tetapi `DB_DATABASE` salah, Laravel dapat menemukan MySQL tetapi tidak menemukan database yang diminta.

Jadi, jangan hanya memeriksa satu baris ketika koneksi database bermasalah.

## Local dan production dapat memakai nilai yang berbeda

Kita dapat membandingkan bentuk settingnya seperti ini:

| Setting | Contoh local | Contoh production |
| --- | --- | --- |
| `DB_CONNECTION` | `mysql` | `mysql` |
| `DB_HOST` | `127.0.0.1` | alamat database server |
| `DB_PORT` | `3306` | sesuai server |
| `DB_DATABASE` | `laravel_local` | `laravel_produk_production` |
| `DB_USERNAME` | akun MySQL local | akun database production |
| `DB_PASSWORD` | password akun local | password akun production |

Source code Product, Category, controller, Blade, migration, factory, dan seeder tetap dapat sama. File `.env` di masing-masing tempat yang memberi Laravel informasi koneksi yang sesuai.

## Belum saatnya mengubah atau menguji koneksi

Pada tahap ini, tujuan kita hanya mengenal arti setiap setting database.

Jangan mengubah `.env` atau menjalankan perintah seperti ini dulu:

```bash
php artisan migrate
php artisan db:seed
```

Tahap berikutnya akan memandu cara menyesuaikan konfigurasi `.env` untuk database local `laravel_local` dengan lebih aman.

## Ringkasan tahap 5

- Laravel membutuhkan enam setting `DB_...` untuk terhubung ke database.
- `DB_CONNECTION` menentukan jenis database, untuk materi ini: `mysql`.
- `DB_HOST` adalah alamat database, sering `127.0.0.1` di local.
- `DB_PORT` adalah nomor pintu database, MySQL umumnya memakai `3306`.
- `DB_DATABASE` adalah nama database aktif, pada contoh ini: `laravel_local`.
- `DB_USERNAME` dan `DB_PASSWORD` adalah akun untuk masuk ke database.
- Semua setting harus sesuai, bukan hanya nama database.
- Sebelum menjalankan migration atau seeder materi sebelumnya, pastikan keenam nilai menunjuk ke database local yang benar.

Tahap berikutnya akan mempraktikkan cara mengatur setting database local dengan hati-hati, tanpa menyentuh konfigurasi production.

---

**Apakah kamu ingin lanjut ke tahap 6: mengatur database local `laravel_local` di file `.env`?**
