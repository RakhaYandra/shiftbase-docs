# SRS - Shiftbase (IEEE 830)

| | |
|---|---|
| Versi | 1.0.0 (2026-09-17) |
| Acuan | BRD, PRD, FSD v1.0.0 |

## 1. Pendahuluan

### 1.1 Tujuan

SRS ini menetapkan kebutuhan perangkat lunak sistem Shiftbase (API + web + QA + data + ops) sebagai acuan pembangunan, pengujian, dan serah terima. Audiens: developer, QA, operator, stakeholder.

### 1.2 Ruang Lingkup

Produk bernama **Shiftbase**: REST API penjadwalan shift & absensi (Go/Gin + MySQL + JWT + Docker) beserta web roster board, suite QA, ETL analitik, dan paket operasional. Manfaat: roster tanpa bentrok, absensi terverifikasi, laporan lembur/coverage siap payroll. Konsisten dengan BRD/FSD.

### 1.3 Definisi & Singkatan

| Istilah | Arti |
|---|---|
| Roster | Jadwal shift karyawan per tanggal |
| Coverage | Jumlah karyawan terjadwal (headcount) per tanggal |
| Overtime | Jam kerja melebihi 8 jam/hari |
| RBAC | Role-Based Access Control (admin/manager/staff) |
| JWT | JSON Web Token, masa berlaku 24 jam |
| ETL | Extract, Transform, Load |
| WIB | Asia/Jakarta (UTC+7) |

### 1.4 Referensi

1. BRD v1.0.0, PRD v1.0.0, FSD v1.0.0 (repo ini).
2. `api/swagger.yaml`, `migrations/00001-00005`, ADR-001 (no ORM), ADR-002 (goose) - repo `shiftbase`.
3. `test-plan.md`, `data/testcases.yaml` (38 TC) - repo `shiftbase-qa`.
4. `RUNBOOK.md`, `SLA.md`, `TROUBLESHOOTING.md`, `tickets.yaml` - repo `shiftbase-ops`.

### 1.5 Overview

Bagian 2 deskripsi umum; bagian 3 kebutuhan rinci (antarmuka, fungsional FR, non-fungsional NFR, batasan CON); bagian 4 glosarium; bagian 5 referensi; bagian 6 matriks traceability.

## 2. Deskripsi Umum

### 2.1 Perspektif Produk

Sistem mandiri yang terdiri dari: API (`:8080`), MySQL 8.4, web (`:5173`), pipeline ETL offline, dan tooling QA/ops. Antarmuka: REST JSON; komunikasi HTTP; operasi normal via docker compose; instalasi via runbook (migrate + seed).

### 2.2 Fungsi Produk (ringkas)

Autentikasi JWT + RBAC; CRUD karyawan; roster shift anti-bentrok; absensi check-in/out; impor CSV; laporan overtime & coverage; roster board web; analitik parquet/grafik; runbook/SLA/tiket.

### 2.3 Karakteristik Pengguna

Admin/HR dan manajer: melek spreadsheet & operasional shift. Staf: pengguna kasual (login, lihat jadwal, punch). Tak ada kebutuhan aksesibilitas khusus di v1.0.

### 2.4 Lingkungan Operasi

| Aspek | Nilai |
|---|---|
| Runtime API | Go/Gin, container distroless |
| Database | MySQL 8.4 (container, healthcheck) |
| Web | Vite + React 19 + TS (dev `:5173`) |
| Tooling QA | Node 22 (Newman, Playwright) |
| Tooling data | Python 3.13 + pandas/duckdb/matplotlib |
| Zona waktu | Asia/Jakarta (UTC+7) di semua komponen |
| Orkestrasi | Docker Compose (api, mysql, adminer) |

### 2.5 Batasan

- CON-TECH-01: MySQL 8.4; Go; timezone Asia/Jakarta.
- CON-TECH-02: upload CSV maks 2 MB.
- CON-SEC-01: JWT_SECRET >= 32 char di produksi; `.env` tidak di-commit.
- CON-STD-01: mengikuti FSD bagian 2-bagian 5.

### 2.6 Asumsi & Dependensi

- Jam bisnis fixed UTC+7 di semua komponen.
- Staf register baru tanpa link karyawan -> absensi kosong (by design).
- Bergantung pada: image `mysql:8.4`, Go toolchain (CI), Node 22 (CI web/QA), Python 3.13 + pandas/duckdb/matplotlib (CI data).

## 3. Kebutuhan Khusus

### 3.1 Kebutuhan Antarmuka Eksternal

| ID | Kebutuhan |
|---|---|
| FR-INT-01 | API wajib REST JSON dengan kode status & body error terdokumentasi (FSD bagian 2). |
| FR-INT-02 | Web wajib membaca base URL dari `VITE_API_URL` dan mengirim `Authorization: Bearer`. |
| FR-INT-03 | ETL wajib read-only terhadap DB operasional dan menulis ke DuckDB/parquet terpisah. |

### 3.2 Kebutuhan Fungsional

