# Plan review — round 2

14 Nov 2026. Round 2 diperlukan karena perbaikan round 1 mengubah scope (US-15, US-16, metrik K1–K9, alur deploy). Satu CRITIC baru dengan konteks segar, read-only, pada PRD setelah round 1 (commit `a783666`). DISCUSSANT tidak diulang karena usulan strategisnya sudah dibawa ke [Options for Thomas di round 1](plan-review-1.md#options-for-thomas). Ini ronde terakhir yang diizinkan skill plan-review.

| # | Source | Severity | Finding | Decision | Reason | Applied in |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | C1 | BLOCKER | Biaya/fase project private bisa dihitung dengan mengurangkan baris public dari total dan jumlah per fase, atau membandingkan snapshot berurutan | ACCEPT | Aritmetika sederhana; kebocoran nyata | AC-03.6 (total hanya dari project public, kohort private n ≥ 3 tanpa rincian fase, diperbarui harian), AC-03.8 (uji tanpa sisa), AC-04.1, AC-07.2 (gate private tidak tampil di publik), risiko §8 |
| 2 | C2 | BLOCKER | Fixture respons Amp nyata akan masuk riwayat git repo public | ACCEPT | Repo public; G4 | NFR Maintainability (repo public hanya fixture sintetis), AC-03.7 (pemindai generik atas seluruh tree repo) |
| 3 | C3 | BLOCKER | US-07 Must bergantung pada persetujuan yang belum ada; K6 tidak bisa lulus tanpa data gate; penguji publik tidak bisa melihat gate private | ACCEPT sebagian | Persetujuan field `gate`/`threads` menjadi keputusan eksplisit di gate Phase 4 dengan cabang jika ditolak; K6 memakai fixture dengan project public yang menunggu. Klaim bahwa `plan-review-1.md` tidak ada **ditolak**: file ada sejak commit `a783666` (`git ls-files docs/reviews`) | US-07 paragraf sumber, K6 |
| 4 | C4 | MAJOR | `as_of` sumber terlama membuat banner basi permanen dan K1 mustahil | ACCEPT | Nilai fase tertutup dan thread > 90 hari memang lama secara permanen | `run.as_of` untuk kesegaran, `state: "frozen"` tanpa banner (US-10, AC-01.6, AC-10.1, AC-10.2, K1) |
| 5 | C5 | MAJOR | K3 benar dengan sendirinya (fase berjalan = total − fase tertutup) | ACCEPT | Tautologi | Fase berjalan dihitung dengan CLI berentang; K3 membandingkan CLI berentang vs API per thread; AC-02.7 selisih → `partial`; AC-02.4 rentang pra-riwayat didefinisikan |
| 6 | C6 | MAJOR | AC Must mengasumsikan hasil spike; M0 tanpa aturan keputusan; pemicu push tidak terdefinisi | ACCEPT | — | AC-02.2/AC-06.1 dengan cabang `null`; exit M0 dengan cabang keputusan per pertanyaan spike; pemicu push dihapus (cron 3 jam saja) |
| 7 | C7 | MAJOR | Exit M3 tidak terukur dan dry run di hosting publik melewati gate Phase 12 | ACCEPT | Gate produksi milik Thomas | K1-dry/K2-dry, dry run di lingkungan privat tanpa unggah publik, `/data/snapshot.v1.json` naik ke Must, skrip pemeriksa AC-10.5 |
| 8 | C8 | MAJOR | Must tidak muat 16 hari; sesi manusia tidak tercatat sebagai dependency | ACCEPT | — | Budget 19 hari (cadangan ± 15%) + waktu kalender terpisah; peserta uji di §7; pemeriksaan ulang banner di browser turun ke Should |
| 9 | C9 | MAJOR | `public_display_name` bertentangan dengan G4 dan agregasi | ACCEPT | Melindungi ide private (tema D) | AC-03.4 dipindah ke Could |
| 10 | C10 | MAJOR | Kanari K5 hanya ada di fixture, tidak memeriksa build nyata | ACCEPT | — | AC-01.9 kolektor menanam kanari di setiap run nyata sebelum filter |
| 11 | C11 | MINOR | Pola pemindai (`sk-`, email, path) akan menggagalkan build bersih | ACCEPT | `task-` cocok dengan `sk-` | AC-03.2 regex berjangkar, allowlist file lisensi, uji negatif `task-list` |
| 12 | C12 | MINOR | Rute ID/EN bertentangan dengan AC-09.1 | ACCEPT | — | Tabel peta rute di US-09, `contracts/routes.json` |

## Options for Thomas

Tidak ada opsi baru; daftar di [round 1](plan-review-1.md#options-for-thomas) tetap berlaku. Opsi 3 (sumber status gate) dan 4 (field `threads`) kini menjadi keputusan eksplisit di gate Phase 4.

## Carried over (not fixed this round)

- Model berbeda untuk critic tidak dipakai (sama dengan round 1).
- Pertanyaan spike M0 (batas 90 hari pada CLI berentang, makna list price untuk thread berbayar kredit, apakah panggilan API memakai kredit, kredensial baca git factory) sudah punya cabang keputusan di exit M0, tetapi belum dijawab.
- Tidak ada BLOCKER yang terbuka.
