# Tahap 1 - Kenapa Tombol di Website Sebaiknya Dibuat Konsisten?

> Materi: 13. Komponen Button Blade
> Level: Pemula - Fondasi Laravel, MVC, CRUD
> Tujuan tahap ini: Memahami **kenapa** style tombol yang berbeda-beda itu jadi masalah, sebelum kita bikin komponennya.

---

## Analogi: Pulpen di Meja Kerja

Bayangkan kamu bekerja di kantor. Di mejamu ada **5 pulpen** untuk menulis:

- Pulpen merah untuk tanda tangan penting
- Pulpen hitam untuk catatan harian
- Pulpen biru untuk nulis memo
- Pulpen hijau untuk coret-coret
- Pulpen pink untuk stiker

Lalu setiap hari, pulpen-pulpen ini **berpindah-pindah tempat**. Hari ini pulpen merah ada di laci kiri, besok ada di laci kanan, besok lusa ada di atas lemari.

Akibatnya:

- Kamu **lupa** di mana pulpen merah
- Kamu **bingung** pulpen mana yang dipakai untuk tanda tangan
- Kamu **buang waktu** cuma untuk nyari pulpen

**Sekarang bayangkan pulpen-pulpen itu ditaruh di tempat yang sama setiap hari:**

- Pulpen merah selalu di laci kiri
- Pulpen hitam selalu di laci kanan
- Pulpen biru selalu di tempat pena

Maka kamu **langsung tahu** harus ambil pulpen mana, tanpa mikir. Hidup jadi lebih mudah.

### Hubungannya dengan tombol di website

Tombol di website itu seperti **pulpen di meja kerja**.

Kalau setiap halaman membuat tombol dengan style yang berbeda-beda, pengguna website akan **bingung**:

- "Tombol Simpan yang mana ya?"
- "Kenapa tombol Hapus di halaman A warnanya merah, tapi di halaman B warnanya abu-abu?"
- "Apakah tombol biru ini untuk Edit atau untuk Detail?"

Tapi kalau semua tombol dibuat **dengan style yang sama**, pengguna akan **langsung paham**:

- Tombol biru = aksi utama (Simpan, Tambah)
- Tombol abu-abu = aksi biasa (Batal, Kembali)
- Tombol merah = aksi berbahaya (Hapus)

Inilah yang disebut **konsistensi**, dan inilah inti dari pelajaran kita hari ini.

---

## Apa Itu Tombol dalam Website?

Sebelum bicara soal konsistensi, kita perlu paham dulu apa itu **tombol** di website.

Tombol (button) adalah **sesuatu yang bisa diklik pengguna untuk melakukan aksi**.

Contoh aksi di project CRUD Produk kita:

| Tombol       | Aksi yang Dilakukan                          |
| ------------ | -------------------------------------------- |
| Simpan       | Menyimpan produk baru ke database            |
| Edit         | Membuka form untuk mengubah data produk      |
| Hapus        | Menghapus produk dari database               |
| Batal        | Membatalkan isian form, kembali ke daftar    |
| Kembali      | Kembali ke halaman sebelumnya                |
| Detail       | Melihat detail satu produk                   |

Di HTML, tombol biasanya ditulis seperti ini:

```html
<button type="submit">Simpan Produk</button>
```

Atau sebagai link (untuk tombol yang membawa ke halaman lain):

```html
<a href="/products/1/edit">Edit Produk</a>
```

Tanpa CSS, tombol di atas akan tampil **polos** seperti default browser. Tidak berwarna, tidak ada padding, terlihat membosankan.

Itulah kenapa kita biasanya menambahkan **class CSS** seperti Bootstrap atau Tailwind, supaya tombol terlihat bagus.

---

## Masalah yang Sering Terjadi: Style Tombol Berbeda-beda

Sekarang mari kita lihat **masalah nyata** di project kita.

### Contoh di halaman `index.blade.php` (Daftar Produk)

Kamu menulis tombol Hapus seperti ini:

```html
<a href="/products/{{ $product->id }}/edit"
   class="btn btn-warning btn-sm px-3 py-1 rounded">
    Edit
</a>

<form action="/products/{{ $product->id }}" method="POST">
    @csrf
    @method('DELETE')
    <button type="submit"
            class="btn btn-danger btn-sm px-3 py-1 rounded">
        Hapus
    </button>
</form>
```

