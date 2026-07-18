# Tahap 3 — Soft Delete di Controller: Ubah `destroy()` dan Lihat Efeknya

> Materi: Soft Delete Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi: Tombol "Buang" di Dapur yang Sekarang Mengarah ke Tempat Sampah

Di tahap 2, kita sudah **menyiapkan tempat sampah** (kolom `deleted_at` + trait `SoftDeletes`). Sekarang pertanyaannya:

> Bagaimana caranya supaya saat admin klik tombol **"Hapus"** di halaman web, produknya masuk ke tempat sampah, bukan dibuang permanen?

Bayangkan di dapur kamu punya **tombol bertuliskan "BUANG"**. Dulu, tombol itu langsung membuang barang ke luar rumah (permanen). Sekarang, karena kamu sudah punya tempat sampah, kamu tinggal **arahkan ulang selang** tombol itu: yang dibuang masuk dulu ke tempat sampah.

Yang kamu ubah cuma **tujuan tombolnya**. Tombolnya tetap. Labelnya tetap. Alur admin klik tetap sama. Yang beda hanya **apa yang terjadi di belakang layar**.

Di Laravel, ini berarti: kita ubah **isi method `destroy()` di controller**. Nama method tetap, route tetap, tombol di view tetap. Cuma isinya yang kita sesuaikan supaya jadi soft delete.

---

## 2. Recap: Method `destroy()` yang Lama (Hapus Permanen)

Sebelum kita ubah, mari lihat dulu bentuk method `destroy()` yang biasanya ada di `app/Http/Controllers/ProductController.php`:

```php
public function destroy($id)
{
    $product = Product::findOrFail($id);
    $product->delete();

    return redirect('/produk')->with('success', 'Produk berhasil dihapus.');
}
```

Penjelasan per baris:

| Baris | Fungsi |
|---|---|
| `$product = Product::findOrFail($id);` | Cari produk berdasarkan ID. Kalau tidak ketemu, otomatis error 404. |
| `$product->delete();` | Hapus produk. **SEBELUM pakai SoftDeletes**, ini akan menjalankan `DELETE FROM products WHERE id = ?`. Produk hilang permanen. |
| `return redirect('/produk')...` | Kembali ke halaman daftar produk, bawa pesan sukses. |

Ini sudah berfungsi. Tapi "berfungsi" dalam arti hapus permanen.

---

## 3. Apa yang Terjadi Sekarang dengan Trait `SoftDeletes`?

Inilah **bagian ajaibnya**. Karena di tahap 2 kita sudah pakai `use SoftDeletes;` di model `Product`, **method `destroy()` yang lama itu SUDAH otomatis melakukan soft delete tanpa kita ubah apa-apa**.

Coba baca pelan-pelan:

- `$product->delete();` tetap dipanggil seperti biasa.
- Tapi sekarang, **karena model pakai `SoftDeletes`**, method `delete()` itu **berperilaku berbeda**.
- Laravel otomatis **mengganti** perintahnya dari `DELETE FROM` jadi `UPDATE ... SET deleted_at = ...`.

Jadi sebenarnya, **kode controller TIDAK HARUS diubah** untuk melakukan soft delete. Trait sudah melakukan semua keajaiban.

Tapi, untuk pembelajaran, kita akan:

1. **Lihat dulu** efek nyatanya lewat eksperimen kecil.
2. **Lalu ubah sedikit** controller supaya lebih jelas dan ramah pemakai (user-friendly).

---

## 4. Eksperimen: Lihat Efek Soft Delete di Tinker

Sebelum kita sentuh controller, mari kita **buktikan** dengan eksperimen. Buka terminal di folder projek kamu, jalankan:

```bash
php artisan tinker
```

Di dalam tinker, ketik perintah berikut satu per satu:

### Langkah A: Lihat Produk Sebelum Dihapus

```php
$products = Product::all();
$products->count();
```

Misal hasilnya: **3** (ada 3 produk aktif).

### Langkah B: Soft-Delete Satu Produk

```php
$product = Product::first();
$product->delete();
```

Tinker akan bilang: `true` atau `null` (tergantung versi). **Tidak ada error.**

### Langkah C: Lihat Produk Setelah Dihapus

```php
Product::all()->count();
```

Hasilnya sekarang: **2**. Satu produk **menghilang** dari query biasa.

### Langkah D: Coba Cari Produk yang Baru Dihapus

```php
Product::find($product->id);
```

Hasilnya: **`null`**. Laravel bilang "tidak ketemu", padahal produknya masih ada di database.

### Langkah E: Tampilkan Termasuk yang Dihapus

```php
Product::withTrashed()->count();
```

Hasilnya: **3**. Semua produk muncul lagi, termasuk yang sudah di-soft-delete.

### Langkah F: Tampilkan HANYA yang Dihapus

```php
Product::onlyTrashed()->count();
```

Hasilnya: **1**. Hanya produk yang ada di "tempat sampah".

Untuk keluar dari tinker, ketik:

```
exit
```

