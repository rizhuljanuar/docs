# Bagian 3: Validasi Langsung di Controller (Cara A)

> Modul: A. Level Dasar — Fondasi Laravel, MVC, CRUD
> Topik: Request Validation dengan FormRequest
> Prasyarat: Selesai membaca `01` dan `02`

---

## Tujuan Bagian Ini

Sekarang kamu **mulai praktik**. Tujuan tahap ini:

1. Buka file `ProductController.php` yang sudah kamu buat di materi CRUD.
2. Tambahkan **aturan validasi** di dalam method `store()`.
3. Tambahkan juga di method `update()` (karena edit produk juga butuh validasi).
4. Uji coba dengan input yang salah, dan lihat apa yang terjadi.

> **Penting**: Ini adalah **Cara A** (validasi langsung di controller).
> Tujuannya supaya kamu **merasa berhasil** dulu sebelum pindah ke
> FormRequest di Tahap 4.

---

## 1. Buka File ProductController.php

Buka file ini di editor kamu:

```
app/Http/Controllers/ProductController.php
```

Cari method `store()`. Di materi CRUD sebelumnya, kemungkinan besar
bentuknya seperti ini:

```php
public function store(Request $request)
{
    Product::create($request->all());

    return redirect('/products');
}
```

**Arti kode di atas (sebagai pengingat):**
- `store(Request $request)` — method ini menerima data dari form (lewat `$request`).
- `Product::create($request->all())` — simpan semua data ke tabel `products`.
- `redirect('/products')` — setelah simpan, kembali ke halaman daftar produk.

**Masalah**: Di kode di atas **tidak ada validasi**. User bisa input
apa saja, termasuk nama kosong atau harga minus.

Sekarang kita **tambahkan validasi**.

---

## 2. Tambahkan Validasi di method store()

Ubah method `store()` kamu jadi seperti ini:

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

---

## 3. Penjelasan Kode Per Baris

Mari aku bedah **satu per satu** biar kamu paham fungsi setiap bagian.

### Baris 1: Pemanggilan validasi

```php
$request->validate([ ... ]);
```

**Fungsinya**: Meminta Laravel untuk **mengecek semua data di
dalam `$request`** berdasarkan aturan yang ada di dalam kurung siku `[ ... ]`.

**Analogi**: Ini seperti kamu menyuruh kasir: "Tolong cek semua
paket yang masuk hari ini pakai daftar aturan ini ya."

Kalau **ada satu saja** yang tidak lolos, Laravel otomatis:
- Menghentikan method `store()`.
- Mengembalikan user ke halaman form.
- Membawa pesan error.

Jadi **baris di bawahnya** (`Product::create(...)`) **tidak akan
dijalankan** kalau ada yang gagal validasi.

---

### Baris 2: Aturan untuk field nama

```php
'nama' => 'required',
```

**Fungsinya**: Field `nama` **wajib diisi** (tidak boleh kosong).

**Aturan `required`** artinya:
- User **harus** mengisi field ini.
- Kalau dikosongkan → validasi gagal → muncul error "The nama field is required."

**Analogi**: Kasir bilang "Maaf, nama produk wajib diisi."

---

### Baris 3: Aturan untuk field harga

```php
'harga' => 'required|numeric|min:0',
```

Perhatikan ada **3 aturan** sekaligus, dipisahkan oleh tanda `|` (pipe).

Urutan artinya:

| Aturan   | Arti                                                                |
|----------|---------------------------------------------------------------------|
| `required` | Harga wajib diisi.                                                |
| `numeric`  | Isinya harus angka (boleh desimal, misal `15000.50`).            |
| `min:0`    | Tidak boleh lebih kecil dari 0 (artinya tidak boleh minus).       |

**Kenapa pakai `min:0`?** Karena `numeric` saja masih mengizinkan
angka minus. Kita tambahkan `min:0` supaya harga minimal adalah 0.

