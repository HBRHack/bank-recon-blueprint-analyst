# ERD: Bank Reconciliation & Statement Mapping System

> **Notasi proyek (GLOBAL-00):** lihat [`NOTATION.md`](NOTATION.md) — matriks
> keputusan notasi. ERD struktur data memakai notasi entitas-relasi (baris 6),
> **ASCII box-art saja, dilarang Mermaid.**

Model data untuk pencocokan harian antara mutasi bank partner (18.000 baris/hari,
4 bank) dengan transaksi ledger internal (±16.000 transaksi/hari), termasuk
penanganan selisih, koreksi berotorisasi ganda, dan jejak audit yang tidak bisa
diubah. Struktur **logis** (entitas/field/relasi) — bukan skema database fisik.

## Entity Relationship Diagram

```
+---------------------+         +---------------------+         +---------------------+
| StatementSource     |         | StatementBatch      |         | BankStatementLine   |
+---------------------+         +---------------------+         +---------------------+
| PK | source_id      |         | PK | batch_id       |         | PK | line_id        |
|    | bank_code      |--1:N--- | FK | source_id      |--1:N--- | FK | batch_id       |
|    | bank_name      |         | FK | run_id         |         | FK | run_id         |
|    | channel_type   |         |    | batch_date     |         |    | source_line_no |
|    | account_ref    |         |    | received_at    |         |    | value_date     |
|    | stmt_format    |         |    | file_name      |         |    | booking_date   |
|    | arrive_expect  |         |    | record_count   |         |    | reference_no   |
|    | cutoff_time    |         |    | total_debit    |         |    | description    |
|    | retry_policy   |         |    | total_credit   |         |    | dr_cr_flag     |
|    | active_flag    |         |    | checksum_value |         |    | currency       |
|    | created_at     |         |    | status         |         |    | amount         |
|    | updated_at     |         |    | created_at     |         |    | is_reversal    |
+---------------------+         +---------------------+         +---------------------+
                                          ^
                                          | 1:N
                                          |
+---------------------+         +---------------------+         +---------------------+
| SystemUser          |         | ReconciliationRun   |         | InternalTransaction |
+---------------------+         +---------------------+         +---------------------+
| PK | user_id        |         | PK | run_id         |         | PK | txn_id         |
|    | full_name      |--1:N--- | FK | triggered_by   |--1:N--- | FK | run_id         |
|    | email          |         | FK | ruleset_id     |         |    | external_ref   |
|    | role           |         |    | run_date       |         |    | txn_type       |
|    | assigned_scope |         |    | cutoff_time    |         |    | counterparty   |
|    | active_flag    |         |    | started_at     |         |    | merchant_id    |
|    | last_login_at  |         |    | finished_at    |         |    | account_ref    |
|    | fail_count     |         |    | trigger_type   |         |    | value_date     |
|    | two_factor_flag|         |    | status         |         |    | booking_date   |
|    | locale         |         |    | pending_banks  |         |    | amount         |
|    | created_at     |         |    | carry_over_flag|         |    | currency       |
|    | updated_at     |         |    | total_unmatched|         |    | status         |
+---------------------+         +---------------------+         +---------------------+
                                          |
                                          | 1:N
                                          v
+---------------------+         +---------------------+         +---------------------+
| MatchRule           |         | MatchResult         |         | MatchResultLeg      |
+---------------------+         +---------------------+         +---------------------+
| PK | rule_id        |         | PK | match_id        |         | PK | leg_id        |
|    | rule_code      |--1:N--- | FK | rule_id        |--1:N--- | FK | match_id      |
|    | rule_name      |         | FK | run_id         |         | FK | bank_line_id  |
|    | match_tier     |         | FK | matched_by     |         | FK | ledger_txn_id |
|    | field_criteria |         |    | match_status   |         |    | side          |
|    | amt_tolerance  |         |    | match_type     |         |    | leg_role      |
|    | tol_days       |         |    | confidence     |         |    | leg_amount    |
|    | normalize_rule |         |    | matched_amount |         |    | leg_sign      |
|    | priority       |         |    | matched_at     |         |    | sequence_no   |
|    | active_flag    |         |    | leg_count      |         |    | is_primary    |
|    | effective_from |         |    | reversal_flag  |         |    | created_at    |
|    | effective_to   |         |    | notes          |         |    | notes         |
+---------------------+         +---------------------+         +---------------------+

+---------------------+         +---------------------+
| UnmatchedItem       |         | AdjustmentRequest   |
+---------------------+         +---------------------+
| PK | item_id        |         | PK | adj_id         |
| FK | run_id         |--1:N--- | FK | item_id        |
| FK | bank_line_id   |         | FK | requested_by   |
| FK | ledger_txn_id  |         | FK | approved_by    |
|    | side           |         |    | adj_type       |
|    | reason_code    |         |    | proposed_amt   |
|    | category       |         |    | reason_text    |
|    | amount         |         |    | status         |
|    | first_seen     |         |    | requested_at   |
|    | aging_days     |         |    | decided_at     |
|    | resolution     |         | FK | posted_txn_id  |
|    | assigned_to    |         |    | posted_at      |
+---------------------+         +---------------------+

+---------------------+         +---------------------+
| SystemUser (ulang)  |         | AuditEvent          |
+---------------------+         +---------------------+
| PK | user_id        |         | PK | event_id       |
|    | full_name      |--1:N--- | FK | actor_id       |
|    | role           |         |    | event_time     |
|    | assigned_scope |         |    | event_type     |
|    | ...            |         |    | subject_type   |
+---------------------+         |    | subject_id     |
                                |    | action_desc    |
                                |    | old_value      |
                                |    | new_value      |
                                |    | acting_as_role |
                                |    | source_channel |
                                |    | hash_seal      |
                                +---------------------+

Legend:
  PK = Primary Key   FK = Foreign Key   1:N = One-to-Many
  Garis horizontal   = relasi yang digambar di diagram ini
  Kolom FK di kotak  = relasi yang lintas band (relasi lengkap ada di
                       Relationship Summary — tabel di bawah adalah sumber
                       kebenarannya)
  MatchResultLeg     = bridge entitas yang mengubah relasi M:N antara
                       BankStatementLine dan InternalTransaction
                       (1 baris bank ↔ banyak transaksi ledger = agregasi;
                        1 transaksi ↔ banyak baris bank = split/fee)
```

