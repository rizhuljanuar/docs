# Tahap 6 — Tombol Hapus dengan Form dan Konfirmasi

> Materi: 13. Komponen Button Blade
> Level: Pemula — Fondasi Laravel, MVC, CRUD
> Fokus tahap ini: **menangani kasus khusus tombol Hapus** yang butuh `<form>`, `@method('DELETE')`, dan konfirmasi pengguna.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. Menjelaskan **kenapa** tombol Hapus berbeda dari tombol lain.
2. Memahami **kenapa** Hapus butuh `<form>` dengan method POST + `@method('DELETE')`.
3. Menambahkan **konfirmasi JavaScript** supaya pengguna tidak sengaja menghapus.
4. Membungkus `<x-button>` di dalam form dengan benar.
5. Membuat **komponen khusus** `<x-button-delete>` untuk tombol Hapus.
6. Menghindari kesalahan umum saat menangani tombol Hapus.

Tombol Hapus adalah tombol **paling rumit** di CRUD. Kalau kamu paham tahap ini, kamu sudah melewati bagian tersulit dari komponen Button.

---

## 2. Analogi: Tombol Alarm Kebakaran vs Tombol Pintu

Bayangkan kamu di gedung perkantoran. Ada dua jenis "tombol":

### Tombol Pintu Biasa

- Kamu **tinggal tekan** atau dorong.
- Pintu terbuka, kamu masuk.
- **Tidak ada konsekuensi besar** kalau salah tekan.

### Tombol Alarm Kebakaran

- Kamu **tidak bisa asal tekan**.
- Begitu ditekan, sprinkler menyala, pemadam datang, semua orang dievakuasi.
- **Konsekuensinya besar** dan tidak bisa dibatalkan.

Itulah kenapa tombol alarm kebakaran biasanya:

1. **Ditutup kaca** (harus pecahkan kaca dulu, supaya tidak sengaja).
2. **Bentuknya berbeda** (merah menyala, tulisan "PUSH").
3. **Sering ada konfirmasi** tertulis: "Hanya untuk darurat".

### Hubungannya dengan tombol di aplikasi

| Jenis Tombol     | Analogi                | Sifat                              |
| ---------------- | ---------------------- | ---------------------------------- |
| Tombol Simpan    | Tombol pintu biasa     | Aman, bisa diulang                 |
| Tombol Edit      | Tombol pintu biasa     | Aman, bisa diulang                 |
| Tombol Detail    | Tombol pintu biasa     | Aman, hanya melihat                |
| Tombol Batal     | Tombol pintu biasa     | Aman, tidak ada konsekuensi        |
| **Tombol Hapus** | **Tombol alarm**       | **Berbahaya, tidak bisa dibatalkan!** |

Tombol Hapus butuh **perlakuan khusus**:

1. **Visual yang berbeda** (warna merah / variant `danger`) supaya pengguna sadar ini bahaya.
2. **Konfirmasi** sebelum benar-benar menghapus, supaya tidak klik sengaja.
3. **Form dengan method DELETE**, supaya Laravel tahu ini aksi hapus (bukan tambah/update).

---

## 3. Kenapa Tombol Hapus Berbeda dari Tombol Lain?

Mari kita bandingkan **semua tombol** di CRUD Produk dan cara kerjanya:

| Tombol       | Cara Kerja                                     | Butuh Form? | Method HTTP |
| ------------ | ---------------------------------------------- | ----------- | ----------- |
| Tambah       | Link ke form tambah                            | Tidak       | GET         |
| Detail       | Link ke halaman detail                         | Tidak       | GET         |
| Edit         | Link ke form edit                              | Tidak       | GET         |
| Simpan       | Kirim data baru ke server                      | Ya          | POST        |
| Update       | Kirim data yang sudah diubah ke server         | Ya          | PUT/PATCH   |
| Batal        | Link kembali ke daftar                         | Tidak       | GET         |
| Kembali      | Link kembali ke daftar                         | Tidak       | GET         |
| **Hapus**    | **Kirim perintah hapus data ke server**        | **Ya**      | **DELETE**  |

Tombol Hapus **butuh form**, sama seperti tombol Simpan dan Update. Tapi bedanya:

- Tombol Simpan dan Update **mengirim data** (nama, harga, deskripsi).
- Tombol Hapus **tidak mengirim data**, hanya mengirim **perintah "hapus data ini"**.

