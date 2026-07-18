# Tahap 10: Tampilkan Nama Kategori di List Produk

## Tujuan Tahap Ini

Di tahap 9 form produk sudah bisa pilih kategori. Tapi di halaman **daftar produk**, kolom kategori masih tampil sebagai angka `category_id` (1, 2, 3, dst), bukan nama kategori yang sebenarnya.

Tujuan kita sekarang: tampilkan **nama kategori** ("Elektronik", "Pakaian", dst) di tabel produk, dengan memakai relasi `belongsTo` yang sudah dibuat di tahap 6.

## Analogi Sederhana: Buku Tamu Acara

Bayangkan kamu panitia acara. Buku tamu berisi:

| No | Nama   | Kode Undangan |
|----|--------|---------------|
| 1  | Budi   | U-005         |
| 2  | Sinta  | U-012         |

Angka `U-005` tidak berarti apa-apa untuk orang yang lihat. Mereka tidak tahu itu undangan dari mana.

Tapi kamu punya **buku daftar undangan** terpisah:

| Kode  | Asal            |
|-------|-----------------|
| U-005 | Perusahaan A    |
| U-012 | Perusahaan B    |

Untuk membuat buku tamu yang ramah dibaca, kamu harus **mencocokkan** kode di buku tamu dengan kode di buku daftar undangan, lalu tampilkan namanya:

| No | Nama   | Asal         |
|----|--------|--------------|
| 1  | Budi   | Perusahaan A |
| 2  | Sinta  | Perusahaan B |

Inilah yang akan kita lakukan:

- `category_id` di tabel products = kode undangan.
- `name` di tabel categories = nama asal.
- Kita pakai relasi `belongsTo` untuk "mencocokkan" dan tampilkan nama.

## Langkah 1: Ubah Method `index()` di ProductController

Buka `app/Http/Controllers/ProductController.php`. Ubah method `index()` menjadi:

```php
public function index()
{
    $products = Product::with('category')->latest()->get();
    return view('products.index', compact('products'));
}
```

Penjelasan:

| Bagian                  | Arti                                                                    |
|-------------------------|-------------------------------------------------------------------------|
| `Product::with('category')` | Ambil produk **sekaligus** kategori terkait (eager loading)         |
| `'category'`            | Nama method relasi `belongsTo` yang sudah dibuat di model Product       |
| `->latest()`            | Urutkan dari yang paling baru                                           |
| `->get()`               | Eksekusi query                                                          |

## Apa itu Eager Loading?

### Lazy Loading (Bisa Boros Query)

Kalau kita **tidak** pakai `with(...)`) dan di view kita akses `$product->category->name`, Laravel menjalankan query ke database **untuk tiap produk**:

```sql
SELECT * FROM products;                    -- 1 query
SELECT * FROM categories WHERE id = 1;     -- query untuk produk 1
SELECT * FROM categories WHERE id = 2;     -- query untuk produk 2
SELECT * FROM categories WHERE id = 3;     -- query untuk produk 3
...dst
```

Jika ada 100 produk, ada 101 query. Ini disebut **N+1 problem** (1 query untuk produk, lalu N query untuk kategori tiap produk). Boros dan lambat.

### Eager Loading (Hemat Query)

Dengan `Product::with('category')`, Laravel menjalankan **hanya 2 query**:

```sql
SELECT * FROM products;
SELECT * FROM categories WHERE id IN (1, 2, 3, 4);
```

Hasilnya disimpan di memori, lalu otomatis dicocokkan ke produk yang sesuai. Cepat dan efisien.

> **Aturan praktis:** Saat menampilkan relasi di list/loop, **selalu pakai eager loading** dengan `with(...)`.

## Langkah 2: Ubah Tabel di View `index.blade.php`

Buka `resources/views/products/index.blade.php`. Tambahkan kolom "Kategori" yang menampilkan nama, bukan id.

Sebelum (hanya angka):

```blade
<table border="1">
    <tr>
        <th>Nama</th>
        <th>Harga</th>
        <th>Kategori ID</th>
        <th>Aksi</th>
    </tr>
    @foreach ($products as $product)
    <tr>
        <td>{{ $product->name }}</td>
        <td>{{ $product->price }}</td>
        <td>{{ $product->category_id }}</td>
        <td>...</td>
    </tr>
    @endforeach
</table>
```

Ubah menjadi:

```blade
<table border="1">
    <tr>
        <th>Nama</th>
        <th>Harga</th>
        <th>Kategori</th>
        <th>Aksi</th>
    </tr>
    @foreach ($products as $product)
    <tr>
        <td>{{ $product->name }}</td>
        <td>Rp {{ number_format($product->price, 0, ',', '.') }}</td>
        <td>{{ $product->category?->name ?? '-' }}</td>
        <td>
            <a href="{{ route('products.show', $product) }}">Lihat</a>
            <a href="{{ route('products.edit', $product) }}">Edit</a>
            <form action="{{ route('products.destroy', $product) }}" method="POST" style="display:inline;">
                @csrf
                @method('DELETE')
                <button type="submit" onclick="return confirm('Hapus produk ini?')">Hapus</button>
            </form>
        </td>
    </tr>
    @endforeach
</table>
```

## Penjelasan Bagian Penting

### Akses Relasi di View

```blade
<td>{{ $product->category?->name ?? '-' }}</td>
```

