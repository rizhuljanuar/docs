# Tahap 4 — Membuat Halaman "Tong Sampah Produk" (`onlyTrashed`)

> Materi: Soft Delete Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi: Membuka Tutup Tempat Sampah

Di tahap 3, setiap kali admin klik **Hapus**, produknya **menghilang dari daftar**. Tapi datanya masih ada di database. Seperti kertas yang sudah dibuang ke tempat sampah: tidak ada di meja, tapi masih di rumah.

Sekarang pertanyaannya:

> Kalau produknya "menghilang", bagaimana admin bisa melihat produk yang sudah dihapus itu lagi?

Jawabannya: **kita harus bikin halaman khusus yang isinya barang-barang dari tempat sampah.** Halaman ini kita beri nama **"Tong Sampah Produk"** (atau "Trash Produk").

Bayangkan kamu membuka tutup tempat sampah buat memeriksa: "Eh, apa ada barang yang kubuang kemarin ternyata masih berguna?"

Halaman Tong Sampah Produk itu seperti **daftar inventaris isi tempat sampah**. Isinya:

- Daftar semua produk yang sudah di-soft-delete.
- Kapan produk itu dihapus (`deleted_at`).
- (Nanti) tombol untuk **mengembalikan** produk (restore).
- (Nanti) tombol untuk **membuang permanen** produk (force delete).

Tapi di tahap 4 ini, kita fokus dulu ke **menampilkan isi tong sampah**. Tombol-tombol aksi menyusul di tahap 5 dan 6.

---

## 2. Rencana Halaman Baru: `/produk/sampah`

Kita akan bikin halaman baru dengan URL:

```
/produk/sampah
```

**Kenapa pakai kata "sampah", bukan "trash" atau "deleted"?**

Beberapa alasan:

- **Mudah dibaca pemula Indonesia.** "Sampah" langsung paham maksudnya.
- **Konsisten dengan analogi** yang sudah kita pakai dari tahap 1.
- Boleh juga pakai `/produk/trash` kalau kamu lebih suka. Tidak ada aturan baku. Yang penting konsisten di semua tempat (route, view, controller).

Halaman ini akan menampilkan tabel seperti ini:

| ID | Nama Produk | Harga | Kategori | Dihapus Pada | Aksi |
|---|---|---|---|---|---|
| 5 | Kopi Susu Vanilla | 18000 | Minuman | 18 Jul 2026, 11:02 | (tombol restore & force delete — tahap 5 & 6) |
| 12 | Roti Coklat | 12000 | Makanan | 17 Jul 2026, 14:30 | (tombol restore & force delete — tahap 5 & 6) |

**Yang muncul di halaman ini HANYA produk yang `deleted_at`-nya terisi.** Produk aktif tidak muncul. Sudah otomatis difilter oleh `onlyTrashed()`.

---

## 3. Langkah 1: Tambah Route Baru

Buka file `routes/web.php`. Di file ini, biasanya kamu sudah punya route-route CRUD seperti ini:

```php
use App\Http\Controllers\ProductController;
use Illuminate\Support\Facades\Route;

Route::get('/produk', [ProductController::class, 'index'])->name('produk.index');
Route::get('/produk/create', [ProductController::class, 'create'])->name('produk.create');
Route::post('/produk', [ProductController::class, 'store'])->name('produk.store');
Route::get('/produk/{id}', [ProductController::class, 'show'])->name('produk.show');
Route::get('/produk/{id}/edit', [ProductController::class, 'edit'])->name('produk.edit');
Route::put('/produk/{id}', [ProductController::class, 'update'])->name('produk.update');
Route::delete('/produk/{id}', [ProductController::class, 'destroy'])->name('produk.destroy');
```

**Sekarang tambahkan satu route baru** untuk halaman sampah:

```php
Route::get('/produk/sampah', [ProductController::class, 'trash'])->name('produk.trash');
```

Taruh **di atas** route `/produk/{id}` (route `show`). Kenapa urutannya penting? Akan dijelaskan di bagian **Penting: Urutan Route** di bawah.

Jadi urutan route-nya jadi seperti ini:

```php
Route::get('/produk', [ProductController::class, 'index'])->name('produk.index');
Route::get('/produk/create', [ProductController::class, 'create'])->name('produk.create');
Route::post('/produk', [ProductController::class, 'store'])->name('produk.store');

// Route sampah — HARUS di atas route /produk/{id}
Route::get('/produk/sampah', [ProductController::class, 'trash'])->name('produk.trash');

Route::get('/produk/{id}', [ProductController::class, 'show'])->name('produk.show');
Route::get('/produk/{id}/edit', [ProductController::class, 'edit'])->name('produk.edit');
Route::put('/produk/{id}', [ProductController::class, 'update'])->name('produk.update');
Route::delete('/produk/{id}', [ProductController::class, 'destroy'])->name('produk.destroy');
```

Penjelasan baris baru:

| Bagian | Arti |
|---|---|
| `Route::get('/produk/sampah', ...)` | Saat user akses URL `/produk/sampah`, jalankan controller ini |
| `[ProductController::class, 'trash']` | Method yang dijalankan: `trash()` di `ProductController` |
| `->name('produk.trash')` | Beri nama route `produk.trash` supaya gampang dipanggil dari view pakai `route('produk.trash')` |

---

## 4. ⚠️ PENTING: Urutan Route Itu Mengubah Perilaku

Laravel membaca route **dari atas ke bawah**. Route pertama yang cocok dengan URL akan **menang**. Route di bawahnya **tidak dipakai**.

Sekarang bandingkan dua route ini:

```php
Route::get('/produk/{id}', ...);       // menangkap /produk/5, /produk/create, /produk/sampah
Route::get('/produk/sampah', ...);     // menangkap /produk/sampah
```

Kalau `Route::get('/produk/{id}', ...)` ada **di atas**, apa yang terjadi saat user akses `/produk/sampah`?

- Laravel akan **cocokin** dengan `/produk/{id}` lebih dulu.
- `{id}` adalah wildcard (bisa apa saja), jadi cocok dengan "sampah".
- Laravel pikir `$id = 'sampah'`.
- Method `show('sampah')` dipanggil, dan karena 'sampah' bukan angka, biasanya error atau 404.

**Solusinya**: taruh route statis (`/produk/sampah`, `/produk/create`) **di atas** route dinamis (`/produk/{id}`). Aturan praktis Laravel:

> **Route yang URL-nya spesifik (punya kata tetap) harus dideklarasi SEBELUM route yang pakai wildcard `{ }`.**

---

## 5. Langkah 2: Tambah Method `trash()` di Controller

Buka `app/Http/Controllers/ProductController.php`. Tambahkan satu method baru:

```php
public function trash()
{
    $trashedProducts = Product::onlyTrashed()
        ->latest('deleted_at')
        ->paginate(10);

    return view('products.trash', compact('trashedProducts'));
}
```

Penjelasan baris per baris:

### `Product::onlyTrashed()`

Ini method khusus dari trait `SoftDeletes`. Fungsinya: **ambil HANYA produk yang `deleted_at`-nya terisi** (yang ada di tong sampah). Produk aktif tidak ikut.

### `->latest('deleted_at')`

Urutkan dari yang **paling baru dihapus** dulu. Biasanya admin lebih ingin lihat produk yang baru dihapus (kemarin, hari ini) ketimbang yang sudah lama (berbulan-bulan lalu).

`latest()` default-nya urut berdasar `created_at`. Tapi di halaman sampah, kita mau urut berdasar `deleted_at` (tanggal penghapusan), makanya kita kasih argumen `'deleted_at'`.

### `->paginate(10)`

Bagi hasil jadi halaman-halaman, 10 produk per halaman. Kalau di tong sampah ada 50 produk, halaman pertama tampil 10, ada tombol "Next" ke halaman 2, 3, dst.

Pakai `paginate`, bukan `get`, supaya halaman tidak penuh sesak kalau ada banyak produk dihapus.

### `return view('products.trash', ...)`

Laravel akan cari file view di `resources/views/products/trash.blade.php` (kita bikin di langkah berikutnya).