**Kesimpulan eksperimen**: trait `SoftDeletes` sudah bekerja sempurna. Method `$product->delete()` otomatis melakukan soft delete tanpa kita ubah kode controller.

---

## 5. Modifikasi Controller: Tambahkan Pesan yang Lebih Jelas

Walaupun kode lama sudah berfungsi, ada satu hal kecil yang akan kita tambahkan: **pesan sukses yang lebih informatif**. Karena sekarang produk tidak benar-benar hilang, pesan "Produk berhasil dihapus" agak menyesatkan. Mungkin lebih tepat: "Produk berhasil dipindahkan ke sampah."

Buka `app/Http/Controllers/ProductController.php`, lalu ubah method `destroy()`:

```php
public function destroy($id)
{
    $product = Product::findOrFail($id);
    $product->delete();

    return redirect('/produk')
        ->with('success', 'Produk berhasil dipindahkan ke sampah.');
}
```

Perubahan kecilnya:

| Sebelum | Sesudah |
|---|---|
| `'Produk berhasil dihapus.'` | `'Produk berhasil dipindahkan ke sampah.'` |

**Kenapa diubah?** Karena sekarang produk masih bisa dikembalikan. Pesan baru ini **memberitahu admin** bahwa produk belum hilang selamanya, dan dia masih bisa mengembalikannya nanti. Pesan yang akurat bikin admin tenang.

---

## 6. Apa yang Terjadi Saat Admin Klik Tombol Hapus Sekarang?

Mari kita ikuti alur lengkapnya, dari klik sampai database.

### Di Halaman View

Admin buka halaman `/produk`. Ada tabel produk, masing-masing ada tombol **Hapus**. Tombol itu biasanya berupa form ke route `DELETE /produk/{id}`.

Kode di view (Blade) kira-kira seperti ini (di tahap 8 sorting kamu sudah pernah lihat yang serupa):

```blade
<form action="/produk/{{ $product->id }}" method="POST" style="display:inline;">
    @csrf
    @method('DELETE')
    <button type="submit" onclick="return confirm('Yakin pindahkan produk ke sampah?')">
        Hapus
    </button>
</form>
```

Penjelasan singkat:

| Baris | Fungsi |
|---|---|
| `action="/produk/{{ $product->id }}"` | Form akan dikirim ke URL `/produk/5` (misalnya) |
| `method="POST"` | Form pakai POST (standar) |
| `@csrf` | Token keamanan Laravel supaya form tidak bisa dipalsukan pihak luar |
| `@method('DELETE')` | Bilang ke Laravel: "Ini sebenarnya request DELETE" (karena HTML hanya dukung GET/POST) |
| `onclick="return confirm(...)"` | Tampilkan dialog konfirmasi sebelum kirim. Kalau user klik Cancel, form tidak jadi dikirim |
| `Hapus` | Label tombol |

### Di Route

Request `DELETE /produk/5` masuk ke route. Di `routes/web.php` biasanya seperti ini:

```php
Route::delete('/produk/{id}', [ProductController::class, 'destroy'])->name('produk.destroy');
```

Laravel akan mengarahkan request ini ke method `destroy($id)` di `ProductController`, dengan `$id = 5`.

### Di Controller

Method `destroy(5)` berjalan:

1. `Product::findOrFail(5)` → cari produk id=5. Kalau tidak ada, error 404.
2. `$product->delete()` → **karena model pakai SoftDeletes**, yang terjadi adalah `UPDATE products SET deleted_at = NOW() WHERE id = 5`. Bukan `DELETE FROM`.
3. `redirect('/produk')` → kembali ke halaman daftar produk.
4. `->with('success', '...')` → bawa pesan sukses yang akan ditampilkan sekali (flash session).

### Di Database

Isi tabel `products` untuk id=5 sekarang:

| id | nama | harga | deleted_at |
|---|---|---|---|
| 5 | Kopi Susu Vanilla | 18000 | **2026-07-18 11:02:15** |

Produknya **masih ada**. Cuma `deleted_at`-nya terisi timestamp saat dihapus.

### Di Halaman Berikutnya

Saat halaman `/produk` dimuat ulang, query di `index()` method controller (`Product::all()` atau `Product::paginate(10)`) **otomatis exclude** produk id=5, karena `deleted_at`-nya tidak NULL. Jadi produk itu **tidak muncul** di daftar.

Tapi produknya **tidak hilang dari database**. Masih bisa di-restore. Itulah soft delete.

---

## 7. Hal-hal Penting yang Sering Bikin Pemula Bingung

### a. `findOrFail` vs `find` di Controller Destroy

Pakai `findOrFail($id)`, bukan `find($id)`.

```php
$product = Product::findOrFail($id);  // BENAR: otomatis 404 kalau tidak ketemu
```

Kenapa? Karena kalau user klik tombol hapus untuk produk yang **sudah di-soft-delete** sebelumnya (misal lewat klik cepat), `find($id)` akan return `null`. Lalu `$product->delete()` akan error karena kita panggil method di `null`.

`findOrFail` lebih aman: kalau tidak ketemu, Laravel otomatis tampilkan halaman 404, bukan error panjang.

