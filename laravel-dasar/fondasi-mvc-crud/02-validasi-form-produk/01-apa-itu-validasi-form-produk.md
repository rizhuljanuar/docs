# Materi 2: Validasi Form Produk

> Modul: A. Level Dasar — Fondasi Laravel, MVC, CRUD
> Topik: Request Validation dengan FormRequest

---

## Bagian 1: Apa itu Validasi Form Produk dan Kenapa Penting?

### Pahami dulu dengan analogi sehari-hari

Bayangkan kamu kerja sebagai kasir di toko. Ada seseorang datang
dan ingin **mendaftarkan produk baru** ke sistem toko kamu.

Lalu dia bilang begini:

> "Mas, saya mau tambah produk. Tapi nama produknya saya kosongkan ya.
> Harganya minus sepuluh ribu. Stoknya juga kosong aja. Deskripsinya
> nggak usah diisi."

Apa yang akan kamu lakukan sebagai kasir yang waras?

Tentu kamu akan **menolak** dan bilang:

> "Maaf, gak bisa. Nama produk wajib diisi. Harga tidak boleh minus.
> Stok minimal harus diisi dulu."

Nah, **tindakan menolak dan mengecek data itu** disebut **validasi**.

Dalam aplikasi Laravel, kamu adalah **pemilik toko** (developer).
Sistem kamu adalah **kasir**-nya. Form adalah **meja pendaftaran**.
Dan **validasi form** adalah **aturan kasir** tentang data apa yang
boleh masuk ke sistem dan apa yang harus ditolak.

---

### Apa itu Validasi Form Produk?

**Validasi form produk** adalah proses **memeriksa data yang dikirim
user dari form** sebelum data itu disimpan ke database.

Contoh untuk produk kita (field `name`, `price`, `stock`, `description`
yang sudah dibuat di **Materi 1: CRUD Data Produk**):

| Field        | Aturan yang wajar                                  |
|--------------|----------------------------------------------------|
| `name`       | Wajib diisi, tidak boleh kosong                    |
| `price`      | Wajib diisi, harus angka, tidak boleh minus        |
| `stock`      | Wajib diisi, harus angka bulat (tidak boleh pecahan)|
| `description`| Boleh dikosongkan, tapi kalau diisi minimal 10 huruf|

Aturan-aturan seperti di atas itu yang disebut **rules validasi**
(aturan validasi).

---

### Kenapa Validasi Itu Penting?

Bayangkan toko kamu **tidak ada kasir** dan **tidak ada aturan**.
Semua orang bebas masuk, menaruh barang di rak, dan menulis label
sendiri. Apa yang terjadi?

1. **Ada produk tanpa nama** → bingung mau cari barang itu apa.
2. **Ada produk harga minus** → setiap barang terjual, toko malah
   rugi karena harga minus dihitung berulang.
3. **Ada stok kosong** → sistem tidak tahu apakah barang ini masih
   ada atau tidak.
4. **Ada deskripsi aneh** → pelanggan bingung dan tidak percaya.

Aplikasi tanpa validasi itu seperti toko tanpa kasir dan tanpa aturan:
**berantakan, berbahaya, dan bisa rugi**.

---

### Masalah yang Terjadi Kalau Validasi Tidak Dibuat

Berikut masalah nyata di aplikasi Laravel kamu:

#### 1. Data kosong / null tersimpan di database

```php
$produk = new Product();
$produk->name        = $request->name;        // bisa kosong
$produk->price       = $request->price;       // bisa null
$produk->stock       = $request->stock;
$produk->description = $request->description;
$produk->save();
```

Tanpa validasi, kode di atas **tetap jalan**. Database tetap menerima
`name = null` dan `price = null`. Halaman daftar produk nanti bakal
penuh dengan baris yang **nama dan harganya kosong**.

#### 2. Harga negatif membuat laporan keuangan salah

Kalau user input `price = -10000`, laporan penjualan kamu jadi kacau.
Total pendapatan jadi negatif. Ini bug serius yang susah dilacak
kalau sudah menumpuk.

#### 3. Tipe data salah bisa error saat ditampilkan

Misal user input `price = "mahal"` (string). Saat halaman ingin
menampilkan `Rp "mahal"` atau menghitung total, aplikasi bisa
**error 500** atau menampilkan tulisan aneh.

#### 4. Keamanan dan kepercayaan pengguna jatuh

Pelanggan yang melihat produk dengan nama kosong dan harga aneh
bakal **pergi dari toko kamu**. Validasi adalah bentuk perhatian
kepada pengguna: **kami menjaga kualitas data produk yang kamijual**.

---

### Jadi, Apa Itu Validasi dalam Satu Kalimat?

> Validasi adalah **penjaga gerbang** yang memastikan hanya data
> yang **masuk akal dan aman** yang boleh disimpan ke database,
> dan menolak sisanya dengan pesan error yang jelas.

Dalam Laravel, "penjaga gerbang" ini punya nama resmi:

- **Request Validation** → proses memvalidasi data dari request HTTP.
- **FormRequest** → cara Laravel untuk membuat penjaga gerbang ini
  dalam bentuk **kelas terpisah**, supaya rapi dan bisa dipakai ulang.

Kita akan bahas kedua istilah ini **pelan-pelan di bagian selanjutnya**.

---

### Ringkasan Bagian 1

- **Validasi** = mengecek data dari form sebelum disimpan.
- **Analogi**: kasir toko yang menolak data aneh dari pelanggan.
- **Tanpa validasi**: data kosong, harga minus, laporan rusak,
  pengguna kabur.
- **Di Laravel**, validasi dilakukan oleh **Request Validation**,
  dan diatur rapi lewat **FormRequest**.

---

> **Berhenti di sini.**
>
> Pada bagian berikutnya kita akan masuk ke **langkah praktis**:
> membuat aturan validasi untuk produk (nama, harga, stok, deskripsi)
> menggunakan **FormRequest** di Laravel.
>
> **Apakah kamu ingin lanjut ke langkah berikutnya: membuat aturan
> validasi produk?**
