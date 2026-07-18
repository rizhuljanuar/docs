# Tahap 5 — Fitur Restore: Mengembalikan Produk dari Sampah ke Daftar Aktif

> Materi: Soft Delete Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi: Mengambil Kembali Barang dari Tempat Sampah

Di tahap 4, kita sudah bisa **membuka tutup tempat sampah** dan melihat isinya. Tapi admin cuma bisa melihat, belum bisa berbuat apa-apa.

Sekarang bayangkan: admin melihat ada **"Kopi Susu Vanilla"** di tong sampah. Ternyata, supplier sudah kirim stok kopi lagi. Produk ini harus **dijual lagi**.

Apa yang harus admin lakukan?

> **Mengambil produk itu dari tong sampah, dan menaruhnya kembali di rak.**

Itulah **restore**. Mengembalikan produk yang sudah di-soft-delete ke daftar produk aktif. Data produk tidak hilang, tidak perlu input ulang, cukup "ditarik keluar dari sampah".

Dalam teknis database, restore artinya: **set kolom `deleted_at` kembali jadi `NULL`**. Sekali `deleted_at` NULL lagi, produk otomatis muncul di query biasa (`Product::all()`, `Product::find($id)`, dst).

---

## 2. Apa yang Terjadi Secara Teknis Saat Restore?

Sebelum restore (produk masih di sampah):

| id | nama | harga | deleted_at |
|---|---|---|---|
| 5 | Kopi Susu Vanilla | 18000 | **2026-07-18 11:02:15** |

Setelah restore:

| id | nama | harga | deleted_at |
|---|---|---|---|
| 5 | Kopi Susu Vanilla | 18000 | **NULL** |

Query SQL yang Laravel jalankan saat restore:

```sql
UPDATE products SET deleted_at = NULL WHERE id = 5;
```

Itu saja. Kolom `deleted_at` dikosongkan. Data lain (nama, harga, stok, deskripsi, kategori, slug, gambar) **tidak diubah sama sekali**. Produk kembali persis seperti sedia kala.

Setelah restore:
- `Product::find(5)` → ketemu lagi.
- `Product::all()` → produk id=5 muncul kembali.
- Halaman `/produk` → produk muncul lagi.
- Halaman `/produk/sampah` → produk hilang dari daftar sampah.

---

## 3. ⚠️ Penting: `find()` Biasa TIDAK Bisa Menemukan Produk Sampah

Sebelum kita mulai koding, ada **jebakan besar** yang sering bikin pemula bingung.

Bayangkan kita mau restore produk id=5. Logika awam:

```php
$product = Product::find(5);  // ❌ SALAH!
$product->restore();
```

Tapi saat dijalankan, `$product` adalah **`null`**. Kenapa?

**Karena produk id=5 sudah di-soft-delete**, dan query biasa (`Product::find(5)`) **tidak menemukan** produk yang `deleted_at`-nya terisi. Dari sudut pandang query biasa, produk itu "sudah tidak ada".

**Solusinya**: kita harus pakai `withTrashed()` dulu supaya produk sampah bisa ditemukan, **baru** kita restore.

```php
$product = Product::withTrashed()->find(5);  // ✅ BENAR
$product->restore();
```

Atau lebih aman pakai `findOrFail` (supaya otomatis 404 kalau tidak ketemu):

```php
$product = Product::withTrashed()->findOrFail(5);  // ✅ PALING AMAN
$product->restore();
```

**Aturan emas restore**: kalau kamu mau cari produk yang sudah di-soft-delete, **selalu** pakai `withTrashed()` (atau `onlyTrashed()`). Tanpa itu, produk tidak akan ketemu.

---

## 4. Rencana 3 Langkah Restore

Kita akan kerjakan restore dalam 3 langkah, mirip seperti tahap-tahap sebelumnya:

1. **Route**: tambah route `POST /produk/{id}/restore`.
2. **Controller**: tambah method `restore($id)` yang pakai `withTrashed()->findOrFail($id)->restore()`.
3. **View**: tambah tombol "Kembalikan" di setiap baris tabel sampah.