Itulah kenapa tombol Hapus butuh form, tapi form-nya **sederhana** (hanya `@csrf` dan `@method('DELETE')`, tanpa input apapun).

---

## 4. Kenapa Hapus Pakai Method DELETE, Bukan GET?

Ini pertanyaan penting. Kenapa tidak pakai link biasa saja?

### Cara salah (link biasa GET)

```html
<a href="/products/1/delete">Hapus</a>
```

**Kenapa ini berbahaya?**

Karena link GET bisa **dieksekusi tanpa klik langsung**. Contoh:

1. **Bot crawler** (Google, Bing) bisa "mengikuti" link ini tanpa sengaja, dan **menghapus data**.
2. **Browser prefetch** bisa preload link ini sebagai "optimasi", dan **menghapus data**.
3. **Serangan CSRF** bisa membujuk pengguna mengklik link ini, dan **menghapus data**.

### Cara benar (form dengan POST + DELETE)

```html
<form action="/products/1" method="POST">
    <input type="hidden" name="_method" value="DELETE">
    <input type="hidden" name="_token" value="csrf-token">
    <button type="submit">Hapus</button>
</form>
```

**Kenapa ini aman?**

1. **Bot crawler tidak mengisi form**, jadi tidak akan sengaja menghapus.
2. **Browser prefetch tidak mengirim POST**, jadi tidak akan sengaja menghapus.
3. **@csrf token** melindungi dari serangan CSRF (tidak bisa dipalsukan dari situs lain).

> **Pesan mentor:**
> Aturan keamanan web: **GET untuk membaca, POST/PUT/DELETE untuk mengubah/menghapus**. Jangan pernah pakai GET untuk aksi yang mengubah atau menghapus data. Ini bukan saran, ini **aturan mutlak**.

---

## 5. Struktur Form Hapus (Dasar)

Sebelum kita pakai komponen `<x-button>`, mari pahami dulu **struktur form Hapus** dalam HTML biasa.

### 5.1 Form Hapus sederhana

```html
<form action="/products/1" method="POST">
    @csrf
    @method('DELETE')
    <button type="submit">Hapus Produk</button>
</form>
```

### 5.2 Penjelasan baris per baris

#### `<form action="/products/1" method="POST">`

- `action="/products/1"` → tujuan form adalah URL produk dengan ID 1.
- `method="POST"` → form mengirim dengan method POST.

> Tunggu, kenapa method POST padahal kita mau DELETE?
>
> Karena **HTML hanya mendukung GET dan POST**. Browser tidak bisa mengirim PUT, PATCH, atau DELETE langsung. Laravel (dan banyak framework lain) mengakali ini dengan **field tersembunyi** `_method`.

#### `@csrf`

Directive Laravel untuk menambahkan **token CSRF** sebagai field tersembunyi. Token ini mencegah serangan CSRF (Cross-Site Request Forgery).

Tanpa `@csrf`, Laravel akan menolak form dengan error `419 Page Expired`.

#### `@method('DELETE')`

Directive Laravel untuk menambahkan field tersembunyi `_method=DELETE`. Ini memberitahu Laravel: **"Treat form ini seolah-olah method-nya DELETE, walaupun sebenarnya POST."**

#### `<button type="submit">Hapus Produk</button>`

Tombol dengan `type="submit"` akan **mengirim form** saat diklik.

---

## 6. Tombol Hapus dengan Konfirmasi JavaScript

Tombol Hapus itu **berbahaya**. Klik sengaja = data hilang permanen. Itulah kenapa kita butuh **konfirmasi** sebelum benar-benar menghapus.

### 6.1 Cara paling sederhana: `onclick="return confirm(...)"`

Tambahkan atribut `onclick` di tombol:

```html
<form action="/products/1" method="POST">
    @csrf
    @method('DELETE')
    <button type="submit"
            onclick="return confirm('Yakin ingin menghapus produk ini?')">
        Hapus Produk
    </button>
</form>
```

### 6.2 Cara kerja `confirm(...)`

`confirm(...)` adalah fungsi JavaScript bawaan browser yang **menampilkan dialog konfirmasi** dengan tombol OK dan Cancel.

```
┌─────────────────────────────────────────────┐
│  Yakin ingin menghapus produk ini?          │
│                                             │
│              [ Cancel ]    [ OK ]           │
└─────────────────────────────────────────────┘
```

