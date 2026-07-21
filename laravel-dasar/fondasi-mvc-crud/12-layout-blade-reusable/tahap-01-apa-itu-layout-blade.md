# Tahap 1 — Apa Itu Layout Blade Reusable?

> Mentor: Laravel Dasar — Fondasi MVC, CRUD
> Topik: 12. Layout Blade Reusable
> Fokus tahap ini: **memahami masalah duplikasi header/footer**, belum menulis kode layout.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa menjawab pertanyaan berikut dengan kata-kata sendiri:

1. Apa itu **layout** di halaman website?
2. Kenapa menulis **header dan footer berulang** di setiap file Blade menjadi masalah?
3. Apa **manfaat** membuat layout Blade yang reusable (bisa dipakai ulang)?
4. Apa itu **template composition** dalam bahasa sederhana?

Kita **belum menulis kode** di tahap ini. Kita membangun pemahaman dulu, seperti memahami peta sebelum naik gunung.

---

## 2. Analogi Sehari-hari: Buku Catatan Sekolah

Bayangkan kamu punya **5 buku catatan** untuk 5 mata pelajaran:
Matematika, IPA, Bahasa, Sejarah, dan Seni.

Di setiap halaman buku, kamu selalu menulis hal yang sama di bagian atas:

```
Nama: Andi
Kelas: 6B
Tanggal: _______
```

Lalu di bagian bawah halaman, kamu juga selalu menulis:

```
Catatan guru: ___________________
Tanda tangan ortu: ______________
```

Suatu hari kamu pindah kelas, dari **6B ke 7A**.

Kamu harus mengubah tulisan "Kelas: 6B" menjadi "Kelas: 7A" di...

- buku Matematika
- buku IPA
- buku Bahasa
- buku Sejarah
- buku Seni

**Satu per satu. Halaman per halaman.** Capek, ya? 😅

> 📝 **Pesan mentor:**
> Inilah masalah yang sama yang terjadi di halaman website jika kita menulis header dan footer berulang di setiap file Blade.

---

## 3. Apa Itu Layout di Halaman Website?

Saat kamu membuka sebuah website, misalnya **toko online**, perhatikan layar kamu.

Bagian atas layar biasanya selalu ada:

- logo toko
- menu (Beranda, Produk, Keranjang, Kontak)
- tombol login

Itu disebut **header**.

Bagian samping (kadang di kiri atau kanan) biasanya ada:

- menu dashboard
- daftar kategori
- ringkasan statistik

Itu disebut **sidebar**.

Bagian bawah layar biasanya ada:

- alamat toko
- link media sosial
- copyright © 2026

Itu disebut **footer**.

Dan di **tengah-tengah** layar, ada **isi halaman** yang berbeda-beda tergantung halaman yang sedang dibuka. Itu disebut **konten utama**.

### Definisi sederhana: Layout

> **Layout** adalah **kerangka tetap** sebuah halaman website.
> Isinya: header, navbar, sidebar, footer, dan sebuah "lubang" untuk konten utama yang berubah-ubah tiap halaman.

**Analogi yang lebih dekat:**

Bayangkan sebuah **pigura foto** (picture frame).

- pigura kayunya = header, footer, sidebar (tetap, tidak berubah)
- kaca di tengah = "lubang" tempat foto ditukar-tukar
- foto yang bisa diganti = konten utama setiap halaman

Layout = pigura. Konten = foto yang ditukar.

Layout **membungkus** konten. Konten **berubah** sesuai halaman, tapi layout **tetap**.

---

## 4. Masalah: Header dan Footer yang Duplikat

Sekarang mari kita lihat **masalah nyata** di proyek Laravel kamu.

Kamu sudah punya beberapa halaman Blade untuk CRUD Produk:

1. `produk/index.blade.php` → daftar produk
2. `produk/create.blade.php` → tambah produk
3. `produk/edit.blade.php` → edit produk
4. `produk/show.blade.php` → detail produk
5. `dashboard/index.blade.php` → dashboard admin

Tanpa layout, setiap file Blade itu menulis **sendiri-sendiri** tag HTML berikut:

