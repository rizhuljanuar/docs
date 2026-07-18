# Tahap 6 — Menghubungkan Tombol Aktifkan/Nonaktifkan: Route + Controller Toggle Status

> Materi: Status Produk Aktif/Nonaktif
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi: Saklar Lampu yang Benar-benar Bisa Ditekan

Di tahap 5, kita sudah bikin halaman admin yang **menampilkan** status produk (badge hijau/merah) dan **placeholder** tombol. Tapi tombolnya belum berfungsi. Bayangkan: kamu sudah memasang saklar lampu di dinding, tapi kabelnya belum disambung ke listrik. Kamu tekan, tekan, tekan, lampu tetap mati.

Di tahap 6 ini, kita **sambungkan kabelnya**. Saklar (tombol) yang sudah terpasang di view akan kita hubungkan ke:

1. **Route** = jalur kabel dari tombol ke controller.
2. **Controller** = saklar utama yang benar-benar mengubah `is_active` di database.

Setelah selesai, admin bisa klik tombol **"Aktifkan"** atau **"Nonaktifkan"**, dan badge di tabel akan **berubah warna** setelah halaman di-refresh. Inilah saatnya produk benar-benar bisa dikendalikan status publikasinya dari halaman admin.

Yang seru: untuk toggle (bolak-balik) status, kita cuma butuh **satu route, satu method controller, dan satu form**. Tidak perlu dua tombol yang berbeda. Logikanya sederhana: kalau produk sekarang aktif, ubah jadi nonaktif. Kalau sekarang nonaktif, ubah jadi aktif. Cukup satu aksi "toggle status".

---

## 2. Apa Itu "Toggle"? Konsep Penting di Pemrograman

**Toggle** = **bunyikan / alihkan**. Dalam pemrograman, toggle berarti **membalikkan nilai**:

- Kalau `true`, jadikan `false`.
- Kalau `false`, jadikan `true`.
- Kalau `1`, jadikan `0`.
- Kalau `0`, jadikan `1`.

Saking seringnya dipakai, hampir semua bahasa pemrograman punya operator khusus untuk toggle boolean: operator **`!`** (dibaca "NOT" / "bukan").

Di PHP:

```php
$product->is_active = !$product->is_active;
```

Baris itu artinya: "Set `is_active` jadi **kebalikan dari** nilai `is_active` sekarang."

Penjelasan:

| Nilai Awal `$product->is_active` | `!$product->is_active` |
|---|---|
| `true` (1) | `false` (0) |
| `false` (0) | `true` (1) |

Jadi, dengan satu baris kode ini, kita bisa toggle status produk aktif <-> nonaktif. Tidak perlu nulis `if ... else ...`. Operator `!` yang melakukan kebalikannya.

### Analogi Toggle

Bayangkan tombol "Mute" di remote TV. Tombolnya **satu**, bukan dua (tidak ada tombol "Mute On" dan "Mute Off" terpisah). Saat kamu tekan:

- Kalau TV sedang berbunyi → diam (mute on).
- Kalau TV sedang diam → berbunyi lagi (mute off).

Satu tombol, dua efek berlawanan, tergantung kondisi sekarang. Itulah toggle.

Di aplikasi kita, tombol di view akan kita labeli **"Aktifkan"** kalau produk nonaktif, atau **"Nonaktifkan"** kalau produk aktif (ini bikin user experience lebih jelas). Tapi di belakang layar, semua tombol **mengirim ke route yang sama**, dan route itu menjalankan **logika toggle yang sama**. Hemat kode, mudah dipahami.

---

## 3. Rencana 3 Langkah

Kita kerjakan fitur toggle status dalam 3 langkah:

1. **Route**: tambah route `PATCH /admin/produk/{id}/status`.
2. **Controller**: tambah method `updateStatus($id)` yang toggle `is_active`.
3. **View**: ganti placeholder di `admin-index.blade.php` dengan form beneran.