- Kalau pengguna klik **OK** → `confirm(...)` mengembalikan `true` → form dikirim.
- Kalau pengguna klik **Cancel** → `confirm(...)` mengembalikan `false` → form **tidak** dikirim.

### 6.3 Kenapa pakai `return confirm(...)`?

`onclick` menjalankan kode JavaScript saat tombol diklik. Kalau `onclick` **mengembalikan `false`**, aksi default (mengirim form) **dibatalkan**. Kalau mengembalikan `true`, aksi default **dilanjutkan**.

- `onclick="return confirm(...)"` artinya: "Tampilkan dialog. Kalau pengguna klik OK, kembalikan true (lanjut kirim form). Kalau klik Cancel, kembalikan false (batal kirim form)."

> **Pesan mentor:**
> Jangan lupa kata kunci **`return`**. Tanpa `return`, `confirm(...)` akan tampil, tapi form tetap dikirim baik pengguna klik OK maupun Cancel.

---

## 7. Bungkus `<x-button>` di Dalam Form Hapus

Sekarang, mari kita gabungkan **form Hapus** dengan **komponen `<x-button>`** yang sudah kita buat.

### 7.1 Pola dasar

```blade
<form action="{{ route('produk.destroy', $produk->id) }}" method="POST">
    @csrf
    @method('DELETE')
    <x-button variant="danger" type="submit"
              onclick="return confirm('Yakin ingin menghapus produk ini?')">
        Hapus Produk
    </x-button>
</form>
```

### 7.2 Penjelasan pemanggilan komponen

#### `variant="danger"`

Tombol Hapus pakai variant `danger` (merah) untuk menandakan bahaya.

#### `type="submit"`

Supaya tombol **mengirim form** saat diklik.

#### `onclick="return confirm(...)"`

Konfirmasi sebelum benar-benar mengirim form.

### 7.3 Yang terjadi di dalam komponen

Saat Blade memproses `<x-button variant="danger" type="submit" onclick="...">`, komponen akan:

1. Set `$variant = "danger"` → class jadi `btn btn-danger`.
2. Atribut `type="submit"` dan `onclick="..."` masuk ke `$attributes`.
3. `$attributes->merge([...])` menggabungkan semua atribut.
4. Hasil HTML:

```html
<button type="submit"
        onclick="return confirm('Yakin ingin menghapus produk ini?')"
        class="btn btn-danger">
    🗑️ Hapus Produk
</button>
```

Semua atribut tambahan (`type`, `onclick`) **digabung otomatis** berkat `$attributes->merge(...)`. Inilah keuntungan pakai `merge` yang kita pelajari di tahap 2.

---

## 8. Pemakaian di Halaman `produk/index.blade.php`

Di halaman daftar produk, tombol Hapus muncul **untuk setiap produk** di dalam `@foreach`.

### 8.1 Kode lengkap (setelah refactor tahap 5)

```blade
@foreach ($produk as $item)
    <tr>
        <td>{{ $item->nama }}</td>
        <td>Rp {{ number_format($item->harga, 0, ',', '.') }}</td>
        <td>
            <x-button variant="info" href="{{ route('produk.show', $item->id) }}">
                Detail
            </x-button>

            <x-button variant="warning" href="{{ route('produk.edit', $item->id) }}">
                Edit
            </x-button>

            <form action="{{ route('produk.destroy', $item->id) }}"
                  method="POST"
                  style="display:inline">
                @csrf
                @method('DELETE')
                <x-button variant="danger" type="submit"
                          onclick="return confirm('Yakin ingin menghapus {{ $item->nama }}?')">
                    Hapus
                </x-button>
            </form>
        </td>
    </tr>
@endforeach
```

### 8.2 Perhatikan detail pentingnya

#### `style="display:inline"` di `<form>`

Form diberi `display:inline` supaya **tidak membuat baris baru** di tabel. Tanpa ini, form akan memakan ruang vertikal dan merusak layout tabel.

#### Nama produk di pesan konfirmasi

```blade
onclick="return confirm('Yakin ingin menghapus {{ $item->nama }}?')"
```

Nama produk disisipkan ke pesan konfirmasi supaya **lebih jelas** produk mana yang akan dihapus. Contoh: "Yakin ingin menghapus Sepatu Nike?"

#### Form berada **di dalam** `<td>`

