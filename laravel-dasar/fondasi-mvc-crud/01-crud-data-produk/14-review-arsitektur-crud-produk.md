# Tahap 14 — Review Arsitektur CRUD Produk

> **Tujuan tahap ini:** Memahami kembali struktur dan alur kerja CRUD Produk yang sudah dibangun selama Tahap 1-13. **Bukan untuk menambah fitur baru.** Fokus pada pemahaman arsitektur MVC, route, controller, model, dan view.

---

## 1. Pengantar Sederhana

Pada **Tahap 1 sampai 13**, kita sudah membuat **fitur CRUD Produk** secara lengkap.

CRUD Produk berarti aplikasi sudah bisa:

- Menambahkan produk.
- Menampilkan daftar produk.
- Melihat detail produk.
- Mengedit produk.
- Mengupdate produk.
- Menghapus produk.

### Analogi: Sistem Administrasi Toko

| Bagian Laravel     | Peran di Toko                                       |
| ------------------ | --------------------------------------------------- |
| Aplikasi Laravel   | Sistem administrasi toko                            |
| Database           | Lemari arsip                                        |
| Tabel `products`   | Map besar khusus data produk                        |
| Model `Product`    | Petugas arsip                                       |
| `ProductController`| Manajer toko                                        |
| Route              | Papan alamat / resepsionis                          |
| View Blade         | Etalase dan formulir yang dilihat pengguna          |

### Penting!

Tahap ini **bukan untuk menambah kode baru**, tetapi untuk **memahami kembali
arsitektur** yang sudah dibangun. Kita akan menelusuri kembali:

- Apa yang sudah dibuat.
- Bagaimana setiap bagian bekerja sama.
- Mengapa strukturnya seperti ini.
- Apa kelebihan dan kekurangannya.

---

## 2. Masalah Awal yang Ingin Diselesaikan

Sebelum ada aplikasi CRUD, toko mungkin mencatat produk **secara manual** di
buku atau Excel.

### Masalah yang muncul

- Data produk sulit dicari.
- Harga bisa salah catat.
- Stok tidak terpantau.
- Produk lama sulit diperbarui.
- Produk yang tidak dijual lagi sulit dihapus.
- Admin toko kesulitan mengelola banyak produk.

### Solusi yang dibangun

Kita membangun **fitur CRUD Produk** agar admin bisa mengelola data produk
lewat aplikasi web.

| Masalah                        | Solusi di Aplikasi          |
| ------------------------------ | --------------------------- |
| Produk sulit dicatat           | Buat form tambah produk     |
| Produk sulit dilihat           | Buat halaman daftar produk  |
| Informasi produk kurang lengkap | Buat halaman detail produk  |
| Data produk berubah            | Buat form edit + update     |
| Produk tidak dijual lagi       | Buat fitur hapus produk     |

---

## 3. Apa Itu CRUD dalam Studi Kasus Ini?

| CRUD   | Arti             | Contoh di Produk                    | Tahap        |
| ------ | ---------------- | ------------------------------------ | ------------ |
| Create | Membuat data baru | Menambah produk baru                 | Tahap 8 + 9  |
| Read   | Membaca data      | Melihat daftar dan detail produk     | Tahap 7 + 10 |
| Update | Mengubah data     | Mengedit nama, harga, stok produk    | Tahap 11 + 12|
| Delete | Menghapus data    | Menghapus produk dari database       | Tahap 13     |

### CRUD adalah fondasi banyak aplikasi web

Konsep yang sama bisa dipakai untuk:

- Data siswa.
- Data karyawan.
- Data artikel.
- Data transaksi.
- Data pelanggan.
- Data kategori.
- Data pesanan.

---

## 4. Struktur File yang Sudah Dibuat

```text
database/
  migrations/
    xxxx_xx_xx_xxxxxx_create_products_table.php

app/
  Models/
    Product.php
  Http/
    Controllers/
      ProductController.php

routes/
  web.php

resources/
  views/
    products/
      index.blade.php
      create.blade.php
      show.blade.php
      edit.blade.php
```

### Fungsi tiap file

