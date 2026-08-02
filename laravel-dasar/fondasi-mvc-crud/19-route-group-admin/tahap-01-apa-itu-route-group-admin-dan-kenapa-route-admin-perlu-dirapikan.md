# Tahap 1 — Apa Itu Route Group Admin dan Kenapa Route Admin Perlu Dirapikan?

> Fokus: memahami route Laravel, route admin, dan alasan halaman admin perlu dikumpulkan dengan struktur yang rapi. Pada tahap ini kita belum mengubah kode.

Pada materi 18, kita sudah membuat **middleware `admin`**. Middleware tersebut bertugas seperti penjaga: user harus sudah login dan mempunyai role `admin` sebelum boleh mengelola halaman penting.

Sekarang ada satu masalah baru.

Halaman admin memang sudah bisa dilindungi, tetapi route-nya dapat menjadi panjang dan berantakan jika ditulis satu per satu. Materi ini akan membantu kita merapikannya dengan **route group admin**.

## Bayangkan area belakang sebuah toko

Aplikasi CRUD Product dapat kita bayangkan sebagai toko.

Di bagian depan toko, pelanggan melihat produk. Di bagian belakang, ada area kerja khusus untuk pengelola toko. Area itu berisi:

- dashboard admin,
- daftar produk,
- halaman tambah produk,
- halaman edit produk,
- tombol hapus produk,
- daftar order,
- daftar user.

Alamat halaman admin dapat terlihat seperti ini:

```text
/admin/dashboard
/admin/products
/admin/products/create
/admin/products/{id}/edit
/admin/orders
/admin/users
```

Semua alamat itu menuju area yang sama, yaitu area admin. Karena mempunyai tujuan dan aturan yang mirip, sebaiknya alamat-alamat tersebut ditata bersama, bukan dicampur tanpa pola.

## Apa itu route di Laravel?

**Route** adalah aturan yang memberi tahu Laravel apa yang harus dilakukan ketika seseorang membuka sebuah URL.

Contoh paling sederhana:

```text
User membuka URL
        ↓
Route mengenali URL tersebut
        ↓
Laravel menjalankan controller yang sesuai
        ↓
Controller menampilkan halaman atau memproses data
```

Misalnya user membuka:

```text
/products
```

Laravel perlu tahu tiga hal:

| Bagian | Pertanyaan sederhana | Contoh |
| --- | --- | --- |
| URL halaman | “Alamat apa yang dibuka?” | `/products` |
| Route | “Aturan mana yang cocok untuk alamat ini?” | route daftar produk |
| Controller | “Siapa yang mengerjakan permintaan ini?” | `ProductController` |

Nantinya route CRUD Product dapat mengarahkan Laravel ke `ProductController`, sedangkan route dashboard admin dapat mengarah ke `AdminDashboardController`.

Jadi, route seperti petunjuk alamat di sebuah gedung. Route membantu Laravel menemukan ruangan atau petugas yang tepat saat user membuka URL tertentu.

## Apa yang dimaksud route admin?

**Route admin** adalah route untuk halaman atau tindakan yang hanya dipakai pengelola aplikasi.

Contoh pada aplikasi kita:

| Halaman atau tindakan | Contoh URL | Controller yang menangani |
| --- | --- | --- |
| Dashboard admin | `/admin/dashboard` | `AdminDashboardController` |
| Daftar produk | `/admin/products` | `ProductController` |
| Tambah produk | `/admin/products/create` | `ProductController` |
| Edit produk | `/admin/products/{id}/edit` | `ProductController` |
| Hapus produk | `/admin/products/{id}` | `ProductController` |
| Daftar order | `/admin/orders` | `OrderController` |
| Daftar user | `/admin/users` | `UserController` |

Khusus untuk hapus produk, URL tersebut biasanya menerima request `DELETE`. Jadi URL yang sama dapat dipakai untuk data produk tertentu, tetapi Laravel membedakannya melalui jenis request.

Pada materi sebelumnya, kita sudah tahu bahwa route seperti ini harus memakai middleware:

- `auth`, untuk memastikan user sudah login,
- `admin`, untuk memastikan user yang login benar-benar memiliki role admin.

Dengan demikian, middleware adalah penjaga pintunya, sedangkan route adalah petunjuk menuju setiap ruangan.

## Masalah jika route admin ditulis satu per satu

Bayangkan kita menulis semua route admin secara terpisah di `routes/web.php`.

Contoh berikut **hanya untuk melihat masalahnya**, belum perlu kamu salin:

```php
Route::get('/admin/dashboard', [AdminDashboardController::class, 'index'])
    ->middleware(['auth', 'admin'])
    ->name('admin.dashboard');

Route::get('/admin/products', [ProductController::class, 'index'])
    ->middleware(['auth', 'admin'])
    ->name('admin.products.index');

Route::get('/admin/products/create', [ProductController::class, 'create'])
    ->middleware(['auth', 'admin'])
    ->name('admin.products.create');

Route::get('/admin/orders', [OrderController::class, 'index'])
    ->middleware(['auth', 'admin'])
    ->name('admin.orders.index');

Route::get('/admin/users', [UserController::class, 'index'])
    ->middleware(['auth', 'admin'])
    ->name('admin.users.index');
```

Mari lihat bagian yang berulang:

```text
/admin
admin.
['auth', 'admin']
```

Ketiga aturan tersebut ditulis berulang-ulang pada hampir setiap route admin.

Masalahnya bukan karena kode ini langsung salah. Laravel tetap dapat menjalankannya. Namun, jika route makin banyak, cara ini membuat file route menjadi:

1. **Panjang**, karena aturan yang sama terus ditulis ulang.
2. **Sulit dibaca**, karena route admin bercampur dengan route lain.
3. **Mudah salah**, karena kita mungkin lupa memasang middleware pada satu route baru.
4. **Sulit dirawat**, karena saat aturan admin berubah, banyak baris harus diperiksa satu per satu.

Contoh kesalahan yang berbahaya:

```text
Route daftar produk memakai auth dan admin
Route tambah produk memakai auth dan admin
Route hapus produk lupa memakai admin
```

Akibatnya, user biasa yang sudah login mungkin bisa membuka atau menjalankan route hapus tersebut. Kita tidak ingin ada satu pintu area admin yang lupa dijaga.

## Apa itu route group?

**Route group** adalah cara untuk mengumpulkan beberapa route yang memiliki aturan bersama.

Kembali ke analogi toko:

```text
Satu pintu masuk area admin
        ↓
Di belakangnya ada dashboard, produk, order, dan user
        ↓
Semua ruangan mengikuti aturan masuk yang sama
```

Di Laravel, route group memungkinkan kita membuat satu “area admin”. Semua route yang diletakkan di dalam area itu dapat menerima aturan yang sama secara otomatis.

Aturan bersama yang akan kita gunakan nanti adalah:

| Aturan | Fungsi sederhananya |
| --- | --- |
| Prefix `admin` | Menambahkan `/admin` di depan URL halaman admin. |
| Name `admin.` | Menambahkan `admin.` di depan nama route admin. |
| Middleware `auth` | Memeriksa apakah user sudah login. |
| Middleware `admin` | Memeriksa apakah user yang login memiliki role admin. |

Dengan route group, kita cukup menulis aturan ini **sekali** pada pembungkus group. Lalu dashboard, CRUD Product, order, dan user ditulis di dalamnya.

## Kenapa route group membuat kode lebih rapi?

Route group membuat file `routes/web.php` lebih mudah dibaca karena Laravel dan programmer dapat melihat batas area admin dengan jelas.

Bayangkan sebuah map dokumen:

```text
Map Admin
├── Dashboard
├── Produk
├── Order
└── User
```

Daripada meletakkan dokumen admin di berbagai tempat, semuanya masuk ke satu map bernama **Admin**.

Hasil yang kita harapkan nantinya seperti ini:

```text
Route group admin
├── URL diawali /admin
├── Nama route diawali admin.
├── Harus melewati auth
├── Harus melewati admin
├── Dashboard admin
├── CRUD Product
├── Daftar order
└── Daftar user
```

Manfaatnya:

- kode route lebih singkat,
- route admin mudah ditemukan,
- aturan akses lebih konsisten,
- risiko lupa memasang middleware lebih kecil,
- saat aplikasi bertambah besar, struktur tetap lebih mudah dirawat.

## Yang perlu diingat pada tahap ini

Untuk sekarang, ingat empat hal berikut:

1. **Route** adalah aturan yang menghubungkan URL dengan controller atau tindakan Laravel.
2. **Route admin** adalah route untuk area pengelolaan, seperti dashboard, produk, order, dan user.
3. Jika route admin ditulis sendiri-sendiri, bagian `/admin`, nama `admin.`, dan middleware dapat berulang serta mudah terlupa.
4. **Route group** adalah pembungkus untuk mengumpulkan route yang memiliki aturan bersama.

Kita belum menulis route group pada tahap ini. Langkah berikutnya akan membahas tiga aturan penting di dalamnya secara perlahan: prefix `admin`, name `admin.`, dan middleware `auth`.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: memahami prefix `admin`, name `admin.`, dan middleware `auth`?**