```blade
<!DOCTYPE html>
<html>
<head>
    <title>...</title>
</head>
<body>

    <!-- ===== HEADER ===== -->
    <header>
        <h1>Toko Bukhari</h1>
        <nav>
            <a href="/">Beranda</a>
            <a href="/produk">Produk</a>
            <a href="/dashboard">Dashboard</a>
        </nav>
    </header>

    <!-- ===== KONTEN UTAMA (berubah tiap halaman) ===== -->
    <!-- di sini setiap halaman menulis isinya sendiri -->

    <!-- ===== FOOTER ===== -->
    <footer>
        <p>© 2026 Toko Bukhari</p>
    </footer>

</body>
</html>
```

### Pertanyaan refleksi:

Coba lihat baik-baik. Bagian mana yang **sama persis** di kelima halaman?

- ✅ `<!DOCTYPE html>` → sama
- ✅ `<head> ... </head>` → sama
- ✅ `<header> ... </header>` → sama
- ✅ `<footer> ... </footer>` → sama
- ❌ konten utama di tengah → **berbeda-beda**

Artinya, **sekitar 80% kode HTML di setiap halaman adalah duplikasi** dari halaman lain. Hanya 20% yang unik (konten tengah).

> 🪤 **Jebakan pemula:**
> "Ah, tinggal copy-paste saja dari halaman sebelumnya, cepat kok."
> Benar untuk 2-3 halaman. Tapi bencana saat halaman sudah 10, 20, 50 buah.

---

## 5. Kenapa Duplikasi Ini Masalah Besar?

Pertanyaan bagus. Mari kita bahas **tiga alasan** kenapa ini masalah nyata, bukan sekadar "kurang rapi".

### Alasan 1: Sulit Dirawat (Maintainability)

Suatu hari kamu ingin mengubah logo di header dari teks jadi gambar.

Tanpa layout → kamu harus buka **5 file Blade**, ubah satu per satu:

1. buka `produk/index.blade.php` → ubah
2. buka `produk/create.blade.php` → ubah
3. buka `produk/edit.blade.php` → ubah
4. buka `produk/show.blade.php` → ubah
5. buka `dashboard/index.blade.php` → ubah

Lupa ubah satu file saja → tampilan halaman itu jadi **tidak konsisten** dengan yang lain.

> 📝 **Pesan mentor:**
> Setiap duplikasi adalah **calon bug**. Saat kamu mengubah satu tempat tapi lupa di tempat lain, muncullah ketidak-konsistenan yang sulit dilacak.

### Alasan 2: Borong Tempat (Bloat)

File Blade jadi panjang karena penuh kode yang sama.

Padahal yang **benar-benar penting** di file `produk/index.blade.php` adalah **daftar produk**-nya, bukan tag `<header>` yang sama dengan halaman lain.

Akibatnya, saat membuka file, kamu harus **scroll jauh** untuk menemukan bagian yang ingin kamu ubah. Fokus terganggu.

### Alasan 3: Sulit Diperbarui Secara Konsisten (DRY)

Ada prinsip terkenal dalam pemrograman:

> **DRY — Don't Repeat Yourself** (Jangan ulangi dirimu)

Setiap pengetahuan (dalam hal ini: struktur header, footer) harus punya **satu sumber kebenaran** (single source of truth). Jika ada di satu tempat, kita ubah sekali → berubah di semua halaman.

Jika ada di banyak tempat (duplikat), kita harus **mengingat** untuk mengubah semuanya. Manusia pelupa. Bug pun muncul.

---

## 6. Solusi: Layout Blade Reusable

Laravel punya fitur khusus untuk masalah ini, namanya **Blade Layout**.

Idea-nya sederhana:

1. Kita bikin **satu file** berisi kerangka tetap (header, footer, sidebar).
2. Di file itu, kita buat **satu "lubang"** bernama `@yield('konten')`.
3. Setiap halaman (daftar produk, tambah produk, dll.) **cukup** mengisi lubang itu dengan kontennya sendiri.

### Analogi akhir sebelum lanjut ke kode

Bayangkan kamu punya **satu stempel kop surat** (header + footer).

- Setiap kali kamu menulis surat baru, kamu **tidak perlu** menggambar kop dari nol.
- Kamu cukup menempel kop itu di atas kertas, lalu menulis isi surat di tengahnya.
- Jika kop berubah (misalnya alamat kantor pindah), kamu cukup **mengukir ulang satu stempel**, dan **semua surat** otomatis pakai kop baru.

