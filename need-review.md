# need-review.md — Daftar Error & Pekerjaan Perbaikan

**Tanggal:** 26 September 2026 · **Diperbarui:** 27 September 2026
**Sumber:** cek silang otomatis `DESAIN-PROGRAM.md` ↔ `REQUIREMENTS-MATRIX.md` ↔ `ERD-MASTER.md` ↔ `NOTATION.md` ↔ `nfr.md`
**Aturan main:** butir **A (kecuali yang ditandai terbuka) dan B1–B2 sudah
diperbaiki 27 Sep 2026** — rincian di bagian **E**. Sisa yang belum:
**B3–B4** dan **C1–C6** (butuh keputusan/stakeholder).

Legenda: `[BUG]` salah/tidak konsisten · `[GAP]` dokumen belum ada · `[OPEN]` butuh keputusan user · `[COSMETIC]` rapi-rapi.

---

## A. Konsistensi — wajib diperbaiki

| # | Level | Lokasi | Masalah | Rencana perbaikan |
|---|-------|--------|---------|-------------------|
| A1 | `[BUG]` | `DESAIN-PROGRAM.md` baris 16 (indeks header §5) | Indeks masih menulis **"BPMN + Flow per PF"**, padahal §5 sudah memakai Flowchart berlabel aktor + DFD dan `NOTATION.md` sudah mencatat keputusan user "BPMN tidak dipakai" | Ganti sel notasi §5 jadi **`Flowchart berlabel aktor + DFD`** |
| A2 | `[BUG]` | `REQUIREMENTS-MATRIX.md` baris 78 (Coverage Summary) | Baris FR: `23 \| 17 \| 6 \| 0` — angka **salah**. Hitungan aktual: **19 Confirmed / 4 Pending** (Pending = FR-003, FR-008, FR-011, FR-020) | Ubah jadi `23 \| 19 \| 4 \| 0` |
| A3 | `[BUG]` | `DESAIN-PROGRAM.md` §5.3 (DFD), panah **M10** | Label `M10 register + aging` mengalir **turun ke kotak P4** — salah tujuan. M10 = P3 → D3 (register selisih + aging) | Putuskan ulang jalur: panah M10 harus menuju `(D3 UnmatchedItem + Aging)`, bukan ke P4 |
| A4 | `[BUG]` | `DESAIN-PROGRAM.md` §5.3, panah **M9** | Digambar `... <---- M9 antrean persetujuan` (masuk ke P3) — arah terbalik. Antrean dikirim **P3 → P4**, sedangkan keputusan **P4 → P3** (M12) | Balik label/arah: M9 turun P3→P4, M12 naik P4→P3 (M12 sudah benar) |
| A5 | `[BUG]` | `DESAIN-PROGRAM.md` §5.3, entitas **(D3 UnmatchedItem + Aging)** | Tidak punya panah masuk yang jelas; `M13 selisih + alasan` ditulis tapi sumbernya ambigu (P3/P4 ada di atas blok, panahnya ke bawah) | Susun ulang: P3/P4 → (turun) → D3, lalu D3 → M14 → P5 |
| A6 | `[BUG]` | `DESAIN-PROGRAM.md` §5.3, baris `[A3 AUDITOR] -- M17 unduh jejak audit -->` | Panah **menggantung** ke arah kosong (baris `(retensi 10 tahun)` tidak sejajar sebagai tujuan) | Rapikan: `[A3 AUDITOR]` → `\| M17` → `v (retensi 10 tahun)` |
| A7 | `[BUG]` | `DESAIN-PROGRAM.md` §5.3, tabel **Ringkasan aliran data** | Tidak lengkap: **M7, M10, M12, M13, M15, M17** tidak ada di tabel; baris `M14` berbunyi `P3/P5 → A2/A3` padahal di diagram M14 = **D3 → P5**; `M5` ditulis "E2 Ledger" vs diagram "E2 SISTEM LEDGER" | Lengkapi 17 baris M1…M17 + samakan penamaan E2 |
| A8 | `[BUG]` | `DESAIN-PROGRAM.md` §5.1 vs tabel **Serah terima antar aktor** | Tabel mencantumkan handoff **SISTEM → ANALIS** (setelah `[5]`), tetapi diagram hanya punya **2** kotak `SERAH TERIMA` (ANALIS→SUPERVISOR, SUPERVISOR→SISTEM) | Tambah kotak handoff SISTEM→ANALIS sebelum `[6]`, **atau** hapus baris itu dari tabel |
| A9 | `[BUG]` | `00-Global/ERD-MASTER.md` baris 456 | Tulisan tidak lengkap: `... eskalasi >Rp 5 juta → BR-WF.` (kode menggantung) | Ganti jadi `→ BR-AUTH-001 + BR-AUTH-002` (dual-control) atau `BR-WF-003` |
| A10 | `[OPEN]` | Aturan penomoran `DESAIN-PROGRAM.md` header vs §7 | Aturan wajib menyebut kode **`R-` (baris laporan)**, tetapi §7 memakai **`O-01…O-07`** dan tidak ada satu pun `R-` | Putuskan: (a) ganti `O-` jadi `R-`, atau (b) tambah di `NOTATION.md`: keluaran = `O-`, `R-` khusus baris contoh laporan (lalu beri `R-001…` di §7.2) |
| A11 | `[COSMETIC]` | `DESAIN-PROGRAM.md` §5.4 (peta hubungan PF) | Kolom `\|` vertikal tidak selalu sejajar di baris `(tak bisa manual)` — keterbacaan turun | Gambar ulang peta pakai generator seperti §5.1 |
| A12 | `[COSMETIC]` | `DESAIN-PROGRAM.md` §5.1, kotak `[SISTEM] 10` | Baris kedua inden 10 spasi, kotak lain 9 spasi | Samakan inden (atau regenerasi lewat skrip) |
| A13 | `[COSMETIC]` | `DESAIN-PROGRAM.md` §6 catatan penamaan | `BR-CON-*` (aturan) vs `CON-001…009` (kendala SRS) rawan tertukar — sudah ada catatan, tapi belum ada penanda di `GLOSSARY.md` | Tambah entri glossary: "BR-CON vs CON" |

