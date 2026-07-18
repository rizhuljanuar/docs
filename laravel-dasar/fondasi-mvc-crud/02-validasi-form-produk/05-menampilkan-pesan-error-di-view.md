# Bagian 5: Menampilkan Pesan Error di View (Form Blade)

> Modul: A. Level Dasar — Fondasi Laravel, MVC, CRUD
> Topik: Request Validation dengan FormRequest
> Prasyarat: Selesai membaca `01`, `02`, `03`, dan `04`

---

## Tujuan Bagian Ini

Sampai Tahap 4, validasi sudah **berfungsi**: data salah ditolak
dan tidak masuk database. Tapi ada masalah dari sudut pandang user:

> User input data salah, klik Simpan, lalu halaman form **reload
> tanpa pesan apa-apa**. User bingung: "Kenapa produk saya tidak
> tersimpan? Ada yang salah dengan apa yang saya input?"

Di Tahap 5 ini, kita buat **pesan error muncul** di halaman form
supaya user tahu **persis field mana yang salah dan kenapa**.

Kita juga akan pelajari cara **mempertahankan input user** yang
sudah benar supaya user tidak perlu **ngetik ulang semuanya** dari
awal.

---

## 1. Analogi: Kasir yang Sopan vs Kasir Cueang

**Kasir cueang (sebelum Tahap 5):**
Pelanggan: "Saya mau daftar produk."
Kasir: (cek paket, lalu diam-diam lempar ke tempat sampah, suruh
pelanggan balik)
Pelanggan: "...?"

**Kasir sopan (sesudah Tahap 5):**
Pelanggan: "Saya mau daftar produk."
Kasir: "Maaf, ada 2 masalah:
  - Harga harus angka positif (kamu input minus).
  - Nama produk wajib diisi.
  Oh, dan saya simpan dulu data kamu yang lain (deskripsi, stok)
  supaya kamu tidak perlu ngetik ulang. Tinggal perbaiki yang
  saya sebut saja ya."

Pesan error + data tersimpan = pengalaman user yang ramah.

---

## 2. Dari Mana Pesan Error Itu Berasal?

Sebelum nulis kode, kamu harus tahu **darimana data error itu
datang**. Jawabannya:

> Saat validasi gagal, Laravel **otomatis** menyimpan semua pesan
> error ke dalam **variabel global** bernama `$errors`.

Variabel `$errors` ini **selalu tersedia** di setiap halaman view
(Blade) kamu, **tanpa perlu kamu kirim manual** dari controller.
Laravel sudah menyiapkannya untukmu.

Beberapa method penting yang sering dipakai di Blade:

| Method                    | Arti                                              |
|---------------------------|---------------------------------------------------|
| `$errors->any()`          | Apakah ada **satu pun** error? (true/false)       |
| `$errors->all()`          | Ambil **semua** pesan error sebagai array.        |
| `$errors->has('nama')`    | Apakah ada **error untuk field nama**?            |
| `$errors->first('nama')`  | Ambil **pesan error pertama** untuk field nama.   |
| `$errors->get('nama')`    | Ambil **semua pesan** error untuk field nama.     |

Kita akan pakai method-method ini di form.

---

## 3. Dari Mana "Old Input" Berasal?

Selain `$errors`, Laravel juga menyediakan **input lama user** di
variabel global bernama `old()`.

```blade
{{ old('nama') }}
```

**Fungsinya**: Mengambil **nilai yang sudah user ketik sebelumnya**
untuk field tertentu, supaya tidak hilang saat halaman form reload
karena validasi gagal.

Ini **kunci** supaya user tidak marah: mereka sudah ngetik panjang,
eh form reload dan semuanya hilang. Dengan `old()`, input mereka
yang **sudah benar** akan **tetap ada**.

---

## 4. Langkah 1: Tampilkan Daftar Error di Atas Form (Ringkasan Cepat)

Buka file form tambah produk kamu, kemungkinan ada di:

```
resources/views/products/create.blade.php
```

Di bagian **atas form** (atau di atas tag `<form>`), tambahkan blok
berikut:

