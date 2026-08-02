# Tahap 1 — Mengapa Tombol Harus Konsisten

Pada aplikasi saat ini, halaman daftar `/products`, form tambah dan edit, serta detail product memakai tindakan yang berulang: tambah, simpan, update, detail, edit, kembali, dan hapus. Jika setiap view menulis class tombol sendiri, tampilan dan makna tindakan mudah berbeda.

Komponen Blade menyimpan pola itu sekali lalu memakainya kembali. Tujuannya hanya presentasi: query paginator `$products`, filter category, upload image, validasi, endpoint, dan controller yang sudah ada tidak diubah.

## Peran variant

| Variant | Peran | Contoh |
| --- | --- | --- |
| `primary` | tindakan utama pada kelompok tindakan terkait | Add, Save, Update |
| `secondary` | navigasi netral | Cancel, Back |
| `info` | melihat informasi | Detail |
| `warning` | mengubah data | Edit |
| `danger` | tindakan berisiko | Delete |

Variant menyatakan makna, bukan sekadar warna. Satu kelompok tindakan perlu memiliki tindakan utama yang jelas; bukan berarti setiap halaman hanya boleh memiliki satu tombol `primary` dalam semua konteks.

Pada tahap berikutnya kita membuat anonymous component `<x-button>`. Komponen ini tidak mengubah model Product, relasi category, SoftDeletes, atau status `is_active`.
