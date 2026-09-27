# Glossary: Bank Reconciliation & Statement Mapping System

Istilah bisnis domain rekonsiliasi bank fintech — memastikan System Analyst,
Developer, QA, Finance, dan Auditor memakai definisi yang sama. Untuk definisi
teknis per field lihat [`ERD-MASTER.md`](ERD-MASTER.md).

## Document Info

| Field | Value |
|-------|-------|
| **Version** | 1.1 |
| **Last Updated** | 27 September 2026 |
| **Domain** | Finance / Banking — Rekonsiliasi Bank |
| **Owner** | System Analyst (SH-005) — review bersama Supervisor Keuangan |

---

## Terms by Category

### 1. Business Process

#### Rekonsiliasi Bank

| Field | Value |
|-------|-------|
| **Definition** | Proses membandingkan transaksi yang tercatat di pembukuan internal perusahaan dengan mutasi rekening bank untuk memastikan keduanya sama dan selisihnya diketahui. |
| **Context** | Proses inti sistem ini, dijalankan setiap hari kerja pada jendela 06:00–07:30 WIB. |
| **Synonyms** | Rekonsiliasi harian, cek mutasi |
| **Avoid** | "Rekonsiliasi pajak" (proses berbeda) |
| **Example** | 18.000 baris bank dicocokkan dengan 16.000 transaksi ledger hari Senin. |
| **Source** | Jawaban question-framework aspek A |

#### Cut-off Harian

| Field | Value |
|-------|-------|
| **Definition** | Batas waktu ketika mutasi bank dianggap final untuk satu hari (06:00 WIB) dan batas selesainya rekonsiliasi (07:30 WIB). |
| **Context** | Menentukan seluruh jendela proses dan ketersediaan sistem. |
| **Synonyms** | Batas waktu mutasi, jam potong |
| **Avoid** | "Deadline" tanpa waktu spesifik |
| **Example** | Statement bank yang tiba setelah 06:00 masuk ke run hari berikutnya. |
| **Related** | Carry-over, ReconciliationRun |
| **Source** | Jawaban aspek A (cut-off harian) · CON-002 |

#### Carry-over

| Field | Value |
|-------|-------|
| **Definition** | Selisih yang belum terselesaikan pada hari itu dan dibawa ke hari berikutnya — maksimal 0,5% dari jumlah baris (90 dari 18.000). |
| **Context** | Ambang tutup hari; melebihi ini = hari tidak boleh ditutup. |
| **Synonyms** | Sisa bawa, utang hari berikutnya |
| **Avoid** | "Pending" (terlalu umum, bisa berarti apa saja) |
| **Example** | Hari itu tersisa 70 baris belum ketemu → masih di bawah 90, boleh ditutup. |
| **Related** | Aging, Penutupan hari |
| **Source** | Jawaban aspek A · CON-006 |

#### Pencocokan Berjenjang (Matching Tier)

| Field | Value |
|-------|-------|
| **Definition** | Urutan evaluasi aturan pencocokan dari yang paling ketat ke paling longgar: T1 referensi identik → T2 nominal identik + tanggal ≤1 hari → T3 agregasi → T4 keterangan mirip → T5 nominal parsial. |
| **Context** | Logika inti FR-005; sisa yang gagal di semua tier menjadi selisih. |
| **Synonyms** | Tier, hierarki aturan |
| **Avoid** | "Algoritma AI" — sistem memakai aturan eksplisit yang bisa ditest |
| **Example** | Baris dengan referensi `TRF-123` cocok di T1; baris QRIS tanpa referensi jatuh ke T3/T4. |
| **Related** | MatchRule, Skor keyakinan |
| **Source** | Gate G-01 (dikonfirmasi `[setuju]`) |

#### Agregasi

| Field | Value |
|-------|-------|
| **Definition** | Pencocokan banyak transaksi ledger ke satu baris bank (atau sebaliknya) dalam satu hasil pasangan berisi banyak leg. |
| **Context** | Dipakai untuk settlement harian per merchant dan biaya yang dipotong terpisah. |
| **Synonyms** | Grouping, pengelompokan |
| **Avoid** | "Gabung baris" — baris asli tidak pernah digabung atau diubah |
| **Example** | 40 transaksi settlement satu merchant tercatat sebagai 1 baris debit di bank. |
| **Related** | Split, MatchResultLeg |
| **Source** | Gate G-02 (dikonfirmasi `[setuju]`) |

