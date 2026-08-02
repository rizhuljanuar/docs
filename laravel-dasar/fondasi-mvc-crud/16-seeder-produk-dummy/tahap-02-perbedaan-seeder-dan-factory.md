# Tahap 2 — Memahami Perbedaan Seeder dan Factory di Laravel

> Fokus: membedakan tugas seeder dan factory sebelum membuat file kode.

Pada tahap 1, kita belajar bahwa database kosong membuat daftar Product, pencarian, pagination, sorting, dashboard, dan relasi kategori sulit diuji. Kita membutuhkan data dummy agar fitur-fitur tersebut memiliki isi untuk dicoba.

Sekarang mari pisahkan dua alat Laravel yang membantu membuat data dummy: **factory** dan **seeder**.

## Analogi: pabrik dan petugas pengisi toko

Bayangkan sebuah toko ingin mengisi rak dengan banyak product contoh.

Ada dua peran:

1. **Pabrik** membuat barang contoh, misalnya membuat satu headphone dengan nama, harga, stok, dan deskripsi.
2. **Petugas pengisi toko** memutuskan kapan dan berapa banyak barang dari pabrik yang dimasukkan ke rak.

Di Laravel:

| Di dunia toko | Di Laravel | Tugas |
| --- | --- | --- |
| Pabrik atau cetakan product | Factory | Menentukan bentuk data dummy satu model |
| Petugas pengisi rak | Seeder | Menjalankan proses pengisian data ke database |

Jadi:

- **Factory** menjawab: *"Satu Product dummy bentuknya seperti apa?"*
- **Seeder** menjawab: *"Kapan dan berapa banyak Product dummy dibuat?"*

## Apa itu factory?

**Factory** adalah file yang menjadi cetakan untuk membuat data dummy sebuah model.

Untuk `Product`, factory nantinya menentukan nilai contoh untuk field seperti:

| Field Product | Contoh yang dibuat factory |
| --- | --- |
| `name` | Nama product contoh |
| `price` | Harga angka acak yang masuk akal |
| `stock` | Jumlah stok contoh |
| `description` | Deskripsi contoh |
| `slug` | Teks URL yang dibuat dari nama product |
| `image` | Path gambar contoh atau nilai kosong, sesuai aturan aplikasi |
| `is_active` | Status aktif atau tidak aktif |
| `category_id` | ID Category yang terhubung |

Factory memakai `fake()` dari Laravel untuk membantu membuat data contoh yang bervariasi. Misalnya, ketika factory membuat 30 Product, kita tidak perlu menulis 30 nama, 30 harga, dan 30 stok secara manual.

Secara sederhana, hasil kerja factory bisa dibayangkan seperti ini:

```text
Product 1: Nama berbeda, harga berbeda, stok berbeda
Product 2: Nama berbeda, harga berbeda, stok berbeda
Product 3: Nama berbeda, harga berbeda, stok berbeda
...
```

Factory tidak harus langsung menyimpan data. Factory dapat menyiapkan object Product di memori terlebih dahulu, atau menyimpan data jika diminta dengan method `create()`.

## Apa itu seeder?

**Seeder** adalah file yang mengatur dan menjalankan pengisian data ke database.

Seeder cocok untuk dua jenis kebutuhan:

1. **Data tetap atau data master**, misalnya lima kategori yang memang ingin selalu ada.
2. **Data banyak yang bervariasi**, misalnya 30 Product dummy yang dibuat oleh factory.

Untuk materi ini, seeder nantinya memiliki alur seperti berikut:

```text
1. Siapkan kategori Elektronik, Pakaian, Makanan, Buku, dan Aksesoris.
2. Setelah kategori ada, minta factory membuat banyak Product dummy.
3. Simpan semua data tersebut ke database.
```

Seeder biasanya berada di folder:

```text
database/seeders
```

Saat perintah berikut dijalankan:

```bash
php artisan db:seed
```

Laravel menjalankan method `run()` pada `DatabaseSeeder`, lalu `DatabaseSeeder` dapat memanggil seeder lain secara berurutan.

