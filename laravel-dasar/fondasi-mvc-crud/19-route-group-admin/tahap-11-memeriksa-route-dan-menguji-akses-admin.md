# Tahap 11 — Memeriksa Route dan Menguji Akses Admin

> Fokus: memastikan route group admin benar-benar menghasilkan URL, nama route, dan middleware yang tepat, lalu menguji akses sebagai guest, user biasa, dan admin.

Pada tahap 10, link Blade, form, dan redirect CRUD Product sudah diperbarui ke nama route `admin.products.*`.

Sekarang jangan langsung menganggap semuanya selesai. Kita perlu membuktikan dua hal:

1. Laravel sudah mendaftarkan route admin dengan struktur yang benar.
2. Akses benar-benar ditolak atau diizinkan sesuai jenis user.

## Bayangkan memeriksa peta dan pintu toko

Route group sudah seperti satu area belakang toko yang rapi. Sebelum toko dibuka, kita melakukan dua pemeriksaan:

```text
Periksa peta gedung
→ Apakah alamat dashboard dan produk benar?

Periksa pintu masuk
→ Apakah guest dan user biasa ditolak?
→ Apakah admin bisa masuk?
```

Di Laravel:

```text
Periksa peta → php artisan route:list
Periksa pintu → uji URL di browser
```

## Bagian 1, memeriksa daftar route dengan Artisan

Dari folder utama project Laravel, jalankan:

```bash
php artisan route:list --path=admin
```

Perintah ini hanya menampilkan route yang URL-nya mengandung `admin`.

Laravel biasanya menampilkan informasi seperti:

- method HTTP,
- URI atau URL route,
- nama route,
- action controller,
- middleware.

Contoh hasil yang disederhanakan:

```text
GET|HEAD  admin/dashboard                admin.dashboard          AdminDashboardController@index  web, auth, admin
GET|HEAD  admin/products                 admin.products.index     ProductController@index         web, auth, admin
GET|HEAD  admin/products/create          admin.products.create    ProductController@create        web, auth, admin
POST      admin/products                 admin.products.store     ProductController@store         web, auth, admin
GET|HEAD  admin/products/{product}/edit  admin.products.edit      ProductController@edit          web, auth, admin
PUT|PATCH admin/products/{product}       admin.products.update    ProductController@update        web, auth, admin
DELETE    admin/products/{product}       admin.products.destroy   ProductController@destroy       web, auth, admin
```

Tampilan tepatnya dapat berbeda menurut versi Laravel dan route yang benar-benar tersedia pada project. Yang diperiksa adalah isi pentingnya.

## Checklist hasil `route:list`

Periksa setiap route dashboard dan Product yang memang kamu miliki.

| Bagian yang diperiksa | Hasil yang diharapkan |
| --- | --- |
| URL dashboard | `admin/dashboard` |
| Nama dashboard | `admin.dashboard` |
| URL daftar Product | `admin/products` |
| Nama daftar Product | `admin.products.index` |
| URL form tambah | `admin/products/create` |
| Nama form tambah | `admin.products.create` |
| Middleware | Ada `auth` dan `admin` |
| Controller | Mengarah ke controller serta method yang benar |

Jika tahap 9 sudah diterapkan dan controllernya tersedia, daftar route dapat juga berisi:

```text
admin/orders → admin.orders.index
admin/users  → admin.users.index
```

## Jika hasil route belum benar

Gunakan panduan cepat ini.

| Gejala | Kemungkinan penyebab | Periksa |
| --- | --- | --- |
| URL masih `products` tanpa `admin` | Resource masih berada di luar group | Pindahkan `Route::resource('products', ...)` ke dalam group. |
| Nama masih `products.index` | Resource tidak berada dalam `name('admin.')` | Periksa posisi resource dan awalan nama group. |
| Tidak ada middleware `admin` | Group hanya memakai `auth`, atau route berada di luar group | Pastikan `middleware(['auth', 'admin'])`. |
| Route `admin.*` tidak ada | Group belum ditulis, atau ada kesalahan syntax | Periksa `routes/web.php`, lalu jalankan ulang `route:list`. |
| Error middleware `[admin]` tidak dikenal | Alias belum terdaftar | Periksa `bootstrap/app.php` dari materi 18. |