### `compact('trashedProducts')`

Oper variabel `$trashedProducts` ke view, supaya di view bisa dipakai pakai nama `$trashedProducts` juga. Sama seperti cara kamu oper produk ke view `index`.

---

## 6. Langkah 3: Buat View `trash.blade.php`

Buka folder `resources/views/products/`. Di sini biasanya ada file-file view CRUD kamu (`index.blade.php`, `create.blade.php`, `show.blade.php`, dll).

**Bikin file baru** bernama `trash.blade.php` di folder itu:

```
resources/views/products/trash.blade.php
```

Berikut isi file-nya. Aku pecah penjelasannya per bagian supaya gampang dipahami.

### Bagian A: Header Halaman

```blade
@extends('layouts.app')

@section('title', 'Tong Sampah Produk')

@section('content')
<div class="container mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">Tong Sampah Produk</h1>

    <p class="text-gray-600 mb-4">
        Produk-produk berikut sudah dipindahkan ke sampah.
        Datanya masih tersimpan dan bisa dikembalikan.
    </p>
```

Penjelasan:

| Baris | Fungsi |
|---|---|
| `@extends('layouts.app')` | Pakai layout utama projek kamu (header, footer, dll) |
| `@section('title', ...)` | Set judul tab browser |
| `@section('content')` | Isi halaman utama |
| `<h1>` | Judul halaman |
| `<p>` | Penjelasan singkat supaya admin paham konteks |

### Bagian B: Tombol Kembali ke Daftar Produk

```blade
    <a href="{{ route('produk.index') }}" class="text-blue-600 underline mb-4 inline-block">
        &larr; Kembali ke Daftar Produk
    </a>
```

Penjelasan:

- `route('produk.index')` = helper Laravel untuk bikin URL dari nama route. Hasilnya: `/produk`.
- `&larr;` = karakter panah kiri (←) di HTML.
- Admin gampang balik ke daftar produk aktif tanpa harus ketik URL.

### Bagian C: Tabel Daftar Produk Sampah

```blade
    @if($trashedProducts->count() > 0)
        <table class="w-full border border-gray-300">
            <thead class="bg-gray-100">
                <tr>
                    <th class="border p-2 text-left">ID</th>
                    <th class="border p-2 text-left">Nama Produk</th>
                    <th class="border p-2 text-left">Harga</th>
                    <th class="border p-2 text-left">Kategori</th>
                    <th class="border p-2 text-left">Dihapus Pada</th>
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
                    </tr>
                @endforeach
            </tbody>
        </table>
```

Penjelasan:

| Baris | Fungsi |
|---|---|
| `@if($trashedProducts->count() > 0)` | Cek apakah ada produk di sampah. Kalau kosong, tampilkan pesan alternatif (di Bagian D). |
| `<table>` | Tampilkan data dalam bentuk tabel. |
| `@foreach($trashedProducts as $product)` | Loop tiap produk sampah, tampilkan satu per satu. |
| `number_format($product->harga, 0, ',', '.')` | Format harga jadi "Rp 18.000" biar enak dibaca (pemisah ribuan pakai titik). |
| `$product->deleted_at->format('d M Y, H:i')` | `deleted_at` itu objek `Carbon` (tanggal Laravel). `->format('d M Y, H:i')` akan tampilkan "18 Jul 2026, 11:02". |

**Penting tentang kolom `deleted_at`**:

Di tahap 2, kita menambahkan kolom `deleted_at` ke tabel. Kolom ini otomatis **dikonversi jadi objek `Carbon`** oleh Laravel saat diambil dari database. `Carbon` punya banyak method, salah satunya `format()` untuk menampilkan tanggal dalam format yang kita mau.

Kalau kamu akses `$product->deleted_at` langsung tanpa `->format(...)`, tampilnya: "2026-07-18 11:02:15" (format default database). Pakai `->format(...)` bikin tampilannya lebih manusiawi.

### Bagian D: Pesan Kalau Tong Sampah Kosong

```blade
    @else
        <div class="bg-green-100 border border-green-300 p-4 rounded">
            Tong sampah kosong. Tidak ada produk yang dihapus.
        </div>
    @endif
```