Form diletakkan di dalam kolom "Aksi" (`<td>`) tabel, **berdampingan** dengan tombol Detail dan Edit.

---

## 9. Masalah: Form Hapus Terlalu Bertele-tele

Lihat kode form Hapus di atas. Cukup **panjang** ya? Tiap kali kita mau bikin tombol Hapus, kita harus menulis:

```blade
<form action="..." method="POST" style="display:inline">
    @csrf
    @method('DELETE')
    <x-button variant="danger" type="submit" onclick="...">
        Hapus
    </x-button>
</form>
```

Itu **8 baris kode** untuk satu tombol Hapus. Bayangkan kalau di satu halaman ada **5 tombol Hapus** (misal: di tabel daftar produk). Beruntun **40 baris** hanya untuk tombol Hapus.

### Solusi: Bikin Komponen Khusus `<x-button-delete>`

Kita bisa bikin **komponen terpisah** khusus untuk tombol Hapus. Komponen ini sudah **mengurus form** dan **konfirmasi** secara otomatis. Pemanggil cukup sebut **tujuan URL** dan **teksnya**.

---

## 10. Membuat Komponen `<x-button-delete>`

### 10.1 Buat file baru

```
resources/views/components/button-delete.blade.php
```

### 10.2 Isi komponen

```blade
@props([
    'action' => null,
    'confirm' => 'Yakin ingin menghapus data ini?',
])

<form action="{{ $action }}" method="POST" style="display:inline">
    @csrf
    @method('DELETE')
    <x-button
        variant="danger"
        type="submit"
        onclick="return confirm('{{ $confirm }}')">
        {{ $slot }}
    </x-button>
</form>
```

### 10.3 Penjelasan baris per baris

#### `@props(['action' => null, 'confirm' => '...'])`

Komponen ini menerima **2 props**:

- `action` → URL tujuan form (misal: `route('produk.destroy', $produk->id)`).
- `confirm` → Pesan konfirmasi yang akan tampil di dialog (default: "Yakin ingin menghapus data ini?").

#### `<form action="{{ $action }}" method="POST" style="display:inline">`

Form dengan:

- `action` diisi dari props `action`.
- `method="POST"` (tetap).
- `style="display:inline"` supaya tidak buat baris baru.

#### `@csrf` dan `@method('DELETE')`

Token CSRF dan field `_method=DELETE`, seperti yang sudah kita bahas di bagian 5.

#### `<x-button variant="danger" type="submit" onclick="...">`

Pemanggilan komponen `<x-button>` (komponen kita sendiri!) dengan:

- `variant="danger"` → merah.
- `type="submit"` → mengirim form.
- `onclick="return confirm('{{ $confirm }}')"` → konfirmasi dengan pesan dari props `confirm`.

#### `{{ $slot }}`

Teks tombol dari pemanggil (misal: "Hapus" atau "Hapus Produk").

### 10.4 Cara pakai komponen `<x-button-delete>`

Sekarang, di halaman `index.blade.php`, kita bisa **mengganti** kode form Hapus yang panjang dengan:

**Sebelum (8 baris):**

```blade
<form action="{{ route('produk.destroy', $item->id) }}"
      method="POST"
      style="display:inline">
    @csrf
    @method('DELETE')
    <x-button variant="danger" type="submit"
              onclick="return confirm('Yakin ingin menghapus {{ $item->nama }}?')">
        Hapus
    </x-button>
</form>
```

**Sesudah (3 baris):**

```blade
<x-button-delete
    action="{{ route('produk.destroy', $item->id) }}"
    confirm="Yakin ingin menghapus {{ $item->nama }}?">
    Hapus
</x-button-delete>
```

**Lebih singkat, lebih jelas, lebih mudah dibaca.**

### 10.5 Manfaat komponen `<x-button-delete>`

| Aspek                | Tanpa Komponen                         | Pakai Komponen                          |
| -------------------- | -------------------------------------- | --------------------------------------- |
| Jumlah baris         | 8 baris per tombol Hapus               | 3 baris per tombol Hapus                |
| `@csrf`              | Harus ditulis manual tiap kali         | Otomatis dari komponen                  |
| `@method('DELETE')`  | Harus ditulis manual tiap kali         | Otomatis dari komponen                  |
| `style="display:inline"` | Harus diingat tiap kali             | Otomatis dari komponen                  |
| Pesan konfirmasi     | Harus nulis `onclick` tiap kali        | Cukup kirim props `confirm`             |
| Konsistensi          | Rawan beda-beda                        | Semua Hapus pakai pola sama             |

