# Assumption & Constraint Log: Bank Reconciliation & Statement Mapping System

Asumsi = hal yang kita anggap benar tetapi belum diverifikasi — kalau salah,
requirement berubah. Constraint = batas keras yang tidak bisa dinegosiasi —
mendefinisikan batas ruang solusi.

## Document Info

| Field | Value |
|-------|-------|
| **Version** | 1.0 |
| **Last Updated** | 26 September 2026 |
| **Owner** | System Analyst (SH-005) |

---

## ASSUMPTIONS

### ASM-001: Volume harian ±18.000 baris mutasi & ±16.000 transaksi ledger

| Field | Value |
|-------|-------|
| **ID** | ASM-001 |
| **Date Made** | 26 Sep 2026 |
| **Made By** | System Analyst |
| **Status** | Open |
| **Priority** | High |
| **Impact if Wrong** | Desain kapasitas & estimasi durasi proses (NFR-PERF-001/002, NFR-SCALE-001) meleset; bila ternyata 3× lebih besar, jendela 30 menit tidak tercapai |

**Assumption:** ±18.000 baris mutasi/hari (4 bank, ±4.500 per rekening) dan
±16.000 transaksi ledger/hari; puncak H+1 = 3× lipat (±54.000 baris).

**Basis:** Jawaban question-framework aspek A (volume & tingkat match manual)
dan E (total baris & batas waktu proses).

**Verification Plan:** Ambil hitungan riil 1 bulan penuh per bank + hari tutup
bulan lalu; bandingkan dengan angka asumsi sebelum desain final.

**Verification Result:** Pending — belum diambil data riil.

**Related Requirements:** FR-005, FR-006 · NFR-PERF-001, NFR-PERF-002, NFR-SCALE-001

---

### ASM-002: Selisih ±2.000 baris/hari sebagian besar bukan kesalahan

| Field | Value |
|-------|-------|
| **ID** | ASM-002 |
| **Date Made** | 26 Sep 2026 |
| **Made By** | System Analyst |
| **Status** | Open |
| **Priority** | High |
| **Impact if Wrong** | Bila ternyata sebagian besar adalah selisih nyata, beban review jauh lebih besar dari perkiraan dan target "selesai 20 menit" sulit tercapai |

**Assumption:** Selisih awal antara kedua sisi (±18.000 vs ±16.000) sebagian
besar berasal dari baris bank yang memang tidak punya pasangan ledger: biaya
admin yang dipotong bank, saldo minimum/biaya bulanan, transfer antar-rekening
internal — bukan kesalahan data.

**Basis:** Jawaban aspek A (penyebab selisih tersering: biaya admin di baris
terpisah) + keputusan disetujui bahwa kategori ini = "legitimately
non-matching" dan **tetap tampil di laporan** (FR-010) — bukan disembunyikan.

**Verification Plan:** Sampel 1 bulan mutasi → klasifikasi manual 100% baris
yang tidak berpasangan ke 8 reason code; hitung proporsi `NON_MATCHING` vs
`UNMATCHED` (jawab OQ-001).

**Verification Result:** Pending — sampel belum diambil.

**Related Requirements:** FR-009, FR-010, FR-015, FR-016

---

### ASM-003: Satu run per hari; diulang = jalur ulang, bukan run baru

| Field | Value |
|-------|-------|
| **ID** | ASM-003 |
| **Date Made** | 26 Sep 2026 |
| **Made By** | System Analyst (gate G-06) |
| **Status** | **Verified** (dikonfirmasi user sebagai asumsi final) |
| **Priority** | High |
| **Impact if Wrong** | Bila run ulang harus jadi entitas terpisah, logika idempotensi & laporan harian perlu redesain |

**Assumption:** Satu hari = satu `ReconciliationRun`. Menjalankan ulang memakai
jalur ulang (`RERUN`) pada run yang sama, **mempertahankan** koreksi manual yang
sudah ada (tidak direset), dan tidak menghasilkan baris ganda.

**Basis:** Gate G-06 dikonfirmasi `[setuju]`: "jika diulang, koreksi manual
dipertahankan — bukan direset."

**Verification Plan:** Uji run ulang ≥3 kali dengan koreksi manual di dalamnya
(NFR-DATA-001).

**Verification Result:** Konfirmasi requirement ✓ — pengujian teknis menunggu implementasi.

**Related Requirements:** FR-005, FR-014 · NFR-DATA-001

---

