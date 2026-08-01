# Tahap 11: Filter Produk Berdasarkan Kategori

## Tujuan Tahap Ini

Sekarang list produk sudah tampil dengan nama kategori. Tapi belum bisa **difilter**: user belum bisa klik "Pakaian" dan hanya melihat produk pakaian saja.

Tujuan kita di tahap terakhir: tambah **filter berdasarkan kategori**, supaya user bisa:

- Lihat semua produk (default).
- Lihat hanya produk dari satu kategori tertentu.

## Analogi Sederhana: Filter di Toko Online

Bayangkan kamu buka aplikasi toko online. Di halaman utama, semua produk tampil bercampur. Lalu ada **menu kategori** di samping:

- Semua
- Elektronik
- Pakaian
- Makanan
- Buku

Saat kamu klik **"Pakaian"**, hanya produk pakaian yang tampil. Saat klik **"Semua"**, semua produk kembali muncul.

Ini yang akan kita buat. Idenya sederhana: kirim parameter ke URL, controller terima dan filter data.

## Cara Kerja Filter di Web

Filter produk berdasarkan kategori memakai pola URL:

```
/products               -> tampilkan SEMUA produk
/products?category=2    -> tampilkan produk kategori id=2 (Pakaian)
```

Bagian `?category=2` disebut **query string**:

| Bagian      | Arti                                                                  |
|-------------|-----------------------------------------------------------------------|
| `?`         | Tanda mulai query string                                              |
| `category`  | Nama parameter (bisa apa saja, kita pilih singkat)                    |
| `=`         | Pemisah antara nama parameter dan nilai                               |
| `2`         | Nilai parameter (id kategori yang difilter)                           |

Di Laravel, parameter ini bisa diakses lewat `$request->query('category')` atau `$request->category`.

## Langkah 1: Ubah Method `index()` di ProductController

Buka `app/Http/Controllers/ProductController.php`. Ubah method `index()`:

```php
public function index(Request $request)
{
    $categories = Category::orderBy('name')->get();

    $query = Product::with('category')->latest();

    if ($request->filled('category')) {
        $query->where('category_id', $request->category);
    }

    $products = $query->get();

    return view('products.index', compact('products', 'categories'));
}
```

### Penjelasan Bagian Per Bagian

#### Tambah Parameter `Request $request`

```php
public function index(Request $request)
```

Method sekarang menerima request HTTP, supaya bisa baca query string.

#### Query Builder Bertahap

```php
$query = Product::with('category')->latest();
```

Perhatikan: kita **tidak langsung** panggil `->get()`. Kita simpan dulu sebagai query builder. Kenapa? Supaya bisa **tambah kondisi `where`** kalau perlu.

Ini disebut pola **deferred query**: bangun query dulu, baru eksekusi di akhir.

#### Cek Parameter Terisi

```php
if ($request->filled('category')) {
    $query->where('category_id', $request->category);
}
```

Penjelasan:

| Bagian                          | Arti                                                              |
|---------------------------------|-------------------------------------------------------------------|
| `$request->filled('category')`  | Cek apakah parameter `category` ada **dan tidak kosong**          |
| `$query->where('category_id', $request->category)` | Tambah filter: hanya produk dengan category_id tersebut |

Bedanya `filled` vs `has`:

- `has('category')` -> true kalau parameter ada, walau nilai kosong (`?category=`).
- `filled('category')` -> true kalau parameter ada **dan** nilainya tidak kosong.

Pakai `filled` lebih aman untuk filter.

#### Eksekusi Query

```php
$products = $query->get();
```

Baru di sini query benar-benar dijalankan ke database. Hasilnya: collection produk (semua atau terfilter).

#### Kirim `$categories` ke View

```php
return view('products.index', compact('products', 'categories'));
```

Sekarang view juga menerima daftar kategori, supaya bisa bikin tombol filter.

## Langkah 2: Tambah Tombol Filter di View `index.blade.php`

Buka `resources/views/products/index.blade.php`. Tambahkan menu filter di atas tabel:

```blade
<h1>Daftar Produk</h1>

<div class="filter">
    <a href="/products">Semua</a>
    @foreach ($categories as $category)
        <a href="/products?category={{ $category->id }}">
            {{ $category->name }}
        </a>
    @endforeach
</div>

<table border="1">
    ...
</table>
```

### Penjelasan Link Filter

#### Link "Semua"

```blade
<a href="/products">Semua</a>
```

Tanpa parameter -> URL-nya `/products` -> semua produk tampil.

