# Plan review — round 1

14 Nov 2026. CRITIC dan DISCUSSANT berjalan paralel sebagai subagent dengan konteks baru, read-only, pada `docs/prd.md` draft (commit `b0ff587`). Model kedua untuk critic tidak dipakai karena thread ini diminta tidak membuat thread lain; kedua reviewer memakai model yang sama dengan penulis (risiko blind spot yang sama dicatat di sini). Konsolidator: agent orchestrator.

Sumber: C = critic, D = discussant. Semua perubahan diterapkan di `docs/prd.md` versi round 1.

| # | Source | Severity | Finding | Decision | Reason | Applied in |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | C1 | BLOCKER | Batas 90 hari dihitung dari pembuatan thread; thread orchestrator panjang berhenti terbaca, AC-01.6 menyamarkannya sebagai "arsip" | ACCEPT | Dibuktikan di Amp External API (P12) | AC-01.6 (batas 89/91 hari, label "tidak lengkap"), AC-01.7 (peringatan handoff hari ke-75), risiko §8, spike M0 |
| 2 | C2, D17 | BLOCKER | Set thread per project tidak punya sumber; registry hanya `thread_url` tunggal; K3 memeriksa dirinya sendiri | ACCEPT | P8, P13 | Definisi set thread di US-01 (semua `thread_url` historis dari git + field `threads: []` usulan), AC-01.1/01.2, metrik K4 audit manual. Bukti tambahan: subagent Task termasuk dalam usage thread induk (baris routing purpose `Task` di usage thread perencanaan stack) |
| 3 | C3, D8 | BLOCKER | Penyimpanan data yang tidak bisa dipulihkan belum ditentukan; runner Actions sementara; anggaran waktu kontradiktif (≤ 5 vs ≤ 2 menit) dan kuota 2.000 menit bisa habis | ACCEPT | Matematika kuota diverifikasi kedua reviewer | NFR Reliability dan "Jadwal dan kuota" (tiap 3 jam + push registry, ≤ 1 menit/run, ≤ 300 menit/bulan), AC-01.8 restore di runner baru, lingkungan dan penyimpanan jadi kriteria Phase 5 |
| 4 | C4 | BLOCKER | Pemindai tidak mengenali `github_pat_` dan key Amp, denylist hanya fixture, K-privasi menilai diri sendiri, email kontak akan tertangkap | ACCEPT | Prefix `sgamp_` dari [docs Amp execute mode](https://ampcode.com/docs/cli/execute-mode) | AC-03.2 (pola lengkap, denylist dari registry nyata, email kontak di-allowlist), AC-03.3 kanari, K5 |
| 5 | C5 | MAJOR | Scope kredensial tidak dibatasi; rekonsiliasi di CI memasukkan kredensial Amp ke CI | ACCEPT | P15 | NFR Security (scope token), K3 hanya di lingkungan privat, AC-03.7 |
| 6 | C6, D5 | MAJOR | Biaya per fase bergantung pada teks CLI; dua metode; pembulatan; author vs committer date; clone dangkal | ACCEPT | P6, P11b | US-02: satu metode (CLI berentang, dihitung sekali per fase tertutup), author date, AC-02.5 clone dangkal, AC-02.6, toleransi n × US$0,005, spike M0 |
| 7 | C7 | MAJOR | Publik hanya melihat 2 dari 5 project wave 1; cockpit Thomas hanya lokal padahal persona memakai HP; agregat n = 1 membuka project private | ACCEPT sebagian | Agregat: diterima penuh (AC-03.6, n < 3). Akses HP: Thomas memakai mode publik di HP; akses pemilik dari HP mengubah scope (auth) → opsi untuk Thomas | AC-03.6, §3 Personas, K6 menyebut mode |
| 8 | C8, D11 | MAJOR | Field `gate` belum ada; fallback `status: blocked` salah tafsir | ACCEPT | P8 | US-07: spesifikasi `gate: {phase, since}`, AC-07.4 "Data gate belum tersedia", persetujuan jadi syarat masuk M2. Alternatif sumber gate (D11) → opsi |
| 9 | C9 | MAJOR | M7 (traffic API GitHub) mengukur halaman repo, bukan situs; beacon pihak ketiga bertentangan dengan CSP | ACCEPT | Dokumentasi traffic API GitHub hanya untuk repo | K8 dari analitik sisi server penyedia, atau "tidak terukur"; syarat hosting §6 |
| 10 | C10 | MAJOR | ID metrik dan milestone bertabrakan; exit M3 memakai metrik pasca-launch | ACCEPT | — | Metrik K1–K9; exit M3 = dry run 72 jam |
| 11 | C11 | MAJOR | Bagian UU PDP keliru: IP pengunjung, kontak, transfer lintas negara, notifikasi 3×24 jam, halaman privasi | ACCEPT | UU 27/2022 Pasal 46, 56 | Inventaris data dengan dasar pemrosesan, prosedur insiden, US-15 halaman privasi (Must) |
| 12 | C12, D10 | MAJOR | Snapshot yang di-commit ke repo public tidak bisa di-rollback; pemindai harus sebelum unggah | ACCEPT | Riwayat git publik permanen | Alur US-03 (pindai sebelum unggah, deploy artefak), AC-03.7, syarat hosting §6 |
| 13 | C13 | MAJOR | WCAG 2.2 AA hanya daftar sebagian dan satu alat | ACCEPT | — | NFR Accessibility (1.4.11, 1.4.10, 2.4.7, 2.4.11, 4.1.3, nama aksesibel), test keyboard-only dan 320 px, sesi screen reader M3 |
| 14 | C14 | MAJOR | Header keamanan tidak bisa di GitHub Pages | ACCEPT | Pages tidak mendukung header kustom | Syarat hosting §6; header diuji e2e |
| 15 | C15, D6 | MAJOR | Scope Must ketat; 16 hari > 3 minggu; Should masih di M2 | ACCEPT | — | Filter (US-05b) dan sort tanpa JS dipindah ke Should; garis potong eksplisit; budget dinyatakan 3,2 minggu tanpa cadangan |
| 16 | C16 | MAJOR | `public_costs` hanya di tabel risiko; `cost_usd` registry tidak dijelaskan | ACCEPT | P8 | AC-03.5. `cost_usd` registry tidak dipakai untuk tampilan; dicatat di sini dan dibahas di ADR M0 |
| 17 | C17 | MINOR | Nama tampilan project private membuka slug repo | ACCEPT | Nama repo = slug ide | AC-03.4 `public_display_name` |
| 18 | C18 | MINOR | State offline tanpa service worker; `?fase=` butuh JS | ACCEPT | — | Offline jadi non-goal; US-05b mendefinisikan perilaku tanpa JS |
| 19 | C19 | MINOR | Celah ambang basi, tanpa JS tidak ada banner, `partial` memberi `generated_at` segar | ACCEPT | — | Satu ambang 6 jam, `as_of` dan `complete` per nilai, banner HTML statis saat build (AC-10.1, AC-10.2) |
| 20 | C20 | MINOR | AC samar (metodologi, durasi, alasan, perintah deploy, format angka) | ACCEPT | — | Anchor bernama AC-08.1/08.2, format durasi AC-06.1, AC-06.3 tanpa "alasan", AC-11.2 pada `infra/deploy`, AC-09.2 dengan mata uang |
| 21 | C21 | MINOR | Keterlacakan G3 lemah; uji bacaan bukan metrik | ACCEPT | — | G3 dirumuskan ulang, K9 |
| 22 | C22 | MINOR | M5/M8 lemah | ACCEPT | — | K6 dengan event dan 2 peserta non-pemilik; metrik retro dengan batas 31 Mar 2027 dan boleh mengutip ekspor |
| 23 | C23 | MINOR | INP tidak keluar dari Lighthouse | ACCEPT | Lighthouse navigation tidak mengukur INP | TBT ≤ 200 ms |
| 24 | C24 | MINOR | Penjumlahan list price lintas mode billing tidak jelas | ACCEPT | — | Spike M0 dua mode billing; AC-08.1 `#list-price` |
| 25 | D1–D3 | — | Steelman: satu pipeline, angka jujur, privasi menggagalkan build | ACCEPT | Dipertahankan | — |
| 26 | D7 | — | JSONL append-only di repo git private sebagai pengganti PostgreSQL | Dibawa ke Phase 5 | Keputusan stack, bukan PRD | Kriteria penyimpanan §6/§7 |
| 27 | D9 | — | Data dulu: M1 mengirim kolektor + snapshot + beranda sederhana | ACCEPT | Riwayat mulai tercatat lebih cepat | Milestone M1 |
| 28 | D13, D15, D16 | — | Perbandingan fase, waktu tunggu gate, validasi registry | ACCEPT sebagai Could | Tidak muat di Must | §4 Could |
| 29 | D14 | — | Tabel retro Markdown | ACCEPT sebagai Should | Mendukung metrik retro | US-16 |
| 30 | D18 | — | Angka utama: kredit atau list price | Opsi untuk Thomas | Keputusan presentasi publik | Default: dua angka setara |
| 31 | D20 | — | Lokasi kolektor dan situs | Dibawa ke Phase 5 | Syarat sudah ditetapkan di §6 | §6, §7 |

## Options for Thomas

Usulan yang mengubah scope atau kebijakan publik; tidak diterapkan diam-diam.

1. **(D4) Mode pemilik sebagai laporan CLI/Markdown, bukan situs kedua.** Lebih sederhana (hapus penanda `owner` dan guard deploy). Trade-off: tidak nyaman di HP; K6 Thomas diukur di terminal. Rekomendasi agent: tetap situs lokal (keputusan default PRD), karena memakai komponen yang sama.
2. **(C7, D12) Akses pemilik dari HP lewat Cloudflare Access.** Trade-off: menambah auth (non-goal), target deploy kedua dan dampak kebocoran lebih besar. Rekomendasi agent: tidak di MVP.
3. **(C8, D11) Sumber status gate.** (a) Field registry `gate: {phase, since}` + perubahan skill factory-orchestrator (default PRD, butuh persetujuanmu dan push skill); (b) parsing tabel "Gates" di `docs/progress.md` tiap project (rapuh, tanpa ubah registry); (c) tebakan dari fase gate + `agentState: idle` hanya di mode pemilik (bertentangan dengan AC-07.4).
4. **(C2, D17) Field registry `threads: []`** untuk thread anak `create_thread`, ditulis orchestrator. Tanpa ini, biaya thread anak tidak terhitung.
5. **(D18) Angka biaya utama.** Kredit Amp (yang benar-benar dibayar; thread ini US$0,16), estimasi list price (pembanding untuk tim lain; thread ini US$9,67), atau dua-duanya setara (default PRD).
6. **(D19) Apa yang boleh publik.** Default PRD: nama dan biaya project public; project private hanya agregat (n ≥ 3); project `killed` tampil. Opsi tambahan: nama ide kategori public sebagai pipeline "akan datang", atau nama tampilan project private tertentu.
7. **(D8) Frekuensi kolektor.** Default PRD tiap 3 jam + saat push registry (ambang basi 6 jam). Lebih sering = lebih banyak menit Actions private.

## Carried over (not fixed this round)

- Model berbeda untuk critic tidak dipakai (lihat paragraf pembuka).
- D7 (penyimpanan JSONL vs PostgreSQL) dan D20 (lokasi kolektor/hosting): Phase 5.
- Terjawab setelah round 1 (P17): CLI berentang memuat list price dan membulatkan ke sen.
- Spike M0 harus memastikan: batas 90 hari pada CLI berentang, makna list price untuk thread berbayar kredit, apakah panggilan API memakai kredit, read-only git Amp-hosted.
