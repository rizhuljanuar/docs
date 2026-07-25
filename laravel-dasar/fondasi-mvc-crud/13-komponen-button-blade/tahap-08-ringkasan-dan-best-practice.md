# Tahap 8 — Ringkasan dan Best Practice

> Materi: 13. Komponen Button Blade
> Level: Pemula — Fondasi Laravel, MVC, CRUD
> Fokus tahap ini: **merangkum semua konsep** dari tahap 1-7 dan membahas **best practice** untuk dikembangkan ke depan.

---

## 1. Tujuan Belajar Tahap Ini

Setelah tahap ini, kamu harus bisa:

1. **Merangkum** semua konsep komponen Button dari tahap 1 sampai 7.
2. Menyebutkan **best practice** membuat komponen Blade.
3. Menghindari **kesalahan umum** yang sering dilakukan pemula.
4. Memahami **struktur final** semua file yang dibuat di materi ini.
5. Mengetahui **ide pengembangan** komponen ke depan (size, outline, disabled).
6. Merasa **percaya diri** untuk membuat komponen UI sendiri.

Ini adalah **tahap terakhir**. Setelah ini, kamu sudah lulus dari materi Komponen Button Blade.

---

## 2. Perjalanan Kita: Dari Masalah ke Solusi

Mari kita lihat **perjalanan lengkap** kita selama 8 tahap:

### Tahap 1: Kenapa Tombol Harus Konsisten?

**Masalah:** Style tombol berbeda-beda di setiap halaman.

```
Halaman index  → class="btn btn-warning btn-sm px-3 py-1 rounded"
Halaman create → class="btn btn-primary"
Halaman edit   → class="btn btn-success px-4"
Halaman show   → class="btn btn-info"
```

**Kesadaran:** Ini masalah karena tidak rapi, sulit dirawat, mudah typo.

### Tahap 2: Membuat Komponen `<x-button>` Sederhana

**Solusi mulai:** Bikin file `components/button.blade.php`.

```blade
@props(['variant' => 'primary'])

<button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
    {{ $slot }}
</button>
```

**Hasil:** Bisa dipanggil dengan `<x-button variant="primary">Simpan</x-button>`.

### Tahap 3: Memahami Props dan Variant

**Pemahaman:** Variant bukan sekadar warna, tapi **peran** tombol.

| Variant     | Peran           | Contoh       |
| ----------- | --------------- | ------------ |
| primary     | Aksi utama      | Simpan       |
| secondary   | Netral          | Batal        |
| success     | Konfirmasi      | Selesai      |
| danger      | Berbahaya       | Hapus        |
| warning     | Perlu perhatian | Edit         |
| info        | Informatif      | Detail       |

### Tahap 4: Slot dan Isi Tombol

**Kekuatan:** Slot memungkinkan isi tombol **fleksibel** (teks, ikon, HTML).

```blade
<x-button variant="primary">
    <i class="fas fa-save"></i> Simpan Produk
</x-button>
```

### Tahap 5: Refactor Semua Halaman

**Praktik:** Ganti semua tombol manual di 4 halaman CRUD dengan `<x-button>`.

**Hasil:** Konsistensi tercapai di seluruh aplikasi.

### Tahap 6: Tombol Hapus dengan Form dan Konfirmasi

**Kasus khusus:** Tombol Hapus butuh form, `@method('DELETE')`, dan konfirmasi.

**Solusi:** Bikin komponen `<x-button-delete>` yang membungkus semua kerumitan.

### Tahap 7: Variant Tambahan dan Tipe Link

**Penyempurnaan:** Komponen otomatis jadi `<a>` kalau ada `href`.

```blade
<!-- Otomatis jadi <a> -->
<x-button variant="secondary" href="/products">Batal</x-button>

<!-- Otomatis jadi <button> -->
<x-button variant="primary" type="submit">Simpan</x-button>
```

### Hasil Akhir

Dari **tombol berbeda-beda** di setiap halaman, sekarang menjadi **satu komponen konsisten** yang dipakai di seluruh aplikasi.

---

## 3. Struktur Final Semua File

Berikut adalah **daftar lengkap** semua file yang dibuat dan dimodifikasi di materi ini:

### 3.1 File Komponen (Baru)

```
resources/views/components/
├── button.blade.php           ← Komponen utama (otomatis <a> atau <button>)
└── button-delete.blade.php    ← Komponen khusus tombol Hapus
```

### 3.2 File Halaman (Dimodifikasi)