| File/Folder        | Fungsi                                       |
| ------------------ | -------------------------------------------- |
| Migration          | Membuat struktur tabel `products`            |
| Model `Product`    | Menghubungkan Laravel dengan tabel `products`|
| `routes/web.php`   | Mendaftarkan alamat URL aplikasi             |
| `ProductController`| Mengatur alur fitur produk                   |
| `index.blade.php`  | Menampilkan daftar produk                    |
| `create.blade.php` | Menampilkan form tambah produk               |
| `show.blade.php`   | Menampilkan detail produk                    |
| `edit.blade.php`   | Menampilkan form edit produk                 |

---

## 5. Review Konsep MVC

**MVC** singkatan dari **Model - View - Controller**.

| Bagian     | Tugas                              | Di Studi Kasus Produk                    |
| ---------- | ---------------------------------- | ---------------------------------------- |
| Model      | Mengurus hubungan dengan data      | `Product.php`                            |
| View       | Mengurus tampilan                  | File Blade di `resources/views/products` |
| Controller | Mengatur alur permintaan user      | `ProductController.php`                  |

### Analogi toko

- **Model** seperti petugas arsip yang tahu cara mengambil dan menyimpan data.
- **View** seperti etalase atau formulir yang dilihat admin.
- **Controller** seperti manajer yang mengatur permintaan admin.

### Manfaat MVC

- Kode lebih rapi.
- Tugas tiap bagian lebih jelas.
- Aplikasi lebih mudah dipahami.
- Aplikasi lebih mudah dikembangkan.
- Error lebih mudah dilacak.

---

## 6. Alur Besar Aplikasi CRUD Produk

```text
User membuka halaman di browser
        |
        v
Route menerima alamat URL
        |
        v
Route memanggil method di Controller
        |
        v
Controller menjalankan proses
        |
        v
Jika perlu data, Controller memakai Model
        |
        v
Model berhubungan dengan database
        |
        v
Controller mengirim data ke View
        |
        v
View menampilkan halaman ke browser
```

### Penjelasan

User tidak langsung berbicara dengan database. User berbicara lewat browser,
Laravel mengatur alurnya melalui **route, controller, model, dan view**.

---

## 7. Review Alur Menampilkan Daftar Produk

```text
User membuka /products
        |
        v
Route GET /products
        |
        v
ProductController@index
        |
        v
Product::all()
        |
        v
Ambil semua data dari tabel products
        |
        v
Kirim data ke products.index
        |
        v
Tampilkan tabel daftar produk
```

### Kode route

```php
Route::get('/products', [ProductController::class, 'index']);
```

### Kode controller

```php
public function index()
{
    $products = Product::all();

    return view('products.index', compact('products'));
}
```

### Penjelasan

- Route menentukan alamat.
- Controller mengambil semua produk.
- View menampilkan produk dalam tabel.

---

## 8. Review Alur Tambah Produk

Fitur tambah produk terdiri dari **dua bagian**:

### Bagian 1: Menampilkan form tambah produk

```text
User membuka /products/create
        |
        v
Route GET /products/create
        |
        v
ProductController@create
        |
        v
Tampilkan products.create
```

**Kode:**

```php
Route::get('/products/create', [ProductController::class, 'create']);
```

```php
public function create()
{
    return view('products.create');
}
```

### Bagian 2: Menyimpan produk baru

```text
User mengisi form
        |
        v
User klik Simpan Produk
        |
        v
Form POST ke /products
        |
        v
Route POST /products
        |
        v
ProductController@store
        |
        v
Validasi data
        |
        v
Product::create()
        |
        v
Data masuk ke database
        |
        v
Redirect ke /products
```

**Kode:**

```php
Route::post('/products', [ProductController::class, 'store']);
```

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'name'        => 'required',
        'description' => 'nullable',
        'price'       => 'required|integer|min:0',
        'stock'       => 'required|integer|min:0',
    ]);

    Product::create($validated);

    return redirect('/products')->with('success', 'Produk berhasil ditambahkan.');
}
```

### Penjelasan

- Form tambah produk hanya **menampilkan input**.
- Method `store()` yang **benar-benar menyimpan data**.

---

## 9. Review Alur Detail Produk

```text
User klik Detail
        |
        v
Browser membuka /products/{id}
        |
        v
Route GET /products/{id}
        |
        v
