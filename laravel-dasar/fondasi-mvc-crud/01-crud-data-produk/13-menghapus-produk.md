# Tahap 13 — Menghapus Produk

> **Tujuan tahap ini:** Membuat fitur hapus produk dari database. **Belum membahas soft delete, pagination, search, upload gambar, atau fitur lanjutan.** Hanya fokus pada proses Delete (CRUD lengkap).

---

## 1. Pengantar Sederhana

Pada tahap-tahap sebelumnya, kita sudah bisa:

- **Menambahkan produk** (Tahap 8 + 9).
- **Menampilkan daftar produk** (Tahap 7).
- **Melihat detail produk** (Tahap 10).
- **Mengedit produk** (Tahap 11).
- **Mengupdate produk** (Tahap 12).

Sekarang kita akan membuat **fitur terakhir** dari CRUD, yaitu **menghapus produk**.

### Analogi: Membuang Lembar Arsip

| Hal                    | Analogi                                                  |
| ---------------------- | -------------------------------------------------------- |
| Database               | Lemari arsip                                              |
| Tabel `products`       | Map besar berisi semua data produk                       |
| Satu produk            | Satu lembar arsip                                        |
| Tombol **Hapus**       | Perintah untuk membuang lembar arsip tertentu            |
| Controller (`destroy`) | Petugas yang menerima perintah hapus                     |
| Model `Product`        | Petugas yang mengambil data dari lemari lalu membuangnya |

### Penting!

Pada tahap ini kita akan **benar-benar menghapus data produk** dari database.
Untuk belajar dasar, kita memakai **hapus permanen** dulu.

**Soft delete** (hapus dengan kemampuan memulihkan data) akan dipelajari nanti
pada materi lanjutan.

---

## 2. Tujuan Tahap Ini

Ketika user melihat daftar produk atau detail produk, user bisa menekan tombol:

```text
Hapus
```

Lalu sistem akan:

1. Menampilkan **konfirmasi sederhana**.
2. Mengirim permintaan hapus ke route DELETE `/products/{id}`.
3. Menjalankan method `destroy()` di `ProductController`.
4. Mencari produk berdasarkan ID.
5. Menghapus produk dari tabel `products`.
6. Mengarahkan user kembali ke halaman daftar produk.
7. Menampilkan pesan sukses.

Tahap ini adalah bagian **Delete** dalam CRUD.

---

## 3. Alur Kerja Menghapus Produk

```text
User klik tombol Hapus
        |
        v
Browser menampilkan konfirmasi
        |
        v
User menyetujui penghapusan
        |
        v
Form mengirim permintaan ke /products/{id}
        |
        v
Laravel membaca @method('DELETE')
        |
        v
Route DELETE /products/{id} menerima permintaan
        |
        v
Route memanggil ProductController@destroy
        |
        v
Controller mencari produk berdasarkan ID
        |
        v
Model Product menghapus data dari database
        |
        v
User diarahkan kembali ke daftar produk
        |
        v
Muncul pesan sukses
```

Inilah alur Delete dalam Laravel.

---

## 4. Kenapa Hapus Data Memakai Form?

Menghapus data **tidak disarankan** menggunakan link biasa seperti:

```blade
<a href="/products/1/delete">Hapus</a>
```

### Alasan sederhana

Link biasa digunakan untuk **melihat atau membuka halaman**.
Sedangkan menghapus data adalah **aksi penting** yang mengubah database.

Jika hapus dibuat sebagai link, bisa terjadi hal berbahaya:
misalnya browser prefetch (membuka link secara otomatis di belakang layar),
crawler mesin pencari, atau pengguna yang tidak sengaja klik.

Karena itu, kita memakai **form** dengan **method khusus**.

### Form hapus di Laravel

```blade
<form action="/products/{{ $product->id }}" method="POST">
    @csrf
    @method('DELETE')
    <button type="submit">Hapus</button>
</form>
```

### Penjelasan

- `method="POST"` dipakai karena HTML form tidak mendukung DELETE secara langsung.
- `@method('DELETE')` memberi tahu Laravel bahwa permintaan ini sebenarnya adalah permintaan **hapus**.
- `@csrf` melindungi form dari penyalahgunaan pihak luar.

