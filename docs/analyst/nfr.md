# Non-Functional Requirements: Bank Reconciliation & Statement Mapping System

Dokumen NFR mandiri karena sistem ini menangani data keuangan terikat ketentuan
pengawasan — ketersediaan, keamanan, dan kepatuhan tidak boleh terkubur jadi
satu baris di SRS. Setiap NFR ditulis sebagai **skenario QA 6 bagian**
(Source · Stimulus · Environment · Artifact · Response · Response Measure) agar
punya angka ambang dan bisa langsung diuji.

## Document Info

| Field | Value |
|-------|-------|
| **Version** | 1.0 |
| **Last Updated** | 26 September 2026 |
| **Owner** | System Analyst (disepakati dengan Manajer Keuangan) |
| **Approval Status** | Draft — menunggu sign-off |
| **Sumber nilai** | `question-framework.md` aspek E + gate G-06 + `00-Global/SRS-MASTER.md` §1.4 |

## NFR Summary

| Category | Total | Critical | High | Medium | Low |
|----------|-------|----------|------|--------|-----|
| Performance | 4 | 3 | 1 | 0 | 0 |
| Availability | 2 | 2 | 0 | 0 | 0 |
| Security | 4 | 3 | 1 | 0 | 0 |
| Compliance | 2 | 2 | 0 | 0 | 0 |
| Data Management | 3 | 2 | 1 | 0 | 0 |
| Scalability | 1 | 0 | 1 | 0 | 0 |
| Usability | 2 | 0 | 1 | 1 | 0 |
| Maintainability | 2 | 0 | 2 | 0 | 0 |
| **TOTAL** | **20** | **12** | **7** | **1** | **0** |

> **Klasifikasi runtime vs development-time** (SAIP Bab 3.2): runtime =
> Performance, Availability, Security, Compliance, Data Management (sebagian),
> Usability · development-time = Maintainability, Scalability (diuji saat
> pengembangan/scale test), Data Management (backup drill).

---

## 1. PERFORMANCE

### NFR-PERF-001: Proses rekonsiliasi harian selesai ≤30 menit

| Field | Value |
|-------|-------|
| **ID** | NFR-PERF-001 |
| **Priority** | Critical |
| **Status** | [x] Confirmed |
| **Measurement** | Uji beban dengan volume riil 18.000 baris + 16.000 transaksi, diulang ≥5 kali |

**Requirement (QA 6 bagian):** Ketika penjadwal (**source**) menyalakan proses
rekonsiliasi harian pada 06:00 WIB (**stimulus**) dalam kondisi keempat bank
sudah mengirim statement dan volume normal 18.000 baris (**environment**),
sistem (**artifact**) membaca, memvalidasi, menormalisasi, dan mencocokkan seluruh
baris (**response**) **selesai ≤30 menit, 0 baris gagal proses, hasil akhir
tersedia sebelum 06:30 WIB** (**response measure**).

**Metric:**

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| Durasi proses penuh | ≤30 menit (selesai ≤06:30) | Stempel waktu `started_at` → `finished_at` |
| Baris gagal proses | 0 dari 18.000 | Pencocokan `record_count` vs baris terproses |
| Waktu per 1.000 baris | ≤1,7 menit | Durasi ÷ (baris ÷ 1.000) |

**Context:**
- **Peak Load:** 05:00–06:00 semua bank kirim bersamaan; hari tutup buku bulanan (H+1) 3× lipat → lihat NFR-PERF-002
- **Baseline:** manual 3,5 jam (06:30–10:00)
- **Trade-offs:** laporan analitik boleh lebih lambat; **jendela cut-off tidak boleh meleset**

**Verification:**
- [ ] Uji beban dengan data 18.000 baris, ≥5 pengulangan
- [ ] Benchmark durasi didokumentasikan
- [ ] Pemantauan durasi run dikonfigurasi

---

### NFR-PERF-002: Volume tutup buku bulanan (3×) selesai sebelum 07:00

