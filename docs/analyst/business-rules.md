# Business Rules — Bank Reconciliation & Statement Mapping System

Katalog lengkap **40 aturan bisnis** sistem rekonsiliasi bank, dikelompokkan ke
dalam **6 kategori kode** (`VAL · CALC · AUTH · WF · CON · RET`). Setiap aturan
memuat *Rule / When / Then / Else* + contoh konkret + jejak ke FR, UC, proses
(`PF-`), dan entitas data sehingga bisa langsung diuji QA.

> **Hubungan dengan `DESAIN-PROGRAM.md` §6:** §6 adalah **ringkasan desain**
> (tabel When/Then/Else), katalog ini adalah **sumber rujukan lengkap**.
> **ID dan bunyi When/Then/Else wajib identik** — bila ada selisih, §6 yang
> diperbaiki agar cocok ke katalog ini (lihat Change Log).
> Katalog ini juga **menggantikan** nama lama `BR-MATCH-*` yang sudah
> dipensiunkan.

## Document Info

| Field | Value |
|-------|-------|
| **Version** | 1.0 |
| **Last Updated** | 27 September 2026 |
| **Owner** | System Analyst |
| **Approval Status** | Draft — menunggu sign-off Manajer Keuangan (SH-001) |
| **Sumber nilai** | `question-framework.md` (A–F + 6 gate) · `00-Global/SRS-MASTER.md` · `00-Global/REQUIREMENTS-MATRIX.md` · `DESAIN-PROGRAM.md` §6 |
| **Total** | 40 aturan — VAL 11 · CALC 4 · AUTH 8 · WF 9 · CON 5 · RET 3 |

**Ringkasan status:** 34 Confirmed · 6 Pending (BR-VAL-001, BR-VAL-006,
BR-CALC-002, BR-CALC-003, BR-WF-001, BR-RET-002 — mengikuti status FR sumbernya).

**Ringkasan prioritas:** 31 Must · 9 Should · 0 Could.

## Rule Categories

| Code | Category | Description | Jumlah |
|------|----------|-------------|--------|
| VAL | Validation | Validasi data masuk & aturan pencocokan | 11 |
| CALC | Calculation | Rumus, ambang, dan pembulatan | 4 |
| AUTH | Authorization | Siapa boleh apa, kendali ganda, masking | 8 |
| WF | Workflow | Peralihan status, jadwal, eskalasi, ulang proses | 9 |
| CON | Constraint | Batas keras integritas data | 5 |
| RET | Retention | Retensi, arsip, kepemilikan versi aturan | 3 |

> **Catatan penamaan:** `BR-CON-*` = *aturan bisnis kategori kendali data*,
> **berbeda** dari `CON-001…009` di `SRS-MASTER.md` yang = *kendala proyek*.
> Awalan `BR-` yang membedakan.

## Rules by Category — Indeks

| ID | Judul | Prioritas | Status | FR | UC | PF |
|----|-------|-----------|--------|----|----|----|
| `BR-VAL-001` | Kontrol total per batch | Must | [ ] Pending | FR-003 | UC-01 | PF-001 |
| `BR-VAL-002` | Normalisasi referensi lintas bank | Must | [x] Confirmed | FR-004 | UC-02 | PF-002 |
| `BR-VAL-003` | Tier 1 — referensi identik | Must | [x] Confirmed | FR-005 | UC-02 | PF-002 |
| `BR-VAL-004` | Tier 2 — nominal & tanggal | Must | [x] Confirmed | FR-005 | UC-02 | PF-002 |
| `BR-VAL-005` | Tier 3 — agregasi | Should | [x] Confirmed | FR-005, FR-006 | UC-02 | PF-002 |
| `BR-VAL-006` | Tier 4 — keterangan mirip | Should | [ ] Pending | FR-005, FR-008 | UC-02 | PF-002 |
| `BR-VAL-007` | Tier 5 — nominal parsial | Should | [x] Confirmed | FR-005, FR-009 | UC-02 | PF-002 |
| `BR-VAL-008` | Split & leg banyak | Should | [x] Confirmed | FR-006 | UC-02 | PF-002 |
| `BR-VAL-009` | Wajib satu dari 8 kode alasan | Must | [x] Confirmed | FR-009 | UC-03 | PF-003 |
| `BR-VAL-010` | `NON_MATCHING` tetap tampil | Must | [x] Confirmed | FR-010, FR-016 | UC-09 | PF-003 |
| `BR-VAL-011` | Pasangan reversal | Should | [x] Confirmed | FR-009 | UC-02 | PF-002 |
| `BR-CALC-001` | Toleransi nominal ≤ Rp 1 | Must | [x] Confirmed | FR-005 | UC-02 | PF-002 |
| `BR-CALC-002` | Skor keyakinan per tier | Should | [ ] Pending | FR-008 | UC-02 | PF-002 |
| `BR-CALC-003` | Aging selisih > 1 hari | Must | [ ] Pending | FR-011 | UC-03 | PF-003 |
| `BR-CALC-004` | Persentase carry-over ≤ 0,5% | Must | [x] Confirmed | FR-021 | UC-06 | PF-006 |
| `BR-AUTH-001` | Persetujuan koreksi + eskalasi >Rp 5 juta | Must | [x] Confirmed | FR-013 | UC-04, UC-05 | PF-004 |
| `BR-AUTH-002` | Penyetuju ≠ pengaju | Must | [x] Confirmed | FR-013 | UC-05 | PF-004 |
| `BR-AUTH-003` | SA tanpa persetujuan & tanpa tutup hari | Must | [x] Confirmed | FR-019, FR-021 | UC-08 | PF-007 |
| `BR-AUTH-004` | Auditor baca-saja | Must | [x] Confirmed | FR-018 | UC-09, UC-10 | — |
| `BR-AUTH-005` | Tidak ada peran boleh ubah baris mentah | Must | [x] Confirmed | FR-022 | UC-03 | — |
| `BR-AUTH-006` | 2FA + kunci setelah 5 kali gagal | Must | [x] Confirmed | — (NFR-SEC-001) | semua | — |
| `BR-AUTH-007` | Masking data pribadi | Should | [x] Confirmed | FR-018 | UC-09, UC-10 | — |
| `BR-AUTH-008` | `acting_as_role` saat rangkap peran | Should | [x] Confirmed | FR-023 | UC-03, UC-06 | PF-004 |
| `BR-WF-001` | Pemilik selisih + aging > 1 hari | Must | [ ] Pending | FR-011 | UC-03 | PF-003 |
| `BR-WF-002` | Aksi manual wajib alasan + jejak | Must | [x] Confirmed | FR-012 | UC-03 | PF-003 |
| `BR-WF-003` | Koreksi jadi baris ledger baru | Must | [x] Confirmed | FR-013, FR-014 | UC-05 | PF-004 |
| `BR-WF-004` | Peringatan 90/200 + respons 15 menit | Must | [x] Confirmed | FR-017 | UC-07 | PF-005 |
| `BR-WF-005` | Aturan berlaku run berikutnya | Must | [x] Confirmed | FR-019, FR-020 | UC-08 | PF-007 |
| `BR-WF-006` | Tutup hari ≤ 0,5% sebelum 07:30 | Must | [x] Confirmed | FR-021 | UC-06 | PF-006 |
| `BR-WF-007` | Jalur ulang idempoten | Must | [x] Confirmed | FR-005 | UC-12 | PF-002 |
| `BR-WF-008` | Proses parsial saat bank telat | Must | [x] Confirmed | FR-001, FR-002 | UC-01 | PF-001 |
| `BR-WF-009` | Fallback manual bila belum ada hasil 06:30 | Must | [x] Confirmed | FR-002, FR-017 | UC-07 | PF-005 |
| `BR-CON-001` | Anti pasangan ganda | Must | [x] Confirmed | FR-007 | UC-03 | PF-002 |
| `BR-CON-002` | Koreksi ≠ edit data lama | Must | [x] Confirmed | FR-014 | UC-05 | PF-004 |
| `BR-CON-003` | Append-only + segel rantai jejak | Must | [x] Confirmed | FR-022 | UC-10 | — |
| `BR-CON-004` | Cut-off 06:00 / tutup 07:30 | Must | [x] Confirmed | FR-001 | UC-01, UC-06 | PF-001 |
| `BR-CON-005` | IDR 2 desimal, tanpa konversi | Must | [x] Confirmed | — (CON-004) | UC-01 | PF-001 |
| `BR-RET-001` | Retensi 10 tahun utuh | Must | [x] Confirmed | FR-022 | UC-10 | — |
| `BR-RET-002` | Arsip versi aturan per run | Must | [ ] Pending | FR-020 | UC-08 | PF-007 |
| `BR-RET-003` | Ringkasan 2 tahun | Should | [x] Confirmed | FR-015 | UC-09 | PF-006 |

