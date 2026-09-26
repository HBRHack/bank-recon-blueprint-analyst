# Stakeholder Register: Bank Reconciliation & Statement Mapping System

Sistem ini menyentuh Finance/Operations, Auditor Internal, kepatuhan, dan 4 bank
partner — register dibuat di awal discovery sebagai sumber kolom `Source` di
requirements matrix dan input RACI.

## Register

| ID | Nama / Peran | Organisasi | Tipe | Influence | Interest | Kebutuhan dari Sistem | Cara Libatkan |
|----|--------------|------------|------|-----------|----------|----------------------|---------------|
| SH-001 | Manajer Keuangan | Internal (Finance) | Sponsor | High | High | Selesai rekonsiliasi ≤07:30, selisih tak terdeteksi <Rp 5 jt/bulan, jejak audit 10 tahun | Sign-off scope & NFR kritis; review bulanan |
| SH-002 | Supervisor Keuangan | Internal (Finance) | User (Approver) | High | High | Persetujuan koreksi & tutup hari, peringatan bertingkat, register selisih + aging | Interview + workshop aturan carry-over |
| SH-003 | Finance Ops Analyst (3 orang) | Internal (Finance) | End User | Med | High | Pencocokan otomatis, saran kandidat, penyelesaian selisih cepat, tanpa kerja ganda | Observasi 1 hari (proses 06:30–10:00) + UAT |
| SH-004 | Auditor Internal | Internal (Risk/Compliance) | Regulator (internal) | High | High | Laporan utuh, register selisih + siapa menyelesaikan, unduh jejak audit, kategori non-matching tetap terlihat | Review dokumen + sign-off bukti audit |
| SH-005 | System Analyst | Internal (Tech) | Tech Owner / Config | High | Med | Ubah parameter aturan tanpa menyentuh logika program; jejak perubahan | Interview aturan pencocokan; pemilik `nfr.md` |
| SH-006 | Tim Operasional TI | Internal (Tech) | Ops | Med | Med | Pemantauan run, peringatan, pemulihan ≤60 menit, jadwal pemeliharaan di luar jendela kritis | Koordinasi jendela cut-off & drill pemulihan |
| SH-007 | Bank Partner (4 bank) | Eksternal | Vendor / Sumber data | Med | Low | Format file jelas, jadwal 05:00, aturan portal 30 hari | Interface analysis per bank; perjanjian kanal |
| SH-008 | OJK | Eksternal | Regulator | High | Low | Retensi 10 tahun, audit trail tidak bisa diubah, pelaporan sesuai ketentuan | Bedah dokumen ketentuan; bukti kepatuhan |
| SH-009 | Tim Klaim / Divisi Merchant | Internal | User (Penerima Output) | Low | Med | Selisih settlement merchant terselesaikan <4 jam | Konfirmasi ambang eskalasi |
| SH-010 | Merchant / Nasabah layanan | Eksternal | Customer (tidak langsung) | Low | Med | Settlement terkonfirmasi tepat waktu | Representasi via Tim Klaim (tidak dilibatkan langsung) |
| SH-011 | Proyek / PM | Internal | PM | Med | Med | Jadwal rilis 2 tahap, risiko, kelengkapan dokumen | Status review mingguan |

> **Sweep 11 peran generik (BABOK Ch 2):** analis bisnis → SH-005 ·
> pengguna langsung → SH-002, SH-003 · ahli domain → SH-002 ·
> sponsor/penyandang dana → SH-001 · manajer proyek → SH-011 ·
> tim operasional → SH-006 · ahli implementasi/teknis → SH-005, SH-006 ·
> tester/QA → **belum ada (gap)** · regulator → SH-008 ·
> supplier/vendor → SH-007 · customer → SH-009/SH-010.
> Satu orang boleh >1 peran (Supervisor kadang menjadi analis saat cuti →
> dicatat sebagai `acting_as` di jejak audit).

## Influence × Interest Map

```
              LOW Interest        HIGH Interest
            ┌──────────────────┬──────────────────┐
 HIGH       │ MONITOR          │  MANAGE CLOSELY  │
 Influence  │ (info berkala)   │  (libatkan aktif)│
            │ SH-007, SH-008   │  SH-001, SH-002  │
            │                  │  SH-004, SH-005  │
            ├──────────────────┼──────────────────┤
 LOW        │ MONITOR          │  KEEP INFORMED   │
 Influence  │ (minim effort)   │  (update rutin)  │
            │ SH-010           │  SH-003, SH-006  │
            │                  │  SH-009, SH-011  │
            └──────────────────┴──────────────────┘
```

## Key Decisions per Stakeholder

| Stakeholder | Keputusan yang Butuh Mereka | Kapan Dibutuhkan |
|-------------|----------------------------|------------------|
| SH-001 Manajer Keuangan | Sign-off scope & sasaran (OBJ-001…005), NFR kritis, koreksi >Rp 5 juta | Kickoff + akhir tiap fase |
| SH-002 Supervisor Keuangan | Aturan carry-over (0,5% / 90 baris), ambang eskalasi (90/200), tutup hari | Sebelum rilis 1 |
| SH-003 Finance Ops Analyst | Kelayakan pakai layar penanganan selisih + UAT exit | Desain rilis 1 & UAT |
| SH-004 Auditor Internal | Kelengkapan bukti audit, kewajiban tampilnya kategori non-matching | Review dokumen sebelum final |
| SH-005 System Analyst | Daftar parameter aturan yang boleh diubah + struktur NFR | Desain rilis 1 & 2 |
| SH-006 Tim Operasional TI | Jendela pemeliharaan & jalur pemulihan | Sebelum go-live |
| SH-007 Bank Partner | Jadwal & format kanal; aturan portal 30 hari | Interface analysis |
| SH-008 OJK | Interpretasi retensi & jejak audit | Kepatuhan (paralel) |
| SH-011 PM | Urutan rilis & penundaan BAB 1–2 | Perubahan scope |

## Rules

- Setiap requirement di `00-Global/REQUIREMENTS-MATRIX.md` wajib bisa dilacak ke
  minimal satu `SH-XXX` lewat kolom `Source`.
- SH-001, SH-002, SH-004, SH-005 (High × High) **wajib** ada di sign-off checklist.
- Regulator (SH-008) & vendor (SH-007) sengaja dieksplisitkan — dua tipe paling
  sering terlupa.
- **Gap terbuka:** belum ada peran tester/QA — calon SH-012, isi saat tim QA
  sudah ada (lihat `00-Global/REQUIREMENTS-MATRIX.md` §Gaps).
- Update register bila muncul stakeholder baru — jangan sampai ada yang protes
  baru saat UAT.