| Field | Value |
|-------|-------|
| **ID** | NFR-PERF-002 |
| **Priority** | Critical |
| **Status** | [x] Confirmed |
| **Measurement** | Uji beban pada simulasi hari H+1 (±54.000 baris) |

**Requirement (QA 6 bagian):** Ketika penjadwal (**source**) menjalankan proses
pada hari tutup buku (**stimulus**) dengan volume 3× lipat ±54.000 baris
(**environment**), sistem (**artifact**) menyelesaikan rekonsiliasi
(**response**) **selesai sebelum 07:00 WIB tanpa penurunan kualitas pencocokan
(persen terpasangkan otomatis ≥97%)** (**response measure**).

**Metric:**

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| Durasi proses volume 54.000 baris | ≤60 menit | Stempel waktu run |
| % terpasangkan otomatis | ≥97% | Statistik hasil pencocokan per tier |

**Context:**
- **Peak Load:** tanggal 1 bulan berikutnya
- **Baseline:** belum ada — seluruh proses masih manual dan paling sering molor
- **Trade-offs:** bila meleset, eskalasi KRITIS otomatis terkirim (FR-017)

**Verification:**
- [ ] Uji beban simulasi H+1 ≥3 pengulangan
- [ ] Benchmark durasi didokumentasikan

---

### NFR-PERF-003: Layar penanganan selisih merespons <2 detik

| Field | Value |
|-------|-------|
| **ID** | NFR-PERF-003 |
| **Priority** | Critical |
| **Status** | [ ] Pending |
| **Measurement** | Pengukuran waktu respons p95 pada 5 pengguna bersamaan |

**Requirement (QA 6 bagian):** Ketika 5 pengguna (**source**) membuka daftar
selisih dan menyelesaikan satu baris (**stimulus**) pada puncak kerja jam 07:00
WIB (**environment**), layar penanganan selisih (**artifact**) menampilkan
hasil dan menyimpan tindakan (**response**) **dalam <2 detik (p95) untuk
pembukaan daftar dan <1 detik untuk penyimpanan satu tindakan**
(**response measure**).

**Metric:**

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| Waktu buka daftar selisih (p95) | <2 detik | Pengukuran sisi klien, 5 pengguna |
| Waktu simpan satu tindakan | <1 detik | Pengukuran sisi klien |

**Context:**
- **Peak Load:** 07:00, 3 analis + Supervisor + SA bersamaan
- **Baseline:** belum ada sistem — pembanding = buka/menyaring berkas besar di lembar kerja
- **Trade-offs:** laporan berat boleh diproses di luar jam sibuk

**Verification:**
- [ ] Pengukuran waktu respons p95 dilakukan
- [ ] Benchmark didokumentasikan
- [ ] Pemantauan waktu respons dikonfigurasi

---

### NFR-PERF-004: Ekspor laporan ≤2 menit untuk 18.000 baris

| Field | Value |
|-------|-------|
| **ID** | NFR-PERF-004 |
| **Priority** | High |
| **Status** | [ ] Pending |
| **Measurement** | Pengukuran waktu ekspor ringkasan + register selisih + jejak audit |

**Requirement (QA 6 bagian):** Ketika Auditor (**source**) meminta unduhan
laporan harian dan jejak audit (**stimulus**) di jam kerja normal
(**environment**), fasilitas ekspor (**artifact**) menyiapkan berkas
(**response**) **dalam ≤2 menit untuk rentang satu bulan (±390.000 baris), dan
berkas berisi baris yang sama dengan tampilan layar (tidak ada yang terpotong)**
(**response measure**).

**Context:**
- **Peak Load:** awal bulan saat Auditor mengambil laporan bulanan
- **Trade-offs:** ekspor besar boleh berjalan sebagai pekerjaan latar tanpa memblokir layar

**Verification:**
- [ ] Pengukuran waktu ekspor dilakukan (1 bulan penuh)
- [ ] Pemeriksaan kelengkapan isi berkas ekspor

---

## 2. AVAILABILITY

### NFR-AVAIL-001: Jendela kritis 05:00–07:30 WIB wajib tersedia