---

## 4. Langkah 1: Tambah Route untuk Update Status

Buka file `routes/web.php`. Tambahkan satu route baru di grup admin:

```php
use App\Http\Controllers\ProductController;

// Halaman publik (customer)
Route::get('/produk', [ProductController::class, 'index'])->name('produk.index');
Route::get('/produk/{slug}', [ProductController::class, 'show'])->name('produk.show');

// Halaman admin
Route::get('/admin/produk', [ProductController::class, 'adminIndex'])->name('admin.produk.index');

// Aksi admin: toggle status produk
Route::patch('/admin/produk/{id}/status', [ProductController::class, 'updateStatus'])
    ->name('admin.produk.status');
```

Penjelasan baris baru:

| Bagian | Arti |
|---|---|
| `Route::patch(...)` | Method HTTP **PATCH** (untuk update sebagian field) |
| `'/admin/produk/{id}/status'` | URL dengan parameter ID. Contoh: `/admin/produk/3/status` |
| `[ProductController::class, 'updateStatus']` | Method yang dijalankan: `updateStatus()` |
| `->name('admin.produk.status')` | Beri nama route, supaya di view bisa pakai `route('admin.produk.status', $product->id)` |

### Kenapa Pakai Method HTTP `PATCH`?

Di REST (standar arsitektur web), method HTTP punya arti:

| Method | Arti | Contoh |
|---|---|---|
| `GET` | Ambil data (tidak ubah apa-apa) | Buka halaman daftar produk |
| `POST` | Buat data baru | Form tambah produk |
| `PUT` / `PATCH` | Update data yang sudah ada | Form edit produk |
| `DELETE` | Hapus data | Tombol hapus produk |

`PATCH` cocok untuk **update sebagian field** (dalam hal ini: cuma field `is_active`). `PUT` biasanya untuk update seluruh record, tapi dalam praktik Laravel, keduanya sering dipakai bergantian.

Sebenarnya kita bisa pakai `POST` saja (seperti route `restore` di materi 09). Tapi karena ini adalah **update status**, `PATCH` lebih **semantik** (lebih sesuai maksud). Selain itu, ini kesempatan bagus untuk memperkenalkan kamu pada method HTTP selain POST.

**Yang penting**: konsisten. Kalau route pakai `PATCH`, form di view juga harus pakai `@method('PATCH')`.

---

## 5. Langkah 2: Tambah Method `updateStatus()` di Controller

Buka `app/Http/Controllers/ProductController.php`. Tambahkan method baru:

```php
use Illuminate\Http\Request;

// ... class ProductController ...

public function updateStatus(Request $request, $id)
{
    $product = Product::findOrFail($id);

    // Toggle: kalau aktif jadi nonaktif, kalau nonaktif jadi aktif
    $product->is_active = !$product->is_active;
    $product->save();

    $status = $product->is_active ? 'aktif' : 'nonaktif';

    return redirect()->route('admin.produk.index')
        ->with('success', "Produk \"{$product->nama}\" sekarang {$status}.");
}
```

Penjelasan baris per baris:

### `public function updateStatus(Request $request, $id)`

Deklarasi method dengan dua parameter:

| Parameter | Isi | Dari mana |
|---|---|---|
| `Request $request` | Objek request HTTP (berisi data form, session, dll) | Laravel otomatis injek |
| `$id` | ID produk dari URL `{id}` | Dari route parameter |

`Request $request` sebenarnya tidak kita pakai isinya di method ini. Tapi tetap kita terima karena **konvensi Laravel**: setiap method controller yang menangani form biasanya menerima `Request` sebagai parameter pertama. Nanti kalau mau validasi atau ambil input, `$request` sudah siap.

### `$product = Product::findOrFail($id);`

Cari produk berdasarkan ID. Kalau tidak ketemu, otomatis 404.

