# Tahap 10 — Ringkasan dan Best Practice

> Mentor: Laravel Dasar — Fondasi MVC, CRUD
> Topik: 12. Layout Blade Reusable
> Fokus tahap ini: **menyatukan semua konsep**, **best practice**, **kesalahan umum**, dan **checklist refactor**.

---

## 1. Tujuan Belajar Tahap Akhir Ini

Setelah tahap ini, kamu harus bisa:

1. Menyatukan **empat alat Blade** (layout, partial, component, slot) dalam satu gambaran.
2. Memilih **alat yang tepat** untuk setiap situasi.
3. Menghindari **kesalahan umum** pemula Blade.
4. Menilai apakah Blade kamu sudah rapi lewat **checklist refactor**.
5. Tahu **langkah belajar berikutnya** setelah topik 12.

Selamat! Kamu sudah menyelesaikan **seluruh tahap Layout Blade Reusable**. Ini pencapaian besar.

---

## 2. Perjalanan Belajar: Dari Masalah ke Solusi

Mari kita lihat kembali perjalanan 9 tahap terakhir:

### Awal mula (Tahap 1)

Kamu punya **5 halaman** Blade yang masing-masing menulis `<html>`, `<header>`, `<footer>` sendiri-sendiri.

**Masalah:** duplikasi, sulit dirawat, melanggar DRY.

### Kenalan layout (Tahap 2-3)

- **Tahap 2:** bikin satu file `layout/app.blade.php` dengan lubang `@yield('konten')`.
- **Tahap 3:** halaman pakai layout lewat `@extends('layout.app')` + `@section('konten')`.

### Terapkan ke produk (Tahap 4-5)

- **Tahap 4:** ubah halaman **daftar produk** pakai layout.
- **Tahap 5:** ubah halaman **tambah, edit, detail** produk + kenalan `@push('css')`.

### Halaman admin (Tahap 6)

- **Tahap 6:** ubah halaman **dashboard**, kenalan sidebar opsional + multi-layout.

### Bagian reusable (Tahap 7-9)

- **Tahap 7:** **partial** untuk navbar/footer (bagian sekali pakai lintas layout).
- **Tahap 8:** **component** untuk kartu produk (bagian dipakai berkali-kali).
- **Tahap 9:** **slot** untuk isi bebas di dalam component.

Hasil akhir: **satu kerangka layout + banyak bagian kecil reusable** alih-alih 5 file duplikat.

---

## 3. Empat Alat Blade: Tabel Ringkasan Akhir

| Alat | Cara pakai | Untuk | Input |
|---|---|---|---|
| **Layout** | `@extends('layout.app')` + `@section` | kerangka halaman (1 per halaman) | `@yield` lubang |
| **Partial** | `@include('partials.x')` | bagian **sekali pakai** (navbar, footer) | array asosiatif |
| **Component** | `<x-nama />` | bagian **berkali-kali** (kartu, tombol) | props |
| **Slot** | `<x-nama> ISI </x-nama>` | isi **bebas** dalam component | `$slot`, named slot |

### Aturan emas pemilihan alat

```
Saya butuh...
│
├─ kerangka halaman utuh → LAYOUT (@extends + @yield)
│
├─ bagian yang muncul SEKALI per halaman
│   ├─ navbar, footer (lintas layout) → PARTIAL (@include)
│   └─ satu halaman saja → biarkan di layout
│
├─ bagian yang muncul BERKALI-KALI
│   ├─ isinya DATA sederhana (objek, angka, string) → COMPONENT + props
│   └─ isinya HTML/struktur bebas → COMPONENT + slot
│
└─ lubang yang diisi dari luar
    ├─ konteks layout → @yield
    └─ konteks component → $slot
```

---

## 4. Struktur Folder Ideal Views

Setelah menerapkan semua konsep, folder `resources/views/` kamu terlihat seperti ini:

```
resources/views/
│
├── layout/                          ← Kerangka halaman
│   ├── app.blade.php                (layout publik)
│   └── admin.blade.php              (opsional: layout admin)
│
├── partials/                        ← Bagian sekali pakai lintas layout
│   ├── navbar.blade.php
│   ├── footer.blade.php
│   ├── sidebar.blade.php            (opsional)
│   └── flash-message.blade.php      (opsional)
│
├── components/                      ← Bagian dipakai berulang
│   ├── kartu-produk.blade.php
│   ├── tombol.blade.php
│   ├── badge-status.blade.php
│   └── pesan.blade.php              (alert/notifikasi)
│
├── produk/                          ← Halaman terkait produk
│   ├── index.blade.php              (daftar, pakai kartu-produk)
│   ├── create.blade.php
│   ├── edit.blade.php
│   └── show.blade.php
│
├── dashboard/
│   └── index.blade.php
│
└── layout/ sudah ada di atas
```

Setiap folder punya **peran jelas**:

- `layout/` = kerangka.
- `partials/` = potongan sekali pakai.
- `components/` = potongan berulang.
- `produk/`, `dashboard/` = halaman nyata yang menggabungkan ketiganya.

---

## 5. Best Practice Blade untuk Pemula

Berikut **10 aturan praktis** yang membuat Blade kamu rapi dan mudah dirawat:

### 1. Selalu pakai layout (jangan pernah tulis `<html>` di halaman)

Setiap halaman Blade (`produk/index.blade.php`, dll.) **wajib** diawali `@extends('layout.app')`. Tidak boleh ada `<!DOCTYPE html>` di file halaman.

### 2. Nama folder pakai huruf kecil, jamak, konsisten

- `layout/`, `partials/`, `components/`, `produk/`.
- Hindari `Layout/`, `partial/`, `component/`.

### 3. Nama file component pakai kebab-case

- ✅ `kartu-produk.blade.php`
- ❌ `KartuProduk.blade.php`, `kartu_produk.blade.php`

### 4. Pakai `{{ route('nama') }}`, bukan URL manual

- ✅ `{{ route('produk.index') }}`
- ❌ `/produk` (kalau route berubah, harus ubah manual)

### 5. Variabel opsional selalu pakai `?? '...'`

- ✅ `{{ $title ?? 'Toko Bukhari' }}`
- ❌ `{{ $title }}` (error kalau tidak dikirim)

### 6. Pisahkan logic berat ke controller, jangan query di Blade

- ❌ `@foreach (App\Models\Produk::all() as $p)` (query di Blade)
- ✅ Kirim data dari controller, di Blade cuma looping `$produk`

### 7. Gunakan `@push` untuk CSS/JS khusus halaman, bukan inline di layout

- ✅ `@push('css') <style>...</style> @endpush`
- ❌ Semua CSS ditumpuk di layout (layout jadi berantakan)

### 8. Komponen berulang → pakai component, bukan partial

- ✅ `<x-kartu-produk />` untuk kartu yang muncul berkali-kali
- ❌ `@include('partials.kartu')` di dalam `@foreach` (bisa, tapi kurang idiomatik)

### 9. Props untuk data, slot untuk tampilan

- ✅ `:produk="$item"` → kirim object Produk
- ✅ `<x-tombol><i></i> Simpan</x-tombol>` → kirim HTML lewat slot
- ❌ `<x-tombol teks="<i></i> Simpan" />` (HTML di string props, jelek)

### 10. Jangan over-engineer di awal

- Hanya 1 layout dulu sampai butuh layout kedua.
- Partial opsional kalau navbar masih pendek.
- Class-based component hanya kalau anonymous component kurang.

> 📝 **Pesan mentor:**
> Best practice nomor 10 adalah yang paling penting. Siswa yang **terlalu semangat** bikin 5 layout + 20 partial + 15 component di proyek kecil justru **menambah kompleksitas tidak perlu**. Mulai sederhana, refactor saat dibutuhkan.

---

## 6. Kesalahan Umum yang Harus Dihindari

### Kesalahan 1: Tulis `<header>` di halaman dan di layout

**Salah:**

```blade
@extends('layout.app')          ← layout sudah punya <header>

@section('konten')
    <header>...</header>         ← DUPLIKAT! header muncul dua kali
    ...
@endsection
```

