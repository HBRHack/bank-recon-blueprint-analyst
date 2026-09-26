# SRS-MASTER: Bank Reconciliation & Statement Mapping System

> Master lintas modul — detail use case & process flow per fitur akan ada di
> `BAB-{N}-*/USE-CASE.md` dan `BAB-{N}-*/PROCESS-FLOW.md` (**BAB 1 & 2 ditunda**).
> Aturan notasi: [`00-Global/NOTATION.md`](NOTATION.md) ·
> Struktur data: [`00-Global/ERD-MASTER.md`](ERD-MASTER.md) ·
> NFR detail: [`../nfr.md`](../nfr.md)

Sistem yang mencocokkan (reconcile) transaksi ledger internal dengan mutasi bank
partner setiap hari, menandai selisih yang tidak cocok beserta alasannya, dan
menyiapkan jalur koreksi yang butuh persetujuan dua orang — untuk menggantikan
rekonsiliasi manual berbasis lembar kerja spread-sheet.

## 1. System Overview

### 1.1 Purpose

Saat ini 3 Finance Ops Analyst mencocokkan ±18.000 baris mutasi bank harian
dengan ±16.000 transaksi ledger secara manual (filter spread-sheet), mulai
06:30–10:00 WIB dan sering molor ke jam 11. Selisih yang telat terdeteksi membuat
settlement merchant Rp 1,2–2 M mengambang dan divisi klaim komplain bila >1 hari.
Sistem ini mempercepat selesai rekonsiliasi ke target 20 menit, menurunkan
penelusuran manual, dan menghasilkan jejak audit yang memenuhi ketentuan pengawasan.

### 1.2 Scope

**Termasuk (in-scope):**

| # | Kapabilitas |
|---|-------------|
| 1 | Penerimaan mutasi bank: 3 bank otomatis (kanal berkas terjadwal 05:00) + 1 bank manual (unduh portal) |
| 2 | Validasi struktur & control total per batch (jumlah baris, total debit/kredit, isi berkas) |
| 3 | Normalisasi kolom lintas bank (profil format per bank) menjadi bentuk baku |
| 4 | Pencocokan otomatis berjenjang T1→T5, termasuk agregasi (N ledger → 1 baris bank) dan split (1 baris bank → N ledger) |
| 5 | Penanganan selisih (exception) dengan 8 alasan + kategori non-matching yang sah tidak berpasangan |
| 6 | Pencocokan manual oleh analis dengan kandidat saran |
| 7 | Pengajuan & persetujuan koreksi (dual-control), koreksi tercatat sebagai baris baru berikutnya |
| 8 | Tutup hari rekonsiliasi (sign-off) dengan ambang carry-over 0,5% |
| 9 | Laporan harian, register selisih + aging, statistik per tier, ekspor audit |
| 10 | Pengaturan aturan pencocokan oleh System Analyst dengan persetujuan Supervisor |
| 11 | Jejak audit append-only untuk semua aksi tulis |

**Tidak termasuk (out-of-scope) — eksplisit:**

| # | Di luar scope | Alasan |
|---|---------------|--------|
| 1 | Proses pembayaran / transfer / kliring | Ranah sistem pembayaran, bukan rekonsiliasi |
| 2 | Pembuatan statement oleh bank | Bank tetap mengirim file, sistem hanya menerima |
| 3 | Konversi mata uang (FX) | Semua rekening lokal wajib IDR |
| 4 | Pembukuan umum / posting jurnal utama | Sistem hanya **menyiapkan** koreksi; posting jurnal tetap di sistem ledger |
| 5 | Deteksi fraud/AML | Ranah compliance terpisah |
| 6 | Manajemen klaim merchant end-to-end | Sistem hanya menandai selisih yang memicu klaim |
| 7 | Penyimpanan data di luar ketentuan retensi | Retensi 10 tahun mengikat |

### 1.3 Actors

