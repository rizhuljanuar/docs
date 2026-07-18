# Tahap 2 — Persiapan Project Laravel dan Database

> **Tujuan tahap ini:** Menyiapkan project Laravel pertama dan database yang akan dipakai untuk menyimpan data produk. **Belum menulis kode CRUD apa pun, belum membuat migration.**

---

## 1. Apa yang Akan Disiapkan Sebelum Membuat CRUD?

Sebelum bisa menyimpan data produk, kita butuh dua hal utama:

| Yang disiapkan   | Analogi                              | Untuk apa                                  |
| ---------------- | ------------------------------------ | ------------------------------------------ |
| **Project Laravel** | Lahan + pondasi rumah                | Tempat semua kode aplikasi kita tinggal     |
| **Database**       | Gudang penyimpanan                   | Tempat menyimpan data produk secara permanen|

Tanpa project Laravel, tidak ada tempat menulis kode.
Tanpa database, data produk tidak punya tempat disimpan dan akan hilang saat aplikasi ditutup.

Jadi urutan langkahnya:

1. Buat project Laravel baru.
2. Jalankan server Laravel untuk memastikan project berjalan.
3. Buat database baru.
4. Hubungkan project Laravel dengan database lewat file `.env`.

---

## 2. Apa Itu Project Laravel?

### Pengertian sederhana

**Project Laravel** adalah **satu folder besar** yang berisi semua file dan
kode yang dibutuhkan aplikasi kamu. Di dalamnya sudah tersusun rapi:

- Folder untuk menyimpan route (alamat URL).
- Folder untuk controller (logika).
- Folder untuk model (data).
- Folder untuk view (tampilan).
- File konfigurasi (seperti `.env`).
- Dan masih banyak lagi.

### Analogi: Lahan Bangunan

Bayangkan kamu ingin membangun sebuah toko (aplikasi).

- **Project Laravel** itu seperti **lahan bangunan yang sudah disiapkan**:
  tanahnya sudah rata, pondasi sudah dicor, saluran air dan listrik sudah
  dipasang. Kamu tinggal membangun dinding, atap, dan interiornya.
- Tanpa project Laravel, kamu harus menyiapkan semuanya sendiri:
  menggali tanah, mencor pondasi, memasang pipa. Lambat dan rawan salah.

Jadi, membuat project Laravel = menyiapkan lahan siap bangun untuk aplikasi kita.

---

## 3. Apa Itu Database?

### Pengertian sederhana

**Database** adalah tempat menyimpan data secara **terstruktur** dan **permanen**.
Data yang disimpan di database tidak hilang walau komputer dimatikan.

### Analogi: Gudang dengan Rak

Bayangkan sebuah **gudang besar** dengan banyak **rak**.

| Gudang (Database)         | Rak di Gudang       | Isi Rak              |
| ------------------------- | ------------------- | -------------------- |
| Seluruh gudang            | Satu rak tertentu   | Produk-produk tertentu |

- Setiap rak punya **nama**: rak beras, rak gula, rak sabun.
- Di rak beras, ada banyak **kotak** (baris data), masing-masing berisi
  informasi satu produk: nama, harga, stok.

Dalam istilah database:

- **Database** = seluruh gudang.
- **Tabel** = rak (misal: tabel `produk` = rak produk).
- **Baris (row)** = satu kotak di rak (satu produk lengkap dengan atributnya).
- **Kolom (column)** = jenis informasi di kotak (nama, harga, stok).

### MySQL / SQLite / PostgreSQL

Itu adalah beberapa jenis database yang populer. Untuk belajar,
kita akan pakai **MySQL** (atau bisa juga SQLite untuk yang paling sederhana).
Laravel mendukung semuanya, tinggal pilih.

---

## 4. Apa Itu File `.env`?

### Pengertian sederhana

File `.env` adalah **file konfigurasi utama** project Laravel. Di sinilah
kita menulis pengaturan sensitif, seperti:

- Nama aplikasi.
- Jenis database yang dipakai.
- Alamat database.
- Username dan password database.

### Analogi: Buku Catatan Rahasia Tukang

Bayangkan kamu seorang tukang yang punya buku catatan kecil isinya:

- Kunci gudang di mana? Gudang No. 7.
- Siapa penjaga gudang? Pak Surya.
- Berapa sandinya? `12345`.

Buku catatan itu **rahasia**, tidak boleh dilihat orang luar. Di Laravel,
buku catatan itu = file `.env`.

