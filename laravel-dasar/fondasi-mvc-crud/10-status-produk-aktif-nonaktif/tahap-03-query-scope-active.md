# Tahap 3 — Cast dan Query Scope `active()`

Buka `app/Models/Product.php`. Model tetap memakai factory, soft delete, relasi kategori, dan seluruh field lama. Tambahkan `is_active` ke `$fillable`, lalu buat cast dan scope berikut.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
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
        'is_active',
    ];

    protected function casts(): array
    {
        return [
            'is_active' => 'boolean',
        ];
    }

    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    public function scopeActive(Builder $query): Builder
    {
        return $query->where('is_active', true);
    }
}
```

`casts()` memastikan `$product->is_active` dibaca sebagai boolean PHP. Tambahkan `is_active` ke `$fillable` agar update terverifikasi pada tahap berikutnya dapat memakai `$product->update($validated)`; tetap validasi input sebelum mass assignment.

`scopeActive()` adalah local scope tradisional yang typed. Contoh **query public masa depan**:

```php
$products = Product::active()
    ->with('category')
    ->orderBy('name')
    ->paginate(10);
```

Scope tersebut menghasilkan produk dengan `is_active = true`. Karena `Product` memakai `SoftDeletes`, query normal juga otomatis mengecualikan trashed products. Jadi `active()` berarti: aktif **dan** belum di-soft-delete.

Jangan memasang `active()` pada index manajemen `/products`; admin harus tetap melihat kedua status. Jangan menambah storefront sekarang: scope ini hanya persiapan untuk permukaan publik yang berbeda di kemudian hari.
