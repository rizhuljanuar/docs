# Tahap 6 — Fitur Force Delete & Ringkasan Penuh Materi Soft Delete

> Materi: Soft Delete Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk
> Tahap: FINAL (terakhir dari 6 tahap)

---

## 1. Analogi: Petugas Kebersihan Datang

Sampai tahap 5, kita sudah punya sistem tempat sampah yang lengkap:

- Bisa membuang barang ke tempat sampah (soft delete).
- Bisa melihat isi tempat sampah (halaman `/produk/sampah`).
- Bisa mengambil barang dari tempat sampah (restore).

Tapi tempat sampah itu **harus dikosongkan juga** kalau tidak mau penuh. Masalahnya:

- Produk-produk yang sudah lama dihapus dan **tidak akan pernah dijual lagi** masih memenuhi tempat sampah.
- Database jadi **berat** karena banyak data sampah yang tidak berguna.
- Pemilik toko mungkin **mau benar-benar menghapus** produk tertentu (misalnya produk yang recalled, produk yang melanggar, atau data dummy saat testing).

Nah, **force delete** itu seperti **petugas kebersihan datang membawa kantong sampah pergi**. Setelah ini, barang **benar-benar hilang selamanya**. Tidak bisa dikembalikan dengan restore.

Karena sifatnya yang permanen dan tidak bisa dibatalkan, force delete wajib dilakukan dengan **hati-hati** dan **konfirmasi ekstra**. Begitu tombol diklik, tidak ada jalan kembali.

---

## 2. Apa yang Terjadi Secara Teknis Saat Force Delete?

Sebelum force delete (produk masih di sampah):

| id | nama | harga | deleted_at |
|---|---|---|---|
| 5 | Kopi Susu Vanilla | 18000 | 2026-07-18 11:02:15 |

Setelah force delete:

> Baris produk dengan id=5 **hilang selamanya** dari tabel `products`.

Query SQL yang Laravel jalankan saat force delete:

```sql
DELETE FROM products WHERE id = 5;
```

Ini adalah `DELETE FROM` yang sebenarnya, sama seperti `$product->delete()` **sebelum** kita pakai trait `SoftDeletes`. Barisnya hilang dari database, bukan cuma ditandai.

Setelah force delete:
- `Product::find(5)` → tetap `null`.
- `Product::withTrashed()->find(5)` → juga `null` (baris sudah tidak ada).
- `Product::onlyTrashed()->find(5)` → juga `null`.
- Tidak ada cara untuk mengembalikan produk ini. Datanya benar-benar pergi.

**Kontras dengan soft delete**:

| Aksi | Query SQL | Baris di tabel |
|---|---|---|
| Soft delete | `UPDATE ... SET deleted_at = NOW()` | Masih ada, `deleted_at` terisi |
| Restore | `UPDATE ... SET deleted_at = NULL` | Masih ada, `deleted_at` NULL lagi |
| **Force delete** | `DELETE FROM ... WHERE id = ?` | **Hilang selamanya** |

---

## 3. ⚠️ Aturan Emas Force Delete

Sama seperti restore, force delete **memerlukan** `withTrashed()` untuk menemukan produk yang sudah di-soft-delete.

```php
// ❌ SALAH — produk tidak ketemu (sudah di-soft-delete)
$product = Product::findOrFail(5);
$product->forceDelete();

// ✅ BENAR — produk ditemukan
$product = Product::withTrashed()->findOrFail(5);
$product->forceDelete();
```

Tanpa `withTrashed()`, `findOrFail(5)` akan 404 karena query biasa tidak melihat produk sampah.

---

## 4. Rencana 3 Langkah Force Delete (Plus Ringkasan Akhir)

1. **Route**: tambah route `DELETE /produk/{id}/force-delete`.
2. **Controller**: tambah method `forceDelete($id)` yang pakai `withTrashed()->findOrFail($id)->forceDelete()`.
3. **View**: tambah tombol **"Hapus Permanen"** (warna merah) di tabel sampah, dengan **konfirmasi ganda** karena aksi ini tidak bisa dibatalkan.