## Entity Details

### StatementSource

Koneksi ke satu rekening bank partner (1 baris per rekening per bank).

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| source_id | ID | Yes | auto | Kunci utama sumber mutasi |
| bank_code | Teks | Yes | — | Kode bank pendek (mis. `BANKA`) |
| bank_name | Teks | Yes | — | Nama bank untuk laporan |
| channel_type | Enum | Yes | `MANUAL` | Kanal masuk: `SFTP` (3 bank otomatis) / `MANUAL` (portal unduh) |
| account_ref | Teks | Yes | — | Nomor rekening partner (diringkas/di-masking di tampilan) |
| stmt_format | Enum | Yes | — | Profil struktur kolom bank: `FMT_A` / `FMT_B` / `FMT_C` (tanpa referensi) |
| arrive_expect | Waktu | Yes | `05:00` | Jam wajib file tiba — lewat ini sistem tandai telat |
| cutoff_time | Waktu | Yes | `06:00` | Batas statement final harian |
| retry_policy | Enum | Yes | `3x` | Percobaan ulang ambil file: `1x` / `3x` / `MANUAL_ONLY` |
| active_flag | Ya/Tidak | Yes | `true` | Sumber masih aktif dipantau |
| created_at | Waktu | Yes | otomatis | Waktu rekam dibuat |
| updated_at | Waktu | Yes | otomatis | Waktu rekam terakhir diubah |

**Constraints:** `bank_code + account_ref` unik · `arrive_expect < cutoff_time`.
**Indexes:** `bank_code` (tampilan per bank) · `arrive_expect` (pemeriksaan keterlambatan).

### StatementBatch