---

# Rules by Category

## VAL — Validasi & pencocokan (11 aturan)

### BR-VAL-001: Kontrol total per batch

| Field | Value |
|-------|-------|
| **ID** | BR-VAL-001 |
| **Category** | Validation |
| **Priority** | Must |
| **Status** | [ ] Pending |
| **Source** | FR-003 · SH-004 (kebutuhan audit) via `question-framework.md` aspek B |

**Rule:** Sebuah batch mutasi hanya boleh diproses bila jumlah baris, total
debit, dan total kredit **persis sama** dengan nilai pembanding pada kepala
berkas — tidak ada toleransi parsial.

**When:** Batch mutasi diterima dari bank

**Then:** Jumlah baris, total debit, total kredit = nilai pembanding → batch
`VALIDATED`, lanjut ke PF-002

**Else:** Salah satu beda → batch `REJECTED`, nilai lama & baru dicatat di
jejak audit

**Examples:**
- Kepala berkas Bank A: 18.042 baris, debit 4.120.500.000,00 — isi berkas
  identik → `VALIDATED`, masuk antrean normalisasi.
- Kepala berkas bilang 5.000 baris, isi 4.998 baris → `REJECTED`; jejak
  `AuditEvent` menyimpan `expected 5000 / actual 4998`, analis ambil ulang
  file (maks 3 kali).

**Related:**
- Use Case: UC-01, UC-12
- Requirement: FR-003
- Process: PF-001
- Data Entity: `StatementBatch`, `AuditEvent`

---

### BR-VAL-002: Normalisasi referensi lintas bank

| Field | Value |
|-------|-------|
| **ID** | BR-VAL-002 |
| **Category** | Validation |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-004 · SH-003 (aspek B — beda penulisan antar bank) |

**Rule:** Seluruh baris dinormalisasi dengan **satu** profil bank
(`FMT_A/FMT_B/FMT_C`) sebelum tier pencocokan dijalankan — normalisasi tidak
boleh dilewati walau referensi terlihat rapi.

**When:** Sebelum pencocokan dimulai

**Then:** Setiap baris dinormalisasi per profil bank (FMT_A/B/C): tanggal
seragam, spasi & tanda baca referensi dibuang, huruf dikecilkan, nomor rekening
disamakan

**Else:** Baris tanpa profil bank → batch `REJECTED`, tidak ikut dicocokkan

**Examples:**
- `TRF-123 ` (Bank B, huruf besar + spasi) dan `123` (Bank A) → keduanya jadi
  `123` sehingga Tier 1 bisa memasangkan.
- Baris datang dari bank ke-5 yang belum punya profil `FMT_x` → batch
  `REJECTED`, tidak ada baris yang dipaksa lewat tier mana pun.

**Related:**
- Use Case: UC-02, UC-12
- Requirement: FR-004
- Process: PF-002
- Data Entity: `StatementSource`, `BankStatementLine`

---

### BR-VAL-003: Tier 1 — referensi identik

| Field | Value |
|-------|-------|
| **ID** | BR-VAL-003 |
| **Category** | Validation |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-005 · Gate G-01 (SH-002) — inti otomasi mengganti filter manual |

**Rule:** Tier 1 menilai **hanya** `reference_no` hasil normalisasi — bukan
nominal, bukan tanggal.

**When:** Tier 1 dijalankan

**Then:** `reference_no` identik setelah normalisasi → hasil `MATCHED`, skor
**1,00**, 1 leg

**Else:** Belum identik → lanjut Tier 2

**Examples:**
- Bank kirim `QRIS-88213`, ledger punya `qris-88213` → setelah normalisasi
  identik → `MATCHED` skor 1,00 tanpa review.
- QRIS tanpa referensi (`reference_no` kosong) → Tier 1 dilewati, lanjut
  Tier 2.

**Related:**
- Use Case: UC-02
- Requirement: FR-005
- Process: PF-002
- Data Entity: `MatchResult`, `MatchResultLeg`

---

### BR-VAL-004: Tier 2 — nominal identik & selisih tanggal ≤ 1 hari

| Field | Value |
|-------|-------|
| **ID** | BR-VAL-004 |
| **Category** | Validation |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-005 · Gate G-01 (SH-002) · ambang `BR-CALC-001` (CON-005) |

**Rule:** Tier 2 menuntut **dua syarat sekaligus** — nominal sama dan tanggal
dekat; salah satu saja gugur → lanjut Tier 3.

**When:** Tier 2 dijalankan

**Then:** Nominal identik (selisih ≤ Rp 1) **dan** selisih tanggal ≤ 1 hari →
hasil `MATCHED`, skor **1,00**

**Else:** Tidak memenuhi → lanjut Tier 3

**Examples:**
- Transfer 1.250.000,00 di ledger tanggal 25, masuk bank tanggal 26 (value
  date beda 1 hari) → `MATCHED` skor 1,00.
- Nominal sama tapi tanggalnya 3 hari berbeda → gugur di syarat tanggal →
  lanjut Tier 3.

**Related:**
- Use Case: UC-02
- Requirement: FR-005
- Process: PF-002
- Data Entity: `MatchResult`

---

### BR-VAL-005: Tier 3 — agregasi (banyak ledger → 1 baris bank)

| Field | Value |
|-------|-------|
| **ID** | BR-VAL-005 |
| **Category** | Validation |
| **Priority** | Should |
| **Status** | [x] Confirmed |
| **Source** | FR-005, FR-006 · Gate G-02 (SH-002) — settlement & biaya terpisah tak pernah cocok 1:1 |

**Rule:** Sebuah baris bank boleh mewakili **sekelompok** transaksi ledger
(settlement harian), asal jumlahnya persis sama.

**When:** Tier 3 (agregasi) dijalankan

**Then:** Σ banyak transaksi ledger = 1 baris bank dalam toleransi → hasil
`AGGREGATED`, skor **0,95**, satu leg per transaksi

**Else:** Σ tidak sama → lanjut Tier 4

**Examples:**
- Bank A menyetor 1 settlement 92.400.000,00 yang berasal dari 17 transaksi
  ledger → `AGGREGATED` dengan 17 `MatchResultLeg`, skor 0,95.
- Σ 17 transaksi = 92.399.999,50 (kurang Rp 0,50) → masih lolos toleransi
  Rp 1; bila meleset Rp 500 → gugur, lanjut Tier 4.

**Related:**
- Use Case: UC-02
- Requirement: FR-005, FR-006
- Process: PF-002
- Data Entity: `MatchResult`, `MatchResultLeg`

---

### BR-VAL-006: Tier 4 — keterangan mirip + nominal identik

| Field | Value |
|-------|-------|
| **ID** | BR-VAL-006 |
| **Category** | Validation |
| **Priority** | Should |
| **Status** | [ ] Pending |
| **Source** | FR-005, FR-008 · SH-003 (derivasi analis — belum divalidasi ke stakeholder) |

**Rule:** Tier 4 **tidak pernah** menutup selisih sendiri — hasilnya selalu
masuk antrean review manusia.

**When:** Tier 4 dijalankan

**Then:** Keterangan mirip **dan** nominal identik → hasil `MATCHED`, skor
**0,85** + wajib masuk antrean review manusia

**Else:** Tidak memenuhi → lanjut Tier 5

**Examples:**
- Keterangan bank `BIAYA ADMIN BLN 09` vs ledger `biaya admin bulan 09`,
  nominal 2.500,00 sama → `MATCHED` skor 0,85, menunggu konfirmasi analis.
