# Tahap 5 — Membuat Route Produk

> **Tujuan tahap ini:** Memahami konsep **Route** dan membuat route produk pertama. **Belum membuat controller, view, form, atau mengambil data dari database.** Hanya belajar bagaimana route bekerja.

---

## 1. Pengantar Sederhana

Pada tahap-tahap sebelumnya, kita sudah:

- **Tahap 3**: Membuat tabel `products` di database melalui **migration**.
- **Tahap 4**: Membuat Model `Product` sebagai **penghubung** ke tabel `products`.

Tapi sampai di sini, aplikasi Laravel kita masih "tertutup". Belum ada pintu
masuk bagi pengguna untuk melihat produk. Belum ada URL yang bisa diketik
di browser untuk menampilkan data produk.

Di tahap ini kita akan belajar tentang **Route**, yaitu sistem yang menghubungkan
URL (alamat web) dengan aksi tertentu di dalam aplikasi Laravel.

### Analogi: Website seperti Gedung

Bayangkan aplikasi web Laravel kita adalah **gedung perkantoran besar**.

| Hal                   | Analogi                                  |
| --------------------- | ---------------------------------------- |
| Website (aplikasi)    | Gedung perkantoran                        |
| URL                   | Alamat ruangan (misal: Lt. 2, Ruang 205)  |
| **Route**             | **Petugas resepsionis** di lobi gedung    |
| Controller (nanti)    | Orang di dalam ruangan yang melayani tamu |
| View (nanti)          | Tampilan di dalam ruangan                 |

### Cara kerja resepsionis

Ketika seseorang datang ke gedung dan berkata "Saya mau ke ruang produk",
maka **resepsionis** akan:

1. Mendengarkan permintaan tersebut (URL apa yang diminta).
2. Mencari di daftar tugasnya (file route) siapa yang harus melayani.
3. Mengarahkan pengunjung ke ruangan/persona yang benar.

Sama persis dengan Laravel: saat user membuka sebuah URL, **route** akan
menentukan **apa yang harus dilakukan aplikasi**.

### Contoh

Jika user membuka di browser:

```text
/products
```

Maka Laravel akan **mencari di daftar route** apa yang harus dilakukan saat URL
itu dibuka. Misalnya: tampilkan daftar semua produk.

Jika user membuka:

```text
/products/create
```

Laravel akan mencari route lain yang sesuai, misalnya: tampilkan form tambah produk.

Route adalah **peta** yang menentukan setiap URL kemana arahnya.

---

## 2. Apa Itu Route?

### Pengertian sederhana

**Route** adalah aturan yang menyatakan:

> "Jika user membuka URL ini, jalankan aksi ini."

Dalam Laravel, route didefinisikan dengan kode PHP sederhana di sebuah file khusus.

### Analogi: DaftarAlamat di Kantor

Bayangkan kamu punya daftar alamat di dinding kantor:

| URL            | Tugaskan ke          |
| -------------- | -------------------- |
| `/`            | Halaman utama         |
| `/products`    | Bagian Produk         |
| `/products/create` | Bagian Tambah Produk |
| `/about`       | Bagian About          |

Setiap URL diarahkan ke **penangan (handler)** masing-masing.
Penangan ini, nantinya, biasanya adalah **method di Controller**.

> Catatan: Pada tahap ini, kita belum pakai Controller. Kita akan menulis
> route yang **langsung mengembalikan teks**. Tujuannya supaya kita paham
> bagaimana route bekerja tanpa kompleksitas controller dulu.

---

## 3. Di Mana File Route Berada?

Di Laravel, file route utama untuk aplikasi web berada di:

```
routes/web.php
```

### Isi default file `routes/web.php`

Saat kamu membuat project Laravel baru, isi file `routes/web.php` kira-kira seperti ini:

```php
<?php

use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});
```

### Penjelasan

- `use Illuminate\Support\Facades\Route;` - memanggil kelas Route dari Laravel.
- `Route::get('/', ...)` - mendefinisikan route untuk URL `/` (halaman utama).
- `function () { ... }` - fungsi yang dijalankan saat user membuka `/`.
- `return view('welcome');` - mengembalikan tampilan (view) bernama `welcome`.

Jadi, saat kamu membuka http://127.0.0.1:8000/, route ini aktif dan menampilkan halaman welcome Laravel.

---

## 4. Method HTTP: GET dan POST

Sebelum menulis route produk, kita perlu tahu sedikit tentang **method HTTP**.
Method adalah "jenis permintaan" yang dikirim browser ke server.