Satu berkas/file mutasi yang diterima dari satu sumber pada satu tanggal.

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| batch_id | ID | Yes | auto | Kunci utama batch |
| source_id | ID | Yes | — | FK → StatementSource |
| run_id | ID | No | — | FK → ReconciliationRun (kosong bila belum masuk proses) |
| batch_date | Tanggal | Yes | — | Tanggal efektif mutasi |
| received_at | Waktu | Yes | otomatis | Waktu file diterima (dipakai cek keterlambatan) |
| file_name | Teks | Yes | — | Nama berkas masuk |
| record_count | Angka(hitung) | Yes | — | Jumlah baris yang dibaca dari file |
| total_debit | Angka(desimal) | Yes | — | Total debit — dipakai coklat control total vs isi |
| total_credit | Angka(desimal) | Yes | — | Total kredit — dipakai coklat control total |
| checksum_value | Teks | Yes | — | Nilai pembanding isi berkas (deteksi file rusak/ganda) |
| status | Enum | Yes | `RECEIVED` | `RECEIVED` / `VALIDATED` / `REJECTED` / `PENDING` (bank belum kirim) / `PROCESSED` |
| created_at | Waktu | Yes | otomatis | Waktu rekam dibuat |

**Constraints:** `source_id + batch_date + file_name` unik (anti file masuk dobel) ·
`record_count ≥ 0` · `status = REJECTED` wajib punya alasan di AuditEvent.
**Indexes:** `batch_date` (cari hari itu) · `status` (antrean batch pending).

### BankStatementLine

Satu baris mutasi bank — **mentah, tidak pernah diubah atau dihapus**.

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| line_id | ID | Yes | auto | Kunci utama baris |
| batch_id | ID | Yes | — | FK → StatementBatch |
| run_id | ID | No | — | FK → ReconciliationRun yang memprosesnya |
| source_line_no | Angka(hitung) | Yes | — | Nomor urut baris dalam file (jejak telusur) |
| value_date | Tanggal | Yes | — | Tanggal nilai — bahan pencocokan (boleh H+1) |
| booking_date | Tanggal | Yes | — | Tanggal booking di sisi bank |
| reference_no | Teks | No | — | Nomor referensi bank — **kunci pencocokan utama; boleh kosong** (transaksi QRIS) |
| description | Teks | No | — | Keterangan bebas bank — bahan pencocokan fuzzy |
| dr_cr_flag | Enum | Yes | — | Arah dana: `DR` / `CR` |
| currency | Teks | Yes | `IDR` | Mata uang — validasi: wajib `IDR` untuk rekening lokal |
| amount | Angka(desimal) | Yes | — | Nominal mutasi (selalu positif, arah ditandai `dr_cr_flag`) |
| is_reversal | Ya/Tidak | Yes | `false` | Baris ini penarikan/pembatalan — wajib ketemu pasangan reversal |

**Constraints:** `amount > 0` · `currency = IDR` · `value_date ≤ batch_date` ·
`reference_no` **unik per `batch_date` per rekening** (kembar → otomatis masuk
exception `DUPLICATE_REF`, tidak dipaksa dicocokkan) · `source_line_no` unik per `batch_id`.
**Indexes:** `run_id + dr_cr_flag + amount` (kandidat pencocokan) ·
`reference_no` (lookup exact match) · `value_date` (rentang tanggal).

### InternalTransaction

Satu transaksi dari sistem payment internal (ledger sisi perusahaan).

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| txn_id | ID | Yes | auto | Kunci utama transaksi ledger |
| run_id | ID | Yes | — | FK → ReconciliationRun (batch ledger hari itu) |
| external_ref | Teks | No | — | Referensi sistem — pencocokan ke `reference_no` bank |
| txn_type | Enum | Yes | — | `SETTLEMENT` / `FEE` / `REFUND` / `ADJUSTMENT` / `REVERSAL` |
| counterparty | Teks | No | — | Pihak lawan transaksi (merchant/bank lain) |
| merchant_id | Teks | No | — | Kode merchant — bahan pencocokan fallback QRIS |
| account_ref | Teks | Yes | — | Rekening internal terkait |
| value_date | Tanggal | Yes | — | Tanggal efektif — bahan pencocokan |
| booking_date | Tanggal | Yes | — | Tanggal proses di ledger |
| amount | Angka(desimal) | Yes | — | Nominal transaksi |
| currency | Teks | Yes | `IDR` | Mata uang |
| status | Enum | Yes | — | `POSTED` / `PENDING` / `REVERSED` — hanya `POSTED` yang ikut dicocokkan |

