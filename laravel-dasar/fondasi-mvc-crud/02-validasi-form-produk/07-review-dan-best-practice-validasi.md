# Bagian 7: Review, Best Practice, dan Custom Pesan Error

> Modul: A. Level Dasar — Fondasi Laravel, MVC, CRUD
> Topik: Request Validation dengan FormRequest
> Prasyarat: Selesai membaca `01` sampai `06`

---

## Tujuan Bagian Ini (Terakhir)

Ini adalah **penutup modul Validasi Form Produk**. Tiga hal yang akan
kamu dapatkan:

1. **Rangkuman** seluruh materi Tahap 1-6 dalam satu halaman.
2. **Cheat sheet aturan validasi** Laravel yang paling sering dipakai.
3. **Cara kustomisasi pesan error ke bahasa Indonesia** (2 cara).
4. **Best practice** validasi di aplikasi nyata.

Setelah ini, kamu **lulus** modul Validasi Form Produk dan siap ke
materi Laravel dasar berikutnya.

---

## 1. Rangkuman Modul dalam Satu Halaman

### Masalah yang Dipecahkan

Tanpa validasi, user bisa input data kosong, harga minus, stok pecahan,
atau data aneh yang merusak laporan dan kepercayaan pengguna.

### Solusi yang Dipakai

**FormRequest** di Laravel. File terpisah yang berisi aturan validasi,
dipakai otomatis oleh controller.

### 7 Tahapan yang Sudah Dilalui

| # | Tahap                                            | Output                              |
|---|--------------------------------------------------|-------------------------------------|
| 1 | Konsep validasi + analogi kasir toko             | Paham **kenapa** validasi penting   |
| 2 | Cara kerja Request Validation (alur data)        | Paham **alurnya** di Laravel        |
| 3 | Cara A: validasi langsung di controller          | `$request->validate([...])`         |
| 4 | Cara B: FormRequest terpisah                     | `StoreProductRequest.php`           |
| 5 | Tampilkan pesan error + `old()` di Blade         | `@error`, `$errors->any()`, `old()` |
| 6 | Integrasi penuh + uji end-to-end                 | CRUD lengkap dengan validasi        |
| 7 | Review, cheat sheet, custom pesan Indonesia      | (modul ini)                         |

### Struktur Akhir File

```
app/Http/
├── Controllers/ProductController.php     ← pakai StoreProductRequest
└── Requests/StoreProductRequest.php      ← rules() di sini

resources/views/products/
├── create.blade.php                      ← form + @error + old()
└── edit.blade.php                        ← form + @error + old('f', $p->f)
```

### Konsep Kunci (1 kalimat)

> **FormRequest = ruangan khusus untuk aturan validasi.** Controller
> cukup **menyebut namanya** sebagai tipe parameter, dan Laravel
> otomatis menjalankan validasi sebelum method controller dijalankan.

---

## 2. Cheat Sheet: Aturan Validasi Laravel yang Sering Dipakai

Ini daftar **rules Laravel** yang paling sering dipakai di aplikasi
nyata. Simpan sebagai catatan jangka panjang.

### Aturan Dasar (Kehadiran / Ketidakhadiran)

| Rule        | Arti                                              | Contoh                |
|-------------|---------------------------------------------------|-----------------------|
| `required`  | Wajib diisi (tidak boleh kosong)                 | `'nama' => 'required'`|
| `nullable`  | Boleh kosong                                       | `'catatan' => 'nullable\|string'` |
| `present`   | Field harus ada di request (boleh kosong)        | `'agree' => 'present'`|
| `filled`    | Kalau ada, tidak boleh kosong                    | -                     |

### Aturan String dan Teks

