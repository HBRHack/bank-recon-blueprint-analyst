# HANDOFF-BAB-1-2 — Serah Terima Penulisan BAB 1 & BAB 2

> **Status:** siap dikerjakan sesi terpisah · 27 September 2026
> **Pemilik:** System Analyst (SH-005)
> **Aturan main:** BAB 1–2 **ditulis setelah desain beres** (keputusan user) dan
> **jangan diulang dari `DESAIN-PROGRAM.md` §1–§3** — cukup rujuk.

## 1. Tujuan dokumen ini

Menyerahkan segala yang dibutuhkan penulis BAB 1 (Pendahuluan) dan BAB 2
(Analisis Situasi) supaya bisa langsung menulis **tanpa membaca ulang seluruh
repo** dan tanpa menduplikasi isi yang sudah divalidasi.

## 2. Sumber utama — baca ini saja

| BAB | Sumber utama | Yang diambil |
|-----|--------------|--------------|
| BAB 1 · Pendahuluan | `DESAIN-PROGRAM.md` §1 (Problem Statement) | Latar, rumusan masalah, angka nyata klien (18.000 baris · 4 bank · 06:30–10:00 · floating Rp 1,2–2 M) |
| BAB 1 · Pendahuluan | `DESAIN-PROGRAM.md` §2 (Scope & Batasan) | Ruang lingkup in/out, batasan keras, pembagian rilis |
| BAB 1 · Pendahuluan | `README.md` §2 (Tujuan proyek) | Ringkasan eksekutif "masalah → rancangan, bukan aplikasi" |
| BAB 2 · Analisis Situasi | `DESAIN-PROGRAM.md` §3 (Actor & Use Case) | Aktor, hak akses, UC-01…UC-12 |
| BAB 2 · Analisis Situasi | `question-framework.md` | A–F terjawab + 6 gate final (bahan "situasi saat ini") |
| BAB 2 · Analisis Situasi | `stakeholder-register.md` | SH-001…SH-011, peta influence/interest |
| BAB 2 · Analisis Situasi | `assumptions-constraints.md` | 6 ASM + 9 CON + risk register |
| Kedua BAB | `00-Global/GLOSSARY.md` | 31 istilah — pakai definisi persis, jangan istilah baru |

Rujukan pendukung (hanya bila perlu): `00-Global/SRS-MASTER.md` (FR-001…023),
`nfr.md` (20 NFR), `DESAIN-PROGRAM.md` §5–§6 (alur & aturan bisnis).

## 3. Struktur yang disarankan

**BAB 1 — Pendahuluan**

1. Latar belakang → §1.1 Masalah + angka klien
2. Rumusan masalah → daftar perbaikan di §1.2 (jangan menulis ulang narasi panjang)
3. Tujuan & manfaat → §1.3 + README §2
4. Ruang lingkup → §2 In-scope / Out-of-scope
5. Batasan → §2 batasan keras + CON-001…CON-009 (rujuk, jangan salin daftar)

**BAB 2 — Analisis Situasi**

1. Profil proses bisnis saat ini (manual 06:30–10:00, filter manual, selisih lolos)
2. Pemangku kepentingan & peran → §3 aktor + stakeholder-register
3. Analisis kebutuhan ringkas → ringkasan FR/NFR (tabel singkat + rujukan ID)
4. Kendala & asumsi → CON/ASM (rujuk `assumptions-constraints.md`)
5. Peta masalah → solusi: tiap masalah §1.1 dipetakan ke FR/UC terkait

## 4. JANGAN diduplikasi (rujuk, jangan tulis ulang)

- **§1–§3 `DESAIN-PROGRAM.md`** sudah berisi problem statement, scope, actor,
  dan use case — BAB 1–2 cukup merujuk, **tanpa menyalin ulang**.
- Jangan mengulang daftar **40 `BR-*`** (`business-rules.md`), **23 FR**
  (`SRS-MASTER.md`), **20 NFR** (`nfr.md`), **31 istilah** (glossary).
- Jangan menggambar ulang diagram — rujuk `DESAIN-PROGRAM.md` §5.1–§5.4.
- **ID baru dilarang.** Semua rujukan memakai kode yang sudah ada:
  `UC-` `PF-` `BR-` `FR-` `NFR-` `CON-` `ASM-` `SH-`. Tanpa ID = tidak bisa ditest
  QA (aturan penomoran `NOTATION.md`).

## 5. Angka kunci yang boleh dikutip apa adanya

18.000 baris/hari · 4 bank · ±16.000 transaksi ledger · 06:30–10:00 WIB →
target 20 menit · cut-off 06:00 / tutup 07:30 WIB · carry-over ≤0,5% (≤90 baris) ·
eskalasi >Rp 5 juta · retensi jejak 10 tahun · 97,03% cocok otomatis.

> Semua angka sudah divalidasi silang — **jangan diubah** tanpa update sumbernya.

## 6. Daftar keputusan tertunda (C1–C6) — jangan dianggap final

| # | Isi | Perlu dari |
|---|-----|-----------|
| C1 | OQ-001 — proporsi NON_MATCHING belum diukur | Sampel 1 bulan |
| C2 | OQ-002 — koreksi ke ledger langsung atau jadwal berikutnya | System Analyst |
| C3 | OQ-004 — restorasi run gagal setengah jalan (jawaban sementara: RERUN idempoten `BR-WF-007`) | SH-001 |
| C4 | SH-012 — belum ada peran QA/tester | Saat tim QA terbentuk |
| C5 | NFR sign-off — 4 penandatangan masih kosong | Manajer Keuangan |
| C6 | OQ-003 — nama & tanda tangan pemilik dokumen | Manajer Keuangan |

Di BAB 1–2, poin-poin ini ditandai **"menunggu konfirmasi"**, bukan sebagai
fakta final. Sumber lengkap: [`need-review.md`](../../need-review.md) §C.

## 7. Definition of done untuk BAB 1–2

- [ ] Tiap paragraf bernomor merujuk ID yang sudah ada (tidak ada kode baru).
- [ ] Semua angka identik dengan sumber di §5 daftar ini.
- [ ] Diagram (bila ada) **ASCII/Unicode box art** — Mermaid hanya diizinkan di
      `DESAIN-PROGRAM.md` §5.3–§5.4 (lihat `NOTATION.md`).
- [ ] Tanpa nama bahasa pemrograman / framework / basis data (CON-009) dan
      tanpa potongan kode program.
- [ ] Glossary dipakai apa adanya; istilah baru = tambah glossary dulu.
- [ ] `REQUIREMENTS-MATRIX.md` kolom `Chapter` diperbarui ke `BAB-1`/`BAB-2`.

## Change Log

| Versi | Tanggal | Perubahan |
|-------|---------|-----------|
| 1.0 | 27 Sep 2026 | Dokumen serah terima dibuat — menggantikan keputusan "tulis BAB 1–2 di sesi desain" |