**Benar:** hapus `<header>` dari halaman, biar layout yang urus.

### Kesalahan 2: Lupa `@endsection` atau `@endpush`

**Salah:**

```blade
@section('konten')
    <p>Isi...</p>
{{-- lupa @endsection --}}
```

**Akibat:** Blade error atau section tidak ditutup.

**Benar:** selalu tutup dengan `@endsection`, `@endpush`, `@endslot`.

### Kesalahan 3: Pakai `tampilkan="false"` (string truthy)

**Salah:**

```blade
<x-kartu :produk="$item" tampilkan="false" />
```

Tanpa `:`, `"false"` jadi **string truthy** (dianggap true di PHP).

**Benar:**

```blade
<x-kartu :produk="$item" :tampilkan="false" />
```

Pakai `:` supaya dievaluasi sebagai boolean false.

### Kesalahan 4: Query database di Blade

**Salah:**

```blade
@foreach (App\Models\Kategori::all() as $kat)
    <option>{{ $kat->nama }}</option>
@endforeach
```

**Akibat:** Blade jadi bergantung ke Model, sulit diuji, performa buruk.

**Benar:** query di controller, kirim ke view:

```php
// controller
return view('produk.create', ['kategoris' => Kategori::all()]);
```

```blade
@foreach ($kategoris as $kat)
    <option>{{ $kat->nama }}</option>
@endforeach
```

### Kesalahan 5: Tulis HTML di dalam string props

**Salah:**

```blade
<x-tombol teks="<i class='icon'></i> Simpan" />
```

HTML di dalam string = jelek, rawan error escaping.

**Benar:** pakai slot:

```blade
<x-tombol>
    <i class="icon"></i> Simpan
</x-tombol>
```

### Kesalahan 6: Bikin layout kedua terlalu cepat

**Salah:** langsung bikin `layout/admin.blade.php` padahal hanya 1 halaman admin.

**Benar:** mulai dengan `@yield('sidebar')` opsional dulu. Baru pindah ke layout kedua kalau ada 3+ halaman admin.

### Kesalahan 7: Nama section tidak konsisten

**Salah:** layout pakai `@yield('konten')`, halaman pakai `@section('isi')`.

**Akibat:** lubang `konten` tetap kosong, halaman tidak tampil.

**Benar:** nama harus **persis sama**. Gunakan konvensi: `konten`, `sidebar`, `css`, `js`.

---

## 7. Checklist Refactor Blade

Pakai checklist ini untuk menilai apakah Blade kamu sudah rapi. Centang yang sudah dilakukan:

### Struktur folder

- [ ] Folder `layout/`, `partials/`, `components/` sudah ada dan terpisah.
- [ ] Nama folder huruf kecil, jamak.
- [ ] Tidak ada file Blade tersebar di root `views/` (kecuali `welcome.blade.php` bawaan Laravel).

### Layout

- [ ] Ada minimal satu file `layout/app.blade.php`.
- [ ] Layout punya `@yield('konten')` untuk konten utama.
- [ ] Layout punya `@stack('css')` dan `@stack('js')` (opsional tapi disarankan).
- [ ] Judul pakai `{{ $title ?? 'Default' }}` (tidak pernah undefined).

### Halaman

- [ ] Setiap halaman diawali `@extends('layout.app')`.
- [ ] Tidak ada `<!DOCTYPE html>`, `<html>`, `<body>` di halaman (semua di layout).
- [ ] Tidak ada duplikat `<header>` atau `<footer>` antara layout dan halaman.
- [ ] Konten halaman dibungkus `@section('konten') ... @endsection`.

### Variabel

- [ ] Semua variabel dikirim dari controller, tidak query di Blade.
- [ ] Variabel opsional pakai `{{ $var ?? 'default' }}`.
- [ ] Tidak ada `{{ $undefined }}` yang bisa menyebabkan error.

### Component & partial

