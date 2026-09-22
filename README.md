# Shiftbase Docs

[![docs](https://github.com/RakhaYandra/shiftbase-docs/actions/workflows/docs-to-pdf.yml/badge.svg)](https://github.com/RakhaYandra/shiftbase-docs/releases)

> Ekosistem: [api](https://github.com/RakhaYandra/shiftbase) · [web](https://github.com/RakhaYandra/shiftbase-web) · [docs](https://github.com/RakhaYandra/shiftbase-docs/releases) · [qa](https://github.com/RakhaYandra/shiftbase-qa) · [data](https://github.com/RakhaYandra/shiftbase-data) · [ops](https://github.com/RakhaYandra/shiftbase-ops)

Dokumentasi resmi sistem **Shiftbase** (Bahasa Indonesia).

**Shiftbase** adalah sistem terpusat untuk penjadwalan shift dan absensi karyawan operasional (kasir, barista, koki). Pengguna: Admin/HR (kelola karyawan & laporan), Manajer (susun roster & pantau absensi), Staf (lihat jadwal & catat absensi). Repo ini adalah sumber dokumen requirements dan desain; kode sumber ada di repo aplikasi (lihat Sumber Fakta).

| Dokumen | Isi |
|---|---|
| [docs/BRD.md](docs/BRD.md) | Business Requirements Document — kebutuhan bisnis |
| [docs/PRD.md](docs/PRD.md) | Product Requirements Document — fitur & acceptance criteria |
| [docs/FSD.md](docs/FSD.md) | Functional Specification Document — spesifikasi fungsi/teknis |
| [docs/SRS.md](docs/SRS.md) | Software Requirements Specification (IEEE 830) |

Sumber fakta (sumber kebenaran per artefak):

| Repo | Artefak yang dirujuk |
|---|---|
| `shiftbase` | `api/swagger.yaml`, `migrations/00001`-`00005`, `docs/ADR-001` (no ORM), `docs/ADR-002` (goose) |
| `shiftbase-web` | 4 halaman role-aware (Jadwal, Shift, Absensi, Laporan) |
| `shiftbase-qa` | `data/testcases.yaml` (38 TC), `data/results.yaml`, Newman, Playwright |
| `shiftbase-data` | ETL 5 marts (hours_daily, overtime_employee, coverage_daily, attendance_rate, late_arrivals) |
| `shiftbase-ops` | `RUNBOOK.md`, `SLA.md`, `TROUBLESHOOTING.md`, `tickets.yaml` |

## PDF

Tiap tag `v*` memicu workflow `docs-to-pdf` → PDF di-upload sebagai **Release asset**.
Unduh versi formal di halaman [Releases](../../releases).

## Versioning

| Versi | Tanggal | Isi |
|---|---|---|
| v1.0.0 | 2026-09-17 | Rilis awal: BRD, PRD, FSD, SRS + PDF |
| v1.1.0 | 2026-09-20 | Penambahan Guideline v1.0: ID BO/SC, flows PRD, spec pointer FSD, ERD, NFR verifikasi |