**Bonus penting**: karena model pakai `SoftDeletes`, `findOrFail(5)` untuk produk yang sudah di-soft-delete **juga akan 404**. Karena query biasa tidak menemukan produk id=5 (dianggap tidak ada). Ini perilaku yang **kita inginkan**. Admin tidak boleh menghapus produk yang sudah ada di sampah.

### b. Method HTTP Harus `DELETE`

Walaupun kita "menghapus" data, Laravel mau request-nya pakai method HTTP `DELETE` (bukan POST biasa). Itu sebabnya di view kita pakai `@method('DELETE')`.

Kalau kamu lupa pakai `@method('DELETE')`, request akan diterima sebagai POST, dan route `Route::delete(...)` tidak akan ketemu. Hasilnya: error **405 Method Not Allowed**.

### c. Tombol Hapus Wajib Form, Bukan Link

Jangan pakai `<a href="/produk/5">Hapus</a>` untuk hapus. Karena itu akan jadi request **GET**, dan route hapus kita `DELETE`. Selain itu, link GET bisa ke-trigger oleh crawler search engine atau preview browser, yang berarti **produk bisa kehapus tanpa sengaja**.

Selalu pakai form dengan `@method('DELETE')` dan tombol submit. Plus `onclick="confirm(...)"` supaya admin tidak salah klik.

### d. Produk yang Sudah Di-Soft-Delete Tidak Bisa Diakses dari Query Biasa

Ini penting untuk diingat:

- Setelah produk id=5 di-soft-delete, `Product::find(5)` → `null`.
- `Product::where('slug', 'kopi-susu-vanilla')->first()` → `null` juga.
- Halaman detail produk `/produk/kopi-susu-vanilla` → **404**.

Karena dari sudut pandang aplikasi, produk itu "sudah tidak ada". Walaupun datanya masih ada di database. Untuk mengaksesnya lagi, harus pakai `withTrashed()` atau `onlyTrashed()`.

### e. Hapus Bukan Berarti Kehilangan Untuk Selamanya

Yang paling penting buat dipahami pemula: setelah soft delete, **produk BUKAN hilang selamanya**. Itu cuma dipindahkan ke "tempat sampah". Di tahap 5 kita akan belajar cara **restore** (mengembalikan). Dan di tahap 6 cara **force delete** (membuang permanen).

Sekarang, kamu sudah aman melakukan soft delete tanpa rasa takut.

---

## 8. Apa yang Sudah Bisa Kamu Lakukan Setelah Tahap 3?

Setelah tahap 3, alur hapus produk di aplikasi kamu sudah **benar-benar melakukan soft delete**:

1. Admin buka halaman `/produk`.
2. Admin klik tombol **Hapus** di salah satu produk.
3. Browser tampilkan dialog konfirmasi.
4. Admin klik OK.
5. Request `DELETE /produk/{id}` dikirim ke server.
6. Controller `destroy()` berjalan, `$product->delete()` dijalankan.
7. Laravel otomatis melakukan soft delete (isi `deleted_at`).
8. Admin di-redirect balik ke `/produk` dengan pesan sukses.
9. Produk yang dihapus **menghilang dari daftar**, tapi **datanya masih aman di database**.

**Yang belum bisa kamu lakukan sampai tahap ini:**

- Admin belum **bisa lihat** produk yang sudah di-soft-delete (belum ada halaman "tong sampah").
- Admin belum **bisa restore** (mengembalikan) produk yang sudah dihapus.
- Admin belum **bisa force delete** (menghapus permanen) produk yang ada di sampah.

Tiga hal ini akan kita kerjakan di tahap 4, 5, dan 6.

---

## Ringkasan Tahap 3

| Hal | Isi |
|---|---|
| Inti tahap | Memahami efek soft delete di controller |
| Kabar baik | Method `destroy()` TIDAK perlu diubah untuk soft delete — trait yang melakukan pekerjaan |
| Perubahan kecil | Update pesan sukses jadi "Produk berhasil dipindahkan ke sampah." |
| Eksperimen tinker | Buktikan produk menghilang dari query biasa tapi muncul di `withTrashed()` / `onlyTrashed()` |
| Alur klik hapus | View form `@method('DELETE')` → route `Route::delete(...)` → controller `destroy()` → `UPDATE ... SET deleted_at` |
| Yang sudah bisa | Admin klik hapus → produk menghilang dari daftar, data aman di database |
| Yang belum bisa | Lihat sampah, restore, force delete |

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: membuat halaman "Tong Sampah Produk" yang menampilkan produk-produk yang sudah di-soft-delete?**

Kalau iya, tahap 4 kita akan:
1. Tambah route baru `/produk/sampah` (atau `/produk/trash`).
2. Tambah method `trash()` di `ProductController` yang pakai `Product::onlyTrashed()->get()`.
3. Buat view `trash.blade.php` untuk menampilkan daftar produk yang sudah dihapus.
4. Tambah link "Lihat Sampah" di halaman daftar produk supaya admin gampang akses.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
