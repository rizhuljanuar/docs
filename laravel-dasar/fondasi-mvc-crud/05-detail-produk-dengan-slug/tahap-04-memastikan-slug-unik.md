# Tahap 4: Memastikan Setiap Slug Unik

## Tujuan Tahap Ini

Pada Tahap 3, Laravel sudah bisa membuat slug dari nama produk. Namun, dua
produk dengan nama yang sama masih menghasilkan slug yang sama.

Contohnya:

| ID | Nama Produk | Slug         |
|----|-------------|--------------|
| 7  | Kaos Hitam  | `kaos-hitam` |
| 8  | Kaos Hitam  | `kaos-hitam` |

Di tahap ini kita akan:

1. Menambahkan ID produk pada akhir slug.
2. Memperbaiki slug produk lama.
3. Menambahkan aturan unik pada database.

Kita belum menggunakan slug untuk route model binding.

## Apa Arti Unik?

**Unik** berarti satu nilai hanya boleh dimiliki oleh satu produk.

Jika slug dipakai sebagai alamat, Laravel harus dapat menemukan tepat satu
produk. Slug yang sama membuat alamat menjadi membingungkan:

```text
/produk/kaos-hitam
```

Laravel tidak tahu apakah alamat tersebut mengarah ke produk ID `7` atau ID
`8`.

## Analogi Sederhana: Nama Siswa dan Nomor Induk

Dalam satu sekolah mungkin ada dua siswa bernama Budi. Nama saja belum cukup
untuk membedakan mereka.

Sekolah menambahkan nomor induk:

```text
budi-7
budi-8
```

Kita akan melakukan hal yang sama pada slug:

```text
kaos-hitam-7
kaos-hitam-8
```

ID produk selalu berbeda, sehingga slug tersebut juga selalu berbeda.

## Bentuk Slug yang Akan Digunakan

Pola slug kita adalah:

```text
nama-produk-id
```

Contoh:

| ID | Nama Produk        | Slug                    |
|----|--------------------|-------------------------|
| 5  | Sepatu Lari Pria   | `sepatu-lari-pria-5`    |
| 6  | Sepatu Lari Pria   | `sepatu-lari-pria-6`    |

Cara ini sederhana dan tidak memerlukan perulangan untuk mencari slug yang
belum dipakai.

## Langkah 1: Mengubah Proses Penyimpanan Produk

Buka:

```text
app/Http/Controllers/ProductController.php
```

Pastikan import `Str` dari Tahap 3 masih ada:

```php
use Illuminate\Support\Str;
```

Kemudian ubah method `store()` menjadi:

```php
public function store(StoreProductRequest $request): RedirectResponse
{
    $product = Product::create($request->validated());

    $product->update([
        'slug' => Str::slug($product->name) . '-' . $product->id,
    ]);

    return redirect('/products')
        ->with('success', 'Produk berhasil ditambahkan.');
}
```

Kenapa produk disimpan terlebih dahulu?

ID baru dibuat oleh database setelah produk disimpan. Setelah ID tersedia,
Laravel menggabungkan:

```text
Str::slug(nama) + tanda hubung + ID
```

Contohnya:

```text
Str::slug('Kaos Hitam') . '-' . 7

Hasil: kaos-hitam-7
```

Kolom `slug` masih `nullable`, sehingga produk boleh disimpan sebentar tanpa
slug sebelum langsung diperbarui pada baris berikutnya.

## Jika Validasi Masih Langsung di Controller

Jika belum memakai `StoreProductRequest`, simpan hasil validasi terlebih
dahulu:

```php
$data = $request->validate([
    'name' => 'required|string|max:255',
    'description' => 'nullable|string',
    'price' => 'required|integer|min:0',
    'stock' => 'required|integer|min:0',
]);

$product = Product::create($data);

$product->update([
    'slug' => Str::slug($product->name) . '-' . $product->id,
]);
```

