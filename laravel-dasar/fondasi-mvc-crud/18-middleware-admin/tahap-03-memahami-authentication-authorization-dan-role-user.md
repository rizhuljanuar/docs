# Tahap 3 — Memahami Authentication, Authorization, dan Role User

> Fokus: membedakan proses login dengan pemeriksaan izin, lalu memahami mengapa data role `admin` dan `user` perlu disimpan pada setiap akun.

Pada tahap 2, kita melihat bahwa request halaman admin melewati middleware sebelum sampai ke controller.

Sekarang kita perlu menjawab dua pertanyaan yang berbeda:

1. Siapa orang yang sedang membuka aplikasi?
2. Apakah orang tersebut boleh membuka halaman admin?

Dua pertanyaan ini terdengar mirip, tetapi jawabannya berasal dari dua proses yang berbeda: **authentication** dan **authorization**.

## Analogi kartu masuk gedung dan kartu ruangan khusus

Bayangkan ada sebuah gedung kantor.

Setiap orang yang ingin masuk harus menunjukkan kartu masuk di pintu utama. Petugas memeriksa apakah kartu itu milik orang yang terdaftar.

Setelah berhasil masuk gedung, beberapa orang mungkin ingin masuk ke ruang pengelola toko. Ruangan itu hanya boleh dimasuki manager.

Karena itu, ada pemeriksaan kedua:

```text
Pintu utama gedung
    ↓
Apakah kamu pengguna yang sudah terdaftar?
    ↓
Ya, boleh masuk gedung
    ↓
Pintu ruang pengelola
    ↓
Apakah kamu admin?
    ↓
Ya, boleh masuk ruang admin
```

Dalam Laravel:

- pemeriksaan pintu utama disebut **authentication**,
- pemeriksaan pintu ruang admin disebut **authorization**.

## Apa itu authentication?

**Authentication** adalah proses memastikan identitas user.

Pertanyaannya adalah:

> “Siapa kamu, dan apakah kamu sudah berhasil login?”

Ketika user mengisi email dan password pada halaman login, Laravel memeriksa apakah data tersebut cocok dengan akun di database.

Jika cocok, Laravel menganggap user sudah login. Laravel menyimpan informasi login tersebut di session, sehingga pada request berikutnya aplikasi dapat mengenali user yang sama.

Contoh sederhana:

```text
Andi mengisi email dan password
        ↓
Laravel menemukan akun Andi
        ↓
Password cocok
        ↓
Andi berhasil login
```

Setelah itu, Laravel dapat mengetahui user yang sedang aktif. Dalam kode, user yang sudah login dapat diambil dengan:

```php
auth()->user();
```

Penjelasannya:

- `auth()` meminta layanan authentication Laravel.
- `->user()` mengambil data user yang sedang login.

Misalnya Andi sedang login, hasilnya secara konsep adalah data seperti ini:

```text
id: 1
name: Andi
email: andi@example.com
role: user
```

> `auth()->user()` hanya dapat menghasilkan user ketika login sudah berhasil dan request melewati pemeriksaan login.

## Middleware `auth` menjaga user yang belum login

Laravel menyediakan middleware `auth` untuk memastikan user sudah login sebelum membuka halaman tertentu.

Contoh bentuk route:

```php
Route::get('/profile', function () {
    return view('profile');
})->middleware('auth');
```

Penjelasan setiap bagian:

- `Route::get('/profile', ...)` membuat halaman dengan alamat `/profile`.
- `->middleware('auth')` memasang pemeriksa login pada halaman tersebut.
- Jika user sudah login, request diteruskan ke halaman profile.
- Jika user belum login, Laravel menghentikan request dan mengarahkan browser ke halaman login sesuai konfigurasi aplikasi.

Middleware `auth` menjawab:

```text
Apakah user sudah login?
```

Namun middleware `auth` tidak menjawab:

```text
Apakah user itu admin?
```

Itulah alasan kita masih membutuhkan authorization.

## Apa itu authorization?

**Authorization** adalah proses memastikan user mempunyai izin untuk membuka halaman atau melakukan tindakan tertentu.

Pertanyaannya adalah:

> “Setelah diketahui siapa kamu, apakah kamu boleh melakukan ini?”

Contohnya, Andi dan Siti sama-sama sudah login:

| Nama | Sudah login? | Role | Boleh membuka `/admin/dashboard`? |
| --- | --- | --- | --- |
| Andi | Ya | `user` | Tidak |
| Siti | Ya | `admin` | Ya |

Andi lolos authentication karena ia sudah login. Tetapi Andi tidak lolos authorization untuk dashboard admin karena role-nya bukan `admin`.

Jadi ingat perbedaannya:

| Konsep | Pertanyaan | Contoh hasil |
| --- | --- | --- |
| Authentication | “Siapa kamu, apakah kamu sudah login?” | Andi sudah login |
| Authorization | “Apakah kamu punya izin?” | Andi tidak boleh mengelola Product |

## Mengapa login saja tidak cukup?

Ini adalah kesalahan yang sangat umum saat baru membuat aplikasi.

Misalnya sebuah route hanya memakai middleware `auth`:

```php
Route::get('/admin/dashboard', [AdminDashboardController::class, 'index'])
    ->middleware('auth');
```

Kode itu memang mencegah orang yang belum login membuka dashboard. Tetapi **semua user yang sudah login** tetap dapat masuk, termasuk role `user`.

Alurnya seperti ini:

```text
Andi login sebagai user
        ↓
Andi membuka /admin/dashboard
        ↓
Middleware auth memeriksa: sudah login? Ya.
        ↓
Andi lolos dan dashboard tampil
```

Padahal Andi seharusnya tidak boleh masuk.

Untuk halaman admin, kita memerlukan dua pemeriksaan:

```text
1. Authentication: pastikan user sudah login.
2. Authorization: pastikan role user adalah admin.
```

## Apa hubungan role dengan authorization?

Role adalah data yang dipakai aplikasi untuk mengambil keputusan izin.

Untuk materi ini, kita memakai dua role:

| Nilai role | Arti | Akses pada contoh CRUD Product |
| --- | --- | --- |
| `user` | Pengguna biasa | Halaman dan fitur umum yang memang disediakan untuk user |
| `admin` | Pengelola aplikasi | Dashboard admin, kelola Product, order, dan user |

Kita dapat membayangkan aturan sederhananya seperti ini:

```text
Jika role = admin
    izinkan area admin

Jika role = user
    jangan izinkan area admin
```

Nantinya middleware admin akan membaca nilai role dari user yang sedang login:

```php
auth()->user()->role
```

Penjelasan dari kiri ke kanan:

- `auth()` mengakses layanan login Laravel.
- `->user()` mengambil user yang sedang login.
- `->role` mengambil nilai role milik user tersebut.

Jika Siti sedang login, nilainya dapat menjadi:

```text
admin
```

Jika Andi sedang login, nilainya dapat menjadi:

```text
user
```

Jangan gunakan pemeriksaan berdasarkan nama seperti ini:

```php
if (auth()->user()->name === 'Siti') {
    // Jangan lakukan ini.
}
```

Nama bukan penanda akses yang aman. Nama bisa sama, bisa diubah, dan tidak menjelaskan tugas user. Gunakan data `role`.

## Di mana data role disimpan?

Setiap user disimpan dalam tabel database bernama `users`.

Pada project Laravel baru, tabel `users` biasanya sudah memiliki data dasar seperti:

```text
id
name
email
password
```

Agar aplikasi dapat membedakan admin dan user biasa, kita perlu menambahkan satu data lagi:

```text
role
```

Gambaran tabel `users` nanti menjadi seperti ini:

| id | name | email | role |
| --- | --- | --- | --- |
| 1 | Andi | andi@example.com | `user` |
| 2 | Siti | siti@example.com | `admin` |

Laravel tidak dapat menebak sendiri apakah seseorang admin atau user biasa. Aplikasi harus memiliki data role yang jelas di database.

Pada tahap selanjutnya, kita akan menambahkan kolom `role` ini melalui migration. Kita belum mengubah database pada tahap ini.

## Alur lengkap ketika admin membuka halaman Product

Setelah role tersedia, alur akses halaman admin akan menjadi seperti berikut:

```text
Siti membuka /admin/products
        ↓
Laravel menerima request
        ↓
Middleware auth memeriksa login
        ↓
Siti sudah login
        ↓
Middleware admin membaca role Siti
        ↓
Role Siti adalah admin
        ↓
Laravel menjalankan ProductController
        ↓
Halaman daftar Product admin tampil
```

Untuk Andi, alurnya berbeda:

```text
Andi membuka /admin/products
        ↓
Laravel menerima request
        ↓
Middleware auth memeriksa login
        ↓
Andi sudah login
        ↓
Middleware admin membaca role Andi
        ↓
Role Andi adalah user
        ↓
Laravel menolak akses
        ↓
ProductController tidak dijalankan
```

## Kesalahan yang perlu dihindari

### 1. Menganggap `auth` sudah cukup untuk admin

`auth` hanya memastikan user sudah login. Ia tidak memeriksa role `admin`.

### 2. Menyimpan role hanya di Blade

Menyembunyikan tombol admin dengan `@if` di Blade memang membantu tampilan, tetapi bukan perlindungan utama. User masih bisa mencoba URL admin langsung.

Pemeriksaan utama harus dilakukan oleh middleware pada route.

### 3. Menentukan admin dari nama atau email di controller

Jangan membuat aturan seperti “email tertentu selalu admin” langsung di controller. Simpan status admin sebagai role pada data user, lalu periksa melalui middleware.

### 4. Memeriksa role sebelum memastikan user ada

Kode berikut dapat error jika belum ada user yang login:

```php
auth()->user()->role
```

Karena itu, route admin nanti akan memakai middleware `auth` sebelum middleware `admin`. Dengan urutan ini, middleware admin hanya bekerja setelah Laravel memastikan user sudah login.

## Yang perlu diingat pada tahap ini

1. **Authentication** memeriksa siapa user dan apakah ia sudah login.
2. Middleware `auth` melindungi halaman dari orang yang belum login.
3. **Authorization** memeriksa apakah user yang sudah login mempunyai izin.
4. Role `admin` dan `user` adalah data yang membantu Laravel menentukan izin tersebut.
5. Dashboard admin dan CRUD Product membutuhkan `auth` **dan** middleware admin, bukan `auth` saja.
6. Role perlu disimpan di tabel `users`, bukan ditentukan dari nama, email, atau tampilan Blade.

Tahap berikutnya akan mulai menyiapkan database dengan menambahkan kolom `role` ke tabel `users` melalui migration Laravel.

---

**Apakah kamu ingin lanjut ke tahap 4: menambahkan kolom `role` pada tabel `users` dengan migration?**