| Field | Value |
|-------|-------|
| **ID** | NFR-AVAIL-001 |
| **Priority** | Critical |
| **Status** | [x] Confirmed |
| **Measurement** | Pemantauan berkelanjutan + uji pemulihan |

**Requirement (QA 6 bagian):** Ketika komponen layanan (**source**) mengalami
gagal (**stimulus**) di luar jendela kritis, lalu layanan diperlukan pada
jendela 05:00–07:30 WIB (**environment**), layanan rekonsiliasi (**artifact**)
tetap dapat dijalankan (**response**) **ketersediaan ≥99,9% bulanan pada jendela
kritis (maks 26 menit tercatat per bulan), tanpa pemeliharaan terjadwal di
rentang 04:30–08:00 WIB** (**response measure**).

**Metric:**

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| Ketersediaan jendela kritis | ≥99,9% bulanan | Pemantauan berkelanjutan, laporan bulanan |
| Pemeliharaan terjadwal | Di luar 04:30–08:00 WIB | Jadwal operasi |

**Context:**
- **Peak Load:** hari kerja termasuk hari tutup buku
- **Trade-offs:** pemeliharaan di luar jam operasional diperbolehkan

**Verification:**
- [ ] Uji pemulihan dilakukan
- [ ] Pemantauan SLA dikonfigurasi

---

### NFR-AVAIL-002: Pemulihan dalam 60 menit, tanpa kehilangan data mutasi

| Field | Value |
|-------|-------|
| **ID** | NFR-AVAIL-002 |
| **Priority** | Critical |
| **Status** | [x] Confirmed |
| **Measurement** | Uji pemulihan tersendiri (drill), bukan hanya klaim |

**Requirement (QA 6 bagian):** Ketika kegagalan (**source**) membuat layanan
tidak dapat diakses (**stimulus**) pada pukul 05:30 WIB (**environment**), tim
operasional dan sistem (**artifact**) memulihkan layanan (**response**) **normal
kembali sebelum 06:30 WIB (RTO ≤60 menit), tanpa kehilangan satu pun baris
mutasi yang sudah diterima (RPO = 0 — file dapat diambil ulang dari bank), dan
bila melewati 06:30 sistem otomatis mengaktifkan jalur proses manual**
(**response measure**).

**Metric:**

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| RTO (waktu pulih) | ≤60 menit | Kronologi insiden |
| RPO (kehilangan data) | 0 baris mutasi | Pencocokan file yang diterima vs tersimpan |
| Pemicu jalur manual | Otomatis >06:30 WIB | Log peralihan mode |

**Context:**
- **Baseline:** kegagalan sekarang = seluruh proses manual tanpa jalur alternatif terdokumentasi
- **Trade-offs:** bila pemulihan gagal, rekonsiliasi hari itu jalan manual (bukan tertunda ke besok)

**Verification:**
- [ ] Uji pemulihan (drill) dilakukan dan didokumentasikan
- [ ] Pengujian cadangan/pulih data
- [ ] Pemicu jalur manual diuji

---

## 3. SECURITY

### NFR-SEC-001: Autentikasi dua langkah & kunci setelah 5 kali gagal

| Field | Value |
|-------|-------|
| **ID** | NFR-SEC-001 |
| **Priority** | Critical |
| **Status** | [x] Confirmed |
| **Compliance** | Ketentuan keamanan akun internal |

**Requirement (QA 6 bagian):** Ketika penyerang atau pengguna (**source**)
mencoba masuk berulang dengan kata sandi salah (**stimulus**) pada lingkungan
produksi (**environment**), layanan masuk (**artifact**) menghentikan percobaan
(**response**) **dua langkah wajib untuk semua peran, akun terkunci otomatis
setelah 5 kali gagal berturut-turut, kunci dilepas hanya lewat proses
pemulihan berotorisasi, dan setiap percobaan tercatat sebagai jejak audit**
(**response measure**).

**Details:**

