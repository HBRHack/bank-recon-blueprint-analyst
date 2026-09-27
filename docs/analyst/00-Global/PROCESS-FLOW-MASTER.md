# Process Flows — Master (Lintas Chapter)

> **Notasi proyek (GLOBAL-00):** lihat [`NOTATION.md`](NOTATION.md) —
> matriks keputusan notasi per jenis informasi. Diagram per proses mengikuti
> notasi yang dipilih di sana (**Flowchart berlabel aktor + DFD level 0**;
> BPMN ditinggalkan — keputusan user 26 Sep 2026, dicatat di `NOTATION.md`).
> **Dilarang Mermaid** — ASCII/Unicode box art saja.

Sistem ini punya **satu alur besar harian**: mutasi 4 bank diterima dan
divalidasi (PF-001) → dinormalisasi dan dicocokkan otomatis 5 tier (PF-002) →
sisa selisih ditangani analis (PF-003) → yang tak bisa dicocokkan naik jadi
koreksi berkontrol (PF-004) → sepanjang hari sistem memantau ambang dan
mengeskalasi (PF-005) → rekonsiliasi ditutup dengan carry-over ≤ 0,5% (PF-006)
→ seluruh perilaku di atas dikendalikan aturan berversi (PF-007).

> **Kedudukan dokumen:** proyek ini memakai struktur datar (`docs/analyst/*`),
> bab BAB 1–2 ditunda atas keputusan user. Karena itu **rumah lengkap** setiap
> PF adalah [`DESAIN-PROGRAM.md` §5.2](../DESAIN-PROGRAM.md) (pemicu, tujuan,
> aktor, alur, keputusan, pengecualian). File ini adalah **index +
> relationship map** — tidak menduplikasi isi §5.2 (mencegah dua versi kebenaran).

## Index Proses