### Mengapa file `.env` penting?

- File ini **tidak boleh dibagikan** ke orang lain (apalagi diupload ke GitHub).
- Karena berisi **password** dan informasi sensitif.
- Laravel sudah otomatis mengabaikan file ini saat upload ke git (ada di `.gitignore`).

### Contoh isi file `.env` (bagian database):

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=toko_produk
DB_USERNAME=root
DB_PASSWORD=
```

Artinya:
- `DB_CONNECTION`: jenis database (MySQL).
- `DB_HOST`: alamat server database (127.0.0.1 = komputer sendiri).
- `DB_PORT`: port tempat database mendengarkan (3306 = default MySQL).
- `DB_DATABASE`: nama database yang dipakai (`toko_produk`).
- `DB_USERNAME`: username untuk login (default XAMPP/Laragon: `root`).
- `DB_PASSWORD`: password database (kosong jika belum diset).

> Catatan: Kita akan **mengedit** file `.env` ini setelah membuat database.

---

## 5. Cara Membuat Project Laravel Baru

### Prasyarat: pastikan sudah terpasang

Sebelum membuat project, pastikan komputer kamu sudah ada:

| Alat                | Untuk apa                          | Cek dengan perintah          |
| ------------------- | ---------------------------------- | ----------------------------- |
| **PHP** (minimal 8.1) | Menjalankan kode Laravel          | `php -v`                      |
| **Composer**        | Manajer paket PHP (mengunduh Laravel) | `composer -V`                 |
| **MySQL** (atau phpMyAdmin / Laragon / XAMPP) | Database server       | -                             |

Jika belum ada, install dulu:
- PHP: https://www.php.net/
- Composer: https://getcomposer.org/
- MySQL bisa via XAMPP / Laragon / MySQL Community Server.

### Perintah untuk membuat project Laravel

Buka **terminal** (Command Prompt / PowerShell / Git Bash), lalu ketik:

```bash
composer create-project laravel/laravel toko-produk
```

Artinya:
- `composer` = panggil alat Composer.
- `create-project` = perintah untuk membuat project baru.
- `laravel/laravel` = nama paket resmi Laravel.
- `toko-produk` = nama folder project yang akan dibuat.

Setelah perintah ini selesai, akan muncul folder baru bernama `toko-produk`
yang berisi seluruh file project Laravel.

### Masuk ke folder project

```bash
cd toko-produk
```

Artinya: masuk ke folder `toko-produk`. Semua perintah Laravel berikutnya
harus dijalankan dari dalam folder ini.

---

## 6. Cara Menjalankan Server Laravel

Project Laravel punya **server bawaan** untuk keperluan development.
Kita tidak perlu install Apache/Nginx cukup untuk belajar.

### Perintahnya

Pastikan kamu sudah berada **di dalam folder project** (`toko-produk`), lalu ketik:

```bash
php artisan serve
```

Artinya:
- `php` = jalankan PHP.
- `artisan` = alat bantu bawaan Laravel (seperti Swiss Army Knife).
- `serve` = perintah untuk menjalankan server.

### Hasilnya

Kamu akan melihat output seperti ini:

```
Starting Laravel development server: http://127.0.0.1:8000
```

Artinya: server berjalan di alamat **http://127.0.0.1:8000**.

Buka browser lalu ketik alamat itu. Jika muncul **halaman welcome Laravel**
(logo Laravel yang keren), berarti project berhasil dibuat.

### Cara menghentikan server

Tekan `Ctrl + C` di terminal tempat server berjalan.

> Catatan: Server ini hanya untuk development (belajar). Untuk produksi
> (aplikasi online), kita pakai server sungguhan seperti Apache, Nginx, atau
> layanan hosting seperti Forge / Vercel.

---

## 7. Cara Membuat Database

Ada dua cara umum: **lewat phpMyAdmin** (GUI, mudah) atau **lewat terminal** (cepat).

### Opsi A: Lewat phpMyAdmin (Untuk pemula, direkomendasikan)

1. Jalankan MySQL (via XAMPP / Laragon / MySQL Service).
2. Buka browser, ketik: **http://localhost/phpmyadmin**
3. Klik tombol **New** / **Baru** di sidebar kiri.
4. Isi:
   - **Database name:** `toko_produk`
   - **Collation:** biarkan default (biasanya `utf8mb4_general_ci`).
5. Klik **Create**.

Selesai. Database bernama `toko_produk` sudah ada.

### Opsi B: Lewat Terminal MySQL

Jalankan perintah berikut di terminal:

```bash
mysql -u root -p
```

Artinya: login ke MySQL sebagai user `root`, dengan password (kosongkan jika tidak ada).

Setelah masuk ke prompt MySQL (`mysql>`), ketik:

```sql
CREATE DATABASE toko_produk;
```

Artinya: buat database baru bernama `toko_produk`.

Lalu keluar:

```sql
EXIT;
```

### Penamaan database

- Gunakan huruf kecil semua.
- Gunakan underscore `_` sebagai pemisah (bukan spasi).
- Contoh baik: `toko_produk`, `db_produk`, `crud_produk`.
- Hindari: `Toko Produk`, `TOKO-PRODUK`, `tokoProduk` (kurang konsisten).

---

## 8. Cara Mengatur Koneksi Database di File `.env`

Sekarang database sudah dibuat, tapi Laravel belum tahu harus pakai database mana.
Kita harus **memberitahu Laravel** lewat file `.env`.

### Bukka file `.env`

Buka folder project `toko-produk` dengan editor (misal VS Code).
Cari file bernama `.env` di **root folder project** (paling luar).

### Cari bagian database

Cari baris yang dimulai dengan `DB_`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

### Ubah sesuai data kamu

Ganti seperti berikut (sesuaikan jika berbeda):

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=toko_produk
DB_USERNAME=root
DB_PASSWORD=
```