#### Link Tiap Kategori

```blade
<a href="/products?category={{ $category->id }}">
    {{ $category->name }}
</a>
```

Link di atas menghasilkan URL:

```
/products?category=1      (untuk Elektronik)
/products?category=2      (untuk Pakaian)
/products?category=3      (untuk Makanan)
/products?category=4      (untuk Buku)
```

> Catatan: kita pakai URL biasa (`/products?category=...`) supaya konsisten
> dengan **Materi 1, 2, dan 3** yang tidak memakai named route.

## Langkah 3: Tandai Filter yang Sedang Aktif (Opsional tapi Bagus)

Supaya user tahu kategori mana yang sedang dipilih, kita bisa **highlight** link aktif:

```blade
<div class="filter">
    <a href="/products"
       style="{{ request()->missing('category') ? 'font-weight: bold;' : '' }}">
        Semua
    </a>
    @foreach ($categories as $category)
        <a href="/products?category={{ $category->id }}"
           style="{{ request('category') == $category->id ? 'font-weight: bold;' : '' }}">
            {{ $category->name }}
        </a>
    @endforeach
</div>
```

Penjelasan:

| Bagian                                  | Arti                                                                  |
|-----------------------------------------|-----------------------------------------------------------------------|
| `request()->missing('category')`        | True kalau parameter `category` TIDAK ada di URL (mode "Semua")       |
| `request('category') == $category->id`  | True kalau parameter `category` sama dengan id kategori ini           |
| `font-weight: bold;`                    | Style bold untuk tanda aktif                                           |

Helper `request(...)` di Blade = shortcut untuk `request()->input(...)`.

## Langkah 4: Tampilkan Info "Sedang Filter Kategori Apa"

Tambah info di atas tabel supaya user tahu konteks:

```blade
@if (request()->filled('category'))
    <p>
        Menampilkan produk kategori:
        <strong>{{ $products->first()?->category?->name ?? 'Tidak ditemukan' }}</strong>
        - <a href="/products">Tampilkan semua</a>
    </p>
@endif
```

Penjelasan:

| Bagian                                  | Arti                                                              |
|-----------------------------------------|-------------------------------------------------------------------|
| `request()->filled('category')`         | Hanya tampil kalau sedang filter                                  |
| `$products->first()`                    | Ambil produk pertama dari collection                              |
| `?->category?->name`                    | Ambil nama kategori produk pertama (pakai nullsafe)               |

Alternatif yang lebih rapi: kirim nama kategori langsung dari controller.

## Alternatif: Pakai Route Parameter (Bukan Query String)

Selain query string, ada pola lain: route parameter. Contoh URL:

```
/categories/2/products    -> produk di kategori id=2
```

Cara ini cocok kalau struktur halaman mengikuti kategori (lebih SEO friendly).

### Route

Di `routes/web.php`:

```php
Route::get('/categories/{category}/products', [ProductController::class, 'byCategory'])
      ->name('products.byCategory');
```

### Controller

```php
public function byCategory(Category $category)
{
    $products = Product::where('category_id', $category->id)->with('category')->latest()->get();
    return view('products.index', compact('products', 'category'));
}
```

### Link di View

```blade
<a href="{{ route('products.byCategory', $category) }}">
    {{ $category->name }}
</a>
```

Untuk tahap belajar, **query string lebih sederhana**. Route parameter lebih cocok di tahap lanjut.

## Coba di Browser

### Default (Semua Produk)

Akses:

```
http://localhost:8000/products
```

Semua produk tampil. Link "Semua" di-bold.

### Filter Kategori

Klik salah satu kategori (misal "Pakaian"). URL berubah jadi:

```
http://localhost:8000/products?category=2
```

Hanya produk pakaian yang tampil. Link "Pakaian" di-bold. Info "Menampilkan produk kategori: Pakaian" muncul.

### Kembali ke Semua

Klik "Semua" atau "Tampilkan semua". Semua produk kembali tampil.

## Tips Penting

### 1. Filter + Pagination

Kalau produk banyak, tambahkan pagination:

```php
$products = $query->paginate(10);
```

Di view:

```blade
{{ $products->links() }}
```

Pagination otomatis bawa query string saat pindah halaman, jadi filter tetap konsisten.

### 2. Filter Lebih dari Satu Kategori (Many-to-Many)

Kalau relasinya **many-to-many** (satu produk bisa masuk banyak kategori), filter pakai `whereHas`:

```php
$query->whereHas('categories', function ($q) use ($request) {
    $q->where('categories.id', $request->category);
});
```

Tapi untuk kategori-produk yang **one-to-many** (seperti yang kita buat), `where('category_id', ...)` cukup.

### 3. Hindari SQL Injection

Pakai cara Laravel (`where(...)`) otomatis aman dari SQL injection. JANGAN pernah gabung string manual:

```php
// BERBAHAYA - jangan lakukan ini
Product::whereRaw("category_id = " . $request->category);
```

Laravel akan **binding parameter** secara aman kalau pakai `where('category_id', $request->category)`.

## Inti Pelajaran Tahap 11

> Filter produk = kirim parameter di query string (`?category=2`), controller terima lewat `$request`, tambah kondisi `where` di query builder. View bikin link filter yang mengarah ke route dengan parameter.

Yang sudah kita lakukan:

1. Tambah `Request $request` di method `index()`.
2. Bangun query builder bertahap, tambah `where('category_id', ...)` kalau parameter ada.
3. Pakai `$request->filled('category')` untuk cek parameter terisi.
4. Bikin link filter di view, dengan query string `['category' => $id]`.
5. Tandai filter aktif dengan `request(...)` helper.
6. Opsional: pakai route parameter (`/categories/{category}/products`) untuk URL yang lebih bersih.

## Selamat! Materi 4 - Kategori Produk Sudah Selesai

<json-render>
{"root":"r","elements":{"r":{"type":"Box","props":{"flexDirection":"column","padding":2,"gap":1},"children":["h","t","s"]},"h":{"type":"Heading","props":{"text":"Struktur Folder Materi","level":"h2"},"children":[]},"t":{"type":"Table","props":{"headerColor":"green","columns":[{"header":"File","key":"file","width":50},{"header":"Topik","key":"topik","width":40}],"rows":[{"file":"tahap-01-kategori-produk.md","topik":"Konsep kategori + kenapa perlu"},{"file":"tahap-02-relasi-database.md","topik":"Apa itu relasi database"},{"file":"tahap-03-one-to-many.md","topik":"One-to-many + hasMany/belongsTo"},{"file":"tahap-04-migration-categories.md","topik":"Migration tabel categories"},{"file":"tahap-05-foreign-key-products.md","topik":"Tambah category_id + foreign key"},{"file":"tahap-06-model-relasi.md","topik":"Model Category + relasi"},{"file":"tahap-07-seeder-categories.md","topik":"Seeder data kategori"},{"file":"tahap-08-crud-category.md","topik":"CRUD kategori (opsional)"},{"file":"tahap-09-dropdown-form-produk.md","topik":"Dropdown kategori di form produk"},{"file":"tahap-10-list-nama-kategori.md","topik":"Tampil nama kategori di list"},{"file":"tahap-11-filter-kategori.md","topik":"Filter produk per kategori"}]},"s":{"type":"Callout","props":{"type":"success","title":"Selamat!","content":"Kamu sudah menyelesaikan 11 tahap Materi 4 - Kategori Produk. Sekarang kamu bisa bangun aplikasi Laravel dengan relasi one-to-many dari nol."},"children":[]}}}
</json-render>

### Ringkasan Apa yang Sudah Dipelajari

1. **Konsep kategori** dan kenapa produk perlu dikelompokkan.
2. **Apa itu relasi database** dengan analogi sederhana.
3. **One-to-many**: satu kategori banyak produk, satu produk satu kategori.
4. **Migration**: cetak biru tabel database.
5. **Foreign key**: kolom `category_id` yang menghubungkan dua tabel.
6. **Model Eloquent**: jembatan kode <-> database, dengan relasi `hasMany` dan `belongsTo`.
7. **Seeder**: data awal otomatis.
8. **CRUD lengkap**: controller resource + route resource + blade views.
9. **Dropdown** kategori di form produk.
10. **Eager loading**: akses data relasi tanpa N+1 query.
11. **Filter** produk berdasarkan kategori.

### Apa Selanjutnya?

Saran lanjutan:

- **Materi 5: Pencarian Produk** - tambah search box untuk cari produk by nama.
- **Materi 6: Pagination** - tampilkan produk per halaman kalau data banyak.
- **Materi 7: Relasi Many-to-Many** - produk bisa masuk banyak kategori (dengan tabel pivot).
- **Materi 8: Authentication** - batasi akses admin (login).

Ketik **"selesai"** kalau materi 4 sudah cukup, atau sebutkan materi lanjutan yang mau dipelajari.
