# Tahap 1 — Apa Itu Environment dalam Aplikasi Laravel?

> Fokus: memahami alasan satu aplikasi Laravel dapat memakai setting database yang berbeda di komputer local dan server production.

## Bayangkan kamu memiliki dua tempat untuk bekerja

Bayangkan kamu mempunyai toko yang sama, tetapi ada di dua tempat:

1. **Toko latihan di rumah**. Kamu bebas mencoba susunan rak, mengganti harga contoh, atau membuat kesalahan.
2. **Toko asli yang sudah buka**. Tempat ini dipakai pelanggan sungguhan, jadi semua perubahan harus lebih hati-hati.

Aplikasi Laravel juga sering berjalan di dua tempat seperti itu:

- di komputer developer untuk belajar dan membuat fitur,
- di server agar aplikasi dapat dipakai orang lain.

Walaupun kode aplikasinya sama, tempat aplikasi berjalan bisa berbeda. Perbedaan tempat dan kondisi menjalankan aplikasi inilah yang disebut **environment**.

## Apa itu environment?

Secara sederhana, **environment** adalah kondisi atau tempat aplikasi Laravel sedang dijalankan.

Laravel perlu mengetahui apakah ia sedang berjalan di komputer kamu, server testing, atau server asli. Informasi ini membantu Laravel memilih setting yang tepat.

Contohnya, aplikasi CRUD Product yang sudah kita buat pada materi sebelumnya tetap mempunyai kode yang sama:

- tetap memiliki Product dan Category,
- tetap memakai migration, model, controller, dan Blade,
- tetap dapat memakai factory serta seeder untuk data dummy.

Namun database yang dipakai aplikasi bisa berbeda, tergantung environment-nya.

## Apa bedanya local dan production?

Dua environment yang paling sering kamu temui adalah **local** dan **production**.

| Environment | Arti sederhana | Biasanya dipakai untuk |
| --- | --- | --- |
| `local` | Aplikasi berjalan di komputer sendiri | Belajar, membuat fitur, mencoba CRUD, dan menjalankan data dummy |
| `production` | Aplikasi berjalan di server asli | Aplikasi yang dipakai pengguna sungguhan |

### Local

`local` adalah lingkungan latihan di komputer kamu. Di sini kamu biasanya bebas mencoba hal-hal seperti:

- membuat Product contoh,
- mengubah data,
- menjalankan seeder,
- memperbaiki error,
- bahkan menghapus database latihan lalu membuatnya lagi jika memang diperlukan.

Misalnya, database local kamu bernama:

```text
laravel_local
```

Database ini hanya ada di komputer kamu dan dipakai untuk belajar atau mengembangkan aplikasi.

### Production

`production` adalah lingkungan aplikasi yang benar-benar sudah dipublikasikan di server. Data di sana dapat berisi Product yang dilihat atau dikelola pengguna asli.

Karena itu, production harus lebih hati-hati. Kita tidak boleh menganggap database production sebagai tempat latihan.

Misalnya, server production mungkin memakai database bernama:

```text
laravel_produk_production
```

Nama itu hanya contoh. Server bisa saja memakai nama, username, password, dan alamat database yang berbeda dari komputer kamu.

## Kenapa setting database bisa berbeda?

Komputer local dan server production adalah dua mesin yang berbeda.

Di komputer kamu, kamu mungkin membuat database sendiri dengan nama `laravel_local`. Database tersebut berada di laptop atau komputer kamu.

Di server production, pemilik server biasanya sudah menyiapkan database lain. Database itu dapat memiliki:

- nama database yang berbeda,
- username yang berbeda,
- password yang berbeda,
- alamat server database yang berbeda.

Contohnya:

| Tempat aplikasi berjalan | Nama database |
| --- | --- |
| Komputer local | `laravel_local` |
| Server production | `laravel_produk_production` |

Keduanya tetap dapat menjalankan aplikasi CRUD Product yang sama. Laravel hanya perlu tahu database mana yang harus dihubungi pada tempat tersebut.