ProductController@show
        |
        v
Product::findOrFail($id)
        |
        v
Kirim data ke products.show
        |
        v
Tampilkan detail produk
```

### Kode route

```php
Route::get('/products/{id}', [ProductController::class, 'show']);
```

### Kode controller

```php
public function show($id)
{
    $product = Product::findOrFail($id);

    return view('products.show', compact('product'));
}
```

### Penjelasan

- `{id}` adalah ID produk dari URL.
- `findOrFail()` mencari produk berdasarkan ID.
- Jika produk tidak ditemukan, Laravel menampilkan halaman **404**.

---

## 10. Review Alur Edit dan Update Produk

Fitur update terdiri dari **dua bagian**:

### Bagian 1: Menampilkan form edit

```text
User klik Edit
        |
        v
Browser membuka /products/{id}/edit
        |
        v
Route GET /products/{id}/edit
        |
        v
ProductController@edit
        |
        v
Cari produk berdasarkan ID
        |
        v
Tampilkan form edit berisi data lama
```

**Kode:**

```php
Route::get('/products/{id}/edit', [ProductController::class, 'edit']);
```

```php
public function edit($id)
{
    $product = Product::findOrFail($id);

    return view('products.edit', compact('product'));
}
```

### Bagian 2: Menyimpan perubahan

```text
User mengubah data
        |
        v
User klik Update Produk
        |
        v
Form mengirim PUT ke /products/{id}
        |
        v
Route PUT /products/{id}
        |
        v
ProductController@update
        |
        v
Cari produk berdasarkan ID
        |
        v
Validasi data baru
        |
        v
$product->update()
        |
        v
Data di database berubah
        |
        v
Redirect ke halaman detail produk
```

**Kode:**

```php
Route::put('/products/{id}', [ProductController::class, 'update']);
```

```php
public function update(Request $request, $id)
{
    $product = Product::findOrFail($id);

    $validated = $request->validate([
        'name'        => 'required',
        'description' => 'nullable',
        'price'       => 'required|integer|min:0',
        'stock'       => 'required|integer|min:0',
    ]);

    $product->update($validated);

    return redirect('/products/' . $product->id)->with('success', 'Produk berhasil diperbarui.');
}
```

### Penjelasan

- Form edit **menampilkan data lama**.
- Method `update()` **menyimpan perubahan** ke database.

---

## 11. Review Alur Hapus Produk

```text
User klik Hapus
        |
        v
Browser menampilkan konfirmasi
        |
        v
User pilih OK
        |
        v
Form mengirim DELETE ke /products/{id}
        |
        v
Route DELETE /products/{id}
        |
        v
ProductController@destroy
        |
        v
Cari produk berdasarkan ID
        |
        v
$product->delete()
        |
        v
Data terhapus dari database
        |
        v
