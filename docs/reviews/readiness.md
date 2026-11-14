# Readiness — 14 Nov 2026

**Verdict:** GO, dengan dua keputusan Thomas di gate ini (field registry `gate`/`threads`, dan kebijakan data publik). Keduanya sudah punya cabang default di PRD, jadi tidak membuat item gagal.

Dinilai terhadap [prd.md](../prd.md) setelah [plan review 1](plan-review-1.md) dan [plan review 2](plan-review-2.md).

| # | Item | Pass | Evidence |
| --- | --- | --- | --- |
| 1 | Setiap story punya AC yang bisa dites | Ya | 16 story, 58 AC Given/When/Then dengan hasil yang bisa diamati (teks halaman, field JSON, kode keluar, jumlah request di server palsu); konvensi e2e di §5. Setiap story punya minimal satu AC error/edge (mis. AC-01.3, AC-02.5, AC-03.2, AC-04.3, AC-06.2, AC-07.4, AC-09.3, AC-10.1) |
| 2 | Tidak ada TBD di scope MVP | Ya | `grep -i "TBD|TODO|nanti"` di prd.md: hanya cocok dengan kata "metodologi". Ketidakpastian spike M0 ditulis sebagai cabang keputusan di exit M0 (§9), bukan TBD |
| 3 | Metrik punya angka dan cara ukur | Ya | §2: K1–K9, masing-masing dengan target, jendela waktu dan cara ukur; K1/K2 punya versi dry run pra-launch |
| 4 | Risiko besar punya mitigasi | Ya | §8: 12 risiko; semua yang berdampak Tinggi (batas 90 hari, perubahan schema, kebocoran kredensial, kebocoran data, pengurangan agregat) punya mitigasi dan AC/test terkait |
| 5 | Dependency eksternal teridentifikasi | Ya | §7: repo factory, Amp API, Amp CLI, GitHub API, hosting, lingkungan kolektor, field registry, peserta uji, masing-masing dengan akses, biaya dan fallback |
| 6 | MVP muat budget waktu dan uang | Ya | §10: 19 hari agent untuk Must (≈ 3,8 minggu, dalam 2–4 minggu) termasuk cadangan ± 15%; target US$0/bulan, batas < US$20/bulan; Should di belakang garis potong §4 |
| 7 | Mode delivery jelas | Ya | Header PRD: showcase (situs statis publik + kolektor privat). Kategori 8, jadi hosting tidak wajib; deploy publik tetap di gate Phase 12 |
| 8 | Plan review selesai | Ya | plan-review-1.md (4 BLOCKER, semua diperbaiki) dan plan-review-2.md (3 BLOCKER, semua diperbaiki atau ditolak dengan bukti); tidak ada BLOCKER terbuka |

## Keputusan yang diminta di gate ini

1. **Field registry `gate: {phase, since}` dan `threads: []`**, ditulis skill factory-orchestrator. Bila disetujui: US-07 "Menunggu Thomas" Must dan biaya thread anak `create_thread` terhitung. Bila ditolak: US-07 turun ke Should, halaman menulis "Data gate belum tersedia", dan biaya thread anak tidak terhitung. Perubahan skill hanya berlaku setelah di-push ke User Skills (butuh persetujuanmu). Thread lain sedang menyinkronkan skills; perubahan ini perlu dikoordinasikan dengannya.
2. **Kebijakan data publik.** Default PRD: nama, fase dan biaya project public tampil; project private hanya satu baris kohort (n ≥ 3, tanpa fase, diperbarui harian); project `killed` tampil; kredit Amp dan estimasi list price tampil setara.

## Options for Thomas (from plan review)

Lihat [plan-review-1.md](plan-review-1.md#options-for-thomas): mode pemilik sebagai laporan CLI, akses pemilik dari HP, sumber status gate, field `threads`, angka biaya utama, apa yang boleh publik, frekuensi kolektor.

## Cost so far

US$0,17 kredit Amp; estimasi list price US$15,84 untuk inference lewat subscription Claude milikmu (tidak ditagih Amp). Dibaca dengan `amp threads usage --details` pada 14 Nov 2026 06.04 UTC. Waktu ± 30 menit sejak thread dimulai.