---

## 11. Pemakaian di Semua Halaman

Mari kita lihat bagaimana komponen `<x-button-delete>` dipakai di **semua halaman** yang punya tombol Hapus.

### 11.1 Di `produk/index.blade.php` (Daftar Produk)

```blade
@foreach ($produk as $item)
    <tr>
        <td>{{ $item->nama }}</td>
        <td>Rp {{ number_format($item->harga, 0, ',', '.') }}</td>
        <td>
            <x-button variant="info" href="{{ route('produk.show', $item->id) }}">
                Detail
            </x-button>

            <x-button variant="warning" href="{{ route('produk.edit', $item->id) }}">
                Edit
            </x-button>

            <x-button-delete
                action="{{ route('produk.destroy', $item->id) }}"
                confirm="Yakin ingin menghapus {{ $item->nama }}?">
                Hapus
            </x-button-delete>
        </td>
    </tr>
@endforeach
```

### 11.2 Di `produk/show.blade.php` (Detail Produk)

```blade
<div class="mt-3">
    <x-button variant="warning" href="{{ route('produk.edit', $produk->id) }}">
        Edit Produk
    </x-button>

    <x-button-delete
        action="{{ route('produk.destroy', $produk->id) }}"
        confirm="Yakin ingin menghapus {{ $produk->nama }}?">
        Hapus Produk
    </x-button-delete>

    <x-button variant="secondary" href="{{ route('produk.index') }}">
        Kembali
    </x-button>
</div>
```

### 11.3 Perhatikan konsistensi

Di **kedua halaman**, tombol Hapus sekarang pakai **pola yang sama persis**:

```blade
<x-button-delete
    action="..."
    confirm="...">
    Hapus ...
</x-button-delete>
```

Tidak perlu lagi **mengingat** struktur form, `@csrf`, `@method`, atau `style="display:inline"`. Semua sudah diurus komponen.

---

## 12. Diagram: Cara Kerja Komponen `<x-button-delete>`

```
Pemanggil:
┌──────────────────────────────────────────┐
│ <x-button-delete                         │
│     action="/products/1"                 │  ← props: action
│     confirm="Yakin hapus Sepatu Nike?">  │  ← props: confirm
│     Hapus                                │  ← slot: teks tombol
│ </x-button-delete>                       │
└──────────────────┬───────────────────────┘
                   │
                   ▼
Komponen: components/button-delete.blade.php
┌──────────────────────────────────────────┐
│ <form action="/products/1" ...>          │  ← form otomatis
│   @csrf                                  │  ← csrf otomatis
│   @method('DELETE')                      │  ← DELETE otomatis
│   <x-button variant="danger" ...>        │  ← pakai <x-button>
│     {{ $slot }}                          │  ← teks dari pemanggil
│   </x-button>                            │
│ </form>                                  │
└──────────────────┬───────────────────────┘
                   │
                   ▼
Output HTML:
┌──────────────────────────────────────────┐
│ <form action="/products/1" method="POST" │
│       style="display:inline">            │
│   <input type="hidden" name="_token" ...>│
│   <input type="hidden" name="_method"    │
│          value="DELETE">                 │
│   <button type="submit"                  │
│           class="btn btn-danger"         │
│           onclick="return confirm(...)"> │
│     🗑️ Hapus                             │
│   </button>                              │
│ </form>                                  │
└──────────────────────────────────────────┘
```

---

## 13. Troubleshooting

### Error 1: `MethodNotAllowedHttpException`

**Penyebab:**

- Lupa `@method('DELETE')` di form.
- Route untuk `produk.destroy` belum didefinisikan.

**Solusi:**

1. Pastikan form punya `@method('DELETE')`.
2. Cek `routes/web.php` apakah ada `Route::delete('products/{id}', ...)` atau `Route::resource('produk', ...)`.

### Error 2: `419 Page Expired`

**Penyebab:**

- Lupa `@csrf` di form.
- Session expired (lama tidak aktivitas).

**Solusi:**

1. Pastikan form punya `@csrf`.
2. Refresh halaman (untuk kasus session expired).

### Error 3: Data terhapus tanpa konfirmasi

