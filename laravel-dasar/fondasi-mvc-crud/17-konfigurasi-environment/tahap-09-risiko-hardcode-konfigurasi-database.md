# Tahap 9 — Risiko Hardcode Konfigurasi Database di Source Code

> Fokus: memahami mengapa nama database, username, password, dan alamat database tidak boleh ditulis langsung pada controller, model, migration, seeder, atau file source code lain.

## Melanjutkan dari tahap 8

Pada tahap 8, kita sudah melihat alur konfigurasi Laravel:

```text
.env
        ↓
config/database.php
        ↓
Model, Controller, Migration, Seeder, dan Query Laravel
        ↓
Database
```

Dengan alur ini, controller Product tidak perlu tahu apakah ia sedang memakai database `laravel_local` di komputer kamu atau database lain di server production.

Sekarang kita akan membahas alasan penting di balik aturan tersebut: **jangan hardcode konfigurasi database di source code**.

## Apa arti hardcode?

**Hardcode** berarti menulis sebuah nilai tetap langsung di dalam source code, padahal nilai itu seharusnya bisa berbeda pada tiap environment.

Contoh hardcode nama database:

```php
$databaseName = 'laravel_local';
```

Contoh hardcode username dan password:

```php
$username = 'root';
$password = 'password-rahasia';
```

Contoh hardcode alamat database:

```php
$host = '127.0.0.1';
```

Nilai-nilai tersebut mungkin terlihat bekerja di komputer local. Namun, nilai itu menjadi masalah saat aplikasi dipindahkan ke komputer lain atau server production.

## Analogi: alamat gudang ditulis pada setiap pekerjaan

Bayangkan toko memiliki gudang latihan di rumah dan gudang asli di kota lain.

Jika setiap pekerja menulis alamat gudang latihan pada catatannya sendiri, saat toko pindah mereka harus mencari dan mengganti alamat di semua catatan. Ada risiko satu catatan tertinggal dan pekerja pergi ke gudang yang salah.

Cara yang lebih rapi adalah menyimpan alamat di satu buku catatan khusus. Semua pekerja bertanya pada buku itu saat membutuhkan alamat.

Di Laravel:

| Dalam analogi | Dalam Laravel |
| --- | --- |
| Alamat gudang khusus | Nilai pada `.env` |
| Buku petunjuk toko | File konfigurasi dalam `config/` |
| Pekerja toko | Controller, model, migration, seeder, dan query |

Dengan cara ini, saat local memakai `laravel_local` dan production memakai database lain, hanya nilai `.env` di masing-masing tempat yang berbeda.

## Masalah 1: aplikasi sulit dipindahkan

Misalnya kamu menulis di controller Product:

```php
$databaseName = 'laravel_local';
```

Kemudian aplikasi dipasang ke server production yang menggunakan database:

```text
laravel_produk_production
```

Source code tadi masih membawa nama `laravel_local`. Developer harus mencari seluruh file yang mungkin berisi nama itu, lalu mengubahnya satu per satu.

Masalahnya:

- mudah ada file yang terlupa,
- perubahan harus dibuat lagi setiap kali pindah environment,
- code local dan production bisa menjadi berbeda,
- proses deploy menjadi lebih sulit.

Dengan `.env`, source code tetap sama. Cukup gunakan file `.env` yang sesuai di setiap tempat.

```text
Kode Laravel sama
        ↓
.env local: DB_DATABASE=laravel_local
.env production: DB_DATABASE=laravel_produk_production
```

## Masalah 2: password dapat bocor ke Git atau GitHub

Ini adalah risiko yang paling serius.

Bayangkan password database ditulis langsung di controller:

```php
$password = 'password-rahasia';
```

Jika source code di-commit lalu di-push ke GitHub, password itu dapat ikut tersimpan di riwayat Git.

Menghapus baris password dari file kemudian tidak selalu cukup, karena commit lama dapat masih menyimpan password tersebut. Password yang sudah terlanjur tersebar sebaiknya segera diganti oleh pemilik database.

Itulah sebabnya password database harus berada pada `.env` local atau `.env` server yang tidak dibagikan, bukan pada file source code.

> Jangan memakai password asli pada contoh kode, dokumentasi, screenshot, chat publik, commit Git, atau GitHub.

## Masalah 3: berisiko memakai database yang salah

Bayangkan ada kode seperti ini di seeder:

```php
$databaseName = 'laravel_local';
```

Saat kode tersebut dijalankan di server production, hasilnya bisa membingungkan atau gagal. Lebih buruk lagi, jika hardcode menunjuk ke database production pada komputer development, perintah seperti seeder dapat mengubah data yang tidak seharusnya disentuh.

Pada materi 16, kita sudah membuat seeder untuk Category dan Product dummy. Seeder harus mengikuti koneksi Laravel dari `.env`, bukan memilih database sendiri.

Dengan begitu, sebelum menjalankan:

```bash
php artisan db:seed
```

kamu cukup memeriksa `.env` dan memastikan:

```env
APP_ENV=local
DB_DATABASE=laravel_local
```