Bagian per bagian:

| Bagian                     | Arti                                                              |
|----------------------------|-------------------------------------------------------------------|
| `$product->category`       | Akses method relasi `category()` di model Product                 |
| `->name`                   | Ambil kolom `name` dari kategori tersebut                         |
| `?->`                      | Nullsafe operator: kalau `category` null, tidak error             |
| `?? '-'`                   | Null coalescing: kalau null, tampilkan tanda strip `-`            |

### Kenapa Pakai `?->` dan `?? '-'`?

Di tahap 5 kita pakai `onDelete('cascade')`, jadi produk yatim (tanpa kategori) seharusnya tidak ada. Tapi di project nyata, kadang kolom `category_id` bisa null (misal: kategori dihapus tapi foreign key pakai `set null`).

Dengan `?->` dan `?? '-'`, kode tidak error walau kategori null. Tampilan tetap rapi.

Kalau kamu yakin 100% kategori tidak akan null, bisa tulis singkat:

```blade
<td>{{ $product->category->name }}</td>
```

Tapi pakai nullsafe lebih aman, jadi direkomendasikan.

### Format Harga Rupiah

```blade
Rp {{ number_format($product->price, 0, ',', '.') }}
```

`number_format(angka, jumlah_desimal, pemisah_desimal, pemisah_ribuan)`:

- `0` -> tanpa desimal.
- `','` -> desimal pakai koma.
- `'.'` -> ribuan pakai titik.

Hasil: `10000000` jadi `Rp 10.000.000`.

## Coba di Browser

Akses:

```
http://localhost:8000/products
```

Sekarang tabel produk tampil:

| Nama   | Harga         | Kategori   | Aksi    |
|--------|---------------|------------|---------|
| Laptop | Rp 10.000.000 | Elektronik | Lihat Edit Hapus |
| Kaos   | Rp 50.000     | Pakaian    | Lihat Edit Hapus |
| Roti   | Rp 15.000     | Makanan    | Lihat Edit Hapus |
| Novel  | Rp 75.000     | Buku       | Lihat Edit Hapus |

Bandingkan dengan sebelumnya yang tampil "1", "2", "3", "4" di kolom kategori. Sekarang jauh lebih ramah dibaca.

## Langkah 3: Tambah Kategori di View Detail (Opsional)

Buka `resources/views/products/show.blade.php`. Tampilkan kategori di halaman detail produk:

```blade
<h1>{{ $product->name }}</h1>

<p><strong>Harga:</strong> Rp {{ number_format($product->price, 0, ',', '.') }}</p>
<p><strong>Kategori:</strong> {{ $product->category?->name ?? '-' }}</p>
<p><strong>Deskripsi:</strong> {{ $product->description }}</p>

<a href="{{ route('products.index') }}">Kembali</a>
```

Jangan lupa di method `show()` ProductController, tambahkan eager loading:

```php
public function show(Product $product)
{
    $product->load('category');
    return view('products.show', compact('product'));
}
```

Atau alternatif pakai `with` saat ambil data:

```php
public function show($id)
{
    $product = Product::with('category')->findOrFail($id);
    return view('products.show', compact('product'));
}
```

`$product->load(...)` di route model binding sama saja dengan `with(...)`, cuma dipanggil setelah objek diambil.

## Tips Penting

### 1. Selalu Eager Loading di Loop

Di view yang looping produk (`@foreach`), pastikan controller sudah `with('category')`. Tanpa ini, N+1 query akan memperlambat halaman kalau data banyak.

### 2. Cek Query yang Dijalankan

Mau lihat query Laravel sebenarnya? Pakai:

```bash
php artisan tinker
>>> Product::with('category')->get()->toArray();
```

Atau pasang package **Laravel Debugbar** untuk melihat semua query yang dijalankan per halaman (cocok untuk development).

### 3. Pakai Nullsafe di Data Berelasi

Hampir selalu aman pakai `$model->relation?->field`. Walaupun relasi seharusnya selalu ada, bisa saja data tidak konsisten karena migration lama atau bug lain. Nullsafe mencegah halaman error 500 hanya karena satu data null.

## Inti Pelajaran Tahap 10

> Eager loading (`with('category')`) bikin akses nama kategori di view cepat dan hemat query. Akses relasi `$product->category->name` langsung menampilkan data dari tabel terkait, tanpa tulis SQL JOIN manual.

Yang sudah kita lakukan:

1. Ubah `Product::with('category')` di method `index()`.
2. Tampilkan `{{ $product->category?->name }}` di tabel produk.
3. Pakai nullsafe `?->` dan null coalescing `??` supaya aman walau kategori null.
4. Tambah format Rupiah dengan `number_format()`.
5. (Opsional) Tambah kategori di view detail produk.

Sekarang list produk sudah ramah dibaca dengan nama kategori. Tapi belum bisa **filter** berdasarkan kategori. Itu yang akan kita buat di tahap terakhir.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke langkah terakhir: **filter produk berdasarkan kategori**?

Di tahap 11 kita akan:

- Tambah tombol/link filter berdasarkan kategori di halaman list produk.
- Pakai query `where('category_id', $id)` di controller.
- Pakai route dengan parameter: `/products?category=2`.

Ketik **"lanjut"** kalau siap menyelesaikan materi ini.
