# Tahap 1 — Apa itu Status Produk Aktif/Nonaktif & Kenapa Produk yang Belum Siap Tidak Boleh Tampil di Halaman Publik

> Materi: Status Produk Aktif/Nonaktif
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi Sehari-hari: Toko Fisik vs Gudang Persiapan

Bayangkan kamu punya **toko fisik** (toko baju misalnya) di sebuah mall.

Di toko itu ada **dua tempat**:

| Tempat | Siapa yang lihat | Apa isinya |
|---|---|---|
| **Rak Pajangan Depan** | Semua pengunjung toko (public) | Baju yang sudah siap dijual: ada harga, ada label ukuran, sudah disetrika, sudah digantung rapi |
| **Gudang Belakang** | Hanya kamu dan karyawan | Baju yang baru datang dari supplier, masih dalam kardus, belum diberi harga, belum disetrika, belum difoto |

Sekarang, pertanyaannya:

> **Apakah kamu akan menaruh baju yang masih dalam kardus, belum disetrika, dan belum ada harga langsung di rak pajangan depan?**

Jawaban waras: **TIDAK.**

Kenapa? Karena pengunjung yang masuk toko akan bingung:

- "Ini baju dijual atau tidak ya?"
- "Berapa harganya? Tidak ada label."
- "Kok ada baju kusut di rak?"
- "Mungkin saya boleh coba baju ini?" (padahal baju itu belum siap)

Pengunjung bisa **salah membeli**, **kecewa**, atau **merasa toko tidak profesional**.

Nah, di dunia website Laravel, konsepnya sama persis. Kita butuh **dua tempat**:

- **Halaman publik** (yang dilihat user) = seperti **rak pajangan depan**.
- **Database produk** (semua produk yang pernah diinput admin) = seperti **gudang belakang**.

Dan kita butuh cara untuk memisahkan: **produk mana yang sudah siap ditampilkan, dan produk mana yang masih "digudangkan".**

Caranya adalah dengan menambahkan **status aktif / nonaktif** pada setiap produk.

---

## 2. Apa Itu "Status Produk Aktif / Nonaktif"?

**Status** itu seperti **lampu tombol ON/OFF** di rumah.

Setiap produk di database kita kasih "lampu" kecil. Lampunya cuma punya dua keadaan:

| Status | Artinya | Lampu |
|---|---|---|
| **Aktif** | Produk sudah siap ditampilkan ke publik | Lampu menyala (ON) |
| **Nonaktif** | Produk masih dalam persiapan, belum siap ditampilkan | Lampu mati (OFF) |

Dalam bahasa database, kita sebut ini **state sederhana** (state = keadaan). "Sederhana" karena cuma dua pilihan: aktif atau nonaktif. Tidak ada pilihan ketiga.

Contoh konkret untuk produk **"Kopi Susu Vanilla 250ml"**:

| Field produk | Contoh isi | Keterangan |
|---|---|---|
| id | 5 | Nomor urut |
| nama | Kopi Susu Vanilla 250ml | Nama produk |
| harga | 18000 | Sudah final |
| stok | 15 | Sudah ada |
| deskripsi | Kopi susu dengan aroma vanilla... | Sudah lengkap |
| kategori | Minuman | Sudah dipilih |
| slug | kopi-susu-vanilla-250ml | URL ramah |
| gambar | kopi-vanilla.jpg | Sudah diupload |
| **is_active** | **1** (aktif) | **Bisa ditampilkan ke publik** |

Sekarang contoh produk lain, **"Tumbler Limited Edition"**, yang masih dalam persiapan:

| Field produk | Contoh isi | Keterangan |
|---|---|---|
| id | 6 | Nomor urut |
| nama | Tumbler Limited Edition | Sudah diisi |
| harga | 0 | **Belum final** |
| stok | 0 | **Belum tersedia** |
| deskripsi | (kosong) | **Belum lengkap** |
| kategori | Merchandise | Sudah dipilih |
| slug | tumbler-limited-edition | Sudah dibuat |
| gambar | (belum ada) | **Belum diupload** |
| **is_active** | **0** (nonaktif) | **Belum boleh tampil di publik** |