**Analogi**: Kasir bilang "Harga wajib angka, dan tidak boleh minus."

---

### Baris 4: Aturan untuk field stok

```php
'stok' => 'required|integer|min:0',
```

Mirip dengan harga, tapi ada satu perbedaan:

| Aturan    | Arti                                                                |
|-----------|---------------------------------------------------------------------|
| `required`| Stok wajib diisi.                                                   |
| `integer` | Harus **bilangan bulat** (1, 2, 3, 100), tidak boleh pecahan.       |
| `min:0`   | Tidak boleh minus.                                                  |

**Kenapa `integer`, bukan `numeric`?**
Karena stok itu jumlah barang. Barang tidak mungkin pecahan. Kamu
tidak bisa punya "2.5 buku".

**Analogi**: Kasir bilang "Stok wajib angka bulat, tidak boleh
pecahan, tidak boleh minus."

---

### Baris 5: Aturan untuk field deskripsi

```php
'deskripsi' => 'nullable|min:10',
```

| Aturan     | Arti                                                                |
|------------|---------------------------------------------------------------------|
| `nullable` | Boleh dikosongkan (tidak wajib).                                    |
| `min:10`   | **Kalau diisi**, minimal 10 karakter.                               |

**Pentingnya `nullable`**: Ini kebalikan dari `required`. Tanda
"boleh kosong" harus **disebut eksplisit** supaya Laravel tidak
menganggap field wajib.

**Kenapa `min:10`?** Supaya user tidak iseng isi "ok" atau "a"
di deskripsi. Kalau user memutuskan mengisi deskripsi, harus
bermakna.

**Analogi**: Kasir bilang "Deskripsi opsional. Tapi kalau diisi,
minimal 10 huruf ya, supaya informatif."

---

## 4. Ringkasan Aturan Validasi yang Sudah Kamu Tulis

| Field      | Aturan                          | Maksud                              |
|------------|---------------------------------|-------------------------------------|
| nama       | `required`                      | Wajib diisi                         |
| harga      | `required|numeric|min:0`        | Wajib, angka, tidak minus           |
| stok       | `required|integer|min:0`        | Wajib, bulat, tidak minus           |
| deskripsi  | `nullable|min:10`               | Boleh kosong, kalau diisi minimal 10 |

---

## 5. Lengkapnya, method store() Sekarang Seperti Ini

```php
// app/Http/Controllers/ProductController.php

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

---

## 6. Tambahkan Juga di method update()

Method `update()` juga butuh validasi karena user bisa **mengubah
data produk lewat form edit**. Tanpa validasi, user bisa
mengedit nama produk jadi kosong atau harga jadi minus.

Buka method `update()` kamu, tambahkan validasi yang sama:

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

**Catatan**: Kalau method `update()` kamu sudah punya struktur agak
beda (misalnya pakai `Route::model` binding), **tidak masalah**. Yang
penting: **letakkan `$request->validate(...)` di baris pertama method
sebelum proses update dilakukan**.

---

## 7. Cara Menguji Validasi

Sekarang waktunya **mencoba sendiri**. Ikuti langkah-langkah ini:

### Langkah 1: Pastikan server jalan

Di terminal, jalankan:

```bash
php artisan serve
```

Server Laravel biasanya aktif di `http://localhost:8000`.

---

### Langkah 2: Buka form tambah produk

Buka browser, pergi ke:

```
http://localhost:8000/products/create
```

(atau URL form tambah produk sesuai route kamu)

---

### Langkah 3: Coba input SALAH (uji validasi menolak)

Isi form begini:

| Field      | Isi yang salah         | Harusnya                          |
|------------|------------------------|-----------------------------------|
| nama       | (kosongkan)            | Harus muncul error                |
| harga      | `-5000`                | Harus muncul error "min 0"        |
| stok       | `2.5`                  | Harus muncul error "harus integer"|
| deskripsi  | `ok`                   | Harus muncul error "min 10"       |