| Aspect | Specification |
|--------|---------------|
| Dua langkah | Wajib untuk semua peran (`two_factor_flag = true`) |
| Batas gagal | 5 kali → terkunci (`fail_count ≥ 5`) |
| Pemulihan kunci | Hanya oleh Supervisor, tercatat di jejak audit |
| Pelaporan | Setiap percobaan gagal tercatat |

**Verification:**
- [ ] Pengujian keamanan (uji kunci akun & bypass)
- [ ] Pemeriksaan jejak login

---

### NFR-SEC-002: Kendali akses per peran (tanpa hapus baris mentah)

| Field | Value |
|-------|-------|
| **ID** | NFR-SEC-002 |
| **Priority** | Critical |
| **Status** | [x] Confirmed |
| **Compliance** | Prinsip hak minimum (UU PDP) |

**Requirement (QA 6 bagian):** Ketika pengguna (**source**) mencoba melakukan
tindakan di luar wewenang perannya (**stimulus**) di lingkungan produksi
(**environment**), lapisan akses (**artifact**) menolak tindakan
(**response**) **Analis tidak dapat menghapus/mengubah baris mutasi mentah;
Supervisor tidak dapat menyetujui koreksi yang diajukan sendiri; Auditor
bersifat baca-saja termasuk unduhan; System Analyst dapat mengubah aturan tetapi
tidak dapat menyetujui tutup hari; setiap penolakan tercatat** (**response
measure**).

**Details:**

| Aspect | Specification |
|--------|---------------|
| Analis | Cocokkan manual, ajukan koreksi — **tanpa** hapus/ubah baris mentah |
| Supervisor | Setujui koreksi & tutup hari — **tidak** boleh menyetujui miliknya sendiri |
| Auditor | Baca-saja + unduh jejak audit |
| System Analyst | Ubah parameter aturan — **tidak** boleh tutup buku sendiri |

**Verification:**
- [ ] Pengujian keamanan per peran (matriks izin)
- [ ] Uji penolakan akses di luar peran

---

### NFR-SEC-003: Jejak audit tidak bisa diubah (append-only + segel rantai)

| Field | Value |
|-------|-------|
| **ID** | NFR-SEC-003 |
| **Priority** | Critical |
| **Status** | [x] Confirmed |
| **Compliance** | Ketentuan audit trail transaksi keuangan |

**Requirement (QA 6 bagian):** Ketika administrator atau pihak manapun
(**source**) mencoba mengubah atau menghapus catatan jejak audit (**stimulus**)
pada kondisi produksi termasuk akses istimewa (**environment**), penyimpanan
jejak (**artifact**) menolak perubahan (**response**) **0 baris jejak dapat
diubah/dihapus (hanya tambah), setiap baris membawa segel perhitungan dari baris
sebelumnya sehingga perusakan terdeteksi, deteksi perusakan dilaporkan dalam
<5 menit, jejak tersimpan ≥10 tahun** (**response measure**).

**Metric:**

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| Upaya ubah/hapus jejak | 0 yang berhasil | Uji istimewa + laporan |
| Deteksi perusakan | <5 menit, laporan otomatis | Uji perusakan terkontrol |
| Retensi jejak | ≥10 tahun | Pemeriksaan arsip |

**Verification:**
- [ ] Pengujian upaya perubahan pada jejak (termasuk peran administrator)
- [ ] Uji keutuhan segel rantai
- [ ] Audit kepatuhan lulus

---

### NFR-SEC-004: Pelindungan data pribadi pada kolom keterangan

| Field | Value |
|-------|-------|
| **ID** | NFR-SEC-004 |
| **Priority** | High |
| **Status** | [ ] Pending |
| **Compliance** | UU PDP |

**Requirement (QA 6 bagian):** Ketika data pribadi (**source**) terbawa dalam
keterangan mutasi (**stimulus**) pada akses harian (**environment**), tampilan
dan ekspor (**artifact**) menyajikan data terlindungi (**response**) **nomor
identitas dan nomor rekening ditampilkan terpotong (pemotongan sebagian) untuk
peran tanpa kebutuhan penuh; akses nilai penuh hanya untuk Supervisor &
Auditor dan tercatat di jejak audit; data pribadi tidak masuk log operasional**
(**response measure**).

