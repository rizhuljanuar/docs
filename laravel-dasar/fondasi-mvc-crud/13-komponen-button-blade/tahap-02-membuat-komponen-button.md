# Tahap 2 — Membuat File Komponen `<x-button>` Sederhana

> Materi: 13. Komponen Button Blade
> Level: Pemula — Fondasi Laravel, MVC, CRUD
> Fokus tahap ini: **membuat file komponen Button** dan memahami strukturnya, serta mencoba memakainya di satu halaman.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Membuat folder dan file untuk **anonymous Blade component** bernama `button`.
2. Menjelaskan fungsi `@props([...])` di dalam komponen.
3. Menulis struktur dasar HTML `<button>` di dalam file komponen.
4. Memanggil komponen dengan `<x-button>` di halaman `create.blade.php`.
5. Melihat hasilnya di browser dan memahami **cara kerja** komponen secara nyata.

Belum ada variant warna di tahap ini. Itu akan dibahas di tahap 3. Kita fokus dulu ke **struktur dasar** supaya pemahaman kokoh.

---

## 2. Analogi: Mencetak Stempel "DITERIMA"

Bayangkan kamu bekerja di kantor. Setiap dokumen yang masuk harus distempel **"DITERIMA"**.

**Cara lama (tanpa stempel):**

Setiap kali ada dokumen, kamu **menulis tangan** "DITERIMA" pakai spidol.

- Kadang tulisannya **miring**
- Kadau tulisannya **kebesaran**
- Kadang lupa **cap tanggalnya**
- Tiap dokumen hasilnya **beda-beda**

**Cara baru (pakai stempel):**

Kamu bikin **stempel karet** bertuliskan "DITERIMA" + tempat untuk tanggal.

- Cukup **cap** ke dokumen → hasilnya **sama persis** setiap kali
- Kalau tinta stempel diganti (misal dari hitam ke biru), **semua cap berikutnya** akan biru

### Hubungannya dengan komponen Button

- **Cara lama** = menulis `<button class="btn btn-primary px-3 py-1 ...">` manual di setiap halaman → hasilnya **berbeda-beda**.
- **Cara baru** = bikin **komponen `<x-button>`** sekali → pakai di mana saja → hasilnya **sama persis**.

Di tahap ini, kita akan **bikin stempelnya** (file komponen), lalu **mencobanya** di satu dokumen (halaman `create.blade.php`).

---

## 3. Di Mana File Komponen Harus Ditaruh?

Laravel punya **folder khusus** untuk semua komponen Blade, yaitu:

```
resources/views/components/
```

Semua file di folder ini otomatis dikenali sebagai komponen Blade dan bisa dipanggil dengan `<x-nama-komponen>`.

### Struktur folder project kita sekarang

```
resources/views/
├── components/                    ← folder khusus komponen
│   └── kartu-produk.blade.php     ← (sudah ada dari materi 12)
├── layout/
│   └── app.blade.php
├── partials/
│   ├── navbar.blade.php
│   └── footer.blade.php
└── produk/
    ├── index.blade.php
    ├── create.blade.php
    ├── edit.blade.php
    └── show.blade.php
```

> Kita akan menambahkan **satu file baru** di folder `components/` yaitu `button.blade.php`.

---

## 4. Konvensi Penamaan File Komponen

Sebelum bikin file, pahami dulu **aturan nama**-nya. Ini penting supaya tidak error.

### Aturan: pakai **kebab-case**

- Huruf kecil semua
- Kalau lebih dari satu kata, gunakan **tanda hubung** (`-`)
- Ekstensi: `.blade.php`

| Benar                     | Salah                       | Alasan salah                      |
| ------------------------- | --------------------------- | --------------------------------- |
| `button.blade.php`        | `Button.blade.php`          | Pakai huruf besar                 |
| `kartu-produk.blade.php`  | `kartu_produk.blade.php`    | Pakai garis bawah (snake_case)    |
| `badge-status.blade.php`  | `BadgeStatus.blade.php`     | Pakai huruf besar + camelCase     |

