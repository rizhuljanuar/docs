# Tahap 4 — Membuat Model Product

> **Tujuan tahap ini:** Membuat Model `Product` yang menjadi penghubung antara Laravel dan tabel `products` di database. **Belum membuat controller, route, view, form, atau fitur CRUD.** Belum mengambil/menyimpan data apa pun.

---

## 1. Pengantar Sederhana

Pada **Tahap 3**, kita sudah membuat tabel `products` di database lewat migration.
Tabel itu seperti **map kosong** di lemari arsip, siap diisi data produk.

Tapi Laravel **belum tahu** bagaimana cara berbicara dengan map itu.
Kita butuh seorang "petugas" yang tahu cara membuka map produk, menulis data baru,
mengubah data yang ada, dan membuang data yang sudah tidak dipakai.

Petugas itulah yang disebut **Model**.

### Analogi: Lemari Arsip

| Database       | Lemari arsip besar                  | Tempat menyimpan semuanya                      |
| Tabel `products` | Map khusus berisi data produk      | Map khusus untuk satu jenis data (produk)      |
| **Model `Product`** | **Petugas arsip**              | Orang yang tahu cara ambil, simpan, ubah, hapus data di map produk |

Jadi di tahap ini kita **tidak akan** membuat:
- Halaman tampilan (view).
- Tombol tambah / edit / hapus.
- Form input produk.
- Route atau controller.

Kita **hanya** membuat seorang "petugas arsip" yang nanti akan kita perintahkan
melakukan tugas-tugasnya (CRUD) di tahap-tahap berikutnya.

---

## 2. Apa Itu Model?

### Pengertian sederhana

**Model** adalah bagian Laravel yang **mewakili satu jenis data** di database.
Model adalah "perwakilan" tabel database di dalam kode Laravel.

Dalam studi kasus kita:

| Database               | Laravel (kode)        |
| ---------------------- | --------------------- |
| Tabel bernama `products` | Model bernama `Product` |

### Aturan penamaan Laravel

Laravel punya **konvensi penamaan** yang sangat membantu:

| Aturan                          | Contoh                          |
| ------------------------------- | ------------------------------- |
| Nama tabel = **jamak (plural)** | `products`, `users`, `orders`   |
| Nama model = **tunggal (singular)** | `Product`, `User`, `Order` |

Dengan aturan ini, Laravel **otomatis tahu** bahwa model `Product` berhubungan
dengan tabel `products`. Kita tidak perlu menulis pengaturan tambahan.

### Analogi: Satu Produk vs Banyak Produk

- Di database, tabel berisi **banyak produk** -> `products` (jamak).
- Di kode Laravel, kita berpikir dalam **satu jenis benda** -> `Product` (tunggal), seperti cetakan biru untuk satu produk.

---

## 3. Kenapa Kita Butuh Model?

### Tanpa Model (cara manual)

Jika Laravel tidak punya Model, kita harus menulis kode SQL manual setiap kali
mau berkomunikasi dengan database, contoh:

```php
// Cara manual (tidak dipakai di Laravel)
SELECT * FROM products;
INSERT INTO products (name, price) VALUES ('Kaos Hitam', 100000);
```

Ini membuat kode panjang, rawan salah, dan sulit dibaca.

### Dengan Model (cara Laravel)

Laravel punya fitur yang disebut **Eloquent ORM**. Eloquent membuat kode komunikasi
dengan database jadi sangat **mirip bahasa manusia**, contoh:

```php
// Cara Laravel dengan Model
Product::all();                          // Ambil semua produk
Product::find(1);                        // Ambil produk dengan ID 1
Product::create(['name' => 'Kaos Hitam', 'price' => 100000]); // Tambah produk baru
```

### Perbandingan cara berpikir

| Tanpa Model                                   | Dengan Model                      |
| --------------------------------------------- | --------------------------------- |
| "Ambil data dari tabel products di database" | "**Ambil semua Product**"          |
| "Masukkan satu baris ke tabel products"       | "**Buat Product baru**"           |
| "Ubah baris dengan id 5 di tabel products"    | "**Update Product id 5**"         |

Kode dengan Model lebih **singkat, jelas, dan mudah dibaca**. Itulah kekuatan Model.

> Catatan: Pada tahap ini kita belum benar-benar memakai perintah seperti
> `Product::all()`. Itu akan dilakukan di tahap selanjutnya bersama Controller.
> Sekarang kita hanya **membuat** Model-nya terlebih dahulu.

---

## 4. Membuat Model Product

### Perintah terminal

Pastikan kamu berada **di dalam folder project** (`toko-produk`), lalu jalankan:

```bash
php artisan make:model Product
```

### Arti perintah tersebut

| Bagian            | Arti                                                      |
| ----------------- | --------------------------------------------------------- |
| `php`             | Menjalankan PHP                                           |
| `artisan`         | Alat bantu perintah bawaan Laravel                        |
| `make:model`      | Perintah untuk membuat file Model baru                    |
| `Product`         | Nama model, **harus singular** dan **huruf kapital di awal** |

### Hasilnya

Laravel membuat file baru di:

```
app/Models/Product.php
```

Folder `app/Models/` adalah tempat khusus untuk semua model di Laravel.

> Catatan penamaan:
> - File: `Product.php` (P kapital, singular).
> - Lokasi: `app/Models/`.
> - Laravel otomatis tahu model ini berhubungan dengan tabel `products`
>   karena aturan singular/plural.

---

## 5. Struktur File Model Product

Buka file `app/Models/Product.php`. Isi default yang dibuat Laravel adalah:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    //
}
```

### Penjelasan tiap bagian

#### `namespace App\Models;`

Memberitahu Laravel bahwa file ini berada di "folder virtual" `App\Models`.
Anggap saja seperti alamat: model ini tinggal di `App\Models`.

#### `use Illuminate\Database\Eloquent\Model;`

Meminjam kelas `Model` dari Laravel (dari Eloquent ORM).
Seperti meminjam "cetak biru petugas arsip" yang sudah disiapkan Laravel.

#### `class Product extends Model`

Artinya: "Saya membuat kelas bernama `Product` yang **mewarisi** semua kemampuan
dari `Model` bawaan Laravel."

Karena `extends Model`, kelas `Product` otomatis punya kemampuan:
- Mengambil data dari tabel terkait.
- Menyimpan data baru.
- Mengubah data.
- Menghapus data.
- Dan masih banyak lagi.

#### `//` ( komentar kosong)

Tempat kita bisa menulis pengaturan tambahan jika dibutuhkan nanti.

### Analogi: Petugas Arsip Baru

Bayangkan `class Product` ini sebagai **petugas arsip baru** yang baru saja
dipekerjakan. Saat ini ia **sudah dilatih** (extends Model), sudah tahu map mana
yang harus dia urus (otomatis: `products`), dan **siap diperintah**.

Tapi belum ada yang menyuruhnya melakukan apa pun. Di tahap selanjutnya, kita
akan membuat **controller** (atasan) yang memerintahkan petugas ini.

---

## 6. Kode Model Product (Versi Lengkap)

Untuk saat ini, kode default sudah cukup. Tapi kita bisa menambahkan sedikit
pengaturan agar jelas dan aman.

