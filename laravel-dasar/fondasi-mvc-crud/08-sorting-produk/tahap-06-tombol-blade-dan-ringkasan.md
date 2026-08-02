# Tahap 6 — Select Sorting di Blade dan Ringkasan

Tambahkan pilihan sorting ke form pencarian/filter yang sudah ada pada `resources/views/products/index.blade.php`. Form tetap memakai `GET` dan tidak memakai `@csrf`.

```blade
<form action="/products" method="GET">
    <input type="text" name="search" value="{{ request('search') }}" maxlength="100">

    <select name="category_id">
        <option value="">Semua kategori</option>
        @foreach ($categories as $category)
            <option value="{{ $category->id }}"
                @selected((string) request('category_id') === (string) $category->id)>
                {{ $category->name }}
            </option>
        @endforeach
    </select>

    <select name="sort">
        <option value="newest" @selected(request('sort', 'newest') === 'newest')>Terbaru</option>
        <option value="oldest" @selected(request('sort') === 'oldest')>Terlama</option>
        <option value="price-asc" @selected(request('sort') === 'price-asc')>Harga: rendah ke tinggi</option>
        <option value="price-desc" @selected(request('sort') === 'price-desc')>Harga: tinggi ke rendah</option>
        <option value="name-asc" @selected(request('sort') === 'name-asc')>Nama: A–Z</option>
        <option value="name-desc" @selected(request('sort') === 'name-desc')>Nama: Z–A</option>
    </select>

    <button type="submit">Terapkan</button>
    <a href="/products">Reset</a>
</form>
```

Select berada dalam form yang sama agar `search`, `category_id`, dan `sort` dikirim bersama. Form tidak memiliki field `page`; ketika pengguna memilih urutan baru lalu mengirim form, URL baru secara alami dimulai dari halaman pertama. Ini mencegah pengguna tetap berada di halaman N yang mungkin tidak ada untuk urutan baru.

## Pagination

Tidak perlu memakai `appends()`. Query controller sudah memakai `withQueryString()`, sehingga semua parameter URL saat ini—termasuk `search`, `category_id`, dan `sort`—dipertahankan oleh paginator.

```blade
{{ $products->links() }}
```

Contoh setelah pindah halaman:

```text
/products?search=laptop&category_id=2&sort=price-asc&page=2
```

## Bagian daftar produk tetap konsisten

Tetap gunakan variabel dan path dari materi sebelumnya. Detail memakai slug, sedangkan edit dan hapus memakai ID; form hapus tetap memakai CSRF dan method DELETE.

```blade
@forelse ($products as $product)
    <article>
        <h2>{{ $product->name }}</h2>
        <p>{{ $product->category?->name }}</p>
        <a href="/products/{{ $product->slug }}">Detail</a>
        <a href="/products/{{ $product->id }}/edit">Edit</a>

        <form action="/products/{{ $product->id }}" method="POST">
            @csrf
            @method('DELETE')
            <button type="submit">Hapus</button>
        </form>
    </article>
@empty
    <p>Produk tidak ditemukan.</p>
@endforelse

{{ $products->links() }}
```

## Checklist akhir

- [ ] Route indeks tetap `Route::get('/products', [ProductController::class, 'index']);`.
- [ ] Validasi mempertahankan `search` maksimal 100 dan `category_id` yang ada di tabel kategori.
- [ ] `sort` divalidasi menggunakan key dari `$allowedSorts`.
- [ ] Query memakai `with('category')`, `search()`, `filterByCategory()`, dua `orderBy()`, `paginate(10)`, dan `withQueryString()`.
- [ ] Select mempertahankan pilihan aktif dengan `@selected`.
- [ ] Form GET tidak memiliki CSRF dan tidak mengirim `page`.
- [ ] Pagination hanya memakai `{{ $products->links() }}`.
- [ ] Detail tetap memakai slug; edit/hapus memakai ID dan form hapus memiliki `@csrf` serta `@method('DELETE')`.

## Ringkasan

Sorting yang aman memakai map lokal sebagai whitelist. Input `sort` tervalidasi sebelum map menghasilkan kolom dan arah `orderBy()`. Pencarian, filter kategori, relasi kategori, dan pagination tetap berjalan dalam satu query, sementara `withQueryString()` menjaga seluruh kondisi saat pengguna berpindah halaman.