Jika baru mengubah konfigurasi alias middleware dan hasilnya masih tidak terbaca, kamu dapat menjalankan:

```bash
php artisan config:clear
```

Perintah tersebut membersihkan cache konfigurasi. Ia tidak menghapus Product atau user dari database.

## Bagian 2, siapkan tiga kondisi pengujian

Untuk menguji penjagaan route, siapkan tiga keadaan berikut:

| Kondisi | Yang dibutuhkan |
| --- | --- |
| Guest | Browser tidak sedang login. |
| User biasa | Akun dengan `role` bernilai `user`. |
| Admin | Akun dengan `role` bernilai `admin`. |

Role ini adalah kelanjutan dari materi 18. Jangan mengubah role melalui form register umum. Gunakan akun latihan local yang sudah kamu siapkan dengan aman pada materi sebelumnya.

Gunakan URL yang benar-benar tersedia, misalnya:

```text
/admin/dashboard
/admin/products
/admin/products/create
```

Jika dashboard belum dibuat pada project, cukup gunakan `/admin/products`.

## Pengujian 1, guest belum login

1. Logout dari aplikasi, atau buka jendela incognito/private.
2. Buka URL admin, misalnya:

```text
/admin/products
```

3. Amati hasilnya.

Hasil yang diharapkan:

```text
Guest membuka /admin/products
        ↓
auth tidak menemukan user login
        ↓
Laravel mengarahkan guest ke halaman login
        ↓
ProductController@index tidak dijalankan
```

Jika guest dapat melihat halaman admin, berhenti dan periksa route. Kemungkinan masih ada route lama `/products` atau route admin tidak memakai middleware `auth`.

## Pengujian 2, user biasa sudah login

1. Login menggunakan akun dengan role `user`.
2. Ketik URL admin langsung pada browser:

```text
/admin/products/create
```

3. Coba juga halaman daftar Product atau dashboard yang tersedia.

Hasil yang diharapkan:

```text
User biasa membuka /admin/products/create
        ↓
auth berhasil, karena user sudah login
        ↓
admin memeriksa role
        ↓
Role bukan admin
        ↓
Laravel memberi respons 403 Forbidden
```

**403 Forbidden** berarti user dikenali, tetapi tidak diberi izin untuk menjalankan tindakan tersebut.

Jangan hanya menguji tombol yang disembunyikan. User biasa harus tetap ditolak ketika mengetik URL admin langsung.

## Pengujian 3, admin sudah login

1. Logout dari akun user biasa.
2. Login menggunakan akun dengan role `admin`.
3. Buka halaman berikut satu per satu jika fitur tersedia:

```text
/admin/dashboard
/admin/products
/admin/products/create
/admin/products/{id}/edit
```

Ganti `{id}` dengan ID Product dummy yang benar-benar ada di database local.

Hasil yang diharapkan:

```text
Admin membuka /admin/products
        ↓
auth berhasil
        ↓
admin berhasil, karena role adalah admin
        ↓
ProductController@index dijalankan
        ↓
Daftar Product tampil
```

Sebagai admin, uji juga alur CRUD yang aman di database local:

- membuka form tambah,
- menyimpan Product dummy,
- membuka form edit,
- memperbarui Product dummy,
- menghapus Product dummy bila memang ingin menguji aksi hapus.

Jangan memakai data production untuk latihan hapus.

## Ringkasan hasil akses

| Pengunjung | URL admin | Hasil yang benar |
| --- | --- | --- |
| Guest | `/admin/products` | Dialihkan ke login. |
| User biasa | `/admin/products` | 403 Forbidden. |
| User biasa | `/admin/products/create` | 403 Forbidden. |
| Admin | `/admin/products` | Daftar Product tampil. |
| Admin | `/admin/products/create` | Form tambah Product tampil. |
| Admin | Route simpan, update, hapus | Diizinkan sesuai method dan validasi aplikasi. |