### ASM-004: Data historis 3 bulan tersedia untuk uji cocok

| Field | Value |
|-------|-------|
| **ID** | ASM-004 |
| **Date Made** | 26 Sep 2026 |
| **Made By** | System Analyst |
| **Status** | Open |
| **Priority** | Medium |
| **Impact if Wrong** | Uji cocok & paralel run 2 minggu tidak bisa memakai data nyata → kualitas uji turun |

**Assumption:** Mutasi bank dan ledger 3 bulan terakhir masih bisa ditarik
(termasuk dari portal bank yang hanya menyimpan 30 hari → perlu segera diunduh).

**Basis:** Jawaban aspek E (masa transisi: data historis 3 bulan untuk uji cocok)
+ constraint portal 30 hari (CON-008).

**Verification Plan:** Konfirmasi ketersediaan per bank minggu ke-1 project;
untuk portal 30 hari, unduh segera & simpan sebagai bahan uji.

**Verification Result:** Pending.

**Related Requirements:** FR-001 · CON-008

---

### ASM-005: Tidak ada bank yang mengirim mutasi lebih dari 1× per hari per rekening

| Field | Value |
|-------|-------|
| **ID** | ASM-005 |
| **Date Made** | 26 Sep 2026 |
| **Made By** | System Analyst |
| **Status** | Open |
| **Priority** | Medium |
| **Impact if Wrong** | Bila ada kiriman susulan sore hari, jendela cut-off 06:00 harus diubah atau perlu mekanisme tambahan khusus kiriman susulan |

**Assumption:** Setiap bank mengirim satu file per rekening per hari, tiba
sebelum 06:00 WIB.

**Basis:** Jawaban aspek D (jadwal 05:00, satu bank manual) — belum ada
penyebutan kiriman kedua.

**Verification Plan:** Konfirmasi per bank saat interface analysis; periksa
catatan 3 bulan terakhir untuk adanya kiriman kedua.

**Verification Result:** Pending.

**Related Requirements:** FR-001, FR-002 · NFR-AVAIL-001

---

### ASM-006: Seluruh rekening dalam satu mata uang (IDR)

| Field | Value |
|-------|-------|
| **ID** | ASM-006 |
| **Date Made** | 26 Sep 2026 |
| **Made By** | System Analyst |
| **Status** | **Verified** |
| **Priority** | Medium |
| **Impact if Wrong** | Bila ada rekening USD, seluruh logika pencocokan (banding nominal) dan laporan control total perlu memperhitungkan kurs |

**Assumption:** Semua rekening partner dalam IDR; tidak ada konversi mata uang
dalam sistem ini.

**Basis:** Jawaban aspek B (mata uang wajib IDR) + konfirmasi gate.

**Verification Plan:** Daftar rekening partner dikonfirmasi oleh Finance.

**Verification Result:** Dikonfirmasi pada jawaban question-framework ✓

**Related Requirements:** FR-003, FR-005 · CON-004

---

## CONSTRAINTS

### CON-001: Retensi 10 tahun & jejak audit tidak boleh diubah

| Field | Value |
|-------|-------|
| **ID** | CON-001 |
| **Category** | Regulatory |
| **Severity** | Hard |
| **Source** | Ketentuan pengawasan lembaga keuangan (OJK) — SH-008 / SH-004 |

**Constraint:** Baris mutasi mentah, hasil rekonsiliasi, dan jejak audit wajib
disimpan ≥10 tahun dan tidak dapat diubah atau dihapus oleh peran manapun
(termasuk administrator).

**Impact:** Menutup opsi "edit baris" dan "hapus jejak" — semua koreksi harus
berupa transaksi baru; penyimpanan harus append-only dengan mekanisme deteksi
perusakan.

**Workaround:** Tidak ada — ini batas keras. Arsip ulang tahun lama boleh
dipindahkan ke penyimpanan jangka panjang selama tetap dapat dipulihkan utuh.

**Related Requirements:** FR-014, FR-022 · NFR-COMP-001, NFR-SEC-003, NFR-DATA-002

---

### CON-002: Cut-off 06:00 WIB & selesai 07:30 WIB

| Field | Value |
|-------|-------|
| **ID** | CON-002 |
| **Category** | Timeline |
| **Severity** | Hard |
| **Source** | SH-001 Manajer Keuangan |

**Constraint:** Statement dianggap final pukul 06:00 WIB; seluruh rekonsiliasi
wajib selesai pukul 07:30 WIB dan ditandatangani Supervisor.

