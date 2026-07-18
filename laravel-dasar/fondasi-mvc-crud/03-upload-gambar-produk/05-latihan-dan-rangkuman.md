# Upload Gambar Produk — Tahap 5: Latihan, Kasus Lanjutan & Rangkuman

> Bagian dari: Laravel Dasar — Fondasi MVC & CRUD
> Topik: 3. Upload Gambar Produk
> Tahap: 5 dari 5 — **Latihan, Kasus Lanjutan & Rangkuman Akhir** (PENUTUP)

---

## 1. Tujuan Tahap Penutup

Tahap ini berisi:

1. **Latihan soal** untuk menguji pemahamanmu.
2. **Kasus lanjutan** yang sering muncul di dunia nyata (termasuk hapus gambar).
3. **Rangkuman akhir** seluruh topik Upload Gambar Produk.

> **Ponytail:** Kerjakan latihan dulu **tanpa** melihat jawaban. Kalau buntu,
> baru lihat petunjuk. Ini cara belajar paling efektif.

---

## 2. Latihan Soal (Kerjakan Sendiri)

### Soal 1 — Aturan Validasi
Tulis aturan validasi untuk field `gambar` dengan syarat:
- Wajib diisi.
- Harus gambar.
- Format hanya `jpg`, `png`, atau `webp`.
- Ukuran maksimal **1.5 MB**.

### Soal 2 — Debugging
User upload file `foto.jpg` 800 KB. Submit sukses, tidak ada error. Tapi
gambar tidak muncul di halaman daftar produk. Sebutkan **3 kemungkinan penyebab**.

### Soal 3 — Penyimpanan
Jelaskan dengan kata-kata sendiri: apa yang dikembalikan oleh
`$request->file('gambar')->store('produk', 'public')`, dan apa yang kamu
lakukan dengan nilai tersebut?

### Soal 4 — Tampilan Blade
Lengkapi tag `<img>` berikut supaya menampilkan gambar produk dengan aman:

```php
<img src="{{ ??????????? }}" alt="{{ $produk->nama }}" width="200">
```

### Soal 5 — Storage Abstraction
Kalau besok kamu pindahkan penyimpanan dari lokal ke Amazon S3, **bagian mana
saja dari kode aplikasi yang perlu diubah**? (form? controller? blade? migration?)

---

## 3. Petunjuk & Jawaban (Cek Setelah Mencoba)

<details>
<summary><strong>Klik untuk melihat jawaban</strong></summary>

### Jawaban 1
```php
'gambar' => ['required', 'image', 'mimes:jpg,png,webp', 'max:1536'],
```
> 1.5 MB = 1536 KB (karena `max:` dalam kilobyte).

### Jawaban 2
Kemungkinan penyebab:
1. **Belum jalankan `php artisan storage:link`** → symlink `public/storage` belum ada.
2. **Salah disk**: pakai `store('produk')` tanpa argumen `'public'`, jadi file
   masuk ke disk `local` (private), tidak bisa diakses browser.
3. **Symlink rusak / dihapus**: jalankan ulang `php artisan storage:link`.

### Jawaban 3
`store('produk', 'public')` mengembalikan **path relatif terhadap disk**,
misalnya `"produk/aB3xK9pQ.jpg"`. Path ini saya simpan ke kolom `gambar`
di database, karena database hanya perlu tahu **lokasi** file, bukan isi file-nya.

### Jawaban 4
```php
<img src="{{ Storage::url($produk->gambar) }}" alt="{{ $produk->nama }}" width="200">
```

### Jawaban 5
**Tidak ada kode aplikasi yang perlu diubah** (itu kekuatan abstraksi).
Cukup ubah konfigurasi di `config/filesystems.php` dan `.env`
(tambahkan key AWS S3), lalu set `FILESYSTEM_DISK=s3` atau ganti disk
default. Form, controller, blade, migration semua tetap.

</details>

---

## 4. Kasus Lanjutan yang Sering Muncul

Di project nyata, kamu akan menemui kasus berikut. Kita bahas satu per satu.

### Kasus A: Hapus Gambar Saat Produk Dihapus

**Masalah**: Saat produk dihapus dari database, file gambar di storage **masih
tersisa** → storage lama-lama penuh dengan file sampah.

**Solusi**: pakai Laravel Observer atau hapus manual di controller.

Cara manual (paling sederhana, ponytail friendly):

```php
public function destroy(Produk $produk)
{
    if ($produk->gambar && Storage::disk('public')->exists($produk->gambar)) {
        Storage::disk('public')->delete($produk->gambar);
    }

    $produk->delete();

    return redirect()->route('produk.index')->with('success', 'Produk dihapus.');
}
```

Logika:
- Cek `gambar` tidak null **dan** file-nya benar-benar ada (`exists`).
- Baru hapus file-nya (`delete`), lalu hapus record DB.

---

### Kasus B: Ganti Gambar Produk (Update)