---

## 5. Langkah 1: Tambah Route untuk Restore

Buka file `routes/web.php`. Tambahkan satu route baru:

```php
Route::post('/produk/{id}/restore', [ProductController::class, 'restore'])
    ->name('produk.restore');
```

Letakkan di grup route produk yang lain. Contoh posisi lengkap:

```php
Route::get('/produk', [ProductController::class, 'index'])->name('produk.index');
Route::get('/produk/create', [ProductController::class, 'create'])->name('produk.create');
Route::post('/produk', [ProductController::class, 'store'])->name('produk.store');

// Route sampah — HARUS di atas route /produk/{id}
Route::get('/produk/sampah', [ProductController::class, 'trash'])->name('produk.trash');

// Route restore
Route::post('/produk/{id}/restore', [ProductController::class, 'restore'])
    ->name('produk.restore');

Route::get('/produk/{id}', [ProductController::class, 'show'])->name('produk.show');
Route::get('/produk/{id}/edit', [ProductController::class, 'edit'])->name('produk.edit');
Route::put('/produk/{id}', [ProductController::class, 'update'])->name('produk.update');
Route::delete('/produk/{id}', [ProductController::class, 'destroy'])->name('produk.destroy');
```

Penjelasan baris baru:

| Bagian | Arti |
|---|---|
| `Route::post(...)` | Method HTTP-nya **POST** (bukan GET/DELETE) |
| `'/produk/{id}/restore'` | URL dengan parameter ID. Contoh: `/produk/5/restore` |
| `[ProductController::class, 'restore']` | Method yang dijalankan: `restore()` |
| `->name('produk.restore')` | Beri nama route, supaya di view bisa pakai `route('produk.restore', $product->id)` |

### Kenapa Method HTTP `POST`, Bukan `PUT` atau `PATCH`?

Secara teori REST:
- `POST` = buat data baru.
- `PUT/PATCH` = edit data yang ada.
- `DELETE` = hapus.

Restore itu secara teknis **mengubah** `deleted_at` jadi NULL. Bisa dibilang "edit". Jadi sebenarnya `PUT` atau `PATCH` lebih tepat secara teori.

Tapi untuk pemula, **`POST` lebih praktis** karena:
- Form HTML only support GET dan POST secara native (butuh `@method()` untuk PUT/PATCH).
- POST sudah cukup, route-nya jelas, tidak bikin pusing.

Terserah kamu mau pakai `POST` atau `PATCH`. Yang penting **konsisten** di semua tempat. Di materi ini, kita pakai `POST` biar simpel.

---

## 6. Langkah 2: Tambah Method `restore()` di Controller

Buka `app/Http/Controllers/ProductController.php`. Tambahkan method baru:

```php
public function restore($id)
{
    $product = Product::withTrashed()->findOrFail($id);
    $product->restore();

    return redirect()->route('produk.trash')
        ->with('success', "Produk \"{$product->nama}\" berhasil dikembalikan.");
}
```

Penjelasan baris per baris:

### `Product::withTrashed()->findOrFail($id)`

Ini bagian paling penting. Dua lapis method:

- `withTrashed()` = bilang ke Laravel, "Sertakan juga produk yang sudah di-soft-delete dalam pencarian ini."
- `findOrFail($id)` = cari produk berdasarkan ID. Kalau tidak ketemu, otomatis tampilkan error 404.

**Kenapa harus pakai `withTrashed()`?** Karena produk yang ada di sampah **tidak bisa ditemukan** lewat query biasa (seperti dijelaskan di bagian 3). Tanpa `withTrashed()`, `findOrFail($id)` akan return 404.

### `$product->restore()`

Ini method khusus dari trait `SoftDeletes`. Fungsinya: set `deleted_at` jadi `NULL` lagi. Setelah ini, produk "kembali hidup" dari sudut pandang query biasa.

