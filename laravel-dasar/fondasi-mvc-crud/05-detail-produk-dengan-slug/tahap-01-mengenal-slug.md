# Tahap 1: Mengenal Halaman Detail Produk, URL, dan Slug

## Apa Itu Halaman Detail Produk?

**Halaman detail produk** adalah halaman yang menampilkan informasi tentang
satu produk secara lebih lengkap.

Misalnya, halaman daftar produk menampilkan banyak produk sekaligus:

- Kaos Hitam
- Sepatu Lari
- Tas Sekolah

Ketika pengguna memilih **Sepatu Lari**, website membuka halaman khusus yang
berisi nama, gambar, harga, stok, dan deskripsi Sepatu Lari. Itulah halaman
detail produk.

## Apa Itu URL Produk?

**URL produk** adalah alamat yang digunakan browser untuk membuka halaman
suatu produk.

Contohnya:

```text
/produk/1
```

Alamat tersebut berarti website diminta membuka halaman produk dengan ID `1`.

## Kenapa `/produk/1` Kurang Jelas?

Angka `1` berguna bagi database untuk mengenali produk, tetapi kurang berguna
bagi manusia.

Saat melihat `/produk/1`, kita tidak langsung tahu:

- Produk apa yang akan dibuka.
- Apakah produknya kaos, sepatu, atau tas.
- Apakah tautan tersebut sesuai dengan yang sedang dicari.

Alamat itu bekerja dengan benar, tetapi belum mudah dipahami.

## Apa Itu Slug?

**Slug** adalah bagian URL yang berasal dari nama produk dan ditulis dalam
bentuk sederhana.

Contohnya:

```text
Nama produk: Sepatu Lari Pria
Slug:        sepatu-lari-pria
URL:         /produk/sepatu-lari-pria
```

Slug biasanya memakai:

- Huruf kecil.
- Tanda hubung sebagai pengganti spasi.
- Kata-kata yang menggambarkan isi halaman.

## Kenapa Slug Lebih Mudah Dibaca?

Bandingkan dua URL berikut:

```text
/produk/1
/produk/sepatu-lari-pria
```

Pada URL pertama, kita hanya melihat angka `1`.

Pada URL kedua, kita langsung tahu bahwa halaman tersebut membahas
**Sepatu Lari Pria**. URL menjadi lebih jelas bagi pengguna dan mesin pencari.
Karena itu, URL dengan slug sering disebut **SEO-friendly**.

## Analogi Sederhana: Nomor Rumah dan Papan Nama

Bayangkan ada dua toko:

- Toko pertama hanya memiliki tanda **Nomor 1**.
- Toko kedua memiliki papan nama **Toko Sepatu Lari**.

Keduanya bisa ditemukan, tetapi papan nama lebih mudah dikenali dan diingat.

Dalam URL:

- ID seperti `1` mirip nomor toko.
- Slug seperti `sepatu-lari-pria` mirip papan nama toko.

Database tetap dapat mengenali produk, sedangkan pengguna mendapatkan alamat
yang lebih bermakna.

## Inti Tahap 1

> Halaman detail menampilkan satu produk. URL adalah alamat halamannya. Slug
> membuat alamat tersebut lebih jelas karena memakai kata yang menggambarkan
> produk, bukan hanya angka.

Pada tahap ini kita baru memahami konsepnya. Kita belum mengubah database,
route, model, controller, atau menulis kode Laravel.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke **Tahap 2: merencanakan kolom slug pada tabel
products**?

Ketik **"lanjut"** jika sudah siap.
