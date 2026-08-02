# Tahap 1 — Apa Itu Error Handling Form dan Kenapa Pesan Error Harus Jelas untuk User?

> Fokus: memahami bagaimana aplikasi membantu user memperbaiki isian form product yang salah.

## Bayangkan kamu mengisi formulir di loket

Kamu datang ke loket untuk mendaftarkan sebuah barang. Petugas memberi formulir yang berisi nama barang, harga, jumlah stok, kategori, dan foto barang.

Setelah formulir diserahkan, petugas hanya berkata, **"Ada yang salah"**, lalu mengembalikan formulir tanpa menunjukkan bagian yang salah.

Kamu tentu bingung:

- Bagian mana yang belum diisi?
- Apakah harga yang salah?
- Apakah kategori harus dipilih?
- Apakah foto yang saya pilih tidak boleh dipakai?

Agar membantu, petugas seharusnya memberi penjelasan dekat dengan kolomnya, misalnya:

- Di bawah **Nama product**: *Nama produk wajib diisi.*
- Di bawah **Harga**: *Harga tidak boleh kurang dari 0.*
- Di bawah **Stok**: *Stok wajib diisi.*
- Di bawah **Kategori**: *Kategori wajib dipilih.*
- Di bawah **Gambar product**: *File harus berupa gambar.*

Inilah tujuan error handling form pada aplikasi Laravel.

## Apa itu form product?

**Form** adalah halaman yang dipakai user untuk mengirim data ke aplikasi. Pada CRUD Product, form biasanya dipakai saat membuat atau mengubah product.

Contoh field yang diisi user:

| Field | Isi yang diharapkan |
| --- | --- |
| Nama product | Contoh: Kopi Arabika |
| Harga | Contoh: 25000 |
| Stok | Contoh: 10 |
| Deskripsi | Penjelasan singkat tentang product |
| Kategori | Misalnya: Minuman |
| Gambar product | File foto product |

Sebelum data disimpan ke database, Laravel memeriksa apakah isi form sudah memenuhi aturan yang dibuat programmer. Pemeriksaan ini disebut **validasi**.

## Apa itu error handling form?

**Error handling form** adalah cara aplikasi menangani kesalahan isian form dan menjelaskan kepada user apa yang perlu diperbaiki.

Alurnya sederhana:

1. User mengisi form product dan menekan tombol **Save product**.
2. Laravel memeriksa aturan validasi.
3. Jika ada data yang tidak sesuai, product **tidak disimpan**.
4. Laravel kembali ke halaman form.
5. Halaman form menampilkan pesan error pada field yang bermasalah.
6. User memperbaiki isian, lalu mengirim form lagi.

Jadi, error handling bukan berarti aplikasi "rusak". Ini justru cara aplikasi menjaga data tetap benar dan membantu user melanjutkan pekerjaannya.

## Mengapa pesan error harus jelas dan rapi?

Pesan error yang baik menjawab dua pertanyaan penting:

1. **Apa yang salah?**
2. **Apa yang harus saya perbaiki?**

Bandingkan dua pesan berikut:

| Pesan kurang membantu | Pesan yang jelas |
| --- | --- |
| Input invalid | Nama produk wajib diisi |
| Data salah | Harga tidak boleh kurang dari 0 |
| Upload gagal | File harus berupa gambar |
| Error | Ukuran gambar terlalu besar |

Pesan di kolom kanan lebih mudah dipahami karena langsung menyebut masalahnya. User tidak perlu menebak-nebak.

Pesan juga perlu diletakkan dekat field yang salah. Jika error gambar muncul jauh di bagian atas halaman, user mungkin tidak sadar bahwa file gambarnya perlu diganti.

## Masalah jika pesan error tidak ada atau sulit dipahami

Jika aplikasi hanya menolak form tanpa pesan yang jelas, user dapat mengalami beberapa masalah:

- Mengisi ulang semua field walaupun hanya satu field yang salah.
- Mengirim form berkali-kali dengan kesalahan yang sama.
- Mengira tombol **Save product** tidak berfungsi.
- Bingung membedakan harga negatif, stok kosong, kategori belum dipilih, atau gambar tidak valid.
- Meninggalkan halaman karena aplikasi terasa sulit digunakan.

Dalam CRUD Product, misalnya user mengisi harga `-5000`. Pesan **"Terjadi kesalahan"** tidak memberi petunjuk. Pesan **"Harga tidak boleh kurang dari 0"** langsung memberi solusi: ganti harga menjadi `0` atau angka yang lebih besar.

## Apa itu UX validation?

**UX** adalah singkatan dari *user experience*, yaitu pengalaman user saat memakai aplikasi.

**UX validation** berarti membuat proses validasi terasa mudah dimengerti oleh user. Bukan hanya memeriksa data benar atau salah, tetapi juga membantu user memperbaikinya dengan cepat.

Pada form product, UX validation yang baik berarti:

- Pesan memakai bahasa sederhana.
- Pesan menjelaskan kesalahan secara spesifik.
- Pesan muncul dekat dengan field yang bermasalah.
- Data lain yang sudah benar tidak membuat user bingung atau hilang.
- User tahu langkah berikutnya setelah membaca pesan.

Contohnya, saat gambar yang diunggah terlalu besar, pesan **"Ukuran gambar terlalu besar"** lebih ramah daripada istilah teknis yang sulit dipahami.

## Hubungannya dengan materi flash message

Pada materi 14, kita memakai **flash message** untuk memberi kabar setelah tindakan berhasil, misalnya **"Data berhasil disimpan"**.

Materi ini berbeda, tetapi saling berhubungan:

- **Flash message success** memberi tahu bahwa proses CRUD sudah berhasil.
- **Error handling form** memberi tahu bagian mana yang harus diperbaiki ketika validasi gagal.

Untuk error validasi, pesannya sebaiknya tampil di dekat masing-masing field form, bukan hanya sebagai pesan umum di atas halaman.

Di Laravel 13+, data error validasi dapat diakses di Blade melalui variabel `$errors`. Pada langkah berikutnya, kita akan melihatnya pelan-pelan, tanpa langsung membuat semua kode form sekaligus.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: menampilkan `$errors` di Blade?**