Penjelasan:

- `@else` berlaku kalau `$trashedProducts->count()` == 0 (tong sampah kosong).
- Tampilkan pesan ramah supaya admin tahu tidak ada error, cuma memang kosong.
- `@endif` menutup blok `@if`.

### Bagian E: Navigasi Pagination

```blade
    <div class="mt-4">
        {{ $trashedProducts->links() }}
    </div>

</div>
@endsection
```

Penjelasan:

- `$trashedProducts->links()` = tombol navigasi pagination (1, 2, 3, Next). Otomatis di-generate Laravel karena kita pakai `->paginate(10)` di controller.
- `@endsection` = tutup section `content`.

---

## 7. File View Lengkap (Bisa Kamu Copy-Paste)

```blade
@extends('layouts.app')

@section('title', 'Tong Sampah Produk')

@section('content')
<div class="container mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">Tong Sampah Produk</h1>

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

## 8. Langkah 4: Tambah Link "Lihat Tong Sampah" di Halaman Daftar Produk

Halaman sampah sudah jadi, tapi admin harus hafal URL-nya. Lebih baik kita **tambahkan link pintu masuk** di halaman daftar produk (`/produk`).

Buka file `resources/views/products/index.blade.php`. Cari bagian header (biasanya dekat `<h1>Daftar Produk</h1>`), tambahkan link:

```blade
<div class="flex justify-between items-center mb-4">
    <h1 class="text-2xl font-bold">Daftar Produk</h1>

    <div>
        <a href="{{ route('produk.create') }}" class="bg-blue-500 text-white px-3 py-1 rounded mr-2">
            + Tambah Produk
        </a>

        <a href="{{ route('produk.trash') }}" class="bg-gray-500 text-white px-3 py-1 rounded">
            Tong Sampah
        </a>
    </div>