| Actor | Description | Primary Interface |
|-------|-------------|-------------------|
| Finance Ops Analyst | Menjalankan/memonitor rekonsiliasi harian, menyelesaikan selisih, mengajukan koreksi | Aplikasi web (layar review) |
| Supervisor Keuangan | Menyetujui koreksi, menutup hari, menerima peringatan eskalasi | Aplikasi web (layar persetujuan) |
| Auditor Internal | Membaca laporan & mengunduh jejak audit (read-only) | Aplikasi web (baca) + ekspor |
| System Analyst | Mengatur parameter aturan pencocokan (toleransi, tier) — **tidak boleh** menyetujui tutup buku | Aplikasi web (layar pengaturan) |
| Sistem (penjadwal) | Memicu proses otomatis 05:00, mengirim peringatan, menulis log | Otomatis, tanpa antarmuka manusia |
| Bank Partner | Pihak eksternal pengirim mutasi (4 bank) | Kanal berkas / portal bank |
| Sistem Ledger Internal | Sumber transaksi settlement harian + penerima koreksi | Antarmuka data internal |

### 1.4 Business Objectives & Success Metrics

**Objectives:**

| ID | Objective | Owner |
|----|-----------|-------|
| OBJ-001 | Menurunkan waktu penyelesaian rekonsiliasi harian dari 3,5 jam menjadi 20 menit | Manajer Keuangan |
| OBJ-002 | Menurunkan potensi rugi akibat selisih tak terdeteksi dari ±Rp 50 juta/bulan menjadi <Rp 5 juta/bulan | Manajer Keuangan |
| OBJ-003 | Menurunkan baris yang harus ditelusuri manual dari ±1.800/hari menjadi <200/hari | Supervisor Keuangan |
| OBJ-004 | Menjaga jejak audit memenuhi ketentuan pengawasan: retensi 10 tahun, tidak bisa diubah | Auditor Internal |
| OBJ-005 | Mempercepat penyelesaian selisih klaim merchant dari >1 hari menjadi <4 jam | Supervisor Keuangan |

**Success Metrics (diukur 1–3 bulan pasca go-live):**

| Metric | Baseline (sekarang) | Target | Cara Ukur |
|--------|--------------------|--------|-----------|
| Waktu selesai rekonsiliasi harian | 3,5 jam (06:30–10:00, molor 11:00) | 20 menit (selesai ≤06:50) | Stempel waktu proses run + tanda tutup hari |
| % baris terpasangkan otomatis | ±90% via filter manual | ≥97% | Statistik hasil pencocokan per tier |
| Baris ditelusuri manual/hari | ±1.800 | <200 | Register selisih per hari (kategori `UNMATCHED`) |
| Selisih mengendap >1 hari | Sering (settlement Rp 1,2–2 M) | 0 (semua selesai hari yang sama) | Aging selisih |
| Selisih tak terdeteksi/bulan | ±Rp 50 juta | <Rp 5 juta | Hasil rekonsiliasi koreksi + temuan auditor |
| Pelanggaran retensi/jejak audit | 0 (target terjaga) | 0 | Pemeriksaan auditor: baris mentah + audit trail utuh |

> Setiap metric punya baseline + target angka + cara ukur. "Lebih efisien"
> tanpa angka = ditolak (gagal kriteria Testable/CLEAR). Metric ini jadi acuan
> UAT exit & lesson-learned (`analyst-uat-planner`).

## 2. Functional Requirements

> Status: `[x]` Confirmed = bersumber dari jawaban question-framework yang sudah
> dikonfirmasi · `[ ]` Pending = turunan yang belum divalidasi.
> Kolom **Chapter** masih `00-Global` — akan dipindah ke `BAB-{N}-*` setelah
> struktur bab disusun (BAB 1 & 2 ditunda).

### 2.1 Penerimaan & Validasi Mutasi Bank

#### FR-001: Penerimaan mutasi terjadwal
- **Description:** Sistem menerima mutasi dari 3 bank otomatis pada 05:00 WIB dan menyediakan jalur unggah manual untuk 1 bank portal; setiap penerimaan mencatat waktu, nama berkas, dan sumbernya.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-002: Penanda keterlambatan statement
- **Description:** Bila satu bank belum mengirim pada jam yang diharapkan (05:00), batch-nya ditandai `PENDING` dan proses tetap berjalan parsial dengan bank yang sudah lengkap — tidak menunggu semua bank.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-003: Validasi struktur & control total per batch
- **Description:** Sebelum diproses, sistem memeriksa jumlah baris, total debit, total kredit, dan nilai pembanding isi berkas terhadap isi aktual; selisih → batch `REJECTED` dengan alasan.
- **Priority:** Must Have
- **Status:** [ ] Pending