Dua method yang paling sering dipakai:

| Method   | Kapan dipakai                              | Contoh                            |
| -------- | ------------------------------------------ | --------------------------------- |
| **GET**  | Saat user **melihat/membaca** halaman       | Buka `/products` (lihat daftar)   |
| **POST** | Saat user **mengirim data** (form submit)   | Submit form tambah produk         |

### Analogi: ke Kantor Pos

- **GET** = kamu datang ke kantor pos hanya untuk **bertanya/melihat brosur**.
  Tidak mengirim apa pun, hanya menerima informasi.
- **POST** = kamu datang ke kantor pos **membawa paket** untuk dikirim.
  Kamu memberikan sesuatu (data form) ke petugas.

### Di Laravel

- `Route::get(...)` menangani permintaan GET.
- `Route::post(...)` menangani permintaan POST.

Pada tahap ini kita akan fokus ke **GET** dulu (menampilkan halaman),
karena belum membuat form (POST).

---

## 5. Membuat Route Produk Pertama

Sekarang kita akan menambahkan beberapa route untuk produk.
Tapi karena **belum punya controller dan view**, kita akan minta route
**langsung mengembalikan teks sederhana**. Tujuannya: memastikan route bekerja.

### Bukka file `routes/web.php`

Ganti isinya menjadi seperti ini:

```php
<?php

use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return 'Selamat datang di Toko Produk';
});

Route::get('/products', function () {
    return 'Halaman daftar produk';
});

Route::get('/products/create', function () {
    return 'Halaman form tambah produk';
});

Route::get('/products/{id}', function ($id) {
    return 'Halaman detail produk dengan ID: ' . $id;
});
```

### Penjelasan tiap route

#### 1. `Route::get('/', ...)`

URL: `http://127.0.0.1:8000/`

Artinya: saat user membuka halaman utama, tampilkan teks "Selamat datang di Toko Produk".

#### 2. `Route::get('/products', ...)`

URL: `http://127.0.0.1:8000/products`

Artinya: saat user membuka `/products`, tampilkan teks "Halaman daftar produk".

#### 3. `Route::get('/products/create', ...)`

URL: `http://127.0.0.1:8000/products/create`

Artinya: saat user membuka `/products/create`, tampilkan teks "Halaman form tambah produk".

#### 4. `Route::get('/products/{id}', ...)`

URL: `http://127.0.0.1:8000/products/5`

Artinya: saat user membuka `/products/5`, tampilkan teks "Halaman detail produk dengan ID: 5".

Bagian `{id}` disebut **parameter route** - nilainya bisa berubah-ubah tergantung URL yang dibuka.

### Parameter Route `{id}`

Perhatikan route:

```php
Route::get('/products/{id}', function ($id) {
    return 'Halaman detail produk dengan ID: ' . $id;
});
```

- `{id}` di URL ditangkap dan dikirim ke fungsi sebagai variabel `$id`.
- Jika user buka `/products/1`, maka `$id` berisi `'1'`.
- Jika user buka `/products/99`, maka `$id` berisi `'99'`.

Ini berguna nanti untuk melihat detail produk berdasarkan ID.

### Urutan route penting!

Perhatikan bahwa kita menulis `/products/create` **sebelum** `/products/{id}`.
Ini karena Laravel mencocokkan route dari atas ke bawah.

Jika `/products/{id}` ditulis lebih dulu, maka saat user membuka `/products/create`,
Laravel menganggap `create` sebagai nilai `$id`. Akibatnya, halaman form tambah
tidak terbuka, melainkan dianggap detail produk dengan ID = "create".

**Aturan umum:** route yang lebih spesifik ditulis lebih dulu.

---

## 6. Menjalankan dan Menguji Route

### Jalankan server Laravel

Di terminal (di dalam folder project):

```bash
php artisan serve
```

### Coba buka URL berikut di browser

| URL                                              | Hasil yang tampil                          |
| ------------------------------------------------ | ------------------------------------------ |
| http://127.0.0.1:8000/                           | Selamat datang di Toko Produk              |
| http://127.0.0.1:8000/products                   | Halaman daftar produk                      |
| http://127.0.0.1:8000/products/create            | Halaman form tambah produk                 |
| http://127.0.0.1:8000/products/5                 | Halaman detail produk dengan ID: 5         |
| http://127.0.0.1:8000/products/100               | Halaman detail produk dengan ID: 100       |

