# Tahap 4 — Memahami `APP_DEBUG` untuk Local dan Production

> Fokus: memahami mengapa detail error membantu saat belajar di komputer local, tetapi tidak boleh ditampilkan kepada pengguna di production.

## Melanjutkan dari tahap 3

Pada tahap 3, kita sudah mengenal dua setting aplikasi:

```env
APP_NAME="CRUD Product"
APP_ENV=local
```

Sekarang kita akan mengenal satu setting lain yang biasanya berada di dekatnya:

```env
APP_DEBUG=true
```

`APP_DEBUG` berkaitan dengan cara Laravel menampilkan informasi ketika terjadi error.

## Apa itu error?

**Error** adalah masalah yang membuat bagian aplikasi tidak dapat berjalan seperti seharusnya.

Contoh pada aplikasi CRUD Product yang sudah kita bangun:

- Laravel tidak dapat menemukan database `laravel_local`.
- Nama kolom pada query salah.
- Route Product yang dibuka tidak ada.
- Ada kesalahan penulisan kode pada controller atau Blade.

Saat error terjadi, Laravel perlu memberi tahu developer apa yang salah agar dapat diperbaiki.

## Apa tugas `APP_DEBUG`?

`APP_DEBUG` menentukan apakah Laravel menampilkan **informasi error yang sangat rinci**.

Ada dua nilai yang penting:

```env
APP_DEBUG=true
```

atau:

```env
APP_DEBUG=false
```

| Nilai | Arti sederhana |
| --- | --- |
| `true` | Laravel dapat menunjukkan detail error untuk membantu developer mencari masalah |
| `false` | Laravel menyembunyikan detail teknis error dari pengguna |

Analogi sederhananya adalah mekanik dan pelanggan di bengkel.

- Saat mobil diperiksa di bengkel, mekanik perlu membuka mesin dan melihat bagian yang rusak secara lengkap. Ini seperti `APP_DEBUG=true` di local.
- Saat pelanggan datang mengambil mobil, pelanggan tidak perlu melihat kabel, nomor komponen, atau catatan teknis internal. Ini seperti `APP_DEBUG=false` di production.

## Mengapa `APP_DEBUG=true` membantu di local?

Saat belajar atau membuat fitur di komputer sendiri, detail error membantu kamu menemukan sumber masalah.

Misalnya, kamu salah menulis nama database di `.env`:

```env
DB_DATABASE=laravel_lokal
```

Padahal database yang benar bernama:

```env
DB_DATABASE=laravel_local
```

Jika `APP_DEBUG=true`, Laravel biasanya menampilkan halaman error yang lebih lengkap. Kamu dapat melihat pesan bahwa koneksi database gagal dan informasi yang membantu menemukan salah ketik tersebut.

Contoh lain, saat menjalankan seeder dari materi 16:

```bash
php artisan db:seed
```

Jika ada masalah pada koneksi database local atau kode seeder, detail error membantu kamu memahami bagian yang perlu diperiksa.

> `APP_DEBUG=true` membantu membaca masalah, tetapi tidak memperbaiki masalah secara otomatis. Kamu tetap perlu memeriksa `.env`, kode, migration, atau seeder yang terkait.

## Mengapa `APP_DEBUG=true` berbahaya di production?

Di server production, aplikasi dipakai pengguna sungguhan. Jika detail error ditampilkan kepada siapa saja, informasi internal aplikasi dapat terlihat.

Contoh informasi yang bisa muncul pada halaman error rinci:

- lokasi file dan baris kode yang bermasalah,
- nama class, controller, atau route,
- jenis database yang dipakai,
- struktur query atau informasi teknis lain.

Informasi tersebut berguna untuk developer, tetapi tidak diperlukan pengguna. Lebih aman jika detail hanya dilihat melalui log server oleh orang yang berwenang.

Karena itu, production harus memakai:

```env
APP_DEBUG=false
```

Dengan nilai ini, pengguna akan melihat pesan error yang lebih umum. Detail teknis tetap dapat diperiksa developer melalui log Laravel, bukan dari layar pengguna.

## Kombinasi `APP_ENV` dan `APP_DEBUG`

Kedua setting ini saling berkaitan, tetapi tugasnya tidak sama.