**Impact:** Menentukan seluruh jendela proses (≤30 menit), ketersediaan jendela
kritis 05:00–07:30, dan perilaku fallback bila melewati 06:30.

**Workaround:** Bila gagal, rekonsiliasi hari itu jalan jalur manual — bukan
ditunda.

**Related Requirements:** FR-001, FR-017, FR-021 · NFR-PERF-001, NFR-AVAIL-001, NFR-AVAIL-002

---

### CON-003: Kendali ganda + eskalasi >Rp 5 juta

| Field | Value |
|-------|-------|
| **ID** | CON-003 |
| **Category** | Regulatory |
| **Severity** | Hard |
| **Source** | SH-001 Manajer Keuangan (jawaban aspek A & C) |

**Constraint:** Koreksi dan tutup hari butuh dua orang berbeda (pengaju ≠
penyetuju). Koreksi >Rp 5 juta wajib naik ke Manajer Keuangan.

**Impact:** Alur persetujuan wajib dua langkah; Sistem Analyst tidak boleh
memegang persetujuan tutup buku.

**Workaround:** Tidak ada — prinsip pemisahan tugas.

**Related Requirements:** FR-013, FR-021 · NFR-SEC-002

---

### CON-004: Tanpa konversi mata uang (IDR saja)

| Field | Value |
|-------|-------|
| **ID** | CON-004 |
| **Category** | Business |
| **Severity** | Hard |
| **Source** | Finance (SH-002) |

**Constraint:** Semua rekening partner dalam IDR; sistem tidak memproses
konversi mata uang.

**Impact:** Pencocokan cukup membandingkan nominal langsung; laporan tanpa
posisi kurs.

**Workaround:** Bila kelak ada rekening asing, perlu penambahan desain — di luar
scope rilis ini.

**Related Requirements:** FR-003, FR-005

---

### CON-005: Toleransi pencocokan maksimal Rp 1

| Field | Value |
|-------|-------|
| **ID** | CON-005 |
| **Category** | Business |
| **Severity** | Hard |
| **Source** | SH-002 Supervisor Keuangan (jawaban aspek B) |

**Constraint:** Perbedaan nominal ≤Rp 1 (pembulatan) boleh dianggap cocok;
di atas itu wajib jadi selisih.

**Impact:** Menjadi parameter batas atas `amt_tolerance`; perubahan di atas Rp 1
butuh keputusan Finance, bukan sekadar konfigurasi teknis.

**Workaround:** Tidak ada — nilai batas bersifat kebijakan.

**Related Requirements:** FR-005, FR-019

---

### CON-006: Carry-over maksimal 0,5% baris (90 dari 18.000)

| Field | Value |
|-------|-------|
| **ID** | CON-006 |
| **Category** | Business |
| **Severity** | Hard |
| **Source** | SH-002 Supervisor Keuangan (jawaban aspek A) |

**Constraint:** Maksimal 0,5% baris boleh berstatus belum selesai dan dibawa ke
hari berikutnya; sisanya wajib matched/resolved pada hari yang sama.

**Impact:** Menjadi ambang tutup hari + ambang WASPADA (90 baris) pada peringatan
bertingkat.

**Workaround:** Tidak ada — batas harian.

**Related Requirements:** FR-017, FR-021

---

### CON-007: Data pribadi pada kolom keterangan tunduk UU PDP

| Field | Value |
|-------|-------|
| **ID** | CON-007 |
| **Category** | Regulatory |
| **Severity** | Hard |
| **Source** | UU No. 27/2022 — SH-004 |

**Constraint:** Keterangan mutasi berpotensi memuat data pribadi; pemrosesan
harus sesuai tujuan, akses dibatasi per peran, dan tercatat.

**Impact:** Pemotongan tampilan untuk peran tanpa kebutuhan penuh; larangan
mencatat data pribadi utuh ke log operasional.

**Workaround:** Tidak ada — kepatuhan.

**Related Requirements:** FR-018, FR-022 · NFR-SEC-004, NFR-COMP-002

---

### CON-008: Portal 1 bank hanya menyimpan data 30 hari

| Field | Value |
|-------|-------|
| **ID** | CON-008 |
| **Category** | Infrastructure |
| **Severity** | **Soft** (bisa dinegosiasi lewat perjanjian kanal) |
| **Source** | SH-007 Bank Partner (jawaban aspek D) |

**Constraint:** Bank yang diambil manual hanya menyimpan mutasi 30 hari di
portal.