```
resources/views/produk/
├── index.blade.php            ← Refactor: pakai <x-button> dan <x-button-delete>
├── create.blade.php           ← Refactor: pakai <x-button>
├── edit.blade.php             ← Refactor: pakai <x-button>
└── show.blade.php             ← Refactor: pakai <x-button> dan <x-button-delete>
```

### 3.3 Kode Final Komponen `button.blade.php`

```blade
@props([
    'variant' => 'primary',
    'href' => null,
])

@php
    $variantYangDiterima = [
        'primary', 'secondary', 'success',
        'danger', 'warning', 'info', 'light',
    ];

    if (!in_array($variant, $variantYangDiterima)) {
        throw new \InvalidArgumentException(
            "Variant '{$variant}' tidak dikenal. Variant yang valid: " .
            implode(', ', $variantYangDiterima)
        );
    }

    $ikonPerVariant = [
        'primary'   => '💾',
        'secondary' => '↩️',
        'success'   => '✅',
        'danger'    => '🗑️',
        'warning'   => '✏️',
        'info'      => '👁️',
        'light'     => '',
    ];

    $ikon = $ikonPerVariant[$variant] ?? '';
@endphp

@if ($href)
    <a href="{{ $href }}" {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
        @if ($ikon)<span class="me-1">{{ $ikon }}</span>@endif
        {{ $slot }}
    </a>
@else
    <button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}>
        @if ($ikon)<span class="me-1">{{ $ikon }}</span>@endif
        {{ $slot }}
    </button>
@endif
```

### 3.4 Kode Final Komponen `button-delete.blade.php`

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

---

## 4. Tabel Final: Semua Tombol CRUD Produk

Ini adalah **hasil akhir** semua tombol di CRUD Produk setelah 8 tahap:

| Halaman  | Tombol         | Kode                                                       | Tag     | Variant     |
| -------- | -------------- | --------------------------------------------------------- | ------- | ----------- |
| `index`  | Tambah Produk  | `<x-button variant="primary" href="...">`                  | `<a>`   | `primary`   |
| `index`  | Detail         | `<x-button variant="info" href="...">`                     | `<a>`   | `info`      |
| `index`  | Edit           | `<x-button variant="warning" href="...">`                  | `<a>`   | `warning`   |
| `index`  | Hapus          | `<x-button-delete action="..." confirm="...">`             | `<button>` | `danger` |
| `create` | Simpan Produk  | `<x-button variant="primary" type="submit">`               | `<button>` | `primary` |
| `create` | Batal          | `<x-button variant="secondary" href="...">`                | `<a>`   | `secondary` |
| `edit`   | Update Produk  | `<x-button variant="primary" type="submit">`               | `<button>` | `primary` |
| `edit`   | Batal          | `<x-button variant="secondary" href="...">`                | `<a>`   | `secondary` |
| `show`   | Edit Produk    | `<x-button variant="warning" href="...">`                  | `<a>`   | `warning`   |
| `show`   | Hapus Produk   | `<x-button-delete action="..." confirm="...">`             | `<button>` | `danger` |
| `show`   | Kembali        | `<x-button variant="secondary" href="...">`                | `<a>`   | `secondary` |

### Pola konsisten yang tercapai

1. **Aksi utama** (Simpan, Update, Tambah) → `primary` + `<button>` atau `<a>`.
2. **Aksi mengubah** (Edit) → `warning` + `<a>`.
3. **Aksi menghapus** (Hapus) → `danger` + `<button>` via `<x-button-delete>`.
4. **Aksi informatif** (Detail) → `info` + `<a>`.
5. **Aksi netral** (Batal, Kembali) → `secondary` + `<a>`.

---

## 5. Best Practice Membuat Komponen Blade

Ini adalah **panduan** yang harus kamu ikuti saat membuat komponen Blade di masa depan.

### 5.1 Penamaan File

| Praktik Baik                              | Hindari                              |
| ----------------------------------------- | ------------------------------------ |
| Pakai **kebab-case**: `button-delete.blade.php` | `ButtonDelete.blade.php`        |
| Nama **singkat dan jelas**: `button`, `badge` | `tombol_yang_bisa_diklik`         |
| Ekstensi **`.blade.php`** selalu           | `button.html`, `button.php`         |

### 5.2 Deklarasi Props

| Praktik Baik                              | Hindari                              |
| ----------------------------------------- | ------------------------------------ |
| Deklarasi **semua props** di `@props([...])` | Biarkan props tidak terdeklarasi  |
| Beri **nilai default** untuk props opsional | Tidak ada default, props wajib terus |
| Urutan props: **wajib dulu, opsional kemudian** | Acak                         |

