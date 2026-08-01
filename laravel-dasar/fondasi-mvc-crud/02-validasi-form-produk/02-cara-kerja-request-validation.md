# Bagian 2: Cara Kerja Request Validation di Laravel

> Modul: A. Level Dasar — Fondasi Laravel, MVC, CRUD
> Topik: Request Validation dengan FormRequest
> Prasyarat: Selesai membaca `01-apa-itu-validasi-form-produk.md`

---

## Tujuan Bagian Ini

Di bagian sebelumnya kita sudah paham:

> Validasi itu seperti **kasir toko** yang mengecek dan menolak data aneh.

Sekarang kita pelajari **alurnya** secara teknis di Laravel:

> Data form → lewat jalan apa saja → sampai ke database?

Tenang, kita pakai **analogi**, baru di akhir baru ada sedikit kode
untuk nunjukin letaknya.

---

## 1. Analogi: Pengiriman Paket ke Gudang

Bayangkan data produk yang user input di form itu seperti
**paket yang mau dikirim ke gudang toko kamu**.

Alurnya begini:

```
[Pelanggan]  →  [Kurir]  →  [Satpam Gerbang]  →  [Gudang]
   ↑              ↑              ↑                  ↑
 user isi    kirim lewat      cek & tolak        simpan ke
 form        internet HTTP    paket aneh          database
```

Di Laravel, peran-peran ini punya nama resmi:

| Peran di analogi | Nama resmi di Laravel        | Fungsinya                                   |
|------------------|------------------------------|---------------------------------------------|
| Pelanggan        | **User / Browser**           | Yang isi form dan klik "Simpan"             |
| Kurir            | **HTTP Request**             | Pembawa data dari browser ke server         |
| Satpam gerbang   | **Validation**               | Pengecek data: layak atau ditolak           |
| Gudang           | **Database**                 | Tempat penyimpanan akhir                    |
| Pekerja gudang   | **Controller**               | Yang menerima paket dan memasukkan ke gudang|

**Validasi** itu posisinya di **satpam gerbang**.

> Sebelum controller (pekerja gudang) menyimpan data,
> **satpam** harus periksa dulu. Kalau aneh, **tolak** dan suruh
> pelanggan perbaiki. Kalau oke, **lepasin lewat** dan disimpan.

---

## 2. Alur Asli di Laravel (Step by Step)

Sekarang kita ganti analogi jadi **istilah asli Laravel**, tetap pakai
contoh **produk** (`name`, `price`, `stock`, `description` yang sudah
dibuat di **Materi 1: CRUD Data Produk**).

```
1. User buka halaman form tambah produk
       ↓
2. User isi: name="Buku", price=-5000, stock="", description="bagus"
       ↓
3. User klik tombol "Simpan"
       ↓
4. Browser mengirim data (ini disebut HTTP REQUEST)
   lewat "jalur" yang sudah kamu daftarkan di routes/web.php
       ↓
5. Data masuk ke CONTROLLER, biasanya ke method store()
       ↓
6. SEBELUM controller menyimpan, ada VALIDASI yang ngecek data
       ↓
   ├─ Kalau VALID (semua sesuai aturan) → lanjut disimpan ke DB
   └─ Kalau TIDAK VALID → proses berhenti, user dikirim balik
                            ke form + pesan error
```

Hal **paling penting** yang harus kamu ingat:

> **Validasi itu terjadi DI ANTARA request masuk dan database.**
> Bukan di database, bukan di view, tapi **di controller**
> (atau di **kelas khusus** yang dipanggil controller, yaitu FormRequest).

---

## 3. Di Mana Letak Validasi di Kode Laravel?

Ada **2 tempat utama** Laravel menyuruh kamu nulis aturan validasi:

### Cara A: Langsung di dalam Controller

Kamu nulis aturan validasi **di dalam method `store()`** controller
produk kamu. Ini cara **paling cepat dan gampang** untuk belajar.

Lihat lokasinya di kode:

```php
// app/Http/Controllers/ProductController.php

public function store(Request $request)
{
    // ↓↓↓ VALIDASI DI SINI (cara A: langsung di controller)
    $request->validate([
        'name'        => 'required',
        'price'       => 'required|numeric|min:0',
        'stock'       => 'required|integer|min:0',
        'description' => 'nullable|min:10',
    ]);

    // baru di sini data disimpan ke database
    Product::create($request->all());

    return redirect('/products');
}
```

**Cara A cocok untuk:**
- Belajar pertama kali
- Validasi sederhana
- Form yang aturannya jarang dipakai ulang

**Cara A kurang cocok untuk:**
- Controller jadi **panjang dan berantakan** kalau aturan banyak
- Aturan validasi **tidak bisa dipakai ulang** di tempat lain

---

### Cara B: Pakai FormRequest (kelas terpisah)

Laravel menyediakan **file khusus** untuk simpan aturan validasi.
File ini disebut **FormRequest**.

Idenya sederhana:

> Daripada aturan validasi numpuk di controller, **kita pindahkan
> ke ruangan khusus**. Ruangan itu = FormRequest.

```
app/
└── Http/
    ├── Controllers/
    │   └── ProductController.php   ← singkat, hanya logika bisnis
    └── Requests/                    ← ruangan khusus aturan validasi
        └── StoreProductRequest.php  ← aturan untuk tambah produk
```

Di dalam `StoreProductRequest.php`, kamu isi **satu method** bernama
`rules()` yang isinya daftar aturan validasi:

```php
// app/Http/Requests/StoreProductRequest.php

public function rules()
{
    return [
        'name'        => 'required',
        'price'       => 'required|numeric|min:0',
        'stock'       => 'required|integer|min:0',
        'description' => 'nullable|min:10',
    ];
}
```

Lalu di controller, alih-alih kamu pakai `Request $request`, kamu
pakai **`StoreProductRequest $request`**:

```php
// app/Http/Controllers/ProductController.php

public function store(StoreProductRequest $request)
{
    // ↑↑↑ Laravel otomatis jalankan validasi SEBELUM masuk sini

    // Kalau sampai sini, berarti data sudah PASTI valid
    Product::create($request->validated());

    return redirect('/products');
}
```

**Kuncinya**: Laravel **otomatis** menjalankan validasi saat
`StoreProductRequest` dipakai sebagai tipe parameter. Kamu **tidak
perlu** memanggil `$request->validate()` lagi.

**Cara B cocok untuk:**
- Aturan validasi agak panjang
- Aturan dipakai ulang (misal: tambah produk & edit produk)
- Controller tetap bersih dan fokus ke logika bisnis

---

## 4. Apa yang Terjadi Saat Validasi Gagal?

Ini bagian yang sering bikin bingung pemula, jadi perhatikan baik-baik.

Saat Laravel mendeteksi data **tidak valid** (misal price = -5000),
secara default Laravel **otomatis** melakukan hal berikut:

1. **Menghentikan eksekusi** method `store()`.
2. **Tidak menyimpan apa-apa** ke database.
3. **Mengembalikan user ke halaman form sebelumnya**.
4. **Membawa pesan error** supaya bisa ditampilkan di form.
5. **Mengisi ulang (old input)** field-field yang sudah diisi user,
   supaya user tidak perlu ngetik ulang semuanya.

Semua ini terjadi **otomatis** tanpa kamu harus nulis kode tambahan.
Itulah salah satu kelebihan Laravel untuk pemula.

### Analogi ulang (pakai kasir toko):

- Kasir cek barang → ketemu barang aneh → **kasir bilang ke pelanggan**
  "Maaf, harganya gak boleh minus, tolong perbaiki ya."
- Kasir **simpan sementara** data yang sudah benar, supaya pelanggan
  tinggal perbaiki bagian yang salah saja.

Dalam Laravel:
- "Bilang ke pelanggan" = **flash pesan error** ke session.
- "Simpan sementara yang benar" = **old input**.

---

## 5. Ringkasan 1 Kalimat untuk Cara A vs Cara B

| Pertanyaan | Cara A (Controller)       | Cara B (FormRequest)                  |
|------------|---------------------------|---------------------------------------|
| Nulis di mana? | Di dalam `store()`     | Di file terpisah `StoreProductRequest.php` |
| Kapan dipakai? | Validasi sederhana     | Validasi panjang / dipakai ulang      |
| Ditunggangi oleh? | Controller langsung  | Laravel otomatis, **tanpa panggil manual** |
| Bikin controller rame? | Ya, bisa       | Tidak, controller tetap bersih        |

> Tujuan akhir kita di modul ini: pakai **Cara B (FormRequest)**
> karena itu best practice untuk aplikasi Laravel nyata.
> Tapi kamu akan **coba Cara A dulu** di bagian selanjutnya biar
> paham dasarnya, lalu pindah ke Cara B.

---

## 6. Istilah Penting yang Harus Kamu Ingat

| Istilah          | Artinya (bahasa awam)                                       |
|------------------|-------------------------------------------------------------|
| **Request**      | "Paket" yang dibawa kurir (browser) berisi data dari form   |
| **Validation**   | Satpam gerbang yang ngecek paket                            |
| **Rules**        | Aturan-aturan yang dipakai satpam (required, numeric, dll)  |
| **Controller**   | Pekerja gudang yang menerima paket yang sudah lolos         |
| **FormRequest**  | Ruangan khusus tempat menyimpan rules + pesan error         |
| **Old input**    | Data form yang sudah diisi user, dikembalikan supaya tidak  |
|                  | ketik ulang saat validasi gagal                             |
| **Flash error**  | Pesan error satu kali yang dikirim ke form saat gagal       |

---

## 7. Yang Sudah Kamu Pahami Setelah Bagian Ini

- Data dari form **mengalir lewat HTTP Request** menuju controller.
- **Validasi** terjadi **di controller** (Cara A) atau **di FormRequest**
  yang dipanggil controller (Cara B).
- Saat validasi **gagal**, Laravel **otomatis** kembalikan user ke form
  beserta pesan error dan input lama.
- FormRequest = **kelas terpisah** tempat menyimpan rules validasi
  supaya controller tetap rapi.

> **Belum ada kode yang kamu tulis sendiri di bagian ini.**
> Tujuan bagian ini cuma satu: kamu **tahu alurnya** sebelum praktik.

---

> **Berhenti di sini.**
>
> Pada bagian berikutnya kita akan **mulai praktik** dengan cara
> paling sederhana: **menulis aturan validasi langsung di controller
> ProductController** (Cara A). Tujuannya biar kamu **merasa berhasil**
> dulu sebelum pindah ke FormRequest.
>
> **Apakah kamu ingin lanjut ke langkah berikutnya:
> Tahap 3 — Validasi langsung di Controller (Cara A)?**
