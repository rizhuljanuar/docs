# Tahap 1 — Pengenalan CRUD Data Produk

> **Studi kasus:** CRUD Data Produk untuk sebuah toko.
> **Tujuan tahap ini:** Memahami masalah, konsep CRUD, MVC, dan komponen Laravel sebelum menulis kode apa pun.

---

## 1. Masalah Nyata: Toko dengan Banyak Produk

### Bayangkan ini

Pak Budi punya toko kelontong. Tokonya menjual ratusan produk:
beras, gula, minyak, sabun, permen, buku, pulpen, dan masih banyak lagi.

Setiap hari Pak Budi harus mencatat:

- Produk apa saja yang dijual.
- Harga berapa.
- Stok masih ada atau tidak.
- Produk baru yang masuk.
- Produk lama yang sudah tidak dijual.

### Masalah jika dicatat manual (buku tulis / Excel)

| Masalah                          | Akibat                                                |
| -------------------------------- | ----------------------------------------------------- |
| Catatan tangan mudah hilang      | Data produk hilang, stok jadi kacau                   |
| Sulit mencari produk tertentu    | Lama melayani pembeli, pembeli marah                  |
| Sulit mengubah harga             | Kadang harga di buku beda dengan harga sebenarnya     |
| Tidak bisa diakses dari mana pun | Hanya Pak Budi yang tahu, tidak bisa dipantau dari rumah |
| Stok double / tertukar           | Dijual produk yang sebenarnya sudah habis             |

### Kenapa toko butuh sistem

Pak Budi butuh **aplikasi** yang bisa:

- Menyimpan data produk dengan rapi.
- Dicari dengan cepat.
- Diubah kapan saja.
- Dihapus jika tidak dipakai lagi.
- Dilihat dari mana saja (HP, laptop, tablet).

Aplikasi seperti itu dibangun dengan konsep yang disebut **CRUD**.

---

## 2. Apa Itu CRUD?

**CRUD** adalah singkatan dari empat hal dasar dalam mengelola data:

| Huruf | Nama     | Arti                                  | Contoh di Toko Produk                          |
| ----- | -------- | ------------------------------------- | ---------------------------------------------- |
| **C** | Create   | Membuat/menambah data baru            | Menambah produk baru: "Indomie Goreng, Rp 3.500" |
| **R** | Read     | Membaca/melihat data                  | Melihat daftar semua produk beserta stoknya     |
| **U** | Update   | Mengubah data yang sudah ada          | Mengubah harga Indomie dari Rp 3.500 jadi Rp 4.000 |
| **D** | Delete   | Menghapus data                        | Menghapus produk yang sudah tidak dijual        |

### Analogi sederhana

CRUD itu seperti mengelola **buku catatan**:

- **Create** = menulis halaman baru di buku.
- **Read** = membuka buku lalu membaca isinya.
- **Update** = menghapus tulisan lama pakai penghapus, lalu menulis ulang.
- **Delete** = menyobek halaman yang sudah tidak terpakai.

Hampir semua aplikasi yang kamu pakai sehari-hari (Instagram, Gojek, Tokopedia)
pada dasarnya menjalankan CRUD terhadap data mereka.

---

## 3. Kenapa Kita Memakai Laravel?

### Laravel itu apa?

**Laravel** adalah **kerangka kerja (framework)** untuk membuat aplikasi web
menggunakan bahasa pemrograman **PHP**.

### Analogi: Laravel seperti Perkakas Tukang

Bayangkan kamu ingin membangun sebuah rumah.

- Tanpa perkakas, kamu harus menebang pohon sendiri, mencampur semen sendiri,
  membuat paku sendiri. Lama sekali dan hasilnya berantakan.
- Dengan **perkakas tukang** (palu, gergaji, meteran, gerigi),
  kamu tinggal merangkai bagian-bagian rumah dengan rapi.

**Laravel adalah perkakas tukangnya.**
Kamu tidak perlu membuat semuanya dari nol. Laravel sudah menyediakan
alat-alat untuk:

- Menghubungkan ke database.
- Mengatur rute (URL halaman).
- Menampilkan halaman ke pengguna.
- Menangani form dan data.
- Menjaga keamanan aplikasi.

Tanpa Laravel, kamu harus menulis semuanya dari nol. Dengan Laravel, kamu fokus
kepada **logika bisnis** (misalnya: aturan harga produk), bukan kepada teknis di balik layar.

---

## 4. Apa Itu MVC?

**MVC** adalah cara Laravel (dan banyak framework lain) menyusun kode
agar rapi dan mudah dirawat. MVC singkatan dari **Model - View - Controller**.

### Analogi: Restoran

Bayangkan kamu ke restoran. Ada tiga peran penting di sana:

| MVC          | Peran di Restoran   | Tugasnya                                                  |
| ------------ | ------------------- | --------------------------------------------------------- |
| **Model**    | Dapur + Gudang bahan | Menyimpan bahan (data). Tahu di mana bahan disimpan.     |
| **View**     | Piring + Meja makan  | Tampilan yang dilihat pelanggan (makanan disajikan cantik) |
| **Controller** | Pelayan / Waiter    | Menerima pesanan pelanggan, ke dapur, lalu bawa makanan   |

