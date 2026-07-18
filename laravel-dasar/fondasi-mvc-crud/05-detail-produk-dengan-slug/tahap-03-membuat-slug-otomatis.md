# Tahap 3: Membuat Slug Otomatis dari Nama Produk

## Tujuan Tahap Ini

Pada Tahap 2 kita sudah menambahkan kolom `slug` ke tabel `products`. Namun,
kolom tersebut masih kosong.

Sekarang kita akan membuat slug secara otomatis ketika produk disimpan.

Contohnya:

```text
Nama produk: Sepatu Lari Pria
Slug:        sepatu-lari-pria
```

Di tahap ini kita belum memastikan slug unik dan belum menggunakannya pada URL.

## Kenapa Slug Sebaiknya Dibuat Otomatis?

Admin seharusnya cukup menulis nama produk. Jika admin juga harus menulis slug,
pekerjaannya menjadi berulang dan hasilnya bisa tidak konsisten.

Contohnya, beberapa orang mungkin menulis:

```text
Sepatu-Lari-Pria
sepatu lari pria
SEPATU_LARI_PRIA
```

Laravel dapat mengubah semuanya menjadi bentuk yang rapi:

```text
sepatu-lari-pria
```

## Analogi Sederhana: Mesin Pembuat Label

Bayangkan admin menulis **Sepatu Lari Pria** pada formulir. Setelah formulir
diterima, sebuah mesin otomatis mencetak label **sepatu-lari-pria**.

Dalam Laravel:

- Nama produk adalah tulisan dari admin.
- `Str::slug()` adalah mesin pembuat label.
- Kolom `slug` adalah tempat menyimpan label tersebut.

## Mengenal `Str::slug()`

Laravel menyediakan alat bernama `Str`. Salah satu fungsinya adalah
`Str::slug()`.

Contoh:

```php
Str::slug('Sepatu Lari Pria');
```

Hasilnya:

```text
sepatu-lari-pria
```

`Str::slug()` akan:

- Mengubah huruf menjadi kecil.
- Mengganti spasi dengan tanda hubung.
- Menghilangkan karakter yang tidak cocok untuk URL.

Kita tidak perlu membuat fungsi pengubah slug sendiri.

## Langkah 1: Mengizinkan Model Mengisi Kolom Slug

Buka:

```text
app/Models/Product.php
```

Tambahkan `slug` ke dalam daftar `$fillable` yang sudah ada:

```diff
protected $fillable = [
    'name',
+    'slug',
    'description',
    'price',
    'stock',
];
```

Tanda `+` hanya menunjukkan baris baru, jadi tidak perlu diketik. Jika
`$fillable` milikmu juga memiliki `gambar` atau `category_id`, pertahankan
kolom-kolom tersebut.

`$fillable` adalah daftar kolom yang boleh diisi melalui
`Product::create()` atau `$product->update()`.

Jika `slug` tidak ditambahkan, nilai slug yang kita kirim dapat diabaikan oleh
Laravel.

## Langkah 2: Memanggil `Str` di Controller

Buka:

```text
app/Http/Controllers/ProductController.php
```

Di bagian atas file, tambahkan:

```php
use Illuminate\Support\Str;
```

Baris tersebut membuat class `Str` tersedia di dalam controller.

## Langkah 3: Membuat Slug Saat Produk Baru Disimpan

Cari method `store()`. Jika controller menggunakan
`StoreProductRequest`, ubah bagian penyimpanannya menjadi:

```php
public function store(StoreProductRequest $request): RedirectResponse
{
    $data = $request->validated();
    $data['slug'] = Str::slug($data['name']);

    Product::create($data);

    return redirect('/products')
        ->with('success', 'Produk berhasil ditambahkan.');
}
```

Bagian baru yang penting adalah:

```php
$data['slug'] = Str::slug($data['name']);
```

Alurnya:

1. `$request->validated()` mengambil data form yang sudah lolos validasi.
2. `$data['name']` berisi nama produk.
3. `Str::slug(...)` mengubah nama menjadi slug.
4. Hasilnya dimasukkan ke `$data['slug']`.
5. `Product::create($data)` menyimpan semua data ke database.

Slug tidak perlu ditambahkan ke form karena Laravel membuatnya sendiri.

## Jika Validasi Masih Langsung di Controller

Jika method `store()` masih memakai `$request->validate()`, gunakan pola yang
sama:

```php
$data = $request->validate([
    'name' => 'required|string|max:255',
    'description' => 'nullable|string',
    'price' => 'required|integer|min:0',
    'stock' => 'required|integer|min:0',
]);

$data['slug'] = Str::slug($data['name']);

Product::create($data);
```

Pilih kode yang sesuai dengan project-mu. Jangan memakai kedua versi sekaligus.

## Langkah 4: Mengisi Slug Produk Lama

Produk baru sekarang mendapatkan slug otomatis. Produk yang dibuat sebelum
Tahap 3 masih memiliki slug kosong.

Jalankan Tinker:

```bash
php artisan tinker
```

Kemudian jalankan:

```php
\App\Models\Product::whereNull('slug')->get()->each(function ($product) {
    $product->slug = \Illuminate\Support\Str::slug($product->name);
    $product->save();
});
```

Perintah tersebut akan mengambil produk yang slug-nya kosong, membuat slug
dari namanya, lalu menyimpannya. Nama class ditulis lengkap agar Tinker tahu
model dan alat `Str` yang harus digunakan.

Setelah selesai, keluar dari Tinker:

```php
exit
```

## Langkah 5: Menguji Hasil

Tambahkan produk baru melalui form:

```text
Nama: Tas Sekolah Anak
```

Lalu periksa tabel `products`. Hasil yang diharapkan:

| name               | slug                 |
|--------------------|----------------------|
| Tas Sekolah Anak   | `tas-sekolah-anak`   |

Periksa juga produk lama. Kolom slug seharusnya tidak lagi kosong.

## Masalah yang Belum Diselesaikan

Jika ada dua produk dengan nama yang sama:

```text
Kaos Hitam
Kaos Hitam
```

Keduanya akan menghasilkan slug yang sama:

```text
kaos-hitam
kaos-hitam
```

Untuk saat ini, biarkan seperti itu. Kita akan memastikan setiap slug berbeda
pada tahap berikutnya.

## Checklist Tahap 3

- [ ] `slug` sudah ditambahkan ke `$fillable`.
- [ ] Controller sudah mengimpor `Illuminate\Support\Str`.
- [ ] Method `store()` membuat slug dengan `Str::slug()`.
- [ ] Produk baru mendapatkan slug otomatis.
- [ ] Slug produk lama sudah diisi melalui Tinker.

## Inti Tahap 3

> `Str::slug()` mengubah nama produk menjadi teks yang cocok untuk URL, lalu
> Laravel menyimpannya ke kolom `slug`.

Pada tahap ini slug sudah dibuat otomatis, tetapi belum dijamin unik dan belum
digunakan untuk membuka halaman detail produk.

---

## Pertanyaan Lanjutan

Apakah kamu ingin lanjut ke **Tahap 4: memastikan setiap slug unik**?

Ketik **"lanjut"** jika sudah siap.
