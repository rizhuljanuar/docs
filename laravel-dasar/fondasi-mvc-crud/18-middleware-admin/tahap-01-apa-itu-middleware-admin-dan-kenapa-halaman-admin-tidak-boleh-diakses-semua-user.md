# Tahap 1 — Apa Itu Middleware Admin dan Kenapa Halaman Admin Tidak Boleh Diakses oleh Semua User?

> Fokus: mengenal halaman admin, perbedaan user biasa dan admin, role user, middleware, serta authorization tanpa langsung menulis kode.

## Bayangkan sebuah toko dengan ruang khusus

Bayangkan aplikasi CRUD Product kamu adalah sebuah toko.

Toko memiliki area yang boleh dimasuki semua pelanggan, misalnya untuk melihat daftar produk. Tetapi toko juga memiliki ruang belakang yang berisi:

- daftar stok produk,
- tombol menambah produk,
- tombol mengubah produk,
- tombol menghapus produk,
- daftar pesanan pelanggan,
- daftar akun pengguna.

Ruang belakang itu tidak boleh dibuka oleh semua orang. Pelanggan boleh berbelanja, tetapi tidak boleh sembarangan mengganti harga, menghapus produk, atau melihat data pesanan orang lain.

Dalam website Laravel, **halaman admin** adalah seperti ruang belakang toko tersebut.

## Apa itu halaman admin?

**Halaman admin** adalah halaman khusus untuk mengelola isi aplikasi.

Pada aplikasi CRUD Product, contoh halaman admin dapat berupa:

- dashboard admin,
- halaman tambah produk,
- halaman edit produk,
- halaman hapus produk,
- halaman daftar order,
- halaman daftar user.

Misalnya alamat halaman-halaman tersebut adalah:

```text
/admin/dashboard
/admin/products
/admin/orders
/admin/users
```

Di halaman itu, admin dapat melakukan pekerjaan penting, seperti menambah Product baru, mengubah nama atau harga Product, menghapus Product, melihat order, dan mengelola user.

Karena pekerjaannya penting, halaman ini harus dijaga.

## Kenapa halaman admin tidak boleh diakses semua user?

Sekarang bayangkan tidak ada pintu atau penjaga di ruang belakang toko.

Siapa pun yang tahu lokasi ruang tersebut dapat masuk. Bahkan pelanggan yang hanya ingin melihat produk dapat:

- menambah produk palsu,
- mengubah harga produk,
- menghapus produk,
- melihat daftar order,
- melihat daftar user.

Hal yang sama dapat terjadi pada aplikasi web jika halaman admin tidak diberi pembatasan akses.

Seorang user biasa mungkin cukup mengetik alamat ini di browser:

```text
/admin/dashboard
```

Jika aplikasi tidak memeriksa siapa yang membuka alamat tersebut, halaman admin bisa langsung tampil. Ini berbahaya, karena mengetik URL tidak membuktikan bahwa seseorang memang berhak mengelola aplikasi.

> Menyembunyikan tombol admin dari menu saja belum cukup. User tetap dapat mencoba membuka URL halaman admin secara langsung.

Jadi, Laravel perlu memeriksa setiap orang yang mencoba masuk ke area admin.

## Perbedaan user biasa dan admin

Pada contoh ini, kita memakai dua jenis pengguna:

| Jenis pengguna | Peran sederhana | Contoh hal yang boleh dilakukan |
| --- | --- | --- |
| `user` | Pelanggan atau pengguna biasa | Melihat halaman umum dan memakai fitur yang memang disediakan untuk user |
| `admin` | Pengelola aplikasi atau toko | Membuka dashboard admin serta mengelola Product, order, dan user |

User biasa dan admin sama-sama bisa memiliki akun dan melakukan login.

Namun, login hanya menjawab pertanyaan:

> “Apakah orang ini sudah masuk ke aplikasi dengan akun yang benar?”

Login belum menjawab pertanyaan:

> “Apakah orang ini boleh mengelola produk, order, dan user?”

Untuk menjawab pertanyaan kedua, aplikasi perlu mengetahui **role** pengguna.

## Apa itu role user?

**Role** adalah label yang menjelaskan peran atau jenis akses seorang user di aplikasi.

Anggap role seperti kartu identitas di toko:

- kartu bertuliskan `user` berarti orang tersebut adalah pelanggan,
- kartu bertuliskan `admin` berarti orang tersebut adalah pengelola toko.

Dalam database, setiap akun user dapat mempunyai nilai role. Contohnya:

```text
Nama: Andi
Role: user

Nama: Siti
Role: admin
```

Dengan informasi itu, aplikasi dapat membedakan:

- Andi boleh memakai fitur untuk user biasa,
- Siti boleh masuk ke halaman admin untuk mengelola aplikasi.

Role bukan nama orang. Role adalah **tugas atau hak akses** orang tersebut di dalam aplikasi.

## Apa itu middleware?

**Middleware** adalah pemeriksa atau penjaga yang berdiri sebelum user masuk ke sebuah halaman.

Kembali ke analogi toko:

```text
User berjalan ke ruang admin
        ↓
Penjaga memeriksa kartu identitas
        ↓
Jika kartunya admin, user boleh masuk
Jika bukan admin, user tidak boleh masuk
```

Di Laravel, alurnya mirip:

```text
User membuka URL halaman admin
        ↓
Middleware memeriksa user
        ↓
Laravel mengizinkan atau menolak akses
```

Middleware bekerja **sebelum** controller dan halaman dijalankan. Artinya, middleware dapat menghentikan user biasa sebelum user melihat dashboard admin atau sebelum user menjalankan tindakan hapus Product.

## Apa itu authorization?

**Authorization** berarti proses memeriksa apakah seseorang **memiliki izin** untuk melakukan suatu tindakan.

Contoh sederhana:

- User sudah login, tetapi role-nya `user`.
- User mencoba membuka halaman tambah produk.
- Laravel perlu memutuskan: apakah role `user` boleh menambah produk?

Jawabannya pada contoh aplikasi ini adalah tidak.

Jadi, authorization bukan sekadar memeriksa apakah user sudah dikenal oleh aplikasi. Authorization memeriksa apakah user tersebut **berhak** mengakses halaman atau melakukan tindakan tertentu.

Perhatikan perbedaan ini:

| Pemeriksaan | Pertanyaan yang dijawab |
| --- | --- |
| Login atau authentication | “Siapa kamu, dan apakah kamu sudah masuk?” |
| Authorization | “Apakah kamu boleh melakukan ini?” |

Pada materi ini, middleware admin akan membantu Laravel melakukan authorization untuk area admin.

## Apa tugas middleware admin?

Middleware admin adalah penjaga khusus untuk halaman yang hanya boleh dipakai admin.

Tugasnya secara sederhana adalah:

1. Memeriksa apakah user sudah login.
2. Jika belum login, arahkan user ke halaman login.
3. Jika sudah login, periksa role user.
4. Jika role-nya bukan `admin`, tolak akses atau arahkan ke halaman yang aman.
5. Jika role-nya `admin`, izinkan user melanjutkan ke halaman admin.

Alurnya dapat dibayangkan seperti ini:

```text
User membuka /admin/dashboard
        ↓
Apakah user sudah login?
        ├── Belum → arahkan ke halaman login
        └── Sudah
              ↓
        Apakah role user adalah admin?
              ├── Bukan → tolak akses atau arahkan ke halaman lain
              └── Ya → izinkan masuk ke dashboard admin
```

Middleware yang sama nantinya dapat dipakai untuk menjaga semua halaman penting berikut:

- dashboard admin,
- halaman tambah produk,
- halaman edit produk,
- halaman hapus produk,
- halaman daftar order,
- halaman daftar user.

Dengan satu penjaga yang dipasang pada area admin, kita tidak perlu menulis pemeriksaan role yang sama berulang-ulang pada setiap controller.

## Yang perlu diingat pada tahap ini

Untuk sekarang, ingat tiga hal berikut:

1. **Halaman admin** adalah area untuk mengelola aplikasi dan data penting.
2. **Role** membedakan user biasa (`user`) dan pengelola aplikasi (`admin`).
3. **Middleware admin** adalah penjaga yang memastikan hanya admin yang dapat melewati pintu menuju halaman admin.

Kita belum membuat kode pada tahap ini. Langkah berikutnya akan menjelaskan bagaimana sebuah request dari browser melewati middleware di Laravel 13+ sebelum sampai ke route, controller, dan halaman.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: memahami cara kerja middleware admin di Laravel?**