**Penting**: di sini kita **tidak** pakai `withTrashed()`, karena produk yang sudah di-soft-delete (di sampah) **tidak boleh** di-toggle statusnya. Hanya produk aktif yang belum dihapus yang bisa diaktifkan/dinonaktifkan. Ini sengaja, karena produk di sampah statusnya sudah tidak relevan (tidak akan tampil di halaman publik regardless).

### `$product->is_active = !$product->is_active;`

Ini **baris toggle**. Operator `!` membalikkan nilai boolean:

- Kalau `is_active` awalnya `true` → jadi `false`.
- Kalau `is_active` awalnya `false` → jadi `true`.

Baris ini **mengubah property di object PHP**, tapi **belum** mengubah database. Database baru berubah saat kita panggil `$product->save()` di baris berikutnya.

### `$product->save();`

Simpan perubahan ke database. Laravel akan menjalankan query:

```sql
UPDATE products SET is_active = 1, updated_at = '...' WHERE id = 3;
```

Hanya field `is_active` dan `updated_at` yang diubah. Field lain (nama, harga, dll) tetap.

### `$status = $product->is_active ? 'aktif' : 'nonaktif';`

Bikin pesan dinamis. Setelah toggle, `$product->is_active` sudah berisi nilai baru. Kita cek nilainya untuk bikin kata "aktif" atau "nonaktif" yang ramah dibaca.

Ini pakai operator **ternary** `? :`, yang adalah versi singkat dari `if-else`:

```php
// Versi panjang:
if ($product->is_active) {
    $status = 'aktif';
} else {
    $status = 'nonaktif';
}

// Versi singkat (ternary):
$status = $product->is_active ? 'aktif' : 'nonaktif';
```

Keduanya sama. Ternary lebih padat, sering dipakai untuk kasus sederhana seperti ini.

### `return redirect()->route('admin.produk.index')`

Setelah toggle selesai, redirect balik ke **halaman admin index** (`/admin/produk`). Ini supaya admin bisa langsung melihat hasil perubahannya: badge di tabel berubah warna.

### `->with('success', "Produk \"{$product->nama}\" sekarang {$status}.")`

Bawa pesan sukses dinamis. Contoh hasilnya:

- `Produk "Tumbler Limited" sekarang aktif.`
- `Produk "Kopi Susu Vanilla" sekarang nonaktif.`

Tanda `\"` adalah escape karakter untuk menampilkan tanda kutip di dalam string yang juga dibatasi kutip. Alternatifnya: pakai single quote untuk string luar:

```php
->with('success', 'Produk "' . $product->nama . '" sekarang ' . $status . '.');
```

Tapi versi awal dengan interpolation `"{$product->nama}"` lebih ringkas dan lazim dipakai di Laravel.

---

## 6. Langkah 3: Ganti Placeholder di View dengan Form Beneran

Buka `resources/views/products/admin-index.blade.php`. Cari bagian kolom **Aksi** yang masih placeholder abu-abu:

```blade
{{-- Kolom Aksi: tombol kebalikan dari status --}}
<td class="border p-2">
    @if($product->is_active)
        <span class="text-gray-400 text-xs">(tombol nonaktifkan di tahap 6)</span>
    @else
        <span class="text-gray-400 text-xs">(tombol aktifkan di tahap 6)</span>
    @endif
</td>
```

**Ganti seluruh blok itu** dengan form yang sebenarnya:

```blade
{{-- Kolom Aksi: tombol toggle status --}}
<td class="border p-2">
    <form action="{{ route('admin.produk.status', $product->id) }}"
          method="POST"
          style="display:inline;">
        @csrf
        @method('PATCH')
        @if($product->is_active)
            <button type="submit"
                    onclick="return confirm('Yakin nonaktifkan produk ini? Produk tidak akan tampil di halaman publik.')"
                    class="bg-red-500 text-white px-3 py-1 rounded text-sm">
                Nonaktifkan
            </button>
        @else
            <button type="submit"
                    onclick="return confirm('Yakin aktifkan produk ini? Produk akan tampil di halaman publik.')"
                    class="bg-green-500 text-white px-3 py-1 rounded text-sm">
                Aktifkan
            </button>
        @endif
    </form>
</td>
```