> Stempel = **layout**
> Isi surat = **konten utama**
> Mengukir ulang satu stempel = **mengubah satu file layout**

Itulah inti dari layout Blade reusable.

---

## 7. Apa Itu Template Composition?

Istilah keren untuk konsep di atas adalah **template composition** (komposisi template).

Dalam bahasa sederhana:

> **Template composition** = **menyusun** sebuah halaman dari **bagian-bagian kecil** yang bisa dipakai ulang, alih-alih menulis satu file raksasa dari nol.

Bayangkan menyusun rumah dari **bata-bata Lego**:

- satu bata = header
- satu bata = navbar
- satu bata = sidebar
- satu bata = footer
- satu bata = konten utama

Kamu bisa menyusun ulang bata-bata itu jadi **rumah berbeda**:
rumah kecil, rumah besar, rumah dua lantai... tanpa harus membuat bata baru tiap kali.

Dalam Blade, "bata-bata" itu punya nama-nama:

| Nama di Blade | Peran (analogi Lego) |
|---|---|
| **layout** | kerangka rumah ( pondasi + atap + tembok tetap ) |
| **component** | bata siap pakai (misal: tombol, kartu produk) |
| **partial** | potongan kecil (misal: satu form input) |
| **slot** | lubang di dalam bata yang bisa diisi apa saja |

> 📝 **Pesan mentor:**
> Kita akan bahas **satu per satu, pelan-pelan** di tahap-tahap berikutnya. **Jangan** menghafal semuanya sekarang. Yang penting di tahap ini: kamu paham bahwa ** Blade itu bisa menyusun halaman dari bagian-bagian reusable**.

---

## 8. Manfaat Membuat Layout Blade Reusable

Mari kita rangkum **lima manfaat konkret** yang akan kamu rasakan setelah pakai layout:

| # | Manfaat | Efek untuk kamu |
|---|---|---|
| 1 | **Tulis sekali, pakai banyak** | header/footer ditulis di 1 file, dipakai di 5+ halaman |
| 2 | **Ubah sekali, update semua** | ubah logo di layout → 5 halaman otomatis ikut berubah |
| 3 | **Kode lebih pendek & fokus** | file `produk/index.blade.php` hanya berisi daftar produk, tanpa header/footer |
| 4 | **Konsisten otomatis** | tidak mungkin lupa ubah satu halaman karena cuma ada 1 sumber |
| 5 | **Lebih mudah dirawat** | bug di layout cukup diperbaiki 1 tempat, tidak perlu berburu ke semua file |

Ini adalah **perubahan pola pikir besar** dari pemula ke level menengah:
dari "menulis halaman dari nol" ke "menyusun halaman dari bagian-bagian".

---

## 9. Bayangan Sebelum vs Sesudah (Tanpa Kode Dulu)

**Sebelum ada layout** (keadaan kamu sekarang):

```
produk/index.blade.php     → [head][header][DAFTAR PRODUK][footer]
produk/create.blade.php    → [head][header][FORM TAMBAH]  [footer]
produk/edit.blade.php      → [head][header][FORM EDIT]    [footer]
produk/show.blade.php      → [head][header][DETAIL PRODUK][footer]
dashboard/index.blade.php  → [head][header][DASHBOARD]    [footer]
```

Bagian `[head][header]...[footer]` **diulang 5 kali**. Merah semua. Duplikasi.

**Sesudah ada layout** (keadaan tujuan kita):

```
layout/app.blade.php       → [head][header][ @yield('konten') ][footer]
                                ↑                  ↑
                                ditulis 1x        lubang konten (kosong, menunggu diisi)

produk/index.blade.php     → @extends('layout.app')
                              @section('konten')
                                  [DAFTAR PRODUK]
                              @endsection

produk/create.blade.php    → @extends('layout.app')
                              @section('konten')
                                  [FORM TAMBAH]
                              @endsection

... dan seterusnya
```

Perhatikan:

- `[head][header]...[footer]` **hanya ditulis 1 kali** di `layout/app.blade.php`.
- Setiap halaman produk **hanya menulis konten uniknya**, lalu "meminjam" kerangka dari layout.
- Jika header berubah → ubah `layout/app.blade.php` saja → **5 halaman otomatis update**.

