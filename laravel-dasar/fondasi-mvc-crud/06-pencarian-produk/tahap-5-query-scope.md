# Tahap 5: Local Scope untuk Pencarian

## Tujuan

Query dari Tahap 3 sudah bekerja, tetapi lebih rapi jika disimpan sebagai local scope pada model `Product`. Kita memakai gaya tradisional `scopeSearch`, agar jelas hubungan antara method model dan pemanggilan query.

## Tambahkan Scope pada `Product`

Buka `app/Models/Product.php`. Pertahankan `$fillable` dan relasi `category()` yang sudah ada, lalu tambahkan import dan scope berikut:

```php
use Illuminate\Database\Eloquent\Builder;
```

```php
public function scopeSearch(Builder $query, mixed $keyword): Builder
{
    $keyword = trim((string) $keyword);

    if ($keyword === '') {
        return $query;
    }

    return $query->where('name', 'like', '%' . $keyword . '%');
}
```

`scopeSearch()` dipanggil sebagai `->search()`; prefix `scope` tidak ditulis saat pemanggilan.

## Perbarui Controller

Method `index()` menjadi:

```php
public function index(Request $request)
{
    $validated = $request->validate([
        'search' => ['nullable', 'string', 'max:100'],
    ]);

    $products = Product::with('category')
        ->search($validated['search'] ?? '')
        ->paginate(10)
        ->withQueryString();

    return view('products.index', compact('products'));
}
```

## Perilaku Input Kosong

Saat `/products` dibuka tanpa parameter `search`, scope menerima string kosong lalu mengembalikan query tanpa kondisi `LIKE`. Dengan demikian semua produk tampil secara sengaja, bukan karena query `LIKE '%%'`.

## Inti Tahap 5

> Local scope membuat controller lebih singkat dan dapat dipakai ulang. Pada tahap ini `scopeSearch()` masih mencari `name` saja. Tahap 6 memperluasnya ke `description` dan menambah filter `category_id`.