#### FR-004: Normalisasi lintas bank
- **Description:** Setiap baris bank dipetakan ke bentuk baku memakai profil format per bank (`FMT_A`/`FMT_B`/`FMT_C`) sehingga perbedaan penulisan referensi (mis. `TRF-123` vs `123`) tidak menggagalkan pencocokan.
- **Priority:** Must Have
- **Status:** [x] Confirmed

### 2.2 Pencocokan Otomatis

#### FR-005: Pencocokan berjenjang T1→T5
- **Description:** Sistem mengevaluasi aturan berurutan: T1 referensi identik → T2 nominal identik + tanggal ≤1 hari → T3 agregasi kelompok → T4 keterangan mirip → T5 nominal parsial; sisa menjadi selisih.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-006: Dukungan agregasi & split
- **Description:** Satu baris bank boleh mewakili banyak transaksi ledger (settlement harian per merchant) dan satu transaksi ledger boleh terpecah menjadi beberapa baris bank (biaya terpisah), keduanya terekam sebagai banyak leg pada satu hasil pencocokan.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-007: Anti pasangan ganda
- **Description:** Satu baris bank dan satu transaksi ledger hanya boleh berpasangan sekali sepanjang waktu; upaya memaksa pasangan ganda ditolak dan masuk selisih.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-008: Skor keyakinan & antrean review
- **Description:** Setiap hasil pencocokan membawa skor 0,00–1,00; skor rendah diurutkan paling atas dalam antrean review manusia.
- **Priority:** Should Have
- **Status:** [ ] Pending

### 2.3 Penanganan Selisih (Exception)

#### FR-009: Pengkodean alasan selisih
- **Description:** Setiap selisih diberi satu dari 8 alasan: `MISSING_REF`, `AMOUNT_DIFF`, `DATE_DIFF`, `DUPLICATE_REF`, `ORPHAN_BANK`, `ORPHAN_LEDGER`, `AGG_UNRESOLVED`, `REVERSAL_MISMATCH`.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-010: Kategori non-matching tetap terlihat
- **Description:** Selisih yang sah tidak perlu berpasangan (biaya admin, saldo, transfer antar-rekening internal) diberi kategori `NON_MATCHING` — **tetap tampil** di laporan beserta volumenya, tidak boleh disembunyikan dari Auditor.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-011: Aging & penugasan selisih
- **Description:** Setiap selisih mencatat hari pertama terdeteksi, umur (aging), penyelesaian, dan analis penanggung jawab; selisih yang menumpuk >1 hari memicu peringatan.
- **Priority:** Must Have
- **Status:** [ ] Pending

### 2.4 Koreksi & Persetujuan

#### FR-012: Pencocokan manual oleh analis
- **Description:** Analis dapat memasangkan baris bank dengan transaksi ledger secara manual; wajib mencatat alasan dan tercatat sebagai tindakan audit.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-013: Koreksi dengan persetujuan dua orang
- **Description:** Koreksi disiapkan oleh analis, disetujui Supervisor; nama pengaju dan penyetuju wajib berbeda, dan penyetuju wajib berbeda dari pengaju.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-014: Koreksi menjadi baris baru
- **Description:** Koreksi yang disetujui tercatat sebagai transaksi ledger baru pada siklus berikutnya — baris asli tidak pernah diubah, sehingga tidak terjadi hitung ganda.
- **Priority:** Must Have
- **Status:** [x] Confirmed

### 2.5 Laporan, Notifikasi & Ekspor

#### FR-015: Ringkasan rekonsiliasi harian
- **Description:** Laporan berisi control total per sisi (jumlah baris, total debit, total kredit), jumlah terpasangkan per tier, jumlah selisih per kategori, dan status tutup hari.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-016: Register selisih + aging
- **Description:** Daftar selisih lengkap dengan alasan, umur, penanggung jawab, dan status — termasuk kategori `NON_MATCHING` dengan volumenya.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-017: Peringatan bertingkat
- **Description:** Pada 06:30 WIB, bila jumlah selisih >90 baris (0,5%) sistem menandai **WASPADA**; bila >200 baris sistem menandai **KRITIS** dan mengirim eskalasi ke Supervisor (saluran pesan/email).
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-018: Ekspor laporan untuk Auditor
- **Description:** Auditor dapat mengunduh ringkasan, register selisih, dan jejak audit dalam format data terstruktur maupun dokumen cetak, sesuai izin akses baca-saja.
- **Priority:** Must Have
- **Status:** [x] Confirmed