Ganti isi file `app/Models/Product.php` menjadi:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    protected $table = 'products';

    protected $fillable = [
        'name',
        'description',
        'price',
        'stock',
    ];
}
```

### Penjelasan tambahan

#### `protected $table = 'products';`

Memberitahu Laravel secara eksplisit: "Model ini berhubungan dengan tabel `products`."

Sebenarnya Laravel sudah bisa menebak ini secara otomatis (aturan plural),
tapi menuliskannya secara eksplisit membuat kode lebih **jelas** dan aman
jika suatu saat aturan penamaan tidak terpenuhi.

#### `protected $fillable = [...]`

Daftar kolom yang **boleh diisi** melalui kode Laravel (mass assignment).
Ini fitur keamanan Laravel bernama **mass assignment protection**.

Dalam kasus kita, kita izinkan kolom:
- `name`
- `description`
- `price`
- `stock`

Kolom `id`, `created_at`, dan `updated_at` **tidak perlu** dimasukkan ke `$fillable`
karena:
- `id` diisi otomatis oleh database.
- `created_at` dan `updated_at` diisi otomatis oleh Laravel.

### Analogi: Formulir Resmi

`$fillable` itu seperti daftar "isian yang boleh diisi langsung oleh pengguna"
di sebuah formulir resmi. Kolom-kolom tertentu (seperti nomor induk, timestamp)
hanya boleh diisi oleh sistem, bukan oleh pengguna.

---

## 7. Apa Saja yang Bisa Dilakukan Model (Sekilas)

> Catatan: Bagian ini hanya pengenalan. Kita **belum akan mempraktikkannya**
> di tahap ini. Tujuannya hanya supaya kamu tahu gambaran kekuatan Model.

Model Eloquent menyediakan banyak method siap pakai, contoh:

| Method                         | Fungsi                                      |
| ------------------------------ | ------------------------------------------- |
| `Product::all()`               | Ambil semua produk                          |
| `Product::find($id)`           | Ambil satu produk berdasarkan ID            |
| `Product::create([...])`       | Buat produk baru                            |
| `Product::where('stock', '>', 0)->get()` | Ambil produk yang stoknya lebih dari 0 |
| `$product->save()`             | Simpan perubahan pada produk                |
| `$product->delete()`           | Hapus produk                                |

Semua ini sudah berfungsi **secara otomatis** hanya karena model `Product`
`extends Model`. Kita tidak perlu menulis kode SQL sama sekali.

### Cara kerja di balik layar

```
Kode Laravel        ->    Model Product    ->    Tabel products (database)
Product::all()            (penerjemah)           "SELECT * FROM products"
```

Model bertindak sebagai **penerjemah** antara kode PHP Laravel dan database.
Kita bicara dalam bahasa Laravel, Model menerjemahkan ke bahasa SQL.

---

## 8. Menguji Model (Opsional, via Tinker)

> Bagian ini opsional. Hanya untuk memastikan model sudah benar.
> **Belum membuat fitur CRUD sungguhan.**

Laravel punya alat interaktif bernama **Tinker** - semacam "konsol" untuk mencoba kode PHP langsung.

### Jalankan Tinker

Di terminal (dalam folder project), ketik:

```bash
php artisan tinker
```

Kamu akan masuk ke prompt interaktif `>>>`.

### Coba beberapa perintah

Di dalam Tinker, ketik:

```php
>>> Product::all();
```

Artinya: "Tampilkan semua produk." Karena tabel `products` masih kosong,
hasilnya adalah koleksi kosong seperti `Illuminate\Database\Eloquent\Collection {#... }`.

```php
>>> $produk = new Product;
>>> $produk->name = 'Kaos Hitam';
>>> $produk->price = 100000;
>>> $produk->stock = 10;
>>> $produk->save();
```

Artinya: buat objek produk baru, isi atributnya, lalu simpan ke database.
Setelah `save()`, periksa tabel `products` di phpMyAdmin - akan ada satu baris baru!

```php
>>> Product::all();
```

Sekarang hasilnya akan menampilkan produk yang baru saja disimpan.

### Keluar dari Tinker

```php
>>> exit
```

Atau tekan `Ctrl + C`.

> Catatan: Tinker hanya untuk uji coba cepat. Ini **bukan** cara kita akan
> menambah produk dalam aplikasi sungguhan. Nanti, penambahan produk dilakukan
> lewat Controller dan View (form). Tapi ini sudah membuktikan model bekerja!

---

## 9. Ringkasan Alur Tahap Ini

```
1. php artisan make:model Product
        |
        v
2. Buka file app/Models/Product.php
        |
        v
3. Tambahkan $table = 'products' dan $fillable
        |
        v
4. (Opsional) Tes via php artisan tinker
        |
        v
5. Model Product siap diperintah (CRUD) di tahap selanjutnya
```

---

## 10. Checklist Keberhasilan Tahap Ini

Centang (`x`) jika sudah selesai:

- [ ] Saya paham bahwa Model adalah penghubung antara Laravel dan tabel di database.
- [ ] Saya paham aturan penamaan: tabel plural (`products`), model singular (`Product`).
- [ ] Saya sudah membuat model `Product` dengan `php artisan make:model Product`.
- [ ] File `app/Models/Product.php` berisi class `Product` yang `extends Model`.
- [ ] Saya sudah menambahkan `$table = 'products'` dan `$fillable`.
- [ ] (Opsional) Saya sudah mencoba `Product::all()` lewat Tinker.

Jika semua sudah tercentang, model `Product` sudah siap.

---

## 11. Penutup

Selamat! Kamu sudah:

- Membuat **Model pertama** (`Product`).
- Memahami konsep **singular/plural** di Laravel.
- Memahami apa itu `$table` dan `$fillable`.
- (Opsional) Mencoba lewat Tinker bahwa model bekerja.

Sekarang, Laravel sudah punya "petugas arsip" yang siap mengelola data produk.
Tapi belum ada atasan (Controller) yang memerintah, belum ada pintu masuk (Route),
dan belum ada halaman (View) untuk berinteraksi dengan pengguna.

Di **tahap berikutnya**, kita akan belajar tentang **Route** - yaitu cara
menghubungkan URL (alamat web) dengan aksi tertentu di aplikasi kita.

### Jika sudah paham tahap ini, ketik:

```
lanjut
```

Jangan lanjut sebelum kamu benar-benar siap. Pelan-pelan, asal paham.
