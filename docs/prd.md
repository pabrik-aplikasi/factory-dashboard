# PRD — Lantai Pabrik (Factory dashboard publik)

Status: in review (round 1 dan 2 diterapkan, 14 Nov 2026)
Delivery: showcase (situs statis publik + kolektor privat; penyedia hosting dipilih di Phase 5 dengan syarat di §6, disetujui di gate Phase 12)

Dokumen sumber: [brief](brief.md), [research](research.md), [personas](personas.md), [plan review 1](reviews/plan-review-1.md). Kode P1–P16 merujuk ke tabel "Bukti primer" di research.md. Metrik memakai ID **K1–K9**; milestone memakai **M0–M3**.

## 1. Problem and goals

- **Problem:** Progres dan biaya factory tersebar di registry privat, 135 repo, Linear dan ratusan thread Amp. Biaya hanya terlihat per thread dan memiliki dua angka berbeda (P3, P5). Thomas tidak punya satu layar untuk "apa yang menunggu aku dan berapa biayanya", dan publik tidak bisa melihat factory bekerja tanpa risiko membocorkan data privat (P2, P7).
- **Goals:**
  - G1. Thomas melihat posisi setiap project dan gate yang menunggunya dalam satu layar. → [Thomas, pain 1–2](personas.md#persona-1-thomas-pemilik-factory)
  - G2. Biaya agent per project dan per fase dihitung dari data Amp asli, dengan kredit Amp dan estimasi list price dipisah dan dijelaskan. → P3–P5; [Thomas, pain 3–4](personas.md#persona-1-thomas-pemilik-factory); [Dimas, pain 2](personas.md#persona-2-dimas-tech-lead-yang-mengevaluasi-agent-ai)
  - G3. Publik bisa memeriksa bahwa factory nyata: fase, waktu dan biaya ditautkan ke repo public yang bisa dibuka, dalam Bahasa Indonesia dan Inggris, tanpa perlu alat observability. → [Dimas, pain 1, 3, 4](personas.md#persona-2-dimas-tech-lead-yang-mengevaluasi-agent-ai); tema A, B, E di research.md
  - G4. Tidak ada data privat yang keluar: nama/slug/repo project private, ID/judul/isi thread, secret dan data orang lain tidak pernah masuk artefak publik maupun riwayat git publik. → P2, P7, tema D
  - G5. Data tidak pernah basi atau tidak lengkap diam-diam: setiap nilai punya `as_of` dan status kelengkapan. → [Thomas, pain 4–5](personas.md#persona-1-thomas-pemilik-factory); tema C, F
- **Non-goals:**
  - Tidak mengubah registry, Linear atau thread dari dashboard (read-only terhadap semua sumber).
  - Tidak ada login, akun pengguna atau Keycloak di MVP. Mode pemilik berjalan lokal.
  - Bukan observability LLM: tidak ada trace, prompt, isi pesan atau judul thread.
  - Tidak realtime; kolektor terjadwal tiap 3 jam (tanpa pemicu push; Amp-hosted git tidak dipakai sebagai sumber webhook).
  - Tidak ada mode offline/service worker.
  - Bukan produk multi-factory/SaaS untuk pengguna Amp lain di versi ini.
  - Tidak mengonversi ke Rupiah di MVP.

## 2. Success metrics

| ID | Metric | Target | Window | How measured |
| --- | --- | --- | --- | --- |
| K1 | Kesegaran data | `run.as_of` (waktu run kolektor terakhir yang `ok`/`partial`) ≤ 6 jam pada ≥ 95% pemeriksaan per jam | 14 hari setelah launch | Skrip pemeriksa (US-10, AC-10.5) membaca `/data/snapshot.v1.json`. Versi pra-launch K1-dry: setiap run dalam dry run 72 jam punya `run.as_of` ≤ 6 jam |
| K2 | Cakupan biaya | ≥ 95% thread terdaftar yang berumur < 90 hari berhasil dibaca per run | 14 hari setelah launch; K2-dry: semua run dry run 72 jam | Rasio `threads_ok / threads_eligible` di log run kolektor |
| K3 | Rekonsiliasi biaya | Per thread: Σ biaya CLI berentang atas [createdAt, run) = total API thread itu, selisih ≤ n × US$0,005 (n = jumlah rentang yang dijumlahkan) | Setiap run kolektor (di lingkungan privat) | Pemeriksaan di kolektor yang membandingkan dua sumber independen (CLI berentang vs API); hasil di log run |
| K4 | Kelengkapan daftar thread | 0 thread project yang hilang dari set terkumpul | Audit manual sekali di M3 untuk satu project wave 1 | Bandingkan daftar thread project di ampcode.com dengan set kolektor; dicatat di docs/test-report.md |
| K5 | Privasi | 0 temuan pemindai pada artefak publik, dan 0 string kanari di artefak | Setiap build publik sejak M1 | Pemindai (US-03) + kanari yang ditanam di fixture dan konfigurasi uji |
| K6 | Waktu menjawab "apa yang menunggu aku" | ≤ 10 detik dari event `load` sampai peserta menyebut/menyentuh gate yang benar, 3/3 percobaan | Sesi uji di M3: Thomas (mode pemilik, laptop) + 2 developer non-pemilik (mode publik, HP, fixture di mana project yang menunggu adalah project public) | Rekaman layar; dicatat di docs/test-report.md. Bila field `gate` tidak disetujui, K6 menjadi "menemukan fase project tertentu" |
| K7 | Performa mobile | Lighthouse mobile Performance ≥ 90, LCP ≤ 2,0 s, TBT ≤ 200 ms, CLS < 0,1 di beranda dan detail project | Setiap build publik sejak M3 | Lighthouse CI dengan throttling mobile default |
| K8 | Jangkauan publik (proxy) | ≥ 200 pengunjung unik | 30 hari setelah launch | Analitik request sisi server dari penyedia hosting (tanpa script di browser). Bila penyedia tidak menyediakan, K8 dilaporkan "tidak terukur", bukan diganti angka lain |
| K9 | Angka dipahami dengan benar | 3/3 developer non-pemilik menjelaskan dengan benar beda kredit Amp dan estimasi list price setelah membaca situs | Sesi uji di M3 | Pertanyaan tertulis yang sama; jawaban dicatat di docs/test-report.md |

Metrik pendukung retro (bukan syarat launch): retro Phase 13 setiap project wave 1 yang selesai sebelum 31 Mar 2027 mengutip biaya per fase dari ekspor Lantai Pabrik.

## 3. Personas

Primer: **Thomas, pemilik factory** (memutuskan ke mana perhatian dan uang pergi). Sekunder: **Dimas, tech lead yang mengevaluasi agent AI**. Anti-persona: Rani, platform engineer yang mencari observability LLM real-time. Detail di [personas.md](personas.md). Thomas memakai mode pemilik di laptop; di HP ia melihat mode publik (keputusan round 1; opsi akses pemilik dari HP ada di [plan review](reviews/plan-review-1.md#options-for-thomas)).

## 4. MVP scope (MoSCoW)

| Priority | Feature | Story IDs |
| --- | --- | --- |
| Must | Kolektor terjadwal: registry + riwayatnya, biaya thread Amp, riwayat fase, umur thread, status sumber, penyimpanan append-only | US-01, US-02 |
| Must | Snapshot publik ber-allowlist + pemindai privasi sebelum unggah, kanari, denylist dari registry nyata | US-03 |
| Must | Beranda ringkasan | US-04 |
| Must | Daftar project (tabel statis, urut fase) dengan agregat private | US-05 |
| Must | Detail project | US-06 |
| Must | Menunggu Thomas | US-07 |
| Must | Metodologi biaya | US-08 |
| Must | ID/EN | US-09 |
| Must | Peringatan basi/tidak lengkap per nilai | US-10 |
| Must | Mode pemilik lokal | US-11 |
| Must | Halaman privasi | US-15 |
| Should | Filter daftar + state URL | US-05b |
| Must | File `/data/snapshot.v1.json` tervalidasi schema dilayani bersama situs (input K1, AC-13.1) | US-13 |
| Should | Dokumentasi dataset terbuka (README data, lisensi data) | US-13 |
| Should | Tabel retro per project (Markdown, dari ekspor) | US-16 |
| Should | Status CI dan rilis repo public | US-12 |
| Should | Feed perubahan fase | US-14 |
| Could | Perbandingan fase lintas project (median biaya/durasi, dengan n) | — |
| Could | Waktu tunggu gate dan project macet (mode pemilik) | — |
| Could | Laporan validasi registry (mode pemilik) | — |
| Could | Nama tampilan project private (`public_display_name`), badge README, progres Linear, grafik kumulatif, Rupiah, gambar OG | — |
| Won't (this version) | Login/akun, edit registry, realtime, service worker/offline, komentar, multi-factory, judul/isi thread, judul issue Linear publik | — |

**Garis potong:** M1–M2 hanya berisi Must. Should dikerjakan di M3 setelah semua AC Must hijau, urut US-13, US-16, US-05b, US-12, US-14; yang tidak muat pindah ke versi berikut tanpa menunda launch.

## 5. User stories

Konvensi:

- "Artefak publik" = semua file hasil build mode publik (HTML, JSON, feed, aset, header).
- "Fixture registry" = repo git uji dengan riwayat commit, berisi project public dan private, dan string kanari `CANARY-PRIVATE-<n>` di field yang tidak di-allowlist dan di project private.
- Setiap AC dibuktikan oleh test e2e di `e2e/` yang menjalankan kolektor dan build nyata terhadap fixture. Amp API, Amp CLI dan GitHub API diganti server/binary palsu **di batas proses** (bukan mock unit yang diuji). Server palsu mencatat setiap request.
- Path JSON merujuk ke `contracts/snapshot.v1.schema.json` (dikunci di M0).

### US-01 Kolektor membaca registry dan biaya

Sebagai Thomas, aku ingin kolektor terjadwal membaca registry factory dan biaya setiap thread project, supaya angka di dashboard berasal dari sumber asli.

Set thread project = semua nilai `thread_url` project itu di seluruh riwayat git registry + field opsional `threads: []` bila disetujui Thomas (thread anak dari `create_thread`). Subagent Task sudah termasuk dalam usage thread induknya (bukti: usage thread perencanaan stack memuat baris routing ber-purpose `Task`).

Acceptance criteria:
- AC-01.1 Given fixture registry dengan 3 project dan riwayat yang memuat 4 nilai `thread_url`, dan server API palsu mengembalikan `usage: 0.34` dan `estimatedProviderCostAtListPriceUSD: 52.86` untuk thread T1, When kolektor dijalankan, Then log run memuat satu run `ok` dengan `threads_ok = 4`, dan snapshot pemilik memuat `projects[].threads[T1].credits_usd = 0.34` dan `list_price_usd = 52.86` sebagai field terpisah.
- AC-01.2 Given riwayat registry mengganti `thread_url` project A dari X ke Y di commit yang belum pernah dilihat kolektor, When kolektor dijalankan pertama kali, Then biaya A = biaya X + biaya Y.
- AC-01.3 Given API usage untuk satu thread mengembalikan HTTP 5xx, 429, atau JSON yang tidak cocok dengan schema, When kolektor dijalankan, Then run berstatus `partial`, nilai thread itu memakai pembacaan terakhir dengan `as_of` lama dan `complete: false`, dan thread lain diperbarui.
- AC-01.4 Given registry tidak bisa dibaca, When kolektor dijalankan, Then run berstatus `failed` dan snapshot sebelumnya tidak ditimpa (hash file tidak berubah).
- AC-01.5 Given akun Amp punya thread yang tidak ada di set thread project, When kolektor dijalankan, Then server palsu mencatat 0 request ke `GET /api/v2/threads` (daftar) dan 0 request untuk thread itu.
- AC-01.6 Given thread berumur 89 hari (dari `createdAt` metadata), When kolektor dijalankan, Then biaya dibaca normal; Given umur 91 hari, Then nilai terakhir dipakai dengan `state: "frozen"` dan label "biaya tidak lengkap sejak <tanggal hari ke-90>", tidak memicu banner basi, dan thread itu tercatat terpisah di log (tidak dihitung di K2).
- AC-01.7 Given thread berumur ≥ 75 hari yang masih tercatat sebagai `thread_url` aktif, When kolektor dijalankan, Then log run dan mode pemilik memuat peringatan "handoff disarankan sebelum hari ke-85" dengan nama project.
- AC-01.8 Given kolektor dijalankan dua kali berturut-turut pada runner baru yang memulihkan state dari penyimpanan, When biaya dibandingkan, Then total tidak berubah (tidak ada penggandaan).
- AC-01.9 Given run nyata kolektor, When snapshot pemilik dibuat, Then kolektor menanam satu project private kanari dan satu field tak dikenal berisi kanari sebelum filter allowlist (tanpa menulis ke registry), dan laporan pemindai build publik berbunyi "1 kanari ditanam, 0 lolos".

States: empty (registry tanpa project aktif → beranda "Belum ada project berjalan"), error (run failed → banner US-10), success (run ok + `as_of`). Loading dan offline tidak berlaku untuk kolektor.

### US-02 Riwayat fase dan biaya per fase

Sebagai Thomas, aku ingin tahu kapan setiap project masuk setiap fase dan berapa biaya tiap fase, supaya retro dan keputusan budget berdasar angka.

Waktu transisi = **author date** commit registry yang mengubah `phase`; kolektor wajib memakai clone dengan riwayat lengkap (gagal bila clone dangkal). Biaya fase dihitung **sekali** dengan CLI `amp threads usage --start --end` saat fase ditutup (sebelum thread berumur 90 hari), lalu disimpan permanen dengan `state: "frozen"`. Fase yang berjalan juga dihitung dengan CLI berentang [awal fase, run). Baris "Sebelum riwayat fase" = CLI berentang [createdAt thread, transisi pertama yang diketahui).

Acceptance criteria:
- AC-02.1 Given commit registry dengan author date T yang mengubah `phase` A dari 1 ke 2, When kolektor melakukan backfill, Then `projects[A].phases[2].entered_at = T` dan `source = "registry-commit"`, dan halaman detail menampilkan "Fase 2 sejak <T dalam WIB>".
- AC-02.2 Given A di fase 1 dari T1 sampai T2 dan CLI palsu mengembalikan kredit dan list price per rentang, When biaya per fase dihitung, Then `phases[1].credits_usd` dan `phases[1].list_price_usd` = jumlah usage(thread, T1, T2) semua thread A. Given CLI palsu tanpa baris list price, Then `phases[1].list_price_usd = null` dan halaman menulis "tidak tersedia".
- AC-02.3 Given fase berubah dua kali di antara dua run, When run berikutnya, Then kedua transisi tercatat dengan author date masing-masing.
- AC-02.4 Given thread dibuat pada T0 dan transisi pertama yang diketahui pada T1 > T0, When ditampilkan, Then baris "Sebelum riwayat fase" = usage(thread, T0, T1).
- AC-02.5 Given clone registry dangkal, When kolektor dijalankan, Then run `failed` dengan pesan "riwayat registry tidak lengkap".
- AC-02.6 Given CLI berentang gagal untuk fase tertutup, When ditampilkan, Then fase itu menampilkan "belum dihitung" dan `complete: false`, bukan nol.
- AC-02.7 Given CLI berentang dan API untuk satu thread berselisih melebihi toleransi K3, When run selesai, Then run berstatus `partial`, nilai project itu `complete: false`, dan log menyebut thread serta selisihnya.

States: empty (project tanpa transisi → satu baris fase saat ini, "riwayat belum tersedia"), error (AC-02.5, AC-02.6), success.

### US-03 Snapshot publik ber-allowlist

Sebagai Thomas, aku ingin artefak publik hanya berisi field yang di-allowlist dan diperiksa sebelum diunggah, supaya tidak ada data privat yang bocor, termasuk ke riwayat git publik.

Alur: kolektor (privat) → snapshot pemilik → filter allowlist → snapshot publik → build situs → **pemindai** → unggah artefak ke hosting. Tidak ada data hasil kolektor yang di-commit ke repo public.

Acceptance criteria:
- AC-03.1 Given fixture registry dengan project private `201-payroll-umkm`, When build publik dijalankan, Then string `payroll-umkm`, `201-payroll-umkm`, nama project dan URL repo-nya tidak ada di artefak publik.
- AC-03.2 Given artefak berisi salah satu dari: ID thread (`\bT-[0-9a-f]{8}-[0-9a-f]{4}-`), `ampcode.com/threads`, token (`\bgh[pousr]_[A-Za-z0-9]{20,}`, `\bgithub_pat_[A-Za-z0-9_]{20,}`, `\bsgamp_[A-Za-z0-9_]{10,}`, `\bsk-[A-Za-z0-9_-]{20,}`, `\blin_api_[A-Za-z0-9]{20,}`), path absolut (`/home/`, `/Users/`, `/root/`, `C:\\`), email selain alamat kontak yang dikonfigurasi (file lisensi vendor di-allowlist per file), atau slug/`name`/repo project private dari **registry nyata saat build** (cocok tanpa peduli huruf besar-kecil), When pemindai berjalan, Then build keluar dengan kode ≠ 0, menyebut file dan pola, dan tidak ada unggahan (server hosting palsu mencatat 0 request). Given teks biasa `task-list` atau `risk-matrix`, Then pemindai lulus (uji negatif).
- AC-03.3 Given registry mendapat field baru (misalnya `notes: CANARY-PRIVATE-1`), When build publik, Then kanari tidak ada di artefak.
- AC-03.4 Dipindah ke Could (round 2): nama tampilan project private tidak ada di MVP.
- AC-03.5 Given project dengan `public_costs: false` di konfigurasi situs, When build publik, Then biayanya tidak ada di HTML, JSON dan feed, dan halaman menulis "Biaya tidak dipublikasikan".
- AC-03.6 Given snapshot dengan project private dan project `public_costs: false`, When build publik, Then semua total, jumlah per fase, jumlah per status dan daftar "Menunggu" dihitung **hanya dari project public yang biayanya dipublikasikan**; project private muncul hanya sebagai satu baris kohort (jumlah project + total kredit dibulatkan ke US$1) bila n ≥ 3, tanpa rincian fase, dan nilai kohort diperbarui sekali sehari (00.00 WIB), bukan per run.
- AC-03.8 Given artefak publik, When test e2e menjumlahkan baris per project yang terlihat, Then hasilnya sama persis dengan total dan jumlah per fase di beranda (tidak ada sisa yang bisa dikurangkan).
- AC-03.7 Given repo public `factory-dashboard`, When CI repo public berjalan, Then pemindai pola generik (AC-03.2 tanpa denylist registry) dijalankan atas seluruh tree repo dan lulus, tidak ada snapshot hasil kolektor maupun respons Amp nyata di repo, dan workflow public tidak mereferensikan secret Amp, Linear atau git factory.

States: error (build gagal, unggah tidak terjadi), success (laporan pemindai "0 temuan, 0 kanari").

### US-04 Beranda ringkasan

Sebagai Dimas, aku ingin ringkasan factory dalam satu layar, supaya aku cepat menilai skala dan kemajuannya.

Acceptance criteria:
- AC-04.1 Given snapshot dengan 5 project public aktif di fase 1, 2, 2, 4 dan 6 dan 3 project private, When beranda dibuka, Then terlihat jumlah per fase 0–13 dari project public saja (fase 2 = 2), jumlah per status, total kredit Amp dan total estimasi list price sebagai dua angka berlabel, baris kohort "3 project private" (AC-03.6), dan "Data per <run.as_of WIB>".
- AC-04.2 Given 2 project menunggu gate, When beranda dibuka, Then ada tautan "2 menunggu keputusan Thomas" ke halaman US-07.
- AC-04.3 Given snapshot tanpa project aktif, When beranda dibuka, Then tampil "Belum ada project berjalan" dan jumlah project terdaftar, tanpa angka biaya.
- AC-04.4 Given JavaScript dimatikan, When beranda dibuka, Then semua angka, "Data per" dan tautan tampil.

States: empty (AC-04.3), error (banner US-10), success. Loading tidak ada (HTML statis).

### US-05 Daftar project

Sebagai Dimas, aku ingin daftar semua project yang terlihat, supaya aku menemukan yang menarik.

Acceptance criteria:
- AC-05.1 Given 3 project public dan 3 project private di snapshot, When halaman daftar dibuka, Then 3 baris public tampil (nama, kategori, fase, status dengan teks, aktivitas terakhir, kredit, list price) urut fase menurun, plus satu baris "3 project private".
- AC-05.2 Given registry kosong, When dibuka, Then tampil "Belum ada project terdaftar".

**US-05b (Should) filter:** AC-05b.1 Given filter fase = 2, When diterapkan, Then hanya project fase 2 tampil, URL memuat `?fase=2`, membuka URL itu langsung memberi hasil sama, dan jumlah hasil diumumkan lewat region `role="status"`. AC-05b.2 Given filter tanpa hasil, Then "Tidak ada project yang cocok" + tombol reset. Tanpa JavaScript, kontrol filter tidak ditampilkan dan tabel lengkap tetap tampil.

States: empty (AC-05.2), error (banner), success.

### US-06 Detail project

Sebagai Dimas, aku ingin halaman detail tiap project public, supaya aku bisa melihat perjalanan dan biayanya lalu membuka repo-nya.

Acceptance criteria:
- AC-06.1 Given project public A dengan transisi 0→1→2, When `/id/project/<slug>/` dibuka, Then timeline menampilkan tiga fase dengan tanggal masuk, durasi dalam format "N hari M jam" (ID) / "N d M h" (EN), kredit per fase, list price per fase atau "tidak tersedia" bila `list_price_usd = null` (AC-02.2), dan tautan ke repo GitHub public A.
- AC-06.2 Given slug project private atau slug yang tidak ada, When URL dibuka, Then HTTP 404 dengan halaman 404 dan tautan ke daftar; nama project private tidak muncul.
- AC-06.3 Given project `status: killed` atau `blocked`, When detail dibuka, Then status tampil sebagai teks ("Dihentikan", "Terhambat") selain ikon/warna.

States: empty (tanpa biaya → "Belum ada biaya tercatat"), error (banner), success.

### US-07 Menunggu Thomas

Sebagai Thomas, aku ingin daftar gate yang menunggu keputusanku, urut paling lama, supaya tidak ada project yang macet karena aku.

Sumber: field registry opsional `gate: {phase: <4|6|7|12>, since: <ISO8601>}` yang ditulis orchestrator saat berhenti di gate dan dihapus saat Thomas menjawab. Field ini (dan `threads: []`) butuh persetujuan Thomas dan perubahan skill factory-orchestrator; **keputusan diminta di gate Phase 4**. Bila ditolak, US-07 turun ke Should, AC-07.4 tetap berlaku, dan K6 memakai definisi alternatif.

Acceptance criteria:
- AC-07.1 Given registry mencatat `gate` untuk A (phase 4, since T1) dan B (phase 6, since T2 < T1), When halaman dibuka, Then B tampil di atas A dengan nama gate dan lama menunggu "N hari M jam".
- AC-07.2 Given `gate` pada project private, When halaman publik dibuka, Then project itu tidak muncul sama sekali di halaman publik; mode pemilik menampilkan nama dan tautan thread.
- AC-07.3 Given tidak ada `gate` tercatat dan field sudah dipakai registry, When halaman dibuka, Then "Tidak ada keputusan yang menunggu".
- AC-07.4 Given registry belum pernah memakai field `gate`, When halaman dibuka, Then "Data gate belum tersedia" dan tidak ada project yang ditebak menunggu.

States: empty (AC-07.3), error (AC-07.4, banner), success.

### US-08 Metodologi biaya

Sebagai Dimas, aku ingin penjelasan angka biaya, supaya aku tidak salah mengutip.

Acceptance criteria:
- AC-08.1 Given `/id/metodologi/` dibuka, Then halaman memuat heading dengan anchor `#kredit-amp`, `#list-price`, `#tidak-dihitung`, `#per-fase`, `#kelengkapan` dan teks versi "Metodologi v<N>, <tanggal>"; bagian `#list-price` menyatakan kedua angka tidak dijumlahkan dan menjelaskan nilai untuk thread yang dibayar kredit dan yang lewat subscription.
- AC-08.2 Given setiap angka biaya di situs, When tautan info di sebelahnya diaktifkan, Then pengguna sampai ke anchor sesuai jenis angka (kredit → `#kredit-amp`, list price → `#list-price`, biaya fase → `#per-fase`), dan tautan punya nama aksesibel "Tentang angka ini".

States: success (statis).

### US-09 Dua bahasa

Sebagai Dimas, aku ingin memilih Bahasa Indonesia atau Inggris.

Acceptance criteria:
- AC-09.1 Given setiap path ID di peta rute (`contracts/routes.json`), When pengguna memilih English, Then ia sampai ke path EN pasangannya dari peta rute, atribut `lang` = `en`, dan anchor yang sama tetap ada.
- AC-09.2 Given biaya 1234.5 USD dan tanggal 14 Nov 2026 07:00 UTC, When ditampilkan, Then ID = "US$1.234,50" dan "14 Nov 2026 14.00 WIB", EN = "US$1,234.50" dan "14 Nov 2026 14:00 WIB"; biaya > 0 dan < 0,005 tampil "< US$0,01" / "< US$0.01".
- AC-09.3 Given key terjemahan yang hilang, When build, Then build gagal dengan nama key.

Peta rute (dikunci di M0; anchor sama di kedua bahasa):

| ID | EN |
| --- | --- |
| `/id/` | `/en/` |
| `/id/project/` | `/en/project/` |
| `/id/project/<slug>/` | `/en/project/<slug>/` |
| `/id/menunggu/` | `/en/waiting/` |
| `/id/metodologi/` | `/en/methodology/` |
| `/id/privasi/` | `/en/privacy/` |
| `/id/feed.xml` | `/en/feed.xml` |
| `/data/snapshot.v1.json` (satu file, bahasa netral) | sama |

States: success; error (AC-09.3).

### US-10 Data basi dan tidak lengkap

Sebagai Thomas, aku ingin peringatan bila data basi atau tidak lengkap, supaya publik tidak melihat angka salah tanpa konteks.

Satu ambang: **6 jam**, diukur dari `run.as_of` (run terakhir `ok`/`partial`), bukan dari nilai per item. Nilai `frozen` (fase tertutup, thread > 90 hari) tidak pernah memicu banner.

Acceptance criteria:
- AC-10.1 Given `run.as_of` berumur > 6 jam pada waktu build, When halaman dirender, Then banner HTML statis "Data terakhir diperbarui <waktu WIB>, mungkin sudah tidak akurat" tampil tanpa JavaScript. (Pemeriksaan ulang di browser saat halaman dibuka = Should.)
- AC-10.2 Given nilai dengan `complete: false` yang masih bisa dipulihkan (gagal sementara, selisih K3), When ditampilkan, Then nilai itu diberi penanda teks "tidak lengkap" dengan tautan ke `#kelengkapan`, dan beranda menampilkan banner "Sebagian biaya belum lengkap"; nilai `frozen` hanya diberi label, tanpa banner.
- AC-10.3 Given semua nilai `complete: true` dan `as_of` ≤ 6 jam, Then tidak ada banner.
- AC-10.4 Given banner muncul lewat skrip, Then banner berada di region `role="status"`.
- AC-10.5 Given skrip pemeriksa kesegaran dijalankan terhadap `/data/snapshot.v1.json`, When `run.as_of` > 6 jam, Then skrip keluar dengan kode ≠ 0 dan mencatat umurnya (dasar K1).

States: error (AC-10.1, AC-10.2), success (AC-10.3).

### US-11 Mode pemilik lokal

Sebagai Thomas, aku ingin versi lengkap dashboard di laptopku, supaya aku melihat project private dan tautan thread tanpa memublikasikannya.

Acceptance criteria:
- AC-11.1 Given build mode pemilik, When dibuka lokal, Then project private tampil dengan nama, "Menunggu Thomas" memuat tautan thread Amp, dan peringatan AC-01.7 tampil.
- AC-11.2 Given artefak mode pemilik, When skrip deploy repo (`infra/deploy`) dijalankan terhadapnya, Then skrip gagal sebelum unggah karena `snapshot.mode = "owner"` dan `<meta name="lantai-mode" content="owner">`, dan server hosting palsu mencatat 0 request.

States: sama dengan halaman publik.

### US-12 CI dan rilis (Should)

Acceptance criteria:
- AC-12.1 Given GitHub API palsu mengembalikan run workflow terakhir branch default `success` dan rilis `v0.1.0`, When detail project public dibuka, Then "CI: lulus" (teks + ikon) dan "Rilis: v0.1.0 · <tanggal>" dengan tautan.
- AC-12.2 Given repo tanpa workflow, Then "CI belum ada".
- AC-12.3 Given GitHub API 403 rate limit, Then nilai CI terakhir dipakai dengan `as_of` lama dan run `partial`.

### US-13 Ekspor data terbuka (Should)

Acceptance criteria:
- AC-13.1 Given build publik, When `/data/snapshot.v1.json` diunduh, Then file valid terhadap `contracts/snapshot.v1.schema.json` dan setiap angka di beranda sama dengan field JSON yang sesuai.
- AC-13.2 Given pemindai, When dijalankan pada file ekspor, Then 0 temuan.

### US-14 Feed perubahan (Should)

Acceptance criteria:
- AC-14.1 Given dua transisi fase baru project public, When `/id/feed.xml` dibaca, Then ada dua item "<project> masuk Fase N" dengan tanggal transisi.
- AC-14.2 Given transisi project private, Then item itu tidak ada.

### US-15 Halaman privasi (Must)

Acceptance criteria:
- AC-15.1 Given `/id/privasi/` dan `/en/privacy/`, When dibuka, Then halaman menyebut data yang diproses (log akses penyedia hosting, analitik request sisi server, email yang dikirim pengunjung), tujuan, lokasi penyedia, retensi, dan cara meminta akses/penghapusan; setiap halaman memiliki tautan ke halaman ini di footer.

### US-16 Tabel retro (Should)

Acceptance criteria:
- AC-16.1 Given project dengan tiga fase tertutup, When `lantai retro <project-id>` dijalankan di lingkungan pemilik, Then keluar tabel Markdown "Fase | Masuk | Durasi | Kredit | List price" yang angkanya sama dengan snapshot pemilik.

## 6. Non-functional requirements

| Area | Requirement |
| --- | --- |
| Performance | Beranda dan detail: LCP ≤ 2,0 s, TBT ≤ 200 ms, CLS < 0,1 (Lighthouse mobile default); JavaScript ≤ 50 KB gzip per halaman; HTML beranda ≤ 100 KB gzip. Satu run kolektor ≤ 1 menit untuk 5–20 project aktif (fase tertutup tidak dihitung ulang) |
| Reliability | Pembacaan usage disimpan append-only sejak run pertama di penyimpanan permanen di luar runner sementara, dengan backup harian ke lokasi kedua; restore diuji di M3 (AC-01.8). Snapshot publik selalu dari run valid terakhir. Kolektor idempotent |
| Jadwal dan kuota | Kolektor tiap 3 jam: 240 run/bulan × ≤ 1 menit = ≤ 240 menit/bulan (dibulatkan per job). Bila berjalan di Actions repo private org, sisa kuota 2.000 menit/bulan dipantau |
| Security | Situs publik statis, tanpa endpoint tulis. Kredensial hanya di lingkungan kolektor privat: token Amp pribadi dengan scope `amp.api:threads:view` saja; token GitHub fine-grained read-only (Metadata, Contents, Actions) untuk repo public saja; kredensial baca git factory (kemampuan read-only Amp-hosted git: ASUMSI, diverifikasi Phase 5). Rekonsiliasi K3 hanya berjalan di lingkungan privat, tidak di CI repo public. Header: CSP tanpa sumber pihak ketiga, `frame-ancestors 'none'`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, diuji e2e terhadap respons HTTP. OWASP: A01 allowlist, A02 tanpa secret di artefak, A05 header, A06 audit dependency, A09 log tanpa secret |
| Accessibility | WCAG 2.2 AA, termasuk: kontras teks 4,5:1 dan non-teks 3:1 (1.4.11) untuk ikon status dan timeline; status tidak hanya warna; reflow 320 px (1.4.10); fokus terlihat dan tidak tertutup (2.4.7, 2.4.11); target ≥ 24×24 px; tabel dengan header; pesan status di `role="status"` (4.1.3); nama aksesibel untuk tautan ikon. Bukti: axe-core 0 temuan critical/serious di e2e, test e2e navigasi keyboard-only dan viewport 320 px, satu sesi screen reader manual (NVDA atau VoiceOver) di M3 |
| Privacy | Inventaris di bawah. Tidak ada cookie dan tidak ada script pihak ketiga |
| Localization | `id` (default) dan `en`; URL `/id/…`, `/en/…`; WIB berlabel; US$ dua desimal (AC-09.2) |
| Hosting (syarat Phase 5) | Deploy dari artefak (bukan commit snapshot ke repo public), header kustom, HTTP 404 untuk path tidak ada, analitik request sisi server atau log akses, biaya US$0 untuk trafik target. GitHub Pages tidak memenuhi syarat header kustom |
| Maintainability | Schema snapshot, penyimpanan dan peta rute di `contracts/`, berversi; perubahan lewat ADR. Repo public hanya berisi fixture **sintetis** dengan schema sama; respons Amp API/CLI nyata hanya disimpan di lingkungan kolektor privat untuk contract test di sana |

**Inventaris data pribadi (UU 27/2022)**

| Data | Tujuan | Dasar pemrosesan | Disimpan di | Retensi | Publik? |
| --- | --- | --- | --- | --- | --- |
| Nama dan username GitHub Thomas | Identitas pemilik di halaman tentang | Persetujuan Thomas sebagai subjek data | Konfigurasi situs | Selama situs ada | Ya |
| ID/URL thread Amp | Menghitung biaya, tautan mode pemilik | Kepentingan sah pemilik factory (data milik Thomas sendiri) | Penyimpanan kolektor privat | Selama project ada | Tidak |
| IP dan user agent pengunjung di log/analitik penyedia hosting | Operasi dan K8 | Kepentingan sah; diumumkan di halaman privasi | Penyedia hosting (lokasi dicatat di halaman privasi; bila di luar Indonesia = transfer lintas negara, Pasal 56) | Kebijakan penyedia | Tidak |
| Email dan isi pesan pengunjung yang menghubungi | Membalas | Persetujuan (pengunjung mengirim) | Kotak surat Thomas | Hapus 12 bulan setelah percakapan selesai | Tidak |
| Nama author commit/PR | Tidak dikumpulkan | — | — | — | Tidak |

Permintaan akses/penghapusan lewat kontak di halaman privasi, dijawab dalam 3×24 jam. Kegagalan pelindungan data: (1) hentikan kolektor dan hapus artefak di hosting, purge cache; (2) rotasi semua kredensial yang mungkin terekspos; (3) bila ada data pribadi orang lain, beri tahu subjek data paling lambat 3×24 jam (Pasal 46); (4) catat insiden di docs/incidents/. Karena snapshot tidak pernah di-commit ke repo public (AC-03.7), riwayat git publik tidak perlu ditulis ulang; salinan yang sudah diunduh pihak lain tidak bisa ditarik, dan hal itu dicatat di laporan insiden.

## 7. External dependencies

| Dependency | Purpose | Access needed | Cost | Fallback |
| --- | --- | --- | --- | --- |
| Repo factory (Amp-hosted git) | Registry dan riwayatnya | Kredensial baca milik Thomas; clone penuh | 0 | Run `failed`, snapshot lama tetap |
| Amp External API `GET /api/v2/threads/{id}` dan `/usage` | `createdAt`, total biaya (JSON), P11 | Token pribadi `amp.api:threads:view` | ASUMSI: panggilan API tidak memakai kredit; diverifikasi M0 | Pembacaan terakhir, `complete: false` |
| Amp CLI `amp threads usage --start --end` | Biaya fase tertutup (P4, P11b) | `AMP_API_KEY` di lingkungan kolektor | Sama | Fase "belum dihitung" |
| GitHub REST API | CI, rilis repo public (Should) | Token fine-grained read-only | 0 (5.000 request/jam) | "Tidak diketahui" |
| Hosting statis | Situs publik | Akun penyedia sesuai syarat §6 (Phase 5, disetujui Phase 12) | Target US$0 | Artefak lokal |
| Lingkungan kolektor + penyimpanan permanen | Menjalankan kolektor | Dipilih di Phase 5 (kandidat: repo private org + Actions cron + penyimpanan di luar runner; atau mesin Thomas) | Target US$0–5/bulan | Dijalankan manual |
| Field registry `gate` dan `threads` | US-07, set thread lengkap | Persetujuan Thomas + perubahan skill factory-orchestrator | 0 | AC-07.4; hanya `thread_url` historis |
| Peserta uji | Sesi K6/K9 di M3 | 2–3 developer non-pemilik yang diajak Thomas (mengirim undangan = butuh persetujuan Thomas) | 0 | Tunda launch atau lapor K6/K9 "belum diuji" di gate Phase 12 |

## 8. Risks and assumptions

| Risk or assumption | Impact | Likelihood | Mitigation or validation |
| --- | --- | --- | --- |
| Usage hanya tersedia untuk thread berumur < 90 hari sejak dibuat (P12) | Tinggi: biaya thread orchestrator panjang berhenti terbaca | Pasti untuk thread > 90 hari | Simpan append-only sejak awal; peringatan handoff hari ke-75 (AC-01.7); label "tidak lengkap" (AC-01.6); spike M0 menguji apakah CLI berentang punya batas yang sama |
| Thread anak `create_thread` tidak tercatat | Sedang: biaya kurang | Sedang | Field `threads: []` (butuh persetujuan); audit K4 |
| Schema API atau teks CLI berubah | Tinggi | Sedang | Validasi schema, parser ketat, fixture nyata, `complete: false` bukan nol |
| Teks CLI dibulatkan ke sen (terverifikasi, P17) | Rendah: selisih kecil pada K3 | Pasti | Toleransi n × US$0,005; list price per rentang tersedia di output CLI (P17) |
| Makna `estimatedProviderCostAtListPriceUSD` untuk thread berbayar kredit tidak terdokumentasi | Sedang | Sedang | Spike M0 membandingkan thread subscription dan kredit; definisi di metodologi |
| Kredensial Amp bocor | Tinggi | Rendah | Scope `threads:view` saja; hanya di lingkungan privat; log diredaksi |
| Data privat terpublikasi | Tinggi | Sedang | Allowlist, pemindai sebelum unggah, denylist dari registry nyata, kanari, tanpa commit snapshot ke repo public |
| Agregat private membuka satu project lewat pengurangan atau perbandingan snapshot | Tinggi | Sedang | Total hanya dari project public (AC-03.6), uji tanpa sisa (AC-03.8), kohort n ≥ 3 diperbarui harian |
| Field `gate` tidak disetujui | Sedang: US-07 kosong | Sedang | AC-07.4; opsi alternatif di plan review |
| Kuota Actions private org habis | Sedang | Rendah | ≤ 300 menit/bulan; fallback mesin Thomas |
| Hosting gratis tanpa header kustom/analitik | Sedang | Sedang | Syarat hosting §6 sebagai kriteria Phase 5 |
| ASUMSI: publik peduli biaya factory AI | Sedang | Sedang | K8, K9 |

## 9. Milestones

| Milestone | Contents | Exit criteria |
| --- | --- | --- |
| M0 Foundations and contracts | Spike data nyata, schema snapshot v1, penyimpanan, peta rute, fixture sintetis dengan kanari, server palsu, kerangka monorepo, CI | Contract test hijau; CI hijau; setiap pertanyaan spike punya hasil dan cabang keputusan di ADR: (1) CLI berentang untuk thread > 90 hari: bila dibatasi, fase ditutup dan dihitung sebelum hari ke-90 (sudah default US-02); (2) list price per rentang: ada (P17); (3) panggilan API memakai kredit: bila ya dan > US$1/bulan, kurangi frekuensi; (4) kredensial baca git factory: bila hanya ada kredensial baca-tulis, catat sebagai blocker dan tanyakan Thomas (opsi: mirror read-only) |
| M1 Core loop | US-01, US-02, US-03, lalu satu halaman beranda sederhana (US-04) dari snapshot nyata; kolektor mulai berjalan terjadwal | e2e AC US-01–US-04 hijau; kolektor menyimpan pembacaan nyata; pemindai 0 temuan |
| M2 Supporting features | US-05–US-11, US-15 | e2e semua AC Must hijau; axe/keyboard/320 px hijau |
| M3 Polish and launch | Should sesuai garis potong, performa, restore, sesi uji K6/K9, audit K4, screen reader, launch kit | Dry run 72 jam di lingkungan kolektor privat dengan jadwal nyata, artefak dibangun tapi **tidak diunggah ke hosting publik**; K1-dry, K2-dry, K3, K5, K7 (Lighthouse pada server lokal) terpenuhi; K6, K9 lulus; lalu gate Phase 12 |

## 10. Budget

- Time: 19 hari kerja agent untuk Must (M0 4, M1 6, M2 5, M3 4) ≈ 3,8 minggu termasuk cadangan ± 15%; ditambah waktu kalender di luar kerja agent: dry run 72 jam dan sesi uji dengan peserta. Should hanya dikerjakan bila Must selesai lebih cepat (garis potong §4).
- Money: target US$0/bulan; batas < US$20/bulan. Domain opsional, harga dicek dan disetujui di Phase 12. Kredit Amp untuk membangun project ini dicatat di docs/progress.md.