**Constraints:** `amount > 0` · `status ∈ {POSTED, REVERSED}` untuk ikut proses ·
`txn_id` unik (idempoten: muat ulang ledger hari yang sama tidak boleh menggandakan).
**Indexes:** `run_id + amount + value_date` (kandidat pencocokan) · `external_ref` ·
`merchant_id + amount` (fallback QRIS).

### ReconciliationRun

Satu eksekusi rekonsiliasi harian (satu per tanggal cut-off).

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| run_id | ID | Yes | auto | Kunci utama run |
| triggered_by | ID | No | — | FK → SystemUser; **kosong = jadwal otomatis 05:00** |
| ruleset_id | ID | Yes | — | FK → MatchRule (versi aturan yang dipakai run ini) |
| run_date | Tanggal | Yes | — | Tanggal efektif rekonsiliasi (unik) |
| cutoff_time | Waktu | Yes | `06:00` | Batas statement yang dimasukkan |
| started_at | Waktu | Yes | otomatis | Mulai proses otomatis (target 06:00) |
| finished_at | Waktu | No | — | Selesai proses otomatis (target ≤ 06:30) |
| trigger_type | Enum | Yes | `SCHEDULED` | `SCHEDULED` / `MANUAL` / `RERUN` |
| status | Enum | Yes | `QUEUED` | `QUEUED` / `RUNNING` / `AWAITING_REVIEW` / `CLOSED` / `FAILED` |
| pending_banks | Angka(hitung) | Yes | `0` | Bank yang belum kirim → diproses parsial |
| carry_over_flag | Ya/Tidak | Yes | `false` | Masih ada sisa bawa ke besok (maks 0,5% / 90 baris) |
| total_unmatched | Angka(hitung) | Yes | `0` | Jumlah baris belum ketemu pasangan |

**Constraints:** `run_date` unik (1 hari = 1 run; diulang = `RERUN` ke run yang sama) ·
`finished_at ≥ started_at` · `carry_over_flag = true` hanya bila
`total_unmatched ≤ 0,5% dari total baris` (90 dari 18.000).
**Indexes:** `run_date` (unik, pencarian cepat) · `status` (antrean review).

### MatchRule

Konfigurasi satu aturan pencocokan — **berjenjang T1→T5, berlaku per versi.**

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| rule_id | ID | Yes | auto | Kunci utama aturan |
| rule_code | Teks | Yes | — | Kode aturan yang menentukan hasil (mis. `BR-VAL-003`, `BR-VAL-005` — 6 kategori `VAL·CALC·AUTH·WF·CON·RET`) |
| rule_name | Teks | Yes | — | Nama aturan untuk laporan |
| match_tier | Angka(hitung) | Yes | — | Urutan evaluasi: 1 = exact referensi … 5 = fuzzy |
| field_criteria | Teks | Yes | — | Daftar kolom yang harus cocok (mis. `reference_no, dr_cr_flag`) |
| amt_tolerance | Angka(desimal) | Yes | `1` | Toleransi selisih nominal (Rp 1 = pembulatan) |
| tol_days | Angka(hitung) | Yes | `1` | Toleransi selisih tanggal (hari) |
| normalize_rule | Teks | Yes | — | Profil normalisasi sebelum dibanding (mis. buang prefix `TRF-`) |
| priority | Angka(hitung) | Yes | — | Urutan dalam tier yang sama |
| active_flag | Ya/Tidak | Yes | `true` | Aturan ikut dievaluasi |
| effective_from | Tanggal | Yes | — | Mulai berlaku (menjaga hasil run lampau tetap bisa dijelaskan) |
| effective_to | Tanggal | No | — | Berakhir — kosong = masih berlaku |

**Constraints:** `match_tier ∈ 1..5` · `amt_tolerance ≥ 0` ·
`rule_code` unik per `effective_from` · aturan nonaktif **tidak dihapus** (retensi jejak 10 tahun).
**Indexes:** `match_tier + priority` (urutan evaluasi) · `active_flag`.