#### Split

| Field | Value |
|-------|-------|
| **Definition** | Pemecahan satu transaksi ledger menjadi beberapa baris bank (misal transaksi utama + biaya administrasi terpisah). |
| **Context** | Sisi lain dari agregasi; sama-sama terekam sebagai banyak leg. |
| **Synonyms** | Pemecahan, pecahan biaya |
| **Avoid** | "Duplikat" — split sah, duplikat adalah kesalahan |
| **Example** | Settlement Rp 100.000 tercatat di bank sebagai Rp 97.000 masuk + Rp 3.000 biaya. |
| **Related** | Agregasi, Reason code `AMOUNT_DIFF` |
| **Source** | Gate G-02 |

#### Selisih / Unmatched Item

| Field | Value |
|-------|-------|
| **Definition** | Satu baris (dari salah satu sisi) yang gagal berpasangan setelah seluruh tier pencocokan dievaluasi, lengkap dengan alasan dan umurnya. |
| **Context** | Objek kerja utama analis pada layar penanganan selisih. |
| **Synonyms** | Item belum cocok, exception |
| **Avoid** | "Kesalahan" — tidak semua selisih adalah kesalahan (lihat Non-matching) |
| **Example** | Baris bank biaya admin Rp 2.500 tanpa pasangan ledger → selisih `ORPHAN_BANK`. |
| **Related** | Reason code, Aging |
| **Source** | Jawaban aspek A/B |

#### Non-matching (Sah Tidak Berpasangan)

