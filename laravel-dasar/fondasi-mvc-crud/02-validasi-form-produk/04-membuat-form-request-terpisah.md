# Bagian 4: Membuat FormRequest Terpisah (Cara B)

> Modul: A. Level Dasar — Fondasi Laravel, MVC, CRUD
> Topik: Request Validation dengan FormRequest
> Prasyarat: Selesai membaca `01`, `02`, dan `03`

---

## Tujuan Bagian Ini

Di Tahap 3, aturan validasi ditulis **langsung di dalam controller**
(Cara A). Masalahnya: aturan yang sama ditulis **dua kali** (di
`store()` dan `update()`). Bayangkan kalau kamu juga butuh validasi
untuk **import produk dari Excel**, validasi itu harus ditulis lagi.

Sekarang kita pindahkan aturan itu ke **file tersendiri** yang disebut
**FormRequest**. Tujuannya:

1. Aturan validasi **ditulis sekali saja**, di satu tempat khusus.
2. Aturan bisa **dipakai ulang** di controller mana pun yang membutuhkan.
3. Controller jadi **lebih bersih** dan fokus ke logika bisnis, bukan ke
   aturan validasi.

---

## 1. Analogi: Kasir Specialist vs Satpam Specialist

Di Cara A (Tahap 3), controller itu seperti **kasir serba bisa**:
- Menerima pelanggan.
- Mengecek paket (validasi).
- Memasukkan paket ke gudang.

Lama-lama kasir **capek** karena tugasnya terlalu banyak.

Di Cara B (FormRequest), kita **memisahkan peran**:
- **Satpam khusus** (FormRequest) — fokus cuma ngecek paket.
- **Kasir** (Controller) — fokus menerima paket yang **sudah
  pasti lolos** dan memasukkan ke gudang.

Hasilnya: keduanya jadi **lebih cepat dan rapi** karena fokus
masing-masing.

---

## 2. Langkah 1: Buat File FormRequest dengan Artisan

Laravel punya perintah khusus untuk membuat file FormRequest. Buka
**terminal**, pastikan kamu berada di folder project Laravel, lalu
jalankan:

```bash
php artisan make:request StoreProductRequest
```

**Arti perintah ini:**
- `php artisan` — alat bawaan Laravel untuk berbagai tugas.
- `make:request` — buat file FormRequest baru.
- `StoreProductRequest` — nama file-nya. Konvensi Laravel:
  - Mulai dengan **aksi** (`Store` untuk tambah/simpan).
  - Diikuti **entitas** (`Product`).
  - Diakhiri kata `Request`.

> **Catatan penamaan**: Nanti kita akan bahas kenapa kita pakai nama
> `StoreProductRequest` saja (tidak bikin `UpdateProductRequest`
> terpisah). Untuk produk, aturan validasi tambah dan edit biasanya
> **sama persis**, jadi cukup satu file.

---

## 3. Langkah 2: Lihat File yang Baru Dibuat

Setelah perintah di atas dijalankan, file baru muncul di:

```
app/Http/Requests/StoreProductRequest.php
```