Kita belum menulis atau menjalankan kode itu sekarang. Tujuannya hanya memahami alur terlebih dahulu.

## Perbandingan singkat

| Pertanyaan | Factory | Seeder |
| --- | --- | --- |
| Apa peran utamanya? | Membuat pola data dummy satu model | Mengatur proses pengisian database |
| Contoh untuk Category | Bentuk satu Category dummy | Memasukkan kategori ke tabel `categories` |
| Contoh untuk Product | Bentuk nama, harga, stok, slug, dan status satu Product | Meminta factory menyimpan 30 Product |
| Membuat banyak data? | Bisa, jika dipanggil dengan `count(...)` | Bisa, dengan meminta factory membuat banyak data |
| Menentukan urutan Category lalu Product? | Bukan tugas utama | Ya, seeder mengatur urutan ini |
| Dijalankan oleh `php artisan db:seed`? | Tidak secara langsung | Ya |

## Kapan memakai masing-masing?

### Category: data tetap

Kategori berikut sudah kita kenal dari materi relasi category dan Product:

```text
Elektronik
Pakaian
Makanan
Buku
Aksesoris
```

Karena nama-nama ini sengaja ditentukan, kita dapat memasukkannya lewat `CategorySeeder`. Factory tetap akan kita buat untuk belajar dan untuk kebutuhan data Category tambahan, tetapi daftar lima kategori utama sebaiknya jelas dan mudah dibaca di seeder.

### Product: data banyak dan bervariasi

Untuk pagination dan testing search, kita memerlukan banyak Product. Menulis 30 atau 50 `Product::create(...)` secara manual akan panjang dan membosankan.

Di sinilah `ProductFactory` berguna. Seeder cukup meminta factory membuat sejumlah data Product contoh.

Gambaran perintahnya nanti seperti ini:

```php
Product::factory()->count(30)->create();
```

Baca kode ini sebagai kalimat:

| Bagian | Arti sederhana |
| --- | --- |
| `Product::factory()` | Gunakan cetakan Product. |
| `->count(30)` | Siapkan 30 Product. |
| `->create()` | Simpan 30 Product tersebut ke database. |

Jangan salin kode itu ke aplikasi dahulu. Kita akan memastikan factory Product sudah benar sebelum memakainya di seeder.

## Bagaimana factory dan seeder bekerja bersama?

Alurnya bisa dibayangkan seperti ini:

```text
DatabaseSeeder
    ↓ memanggil
CategorySeeder
    ↓ membuat kategori tetap
Elektronik, Pakaian, Makanan, Buku, Aksesoris
    ↓ kategori sudah tersedia
ProductSeeder
    ↓ meminta
ProductFactory membuat banyak Product dummy
    ↓
Product tersimpan dan masing-masing memiliki category_id
```

Urutan ini penting karena Product memiliki `category_id`. Product tidak boleh menunjuk ke Category yang belum ada.

## Hubungannya dengan coding yang sudah ada

Factory dan seeder memakai model serta kolom yang sudah dibuat pada materi-materi sebelumnya:

- Model `Category` dan tabel `categories`.
- Model `Product` dan tabel `products`.
- Relasi `Product` ke `Category` melalui `category_id`.
- `slug` untuk URL detail Product.
- `image` untuk path gambar Product.
- `is_active` untuk status aktif atau nonaktif.
- Soft delete tetap memakai `deleted_at`, bukan `is_active`.

Data dummy tidak melewati form create dan edit, sehingga flash message serta component error dari materi 14 dan 15 tidak muncul ketika seeder berjalan. Keduanya tetap dipakai saat user mengirim form melalui browser.

## Inti tahap ini

Ingat kalimat berikut:

> **Factory membuat bentuk data dummy. Seeder mengatur kapan dan berapa banyak data itu dimasukkan ke database.**

Pada langkah berikutnya, kita akan mulai membuat `CategoryFactory`. Kita akan membuatnya pelan-pelan, lalu melihat alasan factory perlu memakai trait `HasFactory` pada model.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: membuat `CategoryFactory` untuk data kategori dummy?**
