# Tahap 6 — Membuat Controller Product

> **Tujuan tahap ini:** Membuat `ProductController` dan menghubungkannya dengan route. **Belum membuat View Blade, form, atau mengambil data dari database.** Controller hanya mengembalikan teks sederhana agar alur mudah dipahami.

---

## 1. Pengantar Sederhana

Pada **Tahap 5**, kita sudah membuat **route** produk. Route bertugas seperti
**resepsionis** di sebuah toko: ia menerima pengunjung dan mengarahkan mereka
ke tempat yang benar.

Saat ini, route kita masih menulis fungsi langsung di file `web.php`. Itu
cukup untuk belajar, tapi tidak ideal untuk aplikasi sungguhan karena kode
jadi berantakan kalau semua logika ditulis di file route.

Sekarang kita akan membuat **Controller** - tempat khusus untuk menyimpan
logika aplikasi.

### Analogi: Toko Lengkap

| Bagian Laravel   | Peran di Toko                                |
| ---------------- | --------------------------------------------- |
| **Website**      | Toko itu sendiri                              |
| **Route**        | Resepsionis - menerima pengunjung, arahkan    |
| **Controller**   | **Manajer toko** - menentukan apa yang harus dilakukan |
| **Model**        | Petugas arsip - berhubungan dengan data       |
| **View**         | Etalase/tampilan yang dilihat pengunjung      |

### Alurnya

1. Pengunjung datang (user membuka URL).
2. **Resepsionis** (Route) menyambut dan menunjuk **manajer** yang tepat.
3. **Manajer** (Controller) menerima permintaan, lalu:
   - Minta data ke **petugas arsip** (Model) jika perlu.
   - Menyiapkan tampilan di **etalase** (View).
4. Pengunjung menerima hasilnya (halaman web).

### Catatan penting

Pada tahap ini, **Controller belum mengambil data dari database** dan
**belum menampilkan View**. Controller hanya akan **mengembalikan teks sederhana**.

Tujuannya: kita fokus memahami **bagaimana route memanggil controller**, dan
**struktur dasar sebuah controller**. Data dan View akan datang di tahap selanjutnya.

---

## 2. Apa Itu Controller?

### Pengertian sederhana

**Controller** adalah bagian Laravel yang **mengatur alur permintaan dari user**.

Jika route adalah "pintu masuk", maka controller adalah "orang di dalam ruangan"
yang menerima tamu dan memutuskan apa yang akan dilakukan.

### Contoh alur

Jika user membuka:

```text
/products
```

Maka:

1. **Route** menerima permintaan, lalu menugaskan ke **ProductController**.
2. **ProductController** menjalankan method bernama `index()`.
3. Method `index()` bisa:
   - Mengambil data dari Model (tahap selanjutnya).
   - Mengembalikan sebuah View (tahap selanjutnya).
   - Atau untuk sekarang: **mengembalikan teks sederhana**.

### Analogi: Manajer Toko

Bayangkan manajer toko (`ProductController`) punya beberapa tugas tetap:

| Tugas Manajer (Method) | Kapan Dilakukan                           |
| ---------------------- | ----------------------------------------- |
| `index()`              | Saat pengunjung ingin **melihat semua produk** |
| `create()`             | Saat pengunjung ingin **menambah produk baru** |
| `show($id)`            | Saat pengunjung ingin **melihat detail satu produk** |
| `edit($id)`            | Saat pengunjung ingin **mengedit produk**      |
| `store(Request $r)`    | Saat form tambah produk **disubmit**           |
| `update(Request $r, $id)` | Saat form edit produk **disubmit**          |
| `destroy($id)`         | Saat pengunjung ingin **menghapus produk**     |

Setiap method menangani satu jenis aksi. Konvensi ini disebut **resource controller** -
 Laravel sudah punya standar penamaan method untuk CRUD lengkap.

---

## 3. Membuat ProductController

### Perintah terminal

Pastikan kamu berada **di dalam folder project** (`toko-produk`), lalu jalankan:

```bash
php artisan make:controller ProductController
```

### Arti perintah tersebut

| Bagian              | Arti                                                |
| ------------------- | --------------------------------------------------- |
| `php`               | Menjalankan PHP                                     |
| `artisan`           | Alat bantu bawaan Laravel                           |
| `make:controller`   | Perintah untuk membuat file Controller baru         |
| `ProductController` | Nama controller. Konvensi: `<Nama>` + `Controller`  |

### Hasilnya

Laravel membuat file baru di:

```
app/Http/Controllers/ProductController.php
```

Folder `app/Http/Controllers/` adalah tempat khusus semua controller di Laravel.

> Catatan penamaan:
> - **PascalCase** (huruf kapital di awal tiap kata).
> - Selalu diakhiri dengan kata `Controller`.
> - Contoh: `ProductController`, `UserController`, `OrderController`.

---

## 4. Struktur File ProductController