Class-nya: `btn btn-warning btn-sm px-3 py-1 rounded`

### Contoh di halaman `create.blade.php` (Tambah Produk)

Kamu menulis tombol Simpan seperti ini:

```html
<form action="/products" method="POST">
    @csrf
    <button type="submit" class="btn btn-primary">
        Simpan Produk
    </button>
    <a href="/products" class="btn btn-light">
        Batal
    </a>
</form>
```

Class-nya: `btn btn-primary` dan `btn btn-light`

### Contoh di halaman `edit.blade.php` (Edit Produk)

```html
<form action="/products/{{ $product->id }}" method="POST">
    @csrf
    @method('PUT')
    <button type="submit" class="btn btn-success px-4">
        Update Produk
    </button>
    <a href="/products" class="btn btn-secondary">
        Batal
    </a>
</form>
```

Class-nya: `btn btn-success px-4` dan `btn btn-secondary`

### Contoh di halaman `show.blade.php` (Detail Produk)

```html
<a href="/products/{{ $product->id }}/edit"
   class="btn btn-info">
    Edit Produk
</a>
<a href="/products"
   class="btn btn-outline-secondary">
    Kembali
</a>
```

Class-nya: `btn btn-info` dan `btn btn-outline-secondary`

---

## Coba Perhatikan: Ada Yang Aneh?

Mari kita kumpulkan semua class tombol di atas:

| Halaman         | Tombol         | Class CSS yang Dipakai                       |
| --------------- | -------------- | -------------------------------------------- |
| `index.blade`   | Edit           | `btn btn-warning btn-sm px-3 py-1 rounded`   |
| `index.blade`   | Hapus          | `btn btn-danger btn-sm px-3 py-1 rounded`    |
| `create.blade`  | Simpan Produk  | `btn btn-primary`                            |
| `create.blade`  | Batal          | `btn btn-light`                              |
| `edit.blade`    | Update Produk  | `btn btn-success px-4`                       |
| `edit.blade`    | Batal          | `btn btn-secondary`                          |
| `show.blade`    | Edit Produk    | `btn btn-info`                               |
| `show.blade`    | Kembali        | `btn btn-outline-secondary`                  |

**Coba jawab:**

- Tombol "Batal" di halaman `create.blade.php` pakai `btn-light`
- Tombol "Batal" di halaman `edit.blade.php` pakai `btn-secondary`

Padahal fungsinya **sama**: untuk batal dan kembali ke daftar produk.

Tapi karena ditulis di halaman berbeda, **style-nya jadi beda**.

### Dampaknya:

1. **Tampilan tidak rapi** - pengguna bingung, "Kenapa Batal di halaman ini beda dengan Batal di halaman itu?"
2. **Sulit dirawat** - kalau kamu mau ganti warna tombol Batal, kamu harus **mencari semua file** yang ada tombol Batal, lalu ganti satu per satu.
3. **Mudah typo** - siapa sangka `btn-light` dan `btn-secondary` bisa tertukar begitu saja?
4. **Boros waktu** - setiap bikin halaman baru, kamu harus **mikir lagi** class apa yang dipakai untuk tombol Simpan, tombol Hapus, dst.
5. **Tidak konsisten** - akhirnya website terlihat seperti dirakit oleh **beberapa orang yang tidak saling koordinasi**, padahal hanya kamu satu orang.

---

## Analogi Lain: Toko Sepatu dengan Etalase Berantakan

Bayangkan kamu punya **toko sepatu**.

Di etalase:

- Sepatu merah ditaruh di rak kiri (tapi **di bagian bawah**)
- Sepatu merah lain ditaruh di rak kiri (tapi **di bagian atas**)
- Sepatu merah lain lagi ditaruh di rak kanan (tapi **di depan pintu**)

Padahal semua sepatu merah itu **jenis sepatu yang sama**.

Pembeli yang masuk akan **bingung**:

- "Sepatu merah yang mana yang asli?"
- "Kenapa sepatu merah ada di mana-mana?"