Buka file itu di editor kamu. Isi default-nya kurang lebih begini:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreProductRequest extends FormRequest
{
    public function authorize()
    {
        return false;
    }

    public function rules()
    {
        return [
            //
        ];
    }
}
```

Ada **2 method penting** yang harus kamu perhatikan:
- `authorize()` → menentukan **siapa yang boleh** pakai request ini.
- `rules()` → menentukan **aturan validasi**.

Kita bahas keduanya satu per satu.

---

## 4. Method authorize() — Siapa yang Boleh?

```php
public function authorize()
{
    return false;
}
```

**Fungsi**: Menentukan apakah user **berhak** menggunakan request ini
(atau dengan kata lain, apakah user ini boleh melakukan aksi ini).

**Nilai `false` default** artinya: **SEMUA user ditolak**. Ini
fitur keamanan Laravel supaya kamu tidak lupa memikirkan izin.

**Karena sekarang kamu masih belajar dan belum belajar auth/login**,
kita ubah jadi `true` agar semua user diizinkan:

```php
public function authorize()
{
    return true;
}
```

**Analogi**: Satpam bilang "Boleh masuk semua, silakan" karena belum
ada aturan khusus. Nanti kalau kamu sudah belajar sistem login, di
sini kamu bisa tulis misalnya `return auth()->check();` artinya
"hanya yang sudah login yang boleh."

---

## 5. Method rules() — Isi Aturan Validasi

Sekarang isi method `rules()` dengan **aturan yang sama** dari Tahap 3:

```php
public function rules()
{
    return [
        'nama'      => 'required',
        'harga'     => 'required|numeric|min:0',
        'stok'      => 'required|integer|min:0',
        'deskripsi' => 'nullable|min:10',
    ];
}
```

**Fungsinya sama persis** dengan yang kamu tulis di controller di
Tahap 3. Bedanya: sekarang aturan ini **disimpan di file khusus**,
bukan di controller.

---

## 6. Bentuk Akhir File StoreProductRequest.php

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreProductRequest extends FormRequest
{
    public function authorize()
    {
        return true;
    }

    public function rules()
    {
        return [
            'nama'      => 'required',
            'harga'     => 'required|numeric|min:0',
            'stok'      => 'required|integer|min:0',
            'deskripsi' => 'nullable|min:10',
        ];
    }
}
```

**Hanya 2 perubahan kecil dari default Laravel:**
1. `return false;` di `authorize()` diubah jadi `return true;`.
2. Method `rules()` diisi aturan validasi produk.

---

## 7. Langkah 3: Ubah Controller untuk Memakai FormRequest

Sekarang waktunya **menggunakan** FormRequest di controller. Buka:

```
app/Http/Controllers/ProductController.php
```

### 7a. Tambahkan import di bagian atas file

Di bagian atas file (setelah `namespace ...` dan `use ...` yang lain),
tambahkan:

```php
use App\Http\Requests\StoreProductRequest;
```

Fungsinya: Memberi tahu controller "ini, aku mau pakai kelas
`StoreProductRequest` dari folder `app/Http/Requests`".

---

### 7b. Ubah method store()

**Sebelum (Cara A dari Tahap 3):**

```php
public function store(Request $request)
{
    $request->validate([
        'nama'      => 'required',
        'harga'     => 'required|numeric|min:0',
        'stok'      => 'required|integer|min:0',
        'deskripsi' => 'nullable|min:10',
    ]);

    Product::create($request->all());

    return redirect('/products')->with('success', 'Produk berhasil ditambahkan.');
}
```

**Sesudah (Cara B sekarang):**

```php
public function store(StoreProductRequest $request)
{
    Product::create($request->validated());

    return redirect('/products')->with('success', 'Produk berhasil ditambahkan.');
}
```

---

### 7c. Penjelasan Perubahan Per Baris

#### Perubahan 1: Tipe parameter

```php
// Sebelum:
public function store(Request $request)

// Sesudah:
public function store(StoreProductRequest $request)
```

**Artinya**: Kita bilang ke Laravel "method `store()` ini hanya
menerima request yang sudah divalidasi oleh `StoreProductRequest`".

**Yang Laravel lakukan otomatis** saat melihat tipe parameter ini:
1. **Sebelum** method dijalankan, Laravel menjalankan validasi yang
   ada di `StoreProductRequest::rules()`.
2. Kalau validasi **gagal**, Laravel otomatis **kembalikan user ke form**
   + pesan error (sama seperti Cara A).
3. Kalau validasi **lolos**, baru method `store()` dijalankan.

**Kesimpulan**: Kamu **tidak perlu** memanggil `$request->validate()`
lagi. Laravel yang menjalankannya secara otomatis.

#### Perubahan 2: Hapus panggilan validate()

Karena Laravel sudah menjalankan validasi secara otomatis, baris ini
**dihapus**:

```php
// HAPUS baris ini:
$request->validate([ ... ]);
```

#### Perubahan 3: Gunakan $request->validated()

```php
// Sebelum:
Product::create($request->all());

// Sesudah:
Product::create($request->validated());
```

**Bedanya apa?**
- `$request->all()` — ambil **semua data** dari request, tanpa filter.
- `$request->validated()` — ambil **hanya data yang sudah lolos
  validasi**, sesuai rules yang kita definisikan.

**Kenapa pakai `validated()`?**
Ini lebih **aman**. Misal ada user iseng yang **menambahkan field
ekstra** di form (seperti `is_admin = true`), field itu **tidak akan
masuk** ke `Product::create()` karena tidak ada di daftar rules.
Ini mencegah serangan yang disebut **mass assignment vulnerability**.

---

## 8. Langkah 4: Ubah method update() Juga

Method `update()` juga harus pakai FormRequest supaya aturan
validasi **dipakai ulang** (ingat: ini tujuan utama Cara B).

**Sebelum:**

```php
public function update(Request $request, $id)
{
    $request->validate([
        'nama'      => 'required',
        'harga'     => 'required|numeric|min:0',
        'stok'      => 'required|integer|min:0',
        'deskripsi' => 'nullable|min:10',
    ]);

    $product = Product::findOrFail($id);
    $product->update($request->all());

    return redirect('/products')->with('success', 'Produk berhasil diupdate.');
}
```

**Sesudah:**

```php
public function update(StoreProductRequest $request, $id)
{
    $product = Product::findOrFail($id);
    $product->update($request->validated());

    return redirect('/products')->with('success', 'Produk berhasil diupdate.');
}
```

Perhatikan: **tidak ada aturan validasi yang ditulis ulang**. Semua
sudah di-handle oleh `StoreProductRequest`. Controller jadi **lebih
pendek dan bersih**.

---

## 9. Bandingkan: Controller Sebelum vs Sesudah

### Sebelum (Cara A, Tahap 3)

```php
public function store(Request $request)
{
    $request->validate([
        'nama'      => 'required',
        'harga'     => 'required|numeric|min:0',
        'stok'      => 'required|integer|min:0',
        'deskripsi' => 'nullable|min:10',
    ]);

    Product::create($request->all());

    return redirect('/products')->with('success', 'Produk berhasil ditambahkan.');
}

public function update(Request $request, $id)
{
    $request->validate([
        'nama'      => 'required',
        'harga'     => 'required|numeric|min:0',
        'stok'      => 'required|integer|min:0',
        'deskripsi' => 'nullable|min:10',
    ]);

    $product = Product::findOrFail($id);
    $product->update($request->all());

    return redirect('/products')->with('success', 'Produk berhasil diupdate.');
}
```

**Aturan ditulis 2x.** Kalau mau ubah aturan harga, harus ubah di
2 tempat. Berisiko lupa.

### Sesudah (Cara B, sekarang)

```php
public function store(StoreProductRequest $request)
{
    Product::create($request->validated());

    return redirect('/products')->with('success', 'Produk berhasil ditambahkan.');
}

public function update(StoreProductRequest $request, $id)
{
    $product = Product::findOrFail($id);
    $product->update($request->validated());

    return redirect('/products')->with('success', 'Produk berhasil diupdate.');
}
```

**Aturan ditulis 1x** di file `StoreProductRequest.php`. Controller
fokus hanya ke logika bisnis.

---

## 10. Cara Uji Coba

Sama seperti Tahap 3:

1. Jalankan server: `php artisan serve`.
2. Buka `http://localhost:8000/products/create`.
3. Isi data **salah** (nama kosong, harga minus) → klik Simpan.
   - Validasi akan menolak, form tidak tersimpan.
4. Isi data **benar** → klik Simpan.
   - Validasi lolos, data tersimpan.
5. Coba juga edit produk lewat form edit dengan data salah.
   - Validasi `update()` juga menolak.