Method `restore()` **hanya bisa dipanggil** pada instance model yang diambil lewat `withTrashed()` atau `onlyTrashed()`. Kalau dipanggil pada instance dari query biasa, akan error karena produk itu sebenarnya belum dihapus.

### `return redirect()->route('produk.trash')`

Setelah berhasil restore, redirect balik ke **halaman sampah** (`produk.trash`). Kenapa ke halaman sampah, bukan ke daftar produk aktif?

Karena konteks admin: dia sedang berada di halaman sampah, mengelola produk sampah. Setelah klik "Kembalikan", biarkan dia tetap di halaman sampah supaya bisa lanjut mengelola produk sampah lain kalau ada.

Kalau kamu mau redirect ke daftar produk aktif, ganti jadi `redirect()->route('produk.index')`. Terserah preferensi UX.

### `->with('success', "Produk \"{$product->nama}\" berhasil dikembalikan.")`

Bawa pesan sukses yang **menyebut nama produk**. Contoh: `Produk "Kopi Susu Vanilla" berhasil dikembalikan.`

Ini bikin admin yakin produk yang benar sudah dikembalikan. Pakai variabel `{$product->nama}` supaya pesan dinamis, tidak generik.

---

## 7. Langkah 3: Tambah Tombol "Kembalikan" di View `trash.blade.php`

Buka `resources/views/products/trash.blade.php`. Kita akan tambah dua hal:

1. Kolom baru **"Aksi"** di tabel.
2. Tombol **"Kembalikan"** di setiap baris.

### Sebelumnya: Pesan Flash Sukses

Tambahkan blok ini di paling atas section content, supaya pesan sukses dari redirect muncul:

```blade
@if(session('success'))
    <div class="bg-green-100 border border-green-400 text-green-700 px-4 py-2 rounded mb-4">
        {{ session('success') }}
    </div>
@endif
```

Penjelasan:

- `session('success')` = ambil pesan sukses yang kita kirim lewat `->with('success', ...)` di controller. Laravel simpan di session sekali (flash), lalu otomatis hilang setelah dibaca.
- Blok `@if` supaya pesan cuma tampil kalau ada, tidak error kalau kosong.

### Tambah Kolom "Aksi" di Header Tabel

Cari `<thead>`, tambahkan satu kolom baru di ujung kanan:

```blade
<thead class="bg-gray-100">
    <tr>
        <th class="border p-2 text-left">ID</th>
        <th class="border p-2 text-left">Nama Produk</th>
        <th class="border p-2 text-left">Harga</th>
        <th class="border p-2 text-left">Kategori</th>
        <th class="border p-2 text-left">Dihapus Pada</th>
        <th class="border p-2 text-left">Aksi</th>  <!-- ← tambahan -->
    </tr>
</thead>
```

### Tambah Tombol "Kembalikan" di Setiap Baris

Cari `<tbody>`, di dalam `@foreach`, tambahkan satu `<td>` baru di akhir:

```blade
<tbody>
    @foreach($trashedProducts as $product)
        <tr>
            <td class="border p-2">{{ $product->id }}</td>
            <td class="border p-2">{{ $product->nama }}</td>
            <td class="border p-2">Rp {{ number_format($product->harga, 0, ',', '.') }}</td>
            <td class="border p-2">{{ $product->kategori }}</td>
            <td class="border p-2">{{ $product->deleted_at->format('d M Y, H:i') }}</td>

            <!-- ← Tambahan: kolom Aksi -->
            <td class="border p-2">
                <form action="{{ route('produk.restore', $product->id) }}" method="POST"
                      style="display:inline;">
                    @csrf
                    <button type="submit"
                            onclick="return confirm('Yakin kembalikan produk ini?')"
                            class="bg-green-500 text-white px-3 py-1 rounded text-sm">
                        Kembalikan
                    </button>
                </form>
            </td>
        </tr>
    @endforeach
</tbody>
```

Penjelasan per bagian:

#### `<form action="{{ route('produk.restore', $product->id) }}" method="POST">`