Buka file `app/Http/Controllers/ProductController.php`. Isi default yang dibuat Laravel:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ProductController extends Controller
{
    //
}
```

### Penjelasan tiap bagian

#### `namespace App\Http\Controllers;`

Memberitahu Laravel bahwa file ini berada di "folder virtual" `App\Http\Controllers`.
Seperti alamat: controller ini tinggal di `App\Http\Controllers`.

#### `use Illuminate\Http\Request;`

Memanggil kelas `Request` dari Laravel. `Request` akan dipakai nanti untuk
mengambil data yang dikirim lewat form (POST). Untuk sekarang belum dipakai,
tapi biarkan saja.

#### `class ProductController extends Controller`

Artinya: "Saya membuat kelas bernama `ProductController` yang mewarisi semua
kemampuan dari `Controller` bawaan Laravel."

Karena `extends Controller`, kelas ini otomatis punya kemampuan standar
sebagai controller Laravel.

---

## 5. Menambahkan Method ke ProductController

Sekarang kita akan menambahkan beberapa method dasar untuk produk.
Tapi **isi method masih sangat sederhana**: hanya mengembalikan teks.

### Kode lengkap ProductController

Ganti isi file `app/Http/Controllers/ProductController.php` menjadi:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ProductController extends Controller
{
    // Menampilkan daftar semua produk
    public function index()
    {
        return 'Halaman daftar produk (dari Controller)';
    }

    // Menampilkan form tambah produk
    public function create()
    {
        return 'Halaman form tambah produk (dari Controller)';
    }

    // Menyimpan produk baru dari form
    public function store(Request $request)
    {
        return 'Menyimpan produk baru (dari Controller)';
    }

    // Menampilkan detail satu produk berdasarkan ID
    public function show($id)
    {
        return 'Halaman detail produk ID: ' . $id . ' (dari Controller)';
    }

    // Menampilkan form edit produk
    public function edit($id)
    {
        return 'Halaman form edit produk ID: ' . $id . ' (dari Controller)';
    }

    // Menyimpan perubahan dari form edit
    public function update(Request $request, $id)
    {
        return 'Mengupdate produk ID: ' . $id . ' (dari Controller)';
    }

    // Menghapus produk
    public function destroy($id)
    {
        return 'Menghapus produk ID: ' . $id . ' (dari Controller)';
    }
}
```

### Penjelasan tiap method

| Method            | Tugas                                     | Aksi CRUD |
| ----------------- | ------------------------------------------ | --------- |
| `index()`         | Menampilkan daftar semua produk            | Read      |
| `create()`        | Menampilkan form tambah produk             | Create   |
| `store(Request)`  | Menyimpan produk baru dari form submit     | Create   |
| `show($id)`       | Menampilkan detail satu produk             | Read      |
| `edit($id)`       | Menampilkan form edit produk               | Update   |
| `update(Request, $id)` | Menyimpan perubahan dari form edit   | Update   |
| `destroy($id)`    | Menghapus produk                           | Delete    |

> Catatan: Saat ini semua method hanya **mengembalikan teks**.
> Belum ada koneksi ke Model atau View. Ini disengaja agar kita fokus
> pada struktur controller dulu.

### Analogi: Tugas Manajer

Setiap method itu seperti satu tugas tetap manajer toko:

- `index()` = "Tampilkan semua produk ke pengunjung."
- `create()` = "Kasih form kosong untuk isi produk baru."
- `store()` = "Terima form yang sudah diisi, lalu simpan."
- `show($id)` = "Tunjukkan detail satu produk berdasarkan ID."
- `edit($id)` = "Kasih form yang sudah terisi untuk produk ID tertentu."
- `update()` = "Terima form edit, lalu simpan perubahan."
- `destroy($id)` = "Buang produk berdasarkan ID."

---

## 6. Menghubungkan Route ke Controller

Sekarang route kita akan **memanggil controller**, bukan menulis fungsi langsung.

### Bukka file `routes/web.php`

Ganti isinya menjadi seperti ini:

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ProductController;

Route::get('/', function () {
    return 'Selamat datang di Toko Produk';
});