Penjelasan per bagian:

### `<form action="{{ route('admin.produk.status', $product->id) }}" method="POST">`

- `route('admin.produk.status', $product->id)` = generate URL ke `/admin/produk/{id}/status`, dengan `$product->id` dipakai sebagai `{id}`. Contoh hasil: `/admin/produk/3/status`.
- `method="POST"` = form native HTML. Walaupun route kita PATCH, form tetap dikirim sebagai POST, lalu di-transform jadi PATCH lewat `@method('PATCH')` (baris berikutnya).

### `@csrf`

Token keamanan Laravel. Wajib di setiap form POST/PUT/PATCH/DELETE. Tanpa ini, Laravel menolak form dengan error **419 Page Expired**.

### `@method('PATCH')`

Karena HTML form **hanya mendukung GET dan POST**, kita tidak bisa langsung tulis `method="PATCH"`. Browser akan abaikan dan kirim sebagai POST.

Trick Laravel: tulis `method="POST"`, lalu pakai directive `@method('PATCH')`. Laravel akan menambahkan **hidden input** bernama `_method` bernilai `PATCH`. Saat request sampai ke Laravel, Laravel baca `_method` dan **menganggap request sebagai PATCH**, bukan POST.

Ini cara standar Laravel menangani method HTTP selain GET/POST. Sama seperti yang kamu pakai di materi 09 untuk form hapus (yang pakai `@method('DELETE')`).

### Logika Tombol: Label Sesuai Status

```blade
@if($product->is_active)
    <button ... class="bg-red-500 ...">Nonaktifkan</button>
@else
    <button ... class="bg-green-500 ...">Aktifkan</button>
@endif
```

Perhatikan:

- Kalau produk **aktif**, tombol yang tampil adalah **"Nonaktifkan"** (merah). Karena aksi relevan yang bisa dilakukan terhadap produk aktif adalah menonaktifkannya.
- Kalau produk **nonaktif**, tombol yang tampil adalah **"Aktifkan"** (hijau). Karena aksi relevan adalah mengaktifkannya.

**Logika tombol selalu kebalikan dari status saat ini.** Ini user experience yang lebih baik daripada menampilkan dua tombol sekaligus (yang bisa bingungkan admin).

Walaupun label berbeda, **kedua tombol mengirim ke route yang sama** (`admin.produk.status`) dan **eksekusi method controller yang sama** (`updateStatus`). Di controller, logika toggle `$product->is_active = !$product->is_active` akan menangani dua kasus sekaligus.

### `onclick="return confirm('...')"`

Dialog konfirmasi JavaScript. Pesan berbeda sesuai aksi:

- Untuk Nonaktifkan: "Yakin nonaktifkan produk ini? Produk tidak akan tampil di halaman publik."
- Untuk Aktifkan: "Yakin aktifkan produk ini? Produk akan tampil di halaman publik."

**Kenapa pesannya beda?** Karena konteks dan risiko berbeda. Menonaktifkan produk yang aktif bisa **menghilangkan produk dari etalase** (customer tidak bisa lihat). Mengaktifkan produk yang belum siap bisa **menampilkan produk setengah jadi ke publik** (risiko user bingung). Admin perlu sadar dampaknya sebelum klik OK.

### Pesan Flash Sukses di Atas Tabel

Supaya pesan dari redirect muncul, tambahkan blok ini di atas tabel (di dalam section content):

```blade
@if(session('success'))
    <div class="bg-green-100 border border-green-400 text-green-700 px-4 py-2 rounded mb-4">
        {{ session('success') }}
    </div>
@endif
```