- `route('produk.restore', $product->id)` = generate URL ke `/produk/{id}/restore`, dengan `$product->id` dipakai sebagai `{id}`. Hasilnya: `/produk/5/restore`.
- `method="POST"` = form dikirim sebagai POST. Cocok dengan route yang kita definisikan.

#### `@csrf`

Token keamanan Laravel. Wajib di setiap form POST/PUT/DELETE. Tanpa ini, Laravel akan menolak form dengan error "419 Page Expired".

#### `<button type="submit" onclick="return confirm('...')">`

- `type="submit"` = tombol ini mengirim form saat diklik.
- `onclick="return confirm('Yakin kembalikan produk ini?')"` = tampilkan dialog konfirmasi JavaScript. Kalau admin klik **Cancel**, form tidak jadi dikirim. Kalau klik **OK**, form dikirim.

**Kenapa perlu konfirmasi?** Karena restore adalah aksi yang **mengubah status produk**. Meskipun bukan destruktif (produk tidak hilang), tetap baik untuk konfirmasi. Mencegah klik tidak sengaja yang bikin bingung.

#### `class="bg-green-500 ..."`

Warna tombol hijau. Hijau = aksi positif (kembalikan/menghidupkan lagi). Berbeda dari tombol hapus yang biasanya merah.

---

## 8. View `trash.blade.php` Lengkap Setelah Tahap 5

Berikut versi paling update dari `trash.blade.php` (termasuk semua dari tahap 4 + tambahan tahap 5):

```blade
@extends('layouts.app')

@section('title', 'Tong Sampah Produk')

@section('content')
<div class="container mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">Tong Sampah Produk</h1>

    @if(session('success'))
        <div class="bg-green-100 border border-green-400 text-green-700 px-4 py-2 rounded mb-4">
            {{ session('success') }}
        </div>
    @endif

    <p class="text-gray-600 mb-4">
        Produk-produk berikut sudah dipindahkan ke sampah.
        Datanya masih tersimpan dan bisa dikembalikan.
    </p>

    <a href="{{ route('produk.index') }}" class="text-blue-600 underline mb-4 inline-block">
        &larr; Kembali ke Daftar Produk
    </a>

    @if($trashedProducts->count() > 0)
        <table class="w-full border border-gray-300">
            <thead class="bg-gray-100">
                <tr>
                    <th class="border p-2 text-left">ID</th>
                    <th class="border p-2 text-left">Nama Produk</th>
                    <th class="border p-2 text-left">Harga</th>
                    <th class="border p-2 text-left">Kategori</th>
                    <th class="border p-2 text-left">Dihapus Pada</th>
                    <th class="border p-2 text-left">Aksi</th>
                </tr>
            </thead>
            <tbody>
                @foreach($trashedProducts as $product)
                    <tr>
                        <td class="border p-2">{{ $product->id }}</td>
                        <td class="border p-2">{{ $product->nama }}</td>
                        <td class="border p-2">Rp {{ number_format($product->harga, 0, ',', '.') }}</td>
                        <td class="border p-2">{{ $product->kategori }}</td>
                        <td class="border p-2">{{ $product->deleted_at->format('d M Y, H:i') }}</td>
                        <td class="border p-2">
                            <form action="{{ route('produk.restore', $product->id) }}"
                                  method="POST" style="display:inline;">
                                @csrf
                                <button type="submit"
                                        onclick="return confirm('Yakin kembalikan produk ini?')"
                                        class="bg-green-500 text-white px-3 py-1 rounded text-sm">
                                    Kembalikan
                                </button>
                            </form>
                        </td>
                    </tr>
                @endforeach
            </tbody>
        </table>

        <div class="mt-4">
            {{ $trashedProducts->links() }}
        </div>
    @else
        <div class="bg-green-100 border border-green-300 p-4 rounded">
            Tong sampah kosong. Tidak ada produk yang dihapus.
        </div>
    @endif

</div>
@endsection
```

---

## 9. Uji Coba: Cek Apakah Fitur Restore Berfungsi

Sekarang tes end-to-end. Lakukan langkah-langkah berikut:

### Langkah A: Pastikan Ada Produk di Sampah

Akses `/produk/sampah`. Pastikan minimal ada 1 produk di tong sampah. Kalau kosong, hapus satu produk dulu dari halaman `/produk`.

### Langkah B: Klik Tombol "Kembalikan"

Di salah satu baris produk sampah, klik tombol hijau **"Kembalikan"**.

### Langkah C: Konfirmasi Dialog

Browser tampilkan dialog JavaScript: "Yakin kembalikan produk ini?". Klik **OK**.

### Langkah D: Periksa Hasil

Setelah redirect, dua hal harus terjadi:

1. **Pesan sukses hijau** muncul di atas tabel sampah: `Produk "Kopi Susu Vanilla" berhasil dikembalikan.`
2. **Produk yang direstore hilang dari daftar sampah** (karena sudah tidak di sampah lagi).

### Langkah E: Verifikasi di Halaman Produk Aktif

Buka `/produk`. Produk yang baru direstore harus **muncul kembali** di daftar produk aktif, dengan data lengkap (nama, harga, kategori, gambar) persis seperti sebelum dihapus.

### Langkah F: Verifikasi di Database (Opsional)

Untuk yakin 100%, cek database. Buka tinker:

```bash
php artisan tinker
```

```php
$product = Product::withTrashed()->find(5);
echo $product->deleted_at;  // Harusnya NULL (kosong)
```

Atau cek langsung di phpMyAdmin / TablePlus. Kolom `deleted_at` untuk produk yang direstore harus **NULL**.

Kalau semua langkah berhasil, **fitur restore sudah berfungsi sempurna**.

---

## 10. Hal-hal Penting yang Sering Bikin Pemula Bingung

### a. `restore()` Hanya Bekerja pada Model dari `withTrashed()` atau `onlyTrashed()`

Ingat aturan emas di bagian 3:

```php
// ❌ SALAH — produk tidak ketemu, error 404
$product = Product::findOrFail(5);
$product->restore();

// ✅ BENAR — produk ditemukan meski sudah di-soft-delete
$product = Product::withTrashed()->findOrFail(5);
$product->restore();

// ✅ JUGA BENAR — produk diambil khusus dari yang sudah dihapus
$product = Product::onlyTrashed()->findOrFail(5);
$product->restore();
```

Pilih salah satu yang benar. Aku rekomendasi `withTrashed()` karena lebih fleksibel (bisa restore produk yang sudah dihapus maupun produk aktif yang tidak sengaja masuk ke method ini).

### b. `restore()` Bukan `delete()` yang Dibalik

Restore **bukan** kebalikan teknis dari delete. Yang dilakukan restore cuma satu: set `deleted_at = NULL`. Data lain (created_at, updated_at, semua field) **tidak diubah**.

Jadi kalau produk sudah diedit 5 kali sebelum dihapus, lalu di-restore, semua riwayat edit tetap ada di field `updated_at`. Restore cuma "menghidupkan kembali", bukan "mereset".

### c. Bisa Restore Banyak Produk Sekaligus (Bulk Restore)

Laravel juga mendukung restore banyak produk sekaligus tanpa loop:

```php
Product::onlyTrashed()->where('kategori', 'Minuman')->restore();
```

Ini akan restore semua produk minuman yang ada di sampah, sekaligus. Berguna kalau admin mau "kembalikan semua produk minuman yang sudah dihapus".

Untuk pemula, tidak perlu khawatir tentang ini dulu. Cukup paham restore per-produk. Bulk restore bisa dieksplorasi nanti.

### d. Produk yang Tidak Pernah Dihapus Tidak Bisa Di-restore

Kalau kamu coba restore produk yang `deleted_at`-nya masih NULL (produk aktif):

```php
$product = Product::withTrashed()->find(3);  // produk aktif, deleted_at NULL
$product->restore();  // tidak error, tapi tidak ada efek apa-apa
```

Tidak error, tapi tidak terjadi apa-apa. Karena produk itu sebenarnya belum dihapus.

