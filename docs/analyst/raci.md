# RACI Matrix: Bank Reconciliation & Statement Mapping System

Lanskap stakeholder kecil tapi berat pada kendali ganda: Finance memegang
keputusan bisnis, System Analyst memegang definisi & konfigurasi aturan, Tim TI
memegang operasional — **hanya satu `A` per aktivitas**.

## Document Info

| Field | Value |
|-------|-------|
| **Version** | 1.0 |
| **Last Updated** | 26 September 2026 |
| **Owner** | PM (SH-011) |

## RACI Legend

| Letter | Role | Definition |
|--------|------|------------|
| **R** | Responsible | Mengerjakan tugas sampai selesai |
| **A** | Accountable | Pemilik & penandatangan — **tepat satu per baris** |
| **C** | Consulted | Dimintai masukan (dua arah) |
| **I** | Informed | Diberi tahu (satu arah) |

## Stakeholder Directory

| ID | Name | Role/Title | Department | Contact |
|----|------|------------|------------|---------|
| S01 | {待isi} | Manajer Keuangan (SH-001) | Finance | — |
| S02 | {待isi} | Supervisor Keuangan (SH-002) | Finance | — |
| S03 | {待isi} | Finance Ops Analyst (SH-003, 3 org) | Finance Ops | — |
| S04 | {待isi} | Auditor Internal (SH-004) | Risk/Compliance | — |
| S05 | {待isi} | System Analyst (SH-005) | Technology | — |
| S06 | {待isi} | Tim Operasional TI (SH-006) | Technology | — |
| S07 | {待isi} | PM (SH-011) | PMO | — |

---

## RACI by Requirement Area

### Penerimaan & Pencocokan (FR-001 – FR-008)

| Requirement | S01 | S02 | S03 | S04 | S05 | S06 | S07 |
|-------------|-----|-----|-----|-----|-----|-----|-----|
| FR-001: Penerimaan mutasi terjadwal | I | I | C | I | C | R | A |
| FR-002: Penanda keterlambatan statement | I | C | I | I | C | R | A |
| FR-003: Validasi struktur & control total | I | C | C | I | R | C | A |
| FR-004: Normalisasi lintas bank | I | C | C | I | R | C | A |
| FR-005: Pencocokan berjenjang T1→T5 | C | C | C | I | R | I | A |
| FR-006: Dukungan agregasi & split | C | R | C | I | R | I | A |
| FR-007: Anti pasangan ganda | I | C | C | C | R | I | A |
| FR-008: Skor keyakinan & antrean review | I | C | R | C | R | I | A |

### Penanganan Selisih & Koreksi (FR-009 – FR-014)

| Requirement | S01 | S02 | S03 | S04 | S05 | S06 | S07 |
|-------------|-----|-----|-----|-----|-----|-----|-----|
| FR-009: Pengkodean alasan selisih | I | R | C | C | R | I | A |
| FR-010: Kategori non-matching tetap terlihat | I | C | I | R | C | I | A |
| FR-011: Aging & penugasan selisih | I | R | C | C | C | I | A |
| FR-012: Pencocokan manual oleh analis | I | A | R | I | C | I | I |
| FR-013: Koreksi dengan persetujuan dua orang | A | R | R | C | C | I | I |
| FR-014: Koreksi menjadi baris baru | C | A | R | C | R | C | I |

### Laporan, Pengaturan & Audit (FR-015 – FR-023)

| Requirement | S01 | S02 | S03 | S04 | S05 | S06 | S07 |
|-------------|-----|-----|-----|-----|-----|-----|-----|
| FR-015: Ringkasan rekonsiliasi harian | I | A | C | C | C | I | I |
| FR-016: Register selisih + aging | I | A | C | C | C | I | I |
| FR-017: Peringatan bertingkat (90/200) | I | A | I | I | C | R | I |
| FR-018: Ekspor laporan untuk Auditor | I | C | I | R | C | I | A |
| FR-019: Pengaturan parameter aturan | C | R | I | I | R | I | A |
| FR-020: Versi aturan per run | I | C | I | C | R | C | A |
| FR-021: Penutupan hari rekonsiliasi | A | R | C | I | I | I | I |
| FR-022: Jejak audit append-only | C | I | I | R | R | R | A |
| FR-023: Catatan rangkap peran | I | C | I | C | R | I | A |

---

## RACI by Activity