Perhatikan: produk ke-2 ini **sudah dibuat oleh admin**, datanya sudah masuk database. Tapi karena masih ada banyak yang belum lengkap (harga, stok, deskripsi, gambar), statusnya di-set **nonaktif** supaya tidak tampil di halaman publik.

---

## 3. Kenapa Produk yang Belum Siap Tidak Boleh Tampil di Publik?

Ini penting banget buat dipahami. Kita masuk ke kasus nyata.

### Cerita 1: Produk Tanpa Gambar Bikin Website Kelihatan Rusak
Admin sudah input nama produk **"Kaos Polos Hitam"**, tapi belum sempat upload gambarnya. Kalau produk itu langsung tampil di halaman publik, user akan melihat **kotak gambar pecah (broken image)**. Website terlihat seperti error atau belum selesai. user bisa kabur, mengira websitenya penipuan.

### Cerita 2: Harga Belum Final, User Bisa Salah Beli
Admin sudah input **"Sepatu Lari Super"**, tapi harganya masih **Rp 0** karena belum final. Kalau produk itu langsung tampil di publik, user bisa lihat: "Wah, sepatunya gratis!" Lalu user checkout dan marah-marah karena ternyata harganya bukan Rp 0. Berantakan. Risiko komplain besar.

### Cerita 3: Stok Belum Ada, User Beli Tapi Tidak Bisa Dikirim
Produk **"Tumbler Limited"** sudah dibuat, tapi barangnya belum datang dari supplier (stok 0). User beli. Admin panik: "Maaf barangnya belum ada." Reputasi toko turun.

### Cerita 4: Deskripsi Kosong Bikin User Ragu
Produk **"Headphone XYZ"** deskripsinya masih kosong. User bingung: "Ini bisa buat HP atau tidak ya? Ada mikrofon atau tidak?" Tidak beli, pindah ke toko lain.

### Cerita 5: Produk Masih Draft, Tapi User Sudah Bisa Lihat
Admin sedang **setengah input** produk baru (sudah ketik nama, belum selesai atur kategori, belum klik simpan final). Kalau tidak ada sistem status, produk "setengah jadi" itu bisa muncul di halaman publik. User melihat produk aneh dengan data ngawur. Tidak profesional.

---

## 4. Masalahnya: Kalau Semua Produk Langsung Ditampilkan

Tanpa sistem status, halaman publik kita akan jadi seperti **gudang yang dibuka pintunya ke publik**. Semua orang masuk, lihat semua barang, termasuk yang:

- Belum ada harganya
- Belum ada gambarnya
- Belum ada stoknya
- Deskripsinya masih kosong / masih draft
- Masih dalam tahap persiapan admin

Akibatnya (dari sisi user dan toko):

| Masalah | Dampak |
|---|---|
| User bingung melihat produk tidak lengkap | User kabur ke toko lain |
| User beli produk yang harganya belum final | Konplain, refund, review buruk |
| User beli produk yang stoknya 0 | Tidak bisa dikirim, komplain |
| Web terlihat tidak profesional / error | Citra toko rusak |
| Admin tidak punya ruang "persiapan" | Setiap input harus langsung sempurna, tidak fleksibel |
| SEO jelek | Google baca produk-produk kosong, ranking turun |

Singkatnya: **tanpa sistem status, website kita jadi berantakan dan user kabur.**

---

## 5. Solusinya: Pakai "State Sederhana" dengan `is_active`

**State sederhana** = keadaan dengan pilihan minimal (cuma dua: aktif / nonaktif).

Kita tambahkan **satu kolom kecil** di tabel `products`, namanya **`is_active`**.

- `is_active` itu singkatan dari "is active" = "apakah aktif?".
- Isinya cuma dua kemungkinan:
  - `1` = Ya, produk aktif, boleh tampil di publik.
  - `0` = Tidak, produk nonaktif, sembunyi dulu.

Ibarat **saklar lampu** di rumah: cuma dua posisi, ON atau OFF. Tidak rumit, tidak pusing.

Saat admin membuat produk baru:

- Kalau admin sudah lengkapi semua data (nama, harga, stok, deskripsi, gambar) → admin klik **"Simpan & Tampilkan"** → `is_active = 1`.
- Kalau admin baru mulai input, belum selesai → admin klik **"Simpan sebagai Draft"** → `is_active = 0`.

