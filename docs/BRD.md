# BRD - Shiftbase

| | |
|---|---|
| Versi | 1.0.0 (2026-09-17) |
| Sistem | Shiftbase - penjadwalan shift & absensi |
| Bahasa | Indonesia |

## 1. Latar Belakang

Operasional shift (kasir, barista, koki) umumnya dicatat manual (spreadsheet/chat). Shiftbase menjadi sistem pencatatan terpusat: roster shift, absensi check-in/out, impor karyawan CSV, dan laporan lembur/coverage.

## 2. Masalah Bisnis

1. Jadwal shift disusun dari beberapa spreadsheet/chat sehingga bentrok antar shift baru ketahuan belakangan.
2. Absensi tidak terverifikasi: tidak ada catatan tunggal siapa hadir, kapan check-in/out.
3. Hitung lembur tidak konsisten antar manajer sehingga laporan untuk payroll diperdebatkan.

## 3. Tujuan Bisnis

| ID | Tujuan |
|---|---|
| BO-01 | Satu sumber kebenaran untuk jadwal, kehadiran, dan jam lembur. |
| BO-02 | Mencegah shift bentrok pada saat penyusunan roster. |
| BO-03 | Laporan lembur dan coverage yang siap dipakai payroll. |
| BO-04 | Akses berbasis peran: admin, manajer, staf. |

## 4. Stakeholder

| Peran | Tanggung Jawab | Kepentingan |
|---|---|---|
| Admin/HR | Kelola karyawan & akun, impor data | Laporan penuh yang akurat |
| Manajer | Susun roster, pantau absensi & lembur | Roster benar, tanpa bentrok |
| Staf | Lihat jadwal sendiri, catat absensi | Jadwal jelas, absensi tercatat |
| Owner | Pengawasan operasional | SLA & kelangsungan operasional |

## 5. Ruang Lingkup

**Masuk (IN):** akses berbasis peran (3 peran); CRUD karyawan; CRUD shift anti-bentrok; absensi check-in/out; impor CSV; laporan overtime & coverage; web roster board; analitik offline (marts + charts + quality gates); runbook/SLA/tiket operasional.

**Keluar (OUT):** deployment/infra produksi, performance testing, payroll payout, mobile app, shift overnight (web board 06:00-24:00).

## 6. Kebutuhan Bisnis

| ID | Kebutuhan | Prioritas |
|---|---|---|
| BR-01 | Sistem mencatat karyawan dengan identitas unik (email) | Must |
| BR-02 | Roster shift per karyawan per tanggal tanpa bentrok | Must |
| BR-03 | Absensi 1 check-in per karyawan per hari + check-out | Must |
| BR-04 | Impor massal karyawan via CSV (maks 2 MB) dengan laporan baris gagal | Must |
| BR-05 | Laporan jam lembur (> 8 jam/hari) per karyawan | Must |
| BR-06 | Laporan coverage (headcount) per tanggal | Must |
| BR-07 | Hak akses berbeda untuk admin/manajer/staf | Must |
| BR-08 | Analitik offline dengan quality gates | Should |
| BR-09 | Runbook, SLA, dan tiket insiden terdokumentasi | Should |

## 7. Aturan Bisnis

1. Shift yang tumpang tindih (pegawai sama, rentang jam beririsan) wajib ditolak.
2. Lembur dihitung dari jam kerja melebihi 8 jam/hari, zona Asia/Jakarta.
3. Satu pencatatan kehadiran per karyawan per hari.

## 8. Risiko

| Risiko | Dampak | Kemungkinan | Mitigasi |
|---|---|---|---|
| Zona waktu tak konsisten (WIB) | Tinggi | Sedang | Semua tanggal diproses Asia/Jakarta; quality gate menolak data inkonsisten |
| Staf baru tanpa link karyawan -> absensi kosong | Sedang | Sedang | By design; admin wajib link akun |
| Kredensial akses bocor | Tinggi | Rendah | Secret minimal 32 char, `.env` di-gitignore, rotasi berkala |
| Data hilang (`down -v`) | Tinggi | Rendah | Backup harian gzip retensi 7 hari + drill restore bulanan |

## 9. SLA (ringkas, detail di repo `-ops`)

| Severity | Contoh | Respons | Resolusi |
|---|---|---|---|
| High | Absensi/laporan mati, data hilang | 1 jam | 4 jam |
| Medium | Sebagian fitur mati, deploy gagal | 4 jam | 1 hari |
| Low | Kosmetik, pertanyaan | 2 hari | 2 pekan |

Eskalasi +1 level bila respons terlampaui; postmortem wajib untuk High.

## 10. Kriteria Sukses

| ID | Kriteria |
|---|---|
| SC-01 | 38/38 test case QA Pass, Newman 15/15, Playwright 8/8 (tercapai 2026-09-15). |
| SC-02 | 0 bug Critical/High terbuka. |
| SC-03 | CI hijau di 10 repo (backend, web, qa, data, ops x 2 sistem). |
