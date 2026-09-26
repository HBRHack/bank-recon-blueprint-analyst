# Jawaban: Question Framework — Bank Reconciliation & Statement Mapping System

> **Klien:** PayNusa (fintech fiktif) — Tim Finance/Operations, System Analyst, Auditor
> **Status:** Semua pertanyaan A–F dijawab, semua gate G berstatus `[setuju]`.
> **Langkah berikutnya:** `00-Global/NOTATION.md` → `ERD-MASTER.md` → `DESAIN-PROGRAM.md`

---

## A. Bisnis

| Pertanyaan | Jawaban |
|---|---|
| Lama proses manual sekarang | 2 analis, 06:30–10:00 WIB, sering molor ke jam 11 kalau statement bank telat |
| Penyebab selisih tersering | (1) biaya admin dipotong di baris terpisah (2) tanggal value beda H+1 dari ledger (3) transaksi QRIS tanpa referensi jelas |
| Volume & tingkat match manual | ±4.500 baris/hari/rekening (total ±18.000/hari, 4 bank), ~90% cocok otomatis via Excel filter, 10% (~1.800 baris) ditelusuri manual |
| Dampak selisih telat terdeteksi | Settlement merchant bisa "mengambang" Rp 1,2–2 M; divisi klaim komplain ke Finance jika >1 hari |
| Cut-off harian | Statement final 06:00 WIB, rekonsiliasi wajib selesai 07:30, ditandatangani Supervisor Keuangan |
| Approval selisih | >Rp 5 juta wajib naik ke Manajer Keuangan; di bawah itu cukup Supervisor |
| Kegagalan di tengah proses | Lanjut pakai bank yang datanya lengkap; bank gagal ditandai "pending", tidak menunggu semua bank |
| Definisi "selesai" per hari | Maksimal 0,5% baris boleh carry-over ke besok, sisanya wajib matched/resolved |
| PALM (goal bisnis relevan) | Finansial: cegah potensi rugi ~50jt/bulan · Karyawan: beban kerja analis 3,5 jam → target 20 menit · Proses bisnis: ini goal utama project · Reputasi: cegah churn merchant akibat klaim telat resolve |

## B. Data

| Pertanyaan | Jawaban |
|---|---|
| Data mutasi bank per baris | Tanggal value, tanggal transaksi, no. referensi bank, keterangan, debit/kredit, saldo berjalan, kode bank |
| Data ledger internal | No. transaksi sistem, jenis (settlement/fee/refund/adjustment), rekening tujuan, nominal, tanggal proses, status |
| Kolom andalan matching | No. referensi bank (utama); fallback nominal + tanggal + kode merchant kalau referensi kosong (kasus QRIS) |
| Keunikan kolom | 1 no. referensi hanya boleh muncul sekali per hari per rekening; kembar → otomatis exception (dicurigai duplikat) |
| Aturan validasi isian | Tanggal value tidak boleh lebih baru dari tanggal statement dikirim; mata uang wajib IDR |
| Variasi format antar bank | Bank A: `TRF-{angka}` · Bank B: angka polos · Bank C: sebagian transaksi tanpa referensi sama sekali |
| Toleransi selisih (G-01) | Maks Rp 1 (pembulatan) dianggap cocok |
| Retensi data | Baris mentah + audit trail 10 tahun (OJK); ringkasan laporan 2 tahun aktif |
| Transaksi dibatalkan/reversed | Tetap dicocokkan — reversal bank harus ketemu pasangan reversal ledger, bukan dianggap hilang |

## C. Pengguna

| Pertanyaan | Jawaban |
|---|---|
| Aktor sistem | 3 Finance Ops Analyst · 1 Supervisor Keuangan · 1 Auditor Internal · 1 System Analyst |
| Hak akses per peran | Analis: tandai match manual, TIDAK boleh hapus baris mentah · Supervisor: approve koreksi & tutup hari · Auditor: read-only + unduh audit trail · SA: atur parameter toleransi, TIDAK bisa approve tutup buku sendiri |
| Rangkap peran | Boleh (misal Supervisor jadi analis saat cuti), log tetap catat "acting as" |
| Akses per rekening | Ya — analis A pegang rekening operasional, analis B pegang settlement |
| Otoritas keputusan akhir | Supervisor Keuangan (final), eskalasi ke Manajer Keuangan hanya untuk selisih >Rp 5 juta |
| Role sweep relevan (BABOK) | Analis bisnis (SA internal) · pengguna langsung (Finance Ops) · ahli domain (Supervisor) · sponsor (Manajer Keuangan) · regulator (OJK, pasif) · vendor (4 bank partner) · customer (merchant, dampak tidak langsung) |