Produk dengan `is_active = 0` **tetap tersimpan di database** (tidak hilang), cuma **tidak ditampilkan** di halaman publik. Admin bisa lanjut mengeditnya kapan saja, dan ketika sudah siap, tinggal ubah statusnya jadi aktif.

---

## 6. Sketsa Sederhana Alurnya

```
[Admin Input Produk Baru]
            |
            |  Simpan
            v
   +-------------------+
   | Tabel: products   |
   | is_active = 0     |  <-- default: nonaktif (aman)
   +-------------------+
            |
            |  Admin sudah lengkapi: gambar, harga, stok, deskripsi
            |
            v
   [Admin klik: Tampilkan ke Publik]
            |
            v
   +-------------------+
   | Tabel: products   |
   | is_active = 1     |  <-- sekarang aktif
   +-------------------+
            |
            v
   [Halaman publik: produk muncul di /produk]
```

Default-nya begitu admin membuat produk baru, statusnya **nonaktif** dulu (`is_active = 0`). Ini jadi **pengaman otomatis**: produk tidak akan pernah "nyasar" tampil di publik sebelum admin sengaja mengaktifkannya.

Nanti di tahap-tahap selanjutnya, kita akan belajar bagaimana query database hanya mengambil produk yang `is_active = 1`, sehingga produk yang belum siap tetap aman tersembunyi.

---

## 7. Perbedaan dengan Soft Delete (Materi Sebelumnya)

Ini penting biar tidak bingung, karena di materi 09 kita sudah belajar soft delete.

| Aspek | Soft Delete (materi 09) | Status Aktif/Nonaktif (materi 10, sekarang) |
|---|---|---|
| Kolom yang dipakai | `deleted_at` | `is_active` |
| Kapan dipakai? | Saat admin **menghapus** produk | Saat admin **membuat/mengedit** produk |
| Tujuan | Mencegah produk benar-benar hilang dari database | Mencegah produk **belum siap** tampil di publik |
| Analogi | Tempat sampah (produk dibuang, bisa diambil lagi) | Saklar lampu (produk ada, tapi lampunya dimatikan dulu) |
| Apakah produk bisa dikembalikan? | Bisa, lewat **restore** | Bisa, tinggal ubah `is_active` jadi `1` |
| Produk muncul di halaman publik? | Tidak | Tidak (kalau `is_active = 0`) |
| Produk masih ada di database? | Ya, cuma ditandai `deleted_at` | Ya, dan tidak ditandai apa-apa, cuma `is_active = 0` |

**Intinya:**

- **Soft delete** = produk mau "dibuang" dulu, tapi tidak permanen.
- **Status aktif/nonaktif** = produknya tetap ada, mau **dikontrol kapan boleh tampil ke publik**.

Dua-duanya sering dipakai bareng di aplikasi nyata: produk yang sudah siap diaktifkan, produk yang tidak lagi dijual dinonaktifkan (bukan dihapus), dan produk yang benar-benar salah input baru di-soft-delete.

---

## Ringkasan Tahap 1

| Hal | Isi |
|---|---|
| Masalah | Produk yang belum siap (tanpa gambar, harga final, stok, deskripsi) tidak boleh tampil di publik |
| Analogi | Toko fisik: rak pajangan depan (publik) vs gudang belakang (persiapan) |
| Solusi | Tambah status aktif / nonaktif pada setiap produk |
| Konsep | State sederhana: cuma dua keadaan (aktif / nonaktif) |
| Kolom baru | `is_active` di tabel `products` (isi `1` = aktif, isi `0` = nonaktif) |
| Default produk baru | `is_active = 0` (nonaktif, aman) |
| Manfaat | Website rapi, user tidak bingung, admin punya ruang draft, data tidak hilang |
| Beda dengan soft delete | Soft delete = buang sementara; status = kontrol kapan tampil |

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: menambahkan kolom `is_active` pada tabel `products`?**

Kalau iya, tahap 2 kita akan:

1. Buat migration untuk menambah kolom `is_active` ke tabel `products`.
2. Pahami tipe data `boolean` di database (kenapa cuma 0 dan 1).
3. Set default `is_active = 0` (false) supaya produk baru otomatis nonaktif.
4. Lihat bagaimana data produk berubah setelah migration dijalankan.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