| Activity | S01 | S02 | S03 | S04 | S05 | S06 | S07 |
|----------|-----|-----|-----|-----|-----|-----|-----|
| Pengumpulan requirement (interview/observasi) | C | C | R | C | R | I | A |
| Dokumentasi requirement & dokumen core | C | C | C | C | R | I | A |
| Konfirmasi glossary & business rules | A | R | C | C | R | I | I |
| Desain struktur data (ERD) | I | C | I | C | R/A | C | I |
| Penulisan NFR & kepatuhan | A | C | I | R | R | C | I |
| Pengaturan aturan pencocokan | C | R | I | I | R | I | A |
| Uji fungsional (internal) | I | C | R | I | C | R | A |
| UAT (uji terima pengguna) | A | R | R | C | C | I | I |
| Keputusan go-live | A | C | C | C | C | R | R |
| Drill pemulihan & operasional | I | I | I | C | C | R/A | I |
| Review kepatuhan (retensi, jejak audit) | C | I | I | R/A | C | I | I |
| Pelatihan analis (2 hari) | I | A | R | I | C | I | I |
| Perubahan scope/requirement | A | C | C | I | R | I | R |

> `R/A` ditandai bila satu pihak sekaligus mengerjakan & bertanggung jawab
> (aktivitas teknis yang tidak terpecah) — tetap **satu** A.

---

## RACI by Decision Type

| Decision | S01 | S02 | S05 | S04 | Escalation Path |
|----------|-----|-----|-----|-----|-----------------|
| Kebutuhan fungsional (FR) | A | C | R | C | S05 → S02 → S01 |
| Aturan pencocokan & toleransi | C | A | R | I | S05 → S02 → S01 |
| Kebutuhan non-fungsional (NFR) | A | C | R | C | S05 → S01 |
| Desain struktur data | I | C | R/A | C | S05 → S07 |
| Kebijakan keamanan & akses | A | C | R | C | S05 → S01 (→ S04 utk kepatuhan) |
| Kepatuhan (retensi, UU PDP, OJK) | C | I | C | R/A | S04 → S01 |
| Persetujuan koreksi per kasus | A | R | I | I | S02 → S01 (>Rp 5 juta) |
| Tutup hari rekonsiliasi | A | R | I | I | S02 → S01 |
| Keputusan go-live | A | C | C | I | S07 → S01 |
| Perubahan jadwal/scope proyek | A | I | C | I | S07 → S01 |

---

## Conflict Resolution

| Scenario | Resolution |
|----------|------------|
| Lebih dari satu orang mengklaim `A` | Hanya **satu** A. PM (S07) memutuskan dalam 1 hari kerja. |
| `R` dan `A` orang yang sama | Boleh untuk tugas teknis kecil — didokumentasikan (lihat `R/A`). |
| Ada baris tanpa `R` (tidak ada yang mengerjakan) | Kekurangan sumber daya → eskalasi ke PM (S07). |
| Ada baris tanpa `A` (tidak ada yang bertanggung jawab) | **Kritis** — wajib diisi sebelum sign-off. |
| Stakeholder belum terdaftar | Tambahkan ke `stakeholder-register.md` + beri RACI. |
| Deadlock aturan antara Finance vs Technology | **Jalur eskalasi final: S01 (Manajer Keuangan)** — keputusan bisnis selalu di Finance; Technology hanya menyangkut cara. |
| Auditor keberatan dengan suatu requirement | **Veto tersimpan**: Auditor (S04) bisa menahan sign-off dokumen kepatuhan sampai keberatannya tercatat & direspon. |

---

## Sign-off Authority Matrix

| Level | Minimum Approver | Sign-off Format |
|-------|------------------|-----------------|
| Dokumen core (SRS-MASTER, NFR, glossary) | S01 + S04 | Dokumen sign-off formal |
| FR per rilis | S01 (A), dikonfirmasi S02 | Email + dokumen |
| BR (aturan pencocokan) | S02, dikonfirmasi S05 | Dokumen |
| ERD / desain data | S05 (A) | Dokumen |
| UAT exit | S01 (A) + S02 (R) | Dokumen UAT |
| Keputusan koreksi individual | S02 (kecuali >Rp 5 juta → S01) | Catatan di jejak audit |
| Tutup hari | S02 (A), disiapkan S03 | Tanda tangan digital harian |
| Go-live | S01 (A) + S07 (R) | Dokumen go-live |

## Review Schedule

| Review Date | Attendees | Purpose |
|-------------|-----------|---------|
| {kickoff} | S01, S05, S07 | Penetapan RACI awal |
| {akhir rilis 1} | S02, S03, S05, S07 | Review tengah proyek |
| {pra sign-off} | Semua (S01–S07) | Review pra-penandatanganan |

## Sign-off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Business Owner (S01) | {待isi} | {—} | ___________ |
| Product/Business Lead (S02) | {待isi} | {—} | ___________ |
| Technical Owner (S05) | {待isi} | {—} | ___________ |
| PM (S07) | {待isi} | {—} | ___________ |
