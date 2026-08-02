# Tahap 2 — Migration Kolom `is_active`

Kita menambah status publikasi ke tabel `products` yang sudah ada. Jangan mengubah migration lama yang membuat tabel tersebut; buat migration baru.

```bash
php artisan make:migration add_is_active_to_products_table --table=products
```

Isi bagian penting migration:

```php
public function up(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->boolean('is_active')->default(false);
    });
}

public function down(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->dropColumn('is_active');
    });
}
```

Jalankan migration:

```bash
php artisan migrate
```

## Kontrak schema

`is_active` adalah boolean. Pada database tertentu boolean disimpan sebagai `0` dan `1`, tetapi di kode Laravel gunakan `false` dan `true` agar maksudnya jelas.

`->default(false)` membuat produk baru nonaktif secara aman. Untuk tabel yang sudah memiliki data, nilai default membuat baris lama menjadi `false`. Putuskan secara eksplisit bagaimana produk historis yang sudah benar-benar layak dipublikasikan akan dipublikasikan: buat migration/backfill yang disengaja atau periksa lalu ubah melalui proses manajemen. Jangan menganggap seluruh data lama otomatis aman untuk dipublikasikan.

Kolom ini **tidak menghilangkan produk dari index `/products`**. Index manajemen memang menampilkan produk aktif dan nonaktif yang belum dihapus. Filter `active()` baru dipakai nanti oleh query storefront publik yang terpisah.

## Perbedaan dengan `deleted_at`

`deleted_at` tetap milik lifecycle tong sampah dari materi 09. Item dengan `deleted_at` terisi tidak ikut query normal. `is_active` tidak menghapus data; ia hanya menyimpan keputusan publikasi. Saat restore, Laravel mengosongkan `deleted_at` tanpa mengubah `is_active`.