**Penyebab:**

- `onclick` tidak pakai `return`: `onclick="confirm(...)"` (salah).
- Atribut `onclick` tidak masuk ke tombol.

**Solusi:**

1. Pastikan pakai `return`: `onclick="return confirm(...)"`.
2. Kalau pakai komponen `<x-button-delete>`, ini sudah otomatis ditangani.

### Error 4: Dialog konfirmasi tidak muncul

**Penyebab:**

- JavaScript dimatikan di browser.
- Atribut `onclick` salah ketik (mis: `onclik`).
- Komponen tidak menerima props `confirm` dengan benar.

**Solusi:**

1. Cek apakah JavaScript aktif di browser.
2. Periksa ejaan `onclick`.
3. Pastikan props `confirm` dikirim dengan benar: `confirm="Pesan..."`.

### Error 5: Form muncul sebagai blok, merusak layout tabel

**Penyebab:**

- Lupa `style="display:inline"` di `<form>`.
- Kalau pakai komponen `<x-button-delete>`, ini sudah otomatis.

**Solusi:**

Tambahkan `style="display:inline"` di form, atau pakai komponen `<x-button-delete>`.

---

## 14. Latihan Mandiri

**Latihan E:**

Buat komponen `<x-button-delete>` dengan **fitur tambahan**: props opsional `ikon` (default: `🗑️`) yang akan tampil di depan teks tombol.

**Contoh pemakaian yang diharapkan:**

```blade
<!-- Pakai default -->
<x-button-delete action="...">
    Hapus
</x-button-delete>
<!-- Tampil: 🗑️ Hapus -->

<!-- Ganti ikon -->
<x-button-delete action="..." ikon="⚠️">
    Hapus Permanen
</x-button-delete>
<!-- Tampil: ⚠️ Hapus Permanen -->
```

<details>
<summary><strong>Lihat jawaban Latihan E</strong></summary>

**File `components/button-delete.blade.php`:**

```blade
@props([
    'action' => null,
    'confirm' => 'Yakin ingin menghapus data ini?',
    'ikon' => '🗑️',
])

<form action="{{ $action }}" method="POST" style="display:inline">
    @csrf
    @method('DELETE')
    <x-button
        variant="danger"
        type="submit"
        onclick="return confirm('{{ $confirm }}')">
        {{ $ikon }} {{ $slot }}
    </x-button>
</form>
```

**Perubahan:**

1. Tambah props `ikon` dengan default `🗑️`.
2. Di slot, tampilkan `{{ $ikon }}` sebelum `{{ $slot }}`.

**Catatan:**

Kalau kamu sudah pakai fitur ikon otomatis di `<x-button>` (dari tahap 4), maka variant `danger` sudah otomatis dapat ikon `🗑️`. Kamu bisa hapus `{{ $ikon }}` dari komponen ini dan andalkan komponen `<x-button>` saja. Tapi kalau mau **kontrol penuh** di komponen `<x-button-delete>`, cara di atas tetap berlaku.

</details>

---

## 15. Istilah Kunci Tahap Ini

| Istilah            | Arti sederhana                                        |
| ------------------ | ----------------------------------------------------- |
| **Method DELETE**  | Method HTTP untuk menghapus data di server            |
| **`@method('DELETE')`** | Directive untuk "memalsukan" method DELETE via POST |
| **`@csrf`**        | Directive untuk token keamanan CSRF                   |
| **`confirm(...)`** | Fungsi JavaScript untuk dialog konfirmasi             |
| **`return confirm(...)`** | Pola untuk membatalkan submit jika pengguna klik Cancel |
| **`display:inline`** | Style CSS supaya elemen tidak buat baris baru       |
| **Komponen turunan** | Komponen yang memanggil komponen lain di dalamnya  |

---

## 16. Ringkasan Tahap 6

1. **Tombol Hapus berbeda** dari tombol lain karena:
   - Berbahaya (data hilang permanen).
   - Butuh **form** dengan method DELETE.
   - Butuh **konfirmasi** supaya tidak klik sengaja.
2. **Kenapa pakai form, bukan link GET:** GET bisa dieksekusi bot/prefetch. POST + DELETE lebih aman.
3. **Struktur form Hapus** membutuhkan: `@csrf`, `@method('DELETE')`, dan tombol `type="submit"`.
4. **Konfirmasi** dengan `onclick="return confirm(...)"` menampilkan dialog OK/Cancel.
5. **Komponen `<x-button-delete>`** membungkus semua yang ribet jadi 3 baris:

   ```blade
   <x-button-delete action="..." confirm="...">
       Hapus
   </x-button-delete>
   ```