- Keterangan beda jauh (`SETOR TUNAI` vs `TRANSFER MASUK`) walau nominal sama
  → gugur → Tier 5.

**Related:**
- Use Case: UC-02, UC-03
- Requirement: FR-005, FR-008
- Process: PF-002
- Data Entity: `MatchResult`, `UnmatchedItem`

---

### BR-VAL-007: Tier 5 — nominal parsial (selisih ≤ Rp 1 / > Rp 1)

| Field | Value |
|-------|-------|
| **ID** | BR-VAL-007 |
| **Category** | Validation |
| **Priority** | Should |
| **Status** | [x] Confirmed |
| **Source** | FR-005, FR-009 · Gate G-01 + masukan SH-004 (kode alasan wajib) |

**Rule:** Tier 5 adalah tier terakhir — ia tidak menemukan pasangan, ia
**mengklasifikasikan** sisa menjadi selisih bernomor alasan.

**When:** Tier 5 (nominal parsial) dijalankan

**Then:** Selisih ≤ Rp 1 → `MATCHED`, skor **0,60** + antre review · Selisih
> Rp 1 → selisih dengan `reason_code = AMOUNT_DIFF`

**Else:** —

**Examples:**
- Ledger 1.000.000,00 vs bank 999.999,50 (selisih Rp 0,50) → `MATCHED` skor
  0,60, wajib direview analis.
- Ledger 1.000.000,00 vs bank 999.000,00 (selisih Rp 1.000) → selisih
  `AMOUNT_DIFF`, masuk antrean PF-003.

**Related:**
- Use Case: UC-02
- Requirement: FR-005, FR-009
- Process: PF-002
- Data Entity: `MatchResult`, `UnmatchedItem`

---

### BR-VAL-008: Split & leg banyak (1 baris bank → banyak ledger)

| Field | Value |
|-------|-------|
| **ID** | BR-VAL-008 |
| **Category** | Validation |
| **Priority** | Should |
| **Status** | [x] Confirmed |
| **Source** | FR-006 · Gate G-02 (SH-002) |

**Rule:** Setiap pasangan pada hasil `M:N` wajib tercatat sebagai `leg`
terpisah dengan kontribusi nominalnya masing-masing — tidak boleh ada hasil
pecahan tanpa rincian.

**When:** 1 baris bank dipasangkan ke banyak transaksi ledger (split)

**Then:** Sistem membuat satu `MatchResultLeg` per pasangan dan Σ nilai leg =
nilai baris bank (selisih ≤ Rp 1)

**Else:** Σ leg ≠ nilai baris → pasangan dibatalkan, jadi selisih
`AGG_UNRESOLVED`

**Examples:**
- 1 baris bank 50.000.000,00 terbagi ke 3 ledger (20jt + 20jt + 10jt) → 3
  leg, Σ = 50.000.000,00 → sah.
- Σ leg = 49.999.000,00 (meleset Rp 1.000) → pasangan dibatalkan → selisih
  `AGG_UNRESOLVED` ke PF-003.

**Related:**
- Use Case: UC-02
- Requirement: FR-006
- Process: PF-002
- Data Entity: `MatchResultLeg`, `MatchResult`

---

### BR-VAL-009: Wajib tepat satu dari 8 kode alasan selisih