> Kode aplikasi menjelaskan **apa yang aplikasi lakukan**. Konfigurasi database menjelaskan **ke mana aplikasi terhubung**.

## Apa masalah jika konfigurasi database ditulis langsung di kode?

Bayangkan kamu menulis alamat toko latihan langsung pada semua brosur toko. Saat toko pindah ke lokasi lain, kamu harus mencari dan mengganti alamat itu di setiap brosur.

Menulis konfigurasi database langsung di source code juga seperti itu.

Misalnya, jika nama database local ditulis langsung di kode:

```php
$databaseName = 'laravel_local';
```

Kode tersebut akan selalu mencoba mencari database `laravel_local`. Ketika kode yang sama dipindahkan ke server production, server mungkin tidak memiliki database dengan nama itu.

Akibatnya, aplikasi bisa gagal terhubung ke database.

Masalah lain yang lebih berbahaya adalah password database. Jika password ditulis langsung di source code, password itu bisa ikut tersimpan saat kode dibagikan ke Git atau GitHub. Orang yang tidak seharusnya tahu bisa melihatnya.

Jadi, hardcode konfigurasi database menyebabkan beberapa masalah:

1. **Sulit dipindahkan**. Kode harus diubah lagi saat berpindah dari local ke production.
2. **Mudah salah**. Developer bisa lupa mengganti nama database atau alamat server.
3. **Berisiko membocorkan rahasia**. Username dan password database dapat ikut terbagi bersama source code.
4. **Kode menjadi tidak fleksibel**. Satu aplikasi tidak mudah dipakai di tempat yang berbeda.

## Apa itu file `.env`?

File **`.env`** adalah tempat Laravel menyimpan setting yang dapat berbeda-beda untuk setiap environment.

Anggap saja `.env` seperti **kertas catatan pribadi di belakang meja kasir**.

- Papan nama dan aturan toko adalah source code aplikasi. Semua cabang toko boleh memakai aturan yang sama.
- Kertas catatan pribadi berisi alamat gudang, nomor kunci, dan informasi khusus cabang itu. Informasi ini bisa berbeda di setiap tempat dan tidak perlu dipajang ke semua orang.

Dalam Laravel, file `.env` dapat menyimpan informasi khusus seperti nama database yang dipakai di komputer tersebut.

Contoh paling sederhana:

```env
APP_ENV=local
DB_DATABASE=laravel_local
```

Artinya:

- `APP_ENV=local` memberi tahu Laravel bahwa aplikasi sedang berjalan di lingkungan local.
- `DB_DATABASE=laravel_local` memberi tahu Laravel nama database yang harus digunakan di komputer kamu.

Di server production, file `.env` milik server bisa berisi nilai yang berbeda:

```env
APP_ENV=production
DB_DATABASE=laravel_produk_production
```

Source code Laravel tetap sama. Yang berubah hanya catatan setting milik tempat aplikasi itu berjalan.

## Hubungannya dengan materi seeder sebelumnya

Pada materi 16, kita menjalankan seeder untuk membuat Category dan Product dummy. Seeder selalu mengisi database yang sedang dipilih oleh konfigurasi aplikasi.

Itulah sebabnya kamu harus memahami environment:

- saat `APP_ENV=local`, data dummy seharusnya masuk ke database latihan, misalnya `laravel_local`;
- database production tidak boleh dianggap sebagai tempat aman untuk mencoba seeder atau data latihan.

Untuk tahap ini, cukup pahami satu hal penting:

> File `.env` membantu satu source code Laravel memakai setting yang aman dan sesuai, baik di komputer local maupun di server production.

Kita belum membahas semua isi `.env` atau cara mengubah konfigurasi. Itu akan dilakukan bertahap pada materi berikutnya.

---

**Apakah kamu ingin lanjut ke tahap 2: mengenal bagian penting file `.env` untuk konfigurasi aplikasi Laravel?**