Redirect ke daftar produk
```

### Kode route

```php
Route::delete('/products/{id}', [ProductController::class, 'destroy']);
```

### Kode controller

```php
public function destroy($id)
{
    $product = Product::findOrFail($id);

    $product->delete();

    return redirect('/products')->with('success', 'Produk berhasil dihapus.');
}
```

### Penjelasan

Hapus data memakai form dengan `@method('DELETE')`, bukan link biasa, karena
menghapus data adalah **aksi penting** yang mengubah database.

---

## 12. Review HTTP Method

| Method | Fungsi Umum            | Contoh Route             | Dipakai Untuh    |
| ------ | ---------------------- | ------------------------ | ---------------- |
| GET    | Membuka atau melihat   | `/products`              | Daftar produk    |
| GET    | Membuka form           | `/products/create`       | Form tambah      |
| POST   | Mengirim data baru     | `/products`              | Simpan produk    |
| GET    | Melihat detail         | `/products/{id}`         | Detail produk    |
| GET    | Membuka form edit      | `/products/{id}/edit`    | Form edit        |
| PUT    | Mengubah data lama     | `/products/{id}`         | Update produk    |
| DELETE | Menghapus data         | `/products/{id}`         | Hapus produk     |

### Penjelasan

URL bisa sama, tetapi tujuannya berbeda karena **HTTP method-nya berbeda**.

Contoh:

- GET `/products/1` berarti melihat detail produk.
- PUT `/products/1` berarti mengupdate produk.
- DELETE `/products/1` berarti menghapus produk.

---

## 13. Review Route yang Sudah Dibuat

```php
Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/create', [ProductController::class, 'create']);
Route::post('/products', [ProductController::class, 'store']);
Route::get('/products/{id}/edit', [ProductController::class, 'edit']);
Route::put('/products/{id}', [ProductController::class, 'update']);
Route::delete('/products/{id}', [ProductController::class, 'destroy']);
Route::get('/products/{id}', [ProductController::class, 'show']);
```

### Tabel route

| Route                  | Controller Method | Fungsi                       |
| ---------------------- | ----------------- | ---------------------------- |
| GET `/products`        | `index()`         | Menampilkan daftar produk    |
| GET `/products/create` | `create()`        | Menampilkan form tambah      |
| POST `/products`       | `store()`         | Menyimpan produk baru        |
| GET `/products/{id}`   | `show()`          | Menampilkan detail produk    |
| GET `/products/{id}/edit` | `edit()`       | Menampilkan form edit        |
| PUT `/products/{id}`   | `update()`        | Menyimpan perubahan produk   |
| DELETE `/products/{id}`| `destroy()`       | Menghapus produk             |

### Catatan urutan route

- `/products/create` harus ditulis **sebelum** `/products/{id}`.
- `/products/{id}/edit` sebaiknya ditulis **sebelum** `/products/{id}`.

Alasannya: agar Laravel tidak salah menganggap kata seperti `create` sebagai
nilai `{id}`.

---

## 14. Review ProductController

| Method       | Tugas                                |
| ------------ | ------------------------------------ |
| `index()`    | Menampilkan daftar produk            |
| `create()`   | Menampilkan form tambah produk       |
| `store()`    | Menyimpan produk baru                |
| `show()`     | Menampilkan detail satu produk       |
| `edit()`     | Menampilkan form edit produk         |
| `update()`   | Mengupdate produk                    |
| `destroy()`  | Menghapus produk                     |

### Penjelasan

Controller adalah **pusat pengatur alur** fitur produk.

Namun, controller **tidak seharusnya mengurus semua hal** secara berlebihan.
Untuk tahap dasar, controller seperti ini masih wajar. Pada materi lanjutan,
logic bisa dipisah ke **Form Request, Service, Action, atau struktur modular**
jika aplikasi makin besar.

> Catatan: **Jangan praktik refactor dulu** di tahap ini. Cukup pahami arahnya.

---

## 15. Review Model Product

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    protected $fillable = [
        'name',
        'description',
        'price',
        'stock',
    ];
}
```

### Penjelasan

Model `Product` adalah **penghubung Laravel dengan tabel `products`**.

### Fungsi `$fillable`

`$fillable` adalah daftar kolom yang **boleh diisi secara massal**.

| Kolom         | Boleh Diisi Lewat Form? | Alasan                        |
| ------------- | ----------------------- | ----------------------------- |
| `name`        | Ya                      | Data utama produk             |
| `description` | Ya                      | Deskripsi produk              |
| `price`       | Ya                      | Harga produk                  |
| `stock`       | Ya                      | Stok produk                   |
| `id`          | Tidak                   | Dibuat otomatis oleh database |
| `created_at`  | Tidak                   | Diatur otomatis Laravel       |
| `updated_at`  | Tidak                   | Diatur otomatis Laravel       |

### Tujuannya

Menjaga **keamanan** agar user tidak sembarangan mengisi kolom penting.

---