| Field | Value |
|-------|-------|
| **ID** | BR-VAL-009 |
| **Category** | Validation |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-009 · SH-003, SH-004 (aspek B + masukan #3 — "selisih tanpa alasan tidak bisa ditindaklanjuti") |

**Rule:** Tidak ada status "selisih tanpa alasan". Setiap baris selisih
**wajib** membawa tepat satu `reason_code` dari daftar tertutup berikut:
`MISSING_REF`, `AMOUNT_DIFF`, `DATE_DIFF`, `DUPLICATE_REF`, `ORPHAN_BANK`,
`ORPHAN_LEDGER`, `AGG_UNRESOLVED`, `REVERSAL_MISMATCH`.

**When:** Sebuah baris tercatat sebagai selisih

**Then:** `reason_code` diisi **tepat satu** dari 8 kode: `MISSING_REF`,
`AMOUNT_DIFF`, `DATE_DIFF`, `DUPLICATE_REF`, `ORPHAN_BANK`, `ORPHAN_LEDGER`,
`AGG_UNRESOLVED`, `REVERSAL_MISMATCH`

**Else:** Kosong / di luar daftar → penyimpanan ditolak

**Examples:**
- Baris bank `QRIS-778` tidak ada padanan di ledger → `MISSING_REF`.
- Sistem mengisi dua kode (`AMOUNT_DIFF` + `DATE_DIFF`) → penyimpanan ditolak;
  harus dipilih satu alasan utama (yang kedua tampil sebagai keterangan).

**Related:**
- Use Case: UC-03
- Requirement: FR-009
- Process: PF-003
- Data Entity: `UnmatchedItem`

---

### BR-VAL-010: `NON_MATCHING` tetap tampil di semua keluaran

| Field | Value |
|-------|-------|
| **ID** | BR-VAL-010 |
| **Category** | Validation |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-010, FR-016 · SH-004 (masukan #2 — "menyembunyikan volume selisih wajar = red flag audit") |

**Rule:** Angka kecantikan bukan alasan untuk menyembunyikan data. Kategori
`NON_MATCHING` (selisih yang **sah** tidak berpasangan) wajib tampil utuh
beserta volumenya di setiap tempat selisih ditampilkan.

**When:** Selisih berkategori `NON_MATCHING` (sah, memang tidak berpasangan)

**Then:** **Tetap tampil** di register selisih, ringkasan harian, dan ekspor —
lengkap dengan kategori + volumenya

**Else:** Tidak boleh disembunyikan/difilter agar angka terlihat rapi

**Examples:**
- 14 baris `NON_MATCHING` (biaya bank 4, setoran tunai sendiri 10) tetap muncul
  di `O-01` ringkasan walau tidak perlu ditindaklanjuti.
- Laporan yang menyaring `NON_MATCHING` → dianggap tidak sah; Auditor (UC-10)
  wajib melihatnya.

**Related:**
- Use Case: UC-09
- Requirement: FR-010, FR-016
- Process: PF-003
- Data Entity: `UnmatchedItem`

---

### BR-VAL-011: Pasangan reversal dipasangkan lebih dulu

| Field | Value |
|-------|-------|
| **ID** | BR-VAL-011 |
| **Category** | Validation |
| **Priority** | Should |
| **Status** | [x] Confirmed |
| **Source** | FR-009 · SH-004 (kebutuhan audit — arus kas harian harus bersih) |

**Rule:** Reversal bukan dua selisih, melainkan **satu pasangan** — bila
pasangannya ada, keduanya netral dan tidak masuk hitungan selisih.

**When:** Pasangan reversal terdeteksi (referensi/keterangan berpasangan, arah
berlawanan)

**Then:** Dipasangkan lebih dulu sebagai satu pasangan; reversal tanpa pasangan
→ selisih `REVERSAL_MISMATCH` (bukan dua selisih `AMOUNT_DIFF`)

**Else:** —

**Examples:**
- Debit 5.000.000,00 tanggal 26 dengan referensi `RV-001` + kredit
  5.000.000,00 `RV-001` → dipasangkan, tidak dihitung sebagai selisih.
- Debit `RV-002` 750.000,00 tanpa kredit pasangan → satu selisih
  `REVERSAL_MISMATCH` (bukan dua `AMOUNT_DIFF`).

**Related:**
- Use Case: UC-02
- Requirement: FR-009
- Process: PF-002
- Data Entity: `BankStatementLine`, `InternalTransaction`, `UnmatchedItem`

## CALC — Perhitungan (4 aturan)

### BR-CALC-001: Toleransi nominal maksimal Rp 1

| Field | Value |
|-------|-------|
| **ID** | BR-CALC-001 |
| **Category** | Calculation |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-005, CON-005 · SH-001 (batas pembulatan disepakati di kickoff) |

**Rule:** Satu-satunya toleransi nominal di seluruh sistem adalah **Rp 1,00**
(pembulatan bank); di luar itu dianggap beda — tidak ada toleransi "mendekati".

**When:** Menghitung kecocokan nominal

**Then:** Toleransi **maksimal Rp 1,00** per baris (pembulatan) → dianggap sama

**Else:** Selisih > Rp 1 → dianggap beda, lanjut penilaian alasan

**Examples:**
- Ledger 250.000,00 vs bank 250.001,00 → selisih tepat Rp 1 → **sama**.
- Ledger 250.000,00 vs bank 250.002,00 → selisih Rp 2 → beda → masuk penilaian
  alasan (`AMOUNT_DIFF`).

**Related:**
- Use Case: UC-02
- Requirement: FR-005 (CON-005)
- Process: PF-002
- Data Entity: `MatchResult`, `UnmatchedItem`

---

### BR-CALC-002: Skor keyakinan per tier

| Field | Value |
|-------|-------|
| **ID** | BR-CALC-002 |
| **Category** | Calculation |
| **Priority** | Should |
| **Status** | [ ] Pending |
| **Source** | FR-008 · SH-003 (derivasi analis — ambang skor belum divalidasi) |

**Rule:** Skor selalu **turun mengikuti tier**, bukan angka bebas: makin rendah
skor, makin wajib campur tangan manusia.

**When:** Menetapkan skor keyakinan hasil

**Then:** T1 **1,00** · T2 **1,00** · T3 **0,95** · T4 **0,85** · T5 **0,60**;
skor < 1,00 selalu masuk antrean review

**Else:** Skor 1,00 boleh dinyatakan `MATCHED` tanpa review

**Examples:**
- Hasil Tier 3 (agregasi) skor 0,95 → otomatis masuk daftar review analis.
- Hasil Tier 1 skor 1,00 → langsung `MATCHED`, tidak membebani antrean review.

**Related:**
- Use Case: UC-02, UC-03
- Requirement: FR-008
- Process: PF-002
- Data Entity: `MatchResult`

---

### BR-CALC-003: Aging selisih dihitung per hari kalender

| Field | Value |
|-------|-------|
| **ID** | BR-CALC-003 |
| **Category** | Calculation |
| **Priority** | Must |
| **Status** | [ ] Pending |
| **Source** | FR-011 · SH-002 (aspek A — "tanpa umur & pemilik, selisih menumpuk") |

**Rule:** Umur selisih dihitung sejak **baris masuk `UnmatchedItem`**, bukan
sejak tanggal mutasi — agar penumpukan benar-benar terlihat.

**When:** Menghitung aging selisih

**Then:** `aging_days` = hari kalender sejak baris masuk `UnmatchedItem` sampai
`RESOLVED`; **> 1 hari** → peringatan + wajib ditugaskan

**Else:** ≤ 1 hari → urutan kerja normal

**Examples:**
- Selisih masuk antrean 25 Sep, masih `OPEN` tanggal 27 Sep → `aging_days = 2`
  → peringatan + wajib ada pemilik.
- Selisih dibuat dan selesai hari yang sama → `aging_days = 0` → urutan normal.

**Related:**
- Use Case: UC-03
- Requirement: FR-011
- Process: PF-003
- Data Entity: `UnmatchedItem`

---

### BR-CALC-004: Persentase carry-over ≤ 0,50%

| Field | Value |
|-------|-------|
| **ID** | BR-CALC-004 |
| **Category** | Calculation |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-021, CON-006 · SH-001, SH-002 (aspek A/C) |

**Rule:** Ambang tutup hari dihitung terhadap **baris mutasi hari itu**, bukan
terhadap transaksi ledger — pembulatan 2 desimal ke atas (`ceil`) agar lolos
0,5% tidak bisa dimainkan.

**When:** Menghitung persentase bawaan (carry-over)

**Then:** % = (selisih belum selesai ÷ total baris mutasi hari itu) × 100,
dibulatkan 2 desimal; **≤ 0,50%** baru boleh tutup hari

**Else:** > 0,50% → tutup hari ditahan, eskalasi

**Examples:**
- 68 sisa dari 18.042 baris = 0,38% → lolos, tutup hari boleh.
- 91 sisa dari 18.042 baris = 0,5045% → dibulatkan 0,51% → **ditahan** +
  eskalasi Manajer Keuangan.

**Related:**
- Use Case: UC-06
- Requirement: FR-021 (CON-006)
- Process: PF-006
- Data Entity: `ReconciliationRun`, `UnmatchedItem`

---

## AUTH — Otorisasi & kendali akses (8 aturan)

### BR-AUTH-001: Persetujuan koreksi wajib + eskalasi > Rp 5 juta

| Field | Value |
|-------|-------|
| **ID** | BR-AUTH-001 |
| **Category** | Authorization |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-013, CON-003 · SH-001 (gate G-04: restrukturisasi jadi 2 langkah + ambang eskalasi) |

**Rule:** Tidak ada koreksi yang langsung mengubah buku besar — semua lewat
antrean persetujuan; yang melebihi ambang **wajib naik ke Manajer Keuangan**
(total 3 pihak: analis, supervisor, manajer).

**When:** Koreksi diajukan untuk disetujui

**Then:** **Supervisor** menyetujui; nilai > **Rp 5.000.000** wajib **Manajer
Keuangan** (total 3 pihak: analis, supervisor, manajer) sebelum boleh `POSTED`

**Else:** Tanpa persetujuan lengkap → tetap `SUBMITTED`/`APPROVED`, tidak
menjadi baris ledger

**Examples:**
- Koreksi Rp 1.200.000,00 oleh analis → Supervisor boleh menyetujui → jadi
  baris ledger siklus berikutnya.
- Koreksi Rp 8.500.000,00 → Supervisor menyetujui lalu wajib diteruskan ke
  Manajer Keuangan; tanpa tanda tangan manajer status tetap `APPROVED`,
  buku besar tak tersentuh.

**Related:**
- Use Case: UC-04, UC-05
- Requirement: FR-013 (CON-003)
- Process: PF-004
- Data Entity: `AdjustmentRequest`, `SystemUser`

---

### BR-AUTH-002: Penyetuju tidak boleh mengajukan

| Field | Value |
|-------|-------|
| **ID** | BR-AUTH-002 |
| **Category** | Authorization |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-013 · SH-001 (gate G-04 — pemisahan tugas / segregation of duties) |

**Rule:** Prinsip *segregation of duties*: satu orang tidak boleh memegang dua
sisi koreksi yang sama.

**When:** Persetujuan direkam

**Then:** `approved_by` **≠** `requested_by` (kendali ganda)

**Else:** Sama → penolakan otomatis, jejak `ADJ_REJECT`

**Examples:**
- Analis A mengajukan `ADJ-0042` → Supervisor B menyetujui → sah.
- Supervisor B menyetujui koreksi yang diajukan B sendiri → sistem menolak dan
  menulis jejak `ADJ_REJECT` tanpa pertanyaan.

**Related:**
- Use Case: UC-05
- Requirement: FR-013
- Process: PF-004
- Data Entity: `AdjustmentRequest`, `AuditEvent`

---

### BR-AUTH-003: Super Admin tanpa persetujuan & tanpa tutup hari

| Field | Value |
|-------|-------|
| **ID** | BR-AUTH-003 |
| **Category** | Authorization |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-019, FR-021 · SH-005 (aspek F — admin teknis bukan pengambil keputusan bisnis) |

**Rule:** Keistimewaan teknis **tidak** membawa wewenang keuangan: Super Admin
dilarang menyetujui koreksi dan menutup hari.

**When:** System Analyst mengubah aturan / menutup hari

**Then:** SA boleh mengubah parameter aturan, **tidak** boleh menyetujui
perubahan sendiri dan **tidak** boleh menutup hari

**Else:** Percobaan → ditolak + tercatat di jejak audit

**Examples:**
- SA menerbitkan `RuleVersion 12` lalu membuka koreksi miliknya sendiri → tombol
  Setujui tidak tersedia untuk akun SA.
- SA menekan tutup hari pukul 07:00 → sistem menolak + menulis jejak percobaan
  yang menunggu tinjauan Supervisor.

**Related:**
- Use Case: UC-08
- Requirement: FR-019, FR-021
- Process: PF-007
- Data Entity: `SystemUser`, `AdjustmentRequest`, `ReconciliationRun`

---

### BR-AUTH-004: Auditor hanya baca-saja

| Field | Value |
|-------|-------|
| **ID** | BR-AUTH-004 |
| **Category** | Authorization |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-018 · SH-004 (aspek D — audit butuh jejak penuh, bukan kemampuan ubah) |

**Rule:** Auditor membaca segalanya, mengubah apa pun — termasuk laporan yang
sedang dilihatnya.

**When:** Auditor mengakses sistem

**Then:** **Baca-saja** atas seluruh data, laporan, dan unduhan; seluruh
tulis/hapus ditolak

**Else:** Tindakan tulis → ditolak + tercatat

**Examples:**
- Auditor membuka `O-05` laporan jejak audit → filter & ekspor aktif, seluruh
  tombol sunting tidak dirender.
- Auditor mencoba menyetujui lewat URL langsung → sistem menolak + jejak
  mencatat percobaan akses ditolak.

**Related:**
- Use Case: UC-09, UC-10
- Requirement: FR-018
- Process: — (berlaku lintas proses)
- Data Entity: `SystemUser`, `AuditEvent`

---

### BR-AUTH-005: Tidak ada peran boleh mengubah baris mentah

| Field | Value |
|-------|-------|
| **ID** | BR-AUTH-005 |
| **Category** | Authorization |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-022, NFR-DATA-002 · SH-004, SH-008 (gate G-05 — sanksi bila data mentah diubah) |

**Rule:** Ini **bukan** aturan "siapa yang boleh", melainkan larangan mutlak
untuk **semua** peran: data mentah tidak boleh diubah siapa pun, kapan pun.

**When:** Peran manapun mencoba mengubah/menghapus baris mutasi mentah

**Then:** **Ditolak 100%** — tidak ada satu pun peran yang memiliki izin tulis
atas data mentah

**Else:** —

**Examples:**
- Super Admin memperbaiki saldo baris `BankStatementLine` langsung → ditolak;
  jalurnya hanya impor batch ulang yang menghasilkan batch baru.
- Manajer Keuangan mengubah tanggal mutasi lama → tetap ditolak; yang boleh
  hanya unggah ulang file bank (batch baru, jejak lama utuh).

**Related:**
- Use Case: UC-03, UC-10
- Requirement: FR-022
- Process: — (berlaku lintas proses)
- Data Entity: `BankStatementLine`, `InternalTransaction`

---

### BR-AUTH-006: Autentikasi dua faktor + kunci setelah 5 kali gagal

| Field | Value |
|-------|-------|
| **ID** | BR-AUTH-006 |
| **Category** | Authorization |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | NFR-SEC-001 · SH-005 (aspek E — keamanan sebelum otomasi) |

**Rule:** Gerbang masuk sistem: dua faktor untuk semua yang masuk, dan akun
dikunci otomatis setelah lima kali percobaan gagal — hanya Supervisor yang
membuka kunci.

**When:** Pengguna masuk ke sistem

**Then:** Dua langkah (2FA) wajib untuk semua peran; akun terkunci setelah
**5 kali gagal** berturut-turut; hanya Supervisor membuka kunci + tercatat di jejak

**Else:** Percobaan ke-5 gagal → akun `LOCKED`

**Examples:**
- Kata sandi benar, kode OTP salah → percobaan gagal ke-1 dari 5.
- Lima kali salah → akun `LOCKED`; Supervisor membuka kunci dan peristiwa itu
  tercatat di `AuditEvent`.

**Related:**
- Use Case: semua (UC-01…UC-12 — prasyarat masuk)
- Requirement: NFR-SEC-001
- Process: — (berlaku lintas proses)
- Data Entity: `SystemUser`, `AuditEvent`

---

### BR-AUTH-007: Masking data pribadi

| Field | Value |
|-------|-------|
| **ID** | BR-AUTH-007 |
| **Category** | Authorization |
| **Priority** | Should |
| **Status** | [x] Confirmed |
| **Source** | FR-018, CON-007, NFR-SEC-004 · SH-004 (aspek D) + ketentuan UU PDP |

**Rule:** Data pribadi ditampilkan sebagian saja bagi yang tidak berkepentingan
langsung — prinsip *data minimization* UU PDP.

**When:** Menampilkan/mengekspor nomor rekening & nama nasabah

**Then:** Ditampilkan **terpotong** (mis. `1234xxxx5678`) untuk peran tanpa
kebutuhan penuh; nilai penuh hanya Supervisor & Auditor dan tercatat di jejak;
data pribadi tidak masuk log operasional

**Else:** —

**Examples:**
- Tabel `O-02` memperlihatkan nomor `1234xxxx5678`, bukan rekening penuh.
- Ekspor `O-04` memakai pola potong yang sama; Supervisor membuka nilai penuh →
  peristiwa terbuka penuh tercatat di jejak.

**Related:**
- Use Case: UC-09, UC-10
- Requirement: FR-018, CON-007, NFR-SEC-004
- Process: — (berlaku lintas proses)
- Data Entity: `BankStatementLine`, `InternalTransaction`

---

### BR-AUTH-008: `acting_as_role` saat rangkap peran

| Field | Value |
|-------|-------|
| **ID** | BR-AUTH-008 |
| **Category** | Authorization |
| **Priority** | Should |
| **Status** | [x] Confirmed |
| **Source** | FR-023 · SH-003 (aspek C — peran ganda di tim kecil) |

**Rule:** Rangkap peran harus **eksplisit dipilih**, tidak otomatis mengikuti
peran terkuat; setiap tindakan tetap mengikuti peran yang sedang dipakai.

**When:** Supervisor mengerjakan tindakan analis (rangkap peran)

**Then:** Satu akun dengan `acting_as_role` tercatat pada **setiap** jejak
tindakan — bukan akun ganda

**Else:** Jejak tanpa `acting_as_role` → tindakan ditolak

**Examples:**
- Supervisor yang juga Analis masuk memilih `Analis` → mengeksekusi
  pencocokan; `acting_as_role = Analyst` menempel di setiap jejak tindakannya.
- Tindakan tersimpan tanpa `acting_as_role` (peran tak dipilih) → sistem
  menolak penyimpanan.

**Related:**
- Use Case: UC-03, UC-06
- Requirement: FR-023
- Process: PF-004
- Data Entity: `SystemUser`, `AuditEvent`

## WF — Alur kerja (9 aturan)

### BR-WF-001: Setiap selisih punya pemilik; > 1 hari wajib ditindak

| Field | Value |
|-------|-------|
| **ID** | BR-WF-001 |
| **Category** | Workflow |
| **Priority** | Must |
| **Status** | [ ] Pending |
| **Source** | FR-011 · SH-002 (aspek A — "tanpa umur & pemilik, selisih menumpuk tanpa pemilik") |

**Rule:** Selisih tidak boleh berstatus *tanpa pemilik* — sejak masuk antrean
harus sudah ditugaskan; umur > 1 hari memaksa peringatan ke Supervisor, bukan
sekadar catatan.

**When:** Selisih ditugaskan ke analis

**Then:** Selisih selalu punya pemilik; `aging_days > 1` → peringatan +
eskalasi ke Supervisor

**Else:** ≤ 1 hari → urutan kerja normal

**Examples:**
- 68 sisa per 26 Sep, 46 di antaranya `OPEN` H+1 → seluruhnya ditugaskan ke 3
  analis; Supervisor menerima satu daftar peringatan.
- Selisih berumur 0 hari dan sudah punya pemilik → masuk urutan kerja biasa,
  tanpa eskalasi.

**Related:**
- Use Case: UC-03
- Requirement: FR-011
- Process: PF-003
- Data Entity: `UnmatchedItem`, `SystemUser`

---

### BR-WF-002: Setiap aksi manual wajib alasan + tercatat jejak

| Field | Value |
|-------|-------|
| **ID** | BR-WF-002 |
| **Category** | Workflow |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-012, FR-022 · SH-003 (aspek C — pertanggungjawaban audit) |

**Rule:** Tidak ada aksi manual anonim: alasan tertulis adalah syarat simpan,
dan jejak audit mencatat **siapa, kapan, dari mana**.

**When:** Tindakan manual dijalankan (cocokkan manual, ajukan koreksi)

**Then:** Wajib menyimpan **aktor, waktu, dan alasan** → tercatat sebagai
`MATCH_MANUAL` / pengajuan koreksi

**Else:** Alasan kosong → penyimpanan ditolak

**Examples:**
- Analis menutup selisih tanpa alasan → formulir terkunci, tombol simpan mati
  sampai alasan diisi.
- Aksi tersimpan → jejak berisi `analis_b · Supervisor · 26 Sep 08:14 · IP
  kantor · "koreksi referensi ganda"`.

**Related:**
- Use Case: UC-03
- Requirement: FR-012, FR-022
- Process: PF-003
- Data Entity: `AuditEvent`, `UnmatchedItem`

---

### BR-WF-003: Koreksi menjadi baris ledger berikutnya, bukan sunting

| Field | Value |
|-------|-------|
| **ID** | BR-WF-003 |
| **Category** | Workflow |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-013, FR-014, NFR-DATA-002 · SH-004 (masukan #4 — audit tidak menerima baris diam-diam berubah) |

**Rule:** Perbaikan di atas buku besar selalu **tambahan** (append), tidak
pernah **sunting** — baris asli tetap utuh dan tetap terlihat.

**When:** Koreksi disetujui

**Then:** Tercatat sebagai **baris ledger baru pada siklus berikutnya**; baris
lama tidak pernah diubah

**Else:** —

**Examples:**
- Koreksi `ADJ-0042` disetujui 26 Sep → baris ledger baru muncul efektif
  27 Sep, baris asli 24 Sep tetap `ORIGINAL` dan ikut tampil di `O-05`.
- Persetujuan tidak pernah diberikan → koreksi tetap `PENDING`, buku besar
  tidak pernah berubah diam-diam.

**Related:**
- Use Case: UC-05
- Requirement: FR-013, FR-014
- Process: PF-004
- Data Entity: `InternalTransaction`, `AdjustmentRequest`

---

### BR-WF-004: Peringatan dini WASPADA > 90 / KRITIS > 200 + respons ≤ 15 menit

| Field | Value |
|-------|-------|
| **ID** | BR-WF-004 |
| **Category** | Workflow |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-017 · keputusan desain #4 (2 tier peringatan) + SH-001 (ambang eskalasi) |

**Rule:** Dua tingkat peringatan, dua tingkat komitmen waktu — peringatan tanpa
SLA respons dianggap dekoratif; keterlambatan respons justru memicu kiriman
ulang.

**When:** Pukul **06:30 WIB**

**Then:** Selisih > 90 → **WASPADA** (ditandai di laporan) · > 200 → **KRITIS**
+ eskalasi Supervisor; respons ≤ **15 menit**, bila tidak → kirim ulang **maks
3×** + catat keterlambatan

**Else:** ≤ 90 → tanpa peringatan

**Examples:**
- Pukul 06:30 terbaca 120 selisih → WASPADA dan ditandai di laporan.
- Terbaca 430 selisih → KRITIS, Supervisor wajib respons dalam 15 menit; tidak
  ada respons → kirim ulang (maks 3×) + keterlambatan tercatat di jejak.

**Related:**
- Use Case: UC-07
- Requirement: FR-017
- Process: PF-005
- Data Entity: `UnmatchedItem`, `AuditEvent`

---

### BR-WF-005: Perubahan aturan berlaku mulai run berikutnya

| Field | Value |
|-------|-------|
| **ID** | BR-WF-005 |
| **Category** | Workflow |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-019, FR-020 · SH-005 (aspek F — hasil hari ini harus bisa dijelaskan apa adanya) |

**Rule:** Setiap run membawa **satu** versi aturan yang tertutup dan tidak
berubah — versi aktif tidak bisa diedit, hanya diganti; versi lama tetap
tersimpan untuk menjelaskan hasil lampau.

**When:** Parameter aturan diubah

**Then:** Perubahan **berlaku pada run berikutnya** (`effective_from`); run
lampau tetap dijelaskan dengan versi aturannya

**Else:** Tanpa persetujuan Supervisor → parameter dikembalikan, usulan tetap
tercatat

**Examples:**
- SA menerbitkan `RuleVersion 12` pukul 10:00 → run pagi (mulai 06:00) tetap
  memakai `v11`, run besok memakai `v12`.
- Usulan perubahan ambang Tier 4 tanpa persetujuan Supervisor → parameter
  dikembalikan ke nilai lama, usulan tetap tercatat di jejak.

**Related:**
- Use Case: UC-08, UC-11
- Requirement: FR-019, FR-020
- Process: PF-007
- Data Entity: `MatchRule`, `AuditEvent`

---

### BR-WF-006: Tutup hari hanya bila carry-over ≤ 0,5% dan sebelum 07:30

| Field | Value |
|-------|-------|
| **ID** | BR-WF-006 |
| **Category** | Workflow |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-021, CON-006 · SH-001, SH-002 (aspek A/C) |

**Rule:** Tutup hari adalah gerbang ganda: **syarat kuantitatif** (carry-over)
**dan syarat waktu** (sebelum cut-off) harus sama-sama terpenuhi.

**When:** Tutup hari dijalankan

**Then:** Selisih ≤ **0,5%** (≤ 90 baris) dan sebelum **07:30 WIB** → run
`CLOSED` + tanda tangan harian

**Else:** Melebihi → hari **tidak boleh ditutup**, eskalasi Manajer Keuangan

**Examples:**
- 07:15, sisa 68/18.042 = 0,38% → run `CLOSED`, tanda tangan harian terisi.
- 07:10 dengan sisa 91 baris (0,51%) → tidak boleh ditutup, eskalasi Manajer
  Keuangan.

**Related:**
- Use Case: UC-06
- Requirement: FR-021 (CON-006)
- Process: PF-006
- Data Entity: `ReconciliationRun`, `SystemUser`

---

### BR-WF-007: Jalur ulang (RERUN) idempoten — tidak ada duplikasi

| Field | Value |
|-------|-------|
| **ID** | BR-WF-007 |
| **Category** | Workflow |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-005, NFR-DATA-001 · gate G-06 (SH-002) + keputusan desain OQ-004 |

**Rule:** Mengulang berarti **mengulang**, bukan **menambah** — dan hasil
ulangan tidak menghapus pekerjaan manual atau koreksi yang sudah terlanjur
dipercaya.

**When:** Proses dijalankan ulang (`RERUN`)

**Then:** **Idempoten**: 1 hari = 1 run, tanpa baris ganda, tindakan manual &
koreksi lama **dipertahankan**, hasil identik bila data tidak berubah

**Else:** —

**Examples:**
- RERUN ke-3 pada `RUN-2026-09-26` → tetap 18.042 baris hasil, bukan 54.126.
- RERUN setelah SA mematikan aturan tier 4 → hasil tier 4 dihitung ulang,
  tetapi koreksi `ADJ-0042` dan cocokkan manual analis tetap dipertahankan.

**Related:**
- Use Case: UC-12
- Requirement: FR-005, NFR-DATA-001
- Process: PF-002
- Data Entity: `ReconciliationRun`, `MatchResult`

---

### BR-WF-008: Bank telat → proses parsial, sisa menunggu

| Field | Value |
|-------|-------|
| **ID** | BR-WF-008 |
| **Category** | Workflow |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-001, FR-002 · SH-003 (aspek D — jangan blok semua bank gara-gara satu bank) |

**Rule:** Kematangan satu bank tidak boleh mengunci kerja semua bank —
kewajiban berjalan parsial, kekurangannya dinyatakan terbuka.

**When:** Ada bank yang belum mengirim saat run dimulai

**Then:** Proses tetap berjalan **parsial** dengan batch `PENDING`; batch
menyusul diproses hari itu juga; melewati 06:00 → masuk run berikutnya

**Else:** —

**Examples:**
- 06:00 Bank A–C siap, Bank D belum → run `PARTIAL` (3/4 bank, 14.907 baris);
  analis tetap mulai PF-003.
- Batch Bank D menyusul pukul 06:20 (sebelum tutup masuk 07:30) → diproses
  hari itu juga; bila melewati batas 06:00 → masuk run berikutnya.

**Related:**
- Use Case: UC-01, UC-12
- Requirement: FR-001, FR-002
- Process: PF-001
- Data Entity: `StatementBatch`, `ReconciliationRun`

---

### BR-WF-009: Fallback manual bila pukul 06:30 belum ada hasil

| Field | Value |
|-------|-------|
| **ID** | BR-WF-009 |
| **Category** | Workflow |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-002, FR-017, NFR-AVAIL-002 · SH-001, SH-006 (aspek E — layanan tetap jalan walau otomasi mati) |

**Rule:** Otomasi boleh gagal, kerja hari itu tidak boleh batal — pukul 06:30
ada rencana cadangan yang sudah ditentukan sejak awal.

**When:** Pukul 06:30 **belum ada hasil run** (proses lambat/gagal)

**Then:** Peringatan "proses belum selesai" terkirim + **jalur manual
diaktifkan**; run ditandai terlambat di jejak

**Else:** Hasil tersedia → jalur normal

**Examples:**
- 06:30 belum ada run selesai → peringatan + panduan jalan manual
  (filter/pencarian, target ≤ 4 jam per NFR-MAINT-002) dikirim ke analis.
- 06:28 hasil run sudah ada → jalur normal, fallback tidak dipicu.

**Related:**
- Use Case: UC-07
- Requirement: FR-002, FR-017, NFR-AVAIL-002
- Process: PF-005
- Data Entity: `ReconciliationRun`, `AuditEvent`

## CON — Kendali data (5 aturan)

### BR-CON-001: Satu baris bank hanya boleh dipakai sekali

| Field | Value |
|-------|-------|
| **ID** | BR-CON-001 |
| **Category** | Constraint |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-007 · Gate G-03 (SH-004) — aturan keras, tanpa pengecualian |

**Rule:** Anti *double-spending* pasangan: satu baris bank **satu** nasib dan
satu transaksi ledger hanya sekali dipakai — pasangan ganda tidak mungkin
terjadi.

**When:** Sistem memasangkan baris

**Then:** Satu baris bank hanya boleh punya **satu** hasil; satu transaksi
ledger hanya dipakai **sekali** → pasangan ganda ditolak

**Else:** Upaya ke-2 → ditolak + jejak audit

**Examples:**
- 1 settlement 92.400.000,00 sudah `AGGREGATED` ke 17 ledger → baris itu tidak
  muncul lagi sebagai kandidat tier mana pun.
- Upaya memasangkan lagi baris yang sama lewat aksi manual → ditolak + jejak
  percobaan tercatat.

**Related:**
- Use Case: UC-03
- Requirement: FR-007
- Process: PF-002
- Data Entity: `MatchResult`, `BankStatementLine`

---

### BR-CON-002: Koreksi ≠ edit data lama

| Field | Value |
|-------|-------|
| **ID** | BR-CON-002 |
| **Category** | Constraint |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-014 · SH-004 (masukan #4) |

**Rule:** Koreksi tidak pernah menimpa — ia menambah jejak baru sehingga
kesalahan lama **tetap terbaca** untuk audit.

**When:** Koreksi diajukan / disetujui

**Then:** Koreksi **tidak pernah** mengubah data lama — selalu menambah baris
baru

**Else:** —

**Examples:**
- `ADJ-0042` disetujui → dua baris tampil di `O-05`: asli `ORIGINAL` +
  koreksi `POSTED` dengan nomor rujukan sama.
- Tidak ada operasi tulis yang menunjuk baris `ORIGINAL` — perbaikan selalu
  berupa baris tambahan.

**Related:**
- Use Case: UC-05
- Requirement: FR-014
- Process: PF-004
- Data Entity: `InternalTransaction`, `AdjustmentRequest`

---

### BR-CON-003: Jejak audit append-only + segel rantai

| Field | Value |
|-------|-------|
| **ID** | BR-CON-003 |
| **Category** | Constraint |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-022, NFR-SEC-003, CON-001 · SH-004, SH-008 (gate G-05) |

**Rule:** Jejak audit hanya bertambah, tidak pernah diubah/dihapus —
integritasnya dijaga segel rantai sehingga perusakan ketahuan cepat.

**When:** Menyimpan jejak audit & data mentah

**Then:** **Append-only**: hanya tambah, 0 baris bisa diubah/dihapus; setiap
baris membawa segel perhitungan baris sebelumnya, perusakan terdeteksi < 5 menit

**Else:** Upaya perubahan → ditolak + laporan perusakan

**Examples:**
- 12.508 kejadian pada `RUN-2026-09-26` disimpan berurutan dengan segel
  baris sebelumnya → memutus satu entri membuat segel berikutnya tak tercocok
  dan terdeteksi < 5 menit.
- Permintaan hapus jejak lewat antarmuka → endpoint tidak ada; sistem menolak
  dan menulis laporan perusakan.

**Related:**
- Use Case: UC-10
- Requirement: FR-022
- Process: — (berlaku lintas proses)
- Data Entity: `AuditEvent`

---

### BR-CON-004: Cut-off 06:00 / tutup 07:30 WIB

| Field | Value |
|-------|-------|
| **ID** | BR-CON-004 |
| **Category** | Constraint |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-001, CON-002 · SH-004 (cut-off audit), SH-001 (batas tutup hari) |

**Rule:** Batas waktu mutasi adalah batas keras: mutasi setelah cut-off
**terhitung hari berikutnya** — bukan tundaan, tapi pergeseran tanggal efektif.

**When:** Menentuan batas hari kerja

**Then:** Mutasi dinilai final pukul **06:00 WIB**; rekonsiliasi wajib selesai
**07:30 WIB**

**Else:** Melewati → ditandai terlambat di jejak audit

**Examples:**
- Mutasi bank masuk 06:20 → masuk run hari berikutnya, hari ini tetap
  `PARTIAL` bila Bank D belum ada.
- Rekonsiliasi selesai 07:35 → ditandai terlambat di jejak audit, tetapi run
  tetap boleh ditutup bila syarat carry-over terpenuhi.

**Related:**
- Use Case: UC-01, UC-06
- Requirement: FR-001 (CON-002)
- Process: PF-001, PF-006
- Data Entity: `ReconciliationRun`, `StatementBatch`

---

### BR-CON-005: Mata uang IDR, 2 desimal, tanpa konversi

| Field | Value |
|-------|-------|
| **ID** | BR-CON-005 |
| **Category** | Constraint |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | CON-004 · SH-001 (sistem hanya melayani IDR) |

**Rule:** Satu mata uang, satu format — seluruh angka disimpan dan ditampilkan
dalam IDR dua desimal; sistem tidak mengenal kurs.

**When:** Seluruh nilai diproses

**Then:** Dalam **IDR, 2 desimal, tanpa konversi mata uang**

**Else:** Ada nilai non-IDR → batch `REJECTED`

**Examples:**
- 1.234.567,89 disimpan persis `1234567.89`, ditampilkan `Rp 1.234.567,89`.
- Berkas dengan nilai berkedok mata uang asing → batch `REJECTED` oleh
  BR-VAL-001, tidak ada konversi kurs yang perlu dilakukan.

**Related:**
- Use Case: UC-01
- Requirement: CON-004
- Process: PF-001
- Data Entity: `BankStatementLine`, `InternalTransaction`

---

## RET — Retensi & arsip (3 aturan)

### BR-RET-001: Data mentah & jejak audit utuh 10 tahun

| Field | Value |
|-------|-------|
| **ID** | BR-RET-001 |
| **Category** | Retention |
| **Priority** | Must |
| **Status** | [x] Confirmed |
| **Source** | FR-022, NFR-COMP-001, CON-001 · SH-008 (ketentuan pengawasan keuangan) |

**Rule:** Retensi 10 tahun mencakup **seluruh rantai bukti** — baris mentah,
hasil pencocokan, selisih, koreksi, sampai jejak audit — tanpa penghapusan
parsial.

**When:** Penyimpanan & permintaan bukti

**Then:** Baris mentah, hasil pencocokan, selisih, koreksi, dan jejak audit
**diretensi ≥ 10 tahun** dan dapat direkonstruksi utuh; penghapusan otomatis
tidak pernah menyentuh rentang < 10 tahun

**Else:** Ketersediaan bukti per periode > 1 hari kerja → ketidakpatuhan

**Examples:**
- Auditor meminta bukti mutasi 2016 → seluruh rantai (baris → hasil → jejak)
  tersedia utuh pada 2026.
- Permintaan "bersihkan" data mentah usia 3 tahun → tidak ada penghapusan yang
  menyentuh rentang < 10 tahun.

**Related:**
- Use Case: UC-10
- Requirement: FR-022, NFR-COMP-001 (CON-001)
- Process: — (berlaku lintas proses)
- Data Entity: `BankStatementLine`, `InternalTransaction`, `AuditEvent`

---

### BR-RET-002: Versi aturan diarsipkan per run

| Field | Value |
|-------|-------|
| **ID** | BR-RET-002 |
| **Category** | Retention |
| **Priority** | Must |
| **Status** | [ ] Pending |
| **Source** | FR-020 · SH-004 (audit — "hasil masa lalu harus bisa dijelaskan dengan aturan masa lalu") |

**Rule:** Tiap run memakai versi aturan yang **dibekukan** — arsip per run
menyimpan `rule_version_id`, dan run tanpa versi aturan tidak boleh ditutup.

**When:** Setiap run menjalankan pencocokan

**Then:** Run menyimpan **ID versi aturan** yang dipakai (arsip ruleset per run)

**Else:** Run tanpa versi aturan → tidak boleh ditutup

**Examples:**
- `RUN-2026-09-26` mencatat `rule_version_id = 11` — aturan diubah 27 Sep tidak
  mengubah penjelasan run tanggal 26.
- Auditor membuka run 2024 → masih bisa membaca aturan persis yang dipakai kala
  itu lewat arsip ruleset.

**Related:**
- Use Case: UC-08
- Requirement: FR-020
- Process: PF-007
- Data Entity: `ReconciliationRun`, `MatchRule`, `AuditEvent`

---

### BR-RET-003: Ringkasan & snapshot laporan disimpan 2 tahun

| Field | Value |
|-------|-------|
| **ID** | BR-RET-003 |
| **Category** | Retention |
| **Priority** | Should |
| **Status** | [x] Confirmed |
| **Source** | FR-015, NFR-COMP-001 · SH-001, SH-004 (aspek A/B) |

**Rule:** Laporan ringkas disimpan terpisah dan lebih singkat dari data mentah
— 2 tahun untuk analisis, **bukan pengganti** bukti 10 tahun.

**When:** Ringkasan harian dihasilkan

**Then:** Diretensi **2 tahun** untuk analisis — **bukan pengganti** data
mentah 10 tahun

**Else:** —

**Examples:**
- `O-01` ringkasan 26 Sep 2026 masih tersimpan hingga 26 Sep 2028.
- Saat ringkasan lewat 2 tahun dibuang, baris `BankStatementLine` hari itu
  tetap ada (retensi 10 tahun, BR-RET-001).

**Related:**
- Use Case: UC-09
- Requirement: FR-015, NFR-COMP-001
- Process: PF-006
- Data Entity: — (snapshot laporan `O-01`)

---

# Lampiran

## A. Ambang & angka aturan (quick reference)

| Ambang | Nilai | Aturan |
|--------|-------|--------|
| Toleransi nominal | Rp 1,00 / baris | `BR-CALC-001` |
| Toleransi tanggal Tier 2 | ≤ 1 hari | `BR-VAL-004` |
| Skor per tier | 1,00 / 1,00 / 0,95 / 0,85 / 0,60 | `BR-CALC-002` |
| Aging peringatan | > 1 hari kalender | `BR-CALC-003` |
| Carry-over tutup hari | ≤ 0,50% (≤ 90 baris) | `BR-CALC-004`, `BR-WF-006` |
| Eskalasi koreksi | > Rp 5.000.000 | `BR-AUTH-001` |
| Kunci akun | 5 kali gagal | `BR-AUTH-006` |
| Peringatan WASPADA | > 90 selisih `OPEN` (0,5%) | `BR-WF-004` |
| Peringatan KRITIS | > 200 selisih | `BR-WF-004` |
| Respons Supervisor | ≤ 15 menit, kirim ulang maks 3× | `BR-WF-004` |
| Tutup hari | ≤ 0,50% & sebelum 07:30 WIB | `BR-WF-006`, `BR-CON-004` |
| Cut-off mutasi | 06:00 WIB | `BR-CON-004` |
| Fallback manual | aktif pukul 06:30 | `BR-WF-009` |
| Retensi data mentah + jejak | 10 tahun | `BR-RET-001` |
| Retensi ringkasan | 2 tahun | `BR-RET-003` |

## B. Status sumber (menunggu jawaban pemangku kepentingan)

| Aturan | Status | Menunggu |
|--------|--------|----------|
| `BR-VAL-001` | Pending | konfirmasi FR-003 (gagal ambil: retry ≤ 3×, escalate) |
| `BR-VAL-006` | Pending | validasi ambang Tier 4 ke SH-003 |
| `BR-CALC-002` | Pending | validasi daftar skor tier ke SH-003 |
| `BR-CALC-003` | Pending | konfirmasi FR-011 (batas aging eskalasi) |
| `BR-WF-001` | Pending | konfirmasi FR-011 (siapa pemilik & batas waktu) |
| `BR-RET-002` | Pending | konfirmasi FR-020 (kedalaman arsip per run) |

> FR-003, FR-008, FR-011, dan FR-020 masih `Pending` di
> `REQUIREMENTS-MATRIX.md` — status aturan mengikuti status FR sumbernya agar
> konsisten.

## C. Peta FR → aturan bisnis

| FR | Aturan | FR | Aturan |
|----|--------|----|--------|
| FR-001 | `BR-WF-008`, `BR-CON-004` | FR-013 | `BR-AUTH-001`, `BR-AUTH-002`, `BR-WF-003` |
| FR-002 | `BR-WF-008`, `BR-WF-009` | FR-014 | `BR-WF-003`, `BR-CON-002` |
| FR-003 | `BR-VAL-001` | FR-015 | `BR-RET-003` |
| FR-004 | `BR-VAL-002` | FR-016 | `BR-VAL-010` |
| FR-005 | `BR-VAL-003/004/005/006/007`, `BR-CALC-001`, `BR-WF-007` | FR-017 | `BR-WF-004`, `BR-WF-009` |
| FR-006 | `BR-VAL-005`, `BR-VAL-008` | FR-018 | `BR-AUTH-004`, `BR-AUTH-007` |
| FR-007 | `BR-CON-001` | FR-019 | `BR-AUTH-003`, `BR-WF-005` |
| FR-008 | `BR-VAL-006`, `BR-CALC-002` | FR-020 | `BR-WF-005`, `BR-RET-002` |
| FR-009 | `BR-VAL-007`, `BR-VAL-009`, `BR-VAL-011` | FR-021 | `BR-CALC-004`, `BR-WF-006`, `BR-AUTH-003` |
| FR-010 | `BR-VAL-010` | FR-022 | `BR-AUTH-005`, `BR-CON-003`, `BR-RET-001`, `BR-WF-002` |
| FR-011 | `BR-CALC-003`, `BR-WF-001` | FR-023 | `BR-AUTH-008` |
| FR-012 | `BR-WF-002` | | |

## D. Change Log

| Versi | Tanggal | Perubahan |
|-------|---------|-----------|
| 1.0 | 27 Sep 2026 | Katalog awal: 40 aturan (VAL 11 · CALC 4 · AUTH 8 · WF 9 · CON 5 · RET 3), ID & bunyi When/Then/Else identik dengan `DESAIN-PROGRAM.md` §6. Menggantikan nama lama `BR-MATCH-*`. |

