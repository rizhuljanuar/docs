# Tahap 1 — Apa itu Soft Delete Produk & Kenapa Produk yang Dihapus Tidak Selalu Harus Hilang Permanen

> Materi: Soft Delete Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi Sehari-hari: Tempat Sampah di Dapur

Bayangkan kamu lagi masak di dapur. Ada **kertas pembungkus** yang menurutmu sudah tidak terpakai. Kamu buang ke **tempat sampah**.

Pertanyaannya: apakah kertas itu **langsung hilang selamanya** dari muka bumi?

Jawabannya: **tidak**. Kertas itu masih ada di tempat sampah. Kamu masih bisa **mengambilnya kembali** kalau ternyata kamu butuh (misalnya ada resep di balik kertas itu yang penting).

Nah, suatu hari nanti, petugas kebersihan datang dan **membawa kantong sampah itu pergi**. Baru pada saat itu kertasnya **benar-benar hilang selamanya**.

Di kehidupan sehari-hari, kita punya **dua tahap membuang**:

| Tahap | Apa yang terjadi | Bisa dikembalikan? |
|---|---|---|
| Buang ke tempat sampah | Barang pindah dari meja, tapi masih ada di rumah | Bisa, tinggal ambil lagi |
| Petugas bawa kantong sampah | Barang benar-benar pergi selamanya | Tidak bisa lagi |

**Soft delete di Laravel itu seperti tempat sampah.** Bukan pembuangan permanen, tapi "parkir sementara" supaya data masih bisa diambil kalau ternyata dibutuhkan lagi.

---

## 2. Apa Itu "Hapus Produk" dalam CRUD?

Di materi CRUD (Create, Read, Update, Delete), huruf **D** = **Delete** = menghapus data.

Contoh di CRUD Produk:

- Admin punya daftar produk di halaman `/produk`.
- Ada tombol **"Hapus"** di samping setiap produk.
- Admin klik tombol itu → produk hilang dari daftar.

Selama ini (materi 01-08), saat kita klik **Hapus**, biasanya yang terjadi adalah:

```php
Product::find($id)->delete();
```

Dan secara default, Laravel akan menjalankan query SQL seperti ini di belakang layar:

```sql
DELETE FROM products WHERE id = 5;
```

Artinya: **baris produk dengan id=5 benar-benar dihapus dari tabel `products` di database.** Data itu pergi selamanya. Seperti kertas yang sudah dibawa petugas sampah.

---

## 3. Masalahnya: Kalau Produk Langsung Dihapus Permanen

Sekarang bayangkan kasus nyata di toko online.

### Cerita 1: Produk Stok Habis Tapi Akan Dijual Lagi
Admin melihat produk **"Kopi Susu Vanilla 250ml"** stoknya habis. Karena males lihat produk yang tidak bisa dijual, admin klik **Hapus**.

Satu bulan kemudian, supplier kirim lagi kopi itu. Ternyata **produk itu akan dijual lagi**.

Kalau kemarin admin pakai hapus permanen (`DELETE FROM`), sekarang admin harus:
- Buat produk baru dari awal.
- Ketik ulang nama, deskripsi, harga.
- Upload ulang gambar.
- Bikin slug baru.
- Atur kategori lagi.
- Rugi waktu, rugi tenaga.

Padahal data lama **sebenarnya masih berguna**, cuma sedang "tidak aktif".

### Cerita 2: Hapus Karena Salah Klik
Admin mau hapus produk A, tapi salah klik produk B. Produk B langsung hilang. Data pelanggan yang pernah beli produk B jadi **tidak punya link produk** lagi di riwayat transaksi. Berantakan.

### Cerita 3: Data Produk Dipakai di Tempat Lain
Produk yang sudah dihapus mungkin masih **dipakai di tabel lain**: riwayat penjualan, ulasan pelanggan, keranjang lama. Kalau produk dihapus permanen, data-data yang terhubung jadi **yatim piatu** (tidak punya induk lagi).

### Cerita 4: Kebutuhan Audit / Laporan
Pemilik toko suatu saat mau lihat: "Produk apa saja yang pernah kita jual tahun lalu?" Kalau produk yang stoknya habis langsung dihapus permanen, datanya **tidak ada lagi**. Laporan jadi tidak lengkap.

---

## 4. Solusinya: Soft Delete

**Soft delete** = "penghapusan lembut" / "penghapusan palsu".

Cara kerjanya sangat sederhana:

> **Kita TIDAK benar-benar menghapus baris dari tabel. Kita cuma menandai baris itu dengan tanggal penghapusan.**

Caranya: kita tambahkan satu kolom baru di tabel `products`, namanya **`deleted_at`**.

| Field produk | Contoh isi |
|---|---|
| id | 5 |
| nama | Kopi Susu Vanilla 250ml |
| harga | 18000 |
| stok | 0 |
| deskripsi | Kopi susu dengan aroma vanilla... |
| kategori | Minuman |
| slug | kopi-susu-vanilla-250ml |
| gambar | kopi-vanilla.jpg |
| **deleted_at** | **NULL** (artinya: belum dihapus) |

Saat produk masih aktif dijual, `deleted_at` isinya **NULL** (kosong).