Jawabannya: **karena tidak ada standar penempatan**.

### Hubungan dengan tombol di website

Tombol Batal, Edit, Hapus itu seperti **sepatu merah di toko**.

Kalau setiap halaman menempatkan tombol ini dengan style yang berbeda, pengguna akan bingung:

- "Tombol Hapus yang mana?"
- "Kenapa tombol Simpan di halaman ini warnanya biru, tapi di halaman lain warnanya hijau?"

Solusinya: **buat standar**, lalu ikuti standar itu di semua halaman.

Inilah yang akan kita lakukan dengan **Blade component** di tahap-tahap berikutnya.

---

## Visi Solusi: Bayangkan Kalau Semua Tombol Konsisten

Bayangkan kalau di semua halaman, kita bisa menulis tombol seperti ini:

```blade
<!-- Tombol Simpan -->
<x-button variant="primary">
    Simpan Produk
</x-button>

<!-- Tombol Edit -->
<x-button variant="warning">
    Edit Produk
</x-button>

<!-- Tombol Hapus -->
<x-button variant="danger">
    Hapus Produk
</x-button>

<!-- Tombol Batal -->
<x-button variant="secondary">
    Batal
</x-button>

<!-- Tombol Kembali -->
<x-button variant="secondary">
    Kembali
</x-button>

<!-- Tombol Detail -->
<x-button variant="info">
    Detail Produk
</x-button>
```

Lihat betapa **rapi dan konsisten** nya:

- Semua tombol ditulis dengan `<x-button>` (bukan `<button>` atau `<a>` dengan class acak)
- Cukup sebutkan **variant** (`primary`, `danger`, `warning`, dst.) dan style-nya otomatis sesuai
- Gak perlu lagi hafal `btn btn-primary px-3 py-1 rounded` atau `btn btn-outline-secondary`
- Cukup sebut: "Saya mau tombol biru untuk Simpan" -> pakai `variant="primary"`

### Manfaat besar dari pendekatan ini:

1. **Konsisten** - Semua tombol Simpan di semua halaman tampil sama persis, karena style-nya disimpan di **satu tempat**.
2. **Mudah dirawat** - Kalau mau ganti warna tombol Hapus dari merah ke orange, cukup ubah di **satu file**, dan semua tombol Hapus di seluruh aplikasi akan ikut berubah.
3. **Hemat kode** - Tidak perlu lagi nulis `class="btn btn-danger btn-sm px-3 py-1 rounded"` berulang-ulang.
4. **Mudah dibaca** - Kode Blade jadi lebih rapi, cukup `<x-button variant="danger">Hapus</x-button>` daripada HTML panjang.
5. **Anti typo** - Cukup tulis `variant="danger"` dan kamu yakin style-nya benar. Gak ada lagi salah ketik `btn-dnger` atau `btn-dangeer`.

Tapi sebelum kita bikin komponen itu, kita perlu paham dulu: **apa itu UI component?** dan **apa itu Blade component?** Kita akan bahas pelan-pelan di tahap berikutnya.

---

## Apa itu UI Component?

**UI component** adalah **bagian kecil dari tampilan website** yang dibuat sekali, lalu bisa **dipakai ulang** di banyak tempat.

"UI" singkatan dari **User Interface** (antarmuka pengguna), yaitu semua hal yang dilihat dan diklik oleh pengguna di layar.

Contoh UI component di website:

| Komponen   | Contoh Penggunaan                                  |
| ---------- | -------------------------------------------------- |
| Button     | Tombol Simpan, Edit, Hapus                         |
| Card       | Kartu produk, kartu artikel, kartu pengguna        |
| Navbar     | Menu atas dengan logo dan link                     |
| Alert      | Pesan sukses "Produk berhasil disimpan"            |
| Modal      | Jendela pop-up untuk konfirmasi                    |
| Input      | Form input dengan label dan validasi               |
| Badge      | Label kecil "Aktif" atau "Nonaktif"               |
| Table      | Tabel produk dengan header dan baris               |

### Analogi: Balok LEGO

Bayangkan **balok LEGO**.

Kamu punya **balok merah ukuran 2x4**. Balok ini bisa kamu pakai untuk:

- Membuat atap rumah
- Membuat pintu mobil
- Membuat ekor pesawat
- Membuat dasar jembatan

Baloknya **sama**, tapi bisa dipakai di **banyak tempat**.

Inilah prinsip **reusable** (bisa dipakai ulang).

### UI component itu seperti balok LEGO

- Dibuat **sekali** di satu tempat
- Bisa dipakai ulang di **banyak halaman**
- Kalau baloknya diubah, semua tempat yang pakai balok itu akan ikut berubah

Dalam project kita, **Button** adalah salah satu UI component yang akan kita buat.

### Kenapa Button adalah kandidat bagus untuk jadi UI component?

Karena tombol itu:

1. **Sering muncul** - hampir setiap halaman punya tombol
2. **Pola-nya mirip-mirip** - ada teks, ada warna, ada aksi
3. **Riskan tidak konsisten** - karena ditulis berulang-ulang di banyak tempat

Jadi kalau kita bikin komponen Button, kita **menyelesaikan masalah konsistensi** sekaligus di banyak halaman.

---

## Apa Itu Blade Component?

**Blade** adalah **template engine** bawaan Laravel, yaitu alat untuk menulis HTML dengan tambahan kekuatan (seperti `@if`, `@foreach`, `{{ $variable }}`, `@extends`, dll).

**Blade component** adalah fitur Blade untuk membuat **UI component** (seperti balok LEGO tadi).

Cara pakai Blade component:

```blade
<x-nama-komponen>
    Isi...
</x-nama-komponen>
```

Awalan `x-` itu **penanda** bahwa ini adalah Blade component. Nama setelah `x-` adalah **nama komponennya**.

### Contoh sederhana

Misalnya kita bikin komponen bernama `button`. Maka cara pakainya:

```blade
<x-button>
    Simpan Produk
</x-button>
```

Itu akan **diterjemahkan** oleh Blade menjadi HTML tombol yang sudah ada style-nya.

### Bagaimana cara kerjanya? (Secara konsep, tanpa kode dulu)

1. Kamu bikin **satu file** di folder khusus (kita akan bahas di tahap berikutnya).
2. Di file itu, kamu tulis **template HTML + CSS** untuk tombol.
3. Setiap kali kamu menulis `<x-button>` di halaman mana pun, Blade akan **mengambil template** itu dan menyisipkannya ke halaman.
4. Hasilnya: semua tombol di semua halaman **tampil sama**, karena sumber-nya sama.

### Analogi: Stempel Cap

Bayangkan kamu punya **stempel cap** bertuliskan "DITERIMA".

Kamu cukup menekan stempel ke kertas, dan tulisan "DITERIMA" akan muncul dengan **tinta yang sama, ukuran yang sama, style yang sama**.

- Kamu **tidak perlu** menggambar tulisan "DITERIMA" manual setiap kali
- Cukup **tekan stempel** -> hasilnya selalu sama
- Kalau tinta stempel diganti (misal dari hitam ke biru), **semua cap** yang kamu buat selanjutnya akan berwarna biru

**Blade component itu seperti stempel cap:**

- Bikin **sekali** (stempel-nya)
- Pakai **banyak kali** (tekan ke kertas)
- Ganti style di **satu tempat** -> semua pemakaian ikut berubah

---

## Apa Itu Variant?

Dalam project kita, tombol punya beberapa **jenis**:

- Tombol **Simpan** (biru) - aksi utama
- Tombol **Edit** (kuning/oranye) - aksi mengubah
- Tombol **Hapus** (merah) - aksi berbahaya
- Tombol **Batal** (abu-abu) - aksi netral

Kita sebut jenis-jenis ini sebagai **variant**.

**Variant** adalah **versi berbeda** dari komponen yang sama.

### Analogi: Varian Rasa Es Krim

Es krim strawberry ada beberapa **varian**:

- Varian **cone** (di kerucut)
- Varian **cup** (di gelas)
- Varian **sandwich** (di antara wafer)

Rasanya **sama** (strawberry), tapi **bentuk penyajiannya** beda.

### Variant di tombol kita

Semua tombol kita adalah **komponen Button yang sama**, tapi dengan variant berbeda:

| Variant     | Warna       | Digunakan untuk          | Contoh                |
| ----------- | ----------- | ------------------------ | --------------------- |
| `primary`   | Biru        | Aksi utama               | Simpan Produk         |
| `secondary` | Abu-abu     | Aksi biasa               | Batal, Kembali        |
| `success`   | Hijau       | Aksi berhasil            | Selesai, Konfirmasi   |
| `danger`    | Merah       | Aksi berbahaya           | Hapus Produk          |
| `warning`   | Kuning/oranye | Aksi perhatian         | Edit, Nonaktifkan     |

Jadi nanti saat kita menulis:

```blade
<x-button variant="primary">Simpan Produk</x-button>
```

Kita bilang ke Blade: **"Saya mau tombol, tapi versi primary (biru)."**

Dan Blade akan otomatis kasih style biru ke tombol itu.

---

## Ringkasan Tahap 1

Mari kita ingat apa yang sudah kita pelajari:

### 1. Tombol di website adalah sesuatu yang bisa diklik pengguna untuk melakukan aksi
Contoh: Simpan, Edit, Hapus, Batal, Kembali, Detail.

### 2. Masalah: Style tombol yang berbeda-beda itu bermasalah
Di project kita, setiap halaman (`index`, `create`, `edit`, `show`) menulis class CSS sendiri-sendiri untuk tombol, sehingga tampilannya jadi tidak konsisten.

### 3. Dampak masalahnya
- Tampilan tidak rapi
- Sulit dirawat
- Mudah typo
- Boros waktu
- Tidak konsisten

### 4. Solusi: Bikin Blade component `<x-button>`
Dengan komponen ini, kita cukup tulis `<x-button variant="primary">Simpan</x-button>` di mana saja, dan style-nya otomatis konsisten.

### 5. Manfaat konsistensi
- Konsisten di semua halaman
- Mudah dirawat (ubah di satu tempat, efek ke semua)
- Hemat kode
- Mudah dibaca
- Anti typo

### 6. UI component itu seperti balok LEGO
Dibuat sekali, dipakai ulang di banyak tempat.

### 7. Blade component itu seperti stempel cap
Cukup tekan, hasilnya selalu sama. Ganti di sumber, semua pemakaian ikut berubah.

### 8. Variant itu seperti varian rasa
Komponen yang sama, tapi dengan tampilan/versi berbeda (primary, secondary, danger, success, warning).

---

## Pertanyaan untuk Dicerna

Sebelum lanjut, coba jawab pertanyaan berikut di kepala (atau catat di notes kamu):

1. **Di project kamu sendiri**, berapa banyak halaman yang punya tombol? Coba sebutkan.
2. **Apakah style tombol di halaman-halaman itu sudah konsisten?** Atau ada yang berbeda?
3. **Kalau kamu harus mengganti warna tombol "Hapus" di seluruh aplikasi**, berapa banyak file yang harus kamu ubah? (Ini pertanyaan penting untuk memahami kenapa komponen itu berguna.)
4. **Dari 5 variant** (`primary`, `secondary`, `success`, `danger`, `warning`), tombol "Detail Produk" menurutmu sebaiknya pakai variant apa? Kenapa?

Jawaban untuk pertanyaan nomor 4 ini akan kita bahas di tahap berikutnya saat kita mulai bikin komponen.

---

## Apakah Kamu Ingin Lanjut?

Pada tahap ini kita sudah **memahami kenapa** tombol perlu dibuat konsisten dan **apa konsepnya**.

Belum ada kode implementasi sama sekali, karena ini masih **konsep dasar**.

> **Pertanyaan:**
>
> Apakah kamu ingin lanjut ke langkah berikutnya: **membuat Blade component `<x-button>` sederhana**?
>
> Di tahap berikutnya, kita akan:
>
> 1. Membuat folder dan file untuk komponen Button
> 2. Menulis kode dasar komponen (HTML + CSS sederhana)
> 3. Mencoba memakai `<x-button>` di salah satu halaman (misal: `create.blade.php`)
> 4. Melihat hasilnya di browser
>
> Ketik **"lanjut"** atau **"ya"** untuk lanjut ke tahap 2.