Ini sama persis dengan pattern yang kamu pakai di materi 09 tahap 5. `session('success')` berisi pesan yang dikirim controller lewat `->with('success', ...)`. Blok `@if` supaya tidak error kalau session kosong.

---

## 7. File View `admin-index.blade.php` Lengkap Setelah Tahap 6

```blade
@extends('layouts.app')

@section('title', 'Kelola Produk (Admin)')

@section('content')
<div class="container mx-auto p-4">

    <h1 class="text-2xl font-bold mb-2">Kelola Produk (Admin)</h1>
    <p class="text-gray-600 mb-4">
        Daftar semua produk, baik yang aktif maupun yang masih dalam persiapan (nonaktif).
    </p>

    <div class="mb-4">
        <a href="{{ route('produk.index') }}" class="text-blue-600 underline">
            &larr; Lihat Halaman Publik
        </a>
    </div>

    {{-- Bonus ringkasan jumlah --}}
    <div class="flex gap-4 mb-4">
        <div class="bg-green-50 border border-green-200 px-4 py-2 rounded">
            <span class="text-green-700 font-bold">
                {{ $products->where('is_active', true)->count() }}
            </span>
            <span class="text-green-700 text-sm">Aktif</span>
        </div>

        <div class="bg-red-50 border border-red-200 px-4 py-2 rounded">
            <span class="text-red-700 font-bold">
                {{ $products->where('is_active', false)->count() }}
            </span>
            <span class="text-red-700 text-sm">Nonaktif</span>
        </div>
    </div>

    @if(session('success'))
        <div class="bg-green-100 border border-green-400 text-green-700 px-4 py-2 rounded mb-4">
            {{ session('success') }}
        </div>
    @endif

    @if($products->count() > 0)
        <table class="w-full border border-gray-300">
            <thead class="bg-gray-100">
                <tr>
                    <th class="border p-2 text-left">ID</th>
                    <th class="border p-2 text-left">Nama Produk</th>
                    <th class="border p-2 text-left">Harga</th>
                    <th class="border p-2 text-left">Stok</th>
                    <th class="border p-2 text-left">Kategori</th>
                    <th class="border p-2 text-left">Status</th>
                    <th class="border p-2 text-left">Aksi</th>
                </tr>
            </thead>
            <tbody>
                @foreach($products as $product)
                    <tr>
                        <td class="border p-2">{{ $product->id }}</td>
                        <td class="border p-2">{{ $product->nama }}</td>
                        <td class="border p-2">Rp {{ number_format($product->harga, 0, ',', '.') }}</td>
                        <td class="border p-2">{{ $product->stok }}</td>
                        <td class="border p-2">{{ $product->kategori }}</td>

                        <td class="border p-2">
                            @if($product->is_active)
                                <span class="bg-green-100 text-green-800 text-xs px-2 py-1 rounded-full">
                                    Aktif
                                </span>
                            @else
                                <span class="bg-red-100 text-red-800 text-xs px-2 py-1 rounded-full">
                                    Nonaktif
                                </span>
                            @endif
                        </td>

                        <td class="border p-2">
                            <form action="{{ route('admin.produk.status', $product->id) }}"
                                  method="POST"
                                  style="display:inline;">
                                @csrf
                                @method('PATCH')
                                @if($product->is_active)
                                    <button type="submit"
                                            onclick="return confirm('Yakin nonaktifkan produk ini? Produk tidak akan tampil di halaman publik.')"
                                            class="bg-red-500 text-white px-3 py-1 rounded text-sm">
                                        Nonaktifkan
                                    </button>
                                @else
                                    <button type="submit"
                                            onclick="return confirm('Yakin aktifkan produk ini? Produk akan tampil di halaman publik.')"
                                            class="bg-green-500 text-white px-3 py-1 rounded text-sm">
                                        Aktifkan
                                    </button>
                                @endif
                            </form>
                        </td>
                    </tr>
                @endforeach
            </tbody>
        </table>
    @else
        <div class="bg-yellow-100 border border-yellow-300 p-4 rounded">
            Belum ada produk. Tambahkan produk baru lewat halaman create.
        </div>
    @endif

</div>
@endsection
```

