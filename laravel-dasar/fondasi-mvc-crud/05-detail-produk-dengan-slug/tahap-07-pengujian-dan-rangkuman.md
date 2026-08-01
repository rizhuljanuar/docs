# Tahap 7: Pengujian Akhir dan Rangkuman Slug

## Tujuan Tahap Ini

Semua bagian fitur slug sudah dibuat. Sekarang kita akan menguji alurnya dari
awal sampai akhir.

Kita akan memastikan:

- Produk baru mendapatkan slug otomatis.
- Dua nama produk yang sama tetap memiliki slug berbeda.
- Tautan detail memakai slug.
- Method `show()` menemukan produk yang benar berdasarkan slug.
- Slug yang tidak ada menghasilkan halaman 404.
- Fitur edit dan hapus tetap bekerja.

Tidak ada fitur baru yang dibuat pada tahap ini.

## Analogi Sederhana: Memeriksa Jalur Pengiriman

Bayangkan kita baru membuat jalur pengiriman paket:

1. Paket diberi label.
2. Label harus berbeda dari paket lain.
3. Kurir membaca label.
4. Paket sampai kepada penerima yang benar.

Fitur slug bekerja dengan cara yang mirip:

1. Produk dibuat.
2. Laravel membuat slug yang unik.
3. Tautan mengirim slug melalui URL.
4. Method show() menerima slug dan mencari produk.
5. Halaman detail produk yang benar tampil.

## Persiapan Pengujian

Pastikan migration sudah dijalankan:

```bash
php artisan migrate:status
```

Migration penambahan kolom slug dan indeks unik harus berstatus `Ran`.

Periksa route produk:

```bash
php artisan route:list --path=products
```

Pastikan route detail memakai parameter `{slug}` dan tidak ada dua route
detail yang saling bertabrakan.

Jalankan aplikasi:

```bash
php artisan serve
```

Kemudian buka:

```text
http://127.0.0.1:8000/products
```

## Pengujian 1: Membuat Produk Baru

Tambahkan produk:

```text
Nama: Sepatu Lari Pria
```

Periksa tabel `products`. Misalnya produk mendapatkan ID `15`, hasil yang
diharapkan adalah:

```text
name: Sepatu Lari Pria
slug: sepatu-lari-pria-15
```

Berhasil jika:

- Produk tersimpan.
- Slug tidak kosong.
- Slug memakai huruf kecil dan tanda hubung.
- Slug diakhiri ID produk.

## Pengujian 2: Membuat Produk dengan Nama yang Sama

Tambahkan satu produk lagi dengan nama:

```text
Sepatu Lari Pria
```

Jika produk kedua mendapatkan ID `16`, hasilnya:

| ID | Nama Produk        | Slug                    |
|----|--------------------|-------------------------|
| 15 | Sepatu Lari Pria   | `sepatu-lari-pria-15`   |
| 16 | Sepatu Lari Pria   | `sepatu-lari-pria-16`   |

Berhasil jika kedua produk tersimpan dan slug-nya berbeda.

## Pengujian 3: Membuka Detail dari Daftar

Pada halaman daftar produk:

1. Klik tombol **Lihat** pada produk ID `15`.
2. Perhatikan URL pada browser.

Hasil yang diharapkan:

```text
http://127.0.0.1:8000/products/sepatu-lari-pria-15
```

Berhasil jika:

- URL memakai slug, bukan hanya ID.
- Halaman menampilkan produk ID `15`.
- Nama, harga, stok, kategori, dan informasi lainnya sesuai.

## Pengujian 4: Membuka URL Secara Langsung

Salin URL slug, lalu buka pada tab baru:

```text
http://127.0.0.1:8000/products/sepatu-lari-pria-15
```

Berhasil jika halaman detail yang sama tetap tampil tanpa harus masuk melalui
halaman daftar.

## Pengujian 5: Membuka Slug yang Tidak Ada

Buka:

```text
http://127.0.0.1:8000/products/produk-tidak-ada-999
```

Hasil yang diharapkan:

```text
404 Not Found
```

Ini bukan kerusakan. Laravel memang harus menampilkan 404 jika slug tidak
ditemukan.

## Pengujian 6: Mengubah Nama Produk

Ubah nama produk ID `15`:

```text
Sebelum: Sepatu Lari Pria
Sesudah: Sepatu Lari Wanita
```

