# Tahap 5 — Membuat Component Error Message yang Dapat Dipakai Ulang

> Fokus: memindahkan kode pesan error yang berulang ke satu anonymous Blade component.

Pada tahap 4, setiap field form product memakai kode yang sama:

```blade
@error('price')
    <p role="alert">{{ $message }}</p>
@enderror
```

Perbedaannya hanya nama field, misalnya `name`, `price`, `stock`, `category_id`, atau `image`.

Menulis ulang kode yang sama masih bekerja. Namun jika nanti kita ingin mengubah tampilan pesan, kita harus mengubahnya di banyak tempat. Karena itu, sekarang kita membuat satu component kecil bernama **error message**.

## Analogi: cetakan label peringatan

Bayangkan toko memiliki banyak rak product. Setiap rak membutuhkan label peringatan jika ada masalah.

Daripada karyawan menulis label dari awal di setiap rak, toko membuat satu **cetakan label**. Karyawan hanya menyebutkan rak mana yang bermasalah, lalu cetakan menghasilkan label dengan bentuk yang sama.

Component Blade adalah cetakan tersebut:

- File component menyimpan bentuk pesan error.
- Form product cukup mengirim nama field yang ingin diperiksa.
- Laravel mengambil pesan yang sesuai dari `$errors`.

## Apa yang akan berubah?

Sebelum memakai component, error harga ditulis seperti ini:

```blade
@error('price')
    <p role="alert">{{ $message }}</p>
@enderror
```

Sesudah component dibuat, pemakaiannya menjadi lebih pendek:

```blade
<x-error-message field="price" />
```

Keduanya menghasilkan pesan error yang sama. Versi component membantu kita menjaga tampilan tetap konsisten.

## Buat file component

Buat file baru berikut:

```text
resources/views/components/error-message.blade.php
```

File di dalam folder `components` otomatis dikenali Laravel 13+ sebagai **anonymous Blade component**. Kita tidak perlu membuat class PHP, controller, route, atau mendaftarkan component secara manual.

Isi file tersebut dengan kode berikut:

```blade
@props(['field'])

@error($field)
    <p role="alert">{{ $message }}</p>
@enderror
```

## Penjelasan setiap bagian

| Kode | Arti sederhana |
| --- | --- |
| `@props(['field'])` | Component menerima satu data bernama `field`. Data ini wajib dikirim saat component dipakai. |
| `@error($field)` | Periksa `$errors` untuk field yang namanya diberikan ke component. |
| `<p role="alert">` | Wadah untuk menampilkan pesan penting kepada user. |
| `{{ $message }}` | Pesan validasi dari Laravel, misalnya *Harga tidak boleh kurang dari 0*. Blade menampilkan teks ini dengan aman. |
| `@enderror` | Menutup pemeriksaan error. |

Di dalam component, `$field` bisa berisi `name`, `price`, `stock`, `description`, `category_id`, atau `image`. Karena itulah satu component bisa dipakai untuk banyak field.

## Cara memakai component

Setelah file `error-message.blade.php` dibuat, Laravel membuat tag component ini tersedia:

```blade
<x-error-message />
```

Nama file `error-message.blade.php` berubah menjadi tag `<x-error-message />`.

Untuk memberi tahu field mana yang diperiksa, kirim prop `field`:

```blade
<x-error-message field="name" />
<x-error-message field="price" />
<x-error-message field="stock" />
<x-error-message field="description" />
<x-error-message field="category_id" />
<x-error-message field="image" />
```

Baca satu contoh ini:

```blade
<x-error-message field="price" />
```

- `<x-error-message` memanggil file `resources/views/components/error-message.blade.php`.
- `field="price"` mengirim teks `price` ke variabel `$field` pada component.
- `@error($field)` menjadi `@error('price')`.
- Jika harga gagal validasi, `$message` ditampilkan.
- Jika harga valid, component tidak menghasilkan teks apa pun.

## Coba pada satu field dahulu

Jangan langsung mengganti semua field. Mulai dari harga di form create product.

Sebelumnya:

```blade
<label for="price">Harga</label>
<input id="price" type="number" name="price" value="{{ old('price') }}" min="0">

@error('price')
    <p role="alert">{{ $message }}</p>
@enderror
```

Ganti **hanya blok `@error`** dengan component:

```blade
<label for="price">Harga</label>
<input id="price" type="number" name="price" value="{{ old('price') }}" min="0">

<x-error-message field="price" />
```

Jangan mengubah `name="price"`, `old('price')`, rule validasi, route, atau method controller. Component ini hanya mengganti cara menulis tampilan pesan error.

## Apa manfaatnya?

| Tanpa component | Dengan component |
| --- | --- |
| Menulis `@error`, `<p>`, dan `@enderror` berulang kali | Menulis satu tag pendek `<x-error-message ... />` |
| Perubahan tampilan perlu dilakukan di banyak form | Ubah satu file component, semua pemakaian ikut berubah |
| Berisiko ada field yang lupa memakai `role="alert"` | Aksesibilitas dasar lebih konsisten |
| Tampilan pesan dapat berbeda-beda tanpa sengaja | Pesan error memiliki pola yang sama |

Misalnya nanti ingin memberi class CSS pada semua pesan error, cukup ubah file component:

```blade
@props(['field'])

@error($field)
    <p class="error-message" role="alert">{{ $message }}</p>
@enderror
```

Kita tidak perlu mencari dan mengubah setiap form product satu per satu.

## Uji component

1. Buat file `resources/views/components/error-message.blade.php` dengan kode component di atas.
2. Di `resources/views/products/create.blade.php`, ganti blok error untuk harga dengan `<x-error-message field="price" />`.
3. Isi harga `-5000`, lalu isi semua field wajib lain dengan data valid.
4. Kirim form.
5. Pastikan pesan error harga tetap muncul tepat di bawah input harga.
6. Masukkan harga `0` atau lebih besar, lalu kirim ulang form.
7. Pastikan pesan harga hilang. Jika semua field valid, product dapat disimpan dan flash message success dari materi 14 tetap muncul.

Pada tahap berikutnya, kita akan memakai component yang sama pada seluruh field form create dan edit. Tidak perlu membuat component baru untuk setiap field.

---

**Apakah kamu ingin lanjut ke langkah berikutnya: memakai component error message pada form create dan edit product?**