Setelah tiga langkah ini, kita tutup materi dengan **Ringkasan Penuh** dan **Diagram Alur Soft Delete**.

---

## 5. Langkah 1: Tambah Route untuk Force Delete

Buka `routes/web.php`. Tambahkan satu route baru:

```php
Route::delete('/produk/{id}/force-delete', [ProductController::class, 'forceDelete'])
    ->name('produk.forceDelete');
```

Letakkan di grup route produk. Berikut posisi lengkap semua route yang sudah kita punya sejak tahap 1-5:

```php
Route::get('/produk', [ProductController::class, 'index'])->name('produk.index');
Route::get('/produk/create', [ProductController::class, 'create'])->name('produk.create');
Route::post('/produk', [ProductController::class, 'store'])->name('produk.store');

// Route sampah — HARUS di atas route /produk/{id}
Route::get('/produk/sampah', [ProductController::class, 'trash'])->name('produk.trash');

// Route restore & force delete
Route::post('/produk/{id}/restore', [ProductController::class, 'restore'])->name('produk.restore');
Route::delete('/produk/{id}/force-delete', [ProductController::class, 'forceDelete'])->name('produk.forceDelete');

Route::get('/produk/{id}', [ProductController::class, 'show'])->name('produk.show');
Route::get('/produk/{id}/edit', [ProductController::class, 'edit'])->name('produk.edit');
Route::put('/produk/{id}', [ProductController::class, 'update'])->name('produk.update');
Route::delete('/produk/{id}', [ProductController::class, 'destroy'])->name('produk.destroy');
```

Penjelasan baris baru:

| Bagian | Arti |
|---|---|
| `Route::delete(...)` | Method HTTP **DELETE** (bukan POST). Force delete adalah aksi "hapus permanen", jadi DELETE lebih sesuai secara semantik REST. |
| `'/produk/{id}/force-delete'` | URL dengan parameter ID. Contoh: `/produk/5/force-delete`. |
| `[ProductController::class, 'forceDelete']` | Method yang dijalankan: `forceDelete()`. |
| `->name('produk.forceDelete')` | Beri nama route, supaya di view bisa pakai `route('produk.forceDelete', $product->id)`. |

### Kenapa Force Delete Pakai `DELETE`, Bukan `POST`?

Walaupun `POST` juga bisa, `DELETE` lebih tepat karena:

- Force delete **benar-benar menghapus data** (sama seperti destroy di CRUD biasa).
- Konsisten dengan route `Route::delete('/produk/{id}', ...)` untuk soft delete (destroy) — dua-duanya aksi hapus.
- Membantu developer lain (atau dirimu di masa depan) langsung paham: "ini aksi penghapusan".

Di view, kita pakai `@method('DELETE')` karena form HTML hanya mendukung GET/POST secara native.

---

## 6. Langkah 2: Tambah Method `forceDelete()` di Controller

Buka `app/Http/Controllers/ProductController.php`. Tambahkan method baru:

```php
public function forceDelete($id)
{
    $product = Product::withTrashed()->findOrFail($id);
    $nama = $product->nama;
    $product->forceDelete();

    return redirect()->route('produk.trash')
        ->with('success', "Produk \"{$nama}\" berhasil dihapus permanen.");
}
```

Penjelasan baris per baris:

### `Product::withTrashed()->findOrFail($id)`

Sama seperti di restore: cari produk termasuk yang sudah di-soft-delete. Tanpa `withTrashed()`, produk sampah tidak akan ditemukan.

### `$nama = $product->nama;`

Simpan nama produk ke variabel **sebelum** force delete. Kenapa?

Karena setelah `$product->forceDelete()`, instance model `$product` **masih ada di memori PHP**, tapi data di database sudah hilang. Beberapa field masih bisa diakses (karena sudah ter-load ke memori), tapi untuk amannya dan kejelasan kode, kita simpan dulu ke variabel terpisah. Begitu pesan sukses ditampilkan, kita tahu persis produk apa yang baru saja dihapus permanen.

