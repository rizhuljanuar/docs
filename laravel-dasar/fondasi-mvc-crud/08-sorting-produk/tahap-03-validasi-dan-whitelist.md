# Tahap 3 — Kenapa `sort` Harus Divalidasi & Apa Itu Whitelist Kolom

> Materi: Sorting Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi Sehari-hari: Penjaga Klub Malam

Bayangkan ada klub malam. Di pintu masuk ada **penjaga (security)**.

Setiap tamu yang mau masuk, penjaga tanya: **"Nama kamu siapa?"**

Sekarang ada tiga jenis tamu:

- **Tamu A**: namanya ada di **daftar tamu VIP** → masuk.
- **Tamu B**: namanya tidak ada di daftar, tapi sopan → diarahkan ke halaman umum.
- **Tamu C**: bilang "Saya teman si X, suruh masuk langsung aja, ini uangnya" → **ditolak**.

Penjaga tidak boleh **percaya begitu saja**. Dia cuma boleh masukin orang yang namanya **sudah ada di daftar**.

Nah, di Laravel, **nilai `sort` dari user itu seperti "nama tamu"**. Kita (controller) harus jadi **penjaga**:

- Cek apakah nilai `sort` itu **ada di daftar yang kita izinkan**.
- Kalau ada → terima, eksekusi query.
- Kalau tidak ada → **tolak**, pakai default.

Daftar yang diizinkan itu kita sebut **whitelist**.

---

## 2. Bahaya Kalau `sort` Tidak Divalidasi

Ingat di tahap 2, kita ambil nilai dari user:

```php
$sort = request('sort');
```

Sekarang bayangkan **user iseng / jahat** mengirim URL aneh:

```
/produk?sort=' OR '1'='1
/produk?sort=drop table users
/produk?sort==password
```

Atau hal yang lebih "terlihat aman" tapi tetap berbahaya:

```
/produk?sort=password           // kolom sensitif, tidak boleh diketahui urutannya
/produk?sort=                   // kosong
/produk?sort=HARGA              // huruf besar, mungkin tidak match
/produk?sort=nama; DROP TABLE   // percobaan SQL injection
```

Kalau controller **langsung pakai** nilai itu ke query tanpa cek, dua hal bisa terjadi:

### a. Error 500 (Query Crash)
Kalau nilai `sort` tidak cocok dengan kolom manapun di database, Laravel akan melempar **error**. Halaman user jadi rusak.

### b. Kebocoran Data / Serangan (SQL Injection)
Kalau kita tidak hati-hati, user bisa **memanipulasi query** untuk mengakses data yang bukan haknya. Misal: mencoba menebak struktur tabel, atau menyisipkan perintah SQL berbahaya.

> Aturan emas keamanan web: **Jangan pernah percaya input user.** Apapun yang datang dari URL/form, **divalidasi dulu** sebelum dipakai.

---

## 3. Apa Itu Whitelist?

**Whitelist** = daftar nilai yang **diizinkan**. Apapun di luar daftar ini → **ditolak**.

Lawannya adalah **blacklist** (daftar nilai yang dilarang). Whitelist **lebih aman** daripada blacklist, karena:

- Blacklist: kamu harus menebak semua nilai jahat yang mungkin. Sulit, dan kamu pasti ketinggalan.
- Whitelist: kamu cuma izinkan yang **kamu tahu aman**. Sisanya otomatis ditolak.

#### Analogi Sederhana

- **Blacklist** = "Tolong masukin semua orang, kecuali yang ada di daftar hitam ini." → bahaya, karena penjahat baru yang belum ada di daftar tetap bisa masuk.
- **Whitelist** = "Tolong cuma masukin orang yang ada di daftar undangan ini. Sisanya tolak." → aman, karena hanya yang sudah disetujui yang masuk.

Di sorting produk, kita pakai **whitelist**.

---

## 4. Whitelist + Map: Satu Struktur Data Elegan

Di tahap 2 kita sudah punya tabel mapping:

| Nilai `sort` | Kolom | Arah |
|---|---|---|
| `terbaru` | `created_at` | `desc` |
| `terlama` | `created_at` | `asc` |
| `harga-termurah` | `harga` | `asc` |
| `harga-termahal` | `harga` | `desc` |
| `nama-az` | `nama` | `asc` |
| `nama-za` | `nama` | `desc` |

Ide bagusnya: **whitelist dan map bisa digabung jadi satu array asosiatif**.

```php
$allowed = [
    'terbaru'         => ['created_at', 'desc'],
    'terlama'         => ['created_at', 'asc'],
    'harga-termurah'  => ['harga', 'asc'],
    'harga-termahal'  => ['harga', 'desc'],
    'nama-az'         => ['nama', 'asc'],
    'nama-za'         => ['nama', 'desc'],
];
```

Penjelasan per bagian:

- `$allowed` = variabel yang menyimpan whitelist (sekaligus map).
- **Key** (kiri) = nilai `sort` yang diizinkan, misalnya `'terbaru'`.
- **Value** (kanan) = pasangan `[kolom, arah]` yang dipakai di query.

Contoh membacanya:

- Kalau `$sort = 'terbaru'`, maka kita ambil `['created_at', 'desc']`.
- Kalau `$sort = 'harga-termahal'`, maka kita ambil `['harga', 'desc']`.

Array ini **sekaligus dua hal**:

1. **Whitelist**: hanya key yang terdaftar yang sah.
2. **Map**: tiap key langsung punya arti `[kolom, arah]`.

Maka, cek "apakah nilai `sort` valid?" jadi sangat mudah:

```php
isset($allowed[$sort]);
```

- `isset` = "apakah key ini ada di array?"
- Kalau ada → `true` → nilai `sort` sah.
- Kalau tidak ada → `false` → nilai `sort` tidak sah, pakai default.

---

## 5. Mengapa Pendekatan Ini Lebih Aman Daripada `if-else`?

Bandingkan dengan `if-else` panjang di tahap 2.

#### Masalah `if-else` Panjang

- Kalau user kirim `sort=password`, tidak ada `if` yang match → jatuh ke `else` default. **Aman? Tergantung.** Kalau kamu lupa `else`, atau default tidak diset, bisa bocor.
- Repetitif → mudah salah ketik.
- Lupa menambah satu cabang → bug.

#### Kelebihan Whitelist + Map

- **Satu sumber kebenaran**: daftar `$allowed` adalah satu-satunya tempat yang mendefinisikan apa yang sah.
- **Tidak peduli apa yang user kirim**: apapun nilainya, kalau tidak ada di `$allowed`, otomatis ditolak.
- **Mudah dirawat**: mau tambah sort baru? Tambah satu baris di `$allowed`. Selesai.

> Ponypoint: whitelist itu seperti daftar undangan. Hanya yang ada di daftar yang masuk. Yang lain otomatis dapat default.

---

## 6. Validasi Sederhana: Pakai `??` (Null Coalescing)

Sekarang kita bisa tulis logika **"ambil nilai yang sah, atau pakai default"** dengan rapi.

```php
$sort = request('sort');                          // ambil dari URL

$allowed = [
    'terbaru'         => ['created_at', 'desc'],
    'terlama'         => ['created_at', 'asc'],
    'harga-termurah'  => ['harga', 'asc'],
    'harga-termahal'  => ['harga', 'desc'],
    'nama-az'         => ['nama', 'asc'],
    'nama-za'         => ['nama', 'desc'],
];

// Kalau $sort tidak ada di whitelist, pakai 'terbaru' (default)
$sort = $allowed[$sort] ?? 'terbaru';
```

Penjelasan baris terakhir:

- `$allowed[$sort]` = ambil pasangan `[kolom, arah]` kalau `$sort` ada di whitelist.
- `?? 'terbaru'` = kalau tidak ada (null), pakai string `'terbaru'` sebagai default.