```blade
@if ($errors->any())
    <div style="background-color: #fee; border: 1px solid red; padding: 10px; margin-bottom: 15px;">
        <strong>Maaf, ada masalah dengan input kamu:</strong>
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

### Penjelasan per baris

```blade
@if ($errors->any())
```
**Artinya**: "Kalau ada **satu pun** error, jalankan blok ini."

```blade
<div style="background-color: #fee; ...">
```
**Artinya**: Bikin **kotak merah muda** (warning box) biar pesan
error menonjol secara visual. Style-nya inline dulu supaya sederhana.
Nanti kalau kamu pakai Bootstrap/Tailwind, bisa diganti class.

```blade
<strong>Maaf, ada masalah dengan input kamu:</strong>
```
**Artinya**: Judul singkat supaya user paham ini adalah pesan error.

```blade
<ul>
    @foreach ($errors->all() as $error)
        <li>{{ $error }}</li>
    @endforeach
</ul>
```
**Artinya**: Looping semua pesan error, tampilkan sebagai **daftar
poin** (bullet list).

> Hasilnya: User akan lihat **daftar semua error** di atas form,
> jadi tahu semua field yang bermasalah sekaligus.

---

## 5. Langkah 2: Tampilkan Error per Field (Lebih Spesifik)

Daftar error di atas sudah cukup baik, tapi lebih enak kalau user
**langsung lihat** error di **samping field yang salah**.

Mari kita kembangkan form kamu. Contoh form awal yang sederhana
(sebelum validasi):

```blade
<form action="/products" method="POST">
    @csrf

    <label>Nama Produk:</label>
    <input type="text" name="nama">

    <label>Harga:</label>
    <input type="number" name="harga" step="0.01">

    <label>Stok:</label>
    <input type="number" name="stok">

    <label>Deskripsi:</label>
    <textarea name="deskripsi"></textarea>

    <button type="submit">Simpan</button>
</form>
```

Sekarang kita ubah **field nama** untuk menampilkan error + mempertahankan
input lama:

```blade
<label>Nama Produk:</label>
<input type="text" name="nama" value="{{ old('nama') }}">

@error('nama')
    <span style="color: red; font-size: 12px;">
        {{ $message }}
    </span>
@enderror
```

### Penjelasan per baris

```blade
<input type="text" name="nama" value="{{ old('nama') }}">
```
**Artinya**: Isi value input dengan **nilai lama user**. Kalau user
sebelumnya input "Buku", nilai "Buku" akan tetap muncul saat form
reload. Kalau tidak ada old input (misal user baru pertama kali
buka form), `old('nama')` akan kosong.

```blade
@error('nama')
    <span style="color: red;">{{ $message }}</span>
@enderror
```
**Artinya**: Direktif Blade `@error('nama')` mengecek apakah ada
error untuk field `nama`. Kalau ada, jalankan bloknya. Variabel
`$message` sudah otomatis berisi **pesan error untuk field itu**.

---

## 6. Langkah 3: Terapkan Pola yang Sama ke Semua Field

Setelah berhasil di field `nama`, terapkan pola yang sama ke
`harga`, `stok`, dan `deskripsi`. Berikut form lengkapnya:

```blade
<form action="/products" method="POST">
    @csrf

    {{-- Field NAMA --}}
    <div>
        <label>Nama Produk:</label>
        <input type="text" name="nama" value="{{ old('nama') }}">
        @error('nama')
            <span style="color: red; font-size: 12px;">{{ $message }}</span>
        @enderror
    </div>

    {{-- Field HARGA --}}
    <div>
        <label>Harga:</label>
        <input type="number" name="harga" step="0.01" value="{{ old('harga') }}">
        @error('harga')
            <span style="color: red; font-size: 12px;">{{ $message }}</span>
        @enderror
    </div>

    {{-- Field STOK --}}
    <div>
        <label>Stok:</label>
        <input type="number" name="stok" value="{{ old('stok') }}">
        @error('stok')
            <span style="color: red; font-size: 12px;">{{ $message }}</span>
        @enderror
    </div>

    {{-- Field DESKRIPSI --}}
    <div>
        <label>Deskripsi:</label>
        <textarea name="deskripsi">{{ old('deskripsi') }}</textarea>
        @error('deskripsi')
            <span style="color: red; font-size: 12px;">{{ $message }}</span>
        @enderror
    </div>

    <button type="submit">Simpan</button>