```blade
@props([
    'variant' => 'primary',     ← wajib dengan default
    'href' => null,             ← opsional
])
```

### 5.3 Validasi Input

| Praktik Baik                              | Hindari                              |
| ----------------------------------------- | ------------------------------------ |
| Validasi variant dengan `in_array`         | Biarkan variant asal-asalan          |
| Kasih **pesan error jelas**               | Error generik tanpa petunjuk         |
| Daftar variant disimpan di **satu tempat** | Daftar variant tersebar di kode     |

### 5.4 Penggunaan `$attributes->merge`

| Praktik Baik                              | Hindari                              |
| ----------------------------------------- | ------------------------------------ |
| Pakai `->merge(['class' => '...'])` untuk class default | Hardcode `class="..."` yang menimpa |
| Biarkan pemanggil **tambah class** sendiri | Tutup rapat, tidak bisa kustomisasi |

### 5.5 Slot untuk Konten

| Praktik Baik                              | Hindari                              |
| ----------------------------------------- | ------------------------------------ |
| Pakai `{{ $slot }}` untuk **konten bebas** (teks, ikon) | Pakai props untuk teks tombol |
| Beri **default slot** kalau perlu          | Biarkan tombol kosong                |
| Slot untuk **HTML kompleks** (ikon + teks) | Hardcode teks di komponen            |

### 5.6 Konsistensi Variant

| Praktik Baik                              | Hindari                              |
| ----------------------------------------- | ------------------------------------ |
| `primary` hanya untuk **satu aksi utama** per halaman | Semua tombol pakai `primary`  |
| `danger` hanya untuk **aksi merusak**     | Pakai `danger` untuk tombol biasa    |
| `warning` untuk **aksi mengubah**         | Campur `warning` dan `primary`       |
| `secondary` untuk **aksi netral**         | Bikin variant acak seperti `btn-light` |

---

## 6. Kesalahan Umum yang Harus Dihindari

### 6.1 Nama File Tidak Pakai kebab-case

**Salah:**

```
resources/views/components/ButtonDelete.blade.php
resources/views/components/button_delete.blade.php
```

**Benar:**

```
resources/views/components/button-delete.blade.php
```

### 6.2 Lupa `@props([...])`

**Salah:**

```blade
<button class="btn btn-{{ $variant }}">
    {{ $slot }}
</button>
```

Akan error: `Undefined variable $variant`.

**Benar:**

```blade
@props(['variant' => 'primary'])

<button class="btn btn-{{ $variant }}">
    {{ $slot }}
</button>
```

### 6.3 Pakai GET untuk Hapus

**Salah:**

```blade
<a href="/products/1/delete">Hapus</a>
```

Berbahaya: bisa dieksekusi bot, prefetch, CSRF.

**Benar:**

```blade
<form action="/products/1" method="POST">
    @csrf
    @method('DELETE')
    <x-button variant="danger" type="submit">Hapus</x-button>
</form>
```

### 6.4 Lupa `return` di `onclick`

**Salah:**

```blade
<button onclick="confirm('Yakin?')">
```

Form tetap dikirim baik OK maupun Cancel.

**Benar:**

```blade
<button onclick="return confirm('Yakin?')">
```

### 6.5 Semua Tombol Pakai `primary`

**Salah:**

```blade
<x-button variant="primary">Simpan</x-button>
<x-button variant="primary">Batal</x-button>
<x-button variant="primary">Hapus</x-button>
```

Pengguna bingung mana aksi utama.

**Benar:**

```blade
<x-button variant="primary">Simpan</x-button>
<x-button variant="secondary">Batal</x-button>
<x-button variant="danger">Hapus</x-button>
```

### 6.6 Tidak Validasi Variant

**Salah:**

```blade
<button class="btn btn-{{ $variant }}">
```

Kalau salah ketik `variant="prmary"`, tombol tampil putih polos tanpa error.

**Benar:**

```blade
@php
    if (!in_array($variant, ['primary', 'secondary', ...])) {
        throw new \InvalidArgumentException("Variant tidak dikenal");
    }
@endphp
```

### 6.7 Komponen Terlalu Rumit

**Salah:**

Membuat satu komponen yang menangani **semua jenis elemen** (button, link, input, card).

**Benar:**

Satu komponen **satu tanggung jawab**. `<x-button>` untuk tombol, `<x-card>` untuk kartu, `<x-input>` untuk input.

