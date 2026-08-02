# Tahap 7 — Memeriksa Koneksi Laravel ke Database `laravel_local`

> Fokus: memastikan Laravel dapat terhubung ke database local yang benar sebelum menjalankan migration atau seeder.

## Melanjutkan dari tahap 6

Pada tahap 6, kita sudah mengatur bagian database di file `.env` agar menunjuk ke database local:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_local
DB_USERNAME=root
DB_PASSWORD=
```

Sekarang kita perlu menjawab satu pertanyaan penting:

> Apakah Laravel benar-benar berhasil masuk ke database `laravel_local`?

Jangan langsung menjalankan migration atau seeder. Periksa koneksinya terlebih dahulu.

## Analogi: mencoba kunci sebelum membawa barang

Bayangkan database `laravel_local` adalah ruang penyimpanan latihan.

- File `.env` berisi alamat dan kunci untuk masuk ke ruang tersebut.
- Laravel adalah orang yang akan masuk ke ruang tersebut.
- Migration dan seeder adalah kegiatan membawa atau menyusun barang di dalamnya.

Sebelum membawa barang, kita coba dulu apakah pintunya dapat dibuka dengan kunci yang benar.

Pada Laravel, kita dapat melakukan pemeriksaan ini dengan perintah Artisan yang hanya menampilkan informasi database.

## Pastikan terminal berada di project Laravel yang benar

Buka terminal pada root project Laravel CRUD Product, yaitu folder yang memiliki file berikut:

```text
artisan
.env
composer.json
```

Jangan menjalankan perintah Laravel dari folder dokumentasi materi ini. Folder dokumentasi tidak memiliki file `artisan`, sehingga bukan tempat menjalankan perintah Artisan aplikasi.

## Jalankan `php artisan db:show`

Dari root project Laravel, jalankan:

```bash
php artisan db:show
```

Perintah ini meminta Laravel untuk mencoba terhubung ke database yang sedang dipilih oleh `.env`, lalu menampilkan ringkasan database tersebut.

Jika `.env` berisi:

```env
DB_DATABASE=laravel_local
```

maka hasilnya seharusnya menunjukkan bahwa Laravel sedang membaca database `laravel_local`.

Hasil ringkasan dapat berisi informasi seperti:

- nama database,
- jenis koneksi database,
- daftar atau jumlah tabel yang sudah ada,
- informasi database lain yang tersedia dari koneksi tersebut.

Tampilan tiap komputer dapat berbeda. Fokus utama kamu adalah memastikan tidak muncul error koneksi dan nama database yang terlihat adalah `laravel_local`.

> `php artisan db:show` hanya memeriksa dan menampilkan informasi. Perintah ini tidak membuat migration, tidak menambah Product, dan tidak menjalankan seeder.

## Jika koneksi berhasil

Jika perintah selesai tanpa error dan menunjukkan database `laravel_local`, berarti Laravel sudah dapat menjangkau database latihan yang benar.

Kondisi ini aman untuk langkah berikutnya. Kamu dapat melanjutkan ke materi migration atau seeder sesuai kebutuhan aplikasi CRUD Product.

Namun, pada tahap ini kita cukup berhenti setelah memastikan koneksi berhasil. Jangan menjalankan perintah yang mengubah database hanya karena koneksi sudah berhasil.

## Jika muncul error koneksi

Jika `php artisan db:show` menampilkan error, jangan panik dan jangan langsung mengganti banyak setting sekaligus.

Periksa satu per satu kemungkinan berikut.

### 1. MySQL belum berjalan

Jika MySQL dari Laragon, XAMPP, atau layanan lain belum berjalan, Laravel tidak dapat menemukan server database.

Solusi: nyalakan MySQL, lalu jalankan lagi:

```bash
php artisan db:show
```

### 2. Nama database salah atau belum dibuat

Jika `laravel_local` belum ada, atau namanya berbeda, Laravel tidak dapat membuka database tersebut.

Periksa bagian ini di `.env`:

```env
DB_DATABASE=laravel_local
```

Pastikan nama tersebut sama persis dengan database yang sudah dibuat di MySQL.

### 3. Username atau password salah

Jika akun MySQL tidak sesuai, Laravel tidak diizinkan masuk.

Periksa:

```env
DB_USERNAME=root
DB_PASSWORD=
```

`root` dan password kosong hanyalah contoh. Gunakan akun MySQL local milik kamu sendiri. Jangan menampilkan password asli saat meminta bantuan.

### 4. Host atau port tidak sesuai

Pada setup MySQL local umum, nilai berikut sering digunakan:

```env
DB_HOST=127.0.0.1
DB_PORT=3306
```

Jika MySQL kamu dijalankan pada port lain, gunakan port yang memang dipakai setup kamu. Jangan mengubah port secara acak.

### 5. Laravel masih memakai konfigurasi lama

Laravel dapat memakai configuration cache. Jika isi `.env` sudah benar tetapi hasil pemeriksaan masih menunjukkan nilai lama, bersihkan cache konfigurasi dari root project Laravel:

```bash
php artisan config:clear
```

Perintah ini tidak mengubah tabel, Product, Category, migration, atau data dummy. Perintah ini hanya menghapus cache konfigurasi agar Laravel membaca konfigurasi terbaru.

Setelah itu, periksa lagi:

```bash
php artisan db:show
```

## Urutan pemeriksaan yang aman

Gunakan urutan ini saat baru mengubah konfigurasi database local:

```text
Periksa .env
    ↓