</form>
```

### Catatan khusus untuk `<textarea>`

Perhatikan di field `deskripsi`, nilai `old()` diletakkan **di antara
tag pembuka dan penutup textarea**, **bukan** di atribut `value=`:

```blade
<textarea name="deskripsi">{{ old('deskripsi') }}</textarea>
```

Ini karena tag `<textarea>` **tidak punya atribut `value`**. Nilainya
harus diletakkan **di dalam** tag.

---

## 7. Langkah 4: Jangan Lupa Form Edit Juga!

Buka file form edit produk, kemungkinan ada di:

```
resources/views/products/edit.blade.php
```

Di form edit, kita **sudah punya data produk dari database**. Jadi
kita harus **memprioritaskan** data dari database dulu, baru kalau
ada input lama (karena validasi gagal), pakai input lama.

Caranya pakai **operator null coalescing** `??`:

```blade
<input type="text" name="nama" value="{{ old('nama', $product->nama) }}">
```

**Artinya**: Pakai `old('nama')` kalau ada (validasi gagal). Kalau
tidak ada, pakai `$product->nama` (data dari database).

Berikut form edit lengkap:

```blade
<form action="/products/{{ $product->id }}" method="POST">
    @csrf
    @method('PUT')

    <div>
        <label>Nama Produk:</label>
        <input type="text" name="nama" value="{{ old('nama', $product->nama) }}">
        @error('nama')
            <span style="color: red; font-size: 12px;">{{ $message }}</span>
        @enderror
    </div>

    <div>
        <label>Harga:</label>
        <input type="number" name="harga" step="0.01" value="{{ old('harga', $product->harga) }}">
        @error('harga')
            <span style="color: red; font-size: 12px;">{{ $message }}</span>
        @enderror
    </div>

    <div>
        <label>Stok:</label>
        <input type="number" name="stok" value="{{ old('stok', $product->stok) }}">
        @error('stok')
            <span style="color: red; font-size: 12px;">{{ $message }}</span>
        @enderror
    </div>

    <div>
        <label>Deskripsi:</label>
        <textarea name="deskripsi">{{ old('deskripsi', $product->deskripsi) }}</textarea>
        @error('deskripsi')
            <span style="color: red; font-size: 12px;">{{ $message }}</span>
        @enderror
    </div>

    <button type="submit">Update</button>
</form>
```

> **Bedanya `old()` di create vs edit:**
> - **Create**: `value="{{ old('nama') }}"` → hanya pakai old input.
> - **Edit**: `value="{{ old('nama', $product->nama) }}"` → pakai old
>   input kalau ada, kalau tidak ada pakai data dari database.

---

## 8. Cara Uji Coba

### Uji form tambah (create)

1. Pastikan server jalan: `php artisan serve`.
2. Buka `http://localhost:8000/products/create`.
3. Isi:
   - nama: **kosong**
   - harga: `-5000`
   - stok: `2.5`
   - deskripsi: `ok`
4. Klik **Simpan**.

**Yang harus terjadi:**
- Halaman **kembali ke form**.
- **Kotak error merah** muncul di atas, berisi daftar error.
- **Pesan error** muncul di bawah setiap field yang salah:
  - nama: "The nama field is required."
  - harga: "The harga must be at least 0."
  - stok: "The stok must be an integer."
  - deskripsi: "The deskripsi must be at least 10 characters."
- Input yang sudah benar **tetap ada** (misal kalau kamu input
  deskripsi "bagus banget banget banget" yang valid, teks itu tetap
  muncul di textarea karena `old('deskripsi')`).

### Uji form edit

1. Buka halaman edit salah satu produk:
   `http://localhost:8000/products/1/edit`.
2. **Hapus** isi field nama.
3. Ubah harga jadi `-1000`.
4. Klik **Update**.

**Yang harus terjadi:**
- Halaman **kembali ke form edit**.
- Pesan error muncul di field yang salah.
- **Data produk lain tetap ada** (tidak hilang) karena diambil dari
  `$product->...`.
- Field yang kamu ubah tetap mempertahankan input terakhir kamu
  (misal harga yang kamu ketik `-1000` masih terlihat, sebagai sinyal
  "yang ini kamu salah ketik, perbaiki").

---

