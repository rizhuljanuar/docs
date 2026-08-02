# Tahap 6 — Membuat Akun Admin Latihan dengan Aman

> Fokus: mengubah **satu akun local yang sudah ada** menjadi `admin` secara sengaja, lalu memeriksa hasilnya. Akun lain tetap memakai role `user`.

Pada tahap 5, kita menetapkan aturan penting:

> Akun baru harus dimulai sebagai `user`.

Sekarang kita membutuhkan satu akun untuk menguji middleware admin nanti. Akun tersebut akan diberi role `admin` dengan sengaja.

Kita tidak akan membuat semua user menjadi admin. Kita juga tidak akan menambahkan pilihan admin pada form register. Kita hanya memilih satu akun latihan yang jelas.

## Analogi memberi kunci ruang pengelola

Bayangkan semua orang baru mendapat kartu akses umum.

Lalu pemilik toko memilih Siti sebagai pengelola. Pemilik toko sendiri memberikan kunci ruang pengelola kepada Siti.

```text
Akun Andi → user
Akun Budi → user
Akun Siti → admin, karena sengaja diberi akses
```

Ini berbeda dengan membagikan kunci pengelola kepada setiap orang yang baru datang.

Dalam aplikasi Laravel, mengubah role menjadi `admin` adalah tindakan penting. Karena itu, lakukan hanya pada akun yang kamu kenal dan hanya di database local untuk latihan materi ini.

## Sebelum mengubah data user

Pada materi 17, kita belajar bahwa Laravel memakai database berdasarkan file `.env`.

Sebelum menjalankan perintah yang mengubah role, lakukan pemeriksaan berikut:

1. Buka `.env` dari root project Laravel.
2. Pastikan `APP_ENV=local`.
3. Pastikan `DB_DATABASE` adalah database latihan kamu.
4. Dari root project Laravel, jalankan:

```bash
php artisan db:show
```

Pastikan hasilnya menunjukkan database local yang memang aman untuk latihan.

> Jangan menyalin langkah ini ke production. Pemberian role admin pada aplikasi asli harus mengikuti prosedur keamanan dan tanggung jawab aplikasi tersebut.

## Pilih akun yang akan menjadi admin latihan

Kamu memerlukan satu akun yang sudah ada pada tabel `users`.

Jika project kamu sudah memiliki halaman register, buat akun latihan terlebih dahulu melalui halaman tersebut. Karena aturan tahap 5, akun baru ini akan memiliki role `user`.

Contoh akun latihan, gunakan email milikmu sendiri atau email dummy yang hanya dipakai di database local:

```text
Nama: Siti Admin
Email: siti.admin@example.test
Role awal: user
```

Nama dan email di atas hanya contoh. Jangan memakai email atau password asli milik orang lain.

Sebelum mengubah role, kamu dapat melihat daftar user melalui database tool atau Laravel Tinker.

## Mengenal Laravel Tinker

**Laravel Tinker** adalah tempat interaktif untuk menjalankan kode PHP Laravel dari terminal.

Kita akan menggunakannya hanya untuk memilih satu user latihan dan mengubah role-nya secara jelas.

Dari root project Laravel, jalankan:

```bash
php artisan tinker
```

Jika berhasil, terminal akan masuk ke prompt interaktif. Kamu dapat menjalankan kode PHP di sana.

Untuk keluar dari Tinker setelah selesai, jalankan:

```text
exit
```

## Langkah 1, cari akun latihan berdasarkan email

Di dalam Tinker, cari akun latihan dengan email yang kamu buat. Ganti email contoh dengan email akun local kamu sendiri:

```php
$user = App\Models\User::where('email', 'siti.admin@example.test')->firstOrFail();
```

Penjelasan setiap bagian:

- `App\Models\User` adalah model Laravel untuk tabel `users`.
- `where('email', '...')` mencari user berdasarkan kolom email.
- Ganti `'siti.admin@example.test'` dengan email akun latihan yang benar.
- `firstOrFail()` mengambil user pertama yang cocok.
- Jika email tidak ditemukan, Tinker menampilkan error. Ini lebih aman daripada tanpa sengaja mengubah user yang salah.
- `$user = ...` menyimpan hasil user tersebut dalam variabel bernama `$user`.

Setelah perintah berhasil, periksa dahulu user yang dipilih:

```php
$user->only(['id', 'name', 'email', 'role']);
```

Penjelasan:

- `only([...])` menampilkan hanya data yang kita perlukan untuk diperiksa.
- Kita memeriksa `id`, `name`, `email`, dan `role`.
- Jangan menampilkan atau membagikan password, token, atau data rahasia lain.

Contoh hasil konsep:

```text
[
    "id" => 2,
    "name" => "Siti Admin",
    "email" => "siti.admin@example.test",
    "role" => "user",
]
```

Pastikan nama dan email tersebut memang akun latihan yang ingin kamu jadikan admin.

## Langkah 2, ubah role menjadi `admin`

Jika akun yang tampil sudah benar, jalankan dua baris berikut di Tinker:

```php
$user->role = 'admin';
$user->save();
```

Penjelasan setiap bagian:

- `$user->role = 'admin';` mengubah nilai role pada object user yang sudah dipilih di memori.
- `$user->save();` menyimpan perubahan itu ke tabel `users` di database.

