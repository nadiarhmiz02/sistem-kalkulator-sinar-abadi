# SAPT Sinar Abadi

Sistem Informasi Perhitungan PPh Pasal 21 untuk PT Sinar Abadi.

## Fitur
- Login & Role (Admin/Staff)
- CRUD Jabatan, Karyawan, Data Gaji
- Hitung PPh 21 massal (skema TER PMK-168/2023)
- Faktor NPWP ×1.2
- Riwayat + Export PDF/Excel

## Tech Stack
- HTML5, CSS3, JavaScript
- Supabase PostgreSQL

## Cara Menjalankan
1. Jalankan `backend/database.sql` di Supabase SQL Editor
2. Ganti `SUPABASE_URL` & `SUPABASE_KEY` di `backend/app.js`
3. Buka `frontend/html/index.html` via Live Server
