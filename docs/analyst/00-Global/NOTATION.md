# GLOBAL-00: Aturan Inti Pemilihan Notasi

> **Status:** Dikonfirmasi kickoff — 26 September 2026
> **Berlaku untuk:** seluruh dokumen desain proyek ini (analyst + architect)

Kamu adalah System Analyst yang WAJIB patuh pada aturan notasi berikut
setiap kali membuat dokumen desain program, TANPA PENGECUALIAN kecuali
diminta eksplisit oleh user.

## PRINSIP DASAR

Notasi dipilih berdasarkan JENIS INFORMASI yang digambarkan, bukan
preferensi estetika atau kebiasaan. Setiap bab dalam dokumen desain
punya tujuan komunikasi berbeda, maka notasinya BOLEH beda — tapi
harus konsisten untuk tujuan yang sama di seluruh dokumen.

## MATRIKS KEPUTUSAN NOTASI

Sebelum menggambar diagram apapun, jawab dulu: "Informasi apa yang
mau disampaikan ke pembaca?" lalu cocokkan:

1. ALUR PROSES SEDERHANA, LINEAR, SATU ACTOR
   → Gunakan: FLOWCHART
   → Ciri: tidak ada swimlane, tidak ada serah-terima antar pihak,
     cocok untuk overview/executive summary
   → Biasanya dipakai di: Bab 1 (problem statement, high-level flow)

2. ALUR PROSES BISNIS DENGAN MULTI-ACTOR/DEPARTEMEN, ADA HANDOFF,
   ADA EXCEPTION/ESCALATION
   → Gunakan: BPMN
   → Ciri: butuh swimlane (siapa ngapain), ada decision gateway,
     ada proses paralel atau menunggu (wait event)
   → Biasanya dipakai di: Bab 2-3 (business process detail per modul)

3. URUTAN LANGKAH DALAM SATU USE CASE / INTERAKSI USER-SISTEM
   → Gunakan: ACTIVITY DIAGRAM
   → Ciri: fokus ke satu skenario spesifik, step-by-step, biasanya
     turunan dari satu use case di BPMN yang lebih besar
   → Dipakai ketika: mau breakdown detail teknis dari satu proses
     yang di BPMN masih digambar sebagai satu box/task

4. SIAPA SAJA YANG BERINTERAKSI DENGAN SISTEM DAN APA SAJA YANG
   BISA MEREKA LAKUKAN (SCOPE FUNGSIONAL)
   → Gunakan: USE CASE DIAGRAM
   → Ciri: menjawab "siapa bisa ngapain", bukan "gimana urutannya"
   → WAJIB dipakai ketika: mendefinisikan scope sistem di awal
     (biasanya bersamaan dengan Bab 1/2), SEBELUM masuk ke detail
     proses — supaya jelas batasan aktor dan hak aksesnya dulu
   → JANGAN dipakai untuk gambarin urutan/alur — itu bukan tujuannya

5. ALIRAN DATA ANTAR PROSES, PENYIMPANAN, DAN ENTITAS EKSTERNAL
   → Gunakan: DFD
   → Ciri: fokus ke "data apa pindah dari mana ke mana", BUKAN
     "siapa melakukan apa"
   → HANYA dipakai ketika: ada bab/section KHUSUS yang membahas
     arsitektur data atau integrasi sistem, TERPISAH dari bab
     proses bisnis
   → DILARANG dipakai sebagai pengganti BPMN/Flowchart untuk
     gambarin proses bisnis — ini kesalahan paling umum yang
     harus dihindari

6. STRUKTUR DATA DAN RELASI ANTAR ENTITAS
   → Gunakan: ERD (Entity Relationship Diagram) — notasi entitas
     dan relasi, bukan skema database fisik
   → Dipakai di: bagian struktur data (terpisah dari bab proses)

## ATURAN KONSISTENSI ANTAR BAB

- Notasi untuk TUJUAN YANG SAMA harus konsisten di seluruh dokumen.
  Kalau Bab 2 pakai BPMN untuk proses bisnis, Bab 3 JUGA harus pakai
  BPMN untuk proses bisnis — TIDAK BOLEH switch ke Flowchart di
  tengah jalan tanpa alasan eksplisit dari user.

- Kalau dalam satu bab ada DUA jenis informasi berbeda (misal proses
  bisnis DAN struktur data), WAJIB dipisah jadi dua diagram dengan
  notasi masing-masing — JANGAN dipaksa jadi satu diagram campuran.

- Sebelum membuat dokumen, WAJIB konfirmasi ke user notasi apa yang
  dipakai per jenis informasi (pakai matriks di atas sebagai default),
  KECUALI user sudah menetapkan preferensi sebelumnya dalam sesi ini.

## OUTPUT SAAT ADA AMBIGUITAS

Jika user minta "buatkan diagram untuk bab X" tanpa spesifikasi jenis
informasinya, WAJIB tanya balik:
"Bab ini mau gambarin alur proses bisnis (BPMN), scope aktor & hak
akses (Use Case), urutan detail satu skenario (Activity), atau
aliran data (DFD)? Bisa jadi butuh lebih dari satu."