## B. Dokumen belum ada `[GAP]`

| # | Dokumen | Dijelaskan di | Prioritas |
|---|---------|---------------|-----------|
| B1 | `docs/analyst/business-rules.md` | §6 + matrix Gap baris 84 | **Tinggi** — katalog penuh 40 `BR-*` (Rule/When/Then/Else/Source + kasus uji), wajib **identik** dengan §6 |
| B2 | `docs/analyst/00-Global/PROCESS-FLOW-MASTER.md` | §5.2 catatan | Sedang — daftar PF-001…PF-007 |
| B3 | BAB 1 & BAB 2 (pendahuluan, analisis situasi) | keputusan user | Sedang — **ditunda**, jangan diulang dari §1–§3; peta kerja = `00-Global/HANDOFF-BAB-1-2.md` |
| B4 | `docs/analyst/spec.md` / roadmap implementasi | — | Rendah — di luar scope desain |

## C. Butuh keputusan user `[OPEN]`

| # | Isi | Dampak | Perlu dari |
|---|-----|--------|-----------|
| C1 | **OQ-001** — proporsi NON_MATCHING belum pernah diukur (ASM-002) | Target "selesai <20 menit" bisa tidak realistis | Sampel 1 bulan |
| C2 | **OQ-002** — koreksi dikirim ke ledger **langsung** atau **jadwal berikutnya**? Desain sekarang = `BR-WF-003` (baris baru siklus berikutnya) | Bisa mengubah FR-014 + PF-004 | System Analyst |
| C3 | **OQ-004** — restorasi saat run gagal setengah jalan. Desain sekarang = `BR-WF-007` RERUN idempoten, koreksi lama dipertahankan | Risiko data setengah proses | SH-001 |
| C4 | **SH-012** — belum ada peran QA/tester | Tidak ada pemilik UAT exit | Saat tim QA terbentuk |
| C5 | **NFR sign-off** — 4 penandatangan masih `{待定}` | NFR kritis belum sah | Manajer Keuangan |
| C6 | **OQ-003** — nama & tanda tangan pemilik dokumen | Sign-off buntu | Manajer Keuangan |

## D. Yang sudah beres (26 Sep 2026)

