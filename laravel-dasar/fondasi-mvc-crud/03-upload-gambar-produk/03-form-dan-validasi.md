# Upload Gambar Produk — Tahap 3: Form Upload & Validasi File

> Bagian dari: Laravel Dasar — Fondasi MVC & CRUD
> Topik: 3. Upload Gambar Produk
> Tahap: 3 dari 5 — **Form Upload & Validasi File** (mulai kode validasi)

---

## 1. Tujuan Tahap Ini

Di tahap 1 kita tahu **kenapa** harus validasi. Di tahap 3 ini kita tulis
**bagaimana** validasinya di Laravel.

Kita akan:

1. Membuat form HTML dengan input file.
2. Menulis **aturan validasi** (validation rules) untuk file gambar.
3. Menampilkan **pesan error** kalau file tidak valid.

Kita **belum** menyimpan file ke storage (itu tahap 4). Fokus dulu ke "penjaga
pintu" — file yang lewat harus yang benar.

---

## 2. Langkah 1: Form HTML dengan `enctype`

Form upload file **wajib** punya atribut `enctype="multipart/form-data"`.
Kalau tidak, file tidak akan terkirim ke server.

`resources/views/produk/create.blade.php`:

```php
<form action="{{ route('produk.store') }}" method="POST" enctype="multipart/form-data">
    @csrf

    <label for="nama">Nama Produk</label>
    <input type="text" name="nama" id="nama" value="{{ old('nama') }}">
    @error('nama') <span style="color:red">{{ $message }}</span> @enderror

    <br>

    <label for="gambar">Gambar Produk</label>
    <input type="file" name="gambar" id="gambar">
    @error('gambar') <span style="color:red">{{ $message }}</span> @enderror

    <br>

    <button type="submit">Simpan</button>
</form>
```

Hal penting:

- `enctype="multipart/form-data"` → **wajib** untuk upload file.
- `@csrf` → proteksi bawaan Laravel (tidak ada hubungan langsung dengan file,
  tapi wajib di setiap form POST).
- `<input type="file">` → input khusus untuk memilih file.
- `@error('gambar')` → otomatis menampilkan pesan error untuk field `gambar`.

---

## 3. Langkah 2: Aturan Validasi untuk File Gambar

Di controller, kita pakai method `validate()` bawaan Laravel.

`app/Http/Controllers/ProdukController.php`:

```php
public function store(Request $request)
{
    $data = $request->validate([
        'nama'   => ['required', 'string', 'max:100'],
        'gambar' => ['required', 'image', 'mimes:jpg,jpeg,png,webp', 'max:2048'],
    ]);

    // di tahap 4: simpan file ke storage
    // di tahap 5: simpan data ke database

    return redirect()->back()->with('success', 'Data valid!');
}
```

Mari kita bedah **aturan untuk `gambar`** satu per satu:

| Rule       | Arti                                                                   |
| ---------- | ---------------------------------------------------------------------- |
| `required` | Wajib diisi. Tidak boleh kosong.                                       |
| `image`    | Harus file gambar yang valid (png, jpg, jpeg, bmp, gif, svg, webp).    |
| `mimes:`   | Batasi ekstensi tertentu. Di sini hanya jpg/jpeg/png/webp.             |
| `max:2048` | Ukuran maksimal **2048 kilobyte** = 2 MB.                              |

> Catatan: nilai `max:` **dalam kilobyte**, bukan byte. Jadi `max:2048` = 2 MB.

### Kenapa batasi `mimes` walau sudah pakai `image`?

Rule `image` sudah memastikan file adalah gambar valid. Tapi kadang kita ingin
**membatasi lebih ketat**, misal hanya png/jpg/webp (tidak menerima gif/svg
karena alasan keamanan atau desain). Maka tambahkan `mimes:`.

### Kenapa `max:2048`?

- Supaya user tidak upload foto 10 MB (bikin server penuh & halaman lambat).
- 2 MB sudah cukup untuk foto produk berkualitas normal.

---

## 4. Pesan Error Validasi (Bawaan & Custom)

Secara default Laravel sudah punya pesan dalam bahasa Inggris:

```
The gambar must be an image.
The gambar may not be greater than 2048 kilobytes.
```