---

## 7. Checklist Sebelum Deploy

Sebelum aplikasi kamu live, pastikan **semua item** di bawah sudah benar:

### 7.1 Struktur File

- [ ] File komponen ada di `resources/views/components/`.
- [ ] Nama file pakai kebab-case (`button.blade.php`, bukan `Button.blade.php`).
- [ ] Tidak ada file komponen yang duplikat.

### 7.2 Props dan Variant

- [ ] Semua props dideklarasi di `@props([...])`.
- [ ] Variant divalidasi dengan `in_array`.
- [ ] Tidak ada variant asal-asalan yang lolos (seperti `variant="hijau"`).

### 7.3 Keamanan

- [ ] Tombol Hapus pakai **form dengan POST + DELETE**, bukan link GET.
- [ ] Semua form punya `@csrf`.
- [ ] Form Update punya `@method('PUT')`.
- [ ] Form Hapus punya `@method('DELETE')`.

### 7.4 Konfirmasi

- [ ] Tombol Hapus punya konfirmasi `onclick="return confirm(...)"`.
- [ ] Pesan konfirmasi **jelas** (sebut nama item yang akan dihapus).

### 7.5 Konsistensi

- [ ] Hanya **satu tombol `primary`** per halaman.
- [ ] `danger` hanya untuk aksi berbahaya.
- [ ] `warning` hanya untuk aksi mengubah.
- [ ] `secondary` untuk semua aksi netral (Batal, Kembali).

### 7.6 Tipe Tombol

- [ ] Tombol navigasi pakai `href` (jadi `<a>`).
- [ ] Tombol submit pakai `type="submit"` (jadi `<button>`).
- [ ] Tidak ada `href` di tombol submit.
- [ ] Tidak ada `type="submit"` di tombol navigasi.

### 7.7 Tampilan

- [ ] Semua tombol tampil **konsisten** di semua halaman.
- [ ] Ikon (jika dipakai) muncul **konsisten** sesuai variant.
- [ ] Tidak ada tombol dengan style acak atau class manual.

---

## 8. Ide Pengembangan Komponen ke Depan

Komponen Button yang sudah kamu buat adalah **fondasi**. Berikut adalah **ide pengembangan** yang bisa kamu tambahkan sendiri:

### 8.1 Props `size` untuk Ukuran Tombol

Tambahkan props `size` dengan pilihan `sm`, `md`, `lg`:

```blade
<x-button variant="primary" size="sm">Simpan Kecil</x-button>
<x-button variant="primary" size="lg">Simpan Besar</x-button>
```

**Implementasi:**

```blade
@props([
    'variant' => 'primary',
    'href' => null,
    'size' => 'md',
])

@php
    $classSize = [
        'sm' => 'btn-sm',
        'md' => '',
        'lg' => 'btn-lg',
    ][$size] ?? '';
@endphp

<button {{ $attributes->merge(['class' => 'btn btn-' . $variant . ' ' . $classSize]) }}>
    {{ $slot }}
</button>
```

### 8.2 Props `outline` untuk Tombol Outline

Tambahkan props `outline` untuk tombol dengan **border transparan**:

```blade
<x-button variant="primary" :outline="true">Simpan</x-button>
```

**Hasil:** tombol dengan border biru, latar transparan.

**Implementasi:**

```blade
@props([
    'variant' => 'primary',
    'outline' => false,
])

@php
    $prefix = $outline ? 'btn-outline-' : 'btn-';
@endphp

<button {{ $attributes->merge(['class' => 'btn ' . $prefix . $variant]) }}>
    {{ $slot }}
</button>
```

### 8.3 Props `disabled`

Tambahkan props `disabled` untuk tombol yang tidak bisa diklik:

```blade
<x-button variant="primary" :disabled="true">Simpan (Disabled)</x-button>
```

**Implementasi:**

```blade
@props([
    'variant' => 'primary',
    'disabled' => false,
])

<button {{ $attributes->merge(['class' => 'btn btn-' . $variant]) }}
        @if ($disabled) disabled @endif>
    {{ $slot }}
</button>
```

### 8.4 Props `icon` untuk Ikon Kustom

Sekarang ikon otomatis dari variant. Tambahkan props `icon` supaya pemanggil bisa **override**:

```blade
<x-button variant="primary" icon="fas fa-download">Download</x-button>
```

### 8.5 Komponen `<x-badge>`

Bikin komponen untuk label kecil seperti badge status "Aktif" / "Nonaktif":