- [x] §5.1 flowchart berlabel aktor + §7.2 panel ringkasan **tergenerasi presisi** (lebar 85 / 52 kolom, tidak ada baris meleset).
- [x] §6 memuat **40 `BR-*` unik** = inventaris yang disepakati (VAL 11 · CALC 4 · AUTH 8 · WF 9 · CON 5 · RET 3).
- [x] `REQUIREMENTS-MATRIX.md`: kolom **Business Rules** terisi semua dengan 6 kategori; kolom **Use Case** menunjuk UC-01…UC-12 (tidak ada sisa `(ditunda)`).
- [x] `ERD-MASTER.md` contoh `rule_code` diganti `BR-VAL-003 / BR-VAL-005` (tidak ada `BR-MATCH-*` tersisa).
- [x] `NOTATION.md`: pengecualian BPMN dicatat resmi + alasan keputusan user.
- [x] **Nol Mermaid** di seluruh dokumen (hanya muncul sebagai kalimat larangan) — *diperbarui 27 Sep: lihat butir E14*.
- [x] Jumlah silang: 23 FR · 20 NFR · 12 UC · 7 PF · 9 CON · 6 ASM · 30 istilah *(kini 31 — E13)*.

## E. Yang sudah beres (27 Sep 2026)

- [x] **A1** — indeks header §5 kini `Flowchart berlabel aktor + DFD`.
- [x] **A2** — Coverage Summary FR: `23 | 19 | 4 | 0` (Confirmed 19 / Pending 4).
- [x] **A3–A7** — §5.3 DFD **digambar ulang** (M10 → D3; M9 turun P3→P4 & M12 naik; D3 punya masuk M10/M13 & keluar M14 → P5; M17 rapi ke A3; tabel **M1–M17 lengkap 17 baris**, penamaan E2 seragam).
- [x] **A8** — kotak `SERAH TERIMA [SISTEM] → [ANALIS]` ditambahkan di §5.1 sebelum `[6]`.
- [x] **A9** — `ERD-MASTER.md`: `→ BR-AUTH-001 + BR-AUTH-002`.
- [x] **A10** — opsi (b): aturan penomoran header = `O-` laporan + `R-` baris rincian (**reserved**); dicatat di `NOTATION.md` §"ATURAN PENOMORAN".
- [x] **A11** — §5.4 peta digambar ulang (lihat E14).
- [x] **A12** — inden kotak `[SISTEM] 10` disamakan (9 spasi).
- [x] **A13** — entri glossary **"BR-CON vs CON (istilah penamaan)"** ditambahkan (total istilah 30 → 31).
- [x] **B1** — `business-rules.md` ditulis: **40 aturan** (VAL 11 · CALC 4 · AUTH 8 · WF 9 · CON 5 · RET 3), 34 Confirmed / 6 Pending, tiap aturan punya Rule/When/Then/Else/Examples/Related, **divalidasi otomatis identik dengan §6**.
- [x] **B2** — `00-Global/PROCESS-FLOW-MASTER.md` ditulis: index PF-001…PF-007 + relationship map + aturan pemakaian.
- [x] **E14 · Keputusan user (Mermaid)** — atas permintaan user, **§5.3 (DFD) & §5.4 (peta)** digambar dengan **Mermaid** + **deskripsi diagram** di atasnya; **semua diagram lain tetap ASCII**. Pengecualian dicatat di `NOTATION.md`, header `DESAIN-PROGRAM.md`, dan `README.md`. Konsekuensi: checklist "Nol Mermaid" (D) kini berlaku **di luar §5.3–§5.4**.
- [x] **Sinkron** — referensi "belum dibuat" di §5.2/§6/Appendix A, `README.md`, `REQUIREMENTS-MATRIX.md` (coverage BR, gaps, daftar menyusul) diperbarui; Change Log `DESAIN-PROGRAM.md` v0.3.

## Urutan pengerjaan yang disarankan

1. ~~**A1, A2**~~ selesai · 2. ~~**A3–A7**~~ selesai · 3. ~~**A8–A10**~~ selesai ·
4. ~~**A11–A13**~~ selesai · 5. ~~**B1**~~ selesai · 6. ~~**B2**~~ selesai.
7. **B3** — BAB 1–2 mengikuti `00-Global/HANDOFF-BAB-1-2.md` (sesi terpisah).
8. **C1–C6** — dikumpulkan, dikirim ke user/stakeholder sekali jalan.