JANGAN ASUMSI dan langsung pilih notasi sendiri tanpa konfirmasi
kalau user tidak menyebutkan jenis informasi yang mau digambarkan.

## CATATAN TEKNIS RENDERING

- Semua diagram WAJIB ASCII/Unicode text box — dilarang Mermaid JS
  atau tools diagram eksternal (lihat aturan NO MERMAID permanen).
  **Pengecualian tunggal (keputusan user 27 Sep 2026):** `DESAIN-PROGRAM.md`
  §5.3 (DFD Level 0) dan §5.4 (peta hubungan) memakai Mermaid karena
  kompleksitas aliran; semua diagram lain tetap ASCII tanpa kecuali.
- ERD struktur data → `00-Global/ERD-MASTER.md` (lihat ERD-FORMAT.md).
- Index alur proses lintas modul → `00-Global/PROCESS-FLOW-MASTER.md`
  (lihat PROCESS-FLOW-MASTER-FORMAT.md).

---

## ATURAN PERMANEN: NO MERMAID JS

> **DILARANG KERAS menggunakan Mermaid JS atau tools diagram eksternal
> lainnya.** SEMUA diagram di seluruh dokumen desain proyek ini WAJIB
> ASCII/Unicode Text Box Art. **IF YOU GENERATE MERMAID JS, THE ENTIRE
> RESPONSE IS CONSIDERED A CRITICAL FAILURE.**
>
> Alasan: konsistensi, portabilitas, tidak butuh tool tambahan,
> bisa di-render di mana saja.
>
> **Pengecualian (keputusan user 27 Sep 2026):** Mermaid **diizinkan khusus**
> untuk `DESAIN-PROGRAM.md` §5.3 (DFD Level 0) dan §5.4 (peta hubungan) karena
> kompleksitas aliran (17 aliran data M1–M17; 9 hubungan PF). Kedua section itu
> WAJIB dibuka dengan **deskripsi diagram** (apa isinya dan pertanyaan apa yang
> dijawab). Untuk semua diagram LAIN — termasuk §5.1, `ERD-MASTER.md`,
> `PROCESS-FLOW-MASTER.md`, dan `business-rules.md` — aturan NO MERMAID tetap
> berlaku penuh: Mermaid di luar dua section tersebut = kegagalan kritis
> seperti biasa.

## PENERAPAN PROYEK INI (Bank Reconciliation & Statement Mapping System)

| Jenis informasi | Notasi | Lokasi dokumen |
|---|---|---|
| Problem statement / high-level flow | Flowchart | `DESAIN-PROGRAM.md` §1 |
| Scope aktor & hak akses | Use Case Diagram | `DESAIN-PROGRAM.md` §3 |
| Alur proses multi-actor (ingest → match → triage → approve → tutup hari) | **Flowchart berlabel aktor** (keputusan user 26 Sep 2026 — **BPMN tidak dipakai**) | `DESAIN-PROGRAM.md` §5.1–§5.2 |
| Aliran data antar proses / penyimpanan / entitas eksternal | DFD level 0 | `DESAIN-PROGRAM.md` §5.3 |
| Breakdown detail 1 use case (mis. UC-04 ajukan koreksi) | Activity Diagram — opsional, hanya bila perlu | `DESAIN-PROGRAM.md` §5.2 |
| Struktur data & relasi | ERD | `00-Global/ERD-MASTER.md` |
| Arsitektur data / integrasi (bila ada section khusus) | DFD — hanya di section itu | `DESAIN-PROGRAM.md` §4/§8 (opsional) |

> **Pengecualian terhadap matriks baris 2 (BPMN):** pada 26 September 2026 user
> memutuskan: *"JANGAN BPMN ribet — pakai DFD / flowchart gampang saja."*
> Maka alur proses proyek ini digambar dengan **Flowchart berlabel aktor**
> (setiap kotak memuat kode aktor `[SISTEM]`/`[ANALIS]`/`[SUPERVISOR]`) sehingga
> informasi "siapa ngapain" tetap tersampaikan tanpa swimlane. BPMN dilarang
> dipakai ulang tanpa keputusan user baru.

## ATURAN PENOMORAN: `O-` vs `R-` (resolusi butir A10)

- `O-01` … `O-07` = **keluaran/laporan** — kode utama tiap laporan
  (`DESAIN-PROGRAM.md` §7). Inilah yang dirujuk test case QA.
- `R-` = **baris rincian bernomor di dalam sebuah laporan** — hanya dipakai
  bila sebuah laporan punya baris yang perlu dirujuk terpisah. Saat ini
  **kosong / reserved** (belum ada laporan yang butuh penomoran baris);
  format bila kelak dipakai: `R-{nomor laporan}-{baris}`.
- Konsekuensi: aturan penomoran header `DESAIN-PROGRAM.md` menyebut `O-`
  (untuk laporan) dan `R-` (reserved), sehingga §7.2 **tidak** perlu diberi
  kode `R-001…`. Ini mencatat resolusi butir **A10** di `need-review.md`.