Pilih versi yang sesuai dengan project-mu. Jangan gunakan keduanya sekaligus.

## Langkah 2: Memperbarui Slug Saat Nama Produk Diubah

Jika nama produk berubah, slug juga perlu diperbarui.

Ubah method `update()` menjadi:

```php
public function update(
    StoreProductRequest $request,
    Product $product
): RedirectResponse {
    $data = $request->validated();
    $data['slug'] = Str::slug($data['name']) . '-' . $product->id;

    $product->update($data);

    return redirect('/products')
        ->with('success', 'Produk berhasil diupdate.');
}
```

Contohnya:

```text
Sebelum: Kaos Hitam  → kaos-hitam-7
Sesudah: Kaos Polos  → kaos-polos-7
```

ID `7` tetap sama karena produknya masih produk yang sama.

## Langkah 3: Memperbaiki Slug Produk Lama

Sebelum memasang aturan unik di database, ubah semua slug lama agar mengikuti
pola baru.

Jalankan:

```bash
php artisan tinker
```

Kemudian:

```php
\App\Models\Product::all()->each(function ($product) {
    $product->slug = \Illuminate\Support\Str::slug($product->name)
        . '-' . $product->id;
    $product->save();
});
```

Setelah selesai, keluar:

```php
exit
```

Sekarang setiap slug memiliki ID yang berbeda.

## Langkah 4: Membuat Migration Aturan Unik

Buat migration baru:

```bash
php artisan make:migration add_unique_index_to_products_slug
```

Buka file migration yang baru dibuat, lalu isi method `up()` dan `down()`:

```php
public function up(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->unique('slug');
    });
}

public function down(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->dropUnique(['slug']);
    });
}
```

### `$table->unique('slug')`

Perintah ini meminta database memastikan dua produk tidak memiliki slug yang
sama.

Jika aplikasi mencoba menyimpan slug yang sudah dipakai, database akan
menolaknya. Ini menjadi lapisan pengaman terakhir.

### `$table->dropUnique(['slug'])`

Perintah ini menghapus aturan unik ketika migration di-rollback. Kolom `slug`
tetap ada karena migration ini hanya mengatur indeks unik.

## Langkah 5: Menjalankan Migration

Jalankan:

```bash
php artisan migrate
```

Pastikan langkah memperbaiki slug produk lama sudah dilakukan lebih dahulu.
Jika masih ada slug yang sama, migration aturan unik dapat gagal.

## Langkah 6: Menguji Hasil

Buat dua produk dengan nama yang sama:

```text
Produk pertama: Kaos Hitam
Produk kedua:   Kaos Hitam
```

Hasil yang diharapkan:

| ID | Nama Produk | Slug           |
|----|-------------|----------------|
| 7  | Kaos Hitam  | `kaos-hitam-7` |
| 8  | Kaos Hitam  | `kaos-hitam-8` |

Nama produknya sama, tetapi slug tetap berbeda.

Ubah nama salah satu produk, lalu pastikan nama pada slug ikut berubah dan ID
di akhirnya tetap sama.

## Checklist Tahap 4

- [ ] Method `store()` menambahkan ID pada akhir slug.
- [ ] Method `update()` memperbarui slug saat nama berubah.
- [ ] Slug produk lama sudah diperbaiki melalui Tinker.
- [ ] Migration aturan unik sudah dibuat.
- [ ] Perintah `php artisan migrate` berhasil.
- [ ] Dua produk bernama sama memiliki slug berbeda.

## Inti Tahap 4

> Menambahkan ID pada akhir slug membuat slug selalu berbeda. Aturan `unique`
> di database memastikan tidak ada slug yang tersimpan dua kali.

Sekarang setiap produk memiliki slug yang jelas dan unik. Slug tersebut belum
digunakan pada URL.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke **Tahap 5: menggunakan route model binding
berdasarkan slug**?

Ketik **"lanjut"** jika sudah siap.