| Situasi | `APP_ENV` | `APP_DEBUG` | Alasan |
| --- | --- | --- | --- |
| Belajar dan mengembangkan aplikasi di komputer sendiri | `local` | `true` | Detail error membantu memperbaiki aplikasi |
| Aplikasi dipakai pengguna di server asli | `production` | `false` | Detail internal harus disembunyikan dari pengguna |

Contoh `.env` untuk komputer local:

```env
APP_NAME="CRUD Product"
APP_ENV=local
APP_DEBUG=true
DB_DATABASE=laravel_local
```

Contoh `.env` untuk server production:

```env
APP_NAME="CRUD Product"
APP_ENV=production
APP_DEBUG=false
DB_DATABASE=laravel_produk_production
```

Contoh production di atas hanya untuk memahami perbedaan setting. Jangan mengganti `.env` local kamu dengan contoh production tersebut.

## `APP_DEBUG=false` bukan berarti error hilang

Ini bagian yang penting.

Jika `APP_DEBUG=false`, error tetap bisa terjadi. Laravel hanya tidak menampilkan detail teknisnya kepada pengguna.

Analogi sederhananya, menutup pintu ruang mekanik tidak membuat kerusakan mobil hilang. Kita hanya tidak memperlihatkan bagian dalam bengkel kepada pelanggan.

Jadi:

- `APP_DEBUG=true` bukan berarti aplikasi pasti error;
- `APP_DEBUG=false` bukan berarti aplikasi bebas error;
- setting ini mengatur seberapa banyak detail error yang ditampilkan.

## Jangan menampilkan nilai `APP_DEBUG` di halaman aplikasi

Kamu tidak perlu menampilkan `APP_DEBUG` di layout Blade, halaman daftar Product, atau dashboard admin.

Setting ini adalah pengaturan internal. Berbeda dengan `APP_NAME` yang kadang wajar dipakai pada judul halaman melalui `config('app.name')`, `APP_DEBUG` tidak ditujukan untuk ditampilkan kepada pengguna.

Jika suatu saat kamu perlu mengetahui apakah aplikasi berada di mode debug saat membuat kode internal, Laravel menyediakan konfigurasi aplikasi. Namun, untuk materi ini kamu tidak perlu menambah kode apa pun ke controller, model, route, Blade, migration, factory, atau seeder.

## Apa yang perlu dilakukan saat belajar di local?

Untuk aplikasi Laravel di komputer sendiri, periksa bahwa `.env` memiliki nilai berikut:

```env
APP_ENV=local
APP_DEBUG=true
```

Jika ada error, lakukan langkah sederhana ini:

1. Baca bagian utama pesan error, jangan langsung panik.
2. Periksa file atau fitur yang baru saja kamu ubah.
3. Periksa kembali nama database dan setting `.env` jika error berhubungan dengan database.
4. Jika error muncul saat menjalankan seeder, pastikan database yang aktif benar-benar database local.
5. Perbaiki penyebab error, lalu coba lagi.

Jangan mengubah `APP_DEBUG` menjadi `false` hanya untuk menyembunyikan error saat belajar. Error tetap ada dan perlu diperbaiki.

## Ringkasan tahap 4

- `APP_DEBUG` mengatur apakah detail error Laravel ditampilkan.
- Gunakan `APP_DEBUG=true` saat belajar dan mengembangkan aplikasi di local.
- Gunakan `APP_DEBUG=false` di production agar detail internal tidak terlihat pengguna.
- `APP_ENV` menunjukkan lingkungan aplikasi, sedangkan `APP_DEBUG` mengatur tampilan detail error.
- `APP_DEBUG=false` menyembunyikan detail error, bukan menghapus error.
- Jangan menampilkan `APP_DEBUG` pada halaman CRUD Product.
- Sebelum menjalankan migration atau seeder dari materi sebelumnya, pastikan environment dan database yang aktif sudah benar.

Tahap berikutnya akan mulai mengenalkan enam setting `DB_...` yang Laravel gunakan untuk menghubungkan aplikasi ke database.

---

**Apakah kamu ingin lanjut ke tahap 5: mengenal konfigurasi koneksi database Laravel?**
