# Tahap 5 — Memastikan User Baru Otomatis Memiliki Role `user`

> Fokus: memastikan setiap akun baru dimulai sebagai `user`, bukan `admin`. Ini adalah aturan keamanan penting sebelum kita membuat akun admin latihan.

Pada tahap 4, kita menambahkan kolom `role` ke tabel `users` dengan nilai default `user`:

```php
$table->string('role')->default('user');
```

Sekarang kita perlu memahami mengapa nilai default ini penting dan bagaimana menjaganya saat user baru mendaftar.

## Analogi kartu akses baru

Bayangkan sebuah toko membuat kartu akses untuk pegawai atau pelanggan baru.

Kartu baru seharusnya tidak langsung membuka ruang gudang dan ruang pengelola. Kartu itu hanya membuka area umum.

Jika seseorang memang dipercaya menjadi pengelola, pemilik toko memberikan akses admin **secara sengaja** setelahnya.

Aturan yang sama dipakai pada aplikasi Laravel:

```text
Akun baru dibuat
        ↓
Role awal: user
        ↓
Tidak dapat mengakses area admin
        ↓
Hanya akun yang sengaja diubah menjadi admin yang dapat masuk
```

Prinsip ini disebut **default aman**. Ketika kita belum yakin seorang user boleh menjadi admin atau tidak, pilih akses paling kecil, yaitu `user`.

## Mengapa semua user baru tidak boleh menjadi admin?

Bayangkan form pendaftaran memiliki kolom tersembunyi atau pilihan seperti ini:

```text
Nama: Andi
Email: andi@example.com
Role: admin
```

Jika aplikasi menerima nilai `role` dari form pendaftaran tanpa pemeriksaan yang benar, seseorang bisa mencoba mengirim nilai `admin` sendiri. Akibatnya, user biasa dapat membuat akun dengan akses admin.

Itu sangat berbahaya karena akun tersebut dapat mencoba membuka:

- dashboard admin,
- halaman tambah Product,
- halaman edit dan hapus Product,
- daftar order,
- daftar user.

Karena itu, **user tidak boleh memilih role `admin` saat mendaftar**. Aplikasi harus menentukan role awalnya sendiri sebagai `user`.

## Perlindungan pertama, default pada database

Migration dari tahap 4 sudah memberi perlindungan dasar berikut:

```php
$table->string('role')->default('user');
```

Artinya, ketika Laravel membuat baris user tanpa memberikan nilai `role`, database akan mengisi nilai ini:

```text
user
```

Contoh konsep penyimpanan user:

```php
User::create([
    'name' => 'Andi',
    'email' => 'andi@example.com',
    'password' => Hash::make('password-rahasia'),
]);
```

Karena tidak ada key `role` pada data tersebut, database memakai default:

```text
role: user
```

Kode contoh ini belum tentu sama persis dengan kode login atau pendaftaran di project kamu. Tujuannya adalah menunjukkan bahwa **role tidak dikirim oleh user** dan database yang mengisi nilai aman `user`.

## Perlindungan kedua, jangan menerima `role` dari form pendaftaran

Misalnya form pendaftaran hanya meminta data yang memang boleh diisi calon user:

```text
name
email
password
password_confirmation
```

Data role tidak perlu ada di form tersebut.

Saat menyimpan data pendaftaran, bentuk aman sederhananya seperti ini:

```php
$user = User::create([
    'name' => $request->name,
    'email' => $request->email,
    'password' => Hash::make($request->password),
]);
```

Penjelasan setiap bagian:

- `User::create([...])` membuat data user baru di tabel `users`.
- `'name' => $request->name` mengambil nama dari form.
- `'email' => $request->email` mengambil email dari form.
- `'password' => Hash::make($request->password)` mengubah password menjadi hash sebelum disimpan.
- Tidak ada `'role' => ...` pada data pendaftaran ini.
- Karena `role` tidak dikirim, database otomatis memakai default `user` dari migration.

Jangan membuat kode pendaftaran seperti ini:

```php
User::create($request->all());
```

Kode tersebut mengambil semua data request tanpa memilih field satu per satu. Jika request memuat `role=admin`, nilai itu berisiko ikut diproses apabila model mengizinkannya.

Gunakan data yang sudah dipilih dan divalidasi, bukan seluruh request.

## Apa hubungan `$fillable` dengan keamanan role?

Model Eloquent, termasuk `app/Models/User.php`, memiliki perlindungan bernama **mass assignment**.

Mass assignment terjadi saat banyak data dikirim sekaligus ke `create()` atau `update()`, contohnya:

```php
User::create([
    'name' => 'Andi',
    'email' => 'andi@example.com',
    'password' => '...'
]);
```

Properti `$fillable` menentukan kolom mana yang boleh diisi melalui cara tersebut.

Bentuk model `User` pada Laravel baru umumnya mempunyai bagian seperti ini:

```php
protected $fillable = [
    'name',
    'email',
    'password',
];
```

Untuk pendaftaran user biasa, **jangan tambahkan `role` ke `$fillable`** hanya agar role dapat dibuat.

```php
protected $fillable = [
    'name',
    'email',
    'password',
    // Jangan menambahkan role di sini untuk form pendaftaran umum.
];
```

Alasannya sederhana:

- form pendaftaran mengirim data dari luar aplikasi,
- role adalah hak akses penting,
- user biasa tidak boleh bebas mengirim atau mengubah role-nya sendiri.

Untuk tahap ini, migration dengan default `user` sudah cukup. Kita tidak perlu mengubah `$fillable` agar user baru memiliki role `user`.