| Field | Value |
|-------|-------|
| **Definition** | Selisih yang memang tidak perlu punya pasangan karena sifat transaksinya (biaya admin, saldo minimum, transfer antar-rekening internal) — tetap dicatat dan **tetap tampil** di laporan beserta volumenya. |
| **Context** | Kategori `NON_MATCHING` pada UnmatchedItem; wajib terlihat oleh Auditor. |
| **Synonyms** | Legitimately non-matching, selisih wajar |
| **Avoid** | "Dibuang", "diabaikan", "di-hide" — tidak boleh disembunyikan |
| **Example** | 2.000 baris biaya bank masuk kategori ini dan dilaporkan terpisah dari selisih nyata. |
| **Related** | Alasan `ORPHAN_BANK` / `ORPHAN_LEDGER` |
| **Source** | Keputusan disetujui user (masukan #2) + FR-010 |

#### Aging Selisih

| Field | Value |
|-------|-------|
| **Definition** | Jumlah hari sejak selisih pertama kali terdeteksi; dipakai untuk melihat selisih yang menumpuk. |
| **Context** | Laporan register selisih; peringatan bila >1 hari. |
| **Synonyms** | Umur selisih |
| **Avoid** | "Tanggal masuk" (hanya titik awal, bukan umur) |
| **Example** | Selisih terdeteksi Senin, belum selesai Rabu → aging 2 hari. |
| **Related** | Carry-over |
| **Source** | FR-011 |

#### Kendali Ganda (Dual-Control / 4-Eyes)

| Field | Value |
|-------|-------|
| **Definition** | Aturan bahwa satu tindakan (koreksi, tutup hari) harus dilakukan oleh dua orang berbeda: pengaju dan penyetuju tidak boleh orang yang sama. |
| **Context** | FR-013, FR-021; ditegakkan pada AdjustmentRequest. |
| **Synonyms** | Persetujuan dua orang, prinsip 4 mata |
| **Avoid** | "Approval" tanpa penjelasan siapa yang menyetujui |
| **Example** | Analis mengajukan koreksi Rp 8 juta → Supervisor menyetujui; karena >Rp 5 juta, Manajer Keuangan juga ikut menyetujui. |
| **Related** | Eskalasi >Rp 5 juta |
| **Source** | Gate G-04 · CON-003 |

#### Penutupan Hari (Tutup Buku Harian)

| Field | Value |
|-------|-------|
| **Definition** | Tindakan Supervisor menyatakan rekonsiliasi hari itu selesai, dengan syarat selisih yang dibawa ke besok ≤0,5% baris. |
| **Context** | Akhir alur harian; tercatat sebagai keputusan berotorisasi ganda. |
| **Synonyms** | Tutup hari, sign-off harian |
| **Avoid** | "Tutup buku bulanan" (proses berbeda, periode bulanan) |
| **Example** | Supervisor menutup hari Senin dengan 70 baris carry-over. |
| **Related** | Carry-over, Kendali ganda |
| **Source** | FR-021 · jawaban aspek A/C |

#### Control Total

| Field | Value |
|-------|-------|
| **Definition** | Jumlah ringkas yang dipakai untuk memeriksa kelengkapan: jumlah baris, total debit, dan total kredit — dibandingkan antara isi berkas bank, hasil proses, dan laporan. |
| **Context** | Validasi batch (FR-005) dan laporan harian (FR-015). |
| **Synonyms** | Jumlah pengendalian, total pemeriksaan |
| **Avoid** | "Total saldo" (beda hal) |
| **Example** | File bank menyatakan 4.500 baris, total debit Rp 812.000.000 — sistem memastikan isi file persis demikian sebelum diproses. |
| **Related** | StatementBatch |
| **Source** | FR-003, FR-015 |

---

### 2. Data Entity

#### Statement (Mutasi Bank)

| Field | Value |
|-------|-------|
| **Definition** | Daftar resmi dari bank berisi semua pergerakan dana pada suatu rekening dalam periode tertentu. |
| **Context** | Diterima sebagai berkas terjadwal 05:00 (3 bank) atau diunduh manual (1 bank). |
| **Synonyms** | Mutasi rekening, bank statement |
| **Avoid** | "Laporan bank" (bisa berarti laporan apa saja) |
| **Example** | Berkas mutasi 25 September berisi 4.500 baris. |
| **Related** | StatementBatch, BankStatementLine |
| **Source** | Jawaban aspek B/D |

#### Ledger Internal

| Field | Value |
|-------|-------|
| **Definition** | Catatan transaksi milik perusahaan sendiri (sistem pembayaran internal) — sisi pembanding dalam rekonsiliasi. |
| **Context** | ±16.000 transaksi settlement per hari. |
| **Synonyms** | Pembukuan internal, internal transaction |
| **Avoid** | "Buku besar akuntansi" (lebih luas dari isi sistem ini) |
| **Example** | 16.000 transaksi settlement merchant hari Senin. |
| **Related** | InternalTransaction |
| **Source** | Jawaban aspek B/D |

#### Referensi Transaksi

| Field | Value |
|-------|-------|
| **Definition** | Nomor yang ditulis bank (`reference_no`) dan sistem (`external_ref`) sebagai kunci pencocokan utama — boleh kosong pada transaksi tanpa nomor (mis. QRIS harian). |
| **Context** | Bahan pencocokan tier T1; normalisasi menyamakan beda penulisan antar bank. |
| **Synonyms** | Nomor referensi, nomor rujukan |
| **Avoid** | "Nomor transaksi" (bisa merujuk nomor sistem saja) |
| **Example** | Bank A menulis `TRF-123`, Bank B menulis `123` → dinormalisasi jadi `123`. |
| **Related** | Normalisasi, `MISSING_REF` |
| **Source** | Jawaban aspek B (variasi format antar bank) |

#### Nilai (Value Date) vs Tanggal Booking

| Field | Value |
|-------|-------|
| **Definition** | *Value date* = tanggal efektif dana berlaku (boleh H+1 dari transaksi); *tanggal booking* = tanggal bank mencatat. |
| **Context** | Value date dipakai untuk pencocokan — sumber utama selisih tanggal. |
| **Synonyms** | Tanggal nilai, tanggal efektif |
| **Avoid** | Menyamakan keduanya dalam satu istilah "tanggal" |
| **Example** | Transaksi Rabu malam punya value date Kamis di bank → selisih 1 hari, masih lolos tier T2. |
| **Related** | `DATE_DIFF`, tier T2 |
| **Source** | Jawaban aspek B · Gate G-01 alasan |

#### Reversal

| Field | Value |
|-------|-------|
| **Definition** | Pembatalan/transaksi balik atas transaksi yang sudah tercatat; di kedua sisi wajib berpasangan dengan reversal-nya sendiri. |
| **Context** | Menyebabkan reason code khusus `REVERSAL_MISMATCH` bila pasangannya tidak ditemukan. |
| **Synonyms** | Pembatalan, transaksi balik |
| **Avoid** | Menyamainya dengan refund (refund = pengembalian dana ke pelanggan, belum tentu reversal) |
| **Example** | Bank mencatat pembatalan Rp 500.000 tetapi ledger tidak mencatat pembatalan yang sama → `REVERSAL_MISMATCH`. |
| **Related** | `REVERSAL_MISMATCH`, `is_reversal` |
| **Source** | Jawaban aspek B (reversal wajib ketemu pasangan) + masukan #3 user |

#### Reason Code (Kode Alasan Selisih)

| Field | Value |
|-------|-------|
| **Definition** | Satu dari delapan kode wajib yang menjelaskan kenapa sebuah baris tidak berpasangan: `MISSING_REF`, `AMOUNT_DIFF`, `DATE_DIFF`, `DUPLICATE_REF`, `ORPHAN_BANK`, `ORPHAN_LEDGER`, `AGG_UNRESOLVED`, `REVERSAL_MISMATCH`. |
| **Context** | Wajib terisi pada setiap UnmatchedItem; dasar laporan dan tindakan. |
| **Synonyms** | Kode alasan, alasan selisih |
| **Avoid** | "Keterangan error" (harus dari daftar tetap) |
| **Example** | Baris bank ada, ledger tidak ada → `ORPHAN_BANK`. |
| **Related** | UnmatchedItem, Non-matching |
| **Source** | FR-009 + masukan #3 user (kode ke-8) |

#### Ruleset / Versi Aturan

| Field | Value |
|-------|-------|
| **Definition** | Kumpulan aturan pencocokan beserta tanggal berlakunya; setiap run mencatat ruleset mana yang dipakai. |
| **Context** | Memastikan hasil rekonsiliasi lampau tetap bisa dijelaskan setelah aturan berubah. |
| **Synonyms** | Paket aturan, konfigurasi aturan |
| **Avoid** | "Kode program" — aturan adalah data konfigurasi |
| **Example** | Run 25 Sep memakai ruleset v3 (toleransi 1 hari); setelah aturan berubah ke v4, run 25 Sep tetap dijelaskan dengan v3. |
| **Related** | MatchRule, FR-020 |
| **Source** | FR-019, FR-020 |

---

### 3. Role / Actor

#### Finance Ops Analyst

| Field | Value |
|-------|-------|
| **Definition** | Pengguna harian yang menjalankan rekonsiliasi, menyelesaikan selisih, dan mengajukan koreksi — **tanpa** kewenangan menghapus baris mentah. |
| **Context** | 3 orang; pemilik layar penanganan selisih. |
| **Synonyms** | Analis, staf rekonsiliasi |
| **Avoid** | "Admin" (implikasi kewenangan penuh) |
| **Example** | Analis memasangkan baris QRIS secara manual dengan alasan tercatat. |
| **Source** | Jawaban aspek C |

#### Supervisor Keuangan

| Field | Value |
|-------|-------|
| **Definition** | Pemilik keputusan tutup hari dan persetujuan koreksi; penerima eskalasi peringatan KRITIS. |
| **Context** | Satu orang; **tidak boleh** menyetujui koreksi yang diajukan sendiri. |
| **Synonyms** | Approver, atasan langsung |
| **Avoid** | "Manager" (tingkat berbeda — Manajer Keuangan lebih atas) |
| **Example** | Menyetujui koreksi yang diajukan analis; menutup hari Senin. |
| **Source** | Jawaban aspek C · Gate G-04 |

#### Acting As (Rangkap Peran)

| Field | Value |
|-------|-------|
| **Definition** | Pencatatan bahwa satu orang sedang menjalankan peran kedua (misal Supervisor menjadi analis saat cuti) — tercatat di jejak audit, bukan dengan akun kedua. |
| **Context** | Menjaga pertanggungjawaban tetap jelas saat rangkap peran. |
| **Synonyms** | Peran ganda, mewakili |
| **Avoid** | "Login bersama" (dilarang) |
| **Example** | Supervisor mengerjakan selisih sebagai analis → jejak mencatat `acting_as_role = ANALYST`. |
| **Related** | FR-023 |
| **Source** | Jawaban aspek C (rangkap peran) |

---

### 4. Status / State

#### Status Batch: `PENDING`

| Field | Value |
|-------|-------|
| **Definition** | Bank belum mengirim mutasi sesuai jadwal — proses tetap berjalan parsial dengan bank lain. |
| **Context** | Ditetapkan bila file belum tiba melewati jam yang diharapkan. |
| **Synonyms** | Menunggu, belum masuk |
| **Avoid** | "Gagal" (berbeda: `REJECTED` = file diterima tetapi tidak lolos validasi) |
| **Example** | Bank D belum kirim pukul 05:15 → batch `PENDING`, tiga bank lain tetap diproses. |
| **Source** | FR-002 · jawaban aspek A/D |

#### Status Run: `AWAITING_REVIEW` vs `CLOSED`

| Field | Value |
|-------|-------|
| **Definition** | `AWAITING_REVIEW` = proses otomatis selesai, menunggu penyelesaian selisih oleh manusia · `CLOSED` = Supervisor sudah menutup hari. |
| **Context** | Dua tahap akhir alur harian. |
| **Synonyms** | Menunggu review, selesai |
| **Avoid** | Menyebut keduanya "selesai" — belum tentu sama |
| **Example** | Proses otomatis selesai 06:28 → `AWAITING_REVIEW`; Supervisor menutup 07:20 → `CLOSED`. |
| **Related** | Penutupan hari |
| **Source** | FR-021 |

#### Peringatan WASPADA vs KRITIS

| Field | Value |
|-------|-------|
| **Definition** | WASPADA = selisih >90 baris (0,5%) pada 06:30 · KRITIS = selisih >200 baris → eskalasi ke Supervisor. |
| **Context** | Peringatan bertingkat FR-017 — WASPADA adalah peringatan dini sebelum batas carry-over terlampaui. |
| **Synonyms** | Ambang kuning, ambang merah |
| **Avoid** | Menyebutnya "notifikasi" tanpa tingkat |
| **Example** | Pukul 06:30 tersisa 150 baris → WASPADA; bila 240 baris → KRITIS + eskalasi. |
| **Source** | Keputusan #1 user (alert 2-tingkat) · FR-017 |

---

### 5. Technical (Domain-Specific)

#### Skor Keyakinan (Confidence Score)

| Field | Value |
|-------|-------|
| **Definition** | Angka 0,00–1,00 yang menyatakan seberapa yakin sistem sebuah pasangan benar; tier ketat = 1,00, makin rendah makin wajib direview manusia. |
| **Context** | Mengatur urutan antrean review analis. |
| **Synonyms** | Confidence, skor kecocokan |
| **Avoid** | "Probabilitas benar" — ini penanda aturan, bukan model statistik |
| **Example** | Pasangan tier T4 (keterangan mirip) skornya 0,85 → tampil di atas antrean review. |
| **Related** | MatchResult, FR-008 |
| **Source** | FR-008 |

#### Idempoten

| Field | Value |
|-------|-------|
| **Definition** | Sifat proses yang bisa dijalankan berulang dengan data yang sama tanpa menghasilkan data ganda dan tanpa menghapus hasil kerja sebelumnya. |
| **Context** | Run ulang pada hari yang sama mempertahankan koreksi manual (gate G-06). |
| **Synonyms** | Aman diulang |
| **Avoid** | "Reset" — kebalikan dari yang dimaksud |
| **Example** | Proses dijalankan 3 kali → jumlah baris hasil tetap sama, koreksi manual tetap ada. |
| **Related** | NFR-DATA-001, ASM-003 |
| **Source** | Gate G-06 |

#### Append-Only (Jejak Audit)

| Field | Value |
|-------|-------|
| **Definition** | Penyimpanan yang hanya menerima penambahan — tidak ada perubahan atau penghapusan baris yang sudah tercatat. |
| **Context** | Wajib untuk AuditEvent dan baris mutasi mentah. |
| **Synonyms** | Hanya tambah, immutable |
| **Avoid** | "Log biasa" (sering masih bisa diedit) |
| **Example** | Administrator mencoba menghapus jejak persetujuan → ditolak, upaya itu sendiri tercatat. |
| **Related** | NFR-SEC-003, NFR-DATA-002 |
| **Source** | Gate G-05 · CON-001 |

#### BR-CON vs CON (istilah penamaan)

| Field | Value |
|-------|-------|
| **Definition** | Dua kode yang rawan tertukar: `BR-CON-*` = **aturan bisnis** kategori kendala (5 aturan, katalog `business-rules.md` · §6 DESAIN-PROGRAM) — vs `CON-001…CON-009` = **kendala proyek** di `SRS-MASTER.md`. |
| **Context** | Membaca §6 bersama SRS; yang membedakan adalah awalan `BR-`. |
| **Synonyms** | BR-CON (aturan bisnis) vs CON (kendala proyek) |
| **Avoid** | Menyebut `CON-003` sebagai "aturan bisnis", atau `BR-CON-001` sebagai "kendala proyek" |
| **Example** | "Nilai di luar batas tolak" = `BR-CON-005` (aturan bisnis) · "retensi jejak 10 tahun" = `CON-001` (kendala proyek). |
| **Related** | §6 DESAIN-PROGRAM · `SRS-MASTER.md` |
| **Source** | Butir A13 `need-review.md` — catatan penamaan §6 |

---

### 6. Regulatory

#### OJK

| Field | Value |
|-------|-------|
| **Definition** | Otoritas Jasa Keuangan — regulator jasa keuangan di Indonesia yang ketentuannya menjadi dasar retensi 10 tahun dan kebutuhan jejak audit. |
| **Context** | Sumber constraint CON-001, NFR-COMP-001. |
| **Synonyms** | Regulator |
| **Avoid** | Menuliskan singkatan tanpa penjelasan pertama kali |
| **Example** | Pemeriksa meminta bukti rekonsiliasi 8 tahun lalu → harus tersedia. |
| **Source** | Jawaban aspek E (retensi 10 tahun OJK) |

#### UU PDP

| Field | Value |
|-------|-------|
| **Definition** | Undang-Undang No. 27 Tahun 2022 tentang Pelindungan Data Pribadi — mengatur pemrosesan data pribadi. |
| **Context** | Berlaku untuk data pribadi yang terbawa pada kolom keterangan mutasi. |
| **Synonyms** | UU Pelindungan Data Pribadi |
| **Avoid** | Menyamakannya dengan GDPR (regulasi berbeda) |
| **Example** | Nomor identitas pada keterangan ditampilkan terpotong untuk peran tanpa kebutuhan penuh. |
| **Source** | Jawaban aspek E · CON-007 |

---

## Conflict Resolution

| Scenario | Resolution |
|----------|------------|
| Istilah punya banyak definisi | Pakai definisi proyek ini; definisi lain dicatat di **Avoid**. |
| Bentrok dengan Data Dictionary | Data Dictionary menang untuk definisi teknis; Glossary menang untuk definisi bisnis. |
| Istilah dipakai tapi belum ada di glossary | Tambahkan segera + tandai untuk review. |
| Klien memakai istilah lain | Istilah klien dicatat sebagai **Synonyms**; istilah proyek yang dipakai. |

---

## Term Count Summary

| Category | Count |
|----------|-------|
| Business Process | 12 |
| Data Entity | 7 |
| Role/Actor | 3 |
| Status/State | 3 |
| Technical (domain) | 4 |
| Regulatory | 2 |
| **Total** | **31** |

## Sign-off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Business Owner (Manajer Keuangan) | {待isi} | {—} | ___________ |
| Domain Expert (Supervisor Keuangan) | {待isi} | {—} | ___________ |
| SA Lead (System Analyst) | {待isi} | {—} | ___________ |