### `$product->forceDelete()`

Ini method khusus dari trait `SoftDeletes`. Fungsinya: **hapus baris dari database secara permanen**. Menjalankan `DELETE FROM products WHERE id = ?`.

**Perbedaan dengan `delete()`**:

- `delete()` pada model `SoftDeletes` → lakukan soft delete (isi `deleted_at`).
- `forceDelete()` pada model `SoftDeletes` → hapus permanen (`DELETE FROM`).

Jadi `forceDelete()` adalah satu-satunya cara untuk benar-benar menghapus produk dari database, ketika model kita sudah pakai trait `SoftDeletes`.

### `return redirect()->route('produk.trash')`

Redirect balik ke halaman sampah, dengan pesan sukses yang menyebut nama produk yang baru saja dihapus permanen.

---

## 7. Langkah 3: Tambah Tombol "Hapus Permanen" di View `trash.blade.php`

Buka `resources/views/products/trash.blade.php`. Kita akan tambah tombol **"Hapus Permanen"** di kolom Aksi (yang sudah ada tombol "Kembalikan" dari tahap 5).

Cari bagian kolom Aksi di dalam `@foreach`, modifikasi jadi seperti ini:

```blade
<td class="border p-2 whitespace-nowrap">
    <!-- Tombol Kembalikan (hijau) - dari tahap 5 -->
    <form action="{{ route('produk.restore', $product->id) }}" method="POST"
          style="display:inline;">
        @csrf
        <button type="submit"
                onclick="return confirm('Yakin kembalikan produk ini?')"
                class="bg-green-500 text-white px-3 py-1 rounded text-sm">
            Kembalikan
        </button>
    </form>

    <!-- Tombol Hapus Permanen (merah) - tahap 6 -->
    <form action="{{ route('produk.forceDelete', $product->id) }}" method="POST"
          style="display:inline;"
          onsubmit="return konfirmasiForceDelete('{{ addslashes($product->nama) }}');">
        @csrf
        @method('DELETE')
        <button type="submit"
                class="bg-red-600 text-white px-3 py-1 rounded text-sm">
            Hapus Permanen
        </button>
    </form>
</td>
```

Penjelasan bagian baru (tombol Hapus Permanen):

#### `<form action="{{ route('produk.forceDelete', $product->id) }}" method="POST">`

- `route('produk.forceDelete', $product->id)` = generate URL `/produk/{id}/force-delete`.
- `method="POST"` = form dasarnya POST, lalu kita spoof jadi DELETE dengan `@method('DELETE')`.

#### `@method('DELETE')`

Karena route force delete pakai `Route::delete(...)`, kita harus kirim request sebagai DELETE. Tapi form HTML native hanya support GET/POST. `@method('DELETE')` menambah field hidden `_method=DELETE` yang dibaca Laravel untuk mengubah request menjadi DELETE.

#### `onsubmit="return konfirmasiForceDelete('{{ addslashes($product->nama) }}');"`

Ini **konfirmasi ganda** yang lebih ketat daripada `confirm()` biasa. Kita pakai JavaScript custom di bagian bawah view (lihat langkah berikutnya).

`addslashes($product->nama)` = tambah backslash sebelum tanda kutip di nama produk, supaya nama produk dengan tanda kutip (misalnya: `Roti "Special"`) tidak merusak string JavaScript.

#### Tombol warna **merah** (`bg-red-600`)

Merah = aksi berbahaya / tidak bisa dibatalkan. Berbeda dari tombol restore yang hijau (aksi positif).

---

## 8. Tambahkan JavaScript Konfirmasi Ganda

Di bagian bawah view `trash.blade.php` (masih di dalam `@section('content')` atau di `@push('scripts')` kalau layout kamu punya stack scripts), tambahkan JavaScript ini:

