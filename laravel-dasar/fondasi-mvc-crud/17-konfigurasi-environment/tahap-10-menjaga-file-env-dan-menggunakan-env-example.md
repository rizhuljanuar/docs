# Tahap 10 — Menjaga File `.env` dan Menggunakan `.env.example`

> Fokus: memahami mengapa `.env` tidak boleh diunggah ke Git atau GitHub, serta bagaimana `.env.example` membantu orang lain menyiapkan konfigurasi tanpa membagikan rahasia.

## Melanjutkan dari tahap 9

Pada tahap 9, kita belajar bahwa password, nama database production, dan informasi koneksi tidak boleh ditulis langsung di source code.

Sekarang kita perlu menjaga agar informasi tersebut juga tidak ikut terkirim saat source code disimpan di Git atau dibagikan melalui GitHub.

Di project Laravel, dua file ini memiliki tugas yang berbeda:

```text
.env
.env.example
```

Walaupun namanya mirip, isi dan cara memperlakukannya tidak sama.

## Analogi: buku kunci asli dan formulir kosong

Bayangkan sebuah toko memiliki dua dokumen:

1. **Buku kunci asli**, berisi alamat gudang, nomor kunci, dan password. Dokumen ini hanya boleh berada di toko yang bersangkutan.
2. **Formulir kosong**, berisi daftar kolom yang harus diisi jika toko baru ingin memakai sistem yang sama. Formulir ini aman dibagikan karena tidak berisi kunci asli.

Dalam Laravel:

| Analogi | File Laravel | Boleh dibagikan? |
| --- | --- | --- |
| Buku kunci asli | `.env` | Tidak |
| Formulir konfigurasi kosong | `.env.example` | Ya |

## Mengapa `.env` tidak boleh masuk ke Git atau GitHub?

File `.env` berisi setting khusus untuk satu komputer atau server. Isinya dapat mencakup:

- username database,
- password database,
- alamat database production,
- `APP_KEY` aplikasi,
- API key, token, atau password layanan lain.

Contoh bentuk `.env` local:

```env
APP_ENV=local
DB_DATABASE=laravel_local
DB_USERNAME=root
DB_PASSWORD=
```

Contoh tersebut tidak memperlihatkan password asli, tetapi file `.env` milik kamu dapat berisi nilai rahasia yang berbeda.

Jika `.env` diunggah ke GitHub, orang lain dapat melihat atau menyalin informasi sensitif tersebut. Risiko menjadi lebih besar pada `.env` production karena database dan layanan asli dapat terkena dampaknya.

> Anggap `.env` seperti kunci rumah. Kunci rumah tidak ditempelkan di papan pengumuman, walaupun alamat rumahnya mungkin sudah diketahui.

## Peran `.gitignore`

File `.gitignore` memberi tahu Git file atau folder apa yang tidak perlu dilacak dan tidak boleh ikut di-commit.

Pada project Laravel baru, `.gitignore` biasanya sudah memiliki aturan untuk mengabaikan file `.env`.

Bentuknya kurang lebih seperti ini:

```gitignore
.env
```

Artinya, saat kamu menjalankan perintah Git untuk menambahkan file, Git akan mengabaikan `.env`.

Namun, `.gitignore` bukan alasan untuk lengah. Kamu tetap harus memeriksa file yang akan di-commit agar tidak ada password, token, atau file rahasia lain yang ikut masuk.

## Cara memeriksa sebelum commit

Sebelum membuat commit, jalankan perintah berikut dari root repository:

```bash
git status --short
```

Perhatikan daftar file yang muncul.

Kondisi yang aman untuk file environment:

- `.env` **tidak muncul** sebagai file baru atau file yang diubah untuk di-commit;
- `.env.example` boleh muncul jika kamu memang mengubah template konfigurasi;
- file source code tidak berisi password atau token asli.

Jika `.env` muncul di `git status`, berhenti dahulu. Jangan menjalankan `git add .` atau commit sebelum memastikan penyebabnya.

Kemungkinan penyebabnya:

- aturan `.env` belum ada di `.gitignore`,
- file `.env` pernah dipaksa masuk ke Git pada masa lalu,
- kamu berada di repository atau folder yang bukan project Laravel yang dimaksud.

Jangan mengunggah `.env` hanya agar aplikasi “cepat jalan” di komputer lain. Gunakan `.env.example` sebagai panduan, lalu buat `.env` baru di komputer atau server tujuan.

## Apa itu `.env.example`?

`.env.example` adalah **template** atau contoh bentuk file `.env`.

File ini menunjukkan setting apa saja yang perlu disiapkan, tetapi tidak boleh memuat nilai rahasia asli.

Contoh sederhana `.env.example`:

```env
APP_NAME="CRUD Product"
APP_ENV=local
APP_DEBUG=true
APP_KEY=

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=
```

Perhatikan bagian yang sengaja kosong:

```env
APP_KEY=
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=
```

Orang yang menerima source code harus mengisi nilai sesuai komputer atau server mereka sendiri.

## Membuat `.env` dari `.env.example`

Saat seseorang baru mengunduh atau melakukan clone project Laravel, biasanya file `.env` belum ada karena memang tidak dibagikan.

Orang tersebut dapat membuat file `.env` baru dari template dengan perintah ini, dijalankan di root project Laravel:

```bash
cp .env.example .env
```

Setelah itu, file `.env` baru perlu disesuaikan, misalnya untuk database local:

```env
APP_ENV=local
APP_DEBUG=true
DB_DATABASE=laravel_local
```

Username dan password database juga harus diisi sesuai setup MySQL local orang tersebut.

Laravel juga membutuhkan `APP_KEY` yang valid. Jika project baru belum memilikinya, buat key untuk file `.env` lokal dengan:

```bash
php artisan key:generate
```

Perintah tersebut mengisi `APP_KEY` pada `.env` local. Key ini bersifat rahasia dan tidak boleh disalin ke `.env.example` atau dibagikan ke GitHub.

> Jangan jalankan `php artisan key:generate` pada aplikasi production yang sudah berjalan tanpa memahami dampaknya. Mengganti `APP_KEY` dapat membuat data yang sudah dienkripsi dengan key lama tidak dapat dibaca.

## Apa yang boleh dan tidak boleh ditulis di `.env.example`?

Gunakan aturan sederhana berikut.

| Jenis nilai | `.env` | `.env.example` |
| --- | --- | --- |
| Nama aplikasi umum | Boleh | Boleh |
| `APP_ENV` contoh | Boleh | Boleh, biasanya `local` |
| `APP_DEBUG` contoh | Boleh | Boleh, biasanya `true` untuk template local |
| `APP_KEY` asli | Boleh, khusus environment itu | Tidak boleh |
| Nama database local pribadi | Boleh | Sebaiknya kosong atau nama contoh non-rahasia |
| Username database | Boleh | Sebaiknya kosong atau nilai contoh yang aman |
| Password database asli | Boleh, khusus environment itu | Tidak boleh |
| API key atau token asli | Boleh, khusus environment itu | Tidak boleh |

`.env.example` bukan cadangan rahasia. Ia adalah petunjuk untuk membuat konfigurasi baru.

## Hubungan dengan aplikasi CRUD Product

Aplikasi CRUD Product dari materi sebelumnya tetap memakai model, controller, route, Blade, migration, factory, dan seeder yang sama di setiap komputer.

Contoh alurnya saat teman kamu ingin menjalankan project yang sama:

```text
Clone source code Laravel
        ↓
Salin .env.example menjadi .env
        ↓
Isi database local miliknya sendiri
        ↓
Buat APP_KEY miliknya sendiri
        ↓
Jalankan php artisan db:show untuk memeriksa koneksi
        ↓
Baru menjalankan migration atau seeder bila diperlukan
```

Dengan cara ini, kamu dapat membagikan code CRUD Product tanpa membagikan database `laravel_local`, password MySQL, atau setting pribadi kamu.

Seeder dari materi 16 juga tetap aman karena setiap orang harus memeriksa `.env` mereka sendiri sebelum menjalankan:

```bash
php artisan db:seed
```

## Jika `.env` sudah terlanjur masuk GitHub

Jika file `.env` atau secret lain terlanjur di-commit atau diunggah, lakukan tindakan berikut:

1. **Anggap semua rahasia di dalamnya sudah tidak aman.**
2. **Segera ganti password database, API key, token, dan `APP_KEY` yang terpapar sesuai kebutuhan.**
3. **Hapus file atau secret dari source code yang sedang aktif.**
4. **Pastikan `.env` ada dalam `.gitignore`.**
5. **Buat atau perbarui `.env.example` tanpa nilai rahasia.**
6. **Minta bantuan orang yang memahami pengelolaan repository untuk membersihkan riwayat Git jika diperlukan.**

Menghapus file dari commit terbaru tidak selalu menghapusnya dari seluruh riwayat Git. Karena itu, mengganti secret tetap langkah yang penting.

## Checklist tahap 10

- [ ] File `.env` saya tidak diunggah ke Git atau GitHub.
- [ ] `.env` tercantum di `.gitignore` project Laravel.
- [ ] Saya memeriksa `git status --short` sebelum commit.
- [ ] `.env.example` hanya berisi bentuk setting, bukan password, token, atau `APP_KEY` asli.
- [ ] Saya tahu cara membuat `.env` baru dengan `cp .env.example .env`.
- [ ] Saya tahu `php artisan key:generate` membuat `APP_KEY` untuk `.env` local.
- [ ] Saya memeriksa koneksi dengan `php artisan db:show` sebelum menjalankan migration atau seeder.
- [ ] Jika secret pernah terunggah, saya menggantinya, bukan hanya menghapus file dari commit terbaru.

## Ringkasan tahap 10

- `.env` adalah file rahasia khusus untuk satu environment, jadi tidak boleh masuk Git atau GitHub.
- `.gitignore` membantu Git mengabaikan `.env`, tetapi tetap periksa status sebelum commit.
- `.env.example` adalah template aman yang boleh dibagikan.
- Gunakan `cp .env.example .env` untuk membuat konfigurasi local baru dari template.
- Gunakan `php artisan key:generate` untuk membuat `APP_KEY` di `.env` local baru.
- Jangan menaruh password, token, atau `APP_KEY` asli di `.env.example`.
- Setiap developer dan server memiliki `.env` masing-masing, sementara source code CRUD Product tetap sama.

Tahap berikutnya akan membandingkan konfigurasi local dan production secara lebih lengkap, agar kamu memahami nilai mana yang dapat berbeda saat aplikasi benar-benar dipasang di server.

---

**Apakah kamu ingin lanjut ke tahap 11: membandingkan konfigurasi local dan production Laravel?**