## 16. Review Migration Products

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->text('description')->nullable();
    $table->integer('price');
    $table->integer('stock')->default(0);
    $table->timestamps();
});
```

### Fungsi kolom

| Kolom         | Fungsi                           |
| ------------- | -------------------------------- |
| `id`          | Nomor unik produk                |
| `name`        | Nama produk                      |
| `description` | Deskripsi produk                 |
| `price`       | Harga produk                     |
| `stock`       | Jumlah stok                      |
| `created_at`  | Waktu produk dibuat              |
| `updated_at`  | Waktu produk terakhir diubah     |

### Penjelasan

Migration adalah **rancangan struktur tabel**. Sebelum data disimpan, kita harus
menyiapkan tempat penyimpanannya terlebih dahulu.

---

## 17. Review View Blade

| File View         | Fungsi                          |
| ----------------- | ------------------------------- |
| `index.blade.php` | Menampilkan daftar produk       |
| `create.blade.php`| Menampilkan form tambah produk  |
| `show.blade.php`  | Menampilkan detail produk       |
| `edit.blade.php`  | Menampilkan form edit produk    |

### Penjelasan

View **tidak seharusnya mengurus proses database**. View hanya **menerima data
dari controller** lalu menampilkannya ke user.

Contoh yang baik:

```blade
{{ $product->name }}
```

Contoh yang tidak ideal (query kompleks di View):

```blade
@php
    $products = App\Models\Product::where('stock', '>', 0)->get();