> 🎯 **Inti tahap ini:**
> Pahami pola ini secara konsep dulu. Kode `@extends` dan `@section` akan kita bahas **lengkap** di tahap berikutnya. Sekarang cukup pahami **gambarnya**.

---

## 10. Istilah Kunci Tahap Ini

Catat 4 istilah ini, kita akan pakai terus:

| Istilah | Arti sederhana |
|---|---|
| **Layout** | kerangka tetap halaman (header + footer + lubang konten) |
| **Reusable** | bisa dipakai ulang di banyak tempat |
| **Duplikasi** | menulis kode yang sama di banyak file (akar masalah) |
| **DRY** | Don't Repeat Yourself — prinsip "jangan ulangi dirimu" |

---

## 11. Rangkuman Tahap 1

1. **Layout** = kerangka tetap halaman website (header, sidebar, footer) + satu lubang untuk konten utama.
2. Menulis header/footer **berulang** di setiap file Blade menyebabkan **duplikasi**.
3. Duplikasi = **sulit dirawat**, **boros tempat**, dan **melanggar prinsip DRY**.
4. Solusi Laravel: **layout Blade reusable** — tulis sekali, pakai banyak.
5. Konsep kerennya: **template composition** — menyusun halaman dari bagian kecil yang reusable.
6. Blade punya 4 alat untuk ini: **layout, component, partial, slot** (akan dibahas pelan-pelan).

---

## 12. Cek Pemahaman (Jawab Sendiri Dulu)

Sebelum lanjut, coba jawab pertanyaan berikut **tanpa mengintip jawaban di atas**:

1. Apa arti kata "reusable" dalam "layout Blade reusable"?
2. Sebutkan 3 alasan kenapa duplikasi header/footer itu berbahaya.
3. Apa analogi yang kamu ingat untuk layout? (pigura foto, stempel kop, atau bata Lego?)
4. Apa kepanjangan dari DRY?
5. Dengan layout, kalau kita ingin mengubah logo di header, di **berapa file** kita harus mengubahnya?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. **Reusable** = bisa dipakai ulang (dipakai di banyak tempat).
2. Tiga alasan: (a) **sulit dirawat** karena harus ubah banyak file, (b) **borong tempat** karena kode jadi panjang, (c) **melanggar DRY** karena pengetahuan sama tersebar di banyak tempat.
3. Bebas, semuanya benar: pigura foto (layout = pigora, konten = foto), stempel kop (layout = stempel, konten = isi surat), bata Lego (layout = kerangka rumah, konten = isi rumah).
4. **D**on't **R**epeat **Y**ourself.
5. **Cukup 1 file**, yaitu file layout. Semua halaman otomatis ikut berubah.

</details>

---

## 13. Apakah Kamu Ingin Lanjut?

Pada tahap ini kita sudah **paham masalah dan konsep solusinya**, tapi **belum menyentuh kode**.

Langkah berikutnya yang natural:

> ### "Apakah kamu ingin lanjut ke langkah berikutnya: membuat file layout utama Blade?"
>
> Di tahap berikutnya kita akan:
>
> - membuat **1 file** baru: `resources/views/layout/app.blade.php`
> - memahami directive `@yield('konten')` (lubang konten)
> - melihat struktur HTML lengkap layout tersebut
> - **belum** mengubah halaman produk dulu (itu tahap setelahnya)

Jawab: **"Ya, lanjut"** untuk ke tahap 2,
atau **"Ulangi tahap 1"** kalau ada bagian yang masih perlu diperdalam.

---

> 📚 **Daftar Tahap (12. Layout Blade Reusable):**
> - ✅ Tahap 1 — Apa itu layout Blade reusable (kamu di sini)
> - ⏳ Tahap 2 — Membuat file layout utama Blade
> - ⏳ Tahap 3 — `@extends` dan `@section` untuk memakai layout
> - ⏳ Tahap 4 — Mengubah halaman daftar produk agar pakai layout
> - ⏳ Tahap 5 — Mengubah halaman tambah, edit, detail produk
> - ⏳ Tahap 6 — Mengubah halaman dashboard admin
> - ⏳ Tahap 7 — Partial: memecah navbar dan footer
> - ⏳ Tahap 8 — Component: membuat komponen kartu produk
> - ⏳ Tahap 9 — Slot: lubang yang bisa diisi apa saja
> - ⏳ Tahap 10 — Ringkasan dan best practice
