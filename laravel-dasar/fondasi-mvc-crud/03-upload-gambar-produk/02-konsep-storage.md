# Upload Gambar Produk — Tahap 2: Konsep Storage di Laravel

> Bagian dari: Laravel Dasar — Fondasi MVC & CRUD
> Topik: 3. Upload Gambar Produk
> Tahap: 2 dari 5 — **Konsep Storage Abstraction** (masih bertahap, belum praktik penuh)

---

## 1. Masalah yang Ingin Diselesaikan

Di tahap 1 kita sudah tahu: file dari user **harus divalidasi** lalu **disimpan
di server**. Pertanyaannya sekarang:

> "Disimpan di **mana**? Dan bagaimana cara **menampilkannya kembali** di halaman web?"

Kalau kita sembarangan menaruh file (misal di `public/gambar/`), muncul masalah:

- Susah dipindah-pindah (misal nanti mau pindah ke Amazon S3 atau cloud).
- Susah diatur hak aksesnya (mana yang boleh diakses publik, mana yang private).
- Bercampur dengan kode aplikasi, jadi berantakan.

Laravel punya solusi: **Storage Abstraction**.

---

## 2. Apa Itu Storage Abstraction?

**Abstraksi** = kita **tidak peduli** detail teknis di belakangnya, kita hanya
pakai "perintah yang sama" walau lokasi penyimpanannya berbeda.

Analogi:

> Bayangkan kamu pakai ** remot AC **. Kamu cuma tekan tombol "hidup", AC nyala.
> Kamu tidak perlu tahu AC-nya merek apa, daya berapa, atau pakai Freon jenis
> apa. Tombolnya **sama**, efeknya **sama**.

Begitu juga di Laravel. Kode yang sama:

```php
Storage::disk('public')->put('products/sepatu.jpg', $file);
```

bisa dipakai untuk:

- Simpan di **lokal** (`storage/app/public`).
- Simpan di **Amazon S3** (cloud).
- Simpan di **Google Cloud Storage**.
- Simpan di **DigitalOcean Spaces**.

**Cukup ganti konfigurasi**, kode aplikasi tidak perlu diubah. Itulah kekuatan
abstraksi.

---

## 3. Istilah Penting yang Harus Dikuasai

Ada 3 istilah yang sering bikin bingung pemula. Kita bedakan satu per satu.

### a. **Disk**

"Disk" = **nama lokasi penyimpanan** yang sudah didefinisikan di
`config/filesystems.php`. Laravel punya beberapa disk bawaan:

- `local` → folder `storage/app` (private, tidak bisa diakses publik).
- `public` → folder `storage/app/public` (bisa diakses publik lewat browser).
- `s3` → Amazon S3 (cloud).

> Untuk gambar produk, kita pakai disk **`public`** karena fotonya harus
> tampil di halaman toko (diakses orang).

### b. **`storage/app/public` vs `public/storage`** (PENTING!)

Ini sumber kebingungan klasik. Perhatikan baik-baik:

| Lokasi                  | Fungsi                                          | Bisa diakses browser? |
| ----------------------- | ----------------------------------------------- | --------------------- |
| `storage/app/public`    | Tempat Laravel benar-benar menyimpan file       | **TIDAK** (default)   |
| `public/storage`        | **Symlink** (jalan pintas) ke folder di atas   | **YA**                |

Laravel menyimpan file di `storage/app/public` (terpisah dari kode app).
Tapi browser hanya bisa membuka file di dalam folder `public/` (akar web).
Solusinya: bikin **symlink** (jalan pintas) dari `public/storage`
→ `storage/app/public`.

Perintahnya:

```bash
php artisan storage:link
```

Cukup **sekali** jalankan ini di setiap instalasi Laravel, dan gambar yang
disimpan di `storage/app/public/products/x.jpg` bisa dibuka lewat URL
`http://toko.test/storage/products/x.jpg`.

> **Ponytail:** Lihat detail symlink ini di `config/filesystems.php`,
> bagian `links`. Jangan diubah kecuali tahu apa yang kamu lakukan.

### c. **`Storage::url()`**

Fungsi bawaan Laravel untuk **menghasilkan URL publik** dari sebuah file yang
disimpan di disk. Contoh:

```php
Storage::disk('public')->url('products/sepatu.jpg');
// hasil: http://toko.test/storage/products/sepatu.jpg
```

Kenapa pakai ini, bukan nulis URL manual? Karena:

- Kalau nanti pindah ke S3, URL-nya **otomatis** menyesuaikan.
- Lebih rapi, tidak hard-code nama domain.

---

## 4. Alur Lengkap (Konseptual)

```
[ User upload sepatu.jpg ]
            │
            ▼
   [ Form di halaman web ]
            │
            ▼
   [ Controller menerima request ]
            │
            ▼
   [ VALIDASI file: image, ukuran, dst ]   ← Tahap 3
            │
            ▼
   [ Storage::disk('public')->put(...) ]    ← simpan ke storage/app/public
            │
            ▼
   [ Simpan PATH gambar ke database ]        ← contoh: products/gambar/sepatu.jpg
            │
            ▼
   [ Tampilkan di halaman: Storage::url() ]  ← Tahap 4
```

Yang penting diingat: yang disimpan di database **bukan file gambarnya**, tapi
**path-nya saja** (lokasi string). File aslinya ada di storage.

---

## 5. Config: `config/filesystems.php` (Sekilas)

File inilah "daftar disk" yang tersedia. Isinya kurang lebih:

```php
'disks' => [

    'local' => [
        'driver' => 'local',
        'root'   => storage_path('app/private'),
        'throw'  => false,
    ],

    'public' => [
        'driver'     => 'local',
        'root'       => storage_path('app/public'),
        'url'        => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

    's3' => [
        'driver' => 's3',
        // ...key AWS...
    ],
],
```

Pemula cukup tahu: **disk `public`** → folder `storage/app/public` → akses via
`/storage/...` (misal `/storage/products/x.jpg`). Sisanya biarkan default.

---

## 6. Ringkasan Tahap 2

- **Storage abstraction** = kode sama, lokasi bisa beda (lokal / S3 / dll).
- **Disk** = nama lokasi penyimpanan di `config/filesystems.php`.
- Untuk gambar produk, pakai disk **`public`**.
- File disimpan di `storage/app/public`, diakses lewat symlink `public/storage`.
- Jalankan **sekali**: `php artisan storage:link`.
- Gunakan **`Storage::url('path')`** untuk menghasilkan URL gambar.
- Database hanya menyimpan **path**, bukan file-nya.

---

## 7. Cek Pemahaman (jawab di kepala)

1. Kenapa gambar produk tidak disimpan langsung di folder `public/`?
2. Apa beda `storage/app/public` dengan `public/storage`?
3. Mengapa kita perlu menjalankan `php artisan storage:link`?
4. Apa yang disimpan di database: file gambar atau path-nya?

> Jawaban singkat di kepala cukup. Kalau sempat bingung, kita bahas di tahap
> berikutnya.

---

> **Pertanyaan untuk kamu:** Sudah cukup jelas konsep storage?
> Mau lanjut ke **Tahap 3 — Form Upload & Validasi File** (mulai menulis
> kode: rule `image`, `mimes`, `max:`), atau ulas ulang tahap 2 dulu?