Jika semua URL menampilkan teks yang sesuai, berarti route kamu sudah bekerja.

### Catatan penting

Teks yang tampil saat ini **bukan** data dari database. Itu hanya teks statis
dari fungsi route. Belum ada koneksi ke model `Product` atau tabel `products`.

Di tahap selanjutnya, ketika kita membuat **Controller**, fungsi-fungsi route
ini akan kita pindahkan ke controller, dan route akan memanggil controller
alih-alih menulis fungsi langsung di file route.

---

## 7. Daftar Route Produk (Perencanaan ke Depan)

Untuk fitur CRUD lengkap, nanti kita akan punya route seperti ini:

| Method   | URL                  | Tujuan                          | Aksi CRUD  |
| -------- | -------------------- | ------------------------------- | ---------- |
| GET      | `/products`          | Tampilkan daftar produk          | Read       |
| GET      | `/products/create`   | Tampilkan form tambah produk     | Create     |
| POST     | `/products`          | Simpan produk baru dari form     | Create     |
| GET      | `/products/{id}`     | Tampilkan detail produk          | Read       |
| GET      | `/products/{id}/edit`| Tampilkan form edit produk       | Update     |
| PUT/PATCH| `/products/{id}`     | Simpan perubahan dari form edit  | Update     |
| DELETE   | `/products/{id}`     | Hapus produk                     | Delete     |

> Catatan: Jangan khawatir jika belum paham semua. Kita akan membuatnya
> satu per satu di tahap-tahap berikutnya, seiring kita membangun Controller dan View.

Pada tahap ini, kita hanya membuat versi **GET** sederhana untuk membuktikan
route bekerja dan memahami konsepnya.

---

## 8. Cara Melihat Semua Route yang Terdaftar

Laravel punya perintah untuk melihat semua route yang sudah dibuat:

```bash
php artisan route:list
```

Kamu akan melihat tabel berisi semua route, method, URI, dan handler-nya.
Ini sangat berguna untuk memeriksa apakah route kamu sudah benar.

Contoh output (dipotong):

```
GET|HEAD   .............................. .....................................
GET|HEAD   products ..................... closure
GET|HEAD   products/create .............. closure
GET|HEAD   products/{id} ................ closure
```

Kata `closure` artinya route tersebut ditangani oleh fungsi anonim (seperti
yang kita tulis di file `web.php`). Nanti setelah pakai Controller, akan muncul
nama controller di tempat `closure`.

---

## 9. Ringkasan Alur Tahap Ini

```
1. Buka file routes/web.php
        |
        v
2. Tambahkan route produk: /products, /products/create, /products/{id}
        |
        v
3. php artisan serve
        |
        v
4. Cek di browser: 127.0.0.1:8000/products
        |
        v
5. Pastikan teks yang diharapkan muncul
```

---

## 10. Checklist Keberhasilan Tahap Ini

Centang (`x`) jika sudah selesai:

- [ ] Saya paham bahwa route adalah aturan "URL ini -> aksi ini".
- [ ] Saya paham perbedaan GET (lihat) dan POST (kirim data).
- [ ] Saya sudah mengedit file `routes/web.php`.
- [ ] Route `/products` dapat diakses dan menampilkan teks.
- [ ] Route `/products/create` dapat diakses dan menampilkan teks.
- [ ] Route `/products/{id}` dapat diakses dengan ID berbeda.
- [ ] Saya sudah mencoba `php artisan route:list`.

Jika semua sudah tercentang, route dasar produk sudah siap.

---

## 11. Penutup

Selamat! Kamu sudah:

- Memahami konsep **route**.
- Menulis beberapa route GET pertama.
- Menggunakan parameter route `{id}`.
- Menguji bahwa setiap URL mengarah ke teks yang benar.

Saat ini, file route berisi fungsi-fungsi kecil yang mengembalikan teks.
Cara ini bagus untuk belajar, tapi **tidak ideal** untuk aplikasi sungguhan,
karena kode jadi panjang dan sulit dirawat jika ditulis semua di `web.php`.

Di **tahap berikutnya**, kita akan membuat **Controller** - yaitu tempat
menyimpan semua logika aksi produk (fungsi-fungsi yang sekarang ada di route).
Nanti, route akan **memanggil** controller, bukan menulis fungsi sendiri.

### Jika sudah paham tahap ini, ketik:

```
lanjut
```

Jangan lanjut sebelum kamu benar-benar siap. Pelan-pelan, asal paham.