- [ ] Bagian sekali pakai lintas layout dipindah ke `partials/`.
- [ ] Bagian berulang di-refactor jadi `components/`.
- [ ] Nama file component pakai kebab-case.
- [ ] Component pakai `@props([...])` untuk deklarasi input.
- [ ] Isi HTML kompleks dipindah ke slot, bukan dijadikan string props.

### Routing

- [ ] Semua link pakai `{{ route('nama.route') }}`, bukan URL manual.
- [ ] Form pakai `@csrf` dan `@method()` (PUT/DELETE).

> 📝 **Pesan mentor:**
> Kalau checklist di atas lebih dari 80% sudah tercentang, Blade kamu sudah **sangat rapi** untuk level dasar. Kalau di bawah 50%, ada beberapa hal yang perlu dirapikan. Tenang, refactor itu proses, bukan satu kali keajaiban.

---

## 8. Contoh Lengkap: Halaman Daftar Produk Setelah Refactor

Sebagai penutup, ini contoh **hasil akhir** halaman daftar produk setelah menerapkan semua konsep:

### `produk/index.blade.php`

```blade
@extends('layout.app', ['title' => 'Daftar Produk'])

@push('css')
    <style>
        .grid-produk {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
            gap: 15px;
        }
    </style>
@endpush

@section('konten')
    <h2>Daftar Produk</h2>
    <x-tombol tipe="primary" href="{{ route('produk.create') }}">
        + Tambah Produk
    </x-tombol>

    <div class="grid-produk">
        @foreach ($produk as $item)
            <x-kartu-produk :produk="$item">
                @if ($item->aktif)
                    <x-badge-status :aktif="true" />
                @else
                    <x-badge-status :aktif="false" />
                @endif
            </x-kartu-produk>
        @endforeach
    </div>

    @if (session('sukses'))
        <x-pesan tipe="success">{{ session('sukses') }}</x-pesan>
    @endif
@endsection
```

Lihat betapa **ringkas dan ekspresif** nya:

- `@extends` + `@push` + `@section` untuk kerangka & CSS khusus.
- `<x-tombol>` (component + slot) untuk tombol tambah.
- `<x-kartu-produk>` (component + props + slot) untuk kartu produk dengan badge di dalamnya.
- `<x-badge-status>` (component + props) untuk badge aktif/nonaktif.
- `<x-pesan>` (component + slot) untuk flash message.

**Bandingkan** dengan versi lama (sebelum belajar layout): 50+ baris HTML duplikat, tanpa component, sulit dirawat.

Ini **perbedaan pemula vs level menengah** dalam Blade.

---

## 9. Manfaat Nyata Setelah Belajar Layout Blade

Mari kita ringkas manfaat konkret yang sekarang kamu punya:

| Manfaat | Sebelum | Sesudah |
|---|---|---|
| **Kode per halaman** | 50-80 baris (banyak duplikat) | 20-30 baris (hanya konten unik) |
| **Ubah header/footer** | ubah 5 file satu per satu | ubah 1 file (layout) |
| **Tampil kartu produk baru** | copy-paste HTML ke tiap halaman | pakai `<x-kartu-produk />` |
| **Konsistensi** | rawan ada halaman tertinggal | semua halaman otomatis sama |
| **Kolaborasi tim** | orang bentrok di file yang sama | orang kerja folder berbeda (partials/components) |
| **Onboarding member baru** | bingung melihat duplikat | jelas: layout ada di `layout/`, kartu di `components/` |

Ini adalah **kenyataan proyek Laravel nyata**, bukan teori. Skill ini dipakai di setiap proyek Laravel profesional.

---

## 10. Langkah Belajar Berikutnya

Topik 12 selesai. Berikut saran lanjutan:

### Level dasar lanjutan

- **13. Middleware dan Autentikasi** — proteksi halaman admin, hanya user login yang bisa akses.
- **14. Relasi Database (Eloquent)** — Produk belongsTo Kategori, User hasMany Pesanan.
- **15. Migration dan Seeder** — version control skema database.

### Level menengah (setelah dasar kuat)