Kita memakai pengisian satu atribut lalu `save()`, bukan mass assignment. Cara ini jelas menunjukkan bahwa hanya role dari satu akun terpilih yang diubah.

Setelah disimpan, periksa lagi:

```php
$user->fresh()->only(['id', 'name', 'email', 'role']);
```

Penjelasan:

- `fresh()` mengambil ulang data user terbaru dari database.
- `only([...])` menampilkan kolom aman yang perlu diperiksa.

Hasil yang diharapkan:

```text
[
    "id" => 2,
    "name" => "Siti Admin",
    "email" => "siti.admin@example.test",
    "role" => "admin",
]
```

Sekarang akun tersebut siap dipakai nanti untuk menguji middleware admin.

## Keluar dari Tinker

Setelah selesai, keluar dari Tinker:

```text
exit
```

Kemudian buka database tool dan lihat tabel `users` jika kamu ingin melakukan pemeriksaan kedua. Pastikan:

| User | Role yang diharapkan |
| --- | --- |
| Akun latihan Siti Admin | `admin` |
| User lain | `user` |

## Alternatif jika belum memiliki proses register

Jika project kamu belum memiliki halaman register, kamu dapat membuat satu akun latihan dari Tinker. Gunakan data dummy, bukan password asli.

Di dalam Tinker:

```php
$user = new App\Models\User();
$user->name = 'Siti Admin';
$user->email = 'siti.admin@example.test';
$user->password = Illuminate\Support\Facades\Hash::make('password-latihan-yang-kuat');
$user->role = 'admin';
$user->save();
```

Penjelasan setiap bagian:

- `new App\Models\User()` membuat object user baru.
- `name` dan `email` mengisi data akun latihan.
- `Hash::make(...)` mengubah password menjadi hash sebelum disimpan. Jangan menyimpan password dalam teks biasa.
- `role = 'admin'` diberikan secara sengaja karena akun ini memang dibuat khusus sebagai admin latihan.
- `save()` menyimpan akun baru ke database.

Setelah itu, periksa akun tersebut dengan:

```php
$user->fresh()->only(['id', 'name', 'email', 'role']);
```

Gunakan alternatif ini hanya jika belum ada akun yang bisa dipromosikan di database local. Jika sudah ada akun latihan `user`, cara utama pada langkah sebelumnya lebih mudah dipahami.

## Mengapa kita tidak memakai `$fillable` untuk langkah ini?

Pada tahap 5, kita sengaja tidak menambahkan `role` ke `$fillable` karena form register tidak boleh bebas mengubah hak akses.

Pada tahap ini, kita mengisi atribut satu per satu:

```php
$user->role = 'admin';
$user->save();
```

Cara ini tidak membutuhkan penambahan `role` ke `$fillable`. Ini juga membantu membedakan dua situasi:

| Situasi | Cara yang aman |
| --- | --- |
| User mendaftar dari form umum | Role tidak dikirim, database memberi default `user` |
| Developer menyiapkan satu admin local | Pilih akun tertentu, set `role` menjadi `admin`, lalu `save()` |

## Hubungan dengan middleware yang akan dibuat

Setelah tahap ini, database local memiliki dua contoh kondisi nyata:

```text
Andi dengan role user
Siti dengan role admin
```

Nantinya, saat mereka membuka URL yang sama:

```text
/admin/dashboard
```

hasilnya harus berbeda:

```text
Andi, role user
    ↓
Middleware admin menolak akses

Siti, role admin
    ↓
Middleware admin mengizinkan akses
```

Akun latihan ini akan membantu kita membuktikan bahwa middleware benar-benar bekerja, bukan hanya terlihat benar di kode.

## Kesalahan umum yang perlu dihindari

### 1. Mengubah semua user menjadi admin

Jangan menjalankan query atau perubahan yang menjadikan setiap user admin. Hanya pilih satu akun latihan yang jelas.

### 2. Mengubah user berdasarkan nama saja

Nama bisa sama. Cari user berdasarkan email unik, lalu periksa nama dan emailnya sebelum menyimpan perubahan.

### 3. Menulis password tanpa hash

Jangan membuat akun dengan:

```php
$user->password = 'password-latihan';
```

Gunakan `Hash::make(...)` agar password tersimpan aman dalam bentuk hash.

### 4. Menambahkan pilihan role pada register

Akun admin dibuat oleh developer atau pengelola dengan sengaja, bukan dipilih sendiri oleh user saat mendaftar.

### 5. Mencoba langkah ini pada production

Materi ini adalah latihan untuk database local. Jangan mengubah role user production tanpa prosedur yang jelas dan tanpa memeriksa dampaknya.

## Yang perlu diingat pada tahap ini

1. Akun admin harus dipilih dan dibuat dengan sengaja.
2. Sebelum mengubah data, periksa `.env` dan database aktif dengan `php artisan db:show`.
3. Cari akun berdasarkan email unik, lalu periksa datanya sebelum mengubah role.
4. Ubah satu atribut dengan `$user->role = 'admin'`, lalu simpan dengan `$user->save()`.
5. Pastikan akun lain tetap memiliki role `user`.
6. Akun admin latihan akan dipakai untuk menguji middleware pada tahap berikutnya.

Tahap berikutnya akan membuat file middleware admin yang memeriksa role user sebelum request sampai ke dashboard atau CRUD Product.

---

**Apakah kamu ingin lanjut ke tahap 7: membuat middleware admin di Laravel 13+?**