Klik tombol **Simpan**.

**Yang seharusnya terjadi**:

- Kamu **dikembalikan** ke halaman form.
- Form **tidak tersimpan** ke database.
- Tapi... **error-nya belum terlihat**! Kenapa? Karena **form kamu
  belum menampilkan error**. Itu akan kita bahas di **Tahap 5**.

> Tenang kalau belum kelihatan pesan errornya. Yang penting sekarang:
> **data salah tidak masuk ke database**. Itu bukti validasi bekerja.

Cek database atau halaman daftar produk. Produk dengan data salah
**tidak boleh muncul**.

---

### Langkah 4: Coba input BENAR (uji validasi menerima)

Isi form begini:

| Field      | Isi yang benar         |
|------------|------------------------|
| nama       | `Buku Tulis`           |
| harga      | `5000`                 |
| stok       | `20`                   |
| deskripsi  | `Buku tulis 50 lembar` |

Klik **Simpan**.

**Yang seharusnya terjadi**:
- Kamu diarahkan ke halaman daftar produk.
- Produk baru **muncul** di daftar.
- Validasi lolos, data tersimpan.

---

## 8. Cara Melihat Pesan Error (Preview untuk Tahap 5)

Sebelum kita resmi bahas di Tahap 5, ini **preview cepat** cara
melihat pesan error di file Blade:

Di file view form kamu (misal `resources/views/products/create.blade.php`),
tambahkan **di atas form**:

```blade
@if ($errors->any())
    <div style="color: red;">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

**Fungsinya**:
- `$errors->any()` — cek apakah ada error.
- `$errors->all()` — ambil semua pesan error.
- Ditampilkan sebagai **daftar merah** di atas form.

Kita akan bahas ini **lebih lengkap dan rapi** di Tahap 5.

---

## 9. Latihan Mandiri (Sebelum Lanjut)

Supaya makin paham, coba hal berikut sebelum lanjut Tahap 4:

1. **Hapus salah satu aturan**, misalnya hapus `min:0` di harga.
   Lalu coba input `harga = -100`. Apa yang terjadi? Kenapa?
2. **Ubah `min:10` jadi `min:50`** di deskripsi. Lalu coba input
   deskripsi pendek. Apa yang terjadi?
3. **Coba aturan baru**: tambahkan `'nama' => 'required|min:3'`
   di field nama. Coba input nama dengan 1 huruf. Apa yang terjadi?

Latihan ini bikin kamu **merasakan sendiri** bagaimana aturan
validasi mempengaruhi data yang diterima atau ditolak.

---

## 10. Ringkasan Bagian 3

- **Validasi Cara A** = nulis `$request->validate([...])` **langsung
  di dalam method controller**.
- Aturan ditulis dalam bentuk **array asosiatif**: key = nama field,
  value = aturan dipisah `|`.
- Aturan wajib untuk produk kita: `required`, `numeric`, `integer`,
  `min:0`, `nullable`, `min:10`.
- Validasi **diletakkan di awal method** (baik `store()` maupun `update()`),
  **sebelum** proses simpan/update.
- Kalau validasi **gagal**, Laravel otomatis **kembalikan user ke form**.
- **Masalah Cara A**: kalau validasi dipakai di banyak tempat (store
  + update + import), kita harus nulis ulang terus-terusan. Inilah
  alasan kita akan pindah ke **Cara B (FormRequest)** di Tahap 4.

---

> **Berhenti di sini.**
>
> Pada bagian berikutnya kita akan pindah ke **Cara B**: membuat
> **FormRequest** terpisah dengan perintah `php artisan make:request`.
> Ini cara yang lebih **rapi dan bisa dipakai ulang**, dan ini adalah
> **tujuan akhir** dari modul validasi ini.
>
> **Apakah kamu ingin lanjut ke langkah berikutnya:
> Tahap 4 — Membuat FormRequest terpisah (Cara B)?**