### Analogi

Hapus data itu seperti menandatangani surat perintah pembuangan - perlu form
resmi, bukan sekadar mengunjungi sebuah alamat.

---

## 5. Membuat Route DELETE `/products/{id}`

Buka file:

```text
routes/web.php
```

Pastikan ada route berikut:

```php
Route::delete('/products/{id}', [ProductController::class, 'destroy']);
```

### Susunan route sementara yang disarankan

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ProductController;

Route::get('/', function () {
    return view('welcome');
});

Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/create', [ProductController::class, 'create']);
Route::post('/products', [ProductController::class, 'store']);
Route::get('/products/{id}/edit', [ProductController::class, 'edit']);
Route::put('/products/{id}', [ProductController::class, 'update']);
Route::delete('/products/{id}', [ProductController::class, 'destroy']);
Route::get('/products/{id}', [ProductController::class, 'show']);
```

### Penjelasan route

Route `Route::delete('/products/{id}', [ProductController::class, 'destroy'])` berarti:

> Jika ada permintaan DELETE ke alamat `/products/{id}`,
> Laravel akan menjalankan method `destroy()` di `ProductController`.

### Tabel route lengkap CRUD

| Route                | Method | Fungsi                       |
| -------------------- | ------- | ---------------------------- |
| `/products`          | GET     | Menampilkan daftar produk    |
| `/products/create`   | GET     | Menampilkan form tambah      |
| `/products`          | POST    | Menyimpan produk baru        |
| `/products/{id}`     | GET     | Menampilkan detail produk    |
| `/products/{id}/edit`| GET     | Menampilkan form edit        |
| `/products/{id}`     | PUT     | Mengupdate produk            |
| `/products/{id}`     | DELETE  | **Menghapus produk**         |

URL `/products/{id}` bisa dipakai untuk beberapa aksi karena HTTP method-nya berbeda:

- GET `/products/1` = melihat detail produk ID 1.
- PUT `/products/1` = mengupdate produk ID 1.
- DELETE `/products/1` = menghapus produk ID 1.

---

## 6. Membuat Method `destroy()` di ProductController

Buka file:

```text
app/Http/Controllers/ProductController.php
```

Tambahkan method `destroy()` di dalam class `ProductController`:

```php
public function destroy($id)
{
    $product = Product::findOrFail($id);

    $product->delete();

    return redirect('/products')->with('success', 'Produk berhasil dihapus.');
}
```

Method `destroy()` adalah tempat menerima permintaan hapus dan menghapus data
produk dari database.

---

## 7. Contoh ProductController Lengkap Sementara

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    public function index()
    {
        $products = Product::all();

        return view('products.index', compact('products'));
    }

    public function create()
    {
        return view('products.create');
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'name'        => 'required',
            'description' => 'nullable',
            'price'       => 'required|integer|min:0',
            'stock'       => 'required|integer|min:0',
        ]);

        Product::create($validated);

        return redirect('/products')->with('success', 'Produk berhasil ditambahkan.');
    }

    public function show($id)
    {
        $product = Product::findOrFail($id);

        return view('products.show', compact('product'));
    }

    public function edit($id)
    {
        $product = Product::findOrFail($id);

        return view('products.edit', compact('product'));
    }

    public function update(Request $request, $id)
    {
        $product = Product::findOrFail($id);

        $validated = $request->validate([
            'name'        => 'required',
            'description' => 'nullable',
            'price'       => 'required|integer|min:0',
            'stock'       => 'required|integer|min:0',
        ]);

        $product->update($validated);

        return redirect('/products/' . $product->id)->with('success', 'Produk berhasil diperbarui.');
    }

    public function destroy($id)
    {
        $product = Product::findOrFail($id);

        $product->delete();

        return redirect('/products')->with('success', 'Produk berhasil dihapus.');
    }
}
```

### Penjelasan singkat setiap method