Hasil slug yang diharapkan:

```text
sepatu-lari-wanita-15
```

Berhasil jika:

- Bagian nama pada slug ikut berubah.
- ID `15` di akhir slug tetap sama.
- URL baru membuka produk yang benar.

URL lama akan menghasilkan 404 karena implementasi dasar ini tidak menyimpan
riwayat slug. Redirect dari slug lama dapat dipelajari nanti jika aplikasi
benar-benar membutuhkannya.

## Pengujian 7: Memastikan CRUD Lain Tetap Bekerja

Uji tindakan berikut:

1. Buka form edit.
2. Simpan perubahan produk.
3. Buat satu produk sementara.
4. Hapus produk sementara tersebut.

Berhasil jika edit, update, dan hapus tetap bekerja dengan ID seperti
sebelumnya.

## Alur Lengkap Fitur Slug

```text
Admin mengisi nama produk
        |
        v
Product disimpan dan mendapat ID
        |
        v
Str::slug(name) + '-' + ID
        |
        v
Slug disimpan ke tabel products
        |
        v
Blade membuat tautan dari $product->slug
        |
        v
Browser membuka /products/nama-produk-id
        |
        v
Route {slug} menerima slug
        |
        v
Controller mencari Product dengan where('slug', $slug)->firstOrFail()
        |
        v
View menampilkan detail produk
```

## Ringkasan Perubahan Setiap File

### Migration

Migration pertama menambahkan kolom:

```php
$table->string('slug')->nullable()->after('name');
```

Migration berikutnya menjaga nilai slug tetap unik:

```php
$table->unique('slug');
```

### Model `Product`

Kolom `slug` ditambahkan ke `$fillable` agar dapat disimpan melalui Eloquent:

```php
protected $fillable = [
    'name',
    'slug',
    'price',
    'description',
    'category_id',
    'image',
    'stock',
];
```

### `ProductController`

Laravel membuat slug dari nama dan ID:

```php
$validated['slug'] = Str::slug($validated['name']) . '-' . $product->id;
```

Method `show()` menerima `$slug` dan mencari produk berdasarkan kolom slug:

```php
public function show($slug)
{
    $product = Product::with('category')
        ->where('slug', $slug)
        ->firstOrFail();

    return view('products.show', compact('product'));
}
```

### Route

Route detail menerima slug alih-alih ID:

```php
Route::get('/products/{slug}', [ProductController::class, 'show']);
```

Route lain (edit, update, destroy) tetap memakai `{id}`.

### View Daftar Produk

Tautan detail mengirim nilai slug dengan URL biasa:

```blade
<a href="/products/{{ $product->slug }}">Lihat</a>
```

## Masalah dan Tempat Pemeriksaannya

| Masalah                         | Periksa                                      |
|---------------------------------|----------------------------------------------|
| Slug kosong                     | Method `store()` dan `$fillable`             |
| Slug sama                       | Akhiran ID dan migration indeks unik         |
| Klik Lihat menghasilkan 404     | Nilai `$product->slug` pada tautan            |
| Produk salah                    | Parameter `{slug}` di route dan method `show()` |
| Edit atau hapus gagal           | Pastikan route tersebut masih memakai `{id}` |

## Checklist Akhir

- [ ] Kolom `slug` tersedia pada tabel `products`.
- [ ] Slug dibuat otomatis dari nama dan ID.
- [ ] Setiap produk memiliki slug yang berbeda.
- [ ] Database memiliki indeks unik untuk kolom `slug`.
- [ ] Produk lama sudah memiliki slug.
- [ ] Route detail memakai `{slug}`.
- [ ] Method `show()` menerima `$slug` dan memakai `where('slug', ...)->firstOrFail()`.
- [ ] Tautan detail mengirim `$product->slug` dengan URL biasa.
- [ ] URL slug menampilkan produk yang benar.
- [ ] Slug yang tidak ada menghasilkan 404.
- [ ] Edit, update, dan hapus tetap bekerja.

## Inti Seluruh Materi

> Slug membuat URL detail produk lebih mudah dibaca. Laravel membuat slug dari
> nama produk, database menjaganya tetap unik, dan method `show()` mencari
> produk berdasarkan slug tersebut.

Perubahan akhirnya:

```text
Sebelum: /products/15
Sesudah: /products/sepatu-lari-pria-15
```

Materi **Detail Produk dengan Slug** selesai.