Untuk membuat pesan dalam bahasa Indonesia, ubah di
`lang/id/validation.php` (atau `resources/lang/id/validation.php` di versi lama):

```php
// lang/id/validation.php
return [
    'required' => ':attribute wajib diisi.',
    'image'    => ':attribute harus berupa gambar.',
    'mimes'    => ':attribute harus berformat: :values.',
    'max'      => [
        'numeric' => ':attribute tidak boleh lebih dari :max.',
        'file'    => ':attribute tidak boleh lebih dari :max kilobyte.',
        'string'  => ':attribute tidak boleh lebih dari :max karakter.',
    ],
    'attributes' => [
        'nama'   => 'Nama produk',
        'gambar' => 'Gambar produk',
    ],
];
```

Dan set bahasa default ke `id` di `config/app.php`:

```php
'locale' => 'id',
```

Maka pesan error menjadi ramah:

```
Gambar produk wajib diisi.
Gambar produk harus berupa gambar.
Gambar produk tidak boleh lebih dari 2048 kilobyte.
```

---

## 5. Cara Laravel Mengecek "Beneran Gambar atau Nggak"

Laravel tidak hanya melihat ekstensi file (karena ekstensi bisa dipalsukan).
Laravel juga **mengecek isi file** menggunakan metode `MIME type` yang
membaca "tanda tangan" byte pertama dari file.

Jadi user tidak bisa mengubah `script.php` menjadi `script.jpg` lolos — Laravel
akan menolak karena isi file **tidak cocok** dengan format gambar.

> Inilah kenapa di tahap 1 kita bilang validasi itu seperti "penjaga gudang"
> yang tidak hanya melihat label paket, tapi juga mengecek isinya.

---

## 6. Uji Coba Skenario (Tanpa Menyimpan File)

Setelah kode di atas, coba submit form dengan input berikut. Lihat hasilnya:

| Input dari user                      | Hasil validasi   | Alasan                             |
| ------------------------------------ | ---------------- | ---------------------------------- |
| Tidak pilih file apa pun             | **GAGAL**        | Rule `required`                    |
| Upload `laporan.pdf`                 | **GAGAL**        | Bukan image + bukan mimes          |
| Upload `foto.jpg` 5 MB               | **GAGAL**        | Lebih dari `max:2048` (2 MB)       |
| Upload `script.php` ganti nama `.jpg`| **GAGAL**        | MIME check: isi bukan gambar       |
| Upload `sepatu.png` 500 KB           | **LOLOS** ✅      | Sesuai semua aturan                |

Kalau semua skenario "GAGAL" di atas benar menolak, berarti **penjaga pintu**
kita bekerja. Tinggal tahap 4: menyimpan file yang sudah lolos.

---

## 7. Ringkasan Tahap 3

- Form upload file **wajib** `enctype="multipart/form-data"`.
- Input pakai `<input type="file" name="gambar">`.
- Aturan minimal: `'gambar' => ['required', 'image', 'mimes:jpg,jpeg,png,webp', 'max:2048']`.
- `max:` dalam **kilobyte**. `max:2048` = 2 MB.
- Laravel cek **MIME type dari isi file**, bukan cuma ekstensi (aman dari pemalsuan).
- Pakai `@error('gambar')` di Blade untuk tampilkan pesan error.
- Custom bahasa Indonesia lewat `lang/id/validation.php`.

Belum ada penyimpanan file — itu ada di **tahap 4**.

---

## 8. Cek Pemahaman

1. Apa yang terjadi kalau form tidak pakai `enctype="multipart/form-data"`?
2. Kenapa `max:2048` artinya 2 MB, bukan 2048 byte?
3. Kenapa user tidak bisa menipu Laravel dengan rename `.php` jadi `.jpg`?
4. Tulis aturan validasi untuk: gambar wajib, format jpg/png/webp, maksimal 1 MB.

---

> **Pertanyaan untuk kamu:** Sudah paham bagian validasi?
> Mau lanjut ke **Tahap 4 — Menyimpan & Menampilkan Gambar**
> (`Storage::disk('public')->putFileAs(...)`, simpan path ke DB, dan
> `Storage::url()` di Blade), atau ulas ulang tahap 3 dulu?
