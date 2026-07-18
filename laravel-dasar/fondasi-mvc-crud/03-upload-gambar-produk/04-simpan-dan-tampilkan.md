# Upload Gambar Produk — Tahap 4: Menyimpan & Menampilkan Gambar

> Bagian dari: Laravel Dasar — Fondasi MVC & CRUD
> Topik: 3. Upload Gambar Produk
> Tahap: 4 dari 5 — **Menyimpan & Menampilkan Gambar** (kode lengkap)

---

## 1. Tujuan Tahap Ini

Setelah tahap 3, file yang masuk sudah **divalidasi** (penjaga pintu bekerja).
Sekarang kita kerjakan:

1. Menyimpan file yang lolos validasi ke disk `public`.
2. Menyimpan **path** gambar ke database (bukan file-nya!).
3. Menampilkan gambar di halaman daftar produk dengan `Storage::url()`.

> **Ponytail:** Kita pakai API bawaan Laravel (`Storage`, `UploadedFile`),
> tidak ada package tambahan. Cukup satu symlink di awal, sisanya kode native.

---

## 2. Persiapan: Pastikan Symlink Sudah Dibuat

Sebelum mulai, jalankan **sekali** (sudah dibahas di tahap 2):

```bash
php artisan storage:link
```

Cek hasilnya: harus ada folder/file `public/storage` yang menunjuk ke
`storage/app/public`. Tanpa symlink ini, gambar yang disimpan tidak bisa
diakses dari browser meskipun filenya ada.

---

## 3. Migration: Tambah Kolom `gambar` ke Tabel Produk

Pastikan tabel `produk` punya kolom untuk menyimpan path gambar.

`database/migrations/xxxx_create_produk_table.php`:

```php
Schema::create('produk', function (Blueprint $table) {
    $table->id();
    $table->string('nama', 100);
    $table->string('gambar')->nullable();   // path gambar, boleh kosong
    $table->timestamps();
});
```

Jalankan migration:

```bash
php artisan migrate
```

Kenapa `nullable()`? Supaya produk bisa disimpan dulu tanpa gambar, gambar
bisa ditambahkan nanti (opsional, sesuai kebutuhan toko).

> **Catatan penting:** tipe kolomnya `string` (path teks), **bukan** `binary`
> atau `blob`. Kita menyimpan **lokasi file**, bukan isi file.

---

## 4. Controller: Simpan File + Path ke Database

`app/Http/Controllers/ProdukController.php`:

```php
namespace App\Http\Controllers;

use App\Models\Produk;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class ProdukController extends Controller
{
    public function create()
    {
        return view('produk.create');
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'nama'   => ['required', 'string', 'max:100'],
            'gambar' => ['required', 'image', 'mimes:jpg,jpeg,png,webp', 'max:2048'],
        ]);

        // 1. Simpan file ke disk 'public', folder 'produk'
        $path = $request->file('gambar')->store('produk', 'public');
        // contoh hasil $path: "produk/XYZabcsepatu.jpg"

        // 2. Simpan data ke database (path-nya saja, bukan file)
        Produk::create([
            'nama'   => $data['nama'],
            'gambar' => $path,
        ]);

        return redirect()->route('produk.index')->with('success', 'Produk ditambahkan.');
    }
}
```

### Penjelasan `$request->file('gambar')->store('produk', 'public')`:

- `file('gambar')` → ambil instance `UploadedFile` dari input bernama `gambar`.
- `->store('produk', 'public')` → simpan ke disk `public`, di subfolder `produk`.
- Laravel otomatis membuat nama file unik (terenkripsi) supaya tidak tabrakan
  dengan file lama, contoh: `produk/aB3xK9pQ.jpg`.
- Fungsi ini **mengembalikan path relatif** terhadap disk, misalnya
  `"produk/aB3xK9pQ.jpg"`. Path inilah yang disimpan ke database.

### Alternatif: `putFileAs` (nama yang kita tentukan)

Kalau mau nama file mengikuti nama asli / bisa dibaca manusia:

```php
$namaFile = time() . '-' . $request->file('gambar')->getClientOriginalName();
$path = $request->file('gambar')->storeAs('produk', $namaFile, 'public');
// contoh: "produk/1718000000-sepatu-lari.jpg"
```

Untuk pemula, **`store()` saja sudah cukup** (lebih aman, anti tabrakan).
> **Ponytail:** Jangan pakai `storeAs` kecuali kamu butuh nama yang bisa dibaca
> manusia. Default lebih sederhana.

---

## 5. Blade: Menampilkan Gambar di Halaman Daftar Produk

`resources/views/produk/index.blade.php`:

```php
@if (session('success'))
    <div style="background:#e0f7e0; padding:8px;">
        {{ session('success') }}
    </div>
@endif

@foreach ($produks as $produk)
    <div style="border:1px solid #ccc; padding:8px; margin:8px 0;">
        <h3>{{ $produk->nama }}</h3>

        @if ($produk->gambar)
            <img src="{{ Storage::url($produk->gambar) }}"
                 alt="{{ $produk->nama }}"
                 width="200">
        @else
            <p><em>(Tidak ada gambar)</em></p>
        @endif
    </div>
@endforeach
```

Hal penting:

- `Storage::url($produk->gambar)` → menghasilkan URL publik, contoh:
  `http://toko.test/storage/produk/aB3xK9pQ.jpg`.
- Blade butuh directive `@if` karena `gambar` bisa `null` (di tahap migration
  tadi kita set `nullable()`).
- Gunakan `{{ ... }}` (double curly) supaya path di-escape (aman dari XSS).

Jangan lupa import facade `Storage` di atas file Blade **jika** perlu
(dibeberapa versi Blade otomatis resolve). Aman dengan menambah:

```php
@use('Illuminate\Support\Facades\Storage')
```

---

## 6. Route (Untuk Kelengkapan)

`routes/web.php`:

```php
use App\Http\Controllers\ProdukController;

Route::get('/produk',           [ProdukController::class, 'index'])->name('produk.index');
Route::get('/produk/create',    [ProdukController::class, 'create'])->name('produk.create');
Route::post('/produk',          [ProdukController::class, 'store'])->name('produk.store');
```

---

## 7. Alur Lengkap dari Upload ke Tampil

```
1. User pilih sepatu.png, klik Simpan
       │
2. POST /produk  (dengan multipart/form-data)
       │
3. Controller->store()
       │
       ├─ validate()      → cek image, mimes, max:2048
       │
       ├─ ->store('produk','public')
       │     └─ file masuk: storage/app/public/produk/aB3xK9pQ.jpg
       │
       ├─ Produk::create(['gambar' => 'produk/aB3xK9pQ.jpg', ...])
       │     └─ path masuk DB
       │
       └─ redirect ke /produk
              │
4. Halaman index.blade.php
       │
       └─ <img src="{{ Storage::url('produk/aB3xK9pQ.jpg') }}">
              └─ browser request: /storage/produk/aB3xK9pQ.jpg
                  └─ symlink public/storage → storage/app/public
                  └─ gambar muncul di browser ✅
```

---

## 8. Cek Masalah Umum (Troubleshooting)

| Gejala                                     | Penyebab                                | Solusi                                  |
| ------------------------------------------ | --------------------------------------- | --------------------------------------- |
| Gambar tidak muncul (404)                  | Belum jalankan `storage:link`           | `php artisan storage:link`              |
| Gambar broken, padahal filenya ada         | Salah disk / path                       | Pastikan `store('produk', 'public')`    |
| Error "disk public does not exist"         | Konfigurasi `config/filesystems.php`    | Cek disk `public` masih ada             |
| Upload gagal besar (>2MB) padahal belum 2MB| Batas `upload_max_filesize` di `php.ini` | Ubah `php.ini`,重启 PHP                |
| Error `Storage::url` not found di Blade    | Lupa import facade                      | Pakai `@use` atau `\Storage::url(...)`   |

---

## 9. Ringkasan Tahap 4

- Migration: kolom `gambar` tipe **string** (path), `nullable()`.
- Simpan file: `$file->store('produk', 'public')` → kembalikan path relatif.
- Simpan ke DB: path saja, bukan file.
- Tampilkan: `{{ Storage::url($produk->gambar) }}` di tag `<img>`.
- Symlink: jalankan **sekali** `php artisan storage:link`.
- Cek `@if ($produk->gambar)` sebelum tampilkan (karena bisa null).

Sekarang upload gambar produk sudah berfungsi end-to-end!

---

## 10. Cek Pemahaman

1. Apa yang dikembalikan oleh `$file->store('produk', 'public')`?
2. Apa yang disimpan di kolom `gambar` di database: file atau path?
3. Kenapa harus ada `@if ($produk->gambar)` sebelum tampilkan `<img>`?
4. Bagaimana cara menghasilkan URL publik dari path yang tersimpan?

---

> **Pertanyaan untuk kamu:** Sudah berhasil alurnya?
> Mau lanjut ke **Tahap 5 — Latihan / Praktik & Rangkuman Akhir**
> (latihan soal, kasus tambahan seperti hapus gambar saat produk dihapus,
> dan rangkuman seluruh materi), atau ulas ulang tahap 4 dulu?