Yang diubah:
- `DB_DATABASE`: isi dengan nama database yang tadi dibuat, yaitu **`toko_produk`**.
- `DB_USERNAME`: biasanya `root` untuk XAMPP/Laragon default.
- `DB_PASSWORD`: kosongkan jika default (XAMPP/Laragon). Jika kamu memasang password, isi di sini.

### Simpan file

Simpan (Ctrl + S). Jangan lupa.

### Tes koneksi (opsional tapi disarankan)

Pastikan server database (MySQL) menyala, lalu jalankan ulang server Laravel:

```bash
php artisan serve
```

Buka browser ke **http://127.0.0.1:8000**. Jika halaman welcome Laravel tetap
muncul tanpa error, berarti koneksi database sudah benar.

Jika muncul error seperti `SQLSTATE[HY000] [1045] Access denied`, artinya
username atau password di `.env` salah. Periksa kembali.

---

## 9. Ringkasan Alur Tahap Ini

```
1. Install PHP + Composer + MySQL           (sekali seumur hidup)
              |
              v
2. composer create-project laravel/laravel toko-produk
              |
              v
3. cd toko-produk
              |
              v
4. php artisan serve                         (cek halaman welcome)
              |
              v
5. Buat database "toko_produk"                (phpMyAdmin / MySQL CLI)
              |
              v
6. Edit file .env -> DB_DATABASE=toko_produk
              |
              v
7. Jalankan ulang server, pastikan tidak error
```

---

## 10. Checklist Keberhasilan Tahap Ini

Centang (`x`) jika sudah selesai:

- [ ] PHP dan Composer sudah terinstall.
- [ ] MySQL sudah berjalan (XAMPP / Laragon / MySQL Service).
- [ ] Project Laravel bernama `toko-produk` sudah dibuat.
- [ ] Server Laravel bisa dijalankan (`php artisan serve`).
- [ ] Halaman welcome Laravel muncul di browser (http://127.0.0.1:8000).
- [ ] Database `toko_produk` sudah dibuat.
- [ ] File `.env` sudah diubah (`DB_DATABASE=toko_produk`).
- [ ] Tidak ada error koneksi database saat menjalankan server.

Jika semua sudah tercentang, **pondasi sudah siap** untuk mulai mendefinisikan
struktur tabel produk di tahap berikutnya.

---

## 11. Penutup

Selamat! Kamu sudah berhasil:

- Membuat project Laravel pertama.
- Menjalankan server development.
- Membuat database.
- Menghubungkan Laravel dengan database.

Di **tahap berikutnya**, kita akan mulai membahas:

- Apa itu **Migration**.
- Cara membuat struktur tabel produk (nama, harga, stok, deskripsi).
- Cara menjalankan migration.

**Belum menulis kode CRUD.** Kita masih di fase "menyiapkan rak gudang"
sebelum produk bisa masuk.

### Jika sudah paham dan semua checklist tercentang, ketik:

```
lanjut
```

Jangan lanjut sebelum kamu benar-benar siap. Pelan-pelan, asal paham.