### MatchResult

Satu hasil pencocokan: header yang menampung 1 atau banyak "leg".

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| match_id | ID | Yes | auto | Kunci utama hasil |
| rule_id | ID | No | — | FK → MatchRule; **kosong bila match manual** |
| run_id | ID | Yes | — | FK → ReconciliationRun |
| matched_by | ID | No | — | FK → SystemUser; kosong bila otomatis (SI) |
| match_status | Enum | Yes | `MATCHED` | `MATCHED` / `UNMATCHED` / `RESOLVED` / `VOID` |
| match_type | Enum | Yes | `AUTO` | `AUTO` (sistem) / `MANUAL` (analis) / `AGGREGATED` (kelompok) / `SPLIT` |
| confidence | Angka(desimal) | Yes | `1.00` | Skor 0,00–1,00 — tier 1 = 1,00, makin rendah makin wajib direview |
| matched_amount | Angka(desimal) | Yes | — | Total nominal sisi bank yang terpasangkan |
| matched_at | Waktu | Yes | otomatis | Waktu pencocokan terjadi |
| leg_count | Angka(hitung) | Yes | — | Jumlah leg — >1 berarti agregasi/split |
| reversal_flag | Ya/Tidak | Pasangan reversal | `false` | Pasangan ini penanganan pembatalan |
| notes | Teks | No | — | Catatan analis (wajib diisi bila `match_type = MANUAL`) |

**Constraints:** satu `BankStatementLine` boleh muncul di **maksimal 1**
`MatchResult` sepanjang waktu (anti dobel — ditegakkan lewat MatchResultLeg) ·
`confidence` antara 0,00–1,00 · `match_type = MANUAL` wajib `matched_by` dan `notes`.
**Indexes:** `run_id + match_status` (laporan harian) · `confidence`
(antrean review: skor rendah didahulukan).

### MatchResultLeg

Bridge M:N — **entitas inti yang membuat agregasi & split bisa terjadi.**

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| leg_id | ID | Yes | auto | Kunci utama leg |
| match_id | ID | Yes | — | FK → MatchResult (header) |
| bank_line_id | ID | No* | — | FK → BankStatementLine — isi hanya bila `side = BANK` |
| ledger_txn_id | ID | No* | — | FK → InternalTransaction — isi hanya bila `side = LEDGER` |
| side | Enum | Yes | — | Sisi leg: `BANK` / `LEDGER` |
| leg_role | Enum | Yes | `PRIMARY` | Peran dalam kelompok: `PRIMARY` / `FEE` / `GROUP` / `SPLIT` |
| leg_amount | Angka(desimal) | Yes | — | Kontribusi nominal leg ini ke `matched_amount` |
| leg_sign | Enum | Yes | — | `DR` / `CR` — arah leg (harus selaras antar sisi) |
| sequence_no | Angka(hitung) | Yes | `1` | Urutan leg dalam kelompok (1 = induk) |
| is_primary | Ya/Tidak | Yes | `true` | Leg utama — penanda wajib ada per sisi |
| created_at | Waktu | Yes | otomatis | Waktu leg dibuat |
| notes | Teks | No | — | Alasan leg dimasukkan (khusus `AGGREGATED`) |

**Constraints:** (`bank_line_id` XOR `ledger_txn_id`) terisi — satu baris hanya
boleh menunjuk satu sisi · **`bank_line_id` unik di seluruh tabel** dan
**`ledger_txn_id` unik di seluruh tabel** (anti dobel: satu baris/transaksi tidak
pernah berpasangan dua kali) · setiap `match_id` wajib punya minimal 1 leg `side=BANK`
dan 1 leg `side=LEDGER` · Σ `leg_amount` sisi BANK = Σ `leg_amount` sisi LEDGER
(dalam batas `amt_tolerance` aturan yang dipakai).
**Indexes:** `match_id` (leg per hasil) · `bank_line_id` (unik) · `ledger_txn_id` (unik).

### UnmatchedItem