// Route produk diarahkan ke ProductController
Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/create', [ProductController::class, 'create']);
Route::post('/products', [ProductController::class, 'store']);
Route::get('/products/{id}', [ProductController::class, 'show']);
Route::get('/products/{id}/edit', [ProductController::class, 'edit']);
Route::put('/products/{id}', [ProductController::class, 'update']);
Route::delete('/products/{id}', [ProductController::class, 'destroy']);
```

### Penjelasan baris penting

#### `use App\Http\Controllers\ProductController;`

Memberitahu file route: "Saya akan memakai `ProductController` yang ada di
`App\Http\Controllers`." Tanpa baris ini, Laravel tidak akan mengenali
`ProductController::class`.

#### `Route::get('/products', [ProductController::class, 'index']);`

Artinya: "Saat user membuka `/products` dengan GET, panggil method `index`
di `ProductController`."

Format umumnya:

```php
Route::get('/url', [NamaController::class, 'namaMethod']);
```

#### Route dengan parameter

```php
Route::get('/products/{id}', [ProductController::class, 'show']);
```

Nilai `{id}` dari URL akan diteruskan ke method `show($id)` di controller.
Jika user buka `/products/5`, maka `$id` di method `show` berisi `'5'`.

#### Method HTTP yang berbeda

Perhatikan kita memakai beberapa method HTTP berbeda untuk URL yang "mirip":

| Method   | URL                | Method Controller  | Tujuan                |
| -------- | ------------------ | ------------------ | --------------------- |
| GET      | `/products`        | `index`            | Lihat daftar          |
| POST     | `/products`        | `store`            | Simpan produk baru    |
| GET      | `/products/{id}`   | `show`             | Lihat detail          |
| PUT      | `/products/{id}`   | `update`           | Simpan edit           |
| DELETE   | `/products/{id}`   | `destroy`          | Hapus                 |

URL bisa sama (`/products` atau `/products/{id}`), tapi karena method HTTP-nya
berbeda, Laravel tahu itu adalah route yang berbeda dan memanggil method yang berbeda pula.

---

## 7. Cara Singkat: Resource Controller (Opsional)

Laravel punya jalan pintas untuk membuat semua route CRUD sekaligus, yaitu:

```php
Route::resource('/products', ProductController::class);
```

Satu baris ini otomatis membuat **7 route** yang sama dengan yang kita tulis manual:
`index`, `create`, `store`, `show`, `edit`, `update`, `destroy`.

> Catatan: Untuk belajar, kita tulis manual dulu agar **paham** apa yang terjadi.
> Setelah paham, di tahap selanjutnya kamu boleh memakai `Route::resource`
> untuk kode yang lebih singkat.

---

## 8. Menjalankan dan Menguji

### Jalankan server Laravel

```bash
php artisan serve
```

### Tes URL GET berikut di browser

| URL                                          | Hasil                                          |
| -------------------------------------------- | ---------------------------------------------- |
| http://127.0.0.1:8000/products               | Halaman daftar produk (dari Controller)         |
| http://127.0.0.1:8000/products/create        | Halaman form tambah produk (dari Controller)    |
| http://127.0.0.1:8000/products/5             | Halaman detail produk ID: 5 (dari Controller)   |
| http://127.0.0.1:8000/products/5/edit        | Halaman form edit produk ID: 5 (dari Controller)|

> Catatan: Route POST/PUT/DELETE **tidak bisa** dites langsung di browser
> (browser hanya mengirim GET saat mengetik URL). Nanti kita tes lewat form
> atau lewat Postman/Insomnia di tahap-tahap berikutnya.

### Cek dengan `php artisan route:list`

Jalankan:

```bash
php artisan route:list
```

Kamu akan melihat output seperti:

```
GET|HEAD   products .............. ProductController@index
POST       products .............. ProductController@store
GET|HEAD   products/create ....... ProductController@create
GET|HEAD   products/{id} ......... ProductController@show
PUT|PATCH  products/{id} ......... ProductController@update
DELETE     products/{id} ......... ProductController@destroy
GET|HEAD   products/{id}/edit .... ProductController@edit
```

Sekarang, alih-alih `closure`, handler route sudah berupa `ProductController@namaMethod`.
Ini cara yang lebih rapi dan ideal untuk aplikasi sungguhan.

---

## 9. Ringkasan Alur Tahap Ini

```
1. php artisan make:controller ProductController
        |
        v
2. Buka app/Http/Controllers/ProductController.php
        |
        v
3. Tambahkan 7 method: index, create, store, show, edit, update, destroy
        |
        v
4. Edit routes/web.php, arahkan semua route ke ProductController
        |
        v
5. php artisan serve, tes URL di browser
        |
        v
6. php artisan route:list untuk verifikasi
```

---

## 10. Checklist Keberhasilan Tahap Ini

Centang (`x`) jika sudah selesai:

- [ ] Saya paham bahwa controller adalah tempat menyimpan logika aksi aplikasi.
- [ ] Saya sudah membuat `ProductController` dengan `php artisan make:controller`.
- [ ] File `app/Http/Controllers/ProductController.php` berisi 7 method CRUD.
- [ ] File `routes/web.php` sudah mengarahkan semua route ke controller.
- [ ] URL `/products` menampilkan teks dari controller, bukan dari fungsi route langsung.
- [ ] `php artisan route:list` menampilkan `ProductController@...` sebagai handler.

Jika semua sudah tercentang, controller sudah siap dipakai.

---

## 11. Penutup

Selamat! Kamu sudah:

- Membuat **ProductController** dengan 7 method CRUD.
- Menghubungkan **route** ke **controller**.
- Menguji bahwa controller benar-benar dipanggil saat URL dibuka.
- Memahami konsep **resource controller**.

Sekarang, struktur kode kita sudah lebih rapi:

```
Route (resepsionis)  ->  Controller (manajer)  ->  Model (petugas arsip)
                                                       |
                                                       v
                                                  Database
```

Tapi sampai di sini, aplikasi masih **mengembalikan teks**, belum menampilkan
halaman HTML yang menarik. Di **tahap berikutnya**, kita akan belajar tentang
**View Blade** - yaitu cara membuat tampilan HTML yang dilihat oleh pengguna.

### Jika sudah paham tahap ini, ketik:

```
lanjut
```

Jangan lanjut sebelum kamu benar-benar siap. Pelan-pelan, asal paham.
