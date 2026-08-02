# Tahap 2 — Migration dan Trait `SoftDeletes`

Soft delete memerlukan kolom `deleted_at` pada tabel `products` dan trait pada model `Product`.

## Buat migration

```bash
php artisan make:migration add_deleted_at_to_products_table --table=products
```

Isi migration baru sebagai berikut.

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('products', function (Blueprint $table) {
            $table->softDeletes();
        });
    }

    public function down(): void
    {
        Schema::table('products', function (Blueprint $table) {
            $table->dropSoftDeletes();
        });
    }
};
```

`$table->softDeletes()` menambahkan kolom timestamp nullable `deleted_at`. Jalankan migration:

```bash
php artisan migrate
```

## Perbarui model `Product`

Tambahkan import dan trait. Tetap pertahankan field kontrak CRUD yang boleh diisi serta relasi `category()`.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\SoftDeletes;

class Product extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = [
        'name',
        'price',
        'stock',
        'description',
        'category_id',
        'image',
        'slug',
    ];

    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }
}
```

Jangan tambahkan `deleted_at` ke `$fillable`: kolom itu dikelola oleh Laravel melalui trait `SoftDeletes`. Trait inilah yang membuat `delete()`, `restore()`, `onlyTrashed()`, dan `forceDelete()` memiliki perilaku soft delete.

Setelah ini, `Product::query()` otomatis mengecualikan produk yang `deleted_at`-nya terisi.