> **Jebakan pemula:**
> Nama file HARUS **kebab-case**. Kalau kamu simpan dengan nama `Button.blade.php`, Laravel tidak akan menemukannya saat dipanggil dengan `<x-button>`.

### Hubungan nama file dengan tag `<x-...>`

| Nama file                      | Tag pemanggil           |
| ------------------------------ | ----------------------- |
| `button.blade.php`             | `<x-button>`            |
| `kartu-produk.blade.php`       | `<x-kartu-produk>`      |
| `badge-status.blade.php`       | `<x-badge-status>`      |

Aturannya: **buang ekstensi `.blade.php`**, lalu tambahkan awalan `x-`.

---

## 5. Langkah 1: Bikin File `button.blade.php`

Sekarang kita buat filenya.

### 5.1 Lokasi file

```
resources/views/components/button.blade.php
```

### 5.2 Isi file (versi paling sederhana)

```blade
@props(['variant' => 'primary'])

<button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
    {{ $slot }}
</button>
```

### 5.3 Penjelasan baris per baris

Tenang, kita bahas **pelan-pelan**. Setiap baris akan dijelaskan dengan bahasa sederhana.

---

#### Baris 1: `@props(['variant' => 'primary'])`

**Apa fungsinya?**

`@props([...])` adalah directive (perintah khusus Blade) untuk **mendeklarasikan props**, yaitu **input** yang diterima komponen.

Di sini kita mendeklarasikan **satu props** bernama `variant`, dengan **nilai default** `"primary"`.

Artinya:

- Komponen ini bisa menerima input `variant` dari pemanggil.
- Kalau pemanggil **tidak mengirim** `variant`, otomatis pakai nilai default `"primary"`.

**Analogi:**

Bayangkan stempel "DITERIMA" punya **slot untuk tanggal**. Kalau kamu tidak isi tanggalnya, stempel akan pakai **tanggal default** (misal: tanggal hari ini).

Di sini, `variant` adalah "slot" yang bisa diisi. Kalau tidak diisi, pakai default `"primary"`.

**Contoh pemakaian:**

```blade
<!-- Tidak kirim variant → pakai default "primary" -->
<x-button>Simpan Produk</x-button>

<!-- Kirim variant "danger" -->
<x-button variant="danger">Hapus Produk</x-button>
```

> **Pesan mentor:**
> `@props([...])` itu seperti **daftar input** yang diterima komponen. Tanpa deklarasi ini, komponen tidak akan kenal props `variant`, dan akan error saat dipanggil.

---

#### Baris 3: `<button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>`

Ini adalah **tag `<button>` HTML biasa**, tapi dengan dua hal khusus di dalamnya: `$attributes` dan `->merge(...)`.

##### Apa itu `$attributes`?

`$attributes` adalah **variabel otomatis** yang berisi **semua atribut tambahan** yang dikirim ke komponen saat dipanggil.

Contoh: kalau kamu panggil komponen begini:

```blade
<x-button type="submit" id="tombol-simpan">
    Simpan Produk
</x-button>
```

Maka `$attributes` akan berisi `type="submit"` dan `id="tombol-simpan"`.

##### Apa itu `->merge([...])`?

`->merge([...])` adalah cara untuk **menggabungkan** atribut dari pemanggil dengan **atribut default** yang sudah kita tentukan di komponen.

Di sini, kita menetapkan **atribut default** berupa `class`:

```php
['class' => 'btn btn-' . $variant]
```

Artinya: setiap komponen `<x-button>` akan otomatis punya `class="btn btn-{variant}"`.

- Kalau `$variant = "primary"`, maka `class="btn btn-primary"`
- Kalau `$variant = "danger"`, maka `class="btn btn-danger"`

**Analogi:**

Stempel "DITERIMA" sudah punya **teks tetap** "DITERIMA". Tapi kamu masih bisa **tambahkan cap lain** di sebelahnya (misal cap "URGENT"). `->merge([...])` menggabungkan **teks tetap stempel** dengan **cap tambahan**.

##### Kenapa pakai `->merge` dan bukan langsung `class="btn btn-{{ $variant }}"`?

Karena `->merge` **menggabungkan**, bukan **menimpa**. Artinya:

- Kalau pemanggil menambah `class="my-custom-class"`, class itu akan **digabung** dengan `btn btn-primary`, bukan menimpa.
- Hasilnya: `class="btn btn-primary my-custom-class"`.

Ini lebih fleksibel daripada hardcode `class="btn btn-{{ $variant }}"` yang akan **menimpa** class dari pemanggil.

> **Pesan mentor:**
> Untuk pemula, tidak masalah kalau belum sepenuhnya paham `->merge`. Intinya: ini cara Laravel untuk **menggabungkan** atribut default komponen dengan atribut tambahan dari pemanggil. Kita akan lihat contohnya di langkah berikutnya.

---

#### Baris 4: `{{ $slot }}`

**Apa fungsinya?**

`$slot` adalah **variabel otomatis** yang berisi **isi** di antara tag pembuka dan penutup komponen.

Contoh pemanggilan:

```blade
<x-button>Simpan Produk</x-button>
<!--              ↑↑↑↑↑↑↑↑↑↑↑↑
                   ini yang akan masuk ke $slot -->
```

Maka di dalam komponen, `{{ $slot }}` akan menampilkan teks **"Simpan Produk"**.

**Analogi:**

Stempel "DITERIMA" punya **ruang kosong** untuk tanggal. Ruang kosong itu adalah **slot**. Kamu bisa isi dengan tanggal apa pun.

Di komponen Button, **slot** adalah ruang untuk **teks tombol**. Kamu bisa isi dengan apa pun:

- "Simpan Produk"
- "Edit Produk"
- "Hapus Produk"
- "Batal"
- "Kembali"
- "Detail Produk"
- Bahkan HTML seperti `<i class="icon-save"></i> Simpan`

> **Pesan mentor:**
> `$slot` adalah inti dari komponen yang **fleksibel**. Berkat slot, satu komponen `<x-button>` bisa dipakai untuk **banyak jenis tombol** dengan teks berbeda-beda. Tanpa slot, kita harus bikin komponen terpisah untuk "Tombol Simpan", "Tombol Edit", "Tombol Hapus", dst. Ribet!

---

#### Baris 5: `</button>`

Ini hanya **penutup** tag `<button>`. Tidak ada yang istimewa. Cukup pastikan setiap `<button>` punya penutup `</button>`.

---

## 6. Tangkapan Layar Struktur File Komponen

Mari kita lihat **keseluruhan isi file** sekali lagi, dengan komentar penjelas:

```blade
@props(['variant' => 'primary'])

<button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
    {{ $slot }}
</button>
```

**Ringkasan:**

| Bagian                              | Fungsi                                         |
| ----------------------------------- | ---------------------------------------------- |
| `@props(['variant' => 'primary'])`  | Deklarasi props `variant` dengan default primary |
| `$attributes->merge([...])`         | Gabungkan class default dengan class dari pemanggil |
| `'btn btn-' . $variant`             | Class CSS berdasarkan variant (mis: `btn btn-primary`) |
| `{{ $slot }}`                       | Tampilkan isi teks tombol dari pemanggil       |

---

## 7. Langkah 2: Pakai Komponen di Halaman `create.blade.php`

Sekarang kita coba **pakai** komponen yang sudah dibuat. Kita mulai dari halaman **Tambah Produk** karena tombolnya paling sederhana.

### 7.1 Kode lama (sebelum pakai komponen)

Di `produk/create.blade.php`, biasanya kamu menulis tombol seperti ini:

```blade
<form action="/products" method="POST">
    @csrf

    <!-- ...input form... -->

    <button type="submit" class="btn btn-primary">
        Simpan Produk
    </button>

    <a href="/products" class="btn btn-light">
        Batal
    </a>
</form>
```

Perhatikan tombol **Simpan Produk**: class-nya `btn btn-primary`.

### 7.2 Kode baru (pakai komponen `<x-button>`)

Ubah jadi seperti ini:

```blade
<form action="/products" method="POST">
    @csrf

    <!-- ...input form... -->

    <x-button type="submit" variant="primary">
        Simpan Produk
    </x-button>

    <x-button variant="secondary" href="/products">
        Batal
    </x-button>
</form>
```