| Rule              | Arti                                                   |
|-------------------|--------------------------------------------------------|
| `string`          | Harus berupa string (teks)                             |
| `min:10`          | Minimal 10 karakter (untuk string) / 10 angka (untuk number) |
| `max:100`         | Maksimal 100 karakter / nilai maksimum                 |
| `alpha`           | Hanya huruf                                            |
| `alpha_num`       | Huruf dan angka                                        |
| `email`           | Format email valid (misal `a@b.com`)                   |
| `confirmed`       | Harus ada field `xxx_confirmation` yang nilainya sama (untuk password) |

### Aturan Angka

| Rule        | Arti                                              |
|-------------|---------------------------------------------------|
| `numeric`   | Harus angka (boleh desimal)                       |
| `integer`   | Harus bilangan bulat (tidak boleh pecahan)        |
| `min:0`     | Nilai minimal 0                                   |
| `max:100`   | Nilai maksimal 100                                |
| `between:1,100` | Nilai antara 1 sampai 100                    |
| `gt:field`  | Lebih besar dari nilai field lain                 |
| `lt:field`  | Lebih kecil dari nilai field lain                 |

### Aturan Tanggal

| Rule            | Arti                                              |
|-----------------|---------------------------------------------------|
| `date`          | Harus tanggal valid                               |
| `before:today`  | Tanggal sebelum hari ini                          |
| `after:today`   | Tanggal setelah hari ini                          |
| `before_or_equal:tanggal_selesai` | Validasi rentang tanggal |

### Aturan Unik dan Database

| Rule                          | Arti                                       |
|-------------------------------|--------------------------------------------|
| `unique:products,nama`        | Nilai field harus unik di kolom `nama` tabel `products` |
| `unique:products,nama,1,id`   | Unik, tapi abaikan baris dengan ID 1 (untuk update) |
| `exists:categories,id`        | Nilai harus ada di kolom `id` tabel `categories` |

### Aturan Pilihan

| Rule                          | Arti                                       |
|-------------------------------|--------------------------------------------|
| `in:aktif,nonaktif`           | Harus salah satu dari daftar              |
| `not_in:0,1`                  | Tidak boleh salah satu dari daftar        |
| `boolean`                     | Harus true/false (atau 0/1)               |

### Aturan Format dan File

| Rule            | Arti                                              |
|-----------------|---------------------------------------------------|
| `url`           | Harus URL valid                                   |
| `ip`            | Harus alamat IP valid                             |
| `json`          | Harus JSON valid                                  |
| `image`         | Harus gambar (jpg, png, bmp, gif, svg, webp)      |
| `mimes:jpg,png` | Harus file dengan tipe tertentu                   |
| `mimetypes:image/jpeg` | Harus MIME type tertentu                  |
| `file`          | Harus file yang di-upload                         |
| `size:1024`     | Ukuran tepat 1024 kilobyte                        |

### Aturan Gabungan (Composite)

| Rule            | Arti                                              |
|-----------------|---------------------------------------------------|
| `array`         | Harus array                                       |
| `required_if:field,value` | Wajib kalau field lain punya nilai tertentu |
| `required_unless:field,value` | Wajib kecuali field lain punya nilai tertentu |
| `required_with:field` | Wajib kalau field lain ada                 |
| `prohibited`    | Tidak boleh diisi                                 |

> **Tip**: Kamu bisa **menggabungkan** beberapa aturan dengan tanda
> `|`, contoh: `'harga' => 'required|numeric|min:0'`. Atau pakai
> **array** untuk readability:
> ```php
> 'harga' => ['required', 'numeric', 'min:0'],
> ```

---

## 3. Custom Pesan Error ke Bahasa Indonesia

Pesan default Laravel berbahasa Inggris ("The nama field is required.").
Untuk aplikasi Indonesia, kita ingin pesan berbahasa Indonesia.

Ada **2 cara**, masing-masing cocok untuk situasi berbeda.

### Cara 1: Lewat method messages() di FormRequest

Cocok kalau kamu ingin **pesan khusus untuk satu FormRequest**.