@endphp
```

Logic seperti ini sebaiknya diletakkan di **controller**, bukan di View.

---

## 18. Review Validasi Dasar

Validasi yang dipakai pada `store()` dan `update()`:

```php
$validated = $request->validate([
    'name'        => 'required',
    'description' => 'nullable',
    'price'       => 'required|integer|min:0',
    'stock'       => 'required|integer|min:0',
]);
```

### Tabel aturan

| Field         | Aturan    | Arti                            |
| ------------- | --------- | ------------------------------- |
| `name`        | required  | Nama produk wajib diisi         |
| `description` | nullable  | Deskripsi boleh kosong          |
| `price`       | required  | Harga wajib diisi               |
| `price`       | integer   | Harga harus angka bulat         |
| `price`       | min:0     | Harga tidak boleh negatif       |
| `stock`       | required  | Stok wajib diisi                |
| `stock`       | integer   | Stok harus angka bulat          |
| `stock`       | min:0     | Stok tidak boleh negatif        |

### Tujuannya

Mencegah **data buruk** masuk ke database:

- Nama produk kosong.
- Harga negatif.
- Stok negatif.
- Harga bukan angka.

---

## 19. Review CSRF

Setiap form POST, PUT, dan DELETE memakai:

```blade
@csrf
```

### Penjelasan

`@csrf` adalah **pelindung keamanan** bawaan Laravel.

### Analogi

Seperti **stempel resmi** pada formulir. Laravel hanya menerima formulir yang
punya stempel resmi dari aplikasinya sendiri.

### Form yang memakai `@csrf`

- Form tambah produk.
- Form edit produk.
- Form hapus produk.

---

## 20. Review Method Spoofing: PUT dan DELETE

HTML form biasa hanya mendukung:

- GET
- POST

Karena itu Laravel menyediakan direktif:

```blade
@method('PUT')    <!-- untuk update -->
```

dan

```blade
@method('DELETE') <!-- untuk hapus -->
```

### Tabel method spoofing

| Kebutuhan    | Form HTML | Bantuan Laravel    |
| ------------ | --------- | ------------------ |
| Tambah data  | POST      | Tidak perlu `@method` |
| Update data  | POST      | `@method('PUT')`   |
| Hapus data   | POST      | `@method('DELETE')`|

### Analogi

Form tetap dikirim sebagai POST, tapi ada **catatan tambahan** di dalamnya yang
memberi tahu Laravel bahwa niat sebenarnya adalah **update atau hapus**.

---

## 21. Review Flash Message

Pesan sukses seperti:

```php
return redirect('/products')->with('success', 'Produk berhasil ditambahkan.');
```

Atau:

```php
return redirect('/products')->with('success', 'Produk berhasil dihapus.');
```

### Penjelasan

**Flash message** adalah pesan sementara yang muncul **setelah aksi berhasil**.

Contoh:

- Produk berhasil ditambahkan.
- Produk berhasil diperbarui.
- Produk berhasil dihapus.

### Analogi

Seperti **nota kecil** yang diberikan setelah pekerjaan selesai. Pesan ini
muncul sekali lalu hilang saat halaman direfresh.

---

## 22. Review Error Umum yang Pernah Dipelajari

| Error                                | Penyebab Umum                      | Solusi                                  |
| ------------------------------------ | ---------------------------------- | --------------------------------------- |
| View not found                       | File Blade belum dibuat/path salah | Cek folder `resources/views/products`   |
| Method not supported                 | Route POST/PUT/DELETE belum dibuat | Cek `routes/web.php`                     |
| Controller method does not exist     | Method belum dibuat                | Cek `ProductController`                  |
| Class Product not found              | Model belum di-import              | Tambahkan `use App\Models\Product;`     |
| Table products does not exist        | Migration belum dijalankan         | Jalankan `php artisan migrate`          |
| Mass assignment error                | `$fillable` belum dibuat           | Tambahkan `$fillable` di Model          |
| 404 Not Found                        | Data dengan ID tersebut tidak ada  | Cek data di database                     |

### Penjelasan

Memahami error adalah **bagian penting** dari belajar programming.
Error bukan tanda gagal, tapi **petunjuk** bagian mana yang perlu diperbaiki.

---

## 23. Kenapa Arsitektur Ini Sudah Cukup Baik untuk Tahap Dasar?

Arsitektur CRUD ini **cukup baik** untuk pemula karena:

- Route tidak lagi berisi semua logic.
- Controller mengatur alur fitur.
- Model mengurus hubungan ke database.
- View hanya menampilkan tampilan.
- Migration mengatur struktur database.
- Validasi mencegah input buruk.
- CSRF melindungi form.
- Flash message memberi feedback ke user.

### Tabel prinsip arsitektur

| Prinsip Arsitektur      | Contoh di CRUD Produk                          |
| ----------------------- | --------------------------------------------- |
| Separation of Concerns  | Route, Controller, Model, View punya tugas sendiri |
| MVC                     | Produk dibuat dengan Model, View, Controller  |
| Data Validation         | Input dicek sebelum disimpan                  |
| Security Basic          | Form memakai `@csrf`                          |
| User Feedback           | Ada pesan sukses                              |
| Maintainability         | File dipisah sesuai fungsi                    |

### Pesan penting

Arsitektur yang baik **bukan berarti langsung rumit**. Arsitektur yang baik
adalah yang **mudah dipahami, mudah dirawat, dan mudah dikembangkan**.

---

## 24. Apa Kekurangan Versi CRUD Dasar Ini?

Mari jujur, CRUD ini masih versi dasar.

### Kekurangan

- Validasi masih ditulis langsung di controller.
- Tampilan HTML masih berulang di setiap Blade.
- Belum memakai layout Blade reusable.
- Belum memakai named route.
- Belum memakai route resource.
- Belum ada pagination.
- Belum ada search.
- Belum ada upload gambar.
- Belum ada soft delete.
- Belum ada authentication.
- Belum ada authorization.
- Belum ada testing.
- Belum ada service layer.

### Pesan penting

Ini **bukan masalah** untuk tahap dasar. Justru bagus karena kita **mulai dari
sederhana dulu**.

Gunakan prinsip: **mulai sederhana, pahami alur, baru refactor ketika sudah mengerti.**

---

## 25. Perbandingan Kode Pemula dan Arah Kode yang Lebih Rapi

Kode sekarang sudah cukup untuk belajar. Namun nanti bisa diperbaiki secara bertahap.

| Saat Ini                       | Nanti Bisa Diperbaiki Dengan       |
| ------------------------------ | ---------------------------------- |
| Route ditulis satu per satu    | Route resource                     |
| Validasi di controller         | Form Request                       |
| HTML berulang                  | Layout Blade dan component         |
| URL ditulis manual             | Named route                        |
| Query sederhana di controller  | Tetap boleh untuk dasar            |
| Logic bisnis sederhana         | Service/Action jika mulai kompleks |

### Pesan penting

Jangan langsung memakai semua pola desain. Pakai ketika **masalahnya benar-benar muncul**.

---

## 26. Pelajaran Arsitektur Utama dari Studi Kasus Ini

### 1. Pisahkan tanggung jawab

- Route untuk alamat.
- Controller untuk alur.
- Model untuk data.
- View untuk tampilan.

### 2. Jangan taruh semua logic di satu tempat

Kalau semua kode ditaruh di route atau view, aplikasi akan **cepat berantakan**.

### 3. Database harus dirancang dulu

Sebelum menyimpan produk, kita membuat migration agar struktur data jelas.

### 4. Validasi penting

Jangan percaya semua input user. Input harus dicek sebelum masuk database.

### 5. Feedback ke user penting

Setelah tambah, update, atau hapus, user perlu tahu apakah aksinya berhasil.

### 6. Mulai dari sederhana

Tidak semua aplikasi langsung butuh arsitektur kompleks. Untuk pemula,
MVC CRUD seperti ini adalah **fondasi yang sangat penting**.

---

## 27. Ringkasan Perjalanan Tahap 1 sampai 13

| Tahap | Materi                        | Hasil                                           |
| ----- | ----------------------------- | ----------------------------------------------- |
| 1     | Pengenalan CRUD               | Paham masalah dan konsep CRUD                   |
| 2     | Persiapan Laravel + database  | Project dan database siap                       |
| 3     | Migration products            | Tabel `products` siap                           |
| 4     | Model Product                 | Laravel bisa berhubungan dengan tabel produk    |
| 5     | Route Produk                  | URL produk mulai dibuat                         |
| 6     | ProductController             | Alur produk dipindah ke controller              |
| 7     | Daftar Produk                 | Produk bisa ditampilkan                         |
| 8     | Form Tambah                   | Form tambah produk tampil                       |
| 9     | Simpan Produk                 | Produk bisa disimpan                            |
| 10    | Detail Produk                 | Satu produk bisa dilihat detailnya              |
| 11    | Form Edit                     | Form edit produk tampil                         |
| 12    | Update Produk                 | Produk bisa diperbarui                          |
| 13    | Hapus Produk                  | Produk bisa dihapus                             |
| 14    | Review Arsitektur             | **Memahami struktur dan alur CRUD**             |

---

## 28. Checklist Pemahaman Arsitektur

Centang (`x`) jika sudah paham:

- [ ] Saya paham masalah yang diselesaikan oleh CRUD Produk.
- [ ] Saya paham arti Create, Read, Update, Delete.
- [ ] Saya paham fungsi migration.
- [ ] Saya paham fungsi Model `Product`.
- [ ] Saya paham fungsi route.
- [ ] Saya paham fungsi `ProductController`.
- [ ] Saya paham fungsi View Blade.
- [ ] Saya paham alur user dari browser sampai database.
- [ ] Saya paham kenapa form tambah memakai POST.
- [ ] Saya paham kenapa form edit memakai `@method('PUT')`.
- [ ] Saya paham kenapa form hapus memakai `@method('DELETE')`.
- [ ] Saya paham fungsi `@csrf`.
- [ ] Saya paham fungsi validasi dasar.
- [ ] Saya paham fungsi `$fillable`.
- [ ] Saya paham fungsi flash message.
- [ ] Saya paham bahwa arsitektur dasar ini sudah cukup baik untuk pemula.
- [ ] Saya paham bahwa peningkatan arsitektur bisa dilakukan bertahap nanti.

Jika semua sudah tercentang, pemahaman arsitektur CRUD kamu sudah solid.

---

## 29. Kesimpulan Tahap 14

Pada tahap ini kita **tidak menambah fitur baru**, tetapi **merapikan pemahaman**.

### Analogi

Setelah membangun toko sederhana, sekarang kita **berjalan keliling toko** dan
melihat:

- Di mana lemari arsipnya (database).
- Siapa petugasnya (model).
- Di mana meja adminnya (controller).
- Di mana etalasenya (view).
- Bagaimana alur pelanggan datang sampai data tersimpan (route -> controller -> model -> database).

### Fondasi penting

CRUD Produk adalah **fondasi penting** sebelum masuk ke studi kasus berikutnya.

Jika kamu sudah memahami CRUD Produk, maka konsep yang sama bisa digunakan untuk
fitur lain seperti:

- CRUD kategori.
- CRUD pelanggan.
- CRUD artikel.
- CRUD siswa.
- CRUD karyawan.
- CRUD pesanan.

Cukup ganti nama model, nama tabel, dan kolomnya. Pola alurnya **sama persis**.

### Jika sudah paham review arsitektur CRUD Produk, ketik:

```text
lanjut
```

Jangan lanjut ke studi kasus berikutnya sebelum kamu meminta. Pelan-pelan, asal paham.