### 7.3 Apa yang terjadi saat Blade memproses kode ini?

Saat halaman `create.blade.php` dirender, Blade melihat `<x-button type="submit" variant="primary">Simpan Produk</x-button>`.

**Langkah-langkah yang dilakukan Blade:**

1. **Cari file komponen** `resources/views/components/button.blade.php`.
2. **Baca props** `variant` dengan nilai `"primary"` (dari pemanggil).
3. **Baca atribut tambahan**: `type="submit"`.
4. **Baca slot** (isi di antara tag): teks `"Simpan Produk"`.
5. **Eksekusi isi komponen:**
   - `@props(['variant' => 'primary'])` → `$variant` sekarang bernilai `"primary"`.
   - `$attributes->merge(['class' => 'btn btn-primary'])` → hasilnya: `type="submit" class="btn btn-primary"`.
   - `{{ $slot }}` → tampilkan teks `"Simpan Produk"`.
6. **Hasil akhir HTML** yang dikirim ke browser:

```html
<button type="submit" class="btn btn-primary">
    Simpan Produk
</button>
```

**Hasilnya sama persis** dengan kode lama, tapi sekarang kamu **tidak perlu** lagi nulis `class="btn btn-primary"` secara manual. Cukup sebut `variant="primary"`, dan komponen yang urus class-nya.

---

## 8. Coba di Browser

Sekarang save file `button.blade.php` dan `create.blade.php`, lalu buka browser ke:

```
http://localhost:8000/products/create
```

Kamu harusnya melihat halaman **Tambah Produk** dengan tombol **Simpan Produk** berwarna biru (primary).

### Apa yang harus terlihat?

- Tombol **Simpan Produk** tampil dengan style `btn btn-primary` (biru).
- Tombol **Batal** tampil dengan style `btn btn-secondary` (abu-abu).
- Kedua tombol tampil **sama persis** dengan sebelumnya, karena komponen menghasilkan HTML yang sama.

> Kalau tombol tidak muncul atau error, cek **troubleshooting** di bagian bawah.

---

## 9. Diagram Alur Kerja Komponen

```
Halaman: produk/create.blade.php
┌──────────────────────────────────────────────┐
│ <x-button type="submit" variant="primary">   │
│   Simpan Produk                              │
│ </x-button>                                  │
└────────────────────────┬─────────────────────┘
                         │
                         │  Blade mencari komponen
                         ▼
        components/button.blade.php
        ┌───────────────────────────────────────┐
        │ @props(['variant' => 'primary'])      │
        │                                       │
        │ <button {{ $attributes->merge(...) }}>│
        │     {{ $slot }}                       │
        │ </button>                             │
        └───────────────────────────────────────┘
                         │
                         │  Hasil render
                         ▼
        <button type="submit"
                class="btn btn-primary">
            Simpan Produk
        </button>
```

---

## 10. Apa yang Sudah Kita Capai di Tahap Ini?

Mari kita lihat **manfaat** yang sudah kita dapat **hanya** dari komponen sederhana ini:

| Sebelum komponen                           | Sesudah komponen                             |
| ------------------------------------------ | -------------------------------------------- |
| `class="btn btn-primary"` ditulis manual   | Cukup `variant="primary"`, class diurus komponen |
| Kalau salah ketik `btn-prmary`, error CSS  | Variant diketik salah → tetap konsisten      |
| Ganti style → cari semua halaman satu-satu | Ganti style di `button.blade.php` → semua ikut |
| Setiap halaman nulis class sendiri         | Semua halaman pakai komponen yang sama       |

**Dan ini baru tahap 2!** Kita belum bahas variant warna secara lengkap, tombol Hapus yang pakai form, tombol sebagai link `<a>`, dan lainnya. Itu akan dibahas di tahap berikutnya.

---

## 11. Troubleshooting

### Error 1: `Unable to find component [button]`

**Penyebab:**

- File belum dibuat di `resources/views/components/button.blade.php`.
- Nama file salah (misal: `Button.blade.php` dengan huruf besar).

**Solusi:**

Cek path file: `resources/views/components/button.blade.php`. Pastikan nama file **kebab-case**.