## Mengapa perlu menguji simpan, update, dan hapus?

Halaman daftar memakai request `GET`, tetapi CRUD juga memiliki request yang mengubah data.

| Aksi | Method | Nama route | User biasa harus mendapat |
| --- | --- | --- | --- |
| Simpan Product | `POST` | `admin.products.store` | 403 |
| Update Product | `PUT` atau `PATCH` | `admin.products.update` | 403 |
| Hapus Product | `DELETE` | `admin.products.destroy` | 403 |

Karena `Route::resource('products', ProductController::class)` berada di dalam group, middleware berlaku untuk semua route tersebut, bukan hanya route daftar.

## Jika admin mendapat 403

Jika akun yang seharusnya admin masih mendapat 403, periksa nilai role akun pada database local.

Kamu dapat menggunakan Tinker:

```bash
php artisan tinker
```

Lalu cari akun dengan email akun local kamu sendiri:

```php
App\Models\User::where('email', 'admin@example.test')
    ->firstOrFail()
    ->only(['name', 'email', 'role']);
```

Ganti `admin@example.test` dengan email akun admin local milikmu. Hasilnya harus memperlihatkan:

```text
role => admin
```

Keluar dari Tinker dengan:

```text
exit
```

Jika role sudah benar, periksa kembali tiga bagian dari materi 18:

```text
AdminMiddleware.php → membandingkan role dengan 'admin'
bootstrap/app.php    → alias 'admin' terdaftar
routes/web.php       → group memakai ['auth', 'admin']
```

## Jika ada error nama route tidak ditemukan

Contoh error:

```text
Route [products.index] not defined.
```

Artinya masih ada Blade atau controller yang memakai nama lama. Cari dan ubah:

```text
products.* → admin.products.*
```

Contoh error dashboard:

```text
Route [dashboard] not defined.
```

Periksa link atau redirect yang seharusnya mengarah ke dashboard admin, lalu gunakan:

```text
admin.dashboard
```

Ini adalah lanjutan langsung dari tahap 10.

## Pilihan lanjutan, automated test Laravel

Pengujian browser di atas paling mudah untuk pemula. Saat kamu mulai belajar testing Laravel, aturan akses ini dapat dibuat menjadi test otomatis.

Contoh konsep test feature:

```php
use App\Models\User;

it('menolak user biasa dari daftar Product admin', function () {
    $user = User::factory()->create([
        'role' => 'user',
    ]);

    $this->actingAs($user)
        ->get(route('admin.products.index'))
        ->assertForbidden();
});
```

Maknanya:

- `User::factory()` membuat user test.
- `actingAs($user)` menjalankan request seolah-olah user itu sudah login.
- `get(route(...))` membuka URL berdasarkan nama route.
- `assertForbidden()` memastikan responsnya 403.

Contoh ini adalah pilihan lanjutan, bukan syarat untuk menyelesaikan materi dasar. Jangan menambahkannya ke project jika kamu belum membahas struktur test, factory user, dan database test.

## Ringkasan

Setelah route group dibuat dan link diperbarui, selalu lakukan dua jenis pemeriksaan:

```text
php artisan route:list --path=admin
        ↓
Memastikan struktur route benar

Uji sebagai guest, user, dan admin
        ↓
Memastikan akses benar
```

Ingat hasil akses yang benar:

```text
Guest       → login
User biasa  → 403 Forbidden
Admin       → halaman admin terbuka
```

Jika hasil tersebut sesuai pada dashboard dan CRUD Product, route group admin sudah rapi sekaligus terlindungi. Tahap terakhir akan merangkum seluruh konsep, checklist, dan kesalahan umum route group admin.

---

**Apakah kamu ingin lanjut ke langkah terakhir: ringkasan dan checklist route group admin?**
