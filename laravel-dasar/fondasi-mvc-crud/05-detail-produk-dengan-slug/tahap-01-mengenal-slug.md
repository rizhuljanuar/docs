# Tahap 1: Mengenal Halaman Detail Produk, URL, dan Slug

## Apa Itu Halaman Detail Produk?

**Halaman detail produk** menampilkan informasi lengkap untuk satu produk, misalnya nama, gambar, harga, stok, deskripsi, dan kategori.

Pada daftar produk, pengguna dapat melihat banyak produk. Saat memilih satu produk, pengguna membuka halaman khusus untuk produk tersebut.

## URL Produk dengan ID

Sebelumnya, halaman detail dapat memakai ID:

```text
/products/1
```

ID sangat berguna untuk database, tetapi angka tersebut tidak menjelaskan produk yang akan dibuka.

## Apa Itu Slug?

**Slug** adalah teks URL sederhana yang dibuat dari nama produk.

```text
Name: Sepatu Lari Pria
Slug: sepatu-lari-pria-15
URL:  /products/sepatu-lari-pria-15
```

Slug biasanya memakai huruf kecil dan tanda hubung sebagai pengganti spasi. URL ini lebih mudah dibaca oleh pengguna dan mesin pencari.

## Analogi Sederhana

ID seperti nomor toko, sedangkan slug seperti papan nama toko. Database tetap mengenali produk melalui data yang tersimpan, sementara pengguna mendapat alamat yang bermakna.

## Inti Tahap 1

> Halaman detail menampilkan satu produk. Kita akan mengganti URL detail dari `/products/{id}` menjadi `/products/{slug}` tanpa mengubah URL edit dan hapus yang tetap memakai ID.

Pada tahap berikutnya kita menyiapkan kolom `slug` pada tabel `products`.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke **Tahap 2: menambahkan kolom slug pada tabel products**?