Lalu periksa koneksi dengan materi tahap 7:

```bash
php artisan db:show
```

## Masalah 4: kode menjadi sulit dirawat

Bayangkan nama database, host, username, dan password ditulis berulang di controller Product, controller Category, migration, dan seeder.

Jika nama database berubah, semua tempat itu harus diperiksa dan diubah. Ini menghasilkan banyak pengulangan dan meningkatkan risiko salah ketik.

Contoh yang tidak perlu dibuat:

```php
// ProductController
$databaseName = 'laravel_local';

// CategorySeeder
$databaseName = 'laravel_local';
```

Tidak ada manfaat dari pengulangan tersebut. Laravel sudah menyediakan satu konfigurasi database untuk seluruh aplikasi.

Dengan `.env` dan `config/database.php`, perubahan cukup dilakukan pada satu tempat yang sesuai untuk environment tersebut.

## Masalah 5: source code tidak aman dibagikan

Source code Laravel seharusnya dapat dibagikan ke anggota tim atau disimpan di Git tanpa membawa rahasia pribadi setiap komputer.

Contoh informasi yang tidak boleh melekat pada source code:

- password database,
- username database production,
- alamat server database production,
- API key atau token layanan lain.

Karena nilai seperti ini berada di `.env`, developer lain dapat membuat `.env` milik mereka sendiri sesuai setup local mereka.

Contohnya:

| Komputer | `DB_DATABASE` |
| --- | --- |
| Komputer kamu | `laravel_local` |
| Komputer teman | `laravel_local_teman` |
| Server production | `laravel_produk_production` |

Controller, model Product, Category, route, Blade, migration, factory, dan seeder dapat tetap memakai source code yang sama.

## Cara yang benar pada aplikasi CRUD Product

Untuk aplikasi yang sudah kita buat, controller cukup fokus pada tugasnya.

Contoh:

```php
$products = Product::query()->latest()->get();
```

Kode tersebut meminta Laravel mengambil Product. Ia tidak perlu mengetahui database mana yang sedang digunakan.

Saat membuat data dummy, seeder juga cukup memakai Eloquent dan factory:

```php
Product::factory()->count(30)->create();
```

Laravel akan memakai koneksi database yang sudah diatur melalui `.env` dan `config/database.php`.

Kamu tidak perlu menambahkan kode database manual ke:

- `ProductController`,
- `CategoryController`,
- model `Product` atau `Category`,
- migration,
- `ProductFactory` atau `CategoryFactory`,
- `ProductSeeder`, `CategorySeeder`, atau `DatabaseSeeder`,
- route dan Blade.

## Jika source code terlanjur berisi password

Jika kamu menemukan password atau secret lain yang sudah terlanjur ditulis di source code:

1. **Jangan menyalin atau membagikannya.**
2. **Pindahkan nilainya ke `.env` yang sesuai.**
3. **Ubah source code agar membaca konfigurasi melalui `config()` atau memakai konfigurasi Laravel yang sudah tersedia.**
4. **Ganti password atau secret tersebut**, terutama jika pernah masuk commit, GitHub, chat publik, atau tempat lain yang dapat diakses pihak lain.
5. **Periksa kembali** bahwa source code tidak lagi berisi rahasia tersebut.

Langkah mengganti password dilakukan melalui pengelola database atau layanan yang membuat password tersebut. Jangan menunggu sampai aplikasi dipublikasikan untuk memperbaiki kebocoran secret.

## Ringkasan: tempat yang benar untuk setiap nilai

| Jenis informasi | Contoh | Tempat yang benar |
| --- | --- | --- |
| Nama database local | `laravel_local` | `.env` local |
| Nama database production | `laravel_produk_production` | `.env` production/server |
| Username database | akun MySQL | `.env` sesuai environment |
| Password database | password akun MySQL | `.env` sesuai environment, tidak dibagikan |
| Pola koneksi database | `env('DB_DATABASE', 'laravel')` | `config/database.php` |
| Query Product | `Product::query()` | controller, service, atau tempat query aplikasi |

## Ringkasan tahap 9

- Hardcode berarti menulis nilai tetap langsung di source code.
- Database local, host, username, dan terutama password tidak boleh di-hardcode.
- Hardcode membuat aplikasi sulit dipindahkan dari local ke production.
- Password yang masuk ke Git atau GitHub dapat tetap ada di riwayat commit walaupun file sudah diperbaiki.
- Seeder dari materi 16 harus mengikuti koneksi `.env`, bukan memilih database secara manual.
- Source code CRUD Product tetap sama di setiap environment, sedangkan `.env` dapat berbeda.
- Jika secret sudah terlanjur dibagikan, pindahkan dari source code dan segera ganti secret tersebut.

Tahap berikutnya akan membahas alasan file `.env` tidak diunggah ke Git atau GitHub, serta fungsi file `.env.example` untuk membagikan bentuk konfigurasi tanpa membagikan rahasia.

---

**Apakah kamu ingin lanjut ke tahap 10: mengapa `.env` tidak boleh diunggah ke GitHub dan apa fungsi `.env.example`?**