Edit `app/Http/Requests/StoreProductRequest.php`, tambahkan method
`messages()`:

```php
class StoreProductRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'nama'      => 'required',
            'harga'     => 'required|numeric|min:0',
            'stok'      => 'required|integer|min:0',
            'deskripsi' => 'nullable|min:10',
        ];
    }

    public function messages(): array
    {
        return [
            // Pesan untuk aturan spesifik di field spesifik
            'nama.required'      => 'Nama produk wajib diisi.',
            'harga.required'     => 'Harga wajib diisi.',
            'harga.numeric'      => 'Harga harus berupa angka.',
            'harga.min'          => 'Harga tidak boleh minus.',
            'stok.required'      => 'Stok wajib diisi.',
            'stok.integer'       => 'Stok harus bilangan bulat.',
            'stok.min'           => 'Stok tidak boleh minus.',
            'deskripsi.min'      => 'Deskripsi minimal 10 karakter.',
        ];
    }
}
```

**Format kunci**: `'field.aturan' => 'pesan'`.

Setelah ini, semua pesan error untuk produk akan tampil dalam bahasa
Indonesia di form.

### Cara 2: Lewat file bahasa (untuk SEMUA form)

Cocok kalau kamu ingin pesan Indonesia berlaku untuk **seluruh aplikasi**.

#### Langkah 1: Publikasikan file bahasa

Jalankan di terminal:

```bash
php artisan lang:publish
```

Ini akan membuat folder `lang/en/` di project kamu.

#### Langkah 2: Buat folder bahasa Indonesia

Salin folder `lang/en` jadi `lang/id`:

```bash
cp -r lang/en lang/id
```

#### Langkah 3: Ubah bahasa default di config/app.php

Buka `config/app.php`, cari baris `'locale'`:

```php
'locale' => 'id',
```

Ini mengubah bahasa default aplikasi jadi Indonesia.

#### Langkah 4: Edit file lang/id/validation.php

Buka `lang/id/validation.php`. Di sana ada banyak baris pesan. Edit
yang penting, contoh:

```php
return [
    'required' => 'Kolom :attribute wajib diisi.',
    'min'      => [
        'numeric' => 'Kolom :attribute harus minimal :min.',
        'string'  => 'Kolom :attribute harus minimal :min karakter.',
    ],
    'numeric'  => 'Kolom :attribute harus berupa angka.',
    'integer'  => 'Kolom :attribute harus berupa bilangan bulat.',
    // ...
];
```

**Penjelasan placeholder**:
- `:attribute` → otomatis diganti jadi nama field (misal "nama").
- `:min`, `:max`, `:value` → otomatis diganti sesuai nilai aturan.