| Method       | Fungsi                                |
| ------------ | ------------------------------------- |
| `index()`    | Menampilkan daftar produk              |
| `create()`   | Menampilkan form tambah produk         |
| `store()`    | Menyimpan produk baru                  |
| `show()`     | Menampilkan detail satu produk         |
| `edit()`     | Menampilkan form edit produk           |
| `update()`   | Menyimpan perubahan produk             |
| `destroy()`  | **Menghapus produk**                   |

Sekarang **ketujuh method CRUD standar** sudah lengkap.

---

## 8. Menjelaskan `Product::findOrFail($id)`

```php
$product = Product::findOrFail($id);
```

Artinya: Laravel mencari produk berdasarkan ID dari URL.

- Jika produk **ditemukan**, produk siap dihapus.
- Jika produk **tidak ditemukan**, Laravel menampilkan halaman **404 Not Found**.

### Analogi

Petugas arsip mencari lembar produk berdasarkan nomor ID. Kalau lembar ditemukan,
lembar itu bisa dibuang. Kalau tidak ditemukan, petugas memberi tahu bahwa data
tidak ada.

---

## 9. Menjelaskan `$product->delete()`

```php
$product->delete();
```

Artinya: produk yang sudah ditemukan akan **dihapus dari tabel `products`**.

### Analogi

Setelah petugas menemukan lembar arsip produk, lembar itu dikeluarkan dari map
dan dibuang.

### Catatan tentang soft delete

Pada tahap dasar ini, data akan **benar-benar hilang** dari tabel.

Nanti pada materi lanjutan, kita bisa belajar **soft delete**: data seolah-olah
dihapus, tetapi sebenarnya masih tersimpan dan bisa dipulihkan. Untuk sekarang,
kita pakai hapus permanen agar konsepnya sederhana.

---

## 10. Menjelaskan Redirect Setelah Hapus

```php
return redirect('/products')->with('success', 'Produk berhasil dihapus.');
```

Artinya: setelah produk berhasil dihapus, user diarahkan kembali ke **halaman daftar produk**.

### Bagian `->with('success', '...')`

Laravel membawa **pesan sukses sementara** ke halaman daftar produk. Pesan ini
disimpan di session dan otomatis hilang setelah ditampilkan sekali.

### Analogi

Setelah petugas menghapus arsip produk, petugas memberi catatan kecil:

> "Produk berhasil dihapus."

---

## 11. Memastikan Pesan Sukses Tampil di Halaman Daftar Produk

Buka file:

```text
resources/views/products/index.blade.php
```

Pastikan ada kode berikut di dekat bagian atas halaman:

```blade
@if (session('success'))
    <div style="padding: 10px; background: #d1e7dd; border: 1px solid #badbcc; color: #0f5132; margin-bottom: 15px;">
        {{ session('success') }}
    </div>
@endif
```

### Contoh posisi

```blade
<h1>Daftar Produk</h1>

@if (session('success'))
    <div style="padding: 10px; background: #d1e7dd; border: 1px solid #badbcc; color: #0f5132; margin-bottom: 15px;">
        {{ session('success') }}
    </div>
@endif

<a href="/products/create">Tambah Produk</a>
```

### Penjelasan

Kode ini sudah pernah dipakai untuk pesan sukses **tambah produk** dan **update produk**.
Sekarang juga dipakai untuk pesan sukses **hapus produk**. Satu blok yang sama
bisa menampilkan berbagai pesan sukses karena semua dikirim dengan key yang sama,
yaitu `success`.

---

## 12. Menambahkan Tombol Hapus di Halaman Daftar Produk

Buka file:

```text
resources/views/products/index.blade.php
```

Pada kolom aksi, jika sebelumnya seperti ini:

```blade
<td>
    <a href="/products/{{ $product->id }}">Detail</a>
    |
    <a href="/products/{{ $product->id }}/edit">Edit</a>
</td>
```

Ubah menjadi:

```blade
<td>
    <a href="/products/{{ $product->id }}">Detail</a>
    |
    <a href="/products/{{ $product->id }}/edit">Edit</a>
    |
    <form action="/products/{{ $product->id }}" method="POST" style="display: inline;" onsubmit="return confirm('Yakin ingin menghapus produk ini?')">
        @csrf
        @method('DELETE')
        <button type="submit">Hapus</button>
    </form>
</td>
```