> Jangan menambahkan `role` ke `$fillable` hanya karena kolom tersebut baru dibuat. Kolom yang berkaitan dengan hak akses harus diperlakukan lebih hati-hati daripada nama atau email.

## Jika project memakai Laravel starter kit

Beberapa project Laravel memakai starter kit atau fitur login yang sudah menyiapkan proses register. Lokasi kode pendaftaran dapat berbeda-beda, tergantung starter kit yang dipakai.

Namun prinsipnya tetap sama:

1. Form register tidak menyediakan pilihan role `admin`.
2. Data yang diterima dari request hanya nama, email, dan password yang divalidasi.
3. Kolom `role` tidak berasal dari input user.
4. Database memberi default `user`.

Jadi, kamu tidak perlu memaksakan perubahan pada file register yang belum kamu pahami. Periksa dahulu apakah proses register di project sudah membuat user tanpa mengirim nilai `role`. Jika ya, default database dari tahap 4 akan bekerja.

## User lama dan user baru

Setelah migration tahap 4 dijalankan, terdapat dua keadaan:

### User yang sudah ada sebelum kolom role dibuat

Saat kolom `role` ditambahkan dengan default `user`, user lama akan mendapatkan nilai aman:

```text
role: user
```

### User yang dibuat setelah kolom role tersedia

Selama proses pendaftaran tidak mengirim nilai role, database juga memberi nilai:

```text
role: user
```

Hasil akhirnya:

| Jenis akun | Role awal yang diharapkan |
| --- | --- |
| Akun lama | `user` |
| Akun baru dari form register | `user` |
| Akun admin latihan | Akan diatur sengaja pada tahap berikutnya |

## Bagaimana dengan User Factory?

Pada materi 16, factory digunakan untuk membuat data dummy. Jika project kamu memakai `UserFactory`, factory tersebut juga perlu jelas menghasilkan role `user` untuk user dummy biasa.

Contoh bagian `definition()` pada `database/factories/UserFactory.php`:

```php
public function definition(): array
{
    return [
        'name' => fake()->name(),
        'email' => fake()->unique()->safeEmail(),
        'email_verified_at' => now(),
        'password' => static::$password ??= Hash::make('password'),
        'remember_token' => Str::random(10),
        'role' => 'user',
    ];
}
```

Penjelasan baris yang baru:

```php
'role' => 'user',
```

Baris ini memastikan setiap user dummy dari factory memiliki role `user` secara jelas.

Apakah baris factory ini wajib? Jika database sudah mempunyai default `user`, user dummy yang dibuat tanpa `role` juga dapat memperoleh nilai default dari database. Namun menuliskannya di factory membuat tujuan data dummy lebih jelas.

Jika kamu belum memakai `UserFactory` atau belum membuat seeder user, jangan mengubah factory hanya karena contoh ini ada. Terapkan perubahan hanya pada bagian yang benar-benar dipakai project kamu.

## Cara memeriksa role user baru

Setelah membuat akun melalui proses register atau factory, buka tabel `users` pada database tool yang kamu gunakan.

Periksa bahwa user baru memiliki nilai:

```text
role: user
```

Untuk latihan, kamu juga dapat memeriksa melalui Laravel Tinker:

```bash
php artisan tinker
```

Lalu di dalam Tinker:

```php
App\Models\User::latest()->first(['name', 'email', 'role']);
```

Penjelasan singkat:

- `App\Models\User` adalah model untuk tabel `users`.
- `latest()` mengambil data terbaru berdasarkan waktu pembuatan.
- `first(...)` mengambil satu user terbaru dan membatasi kolom yang ditampilkan.
- `['name', 'email', 'role']` berarti kita hanya ingin melihat tiga data tersebut.

Contoh hasil yang diharapkan:

```text
=> App\Models\User {#...
     name: "Andi",
     email: "andi@example.com",
     role: "user",
   }
```

Tinker hanya dipakai untuk memeriksa data pada langkah ini. Jangan membagikan password, token, atau isi `.env` ke dokumentasi atau chat.

## Kesalahan umum yang perlu dihindari

### 1. Menampilkan pilihan `admin` pada form register

Jangan membuat pilihan seperti ini untuk pendaftaran umum:

```html
<select name="role">
    <option value="user">User</option>
    <option value="admin">Admin</option>
</select>
```

User tidak seharusnya dapat menentukan dirinya sendiri sebagai admin.

### 2. Memasukkan `role` dari `$request->all()`

Jangan memakai seluruh request untuk membuat user. Pilih dan validasi data yang benar-benar dibutuhkan.

### 3. Menambahkan `role` ke `$fillable` tanpa alasan

Pada aplikasi sederhana ini, pendaftaran user biasa tidak memerlukan mass assignment untuk `role`. Biarkan role dilindungi dari input umum.

### 4. Menganggap tombol admin yang disembunyikan sudah aman

Menjaga role saat pendaftaran penting, tetapi middleware tetap wajib dibuat. User biasa masih dapat mencoba URL admin secara langsung.

## Yang perlu diingat pada tahap ini

1. Semua akun baru harus dimulai sebagai `user`.
2. Nilai default `user` pada migration adalah lapisan keamanan dasar.
3. Form register tidak boleh menerima pilihan atau input role `admin`.
4. Jangan memakai `$request->all()` untuk membuat user.
5. Jangan menambahkan `role` ke `$fillable` hanya untuk proses register biasa.
6. Akun admin harus dibuat atau diubah dengan sengaja, bukan karena input dari user.

Tahap berikutnya akan membuat satu akun admin latihan secara sengaja dan aman, tanpa menjadikan semua user lain sebagai admin.

---

**Apakah kamu ingin lanjut ke tahap 6: membuat akun admin latihan dengan aman?**