> **Bonus**: Laravel sudah punya **file bahasa Indonesia resmi** di
> [github.com/Laravel-Lang/lang](https://github.com/Laravel-Lang/lang).
> Tinggal download file `id/validation.php` dan taruh di `lang/id/`.
> Lebih cepat dari pada translate manual.

### Mana yang Dipakai?

| Situasi                                   | Cara yang dipakai        |
|-------------------------------------------|--------------------------|
| Pesan khusus untuk 1 FormRequest tertentu | `messages()` di FormRequest |
| Pesan seragam untuk seluruh aplikasi      | File `lang/id/validation.php` |
| Ingin keduanya                            | Bisa dipakai bersamaan. `messages()` menimpa file bahasa. |

> **Best practice**: pakai file bahasa (`lang/id/`) sebagai default.
> Pakai `messages()` hanya untuk pesan **benar-benar khusus** yang
> berbeda dari default.

---

## 4. Custom Nama Attribute (Label yang Lebih Ramah)

Secara default, pesan error menampilkan nama field (`nama`, `harga`).
Tapi kamu mungkin ingin tampil sebagai "Nama Produk", "Harga", dll.

Ada **2 cara**:

### Cara 1: method attributes() di FormRequest

```php
public function attributes(): array
{
    return [
        'nama'  => 'Nama Produk',
        'harga' => 'Harga',
        'stok'  => 'Stok',
        'deskripsi' => 'Deskripsi',
    ];
}
```

Setelah ini, pesan error akan pakai label tersebut:
"Kolom **Nama Produk** wajib diisi." (bukan "Kolom nama wajib diisi.").

### Cara 2: file lang/id/validation.php

Di bagian bawah file `lang/id/validation.php`, ada section `attributes`:

```php
'attributes' => [
    'nama'  => 'Nama Produk',
    'harga' => 'Harga',
    'stok'  => 'Stok',
    'deskripsi' => 'Deskripsi',
],
```

---

## 5. Best Practice Validasi di Aplikasi Nyata

### 5.1 Kapan Pakai FormRequest vs $request->validate()?

| Situasi                                            | Pakai           |
|----------------------------------------------------|-----------------|
| Validasi sederhana, 1-3 aturan, dipakai 1 tempat   | `$request->validate()` |
| Validasi lebih dari 5 field                        | FormRequest     |
| Aturan dipakai di banyak method (store + update)   | FormRequest     |
| Aplikasi tim / kode besar                          | FormRequest     |
| Belajar / prototipe cepat                          | `$request->validate()` |

### 5.2 Gunakan $request->validated(), bukan $request->all()

**Selalu** pakai `$request->validated()` saat menyimpan ke database:

```php
// BENAR
Product::create($request->validated());

// KURANG AMAN
Product::create($request->all());
```

**Kenapa?** Mencegah **mass assignment vulnerability**. Field yang
tidak ada di `rules()` akan diabaikan, jadi user tidak bisa
menyusupkan field rahasia (seperti `is_admin`).

### 5.3 Pisahkan FormRequest untuk Store dan Update Jika Beda

Kalau aturan store dan update **berbeda** (misal di update,
`nama` boleh sama dengan nama sebelumnya via `unique` ignore ID),
bikin 2 file:

- `StoreProductRequest.php` → untuk tambah
- `UpdateProductRequest.php` → untuk edit

Kalau **sama persis** (seperti contoh kita di modul ini), cukup 1 file
saja.

### 5.4 Aturan unique di Update: Jangan Lupa Ignore ID

Ini jebakan klasik. Misal kamu punya aturan nama harus unik:

```php
'nama' => 'required|unique:products,nama',
```

Saat **update** produk, produk yang diedit sendiri akan dianggap
"duplikat" dengan dirinya sendiri → error validasi padahal nama
tidak berubah.

Solusi: ignore ID produk yang sedang diupdate:

```php
// Di UpdateProductRequest.php
public function rules(): array
{
    $id = $this->route('product')->id; // ambil ID dari route

    return [
        'nama' => 'required|unique:products,nama,' . $id,
        // ... aturan lain
    ];
}
```

### 5.5 Jangan Percaya Input User (Defense in Depth)

Validasi di Laravel bagus, tapi **jangan berhenti di situ** untuk
aplikasi produksi:

1. **Validasi di database juga** (NOT NULL, CHECK constraint, unique
   index). Ini pertahanan terakhir kalau ada bug di kode.
2. **Escape output di Blade** dengan `{{ }}` (bukan `{!! !!}`) untuk
   cegah XSS.
3. **Gunakan prepared statement** (Eloquent sudah otomatis) untuk
   cegah SQL injection.

### 5.6 Tambahkan Fillable di Model

Pastikan model `Product` punya `$fillable` agar `$request->validated()`
berfungsi dengan baik:

```php
// app/Models/Product.php
class Product extends Model
{
    protected $fillable = ['nama', 'harga', 'stok', 'deskripsi'];
}
```

Tanpa `$fillable`, `Product::create()` akan **mengabaikan semua field**
mass-assignment (security feature Laravel yang namanya "mass
assignment protection").

---

## 6. Rangkuman Akhir Modul (1 Kalimat)

> **Validasi adalah penjaga gerbang aplikasi.** Di Laravel, penjaga
> terbaik itu **FormRequest**: file terpisah berisi `rules()`,
> dipanggil otomatis oleh controller. Pesan error dan old input
> sudah ditangani Laravel secara otomatis, sehingga controller tetap
> bersih dan user dapat feedback yang jelas.

---

## 7. Daftar Istilah Penting (Glosarium)

| Istilah            | Arti                                                           |
|--------------------|----------------------------------------------------------------|
| Validasi           | Proses mengecek data dari form sebelum disimpan                |
| Request            | Data yang dikirim browser ke server (HTTP request)             |
| FormRequest        | File khusus berisi aturan validasi (subclass dari FormRequest) |
| Rules              | Daftar aturan validasi (`required`, `numeric`, dll)            |
| authorize()        | Method di FormRequest untuk cek siapa yang boleh pakai         |
| validated()        | Method request yang ambil hanya field yang lolos validasi      |
| old()              | Fungsi Blade untuk ambil input lama user                       |
| $errors            | Variabel global Blade berisi pesan error                       |
| @error             | Direktif Blade untuk menampilkan error per field               |
| Mass Assignment    | Serangan dengan menyusupkan field ekstra ke form               |
| $fillable          | Property model yang menentukan field mana yang boleh di-mass-assign |
| Fillable / Guarded | Sistem proteksi Eloquent terhadap mass assignment              |
| lang/              | Folder file bahasa Laravel (`lang/id/` untuk Indonesia)        |
| Flash message      | Pesan sekali pakai yang muncul setelah redirect               |

---

## 8. Selamat! Modul Selesai

Kamu sudah menyelesaikan **Materi 2: Validasi Form Produk** dari awal
sampai akhir. Yang sudah kamu kuasai:

- ✅ Konsep **kenapa validasi penting**.
- ✅ **Alur data** dari form ke database di Laravel.
- ✅ Cara **validasi langsung di controller** (Cara A).
- ✅ Cara **FormRequest terpisah** (Cara B - best practice).
- ✅ Cara **menampilkan pesan error** dan **mempertahankan input user**.
- ✅ Cara **mengintegrasikan** validasi ke CRUD produk.
- ✅ **Cheat sheet rules Laravel** untuk jangka panjang.
- ✅ Cara **custom pesan error ke bahasa Indonesia**.

### Yang Bisa Kamu Pelajari Selanjutnya

Modul ini adalah bagian dari **A. Level Dasar - Fondasi Laravel, MVC,
CRUD**. Materi yang biasanya dipelajari setelah Validasi Form Produk:

1. **Relasi Database** (One-to-Many, Many-to-Many) - misal produk
   punya kategori.
2. **Eloquent Relationships** - cara ambil data relasi di Laravel.
3. **Authentication & Authorization** - sistem login, registrasi,
   role user.
4. **Middleware** - filter request sebelum sampai controller.
5. **File Upload** - upload gambar produk (ini butuh rules `image`
   dan `mimes`).
6. **Pagination** - bagi data produk per halaman (kalau produknya
   ratusan).

Untuk sekarang, **istirahat dulu**, lalu coba **reproduksi sendiri**
semua yang sudah kamu pelajari di project kosong (tanpa lihat catatan).
Ini cara terbaik untuk memantapkan pemahaman.

---

## 9. Pesan Penutup

> Belajar Laravel itu seperti belajar memasak. Resep di buku (tutorial)
> bisa dibaca, tapi kamu cuma jadi **chef** kalau kamu sudah masak
> sendiri tanpa lihat resep.
>
> Modul ini cuma **resep**. Sekarang waktunya **masak sendiri**.

Selamat melanjutkan perjalanan belajar Laravel. Semoga cepat jago!

---

**Akhir Modul 2: Validasi Form Produk.**