6. **Komponen bisa panggil komponen lain** - `<x-button-delete>` memanggil `<x-button>` di dalamnya.
7. **Konsistensi tombol Hapus** terjaga di semua halaman karena pakai komponen yang sama.

---

## 17. Cek Pemahaman

1. Kenapa tombol Hapus tidak bisa pakai link biasa (`<a href="...">`)?
2. Apa fungsi `@method('DELETE')` di form?
3. Kenapa `onclick` harus pakai `return confirm(...)`, bukan sekadar `confirm(...)`?
4. Apa fungsi `style="display:inline"` di form Hapus?
5. Sebutkan 2 props yang diterima komponen `<x-button-delete>`.
6. Apa keuntungan membuat komponen `<x-button-delete>` terpisah dari `<x-button>`?

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

1. Karena link GET bisa dieksekusi **tanpa klik langsung** (oleh bot, prefetch, serangan CSRF). Hapus data harus pakai POST + DELETE supaya aman.
2. Untuk "memberitahu" Laravel bahwa form ini seolah-olah dikirim dengan method DELETE, walaupun sebenarnya POST (karena HTML hanya mendukung GET/POST).
3. Tanpa `return`, `confirm(...)` tampil tapi form **tetap dikirim** baik pengguna klik OK maupun Cancel. Dengan `return`, hasil `confirm` menentukan apakah form dikirim atau dibatalkan.
4. Supaya form **tidak membuat baris baru** di layout (misal: di tabel daftar produk), sehingga tombol Hapus sejajar dengan tombol lain di kolom Aksi.
5. `action` (URL tujuan form) dan `confirm` (pesan konfirmasi, opsional dengan default).
6. Karena tombol Hapus **butuh form, @csrf, @method, dan konfirmasi** yang tidak dimiliki tombol biasa. Komponen terpisah menyembunyikan kompleksitas ini dan membuat pemanggil jadi lebih bersih (3 baris vs 8 baris).

</details>

---

## 18. Apakah Kamu Ingin Lanjut?

Di tahap 6 ini kamu sudah **menaklukkan kasus tersulit**: tombol Hapus dengan form dan konfirmasi. Kamu juga sudah bikin **komponen turunan** `<x-button-delete>` yang memanggil komponen lain.

Tapi masih ada **satu masalah** yang belum diselesaikan sejak tahap 5: tombol dengan `href` (seperti Batal, Kembali, Edit, Detail) saat ini **tidak berfungsi sebagai link** karena komponen kita menghasilkan `<button>`, bukan `<a>`.

Di tahap berikutnya, kita akan **selesaikan masalah ini** dan menambahkan variant tambahan.

> ### Pertanyaan:
>
> Apakah kamu ingin lanjut ke langkah berikutnya: **Variant Tambahan dan Tipe Link**?
>
> Di tahap 7 kita akan bahas:
>
> 1. Memodifikasi komponen supaya **otomatis jadi `<a>`** kalau ada `href`
> 2. Menambahkan variant `info` untuk tombol Detail
> 3. Membuat komponen `<x-button-link>` khusus untuk link
> 4. Menangani tombol sebagai **button** vs **link** dengan benar
> 5. Refactor final semua tombol dengan tipe yang tepat
>
> Ketik **"lanjut"** untuk ke tahap 7,
> atau tanyakan jika ada bagian tahap 6 yang masih bingung.

---

> **Daftar Tahap (13. Komponen Button Blade):**
> - [x] Tahap 1 — Kenapa Tombol Harus Konsisten
> - [x] Tahap 2 — Membuat File Komponen `<x-button>` Sederhana
> - [x] Tahap 3 — Memahami Props dan Variant
> - [x] Tahap 4 — Slot dan Isi Tombol
> - [x] Tahap 5 — Menerapkan `<x-button>` di Halaman Produk
> - [x] Tahap 6 — Tombol Hapus dengan Form dan Konfirmasi (kamu di sini)
> - [ ] Tahap 7 — Variant Tambahan dan Tipe Link
> - [ ] Tahap 8 — Ringkasan dan Best Practice