Satu baris yang tidak ketemu pasangan — **termasuk yang sah tidak perlu dicocokkan.**

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| item_id | ID | Yes | auto | Kunci utama selisih |
| run_id | ID | Yes | — | FK → ReconciliationRun |
| bank_line_id | ID | No* | — | FK → BankStatementLine (isi bila `side = BANK`) |
| ledger_txn_id | ID | No* | — | FK → InternalTransaction (isi bila `side = LEDGER`) |
| side | Enum | Yes | — | Sisi yang bermasalah: `BANK` / `LEDGER` |
| reason_code | Enum | Yes | — | Alasan selisih — **8 kode** (lihat Constraints) |
| category | Enum | Yes | `UNMATCHED` | `UNMATCHED` = selisih nyata · `NON_MATCHING` = sah tidak berpasangan (biaya admin, saldo, transfer internal) — **tetap tampil di laporan, tidak boleh disembunyikan** |
| amount | Angka(desimal) | Yes | — | Nominal baris yang bermasalah |
| first_seen | Tanggal | Yes | — | Hari pertama terdeteksi (dasar perhitungan aging) |
| aging_days | Angka(hitung) | Yes | `0` | Selisih hari dari `first_seen` |
| resolution | Enum | Yes | `OPEN` | `OPEN` / `IN_PROGRESS` / `MATCHED_MANUAL` / `ADJUSTED` / `WRITTEN_OFF` |
| assigned_to | ID | No | — | FK → SystemUser (analis penanggung jawab) |

**Constraints:** (`bank_line_id` XOR `ledger_txn_id`) terisi ·
`reason_code ∈ { MISSING_REF, AMOUNT_DIFF, DATE_DIFF, DUPLICATE_REF, ORPHAN_BANK,
ORPHAN_LEDGER, AGG_UNRESOLVED, REVERSAL_MISMATCH }` —
`REVERSAL_MISMATCH` khusus reversal yang gagal ketemu pasangan reversalnya
(risiko berbeda dari `ORPHAN_*`: menandai pembatalan tidak sinkron, bukan noise data) ·
`category = NON_MATCHING` **wajib** punya `reason_code` turunan (`ORPHAN_*`) dan
tetap dihitung terpisah di laporan (volume + alasan terlihat oleh Auditor) ·
`resolution ≠ OPEN` wajib punya jejak AuditEvent.
**Indexes:** `run_id + category + reason_code` (ringkasan harian) ·
`aging_days` (peringatan utang menumpuk > 1 hari) · `assigned_to + resolution` (beban analis).

### AdjustmentRequest

Koreksi atas selisih — disiapkan sistem, **diposting hanya setelah approve (G-04).**

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| adj_id | ID | Yes | auto | Kunci utama permintaan koreksi |
| item_id | ID | Yes | — | FK → UnmatchedItem yang dikoreksi |
| requested_by | ID | Yes | — | FK → SystemUser (analis — pihak pertama dual-control) |
| approved_by | ID | No | — | FK → SystemUser (Supervisor — **wajib berbeda dari `requested_by`**) |
| adj_type | Enum | Yes | — | Jenis koreksi: `AMOUNT_FIX` / `DATE_FIX` / `MISSING_ENTRY` / `WRITE_OFF` |
| proposed_amt | Angka(desimal) | Yes | — | Nilai koreksi yang diajukan |
| reason_text | Teks | Yes | — | Alasan koreksi — wajib diisi |
| status | Enum | Yes | `DRAFT` | `DRAFT` / `SUBMITTED` / `APPROVED` / `REJECTED` / `POSTED` |
| requested_at | Waktu | Yes | otomatis | Waktu pengajuan |
| decided_at | Waktu | No | — | Waktu keputusan (kosong selama `SUBMITTED`) |
| posted_txn_id | ID | No | — | FK → InternalTransaction hasil koreksi — **baris baru besok, bukan edit baris lama** |
| posted_at | Waktu | No | — | Waktu koreksi tercatat di ledger |

**Constraints:** `approved_by ≠ requested_by` (dual-control) ·
`status = POSTED` wajib `posted_txn_id` terisi ·
`proposed_amt > Rp 5.000.000` wajib catat persetujuan Manajer Keuangan di AuditEvent ·
`status ∈ {APPROVED, REJECTED}` wajib `decided_at` ·
`item_id` hanya boleh punya 1 permintaan berstatus `SUBMITTED` dalam satu waktu.
**Indexes:** `item_id` · `status` (antrean persetujuan) · `requested_by + status`.