> Pesan error **belum terlihat** di form karena kita belum tampilkan.
> Itu topik Tahap 5. Yang penting sekarang: data salah tidak masuk
> database di **store** dan **update**.

---

## 11. Visualisasi Struktur Folder Setelah Perubahan

```
app/
└── Http/
    ├── Controllers/
    │   └── ProductController.php       ← lebih ringkas, tanpa aturan validasi
    └── Requests/
        └── StoreProductRequest.php     ← aturan validasi di sini
```

Dibanding Cara A yang menumpuk semua di controller, sekarang kita
punya **ruangan khusus** untuk validasi. Inilah salah satu **best
practice** Laravel untuk aplikasi nyata.

---

## 12. Latihan Mandiri

1. **Coba ubah aturan** di `StoreProductRequest.php`. Misalnya ubah
   `min:0` jadi `min:1000` di harga. Lalu coba input harga `500`.
   Apa yang terjadi? (Harus ditolak karena di bawah 1000.)
2. **Coba hapus field** `'nama' => 'required'` di `rules()`. Lalu coba
   input nama kosong. Apa yang terjadi? (Harus lolos padahal seharusnya
   wajib.)
3. **Coba komentar sementara** pemanggilan `use App\Http\Requests\StoreProductRequest;`
   di controller. Apa error yang muncul? (Class not found.)

Latihan ini membantu kamu paham **hubungan** antara FormRequest dan
Controller.

---

## 13. Ringkasan Bagian 4

- **FormRequest** = file khusus tempat menyimpan aturan validasi,
  dibuat dengan `php artisan make:request NamaRequest`.
- File FormRequest disimpan di `app/Http/Requests/`.
- Ada **2 method wajib**:
  - `authorize()` — siapa yang boleh (diubah ke `true` selama belajar).
  - `rules()` — tempat aturan validasi ditulis.
- Di controller, **ganti tipe parameter** dari `Request $request`
  menjadi `StoreProductRequest $request` → Laravel otomatis menjalankan
  validasi.
- Gunakan `$request->validated()` (bukan `$request->all()`) supaya
  hanya data yang sudah lolos validasi yang disimpan. Ini juga lebih
  aman dari serangan mass assignment.
- **Satu FormRequest bisa dipakai banyak method** (store, update,
  import, dll). Inilah keunggulan utama Cara B.

---

## 14. FAQ Singkat

**Q: Kenapa namanya `StoreProductRequest`, padahal dipakai juga di
`update()`?**
A: Nama `Store` di sini berarti "menyimpan data produk ke database",
baik itu data baru (store) maupun perubahan data lama (update).
Aturannya sama persis, jadi cukup satu file. Nanti kalau aturan
tambah dan edit **berbeda** (misal edit produk boleh mengosongkan
field tertentu), baru kita buat 2 file: `StoreProductRequest` dan
`UpdateProductRequest`.

**Q: Kalau saya lupa mengubah `authorize()` jadi `true`, apa yang
terjadi?**
A: Semua request akan **ditolak** dengan error 403 Forbidden. Jadi
kalau kamu mendapat error 403 padahal tidak ada login, cek dulu
method `authorize()` di FormRequest kamu.

**Q: Bisa tidak pakai FormRequest, tetap pakai Cara A saja?**
A: Bisa. Tapi untuk project nyata, FormRequest lebih recommended
karena:
- Aturan validasi bisa **dipakai ulang**.
- Controller **lebih bersih**.
- Mudah di-**test** terpisah dari controller.

---

> **Berhenti di sini.**
>
> Pada bagian berikutnya kita akan bahas **bagaimana menampilkan
> pesan error di halaman form** supaya user tahu kenapa data mereka
> ditolak. Ini penting karena sampai sekarang, validasi bekerja
> "diam-diam" (data ditolak, tapi user tidak tahu kenapa).
>
> **Apakah kamu ingin lanjut ke langkah berikutnya:
> Tahap 5 — Menampilkan Pesan Error di View?**