- **16. Service Class** — pisahkan logic dari controller.
- **17. Form Request Validation** — validasi di class terpisah.
- **18. Queue dan Job** — eksekusi tugas berat di background.
- **19. API dan JSON Response** — bikin backend untuk frontend terpisah.
- **20. Testing** — PHPUnit/Pest untuk otomatisasi tes.

### Best practice Blade yang masih bisa dipelajari

- **Blade component class** — component dengan class PHP untuk logic kompleks.
- **View composers** — share variabel ke banyak view otomatis.
- **Livewire / Filament** — library yang bikin Blade lebih interaktif tanpa JavaScript.

---

## 11. Proyek Latihan Akhir

Sebelum lanjut ke topik berikutnya, coba kerjakan **proyek mini** ini untuk menguasai semua konsep:

### Proyek: Toko Online Sederhana (Bagian Tampilan)

Buat tampilan untuk toko online dengan:

1. **Layout publik** (`layout/app.blade.php`):
   - Navbar dengan logo + menu (Beranda, Produk, Tentang, Kontak).
   - Footer dengan copyright + link sosial media.

2. **Layout admin** (`layout/admin.blade.php`):
   - Sidebar dengan menu (Dashboard, Produk, User, Logout).
   - Header "Admin Panel".

3. **Partials**:
   - `partials/navbar.blade.php` (dipakai layout publik).
   - `partials/sidebar-admin.blade.php` (dipakai layout admin).
   - `partials/flash-message.blade.php`.

4. **Components**:
   - `<x-kartu-produk :produk="$p">` — kartu produk untuk daftar.
   - `<x-tombol tipe="...">` — tombol dengan slot.
   - `<x-badge-status :aktif="$x">` — badge aktif/nonaktif.
   - `<x-pesan tipe="...">` — alert notifikasi.

5. **Halaman**:
   - `produk/index.blade.php` — daftar produk pakai kartu-produk.
   - `produk/create.blade.php`, `edit.blade.php`, `show.blade.php`.
   - `dashboard/index.blade.php` — dashboard admin.

### Tujuan proyek

- Tidak ada duplikasi `<html>`, `<header>`, `<footer>` di mana pun.
- Header/footer bisa diubah dari **satu tempat**.
- Kartu produk **satu component** dipakai di banyak halaman.
- Sidebar hanya muncul di halaman admin, tidak di publik.

Kalau kamu bisa menyelesaikan proyek ini, kamu sudah **siap kerja proyek Laravel nyata** sebagai junior developer.

---

## 12. Pesan Penutup Mentor

Selamat! Kamu telah menyelesaikan **Topik 12 — Layout Blade Reusable** dengan lengkap.

Yang kamu pelajari:

- **Konsep:** layout, reusable, template composition, DRY.
- **Alat:** layout (`@extends`/`@yield`), partial (`@include`), component (`<x-...>` + props), slot (`$slot`).
- **Praktik:** refactor 5 halaman produk + dashboard ke pola rapi.
- **Best practice:** 10 aturan + 7 kesalahan umum + checklist refactor.

### Yang penting kamu pegang

> **Coding bukan tentang menghafal, tapi tentang pola.**
>
> Setelah kamu paham **pola** "bikin satu, pakai banyak", kamu bisa pakai pola yang sama di framework lain (React, Vue, Django, Rails). Konsepnya universal.

### Saran ke depan

- **Latihan terus.** Baca tutorial tidak cukup. Refactor proyek lama kamu sampai rapi.
- **Baca kode orang lain.** Lihat proyek open source Laravel, perhatikan struktur folder views-nya.
- **Jangan takut salah.** Error `undefined variable` dan lupa `@endsection` itu **normal**, semua programmer pernah mengalaminya.
- **Pertahankan pola rapi sejak awal proyek baru.** Lebih mudah mulai rapi daripada refactor proyek berantakan.

Selamat belajar, dan sampai jumpa di **topik 13 — Middleware dan Autentikasi**.

---

## 13. Cek Pemahaman Akhir (Gabungan)

Jawab pertanyaan ini tanpa melihat jawaban. Ini menguji seluruh tahap 1-10.