### SystemUser

Pengguna sistem beserta peran — dipakai untuk otorisasi dan jejak audit.

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| user_id | ID | Yes | auto | Kunci utama pengguna |
| full_name | Teks | Yes | — | Nama lengkap |
| email | Teks | Yes | — | Surel — unik |
| role | Enum | Yes | — | `ANALYST` / `SUPERVISOR` / `AUDITOR` / `SYSTEM_ANALYST` |
| assigned_scope | Teks | No | — | Cakupan akses rekening (mis. operasional / settlement) |
| active_flag | Ya/Tidak | Yes | `true` | Akun masih berlaku |
| last_login_at | Waktu | No | — | Login terakhir |
| fail_count | Angka(hitung) | Yes | `0` | Gagal login beruntun — **5 → akun terkunci** |
| two_factor_flag | Ya/Tidak | Yes | `true` | Dua langkah aktif (wajib) |
| locale | Teks | Yes | `id-ID` | Bahasa tampilan |
| created_at | Waktu | Yes | otomatis | Waktu akun dibuat |
| updated_at | Waktu | Yes | otomatis | Waktu perubahan terakhir |

**Constraints:** `email` unik · `role` salah satu dari 4 nilai ·
satu orang boleh memegang >1 peran → rekam sebagai **satu akun dengan `acting_as_role`
di AuditEvent**, bukan akun ganda.
**Indexes:** `role` · `active_flag`.

**Catatan relasi — AdjustmentRequest → InternalTransaction:**
`posted_txn_id` menghubungkan koreksi ke transaksi ledger yang **dibuat baru**.
Baris hasil koreksi ikut masuk run berikutnya sebagai ledger sisi — sehingga
koreksi tidak pernah menghitung dua kali pada hari yang sama.

### AuditEvent

Catatan perubahan — **append-only, tidak bisa diubah atau dihapus siapa pun.**

| Attribute | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| event_id | ID | Yes | auto | Kunci utama peristiwa |
| actor_id | ID | Yes | — | FK → SystemUser (pelaku; `SYSTEM` untuk proses otomatis) |
| event_time | Waktu | Yes | otomatis | Waktu peristiwa (urutan kronologis) |
| event_type | Teks | Yes | — | Jenis: `MATCH_MANUAL` / `RULE_CHANGE` / `ADJ_APPROVE` / `RUN_CLOSE` / `LOGIN` / dst. |
| subject_type | Teks | Yes | — | Entitas yang disentuh (mis. `AdjustmentRequest`) |
| subject_id | ID | Yes | — | Kunci entitas yang disentuh |
| action_desc | Teks | Yes | — | Deskripsi tindakan dalam bahasa manusia |
| old_value | Teks | No | — | Nilai sebelum diubah |
| new_value | Teks | No | — | Nilai sesudah diubah |
| acting_as_role | Teks | No | — | Peran yang sedang dijalankan (saat rangkap peran) |
| source_channel | Teks | Yes | `WEB` | Dari mana aksi dilakukan |
| hash_seal | Teks | Yes | otomatis | Segel perhitungan dari baris sebelumnya — memutus rantai bila ada yang diubah |

**Constraints:** baris **tidak boleh di-update atau di-delete** (hanya tambah) ·
`subject_type + subject_id + event_time` unik (anti ganda saat proses diulang) ·
`actor_id` wajib ada.
**Indexes:** `event_time` (urutan) · `subject_type + subject_id` (jejak satu entitas) ·
`actor_id + event_time` (audit per orang).

## Relationship Summary