**Impact:** Bila telat diunduh, data hilang → perlu pengingat otomatis; juga
memengaruhi ketersediaan data historis (ASM-004).

**Workaround:** Pengingat otomatis sebelum hari ke-30 + pengajuan perubahan kanal
ke bank (negosiasi).

**Related Requirements:** FR-001, FR-002

---

### CON-009: Dokumen desain tanpa keterikatan stack

| Field | Value |
|-------|-------|
| **ID** | CON-009 |
| **Category** | Process |
| **Severity** | Hard |
| **Source** | User (brief desain program) |

**Constraint:** Seluruh dokumen desain tidak boleh menyebut bahasa
pemrograman, framework, atau engine basis data tertentu; fokus pada logika,
struktur data, dan alur proses. Tanpa kode program.

**Impact:** Struktur data ditulis sebagai entitas + tipe logis; proses sebagai
alur berlabel; developer mana pun bisa mengimplementasikan.

**Workaround:** Tidak ada — batas format dokumen.

**Related Requirements:** Seluruh FR & NFR (dokumen)

---

## Assumption vs Constraint Summary

| ID | Type | Statement | Status | Impact |
|----|------|-----------|--------|--------|
| ASM-001 | Assumption | Volume 18.000 baris/hari, puncak 3× | Open | High |
| ASM-002 | Assumption | Selisih ±2.000 mayoritas non-matching | Open | High |
| ASM-003 | Assumption | 1 hari = 1 run; ulang mempertahankan koreksi | Verified | High |
| ASM-004 | Assumption | Data historis 3 bulan tersedia | Open | Medium |
| ASM-005 | Assumption | Tidak ada kiriman kedua per bank/hari | Open | Medium |
| ASM-006 | Assumption | IDR saja, tanpa FX | Verified | Medium |
| CON-001 | Constraint | Retensi 10 tahun, jejak tak-terubah | Hard | High |
| CON-002 | Constraint | Cut-off 06:00, selesai 07:30 | Hard | High |
| CON-003 | Constraint | Kendali ganda + eskalasi >Rp 5 juta | Hard | High |
| CON-004 | Constraint | IDR saja | Hard | Medium |
| CON-005 | Constraint | Toleransi maks Rp 1 | Hard | Medium |
| CON-006 | Constraint | Carry-over maks 0,5% (90 baris) | Hard | High |
| CON-007 | Constraint | UU PDP pada kolom keterangan | Hard | Medium |
| CON-008 | Constraint | Portal bank simpan 30 hari | Soft | Medium |
| CON-009 | Constraint | Dokumen tanpa stack / tanpa kode | Hard | Low |

## Risk Register (derived from assumptions)

| Risk | Source | Likelihood | Impact | Mitigation |
|------|--------|------------|--------|------------|
| Volume ternyata jauh lebih besar → jendela 30 menit gagal | ASM-001 | Medium | High | Ambil hitungan riil sebelum desain final; siapkan strategi pemenggalan proses |
| Selisih ternyata mayoritas nyata → target 20 menit mustahil | ASM-002 | Medium | High | Sampel 1 bulan (OQ-001) sebelum komitmen target ke stakeholder |
| Portal bank kehilangan data historis sebelum uji | ASM-004, CON-008 | High | Medium | Unduh & simpan segera pada minggu ke-1 |
| Kiriman kedua bank membuat run harian tidak memadai | ASM-005 | Low | High | Konfirmasi per bank saat interface analysis |
| Perubahan kebijakan retensi/pelaporan OJK | CON-001 | Low | High | Desain arsip terpisah dari logika bisnis agar mudah menyesuaikan |
| Bank partner tidak mau mengubah kanal 30 hari | CON-008 | Medium | Medium | Eskalasi ke perjanjian kanal; fallback pengingat otomatis |

## Review Schedule

| Review Date | Attendees | Purpose |
|-------------|-----------|---------|
| {akhir minggu ke-1} | SH-005, SH-003 | Verifikasi ASM-001, ASM-002 dari data riil |
| {pra-desain final} | SH-001, SH-005 | Tutup semua ASM berstatus Open |
| {pra go-live} | SH-004, SH-005 | Review constraint & kepatuhan |

## Sign-off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Business Owner (Manajer Keuangan) | {待isi} | {—} | ___________ |
| Technical Owner (System Analyst) | {待isi} | {—} | ___________ |
| PM | {待isi} | {—} | ___________ |