**Verification:**
- [ ] Pemeriksaan tampilan & ekspor per peran
- [ ] Pemeriksaan log operasional (tanpa data pribadi utuh)

---

## 4. COMPLIANCE & REGULATORY

### NFR-COMP-001: Retensi 10 tahun untuk baris mentah & jejak audit

| Field | Value |
|-------|-------|
| **ID** | NFR-COMP-001 |
| **Priority** | Critical |
| **Status** | [x] Confirmed |
| **Regulation** | Ketentuan pengawasan lembaga keuangan (OJK) — data transaksi keuangan |

**Requirement (QA 6 bagian):** Ketika pemeriksa (**source**) meminta bukti atas
periode 10 tahun terakhir (**stimulus**) pada audit kapan pun (**environment**),
arsip (**artifact**) menyediakan bukti (**response**) **seluruh baris mutasi
mentah, hasil pencocokan, selisih, koreksi, dan jejak audit dapat direkonstruksi
utuh untuk 10 tahun terakhir; penghapusan otomatis tidak pernah menyentuh rentang
<10 tahun; ketersediaan bukti per periode ≤1 hari kerja** (**response measure**).

**Details:**

| Aspect | Specification |
|--------|---------------|
| Regulation | Ketentuan pengawasan lembaga keuangan (OJK) — data transaksi keuangan |
| Rentang data | Baris mentah + hasil rekonsiliasi + jejak audit |
| Durasi retensi | 10 tahun |
| Eksepsi ringkasan | Ringkasan laporan cukup 2 tahun aktif (analisis), **bukan pengganti** data mentah |
| Evidence | Ekspor arsip + daftar pemulihan per periode |

**Verification:**
- [ ] Daftar kepatuhan ditandatangani
- [ ] Pengumpulan bukti diuji per periode
- [ ] Jejak audit terkonfigurasi & retensi terverifikasi

---

### NFR-COMP-002: Kepatuhan UU PDP atas data pribadi

| Field | Value |
|-------|-------|
| **ID** | NFR-COMP-002 |
| **Priority** | Critical |
| **Status** | [ ] Pending |
| **Regulation** | UU No. 27 Tahun 2022 tentang Pelindungan Data Pribadi |

**Requirement (QA 6 bagian):** Ketika subjek data (**source**) mengajukan hak
nya atau pemeriksa (**source**) memeriksa kepatuhan (**stimulus**) pada periode
pemeriksaan (**environment**), sistem (**artifact**) menunjukkan kepatuhan
(**response**) **data pribadi diproses sesuai tujuan (rekonsiliasi), akses
dibatasi per peran dan tercatat, pemroses disepakati lewat ketentuan internal,
dan permintaan subjek data dapat dilacak dalam ≤30 hari kerja**
(**response measure**).

**Details:**

| Aspect | Specification |
|--------|---------------|
| Regulation | UU PDP — data pribadi pada kolom keterangan mutasi |
| Pemrosesan | Hanya untuk tujuan rekonsiliasi |
| Akses | Terbatas per peran + tercatat di jejak audit |
| Bukti | Peta data pribadi per entitas + log akses |
| Audit Frequency | Sekali setahun + saat pemeriksaan |

**Verification:**
- [ ] Daftar kepatuhan UU PDP diselesaikan
- [ ] Peta data pribadi terdokumentasi
- [ ] Log akses dikonfigurasi

---

## 5. DATA MANAGEMENT

### NFR-DATA-001: Idempoten — run ulang & muat ulang tidak menggandakan data

| Field | Value |
|-------|-------|
| **ID** | NFR-DATA-001 |
| **Priority** | Critical |
| **Status** | [x] Confirmed (gate G-06) |
| **Measurement** | Uji pengulangan proses dengan data yang sama |

