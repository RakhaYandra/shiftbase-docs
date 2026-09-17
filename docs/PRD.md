# PRD - Shiftbase

| | |
|---|---|
| Versi | 1.0.0 (2026-09-17) |
| Acuan | BRD v1.0.0 (`BR-01`-`BR-09`) |

## 1. Persona

- **Admin/HR**: kelola karyawan & akun, impor CSV, laporan penuh.
- **Manajer**: susun roster, pantau absensi & lembur, impor CSV.
- **Staf**: lihat jadwal sendiri, catat absensi sendiri.

## 2. Fitur

### F-AUTH - Autentikasi (BR-07)

- US-01: Sebagai pengguna baru, saya dapat register (role default staf) sehingga punya akun.
  - AC: `POST /v1/auth/register` 201 + token; email duplikat -> 409 `email_taken`.
- US-02: Sebagai pengguna, saya dapat login dan melihat profil saya.
  - AC: login benar -> 200 + JWT 24 jam; salah -> 401 `invalid_credentials`; `GET /v1/me` butuh token.
- US-03: Sebagai staf, saya ditolak saat akses resource admin.
  - AC: respons 403 `forbidden` (matrix RBAC di FSD bagian 5).

### F-EMP - Karyawan (BR-01)

- US-04: Sebagai admin, saya dapat CRUD karyawan (nama, email unik, telepon, posisi, tanggal masuk).
  - AC: create 201; email ganda 409; format email/tanggal salah 400; hapus 204; detail tak ada 404.

### F-SHIFT - Roster Shift (BR-02)

- US-05: Sebagai manajer, saya dapat membuat shift (karyawan, tanggal, jam mulai < jam selesai).
  - AC: create 201; bentrok (overlap pegawai sama) -> 409 `shift_conflict`; beda pegawai jam sama -> 201.
- US-06: Sebagai staf, saya hanya melihat shift milik saya.
  - AC: list terfilter otomatis ke employee_id sendiri; tulis -> 403.

### F-ATT - Absensi (BR-03)

- US-07: Sebagai staf, saya dapat check-in lalu check-out satu kali per hari.
  - AC: check-in 201; check-in ganda 409; check-out tanpa check-in terbuka 404; check-out 200.
- US-08: Sebagai manajer, saya dapat melihat riwayat absensi dengan filter tanggal/karyawan.
  - AC: `GET /v1/attendance?from=&to=&employee_id=` 200.

### F-IMP - Impor CSV (BR-04)

- US-09: Sebagai admin, saya dapat impor karyawan via CSV dan tahu baris mana yang gagal.
  - AC: header wajib `name,email,phone,position,hire_date`; respons `{imported,failed,errors[{row,error}]}`; tanpa file/lebih 2 MB -> 400; staf -> 403.

### F-REP - Laporan (BR-05, BR-06)

- US-10: Sebagai manajer, saya dapat melihat lembur per karyawan (jam > 8/hari) dan coverage per tanggal.
  - AC: overtime = `GREATEST(total_jam - 8, 0)` per hari; coverage = headcount per tanggal; staf -> 403.

### F-WEB - Roster Board (BR-02, BR-03, BR-05)

- US-11: Sebagai manajer, saya melihat roster visual harian (timeline 06:00-24:00 per karyawan) dan CRUD shift dari tabel per tanggal.
  - AC: 4 halaman role-aware (Jadwal semua; Shift admin/manajer; Absensi semua; Laporan admin/manajer); tab persist setelah reload; dropdown karyawan refresh otomatis setelah impor.
- US-12: Sebagai staf, saya hanya melihat halaman Jadwal & Absensi.
  - AC: nav Shift/Laporan tidak tampil untuk staf.

### F-DATA - Analitik (BR-08)

- US-13: Sebagai manajer, saya mendapat file parquet (hours_daily, overtime_employee, coverage_daily, attendance_rate, late_arrivals) + 4 grafik PNG + laporan quality JSON.
  - AC: quality gate menggagalkan run bila data kosong/orphan/duplikat/waktu inkonsisten; cross-check API overtime toleransi < 0,01.

### F-OPS - Operasional (BR-09)

- US-14: Sebagai operator, saya punya runbook, SLA, troubleshooting terverifikasi, backup/restore, dan tiket insiden.
  - AC: backup gzip rotasi 7 hari; restore konfirmasi + verifikasi COUNT; 10 tiket tercatat dengan SLA.

## 3. Non-Goals (v1.0)

Deploy produksi, performance testing, payroll payout, aplikasi mobile, shift overnight di web board.
