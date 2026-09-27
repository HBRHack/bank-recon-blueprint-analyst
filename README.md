# BANK-RECONCILIATION-ANALYST

**Analysis & System Design Portfolio — Simulasi Rekonsiliasi Bank & Pemetaan Mutasi**
*(Bank Reconciliation & Statement Mapping System)*

| | |
|---|---|
| **Status proyek** | Analysis-first / design portfolio — desain selesai, implementasi bukan fokus |
| **Bentuk** | Dokumen analisis: requirement, struktur data, business rules, process flow, output/laporan, NFR |
| **Bahasa dokumen** | Bahasa Indonesia · **tanpa sebutan bahasa pemrograman / framework / basis data** (CON-009) |
| **Diagram** | ASCII/Unicode box art · Mermaid **hanya** di `DESAIN-PROGRAM.md` §5.3–§5.4 (pengecualian 27 Sep 2026) — sisanya dilarang (`00-Global/NOTATION.md`) |
| **Catatan review** | [`need-review.md`](need-review.md) — daftar error yang belum diperbaiki |

---

## 1. Urutan baca

Baca **dari atas ke bawah** — tiap dokumen hanya punya arti setelah dokumen
sebelumnya selesai. Jangan mulai dari diagram.

```
 README.md  (Anda di sini)
      |
      v
 1. docs/analyst/question-framework.md ........ A–F terjawab + 6 gate final
      |                                          (sumber seluruh keputusan)
      v
 2. docs/analyst/00-Global/SRS-MASTER.md ....... masalah, objektif, FR-001…023
      |
      v
 3. docs/analyst/00-Global/GLOSSARY.md ......... 31 istilah — baca sebelum §6
      |
      v
 4. docs/analyst/DESAIN-PROGRAM.md ............. inti portofolio, 8 bagian:
      |                                          §1 Problem Statement
      |                                          §2 Scope & Batasan
      |                                          §3 Actor & Use Case
      |                                          §4 Struktur Data
      |                                          §5 Alur Proses (PF-001…007)
      |                                          §6 Aturan Bisnis (40 BR)
      |                                          §7 Output/Laporan
      |                                          §8 NFR ringkas
      v
 5. docs/analyst/00-Global/ERD-MASTER.md ....... 12 entitas, 21 relasi (rincian §4)
      |
      v
 6. docs/analyst/00-Global/REQUIREMENTS-MATRIX.md  traceability FR ↔ UC ↔ BR ↔ NFR
      |
      v
 7. docs/analyst/business-rules.md ............. katalog 40 BR-* (latar & kasus uji)
      |
      v
 8. docs/analyst/nfr.md ........................ 20 NFR, skenario QA 6 bagian
 9. docs/analyst/raci.md · stakeholder-register.md · assumptions-constraints.md
 10. docs/analyst/00-Global/PROCESS-FLOW-MASTER.md  index PF-001…PF-007
 11. docs/analyst/00-Global/NOTATION.md ........ aturan gambar & penomoran
```

**Kenapa urutannya begitu:** kalimat di §6 (`BR-VAL-003…`) hanya bisa dibaca
setelah FR-005 di SRS dan istilah di GLOSSARY jelas; diagram §5 hanya masuk akal
setelah tahu siapa aktornya (§3) dan data apa yang dimainkan (§4).

**Sudah beres** (27 Sep 2026): `business-rules.md` (katalog 40 aturan) dan
`00-Global/PROCESS-FLOW-MASTER.md` (PF-001…PF-007). **Yang belum ditulis:**
BAB 1–2 — peta kerjanya ada di
[`00-Global/HANDOFF-BAB-1-2.md`](docs/analyst/00-Global/HANDOFF-BAB-1-2.md).
Daftar penuh di `DESAIN-PROGRAM.md` Appendix A dan [`need-review.md`](need-review.md).

---

## 2. Tujuan proyek

Rekonsiliasi harian 4 bank masih dikerjakan manual: **06:30–10:00, ±3,5 jam**
tiap hari kerja, dengan selisih yang tidak pernah tuntas (floating Rp 1,2–2 M).
Proyek ini menerjemahkan masalah itu menjadi rancangan sistem yang bisa
diimplementasikan — **bukan** menjadi aplikasi.

Apa yang didemonstrasikan:

```
 Masalah bisnis
      v
 Requirement (FR/NFR)  -->  Actor & hak akses  -->  Struktur data
      |                                                      |
      v                                                      v
 Aturan bisnis (40 BR)  <-->  Alur proses (7 PF)  <-->  Output & laporan
      |
      v
 Angka ambang yang bisa diuji (Rp 1 · 90/200 · 0,5% · 10 tahun)
```