### Error 2: `$slot` tidak muncul / tombol kosong

**Penyebab:**

- Kamu menulis komponen sebagai **self-closing**: `<x-button variant="primary" />` tanpa isi.
- Akibatnya `$slot` kosong, dan tombol tampil tanpa teks.

**Solusi:**

Tulis komponen dengan **isi di dalamnya**:

```blade
<!-- Benar: ada isi -->
<x-button variant="primary">
    Simpan Produk
</x-button>

<!-- Salah: self-closing tanpa isi -->
<x-button variant="primary" />
```

### Error 3: `Undefined variable $variant`

**Penyebab:**

- Lupa menulis `@props(['variant' => 'primary'])` di baris pertama komponen.

**Solusi:**

Pastikan baris pertama di `button.blade.php` adalah:

```blade
@props(['variant' => 'primary'])
```

### Error 4: Tombol tampil polos tanpa style Bootstrap

**Penyebab:**

- Bootstrap CSS belum di-load di layout `layout/app.blade.php`.
- Bisa juga karena salah merge class.

**Solusi:**

Cek di `layout/app.blade.php` apakah ada `<link rel="stylesheet" href="...bootstrap...">` di bagian `<head>`. Kalau belum ada, tambahkan.

### Error 5: Class CSS tidak tergabung dengan benar

**Penyebab:**

- Mengecek hasil di browser dan melihat class tidak muncul sebagai `btn btn-primary`.

**Solusi:**

Hindari menulis manual `class="..."` di pemanggil. Biarkan komponen yang urus class. Jika butuh class tambahan:

```blade
<x-button variant="primary" class="mt-3">
    Simpan Produk
</x-button>
```

Maka `->merge` akan menggabung jadi: `class="btn btn-primary mt-3"`.

---

## 12. Latihan Mandiri

**Latihan A:**

Coba ubah **tombol Batal** di `create.blade.php` menjadi **3 variant berbeda** dan lihat hasilnya di browser. Amati apa yang berubah.

1. `variant="primary"` → tombol apa warnanya?
2. `variant="secondary"` → tombol apa warnanya?
3. `variant="danger"` → tombol apa warnanya?

<details>
<summary><strong>Lihat jawaban Latihan A</strong></summary>

Ganti kode tombol Batal menjadi:

```blade
<!-- Coba 1 -->
<x-button variant="primary">Batal</x-button>

<!-- Coba 2 -->
<x-button variant="secondary">Batal</x-button>

<!-- Coba 3 -->
<x-button variant="danger">Batal</x-button>
```

Hasil yang harusnya muncul (dengan Bootstrap):

- `variant="primary"` → tombol **biru**
- `variant="secondary"` → tombol **abu-abu**
- `variant="danger"` → tombol **merah**

Walaupun teksnya sama-sama "Batal", warnanya beda karena variant beda. Di tahap berikutnya kita akan bahas **kapan** pakai variant yang mana.

</details>

---

## 13. Istilah Kunci Tahap Ini

| Istilah            | Arti sederhana                                                     |
| ------------------ | ------------------------------------------------------------------ |
| **Komponen Blade** | File Blade reusable yang dipanggil lewat `<x-nama>`                |
| **Anonymous komponen** | Komponen tanpa class PHP (cuma file Blade)                     |
| **`@props([...])`** | Directive untuk mendeklarasikan input yang diterima komponen      |
| **Props**          | Input/variabel yang dikirim ke komponen saat dipanggil            |
| **`$slot`**        | Variabel otomatis berisi isi di antara tag pembuka dan penutup    |
| **`$attributes`**  | Variabel otomatis berisi atribut tambahan dari pemanggil          |
| **`->merge([...])`** | Menggabungkan atribut default dengan atribut dari pemanggil     |
| **kebab-case**     | Konvensi nama file: huruf kecil + tanda hubung                    |

---

## 14. Ringkasan Tahap 2