```blade
<x-badge variant="success">Aktif</x-badge>
<x-badge variant="danger">Nonaktif</x-badge>
```

### 8.6 Komponen `<x-alert>`

Bikin komponen untuk pesan alert sukses/error:

```blade
<x-alert variant="success">
    Produk berhasil disimpan!
</x-alert>
```

> **Pesan mentor:**
> Jangan over-engineer di awal. Tambahkan fitur **satu per satu** saat memang dibutuhkan. Kalau belum butuh `size` atau `outline`, jangan dibuat dulu. Tunggu sampai ada kebutuhan nyata.
>
> <!-- ponytail: jangan tambah fitur sebelum dibutuhkan. YAGNI. -->

---

## 9. Rangkuman Konsep: Apa yang Sudah Kamu Pelajari?

### Konsep 1: Komponen Blade

Komponen Blade adalah **potongan kode reusable** yang dipanggil lewat `<x-nama-komponen>`.

### Konsep 2: Props

Props adalah **input terstruktur** yang diterima komponen. Dideklarasi dengan `@props([...])`.

### Konsep 3: Slot

Slot adalah **lubang untuk konten bebas** (teks, HTML, ikon). Dipakai dengan `{{ $slot }}`.

### Konsep 4: Variant

Variant adalah **peran** tombol yang menentukan tampilannya (primary, secondary, danger, dll).

### Konsep 5: `$attributes->merge`

Method untuk **menggabungkan** atribut default komponen dengan atribut dari pemanggil.

### Konsep 6: Komponen Otomatis

Komponen bisa **pintar** memilih tag HTML (`<a>` atau `<button>`) berdasarkan props yang dikirim.

### Konsep 7: Komponen Turunan

Komponen bisa **memanggil komponen lain** di dalamnya (seperti `<x-button-delete>` memanggil `<x-button>`).

### Konsep 8: Konsistensi UI

Semua tombol di seluruh aplikasi **tampil sama** karena dibuat dari **satu sumber** (komponen).

---

## 10. Analogi Final: Dari Rumah Berantakan ke Rumah Tertata

Di **awal materi** (tahap 1), aplikasi kamu seperti **rumah berantakan**:

- Setiap kamar punya **pintu berbeda** (tombol dengan class acak).
- Kalau ada yang rusak, kamu harus **cari satu-satu**.
- Tamu (pengguna) **bingung** mana pintu utama.

Setelah **8 tahap**, aplikasi kamu sekarang seperti **rumah tertata**:

- Semua kamar punya **pintu standar** (komponen `<x-button>`).
- Yang beda hanya **warna catnya** (variant).
- Kalau ada yang rusak, cukup **perbaiki di satu tempat** (file komponen).
- Tamu (pengguna) **langsung paham** mana aksi utama, mana aksi berbahaya.

### Rumus sukses konsistensi UI

```
Satu komponen + Banyak variant = Konsistensi di seluruh aplikasi
```

---

## 11. Kesimpulan Akhir

Selamat! Kamu telah menyelesaikan materi **13. Komponen Button Blade**.

### Apa yang kamu bisa lakukan sekarang:

1. Membuat komponen Blade dengan props, slot, dan variant.
2. Membuat tombol yang **konsisten** di seluruh aplikasi.
3. Menangani **kasus khusus** seperti tombol Hapus dengan form dan konfirmasi.
4. Membuat komponen yang **otomatis** memilih tag HTML yang tepat.
5. Menghindari **kesalahan umum** pemula.
6. Mengembangkan komponen lebih lanjut (size, outline, disabled, dll).

### Prinsip yang harus selalu diingat:

1. **Konsistensi di atas kreativitas.** Lebih baik semua tombol sama daripada setiap tombol unik tapi berantakan.
2. **Satu komponen, banyak pemakaian.** Tulis sekali, pakai di mana saja.
3. **Validasi input.** Jangan biarkan variant asal-asalan lolos.
4. **Keamanan primero.** Hapus pakai POST+DELETE, bukan GET.
5. **Jangan over-engineer.** Tambah fitur hanya saat dibutuhkan.
6. **Pesan mentor:**
   - Komponen Blade bukan hanya tentang tombol. Pola yang kamu pelajari di sini berlaku untuk **semua UI component**: card, badge, alert, modal, input, dll.
   - Kalau kamu bisa membuat komponen Button, kamu **bisa membuat komponen apa saja**.

---

## 12. Daftar Istilah Lengkap