### Alurnya

1. **Pelanggan** (User) datang dan memesan melalui **Pelayan** (Controller).
2. **Pelayan** pergi ke **Dapur** (Model) untuk mengambil pesanan dari **Gudang** (Database).
3. **Dapur** menyiapkan pesanan, lalu menyerahkan ke **Pelayan**.
4. **Pelayan** menyajikannya di **Piring** (View), lalu diletakkan di meja pelanggan.

### Dalam bahasa Laravel

1. **User** mengakses sebuah **Route** (seperti memanggil pelayan).
2. **Route** menugaskan **Controller** untuk menangani permintaan.
3. **Controller** meminta data ke **Model**.
4. **Model** mengambil data dari **Database**.
5. **Controller** mengirim data tersebut ke **View**.
6. **View** menampilkan halaman yang dilihat oleh **User**.

```
User  ->  Route  ->  Controller  ->  Model  ->  Database
                                          |
User  <-  View    <-  Controller  <-  Model
```

MVC membuat kode jadi terorganisir: data di satu tempat, tampilan di tempat lain, logika di tempat lain.

---

## 5. Gambaran Fitur CRUD Produk

Dalam studi kasus ini, kita akan membuat aplikasi yang bisa melakukan hal berikut:

| Fitur                 | Aksi CRUD           | Keterangan                                   |
| --------------------- | ------------------- | -------------------------------------------- |
| Menampilkan daftar produk | Read                | Lihat semua produk dalam bentuk tabel        |
| Menambah produk baru  | Create              | Isi form untuk menambah produk baru          |
| Melihat detail produk | Read                | Klik produk untuk melihat info lengkapnya    |
| Mengedit produk       | Update              | Ubah nama / harga / stok dari sebuah produk  |
| Menghapus produk      | Delete              | Hapus produk yang tidak dipakai lagi         |

Aplikasi ini sederhana, tetapi di sinilah pondasi semua aplikasi web dimulai.

---

## 6. Komponen Laravel yang Nanti Kita Pakai

Pada tahap ini kita hanya mengenal namanya. Kode akan dibahas bertahap di tahap berikutnya.

| Komponen      | Analogi                                | Tugasnya                                                  |
| ------------- | -------------------------------------- | --------------------------------------------------------- |
| **Migration** | Cetak biru rumah                       | Mendefinisikan bentuk tabel di database (nama kolom, tipe) |
| **Model**     | Pintu masuk ke gudang                  | Menghubungkan aplikasi dengan tabel di database           |
| **Controller**| Pelayan                                | Mengatur logika antara user, model, dan view              |
| **Route**     | Papan alamat / peta                    | Menentukan URL mana yang memanggil controller mana        |
| **View Blade**| Piring saji                            | Tampilan HTML yang dilihat pengguna (file `.blade.php`)   |

> Catatan: **Blade** adalah mesin template Laravel. Ia memungkinkan kita
> menyisipkan data dari PHP ke dalam HTML dengan mudah.

Kita akan memakai semua komponen ini secara bertahap, satu per satu,
supaya tidak bingung.

---

## 7. Tujuan Akhir Studi Kasus

Setelah menyelesaikan seri ini, kamu diharapkan mampu:

1. **Memahami alur sederhana aplikasi Laravel** dari user mengetik URL
   sampai halaman ditampilkan.
2. **Mengetahui hubungan** antara database, model, controller, route, dan view.
3. **Siap praktik** membuat project Laravel pertama kamu.

Kunci belajar Laravel: **jangan terburu-buru menulis kode sebelum paham gambar besarnya.**
Tahap ini adalah gambar besarnya.

---

## 8. Checklist Pemahaman

Centang (tulis `x`) jika kamu sudah paham:

- [ ] Saya tahu masalah yang ingin diselesaikan (mengelola data produk).
- [ ] Saya tahu arti dari CRUD (Create, Read, Update, Delete).
- [ ] Saya tahu gambaran besar MVC (Model, View, Controller).
- [ ] Saya tahu komponen Laravel yang akan digunakan (Migration, Model, Controller, Route, View Blade).
- [ ] Saya tahu hubungan antara user, route, controller, model, dan view.

Jika semua sudah tercentang, kamu siap melanjutkan.

---

## 9. Penutup

Pada tahap ini kamu belum menulis kode apa pun. Itu disengaja, karena pondasi
pemahaman lebih penting daripada mengetik kode yang tidak dimengerti.

Di **tahap berikutnya**, kita akan:

- Menyiapkan project Laravel pertama.
- Melihat struktur folder Laravel.
- Membuat database pertama untuk produk.

### Jika sudah paham tahap ini, ketik:

```
lanjut
```

Jangan lanjut ke tahap berikutnya sebelum kamu siap.
