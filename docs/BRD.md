# BRD - Shiftbase

| | |
|---|---|
| Versi | 1.0.0 (2026-09-17) |
| Sistem | Shiftbase - penjadwalan shift & absensi |
| Bahasa | Indonesia |

## 1. Latar Belakang

Operasional shift (kasir, barista, koki) umumnya dicatat manual (spreadsheet/chat) sehingga rawan: jadwal bentrok, absensi tak terverifikasi, dan hitung lembur tidak konsisten. Shiftbase menjadi sistem pencatatan terpusat: roster shift, absensi check-in/out, impor karyawan CSV, dan laporan lembur/coverage.

## 2. Tujuan Bisnis

1. Satu sumber kebenaran untuk jadwal, kehadiran, dan jam lembur.
2. Mencegah shift bentrok pada saat penyusunan roster.
3. Laporan lembur dan coverage yang siap dipakai payroll.
4. Akses berbasis peran: admin, manajer, staf.

## 3. Stakeholder

| Peran | Kepentingan |
|---|---|
| Admin/HR | Kelola karyawan & akun, laporan penuh |
| Manajer | Susun roster, pantau absensi & lembur |
| Staf | Lihat jadwal sendiri, catat absensi |
| Owner | SLA & kelangsungan operasional |

## 4. Ruang Lingkup

**Masuk (IN):** autentikasi JWT + RBAC 3 peran; CRUD karyawan; CRUD shift anti-bentrok; absensi check-in/out; impor CSV; laporan overtime & coverage; web roster board; ETL analitik (marts + charts + quality gates); runbook/SLA/tiket operasional.

**Keluar (OUT):** deployment/infra produksi, performance testing, payroll payout, mobile app, shift overnight (web board 06:00-24:00).

## 5. Kebutuhan Bisnis

| ID | Kebutuhan | Prioritas |
|---|---|---|
| BR-01 | Sistem mencatat karyawan dengan identitas unik (email) | Must |
| BR-02 | Roster shift per karyawan per tanggal tanpa bentrok | Must |
| BR-03 | Absensi 1 check-in per karyawan per hari + check-out | Must |
| BR-04 | Impor massal karyawan via CSV (maks 2 MB) dengan laporan baris gagal | Must |
| BR-05 | Laporan jam lembur (> 8 jam/hari) per karyawan | Must |
| BR-06 | Laporan coverage (headcount) per tanggal | Must |
| BR-07 | Hak akses berbeda untuk admin/manajer/staf | Must |
| BR-08 | Analitik offline (parquet + grafik) dengan quality gates | Should |
| BR-09 | Runbook, SLA, dan tiket insiden terdokumentasi | Should |

## 6. Risiko

| Risiko | Mitigasi |
|---|---|
| Zona waktu tak konsisten (WIB) | Semua tanggal diproses Asia/Jakarta; quality gate menolak data inkonsisten |
| Staf baru tanpa link karyawan -> absensi kosong | By design; admin wajib link akun |
| Secret JWT bocor | JWT >= 32 char, `.env` di-gitignore, rotasi berkala |
| Data hilang (`down -v`) | Backup harian gzip retensi 7 hari + drill restore bulanan |

## 7. SLA (ringkas, detail di repo `-ops`)

| Severity | Contoh | Respons | Resolusi |
|---|---|---|---|
| High | Absensi/laporan mati, data hilang | 1 jam | 4 jam |
| Medium | Sebagian fitur mati, deploy gagal | 4 jam | 1 hari |
| Low | Kosmetik, pertanyaan | 2 hari | 2 pekan |

Eskalasi +1 level bila respons terlampaui; postmortem wajib untuk High.

## 8. Kriteria Sukses

1. 38/38 test case QA Pass, Newman 15/15, Playwright 8/8 (tercapai 2026-09-15).
2. 0 bug Critical/High terbuka.
3. CI hijau di 10 repo (backend, web, qa, data, ops x 2 sistem).
