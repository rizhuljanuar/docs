# Tahap 1 — Apa Itu Flash Message?

> Fokus: memahami mengapa user perlu diberi tahu setelah data product disimpan.

## Bayangkan kamu di kasir

Kamu membeli satu barang di minimarket. Setelah menyerahkan uang, kasir diam saja: tidak ada struk dan tidak ada ucapan bahwa pembayaran berhasil.

Kamu pasti bingung:

- Apakah pembayaran sudah diterima?
- Apakah kasir sedang memprosesnya?
- Apakah saya perlu membayar lagi?

Hal yang sama dapat terjadi pada aplikasi Laravel.

Misalnya user mengisi form **Add product**, lalu menekan tombol **Save product**. Controller menyimpan data dan browser kembali ke daftar `/products`. Jika halaman tidak menampilkan pesan apa pun, user tidak tahu apakah product benar-benar tersimpan.

## Apa itu notifikasi di website?

Notifikasi adalah pesan singkat yang memberi tahu hasil tindakan user. Pada CRUD product, contohnya:

| Tindakan | Pesan yang ditampilkan |
| --- | --- |
| Menyimpan product baru | Data berhasil disimpan |
| Mengubah data product | Data berhasil diperbarui |
| Menghapus product | Data berhasil dihapus |
| Mengembalikan product dari trash | Product berhasil dikembalikan |
| Gagal menyimpan product | Terjadi kesalahan, silakan coba lagi |

Pesan ini adalah jawaban aplikasi kepada user: tindakan tadi berhasil, gagal, atau perlu diperiksa lagi.

## Masalah jika tidak ada pesan

Tanpa notifikasi, user dapat menekan tombol **Save product** dua kali karena mengira klik pertama tidak bekerja. User juga tidak tahu apakah data masuk ke database atau terjadi masalah.

Jadi, notifikasi bukan hiasan. Notifikasi membantu user merasa yakin tentang hasil tindakannya.

## Apa itu flash message?

**Flash message** adalah notifikasi sementara yang muncul setelah aplikasi menyelesaikan suatu tindakan lalu memindahkan user ke halaman lain.

Contoh alurnya:

1. User menekan **Save product**.
2. Controller menyimpan product.
3. Controller mengarahkan browser kembali ke `/products`.
4. Halaman daftar menampilkan pesan **Data berhasil disimpan**.
5. Saat user membuka halaman lain atau memuat ulang halaman lagi, pesan itu hilang.

Disebut *flash* karena pesannya muncul sebentar untuk menarik perhatian, lalu tidak disimpan sebagai informasi tetap.

## Session flash, pelan-pelan

Laravel memakai **session** sebagai tempat sementara untuk mengingat informasi milik user antarhalaman. Session flash adalah data session khusus yang Laravel siapkan hanya untuk permintaan berikutnya.

Untuk flash message, yang diingat bukan seluruh product, melainkan pesan pendek seperti:

```text
Data berhasil disimpan
```

Setelah pesan ditampilkan pada halaman tujuan redirect, Laravel tidak perlu menyimpannya lagi. Karena itu pesan flash cocok untuk hasil aksi CRUD, bukan untuk data permanen.

Dokumentasi Laravel menjelaskan bahwa `redirect(...)->with(...)` dapat mengirim data flash bersama redirect. Pada langkah berikutnya kita akan memakai pola itu untuk pesan sukses setelah product berhasil disimpan.

> Catatan kesinambungan: materi ini melanjutkan CRUD `Product` yang sudah ada. Daftar tetap di `/products`, detail tetap memakai slug, dan flash message tidak mengubah paginator, filter, sorting, SoftDeletes, `is_active`, maupun cache dashboard.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: menambahkan flash message success setelah menyimpan produk?**