**Requirement (QA 6 bagian):** Ketika Supervisor (**source**) memutuskan
menjalankan ulang proses hari yang sama (**stimulus**) setelah ada koreksi
manual (**environment**), proses (**artifact**) menghasilkan data konsisten
(**response**) **jalur ulang tidak menghasilkan baris ganda (1 hari = 1 run),
semua tindakan pencocokan manual dan koreksi yang sudah ada DIPERTAHANKAN (bukan
direset), hasil akhir identik dengan hasil sebelumnya bila tidak ada perubahan
data** (**response measure**).

**Context:**
- **Baseline:** gate G-06 dikonfirmasi — "jika diulang, koreksi manual dipertahankan, bukan direset"
- **Trade-offs:** run ulang boleh memakan waktu seperti proses awal

**Verification:**
- [ ] Uji pengulangan proses ≥3 kali dengan koreksi manual di dalamnya
- [ ] Pemeriksaan jumlah baris (tanpa ganda)

---

### NFR-DATA-002: Baris mutasi mentah tidak boleh diubah atau dihapus

| Field | Value |
|-------|-------|
| **ID** | NFR-DATA-002 |
| **Priority** | Critical |
| **Status** | [x] Confirmed |
| **Measurement** | Pengujian larangan tulis pada baris mentah per peran |

**Requirement (QA 6 bagian):** Ketika pengguna (**source**) mencoba mengubah atau
menghapus baris mutasi mentah (**stimulus**) pada kondisi apa pun termasuk
proses koreksi (**environment**), penyimpanan data (**artifact**) menolak
(**response**) **0 baris mentah yang dapat diubah/dihapus oleh peran manapun;
koreksi selalu berupa transaksi baru; seluruh isi berkas asli tetap terbaca
utuh untuk 10 tahun** (**response measure**).

**Verification:**
- [ ] Pengujian larangan tulis per peran
- [ ] Pemeriksaan ulang berkas asli setelah periode uji

---

### NFR-DATA-003: Cadangan & pengujian pemulihan berkala

| Field | Value |
|-------|-------|
| **ID** | NFR-DATA-003 |
| **Priority** | High |
| **Status** | [ ] Pending |
| **Measurement** | Jadwal cadangan + uji pemulihan terjadwal |

**Requirement (QA 6 bagian):** Ketika kehilangan data (**source**) terjadi
(**stimulus**) pada operasi harian (**environment**), penyimpanan cadangan
(**artifact**) mengembalikan data (**response**) **cadangan harian, retensi
cadangan 10 tahun sejajar dengan ketentuan retensi, uji pemulihan dilakukan
setiap kuartal dengan hasil terdokumentasi, dan pemulihan memenuhi RPO 0 baris
mutasi** (**response measure**).

**Verification:**
- [ ] Pengujian cadangan/pemulihan (kuartalan)
- [ ] Dokumen hasil uji pemulihan

---

## 6. SCALABILITY

### NFR-SCALE-001: Kapasitas tiga tahun tanpa perubahan desain

| Field | Value |
|-------|-------|
| **ID** | NFR-SCALE-001 |
| **Priority** | High |
| **Status** | [ ] Pending |
| **Measurement** | Uji beban bertingkat + proyeksi pertumbuhan |

**Requirement (QA 6 bagian):** Ketika volume (**source**) tumbuh seiring waktu
(**stimulus**) pada operasi harian (**environment**), sistem (**artifact**)
tetap memenuhi seluruh NFR kinerja (**response**) **menangani pertumbuhan pada
tabel di bawah tanpa perubahan desain, dengan NFR-PERF-001 dan NFR-PERF-002
tetap terpenuhi** (**response measure**).

**Details:**

| Aspect | Current (Y0) | Target (Y1) | Target (Y3) |
|--------|--------------|-------------|-------------|
| Baris mutasi/hari | 18.000 | 30.000 | 60.000 |
| Bank partner | 4 | 8 | 15 |
| Transaksi ledger/hari | 16.000 | 30.000 | 60.000 |
| Pengguna bersamaan | 5 | 12 | 30 |
| Volume puncak (H+1) | 54.000 | 90.000 | 180.000 |
| Retensi data | 10 tahun | 10 tahun | 10 tahun |