### Penjelasan bagian-bagiannya

#### `action="/products/{{ $product->id }}"`

Form akan mengirim permintaan hapus ke produk sesuai ID.

Contoh: jika produk ID 3, maka action menjadi `/products/3`.

#### `method="POST"`

HTML form hanya mendukung GET dan POST. Karena itu form tetap memakai POST.

#### `@method('DELETE')`

Laravel membaca ini sebagai permintaan **DELETE**.

#### `@csrf`

Pelindung keamanan form Laravel.

#### `onsubmit="return confirm('...')"`

Menampilkan **konfirmasi** sebelum data benar-benar dihapus.

- Jika user memilih **OK**, proses hapus dilanjutkan.
- Jika user memilih **Cancel**, proses hapus dibatalkan.

#### `style="display: inline;"`

Supaya form tampil sejajar dengan link lain di kolom Aksi, tidak pindah ke baris baru.

---

## 13. Menambahkan Tombol Hapus di Halaman Detail Produk

Buka file:

```text
resources/views/products/show.blade.php
```

Tambahkan form hapus di bawah link edit atau sebelum link kembali:

```blade
<form action="/products/{{ $product->id }}" method="POST" onsubmit="return confirm('Yakin ingin menghapus produk ini?')">
    @csrf
    @method('DELETE')
    <button type="submit">Hapus Produk</button>
</form>
```

### Contoh susunan

```blade
<a href="/products/{{ $product->id }}/edit">Edit Produk</a>

<form action="/products/{{ $product->id }}" method="POST" onsubmit="return confirm('Yakin ingin menghapus produk ini?')">
    @csrf
    @method('DELETE')
    <button type="submit">Hapus Produk</button>
</form>

<a href="/products">Kembali ke Daftar Produk</a>
```

### Penjelasan

Ini membuat user bisa **menghapus produk dari halaman detail** produk, selain
dari halaman daftar. Fleksibilitas ini membantu user.

---

## 14. Menambahkan Sedikit CSS untuk Tombol Hapus

Jika ingin tombol hapus terlihat berbeda (warna merah), tambahkan CSS sederhana.

Contoh di bagian `<style>` pada `index.blade.php` atau `show.blade.php`:

```css
button.danger {
    padding: 6px 10px;
    background: #dc3545;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

button.danger:hover {
    background: #bb2d3b;
}
```

Lalu pakai class tersebut:

```blade
<button type="submit" class="danger">Hapus</button>
```

### Penjelasan

Warna **merah** biasanya digunakan untuk aksi berbahaya seperti hapus data.

### Catatan

CSS ini hanya untuk memperjelas tampilan, **bukan bagian utama** dari logika Laravel.
Kamu bisa abaikan jika ingin fokus ke logika dulu.

---

## 15. Mengecek Proses Hapus Produk

Lakukan langkah berikut:

1. Jalankan server Laravel:

```bash
php artisan serve
```

2. Buka halaman daftar produk:

```text
http://127.0.0.1:8000/products
```

3. Pastikan ada data produk.
   Jika belum ada produk, tambahkan produk dulu melalui `/products/create`.

4. Klik tombol **Hapus** pada salah satu produk.

5. Saat muncul konfirmasi, pilih **OK**.

6. Pastikan browser kembali ke halaman daftar produk.

7. Pastikan muncul pesan:

```text
Produk berhasil dihapus.
```

8. Pastikan produk yang dihapus sudah **tidak muncul** di daftar produk.

---

## 16. Mengecek Data di Database

Cara mengecek langsung:

1. Buka phpMyAdmin / Adminer / TablePlus / database client.
2. Pilih database yang digunakan.
3. Buka tabel `products`.
4. Pastikan produk yang dihapus **sudah tidak ada**.

Jika produk hilang dari halaman daftar **dan juga hilang dari tabel database**,
berarti fitur hapus berhasil.

---

## 17. Menguji Tombol Cancel pada Konfirmasi

Coba skenario ini:

1. Klik tombol **Hapus**.
2. Saat muncul konfirmasi, pilih **Cancel**.
3. Pastikan produk **tidak terhapus**.

