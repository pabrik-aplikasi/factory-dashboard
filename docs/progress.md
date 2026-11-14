# Progress — Factory dashboard publik

## Status

- Phase: 4 Readiness selesai, verdict GO; **menunggu gate GO / NO-GO dari Thomas**
- Status: active (wave 1)
- Thread: https://ampcode.com/threads/T-01a1d88d-f59a-72f9-9e3d-280aa4f913a5
- Cost so far: US$0,17 kredit Amp; estimasi list price US$15,84 untuk inference lewat subscription Claude (dibaca 14 Nov 2026 06.04 UTC); ± 30 menit kerja

## Decisions

| Date | Phase | Decision | Why | Decided by |
| --- | --- | --- | --- | --- |
| 2026-11-14 | 0 | Wave 1 berisi 115, 403, 402, 601, 816; satu thread per ide | Persetujuan gate Phase 0 | Thomas |
| 2026-11-14 | 0 | Aturan diversitas stack mengikuti versi longgar 7 Nov (kecocokan > kuota) | `docs/tech-stack-plan.md` di repo factory; keputusan Thomas 7 Nov | Thomas |
| 2026-11-14 | 0 | Default privasi: project private tampil sebagai jumlah, bukan nama | Repo factory tidak publik (HTTP 404 tanpa login); 69 repo private di org | Agent |
| 2026-11-14 | 0 | Hanya thread yang tercatat di registry yang dihitung biayanya | `amp threads list` memuat thread di luar factory | Agent |
| 2026-11-14 | 1 | Varian V1 (snapshot statis ber-allowlist + ekspor data + metodologi), nama kerja "Lantai Pabrik" | Satu pipeline melayani Thomas dan publik; murah; menjaga privasi (research.md) | Agent |
| 2026-11-14 | 1 | Biaya dari Amp External API JSON; per fase dari CLI `--start/--end` | API tidak punya rentang waktu, CLI punya (P11, P11b) | Agent |
| 2026-11-14 | 1 | Kolektor menyimpan pembacaan usage append-only sejak run pertama | Amp hanya menyimpan usage thread 90 hari (P12) | Agent |
| 2026-11-14 | 2 | Tanpa login di MVP; mode pemilik hanya lokal | Budget, privasi, anti-persona observability | Agent |
| 2026-11-14 | 2 | Kredit Amp dan estimasi list price ditampilkan terpisah, tidak pernah dijumlahkan | Thread ini: US$0,16 vs US$9,67; makna berbeda (P5) | Agent |
| 2026-11-14 | 3 | Round 1: 4 BLOCKER dan 12 MAJOR diterima; 7 opsi scope dibawa ke gate | [plan-review-1.md](reviews/plan-review-1.md) | Agent |
| 2026-11-14 | 3 | Round 2: total publik hanya dari project public; kohort private n ≥ 3 harian; repo public hanya fixture sintetis; kesegaran dari `run.as_of` | Menutup kebocoran lewat pengurangan dan riwayat git ([plan-review-2.md](reviews/plan-review-2.md)) | Agent |
| 2026-11-14 | 3 | Kolektor tiap 3 jam tanpa pemicu push; budget Must 19 hari agent | Kuota Actions private dan cadangan waktu | Agent |
| 2026-11-14 | 4 | Readiness GO; dua keputusan diminta di gate (field `gate`/`threads`, kebijakan data publik) | [readiness.md](reviews/readiness.md) | Agent |

## Gates

| Gate | Date | Answer from Thomas |
| --- | --- | --- |
| Phase 0 wave selection | 2026-11-14 | Wave 1 disetujui: 115, 403, 402, 601, 816 |
| Phase 4 GO / NO-GO | 2026-11-14 | Menunggu jawaban Thomas |

## Blockers

None.

## Riwayat sebelum wave

Repo ini dibuat 7 Nov 2026 sebagai repo dokumentasi (idea, stack, stack-support, decisions, sources). Dokumen itu tetap menjadi baseline perencanaan stack, bukan stack final.

## Handoff

Belum diperlukan.