1. **Folder komponen**: semua komponen Blade disimpan di `resources/views/components/`.
2. **Nama file**: harus **kebab-case** (contoh: `button.blade.php`, bukan `Button` atau `button_button`).
3. **Tag pemanggil**: `<x-nama-komponen>` (awalan `x-`, tanpa ekstensi).
4. **Struktur dasar komponen Button:**
   ```blade
   @props(['variant' => 'primary'])

   <button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
       {{ $slot }}
   </button>
   ```
5. **`@props([...])`** mendeklarasikan input komponen.
6. **`$slot`** menampilkan isi teks dari pemanggil.
7. **`$attributes->merge([...])`** menggabungkan class default dengan class dari pemanggil.
8. Saat dipanggil, Blade akan **mencari file komponen**, **menerapkan props**, dan **menyisipkan hasil** ke lokasi `<x-button>`.

---

## 15. Cek Pemahaman

1. Di folder mana file komponen Blade harus disimpan?
2. Benarkan nama file berikut: `Button.blade.php`? Jika tidak, apa yang benar?
3. Apa fungsi `@props(['variant' => 'primary'])`?
4. Apa isi `$slot` saat kamu menulis `<x-button>Halo</x-button>`?
5. Kenapa lebih baik pakai `$attributes->merge(['class' => '...'])` daripada langsung `class="..."`?
6. Bagaimana cara mengirim variant ke komponen?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. Di `resources/views/components/`.
2. **Salah**. Yang benar: `button.blade.php` (kebab-case, huruf kecil semua).
3. Mendeklarasikan bahwa komponen menerima props `variant`, dengan nilai default `"primary"` kalau tidak dikirim.
4. `$slot` berisi teks `"Halo"`.
5. Karena `->merge` **menggabungkan** class default dengan class tambahan dari pemanggil, sehingga pemanggil masih bisa tambah class sendiri tanpa menimpa class default.
6. Dengan menambahkan atribut `variant="..."` di tag, contoh: `<x-button variant="danger">Hapus</x-button>`.

</details>

---

## 16. Apakah Kamu Ingin Lanjut?

Di tahap 2 ini, kamu sudah:

- Membuat **file komponen Button** (`button.blade.php`)
- Memahami **struktur dasar** komponen (`@props`, `$slot`, `$attributes`)
- Memakai komponen di **halaman `create.blade.php`**
- Melihat hasilnya di browser

Tapi perhatikan: di tombol **Batal**, kita pakai `variant="secondary"`. Tapi sebenarnya **tidak ada penjelasan** kenapa secondary itu abu-abu, atau kenapa primary itu biru. Kita hanya mengandalkan **Bootstrap class** yang kebetulan sudah punya warna untuk masing-masing variant.

Di tahap berikutnya, kita akan **bongkar detail variant**: apa saja variant yang ada, kapan pakai primary vs secondary vs danger vs warning vs success, dan bagaimana cara mengatur **logika warna** di dalam komponen.

> ### Pertanyaan:
>
> Apakah kamu ingin lanjut ke langkah berikutnya: **memahami props dan variant secara mendalam**?
>
> Di tahap 3 kita akan bahas:
>
> 1. Daftar lengkap variant (primary, secondary, success, danger, warning, info)
> 2. Kapan pakai variant yang mana (dengan contoh kasus CRUD Produk)
> 3. Cara kerja logika `variant` di dalam komponen
> 4. Mengatur variant default
> 5. Variant tambahan di luar Bootstrap default
>
> Ketik **"lanjut"** untuk ke tahap 3,
> atau tanyakan jika ada bagian tahap 2 yang masih bingung.

---

> **Daftar Tahap (13. Komponen Button Blade):**
> - [x] Tahap 1 — Kenapa Tombol Harus Konsisten
> - [x] Tahap 2 — Membuat File Komponen `<x-button>` Sederhana (kamu di sini)
> - [ ] Tahap 3 — Memahami Props dan Variant
> - [ ] Tahap 4 — Slot dan Isi Tombol
> - [ ] Tahap 5 — Menerapkan `<x-button>` di Halaman Produk
> - [ ] Tahap 6 — Tombol Hapus dengan Form dan Konfirmasi
> - [ ] Tahap 7 — Variant Tambahan dan Tipe Link
> - [ ] Tahap 8 — Ringkasan dan Best Practice