---

## 8. Bonus: Pakai Route-Model Binding (Versi Lebih Laravel)

Sebagai pengenalan pola yang lebih modern di Laravel, kita bisa menyederhanakan controller dengan **route-model binding**. Alih-alih menerima `$id` dan mencari produk sendiri, Laravel bisa otomatis inject objek `Product` ke method.

Di `routes/web.php`:

```php
Route::patch('/admin/produk/{product}/status', [ProductController::class, 'updateStatus'])
    ->name('admin.produk.status');
```

Perhatikan: parameter URL diganti dari `{id}` jadi `{product}`. Nama parameter harus sama dengan nama variabel di method controller.

Di controller:

```php
public function updateStatus(Request $request, Product $product)
{
    $product->is_active = !$product->is_active;
    $product->save();

    $status = $product->is_active ? 'aktif' : 'nonaktif';

    return redirect()->route('admin.produk.index')
        ->with('success', "Produk \"{$product->nama}\" sekarang {$status}.");
}
```

Perhatikan perbedaannya:

- Parameter kedua method adalah `Product $product` (tipe-hinted), bukan `$id`.
- Baris `$product = Product::findOrFail($id);` **hilang**. Laravel otomatis mencari produk berdasarkan ID di URL dan inject ke method.

**Kelebihan route-model binding**:

1. Kode lebih pendek.
2. Otomatis 404 kalau produk tidak ketemu (sama seperti `findOrFail`).
3. Konvensi Laravel yang umum dipakai di projek nyata.

**Kapan pakai cara manual (versi pertama)?**

- Kalau kamu baru belajar Laravel dan mau paham dulu apa yang terjadi di balik layar.
- Kalau ada logika kustom sebelum find (misal cek permission, cache, dll).

Untuk pemula, dua-duanya sah. Mulai dengan yang pertama (manual `$id`) supaya paham konsepnya, lalu pelan-pelan migrasi ke route-model binding begitu nyaman.

---

## 9. Uji Coba: Cek Apakah Fitur Toggle Berfungsi

### Langkah A: Pastikan Ada Data Variatif

Cek di Tinker bahwa di tabel ada campuran produk aktif dan nonaktif:

```bash
php artisan tinker
```

```php
Product::select('id', 'nama', 'is_active')->get();
exit
```

### Langkah B: Buka Halaman Admin

Akses URL:

```
http://localhost:8000/admin/produk
```

### Langkah C: Tes Nonaktifkan Produk Aktif

1. Cari baris produk dengan badge hijau "Aktif".
2. Klik tombol merah **"Nonaktifkan"**.
3. Browser tampilkan dialog: "Yakin nonaktifkan produk ini? Produk tidak akan tampil di halaman publik."
4. Klik **OK**.
5. Halaman refresh. Sekarang:
   - Pesan sukses hijau: `Produk "Kopi Susu Vanilla" sekarang nonaktif.`
   - Badge produk itu berubah dari hijau jadi **merah**.
   - Tombol di kolom Aksi berubah dari merah "Nonaktifkan" jadi hijau **"Aktifkan"**.

### Langkah D: Tes Aktifkan Produk Nonaktif

1. Cari baris produk dengan badge merah "Nonaktif".
2. Klik tombol hijau **"Aktifkan"**.
3. Dialog: "Yakin aktifkan produk ini? Produk akan tampil di halaman publik."
4. Klik **OK**.
5. Halaman refresh. Badge berubah dari merah jadi **hijau**, tombol berubah jadi merah "Nonaktifkan".

### Langkah E: Verifikasi di Halaman Publik

Buka:

```
http://localhost:8000/produk
```

