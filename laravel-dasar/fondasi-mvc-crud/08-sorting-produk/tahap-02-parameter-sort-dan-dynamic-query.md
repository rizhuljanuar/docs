# Tahap 2 — Parameter `sort` di URL & Konsep Dynamic Query

> Materi: Sorting Produk
> Level: Pemula Laravel (Fondasi MVC + CRUD)
> Contoh kasus: CRUD Produk

---

## 1. Analogi Sehari-hari: Pesan Kopi

Bayangkan kamu ke coffee shop. Kamu pesan kopi lewat **kasir**.

Kamu tidak cuma bilang "Satu kopi." Kamu kasih **detail**:

- "Kopi, **yang panas**."
- "Kopi, **yang dingin**."
- "Kopi, **less sugar**."

Nah, di dunia web, **URL itu seperti pesanan kamu ke kasir**, dan **controller itu kasirnya**. Salah satu detail yang bisa kamu kasih adalah: **"tolong urutkan produknya seperti ini."**

Detail itu kita kirim lewat yang namanya **query parameter** di URL.

---

## 2. Apa itu Query Parameter di URL?

Mari kita lihat lagi URL dari tahap 1:

```
/produk?sort=terbaru
```

Bagi menjadi dua bagian:

```
/produk        ?sort=terbaru
  ↑               ↑
  path       query parameter
```

- **Path** = alamat halaman (di sini: halaman daftar produk).
- **Query parameter** = detail tambahan yang dimulai dengan tanda `?`.

Format query parameter:

```
?nama=nilai
```

- `nama` = key (kata kunci), di sini `sort`.
- `nilai` = value (pilihan user), di sini `terbaru`.

Bisa juga ada lebih dari satu parameter, dipisah dengan `&`:

```
/produk?sort=terbaru&page=2
```

Artinya: "Tampilkan produk, urutkan terbaru, dan saya mau halaman 2."

> Ponypoint: query parameter itu cuma pasangan key-value di URL. Laravel sudah otomatis baca untukmu.

---

## 3. Daftar Nilai `sort` yang Akan Kita Dukung

Demi kesederhanaan, kita pakai 6 pilihan ini:

| Nilai `sort` di URL | Maksud user | Field database | Arah |
|---|---|---|---|
| `terbaru` | produk baru dulu | `created_at` | DESC (turun) |
| `terlama` | produk lama dulu | `created_at` | ASC (naik) |
| `harga-termurah` | murah ke mahal | `harga` | ASC |
| `harga-termahal` | mahal ke murah | `harga` | DESC |
| `nama-az` | A ke Z | `nama` | ASC |
| `nama-za` | Z ke A | `nama` | DESC |

Jadi kalau URL-nya `/produk?sort=nama-az`, server harus tahu: "Oh, user mau urutkan kolom `nama` secara ASC."

Tabel di atas itu adalah **peta terjemahan**: dari bahasa user → ke perintah database.

---

## 4. Bagaimana Controller Mengambil Parameter Itu?

Di Laravel, untuk **mengambil** query parameter, kita pakai helper `request()`.

```php
$sort = request('sort');
```

Penjelasan:

- `request()` = helper Laravel untuk akses input user (dari URL, form, dll).
- `'sort'` = nama parameter yang mau diambil.
- Hasilnya disimpan di variabel `$sort`.

Contoh:

| URL yang user buka | Nilai `$sort` |
|---|---|
| `/produk?sort=terbaru` | `'terbaru'` |
| `/produk?sort=harga-termurah` | `'harga-termurah'` |
| `/produk` (tanpa parameter) | `null` (kosong) |

Kalau `$sort` kosong, artinya user **tidak memilih opsi sorting**, jadi kita pakai urutan default (biasanya terbaru).

---

## 5. Konsep Dynamic Query (Pelan-pelan)

Sekarang bagian inti: **dynamic query**.

#### a. Query Statis (yang biasa kamu tulis)

Selama ini kamu mungkin menulis query seperti ini:

```php
$produk = Produk::orderBy('created_at', 'desc')->get();
```

Ini disebut **query statis**: urutannya **ditulis mati** di kode. Selalu `created_at` DESC, tidak bisa diubah user. Semua user lihat urutan yang sama.

#### b. Query Dinamis (yang ingin kita buat)

**Dynamic query** = query yang **bentuknya berubah-ubah tergantung input user**.