| ID | Proses | Chapter | Trigger | UC Terkait | Detail |
|----|--------|---------|---------|------------|--------|
| PF-001 | Penerimaan & validasi mutasi bank | 00-Global (lintas modul) | Jadwal 05:00 WIB / unduhan portal bank | UC-01, UC-12 | [§5.2 PF-001](../DESAIN-PROGRAM.md#pf-001-penerimaan--validasi-mutasi-bank) |
| PF-002 | Normalisasi & pencocokan otomatis (T1 → T5) | 00-Global (lintas modul) | Batch `VALIDATED` + ledger harian dimuat | UC-02, UC-12 | [§5.2 PF-002](../DESAIN-PROGRAM.md#pf-002-normalisasi--pencocokan-otomatis-t1--t5) |
| PF-003 | Penanganan selisih (triage) oleh analis | 00-Global (lintas modul) | Run `AWAITING_REVIEW` + antrean selisih terbit | UC-03, UC-09 | [§5.2 PF-003](../DESAIN-PROGRAM.md#pf-003-penanganan-selisih-triage-oleh-analis) |
| PF-004 | Pengajuan & persetujuan koreksi (kendali ganda) | 00-Global (lintas modul) | Selisih gagal dicocokkan manual (PF-003 langkah 7) | UC-04, UC-05 | [§5.2 PF-004](../DESAIN-PROGRAM.md#pf-004-pengajuan--persetujuan-koreksi-kendali-ganda) |
| PF-005 | Peringatan bertingkat & eskalasi | 00-Global (lintas modul) | Pukul 06:30 WIB / `aging_days > 1` | UC-07 | [§5.2 PF-005](../DESAIN-PROGRAM.md#pf-005-peringatan-bertingkat--eskalasi) |
| PF-006 | Penutupan hari rekonsiliasi | 00-Global (lintas modul) | Selisih ditangani / SUPERVISOR buka ringkasan | UC-06 | [§5.2 PF-006](../DESAIN-PROGRAM.md#pf-006-penutupan-hari-rekonsiliasi) |
| PF-007 | Pengaturan aturan pencocokan (berversi) | 00-Global (lintas modul) | Kebutuhan mengubah parameter aturan | UC-08, UC-11 | [§5.2 PF-007](../DESAIN-PROGRAM.md#pf-007-pengaturan-aturan-pencocokan-berversi) |

- **Kolom ID:** nomor PF-XXX global sequential — sama dengan nomor di
  `DESAIN-PROGRAM.md` §5.2. Jangan nomor ulang.
- **Kolom Chapter:** seluruh proses menyentuh ≥ 2 modul peran (analyst,
  supervisor, admin) → semuanya `00-Global (lintas modul)`.
- Alur lintas modul **ditulis lengkap** di `DESAIN-PROGRAM.md` §5.2; file ini
  hanya meneruskan rujukan (pola master/excerpt: §5.2 = "excerpt yang hidup",
  file ini = index).

## Process Relationship Map

**Deskripsi diagram (dibaca gampang):** peta ini menjawab **"alur mana
memicu alur mana"** — satu gambar untuk melihat ketergantungan ketujuh alur
proses (PF-001…PF-007) sekaligus, tanpa harus membaca §5.2 satu per satu.
Ringkasnya: **PF-007** menyediakan aturan → **PF-001** menerima mutasi →
**PF-002** mencocokkan → kalau ada sisa selisih → **PF-003** memilah → kalau
tidak bisa manual → **PF-004** koreksi dengan persetujuan → hasilnya masuk
**REGISTER SELISIH** dan **BUKU BESAR / LEDGER HARIAN** (dua kotak tanpa
awalan `PF-` = penyimpanan data yang dipakai bersama) → **PF-005** memantau
peringatan → **PF-006** menutup hari 07:30 → besok ulang dari **PF-001**.
Label pada tiap panah = sebab-akibatnya; makna panah-panah khusus dijelaskan
di daftar tepat setelah gambar.

```
                        +---------------------------+
                        | PF-007 Aturan berversi    |
                        | (RuleVersion, effective_  |
                        |  from = run berikutnya)   |
                        +-------------+-------------+
                                      |
                                      | --controls--> (seluruh tier PF-002)
                                      v
+------------------+   triggers   +------------------+   triggers   +------------------+
| PF-001           | ------------>| PF-002           | ------------>| PF-003           |
| Terima & validasi|              | Normalisasi &    |              | Triage selisih   |
| batch 4 bank     |<-------------| cocokkan T1-T5   |              | oleh analis      |
+------------------+  PARTIAL +   +---------+--------+              +--------+---------+
                       re-run               |                                |
                                             | --sisa yang gagal--->          | --gagal manual-->
                                             v                                v
                                  +------------------+   escalates-to  +------------------+
                                  | REGISTER SELISIH |<----------------| PF-004           |
                                  | (UNMATCHED /     |                 | Pengajuan &      |
                                  |  NON_MATCHING)   |                 | persetujuan      |
                                  +--------+---------+                 | koreksi          |
                                           |                           +--------+---------+
                                           |                                     |
                          monitors (06:30) |                                     | --POSTED (siklus
                                           v                                     |    berikutnya)-->
                                  +------------------+                            v
                        +---------| PF-005           |                    BUKU BESAR /
                        |         | Peringatan &     |                    LEDGER HARIAN
                        |         | eskalasi         |                           ^
                        |         +---------+--------+                           |
                        |                   | --carry-over > 0,5%--> ESCALASI   |
                        +-------------------|------------------------------------+
                                            v
                                  +------------------+
                                  | PF-006           |
                                  | Tutup hari 07:30 |
                                  | run = CLOSED     |
                                  +--------+---------+
                                           |
                                           | --siklus berikutnya-->
                                           v
                                     PF-001 (besok)
```

- Panah `--controls-->` PF-007 → PF-002 = versi aturan dibekukan per run
  (`BR-WF-005`, `BR-RET-002`).
- Sisi `PARTIAL` PF-001 → PF-002 = bank telat diproses parsial
  (`BR-WF-008`); run ulang idempoten (`BR-WF-007`).
- `PF-003 --gagal manual--> PF-004` = langkah 7 triage, syarat
  `BR-WF-002` (alasan wajib).
- `PF-005` memantau PF-003 (aging) **dan** PF-006 (gerbang tutup hari):
  ambang WASPADA > 90 / KRITIS > 200, respons ≤ 15 menit (`BR-WF-004`).

## Aturan pemakaian index ini

1. Setiap `PF-XXX` yang ditambahkan di `DESAIN-PROGRAM.md` §5.2 **wajib**
   muncul di Index Proses pada hari yang sama — index tanpa rujukan hidup =
   warning di review.
2. Bila satu PF dihapus/diganti nomor, perbarui baris ini **dan** seluruh
   rujukan `PF-` di `business-rules.md`, `REQUIREMENTS-MATRIX.md`, serta
   `DESAIN-PROGRAM.md` §6 (kolom pemakaian PF).
3. Notasi diagram mengikuti `NOTATION.md` (GLOBAL-00); konsistensi antar
   dokumen di-audit pada Check 9 (Notation Consistency Gate).
4. **Dilarang Mermaid JS** — ASCII/Unicode box art saja; satu baris Mermaid =
   kegagalan kritis dokumen.

## Change Log

| Versi | Tanggal | Perubahan |
|-------|---------|-----------|
| 1.0 | 27 Sep 2026 | Index awal PF-001…PF-007 + relationship map, selaras dengan `DESAIN-PROGRAM.md` §5.2. |