```blade
<script>
function konfirmasiForceDelete(namaProduk) {
    // Tahap 1: konfirmasi pertama dengan informasi yang jelas
    var pesan = 'PERINGATAN:\n\n' +
        'Kamu akan menghapus permanen produk: "' + namaProduk + '"\n\n' +
        'Produk ini akan hilang selamanya dari database.\n' +
        'Tidak bisa dikembalikan dengan restore.\n\n' +
        'Klik OK untuk lanjut, atau Cancel untuk batal.';

    if (!confirm(pesan)) {
        return false;
    }

    // Tahap 2: konfirmasi kedua, ketik nama produk untuk konfirmasi
    var input = prompt('Untuk konfirmasi, ketik ulang nama produk: "' + namaProduk + '"');

    if (input !== namaProduk) {
        alert('Nama tidak cocok. Penghapusan permanen dibatalkan.');
        return false;
    }

    // Lolos dua konfirmasi, form dikirim
    return true;
}
</script>
```

**Kenapa konfirmasi ganda?** Karena force delete adalah **aksi tidak reversibel**. Sekali diklik, data pergi selamanya. Konfirmasi ganda:

1. **Tahap 1**: dialog `confirm()` dengan peringatan jelas. Bilang "OK" atau "Cancel".
2. **Tahap 2**: dialog `prompt()` yang minta admin **ketik ulang nama produk**. Kalau tidak persis sama, form dibatalkan.

Ini seperti saat kamu menghapus repository GitHub penting: GitHub minta kamu ketik nama repo untuk konfirmasi. Pola yang sama kita pakai di sini, supaya admin **benar-benar sadar** apa yang dilakukannya.

Untuk pemula: kalau kamu merasa dua konfirmasi terlalu ribet, kamu bisa pakai satu konfirmasi saja (`confirm()` seperti di tombol restore). Tapi untuk data produksi yang penting, **dua konfirmasi sangat disarankan**.

---

## 9. View `trash.blade.php` Final Lengkap

Berikut versi final `trash.blade.php` setelah semua tahap 1-6:

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
        Datanya masih tersimpan, bisa dikembalikan, atau dihapus permanen.
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
                        <td class="border p-2 whitespace-nowrap">
                            <form action="{{ route('produk.restore', $product->id) }}"
                                  method="POST" style="display:inline;">
                                @csrf
                                <button type="submit"
                                        onclick="return confirm('Yakin kembalikan produk ini?')"
                                        class="bg-green-500 text-white px-3 py-1 rounded text-sm">
                                    Kembalikan
                                </button>
                            </form>

                            <form action="{{ route('produk.forceDelete', $product->id) }}"
                                  method="POST" style="display:inline;"
                                  onsubmit="return konfirmasiForceDelete('{{ addslashes($product->nama) }}');">
                                @csrf
                                @method('DELETE')
                                <button type="submit"
                                        class="bg-red-600 text-white px-3 py-1 rounded text-sm">
                                    Hapus Permanen
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