Kalau user pilih `terbaru` → query jadi `orderBy('created_at', 'desc')`.
Kalau user pilih `harga-termurah` → query jadi `orderBy('harga', 'asc')`.
Kalau user pilih `nama-az` → query jadi `orderBy('nama', 'asc').

Satu kode controller, tapi hasil querynya **berbeda-beda** tiap request. Itulah "dynamic".

#### c. Ide Naif: pakai `if-else`

Cara paling mudah membayangkannya adalah dengan `if-else`:

```php
$sort = request('sort');

if ($sort === 'terbaru') {
    $produk = Produk::orderBy('created_at', 'desc')->get();
} elseif ($sort === 'terlama') {
    $produk = Produk::orderBy('created_at', 'asc')->get();
} elseif ($sort === 'harga-termurah') {
    $produk = Produk::orderBy('harga', 'asc')->get();
} elseif ($sort === 'harga-termahal') {
    $produk = Produk::orderBy('harga', 'desc')->get();
} elseif ($sort === 'nama-az') {
    $produk = Produk::orderBy('nama', 'asc')->get();
} elseif ($sort === 'nama-za') {
    $produk = Produk::orderBy('nama', 'desc')->get();
} else {
    // default: terbaru
    $produk = Produk::orderBy('created_at', 'desc')->get();
}
```

Ini **bisa jalan**, tapi ada dua masalah:

1. **Repetitif.** Hampir setiap baris ngulang pola yang sama: cuma beda kolom dan arah.
2. **Berbahaya secara keamanan.** Nanti di tahap 3 kita bahas kenapa **nilai `$sort` dari user tidak boleh langsung dipakai** tanpa validasi.

Karena itu, di tahap berikutnya kita akan pakai pendekatan yang lebih rapi: **whitelist + map** ke kolom dan arah.

---

## 6. Kerangka Berpikir untuk Dynamic Query Sorting

Sebelum nulis kode, kunci polanya begini:

```
1. Ambil parameter:       request('sort')
2. Cek di whitelist:      apakah nilainya boleh?
3. Kalau boleh →          ambil [kolom, arah] dari peta
4. Kalau tidak boleh →    pakai default
5. Eksekusi query:        Produk::orderBy($kolom, $arah)->...
```

Mental modelnya:

> User kasih kunci (`sort=terbaru`) → kita cek kamus → kalau kuncinya sah, kita ambil artinya (`['created_at', 'desc']`) → kita pakai artinya itu di query.

Dua hal yang harus kita bangun di tahap berikutnya:

1. **Whitelist** = daftar nilai `sort` yang diizinkan.
2. **Map** = peta dari nilai `sort` ke pasangan `[kolom, arah]`.

Kita akan gabungkan keduanya jadi **satu struktur data** (array asosiatif) yang elegan.

---

## Ringkasan Tahap 2

| Hal | Isi |
|---|---|
| Query parameter | Detail di URL setelah `?`, format `?nama=nilai` |
| Ambil di Laravel | `request('sort')` → ambil nilai parameter `sort` |
| Query statis | Urutan ditulis mati di kode, sama untuk semua user |
| Dynamic query | Query berubah-ubah sesuai input user |
| Ide naif | `if-else` panjang → bisa jalan, tapi repetitif & kurang aman |
| Pola rapi (next) | Whitelist + map nilai → `[kolom, arah]` |
| Nilai default | Kalau `$sort` kosong/tidak sah, pakai default (mis. terbaru) |

---

## Kode Sejauh Ini (Preview)

```php
$sort = request('sort'); // ambil parameter dari URL

// TODO tahap 3: validasi $sort dengan whitelist
// TODO tahap 4-5: bangun dynamic query berdasarkan whitelist
```

Belum ada query final, karena kita perlu bahas **keamanan** dulu sebelum eksekusi.

---

## Pertanyaan untuk Kamu

> **Apakah kamu ingin lanjut ke langkah berikutnya: kenapa nilai `sort` dari user harus divalidasi, dan apa itu whitelist kolom yang boleh diurutkan?**

Kalau iya, tahap 3 kita akan bahas:

1. Bahaya kalau user kirim nilai `sort` yang jahat (SQL injection, error database).
2. Apa itu **whitelist** dan kenapa lebih aman daripada `if-else`.
3. Bentuk awal whitelist sebagai array asosiatif.

Tunggu konfirmasi kamu dulu sebelum lanjut ya.