### Penjelasan

Ini berarti konfirmasi sederhana bekerja.

Konfirmasi ini seperti pertanyaan terakhir sebelum membuang arsip:

> "Apakah kamu benar-benar yakin?"

---

## 18. Kenapa Perlu Konfirmasi Sebelum Hapus?

Hapus data adalah **aksi berisiko**. Kalau user tidak sengaja klik tombol hapus,
data bisa hilang permanen.

Konfirmasi membantu **mencegah kesalahan tidak sengaja**.

| Tanpa Konfirmasi                  | Dengan Konfirmasi                  |
| --------------------------------- | ---------------------------------- |
| Data langsung terhapus            | User ditanya dulu                  |
| Risiko salah klik tinggi          | Risiko salah klik lebih kecil      |
| Kurang aman untuk UX              | Lebih aman untuk pengguna          |

### Catatan

Konfirmasi sederhana ini masih dasar (pakai `confirm()` bawaan browser).
Di aplikasi nyata, bisa dibuat **modal** yang lebih rapi (misalnya dengan Bootstrap
atau Tailwind UI).

---

## 19. Masalah Umum dan Solusinya

### Error: `The DELETE method is not supported for route products/{id}`

**Penyebab:** Route DELETE `/products/{id}` belum dibuat.

**Solusi:** Pastikan di `routes/web.php` ada:

```php
Route::delete('/products/{id}', [ProductController::class, 'destroy']);
```

### Error: `Method App\Http\Controllers\ProductController::destroy does not exist`

**Penyebab:** Method `destroy()` belum dibuat di controller.

**Solusi:** Tambahkan method `destroy()` di `ProductController`.

### Error: `Class "Product" not found`

**Penyebab:** Model `Product` belum di-import di controller.

**Solusi:** Pastikan di bagian atas controller ada:

```php
use App\Models\Product;
```

### Tombol hapus tidak melakukan apa-apa

**Kemungkinan penyebab:**

- Form belum memiliki `@csrf`.
- Form belum memiliki `@method('DELETE')`.
- Tombol tidak bertipe `submit`.
- Route DELETE belum dibuat.

**Solusi:** Pastikan form seperti ini:

```blade
<form action="/products/{{ $product->id }}" method="POST" onsubmit="return confirm('Yakin ingin menghapus produk ini?')">
    @csrf
    @method('DELETE')
    <button type="submit">Hapus</button>
</form>
```

### Setelah hapus muncul halaman 404

**Kemungkinan penyebab:** Setelah hapus, sistem masih mencoba membuka detail produk
yang sudah dihapus.

**Solusi:** Pastikan setelah hapus controller mengarahkan ke **daftar produk**,
bukan ke detail:

```php
return redirect('/products')->with('success', 'Produk berhasil dihapus.');
```

### Produk tidak hilang setelah klik hapus

**Kemungkinan penyebab:**

- Method `destroy()` belum memanggil `$product->delete()`.
- Form mengarah ke ID yang salah.
- Route DELETE salah.

**Solusi:** Cek kembali:

- `routes/web.php`
- `ProductController`
- Form hapus di Blade

---

## 20. Struktur File Sampai Tahap Ini

```text
database/
  migrations/
    xxxx_xx_xx_xxxxxx_create_products_table.php

app/
  Models/
    Product.php
  Http/
    Controllers/
      ProductController.php

routes/
  web.php

resources/
  views/
    products/
      index.blade.php
      create.blade.php
      show.blade.php
      edit.blade.php
```

### Fungsi masing-masing file pada proses hapus

| File               | Fungsi dalam proses hapus                           |
| ------------------ | --------------------------------------------------- |
| `index.blade.php`  | Menampilkan tombol hapus di daftar produk           |
| `show.blade.php`   | Menampilkan tombol hapus di detail produk           |
| `routes/web.php`   | Route DELETE menerima permintaan hapus              |
| `ProductController`| Method `destroy()` memproses penghapusan            |
| `Product.php`      | Model menghapus data dari tabel `products`           |

---

## 21. Apa yang Sudah Berhasil Dibuat?