### e. Method HTTP `POST` Wajib untuk Form Restore

Jangan pakai link `<a href="...">Kembalikan</a>`. Karena:

1. Link adalah GET request, bukan POST. Route restore kita `Route::post(...)`, jadi akan 405 Method Not Allowed.
2. Link GET bisa ke-trigger oleh crawler atau preview browser, yang berarti produk bisa ke-restore tanpa sengaja.

Selalu pakai form dengan `method="POST"` dan `@csrf`.

### f. Urutan Route: `restore` Tidak Terlalu Sensitif

Berbeda dengan route `/produk/sampah` yang harus di atas `/produk/{id}`, route `/produk/{id}/restore` **lebih aman**:

- `/produk/{id}/restore` punya 2 segmen: `{id}` + `restore`. Tidak akan ditelan oleh `/produk/{id}` yang cuma 1 segmen.
- Tapi kalau kamu punya route lain seperti `/produk/{id}/edit`, urutan antar 2-segmen tidak masalah (tidak saling konflik).

Tetap praktis yang baik: taruh route statis di atas, route dinamis di bawah.

---

## 11. Apa yang Sudah Bisa Kamu Lakukan Setelah Tahap 5?

Setelah tahap 5, alur lengkap yang bisa admin lakukan:

1. Buka halaman `/produk/sampah`.
2. Lihat daftar produk yang sudah dihapus.
3. Klik tombol **"Kembalikan"** di salah satu produk.
4. Konfirmasi dialog.
5. Produk menghilang dari sampah.
6. Produk muncul kembali di daftar produk aktif (`/produk`) dengan data utuh.

**Siklus hidup produk sampai tahap ini**:

```
Produk dibuat (create)
    ↓
Produk diedit (update)
    ↓
Produk di-soft-delete → masuk sampah
    ↓
Produk di-restore → kembali ke daftar aktif
    ↓
Bisa di-soft-delete lagi → masuk sampah lagi
    ↓
... (bisa berulang)
```

Produk bisa "bolak-balik" antara aktif dan sampah sesering yang dibutuhkan. Tidak ada data yang hilang selama proses ini.

**Yang belum bisa admin lakukan**:

- **Menghapus permanen** produk dari sampah (force delete). Produk yang sudah masuk sampah akan **terus ada** selamanya di database, walaupun tidak akan pernah dijual lagi. Sampah bisa penuh. Ini yang akan kita kerjakan di tahap 6.

---

## Ringkasan Tahap 5

| Hal | Isi |
|---|---|
| Konsep | Restore = set `deleted_at` jadi NULL, produk "hidup" lagi |
| Aturan emas | Harus pakai `withTrashed()` (atau `onlyTrashed()`) sebelum cari produk sampah |
| Route baru | `Route::post('/produk/{id}/restore', ...)->name('produk.restore')` |
| Method controller | `restore($id)` yang pakai `Product::withTrashed()->findOrFail($id)->restore()` |
| View tambahan | Kolom "Aksi" dengan tombol hijau "Kembalikan" + konfirmasi JavaScript |
| Method HTTP | POST (bukan GET, supaya tidak ke-trigger tidak sengaja) |
| Redirect setelah restore | Balik ke halaman sampah dengan pesan sukses |

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: membuat fitur Force Delete (menghapus produk secara permanen dari sampah)?**

Kalau iya, tahap 6 (tahap terakhir) kita akan:
1. Tambah route `DELETE /produk/{id}/force-delete`.
2. Tambah method `forceDelete($id)` di controller.
3. Tambah tombol "Hapus Permanen" (warna merah) di tabel sampah, dengan **konfirmasi ganda** karena aksi ini tidak bisa dibatalkan.
4. Lihat apa yang terjadi saat force delete dengan `withTrashed()` — produk benar-benar hilang dari database (`DELETE FROM`).
5. Ringkasan penuh materi soft delete dari tahap 1 sampai 6, supaya kamu paham gambaran besarnya.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