| ID | Kebutuhan (M = wajib) | Prioritas | Sumber |
|---|---|---|---|
| FR-AUTH-01 | Sistem wajib menerbitkan JWT 24 jam saat login valid. | M | US-02 |
| FR-AUTH-02 | Sistem wajib menolak token tak valid dengan 401 `unauthorized`. | M | US-03 |
| FR-AUTH-03 | Sistem wajib menegakkan matriks RBAC (FSD bagian 5); pelanggaran -> 403 `forbidden`. | M | US-03 |
| FR-EMP-01 | Sistem wajib CRUD karyawan dengan email unik. | M | US-04 |
| FR-SHIFT-01 | Sistem wajib menolak shift yang overlap (pegawai sama) dengan 409 `shift_conflict`. | M | US-05 |
| FR-SHIFT-02 | Sistem wajib memfilter daftar shift staf ke miliknya. | M | US-06 |
| FR-ATT-01 | Sistem wajib membatasi 1 check-in/karyawan/hari; duplikat -> 409. | M | US-07 |
| FR-ATT-02 | Sistem wajib menolak check-out tanpa check-in terbuka dengan 404. | M | US-07 |
| FR-IMP-01 | Sistem wajib impor CSV (header tetap, <= 2 MB) dan melaporkan `{imported,failed,errors[]}`. | M | US-09 |
| FR-RPT-01 | Sistem wajib menghitung overtime `GREATEST(jam-8,0)`/hari zona WIB. | M | US-10 |
| FR-RPT-02 | Sistem wajib menyajikan coverage headcount per tanggal. | M | US-10 |
| FR-WEB-01 | Web wajib menampilkan 4 halaman sesuai peran + board timeline WIB. | M | US-11/12 |
| FR-DATA-01 | ETL wajib menggagalkan run bila quality gate gagal. | S | US-13 |
| FR-OPS-01 | Sistem wajib menyediakan backup gzip rotasi 7 hari + restore terverifikasi. | S | US-14 |

### 3.3 Kinerja

| ID | Kebutuhan | Verifikasi | Bukti |
|---|---|---|---|
| NFR-PERF-01 | `GET /healthz` wajib 200 pada boot normal setelah MySQL healthy. | Uji boot compose | Log compose + `RUNBOOK.md` repo `-ops` |
| NFR-PERF-02 | Suite QA wajib: 38/38 TC Pass, Newman 15/15, Playwright 8/8, coverage service >= 60%. | CI QA | `data/results.yaml`, `reports/newman.json`, `reports/playwright.json` repo `-qa` |

### 3.4 Atribut Sistem

| ID | Kebutuhan | Verifikasi | Bukti |
|---|---|---|---|
| NFR-SEC-01 | Password wajib bcrypt; JWT wajib secret >= 32 char di produksi. | Review kode | `internal/` repo `shiftbase`, `.env.example` |
| NFR-SEC-02 | Secret (`.env`, token, password) wajib tidak ter-commit (dukungan: `redact_reports.py`). | CI + review | `tools/redact_reports.py` repo `-qa` |
| NFR-REL-01 | Backup harian + drill restore bulanan; postmortem wajib untuk insiden High. | Drill ops | `RUNBOOK.md`, `tickets.yaml` repo `-ops` |
| NFR-MAINT-01 | Skema DB wajib berversi via migrasi goose; tanpa ORM (ADR-001). | Review migrasi | `migrations/00001`-`00005` repo `shiftbase` |
| NFR-USA-01 | Web berbahasa Indonesia, desain datar tanpa gradien/shadow/emoji. | Review UI | Repo `shiftbase-web` |
| NFR-AVAIL-01 | MySQL wajib healthcheck dan API wajib `depends_on healthy` agar boot berurutan; service wajib restart otomatis bila mati. | Uji compose | `docker-compose.yml` repo `shiftbase` |

## 4. Glosarium

Lihat bagian 1.3.

## 5. Referensi

Lihat bagian 1.4.

## 6. Matriks Traceability

| BRD | PRD | FSD | SRS | Uji (shiftbase-qa) |
|---|---|---|---|---|
| BR-07 | US-01-03 | FS-02-04 | FR-AUTH-01-03 | TC AUTH (3), RBAC (6) |
| BR-01 | US-04 | FS-05-09 | FR-EMP-01 | TC EMP (3) |
| BR-02 | US-05/06/11 | FS-11-15 | FR-SHIFT-01/02 | TC SHIFT (5), WEB (6) |
| BR-03 | US-07/08 | FS-16-18 | FR-ATT-01/02 | TC ATT (4) |
| BR-04 | US-09 | FS-10 | FR-IMP-01 | TC IMP (6) |
| BR-05/06 | US-10 | FS-19/20 | FR-RPT-01/02 | TC REP (2), Newman |
| BR-08 | US-13 | FSD bagian 7 | FR-DATA-01, FR-INT-03 | quality gates, cross-check |
| BR-09 | US-14 | FSD bagian 8 | FR-OPS-01 | runbook drill, tickets |

Checklist validasi: semua bagian IEEE 830 terisi; semua kebutuhan ber-ID unik dan terverifikasi; istilah terdefinisi; matriks lengkap.