| Bagian CRUD         | Status       |
| ------------------- | ------------ |
| Create              | Sudah dibuat |
| Read daftar produk  | Sudah dibuat |
| Read detail produk  | Sudah dibuat |
| Form Update/Edit    | Sudah dibuat |
| Proses Update       | Sudah dibuat |
| Delete              | **Sudah dibuat** |

### Fitur CRUD dasar produk sudah lengkap!

User sudah bisa:

- Menambahkan produk.
- Melihat daftar produk.
- Melihat detail produk.
- Mengedit produk.
- Mengupdate produk.
- **Menghapus produk.**

---

## 22. Apa yang Belum Dilakukan?

Sampai tahap ini kita **belum** membuat:

- Soft delete (simulasi hapus tanpa benar-benar menghapus).
- Pagination (pembagian halaman).
- Search produk.
- Upload gambar produk.
- Layout Blade reusable.
- Validasi dengan Form Request.
- Flash message component.
- Route resource (penulisan route ringkas).
- Authorization (hak akses).
- Testing.
- Service layer.
- Arsitektur modular.

Semua itu adalah **materi lanjutan**. Tahap ini hanya fokus menyelesaikan
**CRUD dasar produk**.

---

## 23. Checklist Keberhasilan

Centang (`x`) jika sudah selesai:

- [ ] Saya tahu alur menghapus produk.
- [ ] Saya tahu kenapa hapus data memakai form, bukan link biasa.
- [ ] Saya tahu fungsi route DELETE `/products/{id}`.
- [ ] Saya tahu kenapa form hapus memakai `method="POST"` dan `@method('DELETE')`.
- [ ] Saya sudah membuat route DELETE `/products/{id}`.
- [ ] Saya sudah membuat method `destroy()` di `ProductController`.
- [ ] Saya tahu fungsi `Product::findOrFail($id)`.
- [ ] Saya tahu fungsi `$product->delete()`.
- [ ] Saya sudah menambahkan tombol hapus di halaman daftar produk.
- [ ] Saya sudah menambahkan tombol hapus di halaman detail produk.
- [ ] Saya sudah menambahkan konfirmasi sebelum hapus.
- [ ] Saya bisa menghapus produk dari database.
- [ ] Saya bisa melihat pesan sukses setelah hapus.
- [ ] Saya paham bahwa **CRUD dasar produk sudah lengkap**.

Jika semua sudah tercentang, selamat! CRUD lengkap sudah berfungsi.

---

## 24. Kesimpulan Tahap 13

Pada tahap ini kita sudah berhasil membuat **fitur hapus produk**.

### Ringkasan perjalanan belajar (dengan analogi)

| Tahap   | Yang kita buat                                      |
| ------- | --------------------------------------------------- |
| Tahap 3  | Membuat **lemari arsip** produk (migration + tabel)  |
| Tahap 4  | Membuat **petugas arsip** bernama Model Product      |
| Tahap 5  | Membuat **alamat halaman** produk (route)            |
| Tahap 6  | Membuat **manajer** bernama ProductController        |
| Tahap 7  | Membuat **etalase** daftar produk (Read daftar)      |
| Tahap 8  | Membuat **formulir tambah** produk (Form Create)     |
| Tahap 9  | Membuat **proses menyimpan** produk baru (Create)    |
| Tahap 10 | Membuat **kartu detail** produk (Read detail)        |
| Tahap 11 | Membuat **formulir edit** berisi data lama (Form Edit)|
| Tahap 12 | Membuat **proses update** produk (Update)            |
| **Tahap 13** | **Membuat proses menghapus produk dari database** (Delete) |

### CRUD lengkap!

Dengan tahap ini, **CRUD dasar produk sudah selesai**.

- **C**reate (Tahap 8 + 9)
- **R**ead (Tahap 7 + 10)
- **U**pdate (Tahap 11 + 12)
- **D**elete (**Tahap 13**)

### Jika sudah paham dan produk berhasil dihapus, ketik:

```text
lanjut
```

Jangan lanjut ke tahap 14 sebelum kamu meminta. Pelan-pelan, asal paham.