Pastikan produk yang baru saja kamu **aktifkan** sekarang **muncul** di halaman publik. Dan produk yang baru kamu **nonaktifkan** sudah **tidak muncul**.

### Langkah F: Verifikasi di Database (Opsional)

Cek di Tinker:

```bash
php artisan tinker
```

```php
$product = Product::find(3);
echo $product->is_active ? 'aktif' : 'nonaktif';
exit
```

Hasilnya harus konsisten dengan apa yang kamu lihat di halaman admin.

Kalau semua langkah berhasil, **fitur toggle status sudah berfungsi sempurna**.

---

## 10. Hal-hal Penting yang Sering Bikin Pemula Bingung

### a. Form Harus Pakai `@method('PATCH')` Kalau Route Pakai `PATCH`

Kalau di route kamu pakai `Route::patch(...)`, maka di view **wajib** pakai `@method('PATCH')`. Tanpa itu:

- Form dikirim sebagai POST.
- Laravel cari route `Route::post('/admin/produk/{id}/status', ...)`, tidak ketemu.
- Hasilnya: error **405 Method Not Allowed**.

**Debugging**: kalau kamu dapat error 405 setelah klik tombol, pertama cek apakah `@method('PATCH')` ada di form.

### b. `@csrf` Wajib, `@method` Hanya untuk Method Non-GET/POST

| Directive | Kapan dipakai |
|---|---|
| `@csrf` | Setiap form POST/PUT/PATCH/DELETE. Wajib. |
| `@method('PUT')` | Hanya kalau route pakai PUT |
| `@method('PATCH')` | Hanya kalau route pakai PATCH |
| `@method('DELETE')` | Hanya kalau route pakai DELETE |

Form GET (link biasa) tidak butuh `@csrf` atau `@method`.

### c. Toggle vs Set Eksplisit

Kita pakai toggle (`!$product->is_active`). Ini bekerja kalau **satu tombol** yang melakukan toggle. Tapi kalau kamu punya **dua tombol** terpisah (satu untuk "jadikan aktif", satu untuk "jadikan nonaktif"), kamu **tidak bisa** pakai toggle. Kamu harus set eksplisit:

```php
// Set eksplisit (alternatif kalau ada dua tombol terpisah)
$product->is_active = true;   // untuk tombol "Aktifkan"
$product->is_active = false;  // untuk tombol "Nonaktifkan"
```

Tapi di materi ini, kita pakai toggle karena cuma satu tombol per produk (labelnya disesuaikan, tapi aksinya sama). Ini lebih hemat kode.

### d. Method HTTP Bukan Urusan Keamanan

Salah persepsi umum: pemula mengira PATCH lebih aman dari POST. **Tidak**. Method HTTP itu cuma **label semantik**, bukan mekanisme keamanan. Yang melindungi form adalah:

- `@csrf` (token anti-forgery).
- Middleware autentikasi/otorisasi (siapa yang boleh akses route).

Jadi pakai POST atau PATCH, kalau keduanya tidak diproteksi middleware auth, sama-sama bisa diakses siapa saja. **Jangan andalkan method HTTP untuk keamanan**.

### e. Produk yang Sudah Di-soft-Delete Tidak Bisa Di-toggle Status

Di controller kita pakai `Product::findOrFail($id)`, tanpa `withTrashed()`. Ini sengaja. Konsekuensinya:

- Kalau admin mencoba toggle status produk yang ada di tong sampah (dengan ID yang benar), akan **404**.
- Kenapa? Karena `findOrFail` tidak menemukan produk yang sudah di-soft-delete.
- Ini **perilaku yang diinginkan**. Produk di sampah statusnya sudah tidak relevan. Admin harus restore dulu kalau mau atur status lagi.

### f. Cache Route Kadang Perlu Dibersihkan

Kalau kamu sudah tambah route baru, tapi dapat error "route not defined" di view, jalankan:

```bash
php artisan route:clear
```