Bukti bahwa seorang analis mampu memutuskan: **apa masalahnya, siapa yang
bertanggung jawab, data apa yang disimpan, kapan status berubah, apa yang tidak
boleh terjadi, dan bagaimana semuanya saling terhubung** — sebelum satu baris
kode pun ditulis.

---

## 3. Untuk siapa repo ini

| Pembaca | Mulai dari | Yang dicari |
|---------|-----------|-------------|
| **IT Business Analyst / System Analyst** | Bagian 1 → `DESAIN-PROGRAM.md` | Cara masalah diterjemahkan jadi FR, aturan, dan alur |
| **Solution / Software Architect** | §4 + `00-Global/ERD-MASTER.md` | Model data, relasi M:N, batas konsistensi, kendali ganda |
| **Technical interviewer** | §6 + §5 | Business rules bernomor & bisa ditest, process flow + penanganan pengecualian |
| **QA / Tester** | `00-Global/REQUIREMENTS-MATRIX.md` + `nfr.md` | Traceability dan ambang ukur |
| **Pembaca non-teknis** | Bagian 1 → Bagian 5 | Alur kerja harian tanpa perlu istilah teknis |

Pertanyaan lanjutan yang bisa diajukan — dan jawabannya sudah ada di dokumen:

- Apa yang terjadi kalau bank telat mengirim mutasi? → `BR-WF-008`, PF-001
- Kenapa baris mutasi mentah tidak boleh diubah? → `BR-AUTH-005`, `BR-CON-003`
- Siapa yang boleh menyetujui koreksi >Rp 5 juta? → `BR-AUTH-001`
- Kenapa `NON_MATCHING` tidak boleh disembunyikan? → `BR-VAL-010`
- Bagaimana mencegah satu baris bank terpasangkan dua kali? → `BR-CON-001`
- Berapa lama jejak audit disimpan dan kenapa? → `BR-RET-001`, `NFR-COMP-001`

---

## 4. Kenapa tidak langsung membuat program

Karena pada repo ini **program bukan titik yang ingin didemonstrasikan**.
Aplikasi bisa dibuat dari requirement yang sudah ada; nilai analitisnya justru
terlihat *sebelum* kode ditulis.

Implementasi tetap dipikirkan sejak awal, tetapi diperlakukan sebagai **jalur
implementasi yang direkomendasikan**, bukan pusat portofolio.

> **Catatan sengaja:** repo ini **tidak** merekomendasikan stack tertentu.
> `CON-009` melarang dokumen desain terikat bahasa/framework/engine. Rekomendasi
> teknologi baru ditulis setelah desain disetujui — bukan sebaliknya.

---

## 5. Ringkasan proyek

### Skala dokumen

| Keluarga id | Jumlah | Berkas |
|-------------|--------|--------|
| FR (Functional Requirements) | 23 | `00-Global/SRS-MASTER.md` |
| NFR | 20 | `nfr.md` |
| BR (Business Rules, 6 kategori) | 40 | `DESAIN-PROGRAM.md` §6 · katalog: `business-rules.md` (selesai) |
| UC (Use Case) | 12 | `DESAIN-PROGRAM.md` §3 |
| PF (Process Flow) | 7 | `DESAIN-PROGRAM.md` §5.2 |
| Entitas / relasi data | 12 / 21 | `00-Global/ERD-MASTER.md` |
| Output & laporan | 7 (`O-01…O-07`) | `DESAIN-PROGRAM.md` §7 |
| CON (kendala) | 9 | `00-Global/SRS-MASTER.md`, `assumptions-constraints.md` |
| ASM (asumsi) / OQ (terbuka) | 6 / 4 | `assumptions-constraints.md`, `need-review.md` |
| Stakeholder | 11 + 1 gap | `stakeholder-register.md` |
| Istilah (glossary) | 30 | `00-Global/GLOSSARY.md` |

### Angka inti sistem

| Angka | Nilai |
|-------|-------|
| Sumber data | 4 bank partner (3 otomatis 05:00, 1 portal manual) |
| Volume harian | ±18.000 baris mutasi ↔ ±16.000 transaksi ledger |
| Selisih wajar | ±2.000 baris/hari — memang tidak berpasangan |
| Waktu proses target | ≤30 menit, hasil siap ≤06:30 WIB (baseline manual 3,5 jam) |
| Cut-off / tutup hari | 06:00 WIB / 07:30 WIB |
| Toleransi pencocokan | ≤ Rp 1 (pembulatan) |
| Ambang peringatan | WASPADA > 90 · KRITIS > 200 selisih, respons ≤ 15 menit |
| Carry-over maksimal | 0,5% (≤ 90 baris) |
| Eskalasi nilai koreksi | > Rp 5.000.000 → Manajer Keuangan |
| Retensi | jejak audit + data mentah 10 tahun · ringkasan 2 tahun |

