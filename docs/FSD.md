# FSD - Shiftbase

| | |
|---|---|
| Versi | 1.0.0 (2026-09-17) |
| Acuan | PRD v1.0.0 |

## 1. Arsitektur

- Backend Go/Gin + MySQL 8.4 + JWT (repo `shiftbase`). Layer: handler -> service -> repository (`database/sql`, tanpa ORM - ADR-001); migrasi goose (ADR-002).
- Frontend Vite + React 19 + TS + Tailwind v4 (repo `shiftbase-web`). Tanpa react-router: navigasi state 4 page, persist `localStorage["shiftbase_page"]`.
- QA: Newman (API) + Playwright TS (UI) + `tools/*.py` (repo `shiftbase-qa`).
- Data: ETL Python -> DuckDB parquet + matplotlib PNG (repo `shiftbase-data`).
- Ops: runbook/SLA/tiket + script backup/restore (repo `shiftbase-ops`).
- Konfigurasi via env: `PORT=8080`, `DB_DSN`, `JWT_SECRET` (>= 32 char di prod), `FRONTEND_URL`.

## 2. Endpoint API

Base `/v1` (kecuali `/healthz`). Auth: `Authorization: Bearer <JWT 24 jam, claims sub=userID, role>`.

| ID | Method & Path | Akses | Fungsi |
|---|---|---|---|
| FS-01 | `GET /healthz` | publik | 200 `{"status":"ok"}` |
| FS-02 | `POST /v1/auth/register` | publik | register, role default staf; 409 `email_taken` |
| FS-03 | `POST /v1/auth/login` | publik | login -> `{token}`; 401 `invalid_credentials` |
| FS-04 | `GET /v1/me` | semua | profil sendiri |
| FS-05 | `POST /v1/employees` | admin | tambah karyawan |
| FS-06 | `GET /v1/employees` | admin,manajer | list |
| FS-07 | `GET /v1/employees/:id` | admin,manajer | detail; 404 |
| FS-08 | `PUT /v1/employees/:id` | admin | ubah; 404 |
| FS-09 | `DELETE /v1/employees/:id` | admin | hapus; 204 |
| FS-10 | `POST /v1/employees/import` | admin,manajer | CSV multipart `file` maks 2 MB -> `{imported,failed,errors[{row,error}]}` |
| FS-11 | `POST /v1/shifts` | admin,manajer | buat shift; overlap -> 409 `shift_conflict` |
| FS-12 | `GET /v1/shifts[?date=&employee_id=]` | semua | staf otomatis miliknya |
| FS-13 | `GET /v1/shifts/:id` | semua | detail |
| FS-14 | `PUT /v1/shifts/:id` | admin,manajer | ubah; 409 bila bentrok |
| FS-15 | `DELETE /v1/shifts/:id` | admin,manajer | hapus; 204 |
| FS-16 | `POST /v1/attendance/check-in` | semua | staf auto-sendiri; duplikat/hari -> 409 |
| FS-17 | `POST /v1/attendance/check-out` | semua | tanpa check-in terbuka -> 404 |
| FS-18 | `GET /v1/attendance[?from=&to=&employee_id=]` | semua | riwayat `YYYY-MM-DD` |
| FS-19 | `GET /v1/reports/overtime[?from=&to=]` | admin,manajer | `[{employee_id,name,total_hours,overtime_hours}]` |
| FS-20 | `GET /v1/reports/coverage[?date=]` | admin,manajer | `[{date,headcount}]` |

## 3. Model Data (MySQL, goose `00001`-`00005`)

- `users(id PK AI, email UNIQUE, password_hash bcrypt, role ENUM admin/manager/staff DEFAULT staff, created_at)`.
- `employees(id PK, user_id UNIQUE NULL FK->users ON DELETE SET NULL, name, email UNIQUE, phone NULL, position DEFAULT 'staff', hire_date DATE, created_at)`.
- `shifts(id PK, employee_id FK->employees CASCADE, date DATE, start_time/end_time TIME, CHECK(end>start), UNIQUE(employee_id,date,start_time))`.
- `attendance(id PK, employee_id FK->employees CASCADE, date DATE, check_in/out DATETIME NULL, UNIQUE(employee_id,date), CHECK(check_out>=check_in))`.
- Relasi: `users 1-0/1 employees`; `employees 1-N shifts, attendance`.

## 4. Aturan Bisnis

1. Overlap shift (pegawai sama, rentang jam beririsan) ditolak 409 - dicek di service.
2. Overtime = `GREATEST(jam_kerja - 8, 0)` per karyawan per hari, zona Asia/Jakarta.
3. Satu check-in per karyawan per hari (`UNIQUE(employee_id,date)`).
4. CSV: header persis `name,email,phone,position,hire_date`; maks 2 MB; parsial (baris valid tetap masuk + daftar error).
5. Staf hanya membaca/mencatat miliknya; admin wajib kirim `employee_id` eksplisit bila bertindak untuk orang lain.

## 5. Matriks RBAC

| Resource | admin | manager | staff |
|---|---|---|---|
| employees tulis / baca | Ya / Ya | Tidak / Ya | Tidak / Tidak |
| shifts tulis / baca | Ya / Ya | Ya / Ya | Tidak / miliknya |
| attendance | kelola | kelola | miliknya |
| reports | Ya | Ya | Tidak |
| impor CSV | Ya | Ya | Tidak |

## 6. Web UI

Halaman: Login, Schedule (board timeline 06:00-24:00 per karyawan, date picker WIB), Shifts (tabel + dialog CRUD per tanggal), Attendance (tombol punch + filter + upload CSV), Reports (bar lembur + tabel coverage). Token di `localStorage["shiftbase_token"]`, validasi via `/me`; 401 tampil sebagai pesan (logout manual). Base URL: `VITE_API_URL` default `http://localhost:8080`.

## 7. ETL / Analitik

Alur `run.py`: extract (pymysql read-only 3 tabel) -> quality gates (gagalkan run bila: tabel kosong, orphan, `check_out<=check_in`, `end<=start`, duplikat) -> 5 marts (`hours_daily`, `overtime_employee`, `coverage_daily`, `attendance_rate`, `late_arrivals` telat > 5 mnt) -> parquet DuckDB -> 4 PNG -> cross-check API overtime toleransi < 0,01 -> `work/quality_report.json`.

## 8. Operasional

Compose: `mysql:8.4` (healthcheck) -> `api` (distroless, `depends_on healthy`, :8080) + `adminer` (:8082). Backup: `mysqldump --single-transaction --routines` + gzip, rotasi 7 hari. Restore: konfirmasi `ya` + verifikasi COUNT employees. Seed: 3 akun (`admin/manager/staff@shiftbase.local`) + 5 karyawan fiktif.