Pastikan MySQL berjalan
    ↓
Jalankan php artisan db:show
    ↓
Pastikan nama database adalah laravel_local
    ↓
Baru pertimbangkan migration atau seeder
```

Urutan ini mencegah kamu menjalankan `php artisan db:seed` ke database yang salah.

## Hubungan dengan materi CRUD Product dan seeder

Aplikasi yang sudah dibuat sebelumnya memakai database untuk banyak hal:

- daftar Product membaca tabel `products`,
- Category memakai tabel `categories`,
- migration membuat atau mengubah struktur tabel,
- seeder materi 16 menambahkan Category dan Product dummy.

Semua aktivitas itu memakai koneksi dari `.env`.

Jika pemeriksaan menunjukkan `laravel_local`, maka aktivitas tersebut diarahkan ke database latihan. Jika nama database lain yang muncul, berhenti dahulu dan periksa `.env` sebelum menjalankan perintah yang mengubah data.

## Yang belum dilakukan pada tahap ini

Tahap ini sengaja belum menjalankan perintah berikut:

```bash
php artisan migrate
php artisan db:seed
php artisan migrate:fresh --seed
```

Alasannya berbeda-beda:

- `migrate` dapat membuat atau mengubah tabel;
- `db:seed` dapat menambahkan data dummy;
- `migrate:fresh --seed` menghapus seluruh tabel pada database aktif, lalu membuatnya kembali dan menjalankan seeder.

Kita hanya memeriksa koneksi, bukan mengubah isi database.

## Checklist tahap 7

- [ ] Saya menjalankan Artisan dari root project Laravel, bukan folder dokumentasi.
- [ ] Saya memastikan MySQL local sedang berjalan.
- [ ] Saya sudah memeriksa nilai `DB_...` pada `.env`.
- [ ] Saya menjalankan `php artisan db:show`.
- [ ] Perintah selesai tanpa error koneksi.
- [ ] Nama database yang ditampilkan adalah `laravel_local`.
- [ ] Jika Laravel membaca nilai lama, saya menjalankan `php artisan config:clear`, lalu memeriksa ulang.
- [ ] Saya belum menjalankan migration atau seeder sebelum yakin koneksi menuju database local yang benar.

## Ringkasan tahap 7

- `php artisan db:show` membantu memeriksa koneksi database Laravel 13+.
- Perintah tersebut tidak membuat tabel dan tidak menambahkan data.
- Pastikan hasilnya menunjukkan database `laravel_local` sebelum memakai migration atau seeder.
- Jika koneksi gagal, periksa MySQL, nama database, username, password, host, dan port satu per satu.
- Gunakan `php artisan config:clear` hanya jika Laravel masih membaca konfigurasi lama setelah `.env` diubah.
- Jangan menjalankan `migrate:fresh --seed` pada database penting, karena perintah itu menghapus tabel pada database aktif.

Tahap berikutnya akan menjelaskan hubungan file `.env` dengan file konfigurasi Laravel, terutama cara Laravel memakai nilai database tersebut tanpa hardcode di source code.

---

**Apakah kamu ingin lanjut ke tahap 8: memahami hubungan file `.env` dan file konfigurasi Laravel?**