| # | From | Cardinality | To | FK Location | Description |
|---|------|-------------|-----|-------------|-------------|
| 1 | StatementSource | 1:N | StatementBatch | `StatementBatch.source_id` | Satu rekening menghasilkan banyak batch harian |
| 2 | StatementBatch | 1:N | BankStatementLine | `BankStatementLine.batch_id` | Satu file berisi banyak baris mutasi |
| 3 | ReconciliationRun | 1:N | StatementBatch | `StatementBatch.run_id` | Satu run memproses banyak batch (parsial bila ada bank pending) |
| 4 | SystemUser | 1:N | ReconciliationRun | `ReconciliationRun.triggered_by` | Run manual oleh user; null = jadwal otomatis |
| 5 | MatchRule | 1:N | ReconciliationRun | `ReconciliationRun.ruleset_id` | Satu versi aturan dipakai banyak run |
| 6 | ReconciliationRun | 1:N | InternalTransaction | `InternalTransaction.run_id` | Ledger hari itu dimuat ke run |
| 7 | ReconciliationRun | 1:N | MatchResult | `MatchResult.run_id` | Run menghasilkan banyak pasangan |
| 8 | MatchRule | 1:N | MatchResult | `MatchResult.rule_id` | Aturan yang menghasilkan pasangan (null = manual) |
| 9 | SystemUser | 1:N | MatchResult | `MatchResult.matched_by` | Pencocokan manual oleh analis (null = otomatis) |
| 10 | MatchResult | 1:N | MatchResultLeg | `MatchResultLeg.match_id` | Header → leg (1..n) |
| 11 | BankStatementLine | 1:1 (maks) | MatchResultLeg | `MatchResultLeg.bank_line_id` | **Anti dobel:** 1 baris bank hanya boleh berpasangan sekali |
| 12 | InternalTransaction | 1:1 (maks) | MatchResultLeg | `MatchResultLeg.ledger_txn_id` | **Anti dobel:** 1 transaksi hanya boleh berpasangan sekali |
| 13 | **BankStatementLine** | **M:N** | **InternalTransaction** | **via MatchResult + MatchResultLeg** | **Relasi inti: agregasi (N ledger → 1 bank) dan split (1 bank → N ledger)** |
| 14 | ReconciliationRun | 1:N | UnmatchedItem | `UnmatchedItem.run_id` | Selisih per hari |
| 15 | BankStatementLine | 1:0..1 | UnmatchedItem | `UnmatchedItem.bank_line_id` | Selisih di sisi bank (XOR dengan #16) |
| 16 | InternalTransaction | 1:0..1 | UnmatchedItem | `UnmatchedItem.ledger_txn_id` | Selisih di sisi ledger (XOR dengan #15) |
| 17 | UnmatchedItem | 1:N | AdjustmentRequest | `AdjustmentRequest.item_id` | Satu selisih bisa diajukan koreksi berulang |
| 18 | SystemUser | 1:N | AdjustmentRequest | `AdjustmentRequest.requested_by` | Pengaju koreksi |
| 19 | SystemUser | 1:N | AdjustmentRequest | `AdjustmentRequest.approved_by` | Pemberi persetujuan — **wajib ≠ pengaju (4-eyes)** |
| 20 | InternalTransaction | 1:0..1 | AdjustmentRequest | `AdjustmentRequest.posted_txn_id` | Hasil koreksi = transaksi ledger baru besok |
| 21 | SystemUser | 1:N | AuditEvent | `AuditEvent.actor_id` | Jejak tindakan per orang (termasuk saat rangkap peran) |

**Relasi lintas band (tidak digambar sebagai garis):** #3, #4, #6, #7, #8, #9,
#15, #16, #18, #19, #20, #21 — semua terwakili kolom FK di kotak masing-masing
dan tercatat lengkap di tabel di atas (sumber kebenarannya).

## Catatan Desain yang Menyusul ke Dokumen Lain

- 8 `reason_code` + kategori `NON_MATCHING` (tetap tampil di laporan) →
  dipakai di `DESAIN-PROGRAM.md` §6 Aturan Bisnis dan §7 Output.
- Anti-dobel (#11, #12) = penerapan BR anti-pasangan ganda.
- Dual-control `requested_by ≠ approved_by` + eskalasi >Rp 5 juta →
  `BR-AUTH-001` + `BR-AUTH-002` (lihat `DESAIN-PROGRAM.md` §6).
- Segel `hash_seal` pada AuditEvent → NFR audit trail immutable, retensi 10 tahun.
