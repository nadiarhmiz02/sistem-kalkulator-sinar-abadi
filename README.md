# SAPT Sinar Abadi

Sistem Informasi Perhitungan PPh Pasal 21 untuk membantu PT Sinar Abadi mengelola data karyawan, penggajian, dan pencatatan pajak. Perhitungan bulanan menggunakan Tarif Efektif Rata-rata (TER) sesuai PMK 168/2023, sebagai pengganti proses manual di spreadsheet.

![Frontend](https://img.shields.io/badge/Frontend-HTML%20%7C%20CSS%20%7C%20JavaScript-e34f26?style=flat-square) ![Database](https://img.shields.io/badge/Database-Supabase%20%7C%20PostgreSQL-3fcf8e?style=flat-square) ![Bahasa](https://img.shields.io/badge/Bahasa-Indonesia-blue?style=flat-square) ![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)

🌐 **Demo website:** [https://nadiarhmiz02.github.io/sistem-kalkulator-sinar-abadi/](https://nadiarhmiz02.github.io/sistem-kalkulator-sinar-abadi/)

## Fitur

- Login email/password dengan Supabase Auth dan role Admin HR atau Staff.
- Dashboard: karyawan aktif, jumlah data gaji bulan ini, total PPh 21 terutang, dan gaji yang belum dihitung.
- CRUD jabatan dengan level staff, supervisor, dan manajer serta nominal default.
- CRUD karyawan: NIK, NPWP, status PTKP dan kepegawaian, jabatan, tanggal masuk, serta status aktif.
- Input dan pengelolaan gaji bulanan, tunjangan, dan potongan BPJS Kesehatan, JHT, dan JP.
- Perhitungan PPh 21 massal dengan kategori TER A, B, dan C serta faktor 1,2 untuk karyawan tanpa NPWP.
- Riwayat dengan filter nama/periode dan pagination; ekspor riwayat ke Excel dan slip gaji atau riwayat ke PDF.
- Nomor slip otomatis dengan format `SG/{tahun}/{bulan}/{urut}` dan audit waktu `created_at`/`updated_at`.

## Struktur repository

```text
pph21-sinar-abadi/
├── backend/
│   ├── app.js
│   └── database.sql
├── frontend/
│   ├── css/
│   │   └── style.css
│   ├── html/
│   │   └── index.html
│   └── js/
│       └── script.js
├── index.html
├── .gitignore
└── README.md
```

`frontend/html/index.html` adalah halaman untuk Live Server lokal, sedangkan `index.html` di root dipakai GitHub Pages. `frontend/js/script.js` mengelola interaksi antarmuka; `backend/app.js` berjalan di browser dan memuat helper Supabase serta logika perhitungan. `backend/database.sql` berisi schema, relasi, kebijakan akses, trigger audit, dan data awal jabatan.

## Menjalankan aplikasi lokal

### Persiapan Supabase

1. Buat project Supabase dan buka **SQL Editor**.
2. Jalankan seluruh isi [`backend/database.sql`](backend/database.sql). Script membuat empat tabel operasional, tabel `profiles`, RLS, trigger, indeks, dan seed jabatan.
3. Di **Authentication → Users**, buat akun operator. Trigger SQL akan membuat profil awal dengan role `staff`.
4. Untuk menetapkan Admin, ubah role profil dari SQL Editor memakai akun/proyek yang berwenang, misalnya:

	```sql
	UPDATE public.profiles
	SET role = 'admin'
	WHERE email = 'admin@sinarabadi.com';
	```

	Konfirmasi email pengguna bila pengaturan Auth project mewajibkannya.

### Menjalankan frontend

1. Buka `backend/app.js` dan isi `SUPABASE_URL` serta `SUPABASE_KEY` dengan URL project dan **publishable/anon key** dari Supabase. Jangan gunakan `service_role` key di browser atau repository publik.
2. Pastikan koneksi internet tersedia untuk Supabase SDK dan library CDN.
3. Buka `frontend/html/index.html` di VS Code, lalu pilih **Open with Live Server**.
4. Masuk dengan akun Supabase yang sudah dibuat.

Alternatif server lokal dari direktori root repository:

```sh
python3 -m http.server 8000
```

Buka `http://localhost:8000/frontend/html/` di browser.

## GitHub Pages

Demo menggunakan `index.html` di root repository; file ini memiliki path asset yang sesuai untuk publikasi Pages. Aktifkan Pages pada repository melalui **Settings → Pages**, pilih branch dan folder root yang digunakan, lalu tunggu proses deployment. Pastikan URL project Supabase, Auth URL/redirect settings, dan izin origin/API sesuai dengan domain `nadiarhmiz02.github.io`. Karena frontend berjalan di browser, gunakan hanya publishable/anon key dengan RLS aktif.

## Dokumentasi database

Relasi utama: satu `jabatan` dapat dimiliki banyak `karyawan`; satu karyawan dapat memiliki banyak `data_gaji_bulanan`; setiap baris gaji memiliki paling banyak satu `perhitungan_pph21` melalui `gaji_id` yang unik. `profiles.id` terhubung ke `auth.users.id` dan menyimpan role aplikasi.

- `jabatan`: nama dan level jabatan, gaji pokok/tunjangan default, serta waktu audit.
- `karyawan`: identitas, PTKP, status kepegawaian, jabatan, tanggal masuk, dan status aktif.
- `data_gaji_bulanan`: periode, komponen gaji, tunjangan, serta potongan BPJS.
- `perhitungan_pph21`: bruto, kategori/tarif TER, PPh normal/terutang, status pembayaran, dan nomor slip.
- `profiles`: email, nama, dan role (`admin` atau `staff`) untuk pengguna Supabase Auth.

Schema lengkap, constraint, foreign key, RLS, trigger `updated_at`, dan seed tersedia di [`backend/database.sql`](backend/database.sql).

## Perhitungan PPh 21

Untuk periode Januari-November, bruto bulanan dihitung sebagai:

```text
Bruto = gaji pokok + tunjangan tetap + tunjangan tidak tetap
PPh normal = pembulatan(bruto × tarif TER sesuai kategori PTKP)
PPh terutang = PPh normal × 1,2 jika tanpa NPWP; selain itu PPh normal
```

Potongan BPJS dicatat pada data gaji/slip, tetapi tidak mengurangi bruto TER. TER mengacu kategori A, B, atau C berdasarkan status PTKP. Perhitungan ulang tahunan bulan Desember (Pasal 17) tidak termasuk cakupan aplikasi.

## Keamanan

Supabase Auth mengautentikasi pengguna. Tabel memakai Row Level Security: pengguna terautentikasi dapat membaca data, sementara perubahan data operasional dibatasi ke role `admin`; role `staff` bersifat baca-saja. `profiles` memiliki kebijakan terpisah, dan perubahan role harus dilakukan melalui jalur berwenang, bukan dari browser. Audit `updated_at` dijalankan oleh trigger database.

> **Penting:** jangan menaruh `service_role` key atau data karyawan nyata di repository publik. Pastikan RLS dan policies telah aktif sebelum memakai aplikasi; publishable/anon key tetap dapat dilihat publik pada aplikasi browser.

## Asumsi & Batasan

- Cakupan perhitungan adalah TER bulanan Januari-November; rekonsiliasi Desember/Pasal 17 belum tersedia.
- Kategori bukan pegawai, termasuk tenaga ahli, tidak menjadi cakupan perhitungan khusus. Verifikasi status sebelum memasukkan gaji untuk dihitung.
- BPJS hanya dicatat sebagai potongan dan tidak mengurangi dasar bruto TER.
- Seed SQL menyediakan data jabatan, bukan akun demo atau data karyawan.
- Aplikasi memerlukan koneksi internet untuk Supabase dan library CDN.

## Teknologi

- Frontend: HTML5, CSS3, JavaScript vanilla; UI memakai Lucide Icons.
- Backend logic: JavaScript di browser (`backend/app.js`), bukan server backend tersendiri.
- Database dan API: Supabase PostgreSQL, Supabase JavaScript SDK, serta RLS.
- Autentikasi: Supabase Auth (email dan password).
- Library: Chart.js, jsPDF + AutoTable, SheetJS, dan Google Fonts (Inter, JetBrains Mono, Playfair Display).

## Akun Demo

- Admin: `admin@sinarabadi.com` / `admin123`
- Staff: `staff@sinarabadi.com` / `staff123`

> Kredensial demo bersifat publik dan tidak dibuat oleh seed `database.sql`; pastikan akun tersedia di Supabase Auth serta role profil Admin sudah ditetapkan. Gunakan project dan data dummy khusus demo, jangan gunakan password tersebut untuk akun pribadi atau data produksi.

## Lisensi

© 2026 PT Sinar Abadi | [Nama Kamu] | [NIM]