## D. Sambungan ke Sistem Lain

| Pertanyaan | Jawaban |
|---|---|
| Sumber file mutasi bank | 3 dari 4 bank: otomatis via SFTP jam 05:00 · 1 bank (terkecil): manual unduh dari portal |
| Volume per bank | 1.000–6.000 baris/hari per bank, total ±18.000/hari |
| Sumber data ledger internal | Sistem payment internal, laporan harian ±16.000 transaksi settlement |
| Kegagalan sambungan | Proses jalan parsial dengan bank yang sudah masuk; bank belum masuk ditandai pending |
| Interface tersembunyi wajib ada | Notifikasi selisih (email/Slack ke Supervisor jika >200 baris unmatched jam 06:30) · Ekspor laporan ke Auditor (PDF/CSV on-demand) · Kirim balik koreksi ke sistem ledger (jurnal koreksi otomatis) |
| Aturan mengikat dari bank partner | Portal Bank D (manual) hanya simpan data 30 hari — risiko kehilangan data jika telat unduh, perlu reminder otomatis |

## E. Kebutuhan Teknis

| Pertanyaan | Jawaban |
|---|---|
| Total baris & batas waktu proses | ±18.000 baris/hari, wajib selesai proses otomatis dalam 30 menit (target 06:00–06:30) |
| Jam sibuk | 05:00–06:00, semua bank kirim bersamaan |
| Kondisi volume tinggi | Hari tutup buku bulanan (H+1): volume 3x lipat, tetap harus selesai sebelum 07:00 |
| Waktu pemulihan (recovery) | Down jam 05:30 → wajib normal sebelum 06:30, jika tidak fallback ke manual |
| Pengguna bersamaan | 5 (3 analis + Supervisor + SA), puncak jam 07:00 |
| Retensi & kepatuhan | 10 tahun (OJK) + tunduk UU PDP untuk data pribadi di kolom keterangan |
| Kebutuhan keamanan | 2FA wajib, lock setelah 5x gagal login, audit trail immutable (tidak bisa diubah siapa pun) |
| Masa transisi | Data historis 3 bulan untuk uji cocok · training 3 analis, 2 hari · paralel run 2 minggu (manual vs sistem baru) · rollback ke manual jika migrasi gagal |

## F. Kelompok Fitur

**Urutan prioritas rilis:**

| Rilis | Kelompok Fitur |
|---|---|
| Rilis 1 (core) | Penerimaan & Penyajian Mutasi Bank → Pencocokan Otomatis → Penanganan Selisih (Exception) |
| Rilis 2 | Koreksi & Persetujuan → Laporan & Ekspor → Pengaturan Aturan |

- **Pengaturan aturan:** hanya SA yang boleh ubah toleransi (misal ±1 hari → ±3 hari), perubahan wajib approval Supervisor
- **Laporan Auditor:** daftar selisih + siapa yang resolve + kapan (full trail)
- **Notifikasi:** threshold otomatis jam 06:30 jika masih >200 baris unmatched → alert ke Supervisor

---

## G. Gate — Status Asumsi (Final)

| # | Pernyataan | Status | Alasan |
|---|---|---|---|
| G-01 | Pencocokan berjenjang: referensi identik → nominal+tanggal ≤1 hari → agregasi → keterangan mirip → sisa = selisih | `[setuju]` | Ambang 1 hari karena bank sering proses value date H+1 dari waktu transaksi aktual |
| G-02 | Satu baris bank bisa wakili banyak transaksi internal (agregasi), dan sebaliknya (split/fee) | `[setuju]` | Paling sering dibutuhkan untuk settlement harian per merchant |
| G-03 | Baris yang sudah berpasangan tidak boleh dipakai lagi (anti dobel) | `[setuju]` | Referensi kembar karena error bank → tetap jadi exception, tidak dipaksa cocok |
| G-04 | Tutup hari butuh dua orang: Analis siapkan, Supervisor setujui | `[setuju]` | Dual-control mencegah selisih ditutup sepihak tanpa review |
| G-05 | Baris mentah + audit trail disimpan utuh, tidak bisa diubah siapa pun | `[setuju]` | 10 tahun sesuai ketentuan OJK untuk data transaksi keuangan |
| G-06 | Proses harian otomatis bisa diulang tanpa data ganda | `[setuju]` | Jika diulang, koreksi manual dipertahankan — bukan direset |

**Semua asumsi gate berstatus final.** Dokumen desain (`NOTATION.md`, `ERD-MASTER.md`, `DESAIN-PROGRAM.md`) siap digarap berdasarkan jawaban di atas.