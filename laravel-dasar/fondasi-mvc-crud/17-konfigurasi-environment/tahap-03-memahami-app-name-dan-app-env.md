# Tahap 3 — Memahami `APP_NAME` dan `APP_ENV` di Laravel

> Fokus: memahami nama aplikasi dan penanda environment sebelum membahas setting error atau koneksi database secara lebih rinci.

## Melanjutkan dari tahap 2

Pada tahap 2, kita sudah mengenal bahwa file `.env` berisi pasangan `NAMA_SETTING=nilai`.

Dua baris yang sering kamu temui adalah:

```env
APP_NAME=Laravel
APP_ENV=local
```

Keduanya sama-sama diawali `APP_`, tetapi tugasnya berbeda:

- `APP_NAME` menjelaskan **nama aplikasi**;
- `APP_ENV` menjelaskan **lingkungan tempat aplikasi sedang berjalan**.

Kita akan membahas dua baris ini saja pada tahap ini.

## Apa itu `APP_NAME`?

`APP_NAME` adalah nama untuk aplikasi Laravel kamu.

Misalnya, aplikasi yang kita bangun dari materi CRUD Product dapat diberi nama seperti ini:

```env
APP_NAME="CRUD Product"
```

Tanda kutip dipakai karena nilai tersebut memiliki spasi. Jika nama hanya satu kata, tanda kutip biasanya tidak diperlukan.

```env
APP_NAME=Laravel
```

Analogi sederhananya, `APP_NAME` seperti **papan nama toko**.

- Papan nama membantu orang mengenali nama toko.
- `APP_NAME` membantu Laravel mengetahui nama aplikasi.

Mengubah `APP_NAME` tidak mengubah tabel Product, Category, controller, model, route, atau data di database. Ini hanya mengubah nilai nama aplikasi yang dapat dipakai oleh konfigurasi atau tampilan tertentu.

## Contoh perubahan nama aplikasi

Jika file `.env` masih memiliki nilai bawaan:

```env
APP_NAME=Laravel
```

Kamu dapat menggantinya menjadi:

```env
APP_NAME="CRUD Product"
```

Simpan file setelah mengubahnya.

Nama aplikasi ini dapat digunakan Laravel melalui konfigurasi aplikasi. Misalnya, di Blade kamu dapat menampilkan nama aplikasi dengan:

```blade
{{ config('app.name') }}
```

Jika `APP_NAME` bernilai `"CRUD Product"`, hasil yang dapat tampil adalah:

```text
CRUD Product
```

Pada materi sebelumnya, layout Blade reusable mungkin sudah memiliki judul atau identitas aplikasi sendiri. Kamu tidak perlu mengubah layout tersebut sekarang. Contoh di atas hanya menunjukkan bahwa `APP_NAME` dapat dipakai di source code tanpa menulis nama aplikasi berulang-ulang.

## Apa itu `APP_ENV`?

`APP_ENV` adalah penanda environment aplikasi.

Misalnya:

```env
APP_ENV=local
```

Nilai `local` memberi tahu Laravel bahwa aplikasi sedang berjalan di komputer local untuk belajar atau pengembangan.

Saat aplikasi dipasang pada server asli, file `.env` di server itu dapat memakai nilai:

```env
APP_ENV=production
```

Analogi sederhananya, `APP_ENV` seperti tanda pada pintu toko:

| Tanda di pintu | Keadaan toko | Nilai Laravel |
| --- | --- | --- |
| **Area latihan** | Tempat mencoba dan memperbaiki | `local` |
| **Toko sedang buka** | Tempat melayani pelanggan | `production` |

Tanda ini mengingatkan bahwa perlakuan terhadap aplikasi harus berbeda. Di local, kamu dapat belajar dan mencoba fitur. Di production, kamu harus menjaga data dan keamanan pengguna.

## `APP_NAME` dan `APP_ENV` tidak memiliki tugas yang sama