Ini menghapus cache route lama. Setelah itu, Laravel baca ulang `routes/web.php` dan route baru akan dikenali.

### g. Jangan Pakai Link GET untuk Aksi yang Mengubah Data

Hindari:

```blade
<a href="/admin/produk/3/status">Aktifkan</a>  <!-- ❌ BAHAYA -->
```

Kenapa?

1. Link adalah GET request. Tidak cocok untuk aksi yang ubah data.
2. Crawler search engine bisa klik link itu dan mengubah status ribuan produk tanpa sengaja.
3. Preview browser (yang muncul saat hover) bisa trigger GET request.

Selalu pakai form dengan POST (atau PATCH/DELETE via `@method`) untuk aksi yang ubah data.

---

## 11. Apa yang Sudah Bisa Kamu Lakukan Setelah Tahap 6?

Setelah tahap 6, alur lengkap yang bisa admin lakukan:

1. Buka halaman `/admin/produk`.
2. Lihat semua produk dengan badge status yang jelas.
3. Klik tombol **"Aktifkan"** atau **"Nonaktifkan"** sesuai kebutuhan.
4. Konfirmasi dialog JavaScript (dengan pesan kontekstual).
5. Halaman refresh, badge berubah warna, muncul pesan sukses.
6. Perubahan tercermin di **halaman publik**: produk aktif muncul, produk nonaktif hilang.

**Siklus hidup status produk**:

```
Produk dibuat (create) → default is_active = 0 (nonaktif)
                          ↓
                  Admin klik "Aktifkan"
                          ↓
                    is_active = 1 (aktif, tampil di publik)
                          ↓
                  Admin klik "Nonaktifkan"
                          ↓
                    is_active = 0 (nonaktif, sembunyi)
                          ↓
                      ... (bisa berulang)
```

Produk bisa bolak-balik antara aktif dan nonaktif sesering yang dibutuhkan. Tidak ada data yang hilang, hanya visibility (tampil/tidak) yang berubah.

**Yang bisa kamu lakukan di akhir materi 10**:

- Membuat produk dengan status default nonaktif.
- Mengubah status produk kapan saja dari halaman admin.
- Memisahkan halaman publik (hanya produk aktif) dan halaman admin (semua produk).
- Mencegah user mengakses produk nonaktif lewat URL langsung (404).
- Menampilkan badge visual yang jelas di halaman admin.

---

## Ringkasan Tahap 6

| Hal | Isi |
|---|---|
| Tujuan | Aktifkan/nonaktifkan produk dari halaman admin |
| Konsep | Toggle: `is_active = !is_active` |
| Route baru | `Route::patch('/admin/produk/{id}/status', ...)->name('admin.produk.status')` |
| Method controller | `updateStatus($id)` dengan `Product::findOrFail($id)`, toggle, `save()` |
| View | Form dengan `@csrf` + `@method('PATCH')`, tombol warna sesuai aksi |
| Method HTTP | `PATCH` (semantik untuk update sebagian field) |
| Konfirmasi JS | `onclick="return confirm('...')"` dengan pesan kontekstual |
| Redirect | Balik ke halaman admin index dengan pesan sukses |
| Bonus | Route-model binding: `Product $product` sebagai parameter |

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: tahap 7 (terakhir), ringkasan penuh materi 10 + perbandingan dengan soft delete + best practice?**

Kalau iya, tahap 7 (penutup) kita akan:

1. Recap keseluruhan materi 10 dari tahap 1 sampai 6 (peta jalan).
2. Tabel perbandingan lengkap antara **status aktif/nonaktif** vs **soft delete** (kapan pakai yang mana).
3. Best practice di projek nyata: contoh kasus kombinasi dua sistem (produk stok habis, produk seasonal, dll).
4. Daftar istilah penting yang sudah kamu pelajari.
5. Referensi lanjutan: apa yang bisa dipelajari setelah ini.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