<script>
function konfirmasiForceDelete(namaProduk) {
    var pesan = 'PERINGATAN:\n\n' +
        'Kamu akan menghapus permanen produk: "' + namaProduk + '"\n\n' +
        'Produk ini akan hilang selamanya dari database.\n' +
        'Tidak bisa dikembalikan dengan restore.\n\n' +
        'Klik OK untuk lanjut, atau Cancel untuk batal.';

    if (!confirm(pesan)) {
        return false;
    }

    var input = prompt('Untuk konfirmasi, ketik ulang nama produk: "' + namaProduk + '"');

    if (input !== namaProduk) {
        alert('Nama tidak cocok. Penghapusan permanen dibatalkan.');
        return false;
    }

    return true;
}
</script>
@endsection
```

---

## 10. Uji Coba: Cek Apakah Fitur Force Delete Berfungsi

### Langkah A: Pastikan Ada Produk di Sampah

Akses `/produk/sampah`. Pastikan ada minimal 1 produk sampah.

### Langkah B: Klik "Hapus Permanen"

Klik tombol merah **"Hapus Permanen"** di salah satu baris.

### Langkah C: Konfirmasi Tahap 1

Dialog muncul: *"Kamu akan menghapus permanen produk... Klik OK atau Cancel."*

Klik **OK**.

### Langkah D: Konfirmasi Tahap 2

Dialog prompt muncul: *"Untuk konfirmasi, ketik ulang nama produk..."*

Ketik nama produk **persis sama** (case-sensitive), klik **OK**.

### Langkah E: Periksa Hasil

Setelah redirect:

1. Pesan sukses hijau: `Produk "Kopi Susu Vanilla" berhasil dihapus permanen.`
2. Produk hilang dari daftar sampah.

### Langkah F: Verifikasi Produk Benar-benar Hilang

Buka tinker:

```bash
php artisan tinker
```

```php
Product::find(5);                  // null
Product::withTrashed()->find(5);   // null juga
Product::onlyTrashed()->find(5);   // null juga
exit
```

Ketiga query harus return `null`. Produk benar-benar hilang dari database. Tidak bisa dikembalikan.

Kalau semua langkah berhasil, **fitur force delete sudah berfungsi**.

---

## 11. Hal-hal Penting yang Sering Bikin Pemula Bingung

### a. `delete()` vs `forceDelete()` di Model dengan SoftDeletes

| Method | Pada model SoftDeletes | Pada model TANPA SoftDeletes |
|---|---|---|
| `delete()` | Lakukan soft delete (isi `deleted_at`) | Hapus permanen (`DELETE FROM`) |
| `forceDelete()` | Hapus permanen (`DELETE FROM`) | Sama seperti `delete()` |

**Penting**: kalau model **tidak** pakai `SoftDeletes`, `forceDelete()` tetap bisa dipanggil dan efeknya sama dengan `delete()` (hapus permanen). Tapi kalau model **pakai** `SoftDeletes`, kamu **wajib** pakai `forceDelete()` untuk hapus permanen.

### b. Force Delete Tidak Bisa Dibatalkan

Tidak ada `unforceDelete()`. Tidak ada "sampah kedua". Setelah `forceDelete()`, data pergi selamanya. Itu sebabnya kita pakai konfirmasi ganda di view.

### c. Apa yang Terjadi dengan Data Terkait?

Kalau kamu punya relasi (misalnya `Product` punya banyak `Review`), force delete pada produk **TIDAK OTOMATIS** menghapus review. Review-review itu akan tetap ada, tapi dengan `product_id` yang sudah tidak punya induk.

Untuk mengatasi ini, pelajari **foreign key constraint dengan `onDelete('cascade')`** atau event Eloquent (`deleting` / `deleted`). Ini materi tingkat menengah, di luar scope materi dasar ini.

### d. Force Delete pada Instance dari Query Biasa

Kalau kamu force-delete produk yang sudah di-soft-delete, kamu **harus** pakai `withTrashed()`. Tapi kalau kamu force-delete produk yang belum di-soft-delete (masih aktif), `findOrFail()` biasa cukup:

```php
// Produk aktif (belum dihapus), force delete langsung
$product = Product::findOrFail(3);
$product->forceDelete();  // OK, baris hilang permanen
```

Praktik ini **jarang dipakai** di UI, karena admin biasanya force-delete dari halaman sampah. Tapi penting untuk dipahami.

### e. Method HTTP: `DELETE` untuk Force Delete

Walaupun `POST` bisa dipakai, `DELETE` lebih sesuai secara semantik. Route Laravel dan helper `route()` tetap bekerja dengan baik. Yang penting konsisten.

### f. Jangan Lupa `@method('DELETE')` di Form

Kalau kamu pakai `Route::delete(...)` di route, view harus pakai `@method('DELETE')` di dalam form. Tanpa ini, form akan dikirim sebagai POST, dan route tidak cocok → **405 Method Not Allowed**.

---

## 12. 🎉 RINGKASAN PENUH MATERI SOFT DELETE

Selamat! Kamu sudah menyelesaikan semua 6 tahap materi **Soft Delete Produk**. Mari kita rangkum semuanya supaya kamu punya gambaran utuh.

### a. Tiga Istilah Kunci (Hafalkan Ini!)

| Istilah | Apa yang Terjadi | Bisa Dibatalkan? |
|---|---|---|
| **Soft Delete** | `deleted_at` diisi timestamp. Baris tetap ada. | Bisa, dengan restore |
| **Restore** | `deleted_at` di-set jadi NULL. Produk "hidup" lagi. | Ya (bisa soft-delete lagi) |
| **Force Delete** | Baris dihapus permanen dari tabel. | **TIDAK**, selamanya |

### b. Yang Dilakukan di Setiap Tahap

| Tahap | Fokus | Output |
|---|---|---|
| **Tahap 1** | Konsep | Paham analogi tempat sampah, masalah hapus permanen |
| **Tahap 2** | Persiapan | Migration `softDeletes()` + trait `use SoftDeletes` di model |
| **Tahap 3** | Soft delete di controller | `$product->delete()` otomatis soft delete (tanpa ubah kode) |
| **Tahap 4** | Halaman sampah | Route + method `trash()` + view `trash.blade.php` pakai `onlyTrashed()` |
| **Tahap 5** | Restore | Route + method `restore()` + tombol hijau pakai `withTrashed()->restore()` |
| **Tahap 6** | Force delete | Route + method `forceDelete()` + tombol merah pakai `withTrashed()->forceDelete()` |

### c. Method-Method Penting dari Trait SoftDeletes

| Method | Fungsi |
|---|---|
| `delete()` | Soft delete (isi `deleted_at`) |
| `restore()` | Kembalikan produk dari sampah (`deleted_at = NULL`) |
| `forceDelete()` | Hapus permanen (`DELETE FROM`) |
| `trashed()` (instance) | Cek apakah instance ini sudah di-soft-delete (return bool) |
| `onlyTrashed()` (query) | Ambil HANYA yang sudah di-soft-delete |
| `withTrashed()` (query) | Ambil SEMUA, termasuk yang di-soft-delete |

### d. Diagram Alur Lengkap Soft Delete

```
                  ┌──────────────────┐
                  │  PRODUK DIBUAT   │
                  │  (Create)        │
                  │  deleted_at=NULL │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │  PRODUK AKTIF    │ ←───────┐
                  │  (muncul di      │         │
                  │  /produk)        │         │
                  └────────┬─────────┘         │
                           │                   │
                  klik "Hapus" (tahap 3)      │
                  $product->delete()           │
                           │                   │
                           ▼                   │
                  ┌──────────────────┐         │
                  │  PRODUK DI       │         │
                  │  SOFT DELETE     │         │
                  │  deleted_at=NOW  │         │
                  └────────┬─────────┘         │
                           │                   │
                  tampil di /produk/sampah     │
                           │                   │
              ┌────────────┴────────────┐      │
              │                         │      │
              ▼                         ▼      │
    klik "Kembalikan"          klik "Hapus Permanen"
    (tahap 5)                  (tahap 6)
    ->restore()                ->forceDelete()
              │                         │
              │                         ▼
              │              ┌──────────────────┐
              │              │  BARIS HILANG    │
              │              │  DARI DATABASE   │
              │              │  (selamanya)     │
              │              └──────────────────┘
              │
              └──────────────────────────────┐
                                             │
                              deleted_at=NULL│
                                             │
                  (kembali ke PRODUK AKTIF)──┘