**Verification:**
- [ ] Uji beban pada skala target Y1 (30.000 baris) dan proyeksi Y3
- [ ] Proyeksi pertumbuhan data terdokumentasi (termasuk pemenggalan/partisi bila perlu)

---

## 7. USABILITY

### NFR-USE-001: Analis baru produktif setelah pelatihan 2 hari

| Field | Value |
|-------|-------|
| **ID** | NFR-USE-001 |
| **Priority** | Medium |
| **Status** | [x] Confirmed (rencana transisi) |
| **Measurement** | Pengamatan terarah pasca-pelatihan |

**Requirement (QA 6 bagian):** Ketika analis baru (**source**) mulai
menyelesaikan selisih setelah pelatihan (**stimulus**) pada hari kerja normal
(**environment**), layar penanganan selisih (**artifact**) mendukung kerja
(**response**) **3 analis menyelesaikan pelatihan 2 hari; setelah itu mampu
menyelesaikan 10 selisih pertama tanpa pendampingan, dan waktu penyelesaian per
selisih ≤60 detik** (**response measure**).

**Metric:**

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| Durasi pelatihan | 2 hari untuk 3 analis | Jadwal pelatihan |
| Selisih pertama tanpa pendampingan | 10 baris | Pengamatan terarah |
| Waktu per selisih | ≤60 detik | Pengamatan + log tindakan |

**Verification:**
- [ ] Sesi pelatihan terjadwal & terdokumentasi
- [ ] Pengamatan produktivitas pasca-pelatihan

---

### NFR-USE-002: Umpan balik jelas untuk pilihan pencocokan

| Field | Value |
|-------|-------|
| **ID** | NFR-USE-002 |
| **Priority** | Low |
| **Status** | [ ] Pending |
| **Measurement** | Pengujian pemakaian dengan 3 analis |

**Requirement (QA 6 bagian):** Ketika analis (**source**) memilih di antara
kandidat pasangan (**stimulus**) pada penanganan selisih (**environment**),
layar (**artifact**) menjelaskan alasan sistem (**response**) **setiap kandidat
menampilkan aturan yang dipakai, skor keyakinan, dan titik perbedaan (nominal,
tanggal, referensi) sehingga analis memahami alasan tanpa bertanya; jumlah
kesalahan pilihan <1 dari 20 tindakan** (**response measure**).

**Verification:**
- [ ] Pengujian pemakaian dengan 3 analis
- [ ] Pengukuran tingkat kesalahan pilihan

---

## 8. MAINTAINABILITY

### NFR-MAINT-001: Perubahan aturan tanpa menyentuh logika program

| Field | Value |
|-------|-------|
| **ID** | NFR-MAINT-001 |
| **Priority** | High |
| **Status** | [ ] Pending |
| **Measurement** | Uji perubahan aturan end-to-end |

**Requirement (QA 6 bagian):** Ketika Supervisor & System Analyst
(**source**) mengubah parameter aturan (mis. toleransi tanggal 1→3 hari)
(**stimulus**) pada jam kerja (**environment**), pengaturan aturan
(**artifact**) menerapkan perubahan (**response**) **perubahan selesai diuji
dan berlaku untuk run berikutnya pada hari yang sama, tanpa perubahan logika
program, dengan persetujuan tercatat; hasil run lampau tetap dapat dijelaskan
memakai versi aturan yang berlaku saat itu** (**response measure**).

**Metric:**

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| Waktu berlaku perubahan | Run berikutnya (hari yang sama) | Stempel waktu versi aturan |
| Perubahan logika program | 0 | Pemeriksaan rilis |
| Jejak persetujuan | 100% perubahan punya persetujuan | Audit jejak |

**Verification:**
- [ ] Pengujian perubahan aturan end-to-end (termasuk penolakan tanpa persetujuan)
- [ ] Pemeriksaan jejak persetujuan

---

### NFR-MAINT-002: Pemantauan proses harian & peringatan otomatis