| Istilah              | Arti sederhana                                           |
| -------------------- | -------------------------------------------------------- |
| **Blade**            | Template engine bawaan Laravel                           |
| **Komponen Blade**   | File Blade reusable yang dipanggil lewat `<x-nama>`      |
| **Anonymous komponen** | Komponen tanpa class PHP (cuma file Blade)             |
| **Props**            | Input terstruktur yang diterima komponen                |
| **`@props([...])`**  | Directive untuk mendeklarasi props                       |
| **Slot**             | Lubang untuk konten bebas di dalam komponen              |
| **`$slot`**          | Variabel otomatis berisi isi dari pemanggil              |
| **Default slot**     | Teks yang muncul kalau slot tidak diisi                  |
| **`$attributes`**    | Variabel otomatis berisi atribut tambahan                |
| **`->merge([...])`** | Method untuk menggabungkan atribut default dengan tambahan |
| **Variant**          | Props yang menentukan peran dan tampilan komponen        |
| **Primary**          | Variant untuk aksi utama (biru)                          |
| **Secondary**        | Variant untuk aksi netral (abu-abu)                      |
| **Success**          | Variant untuk konfirmasi keberhasilan (hijau)            |
| **Danger**           | Variant untuk aksi berbahaya (merah)                     |
| **Warning**          | Variant untuk aksi yang perlu perhatian (kuning)         |
| **Info**             | Variant untuk aksi informatif (cyan)                     |
| **kebab-case**       | Konvensi nama file: huruf kecil + tanda hubung           |
| **`@csrf`**          | Directive untuk token keamanan CSRF                      |
| **`@method('DELETE')`** | Directive untuk "memalsukan" method DELETE via POST   |
| **`confirm(...)`**   | Fungsi JavaScript untuk dialog konfirmasi                |
| **Komponen turunan** | Komponen yang memanggil komponen lain di dalamnya        |

---

## 13. Langkah Selanjutnya

Materi **Komponen Button Blade** sudah selesai. Berikut adalah **saran langkah selanjutnya** untuk pengembangan diri:

### Langkah 1: Praktik Mandiri

- Bikin komponen `<x-badge>` untuk label status (Aktif/Nonaktif).
- Bikin komponen `<x-alert>` untuk pesan sukses/error.
- Bikin komponen `<x-input>` untuk form input dengan label.

### Langkah 2: Eksplorasi Lebih Lanjut

- Pelajari **class-based component** (komponen dengan file PHP terpisah).
- Pelajari **named slot** (slot dengan nama, bukan slot default).
- Pelajari **Blade directive** lain seperti `@if`, `@foreach`, `@stack`, `@push`.

### Langkah 3: Terapkan di Project Nyata

- Refactor **semua tombol** di project kamu dengan komponen `<x-button>`.
- Bikin komponen untuk **elemen UI lain** yang sering muncul.
- Jaga konsistensi di **seluruh aplikasi**.

---

## 14. Penutup

Kamu telah melakukan perjalanan panjang:

- Dari **tombol berantakan** ke **tombol konsisten**.
- Dari **menulis class manual** ke **sekadar sebut variant**.
- Dari **copy-paste kode** ke **komponen reusable**.
- Dari **rawan typo** ke **validasi otomatis**.
- Dari **sulit dirawat** ke **mudah dirawat**.

Ini adalah **keterampilan fondasi** yang akan kamu pakai **sepanjang karir** sebagai developer Laravel. Setiap kali kamu membuat aplikasi, kamu akan **membuat komponen**. Setiap kali kamu membuat komponen, kamu akan ingat prinsip-prinsip yang dipelajari di sini.

Teruslah berlatih, teruslah membuat komponen, dan teruslah menjaga konsistensi.

**Selamat, kamu sudah lulus!**

---

> **Daftar Tahap (13. Komponen Button Blade):**
> - [x] Tahap 1 — Kenapa Tombol Harus Konsisten
> - [x] Tahap 2 — Membuat File Komponen `<x-button>` Sederhana
> - [x] Tahap 3 — Memahami Props dan Variant
> - [x] Tahap 4 — Slot dan Isi Tombol
> - [x] Tahap 5 — Menerapkan `<x-button>` di Halaman Produk
> - [x] Tahap 6 — Tombol Hapus dengan Form dan Konfirmasi
> - [x] Tahap 7 — Variant Tambahan dan Tipe Link
> - [x] Tahap 8 — Ringkasan dan Best Practice (kamu di sini)

**Materi 13. Komponen Button Blade: SELESAI.**