Saat admin klik **Hapus**, kita TIDAK menjalankan `DELETE FROM`. Yang kita lakukan adalah:

```sql
UPDATE products SET deleted_at = '2026-07-18 10:30:00' WHERE id = 5;
```

Artinya: **produk masih ada di tabel, tapi sekarang sudah ditandai "dihapus tanggal 18 Juli 2026 jam 10:30".**

Kembali ke analogi tempat sampah: produknya **pindah ke tempat sampah**, tapi belum dibawa petugas. Masih bisa diambil.

---

## 5. Perbedaan Soft Delete, Restore, dan Force Delete

Tiga istilah ini akan sering kamu dengar. Hat-hati bedakan:

### a. Soft Delete (Hapus Sementara)
- Produk **tidak muncul** di halaman `/produk` biasa.
- Tapi data produk **masih ada** di database, cuma `deleted_at`-nya terisi tanggal.
- Bisa dikembalikan kapan saja.
- **Analogi**: membuang kertas ke tempat sampah.

### b. Restore (Kembalikan)
- Membawa produk yang sudah di-soft-delete **kembali ke daftar produk aktif**.
- Caranya: isi `deleted_at` jadi `NULL` lagi.
- **Analogi**: mengambil kembali kertas dari tempat sampah ke meja.

### c. Force Delete (Hapus Permanen)
- Menghapus produk **benar-benar dari database**. Selamanya. Tidak bisa dikembalikan.
- Baris hilang dari tabel `products`.
- **Analogi**: petugas sampah sudah membawa kantong sampah pergi.

Tabel ringkasannya:

| Aksi | Apa yang terjadi di database | Bisa dikembalikan? |
|---|---|---|
| **Soft Delete** | `deleted_at` diisi tanggal | Bisa (dengan restore) |
| **Restore** | `deleted_at` diisi `NULL` lagi | (ini sendiri adalah aksi mengembalikan) |
| **Force Delete** | Baris dihapus permanen (`DELETE FROM`) | Tidak bisa sama sekali |

---

## 6. Kenapa Soft Delete Itu Penting (Manfaat untuk Keamanan Data)

### a. Data Tidak Hilang Selamaya Kalau Salah Klik
Salah klik hapus? Tidak masalah. Produk masih bisa di-restore. Jantung tidak perlu copot.

### b. Bisa Kembalikan Produk yang "Sementara Tidak Dijual"
Produk stok habis? Soft-delete dulu. Kalau nanti stok datang lagi, tinggal restore. Tidak perlu input ulang dari awal.

### c. Data Transaksi Lama Tetap Utuh
Pelanggan yang dulu beli produk itu masih bisa lihat riwayatnya. Produknya "tidak aktif", tapi bukti transaksinya tetap nyambung.

### d. Bisa Audit / Laporan Historis
Pemilik toko tetap bisa tahu produk apa yang pernah dijual. Cukup query "termasuk yang sudah dihapus", dan data muncul.

### e. Tenang Pikiran
Admin tidak perlu takut menghapus data penting. Selalu ada **satu lapis pengaman**: tempat sampah.

---

## 7. Pertanyaan Kunci: Produk Mana yang Muncul di Halaman?

Ini bagian penting yang akan kita implementasikan di tahap-tahap berikutnya.

Saat kita pakai soft delete, Laravel secara otomatis tahu:

- **Query biasa** (`Product::all()`) → **HANYA** produk yang `deleted_at`-nya NULL (yang aktif).
- **Query khusus** (`Product::withTrashed()->get()`) → **SEMUA** produk, termasuk yang sudah di-soft-delete.
- **Query khusus** (`Product::onlyTrashed()->get()`) → **HANYA** produk yang sudah di-soft-delete (yang ada di "tempat sampah").

Jadi, di halaman `/produk` biasa, produk yang sudah dihapus **tidak akan muncul**. Tapi data tetap aman di database, tinggal di-restore kalau dibutuhkan.

Nanti kita akan pelajari satu per satu. Untuk sekarang, kamu cukup paham **konsep dan analoginya dulu**.

---

## Ringkasan Tahap 1

| Hal | Isi |
|---|---|
| Masalah | Hapus permanen = data hilang selamanya, tidak bisa dikembalikan |
| Analogi | Tempat sampah: buang dulu, baru nanti dibawa petugas |
| Solusi | Soft delete: tandai dengan `deleted_at`, baris tidak benar-benar dihapus |
| 3 istilah penting | Soft Delete (hapus sementara), Restore (kembalikan), Force Delete (hapus permanen) |
| Kolom baru | `deleted_at` di tabel `products` |
| Manfaat | Aman dari salah klik, bisa restore, data historis tetap utuh |
| Cara kerja Laravel | Query biasa otomatis exclude yang `deleted_at`-nya terisi |

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: menambahkan fitur SoftDeletes pada model Product?**

Kalau iya, tahap 2 kita akan:
1. Tambah kolom `deleted_at` ke tabel `products` lewat migration.
2. Pakai trait `SoftDeletes` di model `Product`.
3. Lihat bagaimana Laravel otomatis menyembunyikan produk yang sudah di-soft-delete.
4. Coba klik hapus dan lihat apa yang terjadi di database.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