| Field | Value |
|-------|-------|
| **ID** | NFR-MAINT-002 |
| **Priority** | High |
| **Status** | [x] Confirmed |
| **Measurement** | Pemeriksaan log, status run, dan pengiriman peringatan |

**Requirement (QA 6 bagian):** Ketika proses (**source**) berjalan, terlambat,
atau gagal (**stimulus**) pada jendela kritis (**environment**), pemantauan
(**artifact**) memberi tahu pihak berwenang (**response**) **setiap run
memiliki status terlihat (antre, berjalan, menunggu review, gagal, ditutup);
peringatan WASPADA >90 selisih dan KRITIS >200 selisih terkirim pada 06:30 WIB
tepat; keterlambatan bank >05:00 terlaporkan; peringatan terkirim ulang bila
tidak direspons ≤15 menit** (**response measure**).

**Verification:**
- [ ] Pemeriksaan status run & log
- [ ] Pengujian pengiriman peringatan (WASPADA/KRITIS/terlambat)

---

## Traceability Matrix

| NFR ID | Category | Related FR | Verification Method | Status |
|--------|----------|-----------|---------------------|--------|
| NFR-PERF-001 | Performance | FR-001, FR-005, FR-006 | Uji beban 18.000 baris | Confirmed |
| NFR-PERF-002 | Performance | FR-005, FR-017 | Uji beban H+1 | Confirmed |
| NFR-PERF-003 | Performance | FR-011, FR-012 | Pengukuran p95, 5 pengguna | Pending |
| NFR-PERF-004 | Performance | FR-015, FR-018 | Pengukuran waktu ekspor | Pending |
| NFR-AVAIL-001 | Availability | FR-001, FR-002 | Pemantauan + jadwal | Confirmed |
| NFR-AVAIL-002 | Availability | FR-002, FR-017 | Uji pemulihan (drill) | Confirmed |
| NFR-SEC-001 | Security | FR-012–FR-014, FR-022 | Uji kunci akun & bypass | Confirmed |
| NFR-SEC-002 | Security | FR-012, FR-013, FR-019, FR-021 | Uji matriks izin per peran | Confirmed |
| NFR-SEC-003 | Security | FR-022, FR-023 | Uji perubahan jejak + segel rantai | Confirmed |
| NFR-SEC-004 | Security | FR-018 | Pemeriksaan tampilan/ekspor | Pending |
| NFR-COMP-001 | Compliance | FR-015, FR-018, FR-022 | Daftar kepatuhan + uji arsip | Confirmed |
| NFR-COMP-002 | Compliance | FR-014, FR-022 | Daftar kepatuhan UU PDP | Pending |
| NFR-DATA-001 | Data Mgmt | FR-005, FR-014 | Uji run ulang ≥3× | Confirmed |
| NFR-DATA-002 | Data Mgmt | FR-012, FR-014 | Uji larangan tulis baris mentah | Confirmed |
| NFR-DATA-003 | Data Mgmt | — | Uji cadangan/pemulihan kuartalan | Pending |
| NFR-SCALE-001 | Scalability | FR-005, FR-006 | Uji beban bertingkat | Pending |
| NFR-USE-001 | Usability | FR-011, FR-012 | Pengamatan pasca-pelatihan | Confirmed |
| NFR-USE-002 | Usability | FR-008, FR-012 | Pengujian pemakaian | Pending |
| NFR-MAINT-001 | Maintainability | FR-019, FR-020 | Uji perubahan aturan E2E | Pending |
| NFR-MAINT-002 | Maintainability | FR-017, FR-002 | Pemeriksaan log + uji peringatan | Confirmed |

## NFR Sign-off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| System Architect | {待定} | {—} | ___________ |
| Security Officer | {待定} | {—} | ___________ |
| Business Owner (Manajer Keuangan) | {待定} | {—} | ___________ |
| Technical Lead | {待定} | {—} | ___________ |

> NFR kritis (ketersediaan, keamanan, kepatuhan) **wajib sign-off terpisah**
> dari SRS.

## Change Log

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 26 Sep 2026 | System Analyst | Versi awal — 20 NFR dari jawaban question-framework aspek E + gate |