Tunggu, ini ada **hal kecil yang perlu diperbaiki**. Default-nya harus konsisten: harus berupa pasangan `[kolom, arah]`, bukan string. Jadi lebih baik begini:

```php
$sort = request('sort');

$allowed = [
    'terbaru'         => ['created_at', 'desc'],
    'terlama'         => ['created_at', 'asc'],
    'harga-termurah'  => ['harga', 'asc'],
    'harga-termahal'  => ['harga', 'desc'],
    'nama-az'         => ['nama', 'asc'],
    'nama-za'         => ['nama', 'desc'],
];

// Default: terbaru
[$kolom, $arah] = $allowed[$sort] ?? $allowed['terbaru'];
```

Penjelasan baris terakhir:

- `[$kolom, $arah] = ...` → **destructuring**. Memecah array `[kolom, arah]` jadi dua variabel terpisah.
- `$allowed[$sort] ?? $allowed['terbaru']` → kalau nilai `sort` ada di whitelist, ambil pasangannya. Kalau tidak ada, pakai `terbaru`.

Setelah baris ini, kita punya dua variabel bersih:

- `$kolom` → contoh: `'created_at'`, `'harga'`, `'nama'`.
- `$arah` → `'asc'` atau `'desc'`.

Dua variabel inilah yang akan kita pakai di `orderBy()` tahap berikutnya.

---

## 7. Cek: Apakah Sudah Aman?

Mari kita uji dengan beberapa skenario:

| URL user buka | Nilai `$sort` | Hasil setelah validasi |
|---|---|---|
| `/produk?sort=terbaru` | `'terbaru'` | `$kolom='created_at', $arah='desc'` |
| `/produk?sort=nama-az` | `'nama-az'` | `$kolom='nama', $arah='asc'` |
| `/produk?sort=password` | `'password'` | tidak ada di whitelist → default `terbaru` |
| `/produk?sort=' OR 1=1` | `' OR 1=1` | tidak ada di whitelist → default `terbaru` |
| `/produk` (tanpa `?sort=`) | `null` | tidak ada di whitelist → default `terbaru` |

Semua kasus "aneh" → **jatuh ke default**, tidak pernah menyentuh query asli. Inilah keamanan yang kita dapat dari whitelist.

---

## Ringkasan Tahap 3

| Hal | Isi |
|---|---|
| Aturan emas | Jangan pernah percaya input user |
| Bahaya tanpa validasi | Error 500, SQL injection, kebocoran data |
| Whitelist | Daftar nilai yang diizinkan; sisanya ditolak |
| Blacklist vs whitelist | Whitelist lebih aman (izinkan yang diketahui) |
| Whitelist + map | Satu array sekaligus jadi daftar sah + peta ke `[kolom, arah]` |
| Cek validitas | `isset($allowed[$sort])` atau `$allowed[$sort] ?? default` |
| Destructuring | `[$kolom, $arah] = ...` memecah array jadi 2 variabel |
| Default | `'terbaru'` → `[created_at, desc]` |

---

## Kode Sejauh Ini (Preview)

```php
$sort = request('sort');

$allowed = [
    'terbaru'         => ['created_at', 'desc'],
    'terlama'         => ['created_at', 'asc'],
    'harga-termurah'  => ['harga', 'asc'],
    'harga-termahal'  => ['harga', 'desc'],
    'nama-az'         => ['nama', 'asc'],
    'nama-za'         => ['nama', 'desc'],
];

[$kolom, $arah] = $allowed[$sort] ?? $allowed['terbaru'];

// TODO tahap 4-5: pakai $kolom & $arah di Produk::orderBy(...)
```

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: melihat controller produk saat ini (sebelum sorting), lalu menerapkan dynamic query berdasarkan whitelist?**

Kalau iya, tahap 4 kita akan:

1. Lihat bentuk controller produk yang masih statis (pakai `orderBy` mati atau `latest()`).
2. Identifikasi baris mana yang perlu diubah.
3. Siap tempat untuk menyisipkan kode dynamic query.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