</div>
```

Penjelasan:

- `route('produk.trash')` = URL ke `/produk/sampah` (nama route yang sudah kita bikin).
- Tombol ini **selalu muncul**, walaupun tong sampah kosong. Biar admin gampang cek kapan saja.

Bonus: bisa juga tampilkan **badge jumlah** produk di sampah. Tapi ini opsional, lebih cocok dikerjakan di tahap 6 (ringkasan). Untuk sekarang, link sederhana sudah cukup.

---

## 9. Uji Coba: Cek Apakah Halaman Sampah Berfungsi

Sekarang saatnya tes. Lakukan langkah-langkah berikut:

### Langkah A: Pastikan Ada Produk di Sampah

Kalau dari tahap 3 kamu sudah soft-delete beberapa produk, langsung lanjut ke langkah B.

Kalau belum, hapus satu atau dua produk dulu lewat halaman `/produk` (klik tombol Hapus). Atau pakai tinker:

```bash
php artisan tinker
```

```php
$product = Product::first();
$product->delete();
exit
```

### Langkah B: Buka Halaman Sampah

Buka browser, akses:

```
http://localhost:8000/produk/sampah
```

(Ganti port kalau dev server kamu pakai port lain.)

### Langkah C: Periksa Tampilan

Halaman harusnya menampilkan:

- Judul "Tong Sampah Produk".
- Link "Kembali ke Daftar Produk".
- Tabel berisi produk yang sudah dihapus.
- Kolom **Dihapus Pada** terisi tanggal penghapusan yang benar.
- Tombol pagination kalau produk sampah lebih dari 10.

### Langkah D: Coba Tong Sampah Kosong

Untuk pembelajaran: hapus semua produk di sampah (atau restore mereka di tahap 5 nanti). Akses lagi halaman sampah. Harusnya muncul pesan hijau "Tong sampah kosong."

Kalau semua langkah berhasil, **halaman Tong Sampah kamu sudah berfungsi**.

---

## 10. Hal-hal Penting yang Sering Bikin Pemula Bingung

### a. `onlyTrashed()` vs `withTrashed()` vs Query Biasa

Tiga query ini sering tertukar. Hafalkan bedanya:

| Query | Apa yang diambil |
|---|---|
| `Product::all()` atau `Product::get()` | **HANYA** produk aktif (`deleted_at` NULL) |
| `Product::onlyTrashed()->get()` | **HANYA** produk yang sudah dihapus (`deleted_at` terisi) |
| `Product::withTrashed()->get()` | **SEMUA** produk, aktif + yang dihapus |

**Cara mengingat**:

- `only` = **hanya** yang dihapus saja.
- `with` = **ikutsertakan** yang dihapus juga (jadi semua).

### b. Error "Route Not Defined" saat pakai `route('produk.trash')`

Kalau view error karena route tidak dikenali, jalankan:

```bash
php artisan route:clear
```

Lalu coba lagi. Kadang cache route Laravel masih simpan versi lama.

### c. `deleted_at` Tampil Sebagai Default Database

Kalau kamu lupa pakai `->format(...)`, tanggal yang muncul di view akan seperti "2026-07-18 11:02:15". Tidak enak dibaca. Selalu pakai `->format('d M Y, H:i')` atau format serupa.

### d. Tombol Pagination Tidak Muncul

Kalau hasil query kurang dari atau sama dengan 10 (default `paginate(10)`), tombol pagination **tidak muncul**. Itu wajar. Laravel otomatis sembunyikan pagination kalau halaman cuma satu.

Kalau mau selalu tampil, pakai `paginate(10)` di controller, lalu di view tambahkan:

```blade
{{ $trashedProducts->onEachSide(1)->links() }}
```

Tapi untuk pemula, biarkan default saja. Tidak perlu ribet.

### e. View Tidak Ketemu

Error "View [products.trash] not found" berarti file `trash.blade.php` **belum ada** di folder `resources/views/products/`, atau **salah nama**. Periksa:

- Lokasi file harus `resources/views/products/trash.blade.php`.
- Nama file harus persis `trash.blade.php` (case-sensitive di Linux).
- Ekstensi harus `.blade.php`, bukan `.php` saja.

---

## 11. Apa yang Sudah Bisa Kamu Lakukan Setelah Tahap 4?

Setelah tahap 4, admin bisa:

1. Membuka halaman `/produk/sampah`.
2. Melihat daftar semua produk yang sudah di-soft-delete.
3. Melihat **kapan** setiap produk dihapus (kolom `deleted_at` diformat ramah).
4. Klik "Kembali ke Daftar Produk" untuk balik ke halaman utama.
5. Pakai pagination kalau produk di sampah banyak.

**Yang belum bisa admin lakukan**:

- **Mengembalikan** produk dari sampah ke daftar aktif (restore). Ini tahap 5.
- **Menghapus permanen** produk dari sampah (force delete). Ini tahap 6.

Sampai tahap 4, halaman sampah ini **read-only**. Admin bisa lihat, tapi belum bisa bertindak.

---

## Ringkasan Tahap 4

| Hal | Isi |
|---|---|
| Tujuan | Bikin halaman untuk lihat produk yang sudah di-soft-delete |
| Route baru | `Route::get('/produk/sampah', ...)` dengan nama `produk.trash` |
| Method controller baru | `trash()` yang pakai `Product::onlyTrashed()->latest('deleted_at')->paginate(10)` |
| View baru | `resources/views/products/trash.blade.php` dengan tabel + pagination |
| Link masuk | Tombol "Tong Sampah" di halaman daftar produk (`index.blade.php`) |
| Format tanggal | `$product->deleted_at->format('d M Y, H:i')` |
| Aturan urutan route | Route statis (`/produk/sampah`) harus di atas route dinamis (`/produk/{id}`) |

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: membuat fitur Restore (mengembalikan produk dari sampah ke daftar aktif)?**

Kalau iya, tahap 5 kita akan:
1. Tambah route `Route::post('/produk/{id}/restore', ...)` untuk restore.
2. Tambah method `restore()` di controller yang pakai `Product::withTrashed()->findOrFail($id)->restore()`.
3. Tambah tombol "Kembalikan" di setiap baris tabel sampah.
4. Tes klik tombol, lihat produk muncul lagi di daftar aktif.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