### 2.6 Pengaturan Aturan & Otoritas

#### FR-019: Pengaturan parameter aturan
- **Description:** System Analyst dapat mengubah parameter aturan (mis. toleransi tanggal 1→3 hari); perubahan wajib disetujui Supervisor dan berlaku untuk run berikutnya.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-020: Versi aturan per run
- **Description:** Setiap run mencatat versi aturan yang dipakai sehingga hasil rekonsiliasi lampau tetap bisa dijelaskan meski aturan sudah berubah.
- **Priority:** Should Have
- **Status:** [ ] Pending

#### FR-021: Penutupan hari rekonsiliasi
- **Description:** Supervisor menutup hari setelah memastikan selisih ≤0,5% baris (90 dari 18.000) boleh carry-over; penutupan tercatat sebagai keputusan berotorisasi ganda.
- **Priority:** Must Have
- **Status:** [x] Confirmed

### 2.7 Audit & Jejak

#### FR-022: Jejak audit append-only
- **Description:** Semua aksi tulis (pencocokan manual, perubahan aturan, persetujuan, tutup hari, login) tercatat dengan pelaku, waktu, nilai lama & baru — tidak bisa diubah atau dihapus siapa pun, termasuk administrator.
- **Priority:** Must Have
- **Status:** [x] Confirmed

#### FR-023: Catatan rangkap peran
- **Description:** Saat satu orang menjalankan lebih dari satu peran, jejak audit mencatat peran yang sedang dijalankan (`acting_as`) agar pertanggungjawaban tetap jelas.
- **Priority:** Should Have
- **Status:** [x] Confirmed

## 3. Non-Functional Requirements

Ringkas — definisi, skenario QA 6 bagian, dan metode verifikasi lengkap ada di
[`../nfr.md`](../nfr.md) (16 NFR, 8 kategori).

| Kategori | Jumlah | Critical |
|----------|--------|----------|
| Performance | 4 | 2 |
| Availability | 2 | 2 |
| Security | 4 | 3 |
| Compliance | 2 | 2 |
| Data Management | 3 | 2 |
| Scalability | 1 | 0 |
| Usability | 2 | 0 |
| Maintainability | 2 | 1 |
| **Total** | **20** | **12** |

Poin inti: proses 18.000 baris ≤30 menit · volume H+1 3× lipat selesai sebelum
07:00 · jendela kritis 05:00–07:30 tersedia · RTO 60 menit · 2FA + kunci 5× gagal
login · jejak audit tak-terubah · retensi 10 tahun (OJK) + UU PDP · idempoten
(run ulang tidak menggandakan; koreksi manual dipertahankan) · aturan boleh diubah
tanpa menyentuh logika program.

## 4. Data Requirements

Ringkas — definisi per entitas, per field, tipe logis, konstrains, dan indeks ada
di [`00-Global/ERD-MASTER.md`](ERD-MASTER.md).

| Entitas | Peran | Sifat data |
|---------|-------|------------|
| StatementSource / StatementBatch | Sumber & penerimaan file mutasi | Master + transaksi penerimaan |
| BankStatementLine | Baris mutasi bank | **Mentah, tidak boleh diubah/dihapus** |
| InternalTransaction | Transaksi ledger internal | Transaksi, idempoten per hari |
| ReconciliationRun | Satu eksekusi rekonsiliasi harian | Kontrol proses (1 hari = 1 run) |
| MatchRule | Aturan pencocokan berversi | Konfigurasi, diarsip (tidak dihapus) |
| MatchResult + MatchResultLeg | Hasil pencocokan (bridge M:N) | Bukti pasangan, mendukung agregasi & split |
| UnmatchedItem | Selisih + 8 alasan + kategori | Terekam, aging, tidak pernah hilang |
| AdjustmentRequest | Pengajuan/persetujuan koreksi | Dengan kendali ganda |
| SystemUser / AuditEvent | Pengguna & jejak audit | Append-only |

## 5. Integration Points