## 9. Tips Tambahan: Pakai CSS Class Biar Rapi

Sampai sekarang kita pakai inline style (`style="color: red; ..."`).
Cara ini cepat tapi tidak praktis untuk form besar. Kalau kamu sudah
pakai CSS eksternal, lebih baik pakai class.

Contoh (kalau kamu punya file CSS sendiri):

```blade
@error('nama')
    <span class="text-danger error-message">{{ $message }}</span>
@enderror
```

Dan di CSS:
```css
.text-danger {
    color: red;
}
.error-message {
    font-size: 12px;
    display: block;
    margin-top: 4px;
}
```

Kalau kamu pakai **Bootstrap**, `text-danger` adalah class bawaan
Bootstrap untuk teks merah. Tidak perlu bikin CSS sendiri.

> Untuk belajar validasi, **inline style sudah cukup**. Ini bisa
> kamu rapikan nanti.

---

## 10. Latihan Mandiri

1. **Hapus `old('nama')`** dari input nama, lalu input data salah.
   Apa yang terjadi dengan field nama saat form reload?
   (Harus kosong. Inilah kenapa `old()` penting.)
2. **Hapus salah satu blok `@error('harga')`** di form, lalu input
   harga minus. Apa yang terjadi? (Error hanya muncul di daftar atas,
   tidak muncul di samping field harga.)
3. **Coba ubah teks error**: tambahkan `:attribute` atau lihat dokumentasi
   Laravel cara **kustomisasi pesan error**. Ini bonus, tidak wajib.

---

## 11. Ringkasan Bagian 5

- Saat validasi gagal, Laravel otomatis menyimpan **pesan error** ke
  variabel global `$errors` dan **input lama user** ke fungsi `old()`.
- **Tiga alat utama** di Blade:
  - `$errors->any()` dan `$errors->all()` untuk **daftar ringkasan error**.
  - `@error('field') ... @enderror` untuk **error per field**.
  - `old('field')` untuk **mempertahankan input user**.
- Di **form tambah**: gunakan `value="{{ old('nama') }}"`.
- Di **form edit**: gunakan `value="{{ old('nama', $product->nama) }}"`
  supaya data dari database muncul sebagai default.
- Untuk `<textarea>`, taruh `old()` **di dalam tag**, bukan di
  atribut `value`.
- Pesan error default Laravel biasanya dalam **bahasa Inggris**.
  Kalau mau pesan dalam **bahasa Indonesia**, kita akan bahas cara
  kustomisasi pesan di Tahap 7 (Review).

---

## 12. FAQ Singkat

**Q: Kenapa pesan error muncul dalam bahasa Inggris?**
A: Itu default Laravel. Nanti di Tahap 7 kita akan bahas cara ubah
ke bahasa Indonesia, baik lewat file bahasa (`lang/id/validation.php`)
maupun lewat method `messages()` di FormRequest.

**Q: Apakah saya harus selalu pakai `old()` di form?**
A: Sangat **direkomendasikan**. Tanpa `old()`, user harus ngetik
ulang semua field setiap kali salah input. Ini buruk untuk UX
(pengalaman user).

**Q: Kenapa `@error` tidak muncul di `@if ($errors->any())`?**
A: Bisa keduanya dipakai bersamaan:
- Daftar atas untuk **ringkasan semua error**.
- `@error` per field untuk **detail di samping input**.
Tidak konflik, justru saling melengkapi.

**Q: Kenapa di form edit pakai `$product->nama` dan bukan langsung
`old('nama')` saja?**
A: Karena di form edit, kalau user **baru saja membuka halaman**
(belum submit apapun), tidak ada old input. Form harus tetap
menampilkan data dari database (`$product->nama`). Operator `??`
menangani ini: pakai old kalau ada, kalau tidak pakai data DB.

---

> **Berhenti di sini.**
>
> Pada bagian berikutnya kita akan lakukan **integrasi penuh** dari
> semua yang sudah dipelajari (Tahap 1-5) ke dalam project CRUD
> produk kamu. Kita akan pastikan FormRequest, controller, route,
> dan form Blade semua terhubung dengan benar.
>
> **Apakah kamu ingin lanjut ke langkah berikutnya:
> Tahap 6 — Menerapkan FormRequest di Controller CRUD?**