```

### e. Apa yang Bisa Admin Lakukan Sekarang?

Dengan semua 6 tahap selesai, admin bisa:

1. **Lihat daftar produk aktif** di `/produk`.
2. **Hapus produk** (soft delete) → produk pindah ke sampah.
3. **Lihat isi sampah** di `/produk/sampah`.
4. **Kembalikan produk** dari sampah ke daftar aktif.
5. **Hapus produk permanen** dari sampah (force delete).
6. **Bolak-balik** antara aktif dan sampah sesering yang dibutuhkan.

### f. Manfaat yang Didapat

| Manfaat | Detail |
|---|---|
| **Aman dari salah klik** | Soft delete = tidak hilang selamanya, bisa di-restore |
| **Produk stok habis bisa dijual lagi** | Cukup restore, tidak perlu input ulang |
| **Data transaksi lama tetap nyambung** | Produk yang dihapus masih ada di database |
| **Kontrol penuh atas data** | Bisa soft delete (reversible) atau force delete (permanen) |
| **Database tidak penuh sampah** | Admin bisa force delete produk yang benar-benar tidak dibutuhkan |

---

## 13. File-File yang Sudah Kamu Buat / Ubah di Materi Ini

Sebagai referensi, berikut semua file yang berubah selama materi Soft Delete:

### File Baru
- `database/migrations/xxxx_xx_xx_xxxxxx_add_deleted_at_to_products_table.php` (tahap 2)
- `resources/views/products/trash.blade.php` (tahap 4-6)

### File yang Diedit
- `app/Models/Product.php` — tambah `use SoftDeletes;` (tahap 2)
- `app/Http/Controllers/ProductController.php` — tambah 3 method baru:
  - `trash()` (tahap 4)
  - `restore($id)` (tahap 5)
  - `forceDelete($id)` (tahap 6)
  - Ubah `destroy()` pesan sukses (tahap 3)
- `routes/web.php` — tambah 3 route baru:
  - `Route::get('/produk/sampah', ...)` (tahap 4)
  - `Route::post('/produk/{id}/restore', ...)` (tahap 5)
  - `Route::delete('/produk/{id}/force-delete', ...)` (tahap 6)
- `resources/views/products/index.blade.php` — tambah tombol "Tong Sampah" (tahap 4)

---

## 14. Penutup & Langkah Selanjutnya

Selamat! Kamu sudah menyelesaikan materi **Soft Delete Produk** dari awal sampai akhir.

Yang sudah kamu kuasai:
- Konsep soft delete dan analogi tempat sampah.
- Cara menyiapkan migration dan trait SoftDeletes.
- Cara soft delete bekerja otomatis di controller (tanpa banyak ubahan kode).
- Membuat halaman sampah untuk melihat produk yang dihapus.
- Membuat fitur restore (kembalikan produk ke aktif).
- Membuat fitur force delete (hapus permanen dengan konfirmasi ganda).

### Saran Latihan Mandiri

Untuk memperdalam pemahaman, coba kerjakan latihan berikut:

1. **Tambah kolom "Stok" di tabel sampah**, supaya admin bisa lihat stok produk sebelum memutuskan restore atau force delete.
2. **Tambah fitur "Restore Semua"**, tombol di halaman sampah untuk restore semua produk sampah sekaligus (pakai `Product::onlyTrashed()->restore()`).
3. **Tambah fitur "Hapus Permanen Semua"**, tombol untuk mengosongkan tong sampah (pakai `Product::onlyTrashed()->forceDelete()`).
4. **Tambah badge jumlah** di tombol "Tong Sampah" di halaman index, supaya admin tahu berapa banyak produk di sampah tanpa harus buka halaman sampah.
5. **Eksplor relasi**, pelajari apa yang terjadi pada review atau transaksi yang terkait dengan produk yang di-force-delete.

### Materi Selanjutnya di Level Dasar

Setelah Soft Delete, beberapa topik lanjutan yang relevan di Level Dasar Laravel:

- **10. Relasi Produk-Kategori** (One-to-Many relationship)
- **11. Middleware & Authentication dasar** (memprotek halaman admin)
- **12. Seeder & Factory** (mengisi database dengan data dummy untuk testing)

Tapi sebelum lanjut, pastikan kamu **paham betul** materi soft delete ini. Coba implementasikan sendiri di projek latihanmu, tanpa lihat tutorial. Kalau bisa, baru lanjut ke materi berikutnya.

---

## 🎉 Selamat Menyelesaikan Materi Soft Delete Produk!

> "Soft delete bukan fitur mewah. Ini adalah **standar minimum** untuk sistem yang menghargai data pengguna. Sekali kamu terbiasa pakai soft delete, kamu tidak akan mau kembali ke hapus permanen."

Terima kasih sudah mengikuti materi ini dari tahap 1 sampai 6. Semoga bermanfaat untuk perjalanan belajar Laravel kamu! 🚀