---

## 6. Alur kerja harian

```
 05:00   [SISTEM]  terima mutasi 4 bank, validasi struktur & control total
             |
             v
         [SISTEM]  normalisasi lintas bank -> cocokkan berjenjang T1..T5
             |                                    (agregasi / split)
             v
 06:30   [SISTEM]  hitung selisih -> WASPADA >90 / KRITIS >200
             |
             v
         [ANALIS]  triage selisih, pencocokan manual (alasan wajib)
             |          \
             |           +-- tidak ketemu --> ajukan koreksi
             v                                     |
         [SUPERVISOR]  setujui (dual-control) <-----+
             |            nilai > Rp 5 juta --> Manajer Keuangan
             v
 07:30   [SUPERVISOR]  tutup hari bila sisa <= 0,5% (<= 90 baris)
             |
             v
         [SISTEM]  laporan terbit + jejak audit append-only (10 tahun)
```

**Status selisih (ringkas):** `OPEN` → `RESOLVED` (cocok manual) / `ADJUSTED`
(koreksi disetujui) — tidak pernah dihapus, hanya berpindah status dengan jejak.

Detail lengkap (7 alur PF + cabang + penanganan pengecualian) di
`DESAIN-PROGRAM.md` §5.

---

## 7. Aturan bisnis inti

40 aturan dalam 6 kategori: `VAL` validasi · `CALC` perhitungan · `AUTH`
otorisasi · `WF` alur kerja · `CON` kendali data · `RET` retensi.

| Id | Aturan |
|----|--------|
| `BR-CON-001` | Satu baris bank hanya boleh punya satu hasil pasangan |
| `BR-CON-003` | Jejak audit append-only + segel rantai, 0 baris bisa diubah |
| `BR-VAL-009` | Tiap selisih wajib punya **tepat satu** dari 8 kode alasan |
| `BR-VAL-010` | Kategori `NON_MATCHING` tetap tampil — tidak boleh disembunyikan |
| `BR-CALC-001` | Toleransi nominal ≤ Rp 1 |
| `BR-AUTH-001` | Koreksi di Supervisor; > Rp 5 juta wajib Manajer Keuangan |
| `BR-AUTH-002` | Yang menyetujui **tidak boleh** yang mengajukan |
| `BR-WF-004` | Peringatan 06:30: WASPADA >90, KRITIS >200, respons ≤15 menit |
| `BR-WF-006` | Tutup hari hanya bila sisa ≤ 0,5% dan sebelum 07:30 |
| `BR-WF-007` | Jalur ulang idempoten — koreksi manual **dipertahankan** |

8 kode alasan selisih: `MISSING_REF`, `AMOUNT_DIFF`, `DATE_DIFF`,
`DUPLICATE_REF`, `ORPHAN_BANK`, `ORPHAN_LEDGER`, `AGG_UNRESOLVED`,
`REVERSAL_MISMATCH`.

---

## 8. Peran & tanggung jawab

| Kode | Peran | Fokus | Batas hak akses |
|------|-------|-------|-----------------|
| A1 | Analis | Triage selisih, cocokkan manual, ajukan koreksi | Tidak boleh ubah baris mentah / menyetujui sendiri |
| A2 | Supervisor | Setujui koreksi, tutup hari, terima peringatan | Tidak boleh setujui pengajuannya sendiri |
| A3 | Auditor | Tinjau laporan, unduh jejak audit | **Baca-saja** + masking PII |
| A4 | System Analyst | Ubah parameter aturan | Tidak boleh menyetujui perubahan sendiri / tutup hari |
| A5 | Sistem (jadwal 05:00–06:30) | Terima, validasi, cocokkan, peringat, arsip | Tidak punya kewenangan persetujuan |
| B1 | Bank Partner | Kirim mutasi | Hanya sumber data |
| B2 | Manajer Keuangan | Eskalasi koreksi >Rp 5 juta, putus carry-over lewat | Tingkat persetujuan tertinggi |

Matriks hak akses per use case: `DESAIN-PROGRAM.md` §3.5.

---

## 9. Struktur dokumentasi