| Setting | Menjawab pertanyaan | Contoh nilai |
| --- | --- | --- |
| `APP_NAME` | “Apa nama aplikasi ini?” | `CRUD Product` |
| `APP_ENV` | “Aplikasi ini sedang berjalan di lingkungan apa?” | `local` atau `production` |

Jadi, mengganti `APP_NAME` menjadi `CRUD Product` tidak membuat aplikasi berubah menjadi local atau production. Sebaliknya, mengganti `APP_ENV` tidak mengganti nama aplikasi.

## Hubungan `APP_ENV` dengan database local

Pada tahap 1, kita memakai contoh database local bernama `laravel_local`.

Jika aplikasi berjalan di komputer kamu, isi `.env` dapat terlihat seperti ini:

```env
APP_NAME="CRUD Product"
APP_ENV=local
DB_DATABASE=laravel_local
```

Artinya:

- aplikasi ini bernama `CRUD Product`;
- aplikasi sedang dipakai di lingkungan local;
- Laravel akan memakai database local bernama `laravel_local`.

Di server production, source code CRUD Product tetap sama, tetapi `.env` milik server dapat berbeda:

```env
APP_NAME="CRUD Product"
APP_ENV=production
DB_DATABASE=laravel_produk_production
```

Perhatikan bahwa perubahan `APP_ENV` **tidak otomatis mengganti database**. Laravel tetap memakai nilai `DB_DATABASE` yang tertulis pada file `.env` tersebut.

Karena itu, sebelum menjalankan perintah seperti berikut:

```bash
php artisan db:seed
```

pastikan kamu berada di project yang benar dan database yang terpilih memang database latihan. Dari materi 16, seeder dapat menambahkan Category dan Product dummy ke database aktif.

## Jangan asal mengubah `APP_ENV` menjadi `production`

Mengubah nilai ini di komputer local hanya untuk mencoba-coba tidak diperlukan.

`APP_ENV=production` bukan tombol yang membuat aplikasi siap dipublikasikan. Production membutuhkan setup server, domain, keamanan, database yang benar, serta konfigurasi lain yang akan dibahas saat diperlukan.

Untuk belajar di komputer sendiri, tetap gunakan:

```env
APP_ENV=local
```

Saat aplikasi benar-benar dipasang di server, pengaturan production dilakukan pada file `.env` di server tersebut, bukan dengan mengirim file `.env` local kamu.

## Catatan tentang perubahan `.env`

Pada banyak kondisi local, Laravel akan membaca perubahan `.env` saat request atau perintah Artisan berikutnya dijalankan.

Namun, aplikasi Laravel dapat memakai **configuration cache**. Jika nanti kamu mengubah `.env`, tetapi Laravel masih membaca nilai lama, cache konfigurasi mungkin perlu diperbarui. Kita akan membahas hal itu pada tahap yang lebih sesuai, bukan sekarang.

Untuk tahap ini, cukup lakukan perubahan kecil pada `APP_NAME` bila kamu ingin memberi nama aplikasi CRUD Product. Jangan mengubah `APP_ENV` dari `local` jika kamu masih belajar di komputer sendiri.

## Ringkasan tahap 3

- `APP_NAME` adalah nama aplikasi, seperti papan nama toko.
- `APP_ENV` adalah penanda tempat aplikasi berjalan, seperti tanda area latihan atau toko asli.
- Contoh local: `APP_ENV=local`.
- Contoh production: `APP_ENV=production`.
- `APP_ENV` dan `DB_DATABASE` berbeda, mengubah satu nilai tidak otomatis mengubah nilai lainnya.
- Gunakan `APP_NAME` melalui `config('app.name')` jika nama aplikasi ingin dipakai kembali di Blade atau kode lain.
- Jangan mengubah `APP_ENV` menjadi `production` hanya untuk mencoba-coba.
- Sebelum menjalankan seeder materi 16, periksa kembali environment dan database yang aktif.

Tahap selanjutnya membahas `APP_DEBUG`, yaitu setting yang membantu saat mencari error di local, tetapi harus ditangani dengan aman di production.

---

**Apakah kamu ingin lanjut ke tahap 4: memahami `APP_DEBUG` untuk local dan production?**
