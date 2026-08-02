# Tahap 1 — Apa Itu Data Dummy dan Kenapa Developer Membutuhkan Seeder Saat Membuat CRUD Produk?

> Fokus: memahami mengapa aplikasi butuh data contoh sebelum membahas kode seeder dan factory.

## Bayangkan kamu sedang menata toko yang masih kosong

Kamu baru membuka toko. Rak sudah ada, kasir sudah ada, papan kategori sudah ada, tetapi semua rak masih kosong. Tidak ada laptop, baju, makanan, buku, atau aksesoris.

Sekarang bayangkan kamu ingin mencoba beberapa hal:

- Apakah daftar product tampil rapi?
- Apakah pencarian bisa menemukan product bernama **Laptop**?
- Apakah pagination bekerja jika product-nya banyak?
- Apakah sorting harga dari murah ke mahal terlihat benar?
- Apakah dashboard bisa menghitung jumlah product?
- Apakah product sudah masuk ke kategori yang benar?

Sulit menjawab semua pertanyaan itu jika toko hanya berisi nol atau satu product.

Di aplikasi Laravel, toko kosong itu seperti **database kosong**. Kita membutuhkan barang contoh untuk mencoba fitur CRUD Product dengan keadaan yang lebih mirip aplikasi sungguhan.

## Apa itu data dummy?

**Data dummy** adalah data contoh yang sengaja dibuat untuk latihan, testing, atau pengembangan aplikasi.

Data dummy bukan data asli milik pelanggan. Data ini hanya dipakai agar developer dapat melihat dan mencoba hasil pekerjaannya.

Contoh data kategori dummy:

| Nama kategori |
| --- |
| Elektronik |
| Pakaian |
| Makanan |
| Buku |
| Aksesoris |

Contoh satu data product dummy:

| Field product | Contoh isi |
| --- | --- |
| Nama product | Headphone Nirkabel |
| Harga | 250000 |
| Stok | 15 |
| Deskripsi | Headphone untuk mendengarkan musik sehari-hari |
| Kategori | Elektronik |
| Slug | `headphone-nirkabel` |
| Gambar | `products/headphone-nirkabel.jpg` |
| Status aktif | Aktif |

Data itu boleh terlihat seperti product sungguhan, tetapi tujuannya hanya membantu kita mengembangkan dan menguji aplikasi.

## Mengapa developer membutuhkan data contoh?

Pada materi sebelumnya, kita sudah membuat CRUD Product, kategori, detail dengan slug, pencarian, pagination, sorting, dashboard, flash message, dan error handling form.

Fitur-fitur itu lebih mudah diuji jika data contoh cukup banyak.

| Fitur | Masalah jika data sedikit atau kosong | Manfaat data dummy |
| --- | --- | --- |
| Daftar product | Halaman hanya terlihat kosong | Kita bisa melihat susunan banyak product |
| Pencarian product | Tidak ada nama yang dapat dicari | Kita bisa mencari kata seperti `Laptop` atau `Buku` |
| Pagination product | Tidak ada halaman kedua | Kita bisa memastikan tombol halaman bekerja |
| Sorting product | Urutan terlihat sama karena data terlalu sedikit | Kita bisa membandingkan harga dan stok yang berbeda |
| Dashboard admin | Angka ringkasan selalu nol | Kita bisa melihat jumlah product dan data terbaru |
| Relasi kategori dan product | Dropdown atau daftar kategori tidak terlihat berguna | Kita bisa memastikan product tampil pada kategori yang tepat |

Contohnya, pagination yang menampilkan 10 product per halaman tidak dapat benar-benar diuji jika database hanya memiliki 3 product. Kita membutuhkan lebih dari 10 product agar halaman kedua muncul.

## Masalah jika database kosong

Database kosong tidak selalu berarti aplikasi rusak. Halaman daftar memang boleh menampilkan pesan seperti **Belum ada product**.

Namun, database kosong membuat developer sulit memastikan fitur lain bekerja dengan benar.

Misalnya:

- Pencarian selalu mengembalikan hasil kosong. Kita tidak tahu apakah fitur search salah atau memang tidak ada data.
- Sorting harga tidak terlihat berubah karena tidak ada harga untuk dibandingkan.
- Pagination tidak muncul karena jumlah product belum cukup banyak.
- Dashboard menunjukkan angka `0`, sehingga sulit membedakan dashboard yang benar dengan dashboard yang tidak mengambil data.
- Relasi category dan product tidak dapat diperiksa karena belum ada product yang terhubung ke category.

Tanpa data contoh, developer perlu mengisi banyak form Product secara manual satu per satu. Cara itu lambat, membosankan, dan mudah membuat data tidak konsisten.

## Apa itu seeder?

**Seeder** adalah file Laravel yang berisi instruksi untuk mengisi database secara otomatis.

Analogi sederhananya: seeder seperti daftar belanja dan petugas pengisi rak toko.

- Daftar belanja mengatakan kategori apa yang perlu ada, misalnya Elektronik dan Makanan.
- Petugas membaca daftar itu lalu mengisi rak toko.
- Setelah selesai, toko memiliki data untuk dicoba.

Dalam Laravel, seeder dapat mengatakan hal seperti ini:

```text
Buat kategori Elektronik, Pakaian, Makanan, Buku, dan Aksesoris.
Buat banyak product contoh yang terhubung ke kategori tersebut.
```

Saat seeder dijalankan, Laravel memasukkan data itu ke database. Jadi developer tidak perlu mengetik semua product melalui form CRUD secara manual.

Nanti kita menjalankan seeder dengan perintah:

```bash
php artisan db:seed
```

Pada tahap ini, cukup ingat artinya: **jalankan instruksi pengisian data contoh ke database**. Kita belum menjalankan perintah ini dan belum membuat file seeder.

## Lalu, apa itu factory?

Jika seeder adalah petugas yang mengisi rak berdasarkan daftar, **factory** adalah cetakan atau mesin pembuat product contoh.

Factory menentukan bentuk dasar data yang dibuat, misalnya setiap Product memiliki:

- nama product acak,
- harga acak yang masuk akal,
- stok acak,
- deskripsi contoh,
- slug yang sesuai nama,
- status aktif atau tidak aktif,
- serta kategori yang terhubung.

Seeder kemudian bisa meminta factory membuat banyak data sekaligus. Contohnya, nanti seeder dapat meminta:

```text
Buat 30 product contoh.
```

Factory yang akan membantu menentukan isi 30 product tersebut. Jadi seeder dan factory bekerja bersama, tetapi tugasnya tidak sama.

Kita akan membedakan keduanya dengan lebih jelas pada langkah berikutnya.

## Hubungan dengan materi sebelumnya

Materi 16 tidak mengganti fitur yang sudah dibuat. Data dummy hanya memberi isi untuk fitur tersebut agar mudah dicoba.

- Form create dan edit tetap memakai validasi serta `<x-error-message />` dari materi 15.
- Jika Product dibuat lewat form dan berhasil, flash message dari materi 14 tetap muncul.
- Category tetap berelasi dengan Product melalui `category_id`.
- Detail Product tetap memakai slug.
- Search, pagination, sorting, dan dashboard tetap memakai query yang sudah dibuat pada materi sebelumnya.

Seeder dan factory hanya membantu menyiapkan banyak Product dan Category contoh untuk kebutuhan pengembangan dan testing.

> Catatan penting: data dummy sebaiknya dipakai pada database lokal atau environment development. Jangan sembarangan menjalankan perintah yang mengubah data pada database production.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: memahami perbedaan seeder dan factory di Laravel?**