| System | Kanal | Purpose | Direction |
|--------|-------|---------|-----------|
| Bank Partner (3 bank) | Berkas terjadwal 05:00 WIB | Mengirim mutasi harian | Inbound |
| Bank Partner (1 bank) | Unduh portal (manual) | Mengambil mutasi harian | Inbound |
| Sistem Ledger Internal | Baca laporan harian settlement | Sumber ±16.000 transaksi/hari | Inbound |
| Sistem Ledger Internal | Kirim koreksi yang sudah disetujui | Mencatat koreksi sebagai transaksi baru | Outbound |
| Kanal pesan/email Supervisor | Peringatan bertingkat (WASPADA/KRITIS) | Eskalasi selisih >90 / >200 baris | Outbound |
| Ekspor Auditor | Unduh on-demand (data terstruktur + dokumen cetak) | Bukti audit | Outbound |

**Perilaku saat sambungan gagal:** proses berjalan parsial dengan data yang sudah
ada; bank yang belum masuk tetap `PENDING` (tidak menunggu semua bank). Kasus
khusus: portal bank kecil hanya menyimpan data 30 hari → perlu pengingat otomatis
agar tidak kehilangan data.

## 6. Constraints

| ID | Constraint | Kategori | Severity |
|----|-----------|----------|----------|
| CON-001 | Retensi baris mentah + jejak audit 10 tahun, tidak boleh diubah | Regulatory | Hard |
| CON-002 | Statement final 06:00 WIB; rekonsiliasi wajib selesai 07:30 WIB | Timeline | Hard |
| CON-003 | Persetujuan dua orang untuk koreksi & tutup hari; >Rp 5 juta naik ke Manajer Keuangan | Regulatory | Hard |
| CON-004 | Semua rekening lokal dalam IDR — tanpa konversi mata uang | Business | Hard |
| CON-005 | Toleransi pencocokan maksimal Rp 1 (pembulatan) | Business | Hard |
| CON-006 | Carry-over maksimal 0,5% baris (90 dari 18.000/hari) | Business | Hard |
| CON-007 | Data pribadi pada kolom keterangan tunduk UU PDP | Regulatory | Hard |
| CON-008 | Portal 1 bank hanya menyimpan data 30 hari | Infrastructure | Soft |
| CON-009 | Dokumen desain tanpa keterikatan bahasa/framework/engine tertentu | Process | Hard |

Detail lengkap: [`../assumptions-constraints.md`](../assumptions-constraints.md).

## 7. Assumptions

| ID | Asumsi | Status bila belum diverifikasi |
|----|--------|-------------------------------|
| ASM-001 | Volume ±18.000 baris/hari (4 bank) dan ±16.000 transaksi ledger/hari | Open |
| ASM-002 | Selisih ±2.000 baris antara kedua sisi sebagian besar = biaya admin/saldo/transfer internal (bukan kesalahan) | Open |
| ASM-003 | Proses otomatis cukup satu putaran pada jendela cut-off; diulang = `RERUN` | Verified (G-06) |
| ASM-004 | Data historis 3 bulan tersedia untuk uji cocok | Open |
| ASM-005 | Tidak ada bank yang mengirim mutasi lebih dari 1× per hari per rekening | Open |

Detail + rencana verifikasi: [`../assumptions-constraints.md`](../assumptions-constraints.md).

## 8. Open Questions

| # | Pertanyaan | Pemilik | Dampak bila belum terjawab |
|---|-----------|---------|---------------------------|
| OQ-001 | Apakah 2.000 baris selisih benar-benar mayoritas non-matching (perlu sampel 1 bulan)? | Supervisor Keuangan | Menentukan besarnya beban review awal |
| OQ-002 | Apakah koreksi yang sudah disetujui dikirim ke ledger otomatis atau menunggu proses terjadwal? | System Analyst | Menentukan FR-014 pada bab detail |
| OQ-003 | Siapa pemilik formal dokumen ini untuk sign-off? | Manajer Keuangan | Menentukan sign-off matrix |
| OQ-004 | Perlu restorasi bila run gagal di tengah (kembali ke kondisi sebelum run)? | System Analyst | Menentukan desain pemulihan |

> BAB 1 (Problem Statement/Scope) dan BAB 2 (Actor/Use Case detail) **ditunda** —
> materi di halaman ini menjadi bahan utamanya saat bab dibuat.
