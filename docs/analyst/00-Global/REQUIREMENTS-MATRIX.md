# Requirements Matrix — Master

Traceability 23 FR + 20 NFR proyek Bank Reconciliation, terhubung ke sumber
(stakeholder/jawaban/gate), alasan bisnis, aturan, dan dokumen. **BAB 1 & 2
ditunda** — kolom `Chapter` seluruhnya `00-Global` dan akan diperbarui saat
struktur bab disusun.

## Requirements by Priority

### Must Have

| ID | Requirement | Source | Rationale | Chapter | Use Case | Business Rules | Status | Owner |
|----|-------------|--------|-----------|---------|----------|----------------|--------|-------|
| FR-001 | Penerimaan mutasi terjadwal (3 otomatis + 1 manual) | SH-003, SH-007 (aspek D) | Tanpa data masuk, rekonsiliasi tidak jalan sama sekali | 00-Global | UC-01, UC-12 | `BR-CON-004`, `BR-WF-008` | [x] Confirmed | SH-006 |
| FR-002 | Penanda keterlambatan statement, proses parsial | SH-002 (aspek A/D) | Satu bank telat tidak boleh menghentikan seluruh proses | 00-Global | UC-01, UC-12 | `BR-WF-008`, `BR-WF-009` | [x] Confirmed | SH-006 |
| FR-003 | Validasi struktur & control total per batch | SH-004 (audit) | Data rusak/ganda harus tertangkap sebelum mengotori hasil | 00-Global | UC-01 | `BR-VAL-001` | [ ] Pending | SH-005 |
| FR-004 | Normalisasi kolom lintas bank | SH-003 (aspek B) | Beda penulisan referensi antar bank menaikkan selisih palsu | 00-Global | UC-02, UC-12 | `BR-VAL-002` | [x] Confirmed | SH-005 |
| FR-005 | Pencocokan berjenjang T1→T5 | Gate G-01 (SH-002) | Inti otomasi — mengganti filter manual | 00-Global | UC-02 | `BR-VAL-003…007`, `BR-CALC-001` | [x] Confirmed | SH-005 |
| FR-006 | Dukungan agregasi & split (M:N) | Gate G-02 (SH-002) | Settlement & biaya terpisah tidak akan pernah cocok dengan pola 1:1 | 00-Global | UC-02 | `BR-VAL-005`, `BR-VAL-008` | [x] Confirmed | SH-005 |
| FR-007 | Anti pasangan ganda | Gate G-03 (SH-004) | Pasangan ganda menyembunyikan selisih nyata | 00-Global | UC-03 | `BR-CON-001` | [x] Confirmed | SH-005 |
| FR-009 | Pengkodean 8 alasan selisih | SH-003, SH-004 (aspek B + masukan #3) | Selisih tanpa alasan tidak bisa ditindaklanjuti | 00-Global | UC-03 | `BR-VAL-009`, `BR-VAL-011` | [x] Confirmed | SH-005 |
| FR-010 | Kategori non-matching tetap tampil di laporan | SH-004 (masukan #2) | Menyembunyikan volume selisih wajar = red flag audit | 00-Global | UC-09 | `BR-VAL-010` | [x] Confirmed | SH-004 |
| FR-011 | Aging & penugasan selisih | SH-002 (aspek A) | Tanpa umur & pemilik, selisih menumpuk tanpa terdeteksi | 00-Global | UC-03 | `BR-WF-001`, `BR-CALC-003` | [ ] Pending | SH-002 |
| FR-012 | Pencocokan manual oleh analis | SH-003 (aspek C) | Selisih yang lolos otomatis tetap harus bisa diselesaikan | 00-Global | UC-03 | `BR-WF-002` | [x] Confirmed | SH-003 |
| FR-013 | Koreksi dengan persetujuan dua orang | SH-001 (gate G-04, CON-003) | Mencegah selisih ditutup sepihak | 00-Global | UC-04, UC-05 | `BR-WF-003`, `BR-AUTH-001`, `BR-AUTH-002` | [x] Confirmed | SH-002 |
| FR-014 | Koreksi menjadi baris baru (tanpa ubah data lama) | SH-004 (masukan #4) | Menjaga jejak & menghindari hitung ganda | 00-Global | UC-05 | `BR-CON-002` | [x] Confirmed | SH-005 |
| FR-015 | Ringkasan rekonsiliasi harian + control total | SH-001, SH-004 (aspek A/B) | Bukti selesainya hari & bahan tutup buku | 00-Global | UC-06, UC-09 | `BR-RET-003` | [x] Confirmed | SH-002 |
| FR-016 | Register selisih + aging (termasuk non-matching) | SH-004 (aspek A) | Auditor harus melihat sisa kerja & alasannya | 00-Global | UC-09 | `BR-VAL-010`, `BR-CALC-003` | [x] Confirmed | SH-004 |
| FR-017 | Peringatan bertingkat 90 (WASPADA) / 200 (KRITIS) | Keputusan #1 user + SH-002 | Deteksi dini sebelum batas carry-over terlampaui | 00-Global | UC-07 | `BR-WF-004`, `BR-WF-009` | [x] Confirmed | SH-006 |
| FR-018 | Ekspor laporan & jejak audit untuk Auditor | SH-004 (aspek D) | Kebutuhan bukti audit on-demand | 00-Global | UC-10 | `BR-AUTH-004`, `BR-AUTH-007`, `BR-RET-001` | [x] Confirmed | SH-004 |
| FR-019 | Pengaturan parameter aturan + persetujuan | SH-005 (aspek F) | Toleransi berubah tanpa menyentuh logika program | 00-Global | UC-08, UC-11 | `BR-WF-005`, `BR-AUTH-003` | [x] Confirmed | SH-005 |
| FR-021 | Penutupan hari rekonsiliasi oleh Supervisor | SH-001, SH-002 (aspek A/C) | Titik pertanggungjawaban harian + ambang carry-over | 00-Global | UC-06 | `BR-WF-006`, `BR-CALC-004` | [x] Confirmed | SH-002 |
| FR-022 | Jejak audit append-only | SH-004, SH-008 (gate G-05) | Ketentuan audit 10 tahun, tanpa ini lulus pemeriksaan tidak mungkin | 00-Global | UC-10 | `BR-CON-003`, `BR-AUTH-005` | [x] Confirmed | SH-005 |

### Should Have

| ID | Requirement | Source | Rationale | Chapter | Use Case | Business Rules | Status | Owner |
|----|-------------|--------|-----------|---------|----------|----------------|--------|-------|
| FR-008 | Skor keyakinan & antrean review | SH-003 (derivasi) | Mengurangi waktu memilih kandidat yang salah | 00-Global | UC-02, UC-03 | `BR-CALC-002`, `BR-VAL-006` | [ ] Pending | SH-005 |
| FR-020 | Versi aturan per run | SH-004 (audit) | Hasil lampau tetap bisa dijelaskan setelah aturan berubah | 00-Global | UC-08 | `BR-RET-002`, `BR-WF-005` | [ ] Pending | SH-005 |
| FR-023 | Catatan rangkap peran (`acting_as`) | SH-003 (aspek C) | Pertanggungjawaban tetap jelas saat satu orang >1 peran | 00-Global | UC-03, UC-06 | `BR-AUTH-008` | [x] Confirmed | SH-005 |

> **FR-001 – FR-023** lengkap di [`00-Global/SRS-MASTER.md`](SRS-MASTER.md) §2.
> **Use Case** dirujuk ke `DESAIN-PROGRAM.md` §3 (UC-01…UC-12);
> **Business Rules** memakai 6 kode kategori (`VAL·CALC·AUTH·WF·CON·RET`),
> dirujuk ke `DESAIN-PROGRAM.md` §6 — katalog penuh menyusul di
> `business-rules.md`.

## Full Traceability Matrix (NFR)

| ID | Requirement (ringkas) | Source | Rationale | Related FR | Verification | Status |
|----|----------------------|--------|-----------|-----------|--------------|--------|
| NFR-PERF-001 | Proses 18.000 baris ≤30 menit | SH-001 (aspek E) | Jendela cut-off 06:00–06:30 | FR-001, FR-005, FR-006 | Uji beban ≥5× | Confirmed |
| NFR-PERF-002 | Volume H+1 3× selesai sebelum 07:00 | SH-001 (aspek E) | Tutup bulan tidak boleh lebih buruk | FR-005, FR-017 | Uji beban H+1 | Confirmed |
| NFR-PERF-003 | Layar selisih <2 detik p95, 5 pengguna | SH-003 (aspek E) | 5 orang kerja bersamaan jam 07:00 | FR-011, FR-012 | Pengukuran p95 | Pending |
| NFR-PERF-004 | Ekspor ≤2 menit untuk 1 bulan data | SH-004 (aspek D) | Auditor butuh bukti cepat | FR-015, FR-018 | Pengukuran ekspor | Pending |
| NFR-AVAIL-001 | Jendela 05:00–07:30 ≥99,9% | SH-001 (aspek E) | Satu-satunya waktu proses boleh terjadi | FR-001, FR-002 | Pemantauan SLA | Confirmed |
| NFR-AVAIL-002 | RTO ≤60 menit, RPO 0, fallback >06:30 | SH-001, SH-006 (aspek E) | Gagal = rekonsiliasi hari itu tertunda | FR-002, FR-017 | Drill pemulihan | Confirmed |
| NFR-SEC-001 | 2FA + kunci setelah 5× gagal | SH-005 (aspek E) | Akun Finance = akses data keuangan | FR-012–FR-014, FR-022 | Uji bypass | Confirmed |
| NFR-SEC-002 | Kendali akses per peran | SH-002, SH-004 (aspek C) | Pemisahan tugas wajib | FR-012, FR-013, FR-019, FR-021 | Uji matriks izin | Confirmed |
| NFR-SEC-003 | Jejak audit tak-terubah + segel rantai | SH-004, SH-008 (gate G-05) | Perusakan jejak harus terdeteksi | FR-022, FR-023 | Uji perubahan jejak | Confirmed |
| NFR-SEC-004 | Pelindungan data pribadi pada kolom keterangan | SH-004 (CON-007) | UU PDP | FR-018 | Pemeriksaan per peran | Pending |
| NFR-COMP-001 | Retensi 10 tahun utuh | SH-008 (CON-001) | Ketentuan pengawasan | FR-015, FR-018, FR-022 | Daftar kepatuhan + uji arsip | Confirmed |
| NFR-COMP-002 | Kepatuhan UU PDP | SH-004 (CON-007) | Kewajiban hukum | FR-014, FR-022 | Daftar kepatuhan | Pending |
| NFR-DATA-001 | Idempoten: run ulang tanpa ganda | Gate G-06 (SH-002) | Koreksi manual tidak boleh hilang | FR-005, FR-014 | Uji run ulang ≥3× | Confirmed |
| NFR-DATA-002 | Baris mentah tak bisa diubah/dihapus | SH-004 (CON-001) | Integritas bukti | FR-012, FR-014 | Uji larangan tulis | Confirmed |
| NFR-DATA-003 | Cadangan + uji pemulihan kuartalan | SH-006 | Pemulihan bencana | — | Uji cadangan | Pending |
| NFR-SCALE-001 | Kapasitas Y1 30.000 / Y3 60.000 baris | SH-001 (aspek E) | Pertumbuhan tanpa redesain | FR-005, FR-006 | Uji beban bertingkat | Pending |
| NFR-USE-001 | Pelatihan 2 hari → produktif | SH-003 (aspek E) | Masa transisi | FR-011, FR-012 | Pengamatan pasca-latih | Confirmed |
| NFR-USE-002 | Kandidat menampilkan alasan & titik beda | SH-003 (derivasi) | Kurangi salah pilih | FR-008, FR-012 | Pengujian pemakaian | Pending |
| NFR-MAINT-001 | Ubah aturan tanpa ubah logika program | SH-005 (aspek F) | Toleransi sering berubah | FR-019, FR-020 | Uji perubahan E2E | Pending |
| NFR-MAINT-002 | Pemantauan run + peringatan otomatis | SH-002, SH-006 (aspek D/E) | Keterlambatan harus ketahuan sebelum cut-off | FR-017, FR-002 | Uji pengiriman peringatan | Confirmed |

## Coverage Summary

| Area | Total | Confirmed | Pending | N/A |
|------|-------|-----------|---------|-----|
| Functional Requirements | 23 | 17 | 6 | 0 |
| Non-Functional | 20 | 12 | 8 | 0 |
| Assumptions | 6 | 2 (verified) | 4 (open) | 0 |
| Constraints | 9 | 9 (tercatat) | 0 | 0 |
| Business Rules | 40 (`BR-*`, dirujuk di `DESAIN-PROGRAM.md` §6) | — | 40 (menunggu katalog `business-rules.md`) | 0 |

## Gaps and Risks

| Gap | Impact | Mitigation |
|-----|--------|------------|
| **Belum ada `business-rules.md`** — 40 aturan `BR-*` (6 kategori `VAL·CALC·AUTH·WF·CON·RET`) sudah dirujuk per FR di `DESAIN-PROGRAM.md` §6, tetapi katalog penuh (Rule/When/Then/Else/Source + kasus uji) belum ditulis | FR tanpa aturan yang bisa ditest = tidak bisa diimplementasikan | Tulis `business-rules.md` dengan ID & bunyi **identik** §6 |
| **BAB 1 & BAB 2 ditunda** (pendahuluan & analisis situasi) — Problem Statement, Scope, Actor & Use Case sementara menetap di `DESAIN-PROGRAM.md` §1–§3 | Struktur bab formal belum ada; kolom `Chapter` semua `00-Global` | Susun BAB 1–2 memakai `analyst-grill-with-docs` setelah desain beres, tanpa mengulang isi §1–§3 |
| **Belum ada peran tester/QA** (stakeholder-register: gap SH-012) | Tidak ada pemilik untuk strategi pengujian & UAT exit | Tambahkan saat tim QA terbentuk |
| **Belum ada nama & tanda tangan pemilik dokumen** (OQ-003) | Sign-off buntu di akhir | Konfirmasi Manajer Keuangan sebagai Business Owner |
| **OQ-001**: proporsi non-matching belum diukur (ASM-002) | Target "selesai 20 menit" bisa tidak realistis | Sampel 1 bulan sebelum komitmen target |
| **OQ-002**: cara pengiriman koreksi ke ledger belum pasti | FR-014 bisa berubah jadi terjadwal vs langsung | Konfirmasi System Analyst |
| **OQ-004**: perilaku restorasi bila run gagal setengah jalan belum dikonfirmasi stakeholder | Risiko data setengah proses | Jawaban desain sementara: **jalur ulang idempoten** `BR-WF-007` (koreksi manual dipertahankan) — konfirmasi ke SH-001 |
| ASM-001/004/005 masih **Open** | Volume & ketersediaan data historis belum terverifikasi | Jadwal review minggu ke-1 (`assumptions-constraints.md`) |

## Status Legend

| Status | Meaning |
|--------|---------|
| [x] Confirmed | Sudah dikonfirmasi dari jawaban question-framework / gate |
| [ ] Pending | Turunan analis, belum divalidasi ke stakeholder |
| In Progress | Sedang diimplementasikan |
| Deprecated | Tidak lagi berlaku |
| N/A | Tidak berlaku untuk sistem ini |

## Dokumen Core yang Sudah Ada

| Dokumen | Lokasi | Isi |
|---------|--------|-----|
| Notasi (GLOBAL-00) | `00-Global/NOTATION.md` | Matriks 6 baris + aturan NO MERMAID |
| Struktur data | `00-Global/ERD-MASTER.md` | 12 entitas, 21 relasi, bridge M:N |
| Sumber requirement | `question-framework.md` | A–F terjawab + 6 gate final |
| SRS master | `00-Global/SRS-MASTER.md` | Scope, actor, objektif, FR-001…023, integrasi |
| Glossary | `00-Global/GLOSSARY.md` | 30 istilah, 6 kategori |
| NFR | `nfr.md` | 20 NFR, skenario QA 6 bagian |
| Stakeholder | `stakeholder-register.md` | SH-001…011 + peta influence/interest |
| RACI | `raci.md` | Per FR, per aktivitas, per keputusan |
| Asumsi & konstraint | `assumptions-constraints.md` | 6 ASM + 9 CON + risk register |
| Desain program | `DESAIN-PROGRAM.md` | 8 bagian: §1 problem statement → §8 NFR ringkas (flowchart + DFD, tanpa BPMN) |

**Menyusul:** `business-rules.md` (katalog penuh 40 `BR-*`) ·
`00-Global/PROCESS-FLOW-MASTER.md` (PF-001…PF-007) ·
`BAB-1-*` & `BAB-2-*` · `spec.md`.