1. Sebutkan **empat alat** Blade untuk template composition dan kapan masing-masing dipakai.
2. Apa directive untuk: (a) mewarisi layout, (b) membuat lubang konten di layout, (c) menyisipkan partial, (d) mendeklarasikan props di component?
3. Apa beda `:nama="$var"` dan `nama="val"`?
4. Apa beda `@yield` dan `$slot`?
5. Apa risiko tulis `<header>` di halaman yang sudah pakai layout?
6. Apa solusi kalau navbar ditulis dua kali di `layout.app` dan `layout.admin`?
7. Apa yang terjadi kalau lupa `@endsection`?
8. Sebutkan 3 kesalahan umum Blade yang harus dihindari.

<details>
<summary><strong>Klik untuk melihat jawaban akhir</strong></summary>

1. **Layout** (kerangka halaman, `@extends`), **Partial** (bagian sekali pakai, `@include`), **Component** (bagian berulang, `<x-...>` + props), **Slot** (isi bebas dalam component, `$slot`).
2. (a) `@extends`, (b) `@yield`, (c) `@include`, (d) `@props([...])`.
3. `:nama="$var"` evaluasi PHP (kirim nilai variabel). `nama="val"` kirim string literal "val".
4. `@yield` untuk **layout** (lubang konten utama), `$slot` untuk **component** (lubang isi bebas). Konsep sama, konteks berbeda.
5. **Header muncul dua kali** di halaman: satu dari layout, satu dari file halaman. Tampilan jadi aneh.
6. Pindahkan navbar ke **partial** (`partials/navbar.blade.php`), lalu kedua layout pakai `@include('partials.navbar')`. Ubah partial sekali → dua layout ikut berubah.
7. Blade error "section not closed" atau konten setelahnya jadi bagian section yang tidak ditutup.
8. Tiga dari: (a) duplikat header/footer antara layout & halaman, (b) lupa `@endsection`/`@endpush`, (c) `tampilkan="false"` tanpa titik dua (string truthy), (d) query database di Blade, (e) HTML di string props (harusnya pakai slot), (f) bikin layout kedua terlalu cepat, (g) nama section tidak konsisten.

</details>

---

## 14. Penutup Topik 12

> 🎉 **Selamat!**
>
> Kamu telah menyelesaikan **Topik 12: Layout Blade Reusable** dari awal sampai akhir.
>
> - **10 tahap** lengkap.
> - **4 alat Blade** dikuasai: layout, partial, component, slot.
> - **5 halaman produk + dashboard** sudah pakai layout rapi.
> - **Best practice, kesalahan umum, checklist refactor** sudah dipelajari.

Langkah berikutnya terserah kamu:

- **Latihan** dengan proyek mini di section 11.
- **Lanjut topik 13** (Middleware & Autentikasi).
- **Review tahap** yang masih kurang dipahami.

Apapun yang kamu pilih, **fondasi Blade kamu sudah kuat**. Good luck untuk perjalanan selanjutnya.

---

> 📚 **Daftar Tahap (12. Layout Blade Reusable):**
> - ✅ Tahap 1 — Apa itu layout Blade reusable
> - ✅ Tahap 2 — Membuat file layout utama Blade
> - ✅ Tahap 3 — `@extends` dan `@section` untuk memakai layout
> - ✅ Tahap 4 — Mengubah halaman daftar produk agar pakai layout
> - ✅ Tahap 5 — Mengubah halaman tambah, edit, detail produk
> - ✅ Tahap 6 — Mengubah halaman dashboard admin
> - ✅ Tahap 7 — Partial: memecah navbar dan footer
> - ✅ Tahap 8 — Component: membuat komponen kartu produk
> - ✅ Tahap 9 — Slot: lubang yang bisa diisi apa saja
> - ✅ Tahap 10 — Ringkasan dan best practice (kamu di sini) — **SELESAI**

---

> 🚀 **Topik Berikutnya:**
> - 13. Middleware dan Autentikasi
> - 14. Relasi Database (Eloquent)
> - 15. Migration dan Seeder