Saat user mengganti gambar produk, **gambar lama harus dihapus** supaya tidak
menumpuk di storage.

```php
public function update(Request $request, Produk $produk)
{
    $data = $request->validate([
        'nama'   => ['required', 'string', 'max:100'],
        'gambar' => ['nullable', 'image', 'mimes:jpg,jpeg,png,webp', 'max:2048'],
    ]);

    if ($request->hasFile('gambar')) {
        // Hapus gambar lama kalau ada
        if ($produk->gambar && Storage::disk('public')->exists($produk->gambar)) {
            Storage::disk('public')->delete($produk->gambar);
        }

        // Simpan gambar baru
        $data['gambar'] = $request->file('gambar')->store('produk', 'public');
    }

    $produk->update($data);

    return redirect()->route('produk.index')->with('success', 'Produk diperbarui.');
}
```

Catatan: di update, `gambar` biasanya **`nullable`** (boleh tidak diganti).

---

### Kasus C: Batas Upload di Sisi Server (`php.ini`)

Validasi Laravel `max:2048` hanya menolak file besar **setelah** file terkirim
ke server. Tapi PHP sendiri punya batas di `php.ini`:

```ini
upload_max_filesize = 2M
post_max_size = 8M
```

Kalau user kirim file 5 MB padahal `upload_max_filesize = 2M`, PHP menolak
sebelum Laravel sempat memvalidasi → user melihat error bingung (halaman
kosong / 422). Solusinya:

- Set `upload_max_filesize` sesuai kebijakan aplikasi (misal `5M`).
- Set `post_max_size` sedikit lebih besar dari `upload_max_filesize`.

---

### Kasus D: Multiple Upload (Banyak Gambar per Produk)

Kalau satu produk butuh banyak gambar (galeri), input file pakai array:

```php
<input type="file" name="gambar[]" multiple>
```

Validasi pakai wildcard `.*`:

```php
'gambar'   => ['required', 'array'],
'gambar.*' => ['image', 'mimes:jpg,jpeg,png,webp', 'max:2048'],
```

Simpan dengan loop:

```php
$paths = [];
foreach ($request->file('gambar') as $file) {
    $paths[] = $file->store('produk', 'public');
}
// simpan $paths ke tabel terpisah: produk_gambar (produk_id, path)
```

> **Ponytail:** Untuk pemula, jangan langsung lompat ke kasus ini. Pahami
> single upload dulu sampai lancar.

---

## 5. Checklist Pemahaman Akhir

Centang (di kepala) yang sudah kamu pahami:

- [ ] Saya tahu **kenapa** file upload harus divalidasi.
- [ ] Saya bisa menulis aturan validasi: `image`, `mimes:`, `max:`.
- [ ] Saya paham beda `storage/app/public` dengan `public/storage` (symlink).
- [ ] Saya sudah pernah menjalankan `php artisan storage:link`.
- [ ] Saya tahu `store('produk', 'public')` mengembalikan path, bukan URL.
- [ ] Saya bisa tampilkan gambar dengan `Storage::url()` di Blade.
- [ ] Saya tahu cara hapus gambar saat produk dihapus / diupdate.
- [ ] Saya paham kenapa pindah ke S3 tidak butuh ubah kode aplikasi.

Kalau semua centang, **kamu sudah lulus topik Upload Gambar Produk** 🎯.

---

## 6. Rangkuman Akhir Topik

| Tahap | Konsep Utama                                              |
| ----- | --------------------------------------------------------- |
| 1     | Upload = kirim file user → server. Validasi wajib.        |
| 2     | Storage abstraction: disk, `storage/app/public`, symlink. |
| 3     | Form + aturan validasi (`image`, `mimes:`, `max:`).       |
| 4     | Simpan file, simpan path ke DB, tampilkan dengan `Storage::url()`. |
| 5     | Kasus nyata: hapus, update, multi-upload, limit php.ini.  |

**Aturan emas** yang harus selalu diingat:

> **Database menyimpan path. Storage menyimpan file. Browser mengakses lewat
> `Storage::url()` yang butuh symlink `public/storage`.**

Itu inti dari semua tahap di atas.

---

## 7. Apa Selanjutnya?

Setelah menguasai Upload Gambar Produk, materi Laravel Dasar biasanya lanjut ke:

- **4. Relasi Database** (One-to-Many: Produk → Kategori).
- **5. Query Builder vs Eloquent**.
- **6. Authentication bawaan Laravel** (login, register).

Atau, untuk pendalaman storage:

- **Image Manipulation** dengan package `intervention/image` (resize, crop).
- **Queue untuk upload besar** (supaya user tidak menunggu lama).

---

> **Selesai!** 🎉 Kamu sudah menyelesaikan seluruh tahap untuk topik
> **Upload Gambar Produk**. Semoga pemahamanmu kuat. Mau lanjut ke topik
> berikutnya, atau ada bagian yang ingin diperdalam?
