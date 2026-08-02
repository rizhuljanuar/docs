# Tahap 1 — Mengapa Tombol Harus Konsisten

Aplikasi saat ini memiliki tindakan berulang pada daftar `/products`, form tambah/edit, dan detail product: tambah, simpan, update, detail, edit, kembali, dan hapus. Jika setiap view menulis class sendiri, tampilannya mudah tidak seragam.

Blade component menyimpan presentasi tombol di satu tempat. Pelajaran ini tidak mengubah paginator `$products`, filter category, upload image, validasi, endpoint, controller, model Product, SoftDeletes, atau `is_active`.

| Variant | Peran | Contoh |
| --- | --- | --- |
| `primary` | tindakan utama dalam kelompok tindakan terkait | Add, Save, Update |
| `secondary` | navigasi netral | Cancel, Back |
| `info` | melihat informasi | Detail |
| `warning` | mengubah data | Edit |
| `danger` | tindakan berisiko | Delete |

Variant menyatakan makna, bukan sekadar warna. Setiap kelompok tindakan perlu memiliki tindakan utama yang jelas; ini bukan aturan satu tombol `primary` untuk seluruh halaman.