```
Bank-Reconciliation/
├── README.md                     # berkas ini
├── need-review.md                # daftar error & rencana perbaikan
│
└── docs/analyst/
    ├── question-framework.md     # A–F terjawab + 6 gate (sumber keputusan)
    ├── DESAIN-PROGRAM.md         # ★ desain 8 bagian (dokumen inti)
    ├── nfr.md                    # 20 NFR, skenario QA 6 bagian
    ├── stakeholder-register.md   # SH-001…011 + gap SH-012
    ├── raci.md                   # RACI per FR / aktivitas / keputusan
    ├── assumptions-constraints.md# 6 ASM + 9 CON + risk register
    ├── business-rules.md         # ★ katalog 40 BR-* (identik §6)
    │
    └── 00-Global/
        ├── NOTATION.md           # matriks notasi + NO MERMAID (§5.3–§5.4 pengecualian) + penomoran
        ├── SRS-MASTER.md         # scope, objektif, FR-001…023, konstrain
        ├── ERD-MASTER.md         # 12 entitas, 21 relasi, bridge M:N
        ├── GLOSSARY.md           # 31 istilah
        ├── PROCESS-FLOW-MASTER.md# index PF-001…PF-007 + relationship map
        ├── HANDOFF-BAB-1-2.md    # serah terima penulisan BAB 1–2
        └── REQUIREMENTS-MATRIX.md# traceability + coverage + gaps

 Menyusul: BAB 1–2 (mengikuti HANDOFF-BAB-1-2.md) · spec.md
```

---

## 10. Scope

**Termasuk:** penerimaan mutasi 4 bank · validasi struktur & control total ·
normalisasi lintas bank · pencocokan berjenjang T1→T5 (agregasi/split) ·
penanganan selisih manual · koreksi dengan persetujuan dua orang · peringatan
bertingkat · penutupan hari · laporan & jejak audit · pengaturan aturan
berversi.

**Tidak termasuk (eksplisit ditolak):** klasifikasi otomatis penyebab selisih ·
prediksi arus kas · rekonsiliasi berjalan (intraday) · multi-mata uang ·
integrasi akuntansi general ledger · pembelajaran mesin · modul pengadaan/HR ·
multi-cabang multi-entitas (rilis 2).

Pembagian rilis, asumsi yang harus diverifikasi, dan 9 konstrain keras:
`DESAIN-PROGRAM.md` §2.

---

## 11. What I am demonstrating

```
 Problem  ->  Requirement  ->  Data  ->  Rule  ->  Process  ->  Output  ->  NFR
   (§1)        (SRS/FR)      (§4/ERD)  (§6)      (§5)        (§7)      (§8)
```

Satu hari rekonsiliasi dari ujung ke ujung:

```
 Bank kirim mutasi (05:00)
   -> batch divalidasi / ditolak (BR-VAL-001)
   -> dinormalisasi per profil bank (BR-VAL-002)
   -> dicocokkan T1..T5 dengan toleransi Rp 1 (BR-VAL-003..007, BR-CALC-001)
   -> sisa jadi selisih bernomor alasan (BR-VAL-009)
   -> peringatan 06:30 (BR-WF-004)
   -> analis triage, koreksi diajukan (BR-WF-002)
   -> supervisor setujui, >Rp 5 juta naik Manajer (BR-AUTH-001/002)
   -> koreksi jadi BARIS LEDGER BARU, data lama tak tersentuh (BR-WF-003, BR-CON-002)
   -> tutup hari bila sisa <= 0,5% (BR-WF-006)
   -> ringkasan terbit, jejak audit append-only 10 tahun (BR-RET-001)
```

---

## 12. Pengembangan lanjutan (bila diimplementasikan)

Bukan bagian dari portofolio ini — hanya urutan yang disarankan:

```
 Skema data (dari ERD-MASTER)
      v
 Data uji (seed) volume riil 18.000 baris
      v
 Penerimaan & validasi batch  ->  Pencocokan T1..T5
      v
 Layar triage selisih + pencocokan manual
      v
 Alur koreksi & persetujuan dua orang (dual-control)
      v
 Peringatan 06:30 + penutupan hari
      v
 Laporan & ekspor jejak audit
      v
 Pengujian fokus: (a) baris ganda tidak mungkin terjadi,
                  (b) penolakan persetujuan diri sendiri,
                  (c) run ulang idempoten
```

---

## Source of Decisions

| Dokumen | Yang diputuskan |
|---------|-----------------|
| `docs/analyst/question-framework.md` | Asumsi awal, keputusan user, 6 gate final |
| `docs/analyst/00-Global/SRS-MASTER.md` | Scope, objektif, FR-001…023, 9 konstrain |
| `docs/analyst/DESAIN-PROGRAM.md` | Seluruh desain 8 bagian, 40 BR, 7 PF, 7 output |
| `docs/analyst/00-Global/ERD-MASTER.md` | 12 entitas, 21 relasi, field & enum |
| `docs/analyst/nfr.md` | 20 NFR + ambang ukur |
| `docs/analyst/00-Global/NOTATION.md` | Aturan diagram, penomoran, keputusan tanpa BPMN |
| `docs/analyst/00-Global/REQUIREMENTS-MATRIX.md` | Traceability & coverage |
| `need-review.md` | Daftar error yang masih terbuka |
