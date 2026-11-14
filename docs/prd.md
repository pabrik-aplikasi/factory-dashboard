# PRD — Factory dashboard publik

Status: draft
Delivery: showcase (situs statis publik + kolektor privat; penyedia hosting dipilih di Phase 5 dan disetujui di gate Phase 12)

Dokumen sumber: [brief](brief.md), [research](research.md), [personas](personas.md). Kode bukti primer P1–P10 merujuk ke tabel "Bukti primer" di research.md.

## 1. Problem and goals

- **Problem:** Progres dan biaya factory tersebar di registry privat, 135 repo, Linear dan ratusan thread Amp. Biaya hanya terlihat per thread dan memiliki dua angka berbeda (P3, P5). Thomas tidak punya satu layar untuk "apa yang menunggu aku dan berapa biayanya", dan publik tidak bisa melihat factory bekerja tanpa risiko membocorkan data privat (P2, P7).
- **Goals:**
  - G1. Thomas melihat posisi setiap project dan gate yang menunggunya dalam satu layar. → [Persona Thomas, pain 1–2](personas.md#persona-1-thomas-pemilik-factory)
  - G2. Biaya agent per project dan per fase dihitung dari data Amp asli, dengan kredit Amp dan estimasi list price dipisah dan dijelaskan. → P3–P5; [Persona Thomas, pain 3](personas.md#persona-1-thomas-pemilik-factory); [Persona Dimas, pain 2](personas.md#persona-2-dimas-tech-lead-yang-mengevaluasi-agent-ai)
  - G3. Publik bisa memverifikasi bahwa factory nyata: fase, waktu, biaya dan repo public, dalam Bahasa Indonesia dan Inggris. → [Persona Dimas, pain 1](personas.md#persona-2-dimas-tech-lead-yang-mengevaluasi-agent-ai); tema keluhan di research.md
  - G4. Tidak ada data privat yang keluar: nama repo private, ID/judul/isi thread, secret dan data orang lain tidak pernah masuk artefak publik. → P2, P7
  - G5. Data tidak pernah basi diam-diam: setiap angka punya waktu "data per" dan status sumber. → [Persona Thomas, pain 4](personas.md#persona-1-thomas-pemilik-factory)
- **Non-goals:**
  - Tidak mengubah registry, Linear atau thread dari dashboard (read-only terhadap semua sumber).
  - Tidak ada login, akun pengguna atau Keycloak di MVP. Mode pemilik berjalan lokal.
  - Bukan observability LLM: tidak ada trace, prompt, isi pesan atau judul thread.
  - Tidak realtime; pembaruan terjadwal (target per jam).
  - Bukan produk multi-factory/SaaS untuk pengguna Amp lain di versi ini.
  - Tidak mengonversi ke Rupiah di MVP (butuh sumber kurs; lihat Could).

## 2. Success metrics

| Metric | Target | Window | How measured |
| --- | --- | --- | --- |
| M1 Kesegaran data | `generated_at` snapshot publik ≤ 2 jam pada ≥ 95% pemeriksaan per jam | 14 hari setelah launch | Tabel `collector_runs` + skrip pemeriksa yang membaca `generated_at` dari snapshot publik tiap jam |
| M2 Cakupan biaya | ≥ 95% thread yang tercatat berhasil dibaca biayanya per run | 14 hari setelah launch | Rasio `threads_ok / threads_total` di tabel `collector_runs` |
| M3 Rekonsiliasi biaya | 100% project: total di dashboard = jumlah `amp threads usage` semua thread project, selisih ≤ US$0,01 | Setiap build (CI) dan sekali manual di M3 | Skrip rekonsiliasi membandingkan snapshot dengan pemanggilan ulang CLI |
| M4 Privasi | 0 temuan field terlarang di artefak publik | Semua build sejak M1 | Pemindai snapshot dan HTML di CI (jumlah temuan dicatat per build) |
| M5 Waktu menjawab "apa yang menunggu aku" | ≤ 10 detik dari membuka halaman, 3 dari 3 percobaan | Sesi uji dengan Thomas di M3 | Stopwatch pada sesi uji yang direkam; hasil dicatat di docs/test-report.md |
| M6 Performa mobile | Lighthouse mobile Performance ≥ 90 dan LCP ≤ 2,0 s di beranda dan detail project | Setiap build sejak M3 | Lighthouse CI dengan throttling mobile default |
| M7 Jangkauan publik | ≥ 200 pengunjung unik | 30 hari setelah launch | Analitik agregat tanpa cookie dari penyedia hosting; bila tidak tersedia, GitHub traffic API (`/repos/{owner}/{repo}/traffic/views`, unik 14 hari × 2) |
| M8 Dipakai untuk retro | Retro Phase 13 setiap project wave 1 mengutip biaya per fase dari dashboard | Sampai semua project wave 1 selesai Phase 13 | Hitung docs/retro.md wave 1 yang menautkan halaman/ekspor dashboard |

## 3. Personas

Primer: **Thomas, pemilik factory** (job: memutuskan ke mana perhatian dan uang pergi). Sekunder: **Dimas, tech lead yang mengevaluasi agent AI** (job: menilai apakah factory AI nyata dan berapa biayanya). Anti-persona: tim yang mencari observability LLM real-time. Detail di [personas.md](personas.md).

## 4. MVP scope (MoSCoW)

| Priority | Feature | Story IDs |
| --- | --- | --- |
| Must | Kolektor terjadwal: registry, biaya thread Amp, riwayat fase, status sumber | US-01, US-02 |
| Must | Snapshot publik ber-allowlist + pemindai privasi yang menggagalkan build | US-03 |
| Must | Beranda ringkasan: jumlah per fase dan status, gate menunggu, total biaya dua angka, "data per" | US-04 |
| Must | Daftar project dengan filter fase/status/kategori; project private sebagai agregat | US-05 |
| Must | Detail project: timeline fase, biaya per fase, link repo public | US-06 |
| Must | Halaman "Menunggu Thomas" (gate tertunda, urut paling lama) | US-07 |
| Must | Halaman metodologi biaya | US-08 |
| Must | Bahasa Indonesia dan Inggris | US-09 |
| Must | Peringatan data basi / sumber gagal | US-10 |
| Must | Mode pemilik lokal (data lengkap, tidak bisa di-deploy) | US-11 |
| Should | Status CI branch default dan rilis terakhir untuk repo public | US-12 |
| Should | Ekspor data terbuka `snapshot.v1.json` + schema | US-13 |
| Should | Feed perubahan fase (halaman + RSS/Atom) | US-14 |
| Could | Badge README per project (fase, biaya) | — |
| Could | Progres Linear (issue selesai/total) untuk project Phase 8+ | — |
| Could | Grafik biaya kumulatif per minggu | — |
| Could | Tampilan Rupiah dengan kurs harian bersumber dan bertanggal | — |
| Could | Gambar OG per project untuk dibagikan | — |
| Won't (this version) | Login/akun, Keycloak, edit registry, realtime/WebSocket, komentar, multi-factory, judul/isi thread, judul issue Linear publik | — |

## 5. User stories

Konvensi: "artefak publik" = semua file hasil build mode publik (HTML, JSON, feed, aset). "Fixture registry" = registry uji yang berisi project public dan private. Setiap AC dibuktikan oleh test e2e di `e2e/` yang menjalankan kolektor dan build nyata terhadap fixture, dengan CLI Amp dan GitHub API diganti server/binary palsu di batas proses (bukan mock unit yang diuji).

### US-01 Kolektor membaca registry dan biaya

Sebagai Thomas, aku ingin kolektor terjadwal membaca registry factory dan biaya setiap thread project, supaya angka di dashboard berasal dari sumber asli.

Acceptance criteria:
- AC-01.1 Given fixture registry dengan 3 project dan 4 thread, dan CLI usage palsu mengembalikan kredit US$0,34 dan list price US$52,86 untuk satu thread, When kolektor dijalankan, Then tabel `collector_runs` memuat satu run `ok` dengan `threads_ok = 4`, dan snapshot mode pemilik menampilkan kedua angka untuk thread itu secara terpisah.
- AC-01.2 Given registry mengganti `thread_url` project A dari thread X ke thread Y (handoff), When kolektor dijalankan lagi, Then biaya project A = biaya X + biaya Y, dan X tidak hilang dari mode pemilik.
- AC-01.3 Given CLI usage untuk satu thread keluar dengan error atau teks yang tidak bisa diparse, When kolektor dijalankan, Then run berstatus `partial`, thread itu memakai angka terakhir yang valid dengan tanda "angka lama per <waktu>", dan thread lain tetap diperbarui.
- AC-01.4 Given registry tidak bisa dibaca (repo factory tidak terjangkau), When kolektor dijalankan, Then run berstatus `failed`, snapshot sebelumnya tidak ditimpa, dan halaman menampilkan peringatan US-10.
- AC-01.5 Given `amp threads list` memuat thread yang tidak ada di registry, When kolektor dijalankan, Then thread itu tidak dibaca dan tidak muncul di database maupun snapshot (kolektor tidak memanggil `threads list`).

States: empty (registry belum punya project aktif → beranda "Belum ada project berjalan"), loading (tidak ada; situs statis), error (run failed → banner US-10), offline (tidak relevan untuk kolektor; situs statis tetap tampil dari cache browser), success (run ok + waktu).

### US-02 Riwayat fase dan biaya per fase

Sebagai Thomas, aku ingin tahu kapan setiap project masuk setiap fase dan berapa biaya tiap fase, supaya retro dan keputusan budget berdasar angka.

Acceptance criteria:
- AC-02.1 Given riwayat git registry berisi commit yang mengubah `phase` project A dari 1 ke 2 pada waktu T, When kolektor melakukan backfill, Then timeline A menampilkan "Fase 2 sejak T" dengan T dari waktu commit, dan sumbernya tertulis "commit registry".
- AC-02.2 Given A berada di fase 1 dari T1 sampai T2 dan CLI usage palsu mengembalikan biaya per rentang, When biaya per fase dihitung, Then biaya fase 1 = usage(thread, T1, T2) untuk semua thread A, dan jumlah biaya semua fase = total biaya A (selisih ≤ US$0,01).
- AC-02.3 Given fase berubah dua kali di antara dua run kolektor, When run berikutnya membaca riwayat git, Then kedua transisi tercatat dengan waktu commit masing-masing, bukan waktu run.
- AC-02.4 Given biaya tercatat sebelum transisi fase pertama yang diketahui, When ditampilkan, Then biaya itu masuk baris "Sebelum riwayat fase", bukan dibagi rata ke fase.

States: empty (project tanpa transisi → satu baris fase saat ini, "riwayat belum tersedia"), error (git history tidak terbaca → timeline disembunyikan dengan pesan, biaya total tetap tampil), success.

### US-03 Snapshot publik ber-allowlist

Sebagai Thomas, aku ingin artefak publik hanya berisi field yang di-allowlist, supaya tidak ada data privat yang bocor walaupun sumbernya berubah.

Acceptance criteria:
- AC-03.1 Given fixture registry dengan project private "201-payroll-umkm" dan repo private, When build mode publik dijalankan, Then string `payroll-umkm`, URL repo privatnya, dan nama project itu tidak ada di satu pun artefak publik, dan beranda menampilkan "1 project private" pada agregat.
- AC-03.2 Given snapshot berisi ID thread (`T-` + UUID), URL `ampcode.com/threads`, email, token berpola `gh[pousr]_`/`sk-`/`lin_api_`, atau path workspace, When pemindai privasi berjalan di build publik, Then build gagal dengan exit code ≠ 0 dan pesan menyebut file serta pola yang ditemukan.
- AC-03.3 Given registry mendapat field baru yang belum dikenal (misalnya `notes`), When build publik dijalankan, Then field itu tidak muncul di artefak publik (allowlist, bukan denylist).
- AC-03.4 Given Thomas menambah project private ke allowlist nama publik di konfigurasi, When build publik dijalankan, Then nama project itu tampil tetapi URL repo dan nama repo tetap tidak tampil.

States: error (build gagal, deploy tidak terjadi), success (artefak + laporan pemindai "0 temuan").

### US-04 Beranda ringkasan

Sebagai Dimas, aku ingin melihat ringkasan factory dalam satu layar, supaya aku cepat menilai skala dan kemajuannya.

Acceptance criteria:
- AC-04.1 Given snapshot dengan 5 project aktif di fase 1, 2, 2, 4 dan 6, When beranda dibuka, Then terlihat jumlah per fase 0–13 (fase 2 = 2), jumlah per status, total kredit Amp, total estimasi list price dengan label masing-masing, dan "Data per <waktu WIB>".
- AC-04.2 Given 2 project menunggu gate, When beranda dibuka, Then ada tautan "2 menunggu keputusan Thomas" ke halaman US-07.
- AC-04.3 Given snapshot tanpa project aktif, When beranda dibuka, Then tampil "Belum ada project berjalan" dan jumlah project terdaftar, tanpa angka biaya nol yang menyesatkan.
- AC-04.4 Given JavaScript dimatikan, When beranda dibuka, Then semua angka dan tautan tetap tampil.

States: empty (AC-04.3), loading (tidak ada; HTML statis), error (banner US-10), offline (halaman dari cache browser + banner "data per" tetap benar), success.

### US-05 Daftar project

Sebagai Dimas, aku ingin daftar project yang bisa difilter, supaya aku menemukan project yang menarik.

Acceptance criteria:
- AC-05.1 Given 3 project public dan 2 private di snapshot, When halaman daftar dibuka, Then 3 baris project public tampil dengan nama, kategori, fase, status, aktivitas terakhir dan biaya, ditambah satu baris agregat "2 project private".
- AC-05.2 Given filter fase = 2 dipilih, When diterapkan, Then hanya project fase 2 yang tampil, URL memuat `?fase=2`, dan membuka URL itu langsung menampilkan hasil yang sama.
- AC-05.3 Given filter yang tidak cocok dengan project mana pun, When diterapkan, Then tampil "Tidak ada project yang cocok" dan tombol reset filter.
- AC-05.4 Given JavaScript dimatikan, When halaman dibuka, Then seluruh daftar tampil tanpa filter dan tetap bisa diurutkan lewat tautan kolom.

States: empty (registry kosong → "Belum ada project terdaftar"), error (banner US-10), success.

### US-06 Detail project

Sebagai Dimas, aku ingin halaman detail tiap project public, supaya aku bisa melihat perjalanan dan biayanya lalu membuka repo-nya.

Acceptance criteria:
- AC-06.1 Given project public A dengan transisi fase 0→1→2, When `/id/project/<slug>` dibuka, Then timeline menampilkan tiga fase dengan tanggal masuk, durasi, biaya kredit dan list price per fase, dan tautan ke repo GitHub public A.
- AC-06.2 Given slug project private atau slug yang tidak ada, When URL dibuka, Then halaman 404 tampil dengan tautan kembali ke daftar, dan nama project private tidak muncul.
- AC-06.3 Given project berstatus `killed` atau `blocked`, When detail dibuka, Then status tampil dengan teks (bukan warna saja) dan alasan singkat jika ada di allowlist.

States: empty (project tanpa biaya → "Belum ada biaya tercatat"), error (banner), success.

### US-07 Menunggu Thomas

Sebagai Thomas, aku ingin daftar gate yang menunggu keputusanku, urut dari yang paling lama, supaya tidak ada project yang macet karena aku.

Acceptance criteria:
- AC-07.1 Given registry mencatat gate tertunda untuk A (Phase 4, sejak T1) dan B (Phase 6, sejak T2 < T1), When halaman dibuka, Then B tampil di atas A dengan nama gate dan lama menunggu.
- AC-07.2 Given gate tertunda pada project private, When halaman publik dibuka, Then tampil sebagai "1 project private menunggu", sedangkan mode pemilik menampilkan nama dan tautan thread-nya.
- AC-07.3 Given tidak ada gate tertunda, When halaman dibuka, Then tampil "Tidak ada keputusan yang menunggu".
- AC-07.4 Given registry belum punya data gate untuk project yang berada di fase gate, When halaman dibuka, Then project itu tidak ditebak sebagai "menunggu".

States: empty (AC-07.3), error (banner), success.

### US-08 Metodologi biaya

Sebagai Dimas, aku ingin penjelasan angka biaya, supaya aku tidak salah mengutip.

Acceptance criteria:
- AC-08.1 Given halaman metodologi dibuka, When dibaca, Then halaman menjelaskan kredit Amp vs estimasi list price, bahwa keduanya tidak dijumlahkan, apa yang tidak dihitung (langganan bulanan, waktu Thomas, hosting), dan tanggal versi metodologi.
- AC-08.2 Given setiap angka biaya di situs, When pengguna mengaktifkan tautan info di sebelahnya, Then ia sampai ke bagian metodologi yang relevan.

States: success (statis).

### US-09 Dua bahasa

Sebagai Dimas, aku ingin memilih Bahasa Indonesia atau Inggris, supaya aku dan audiens global bisa membaca.

Acceptance criteria:
- AC-09.1 Given `/id/` dibuka, When pengguna memilih English, Then ia sampai ke halaman setara di `/en/`, dan `lang` HTML berubah menjadi `en`.
- AC-09.2 Given angka 1234,5 dan tanggal 14 Nov 2026, When ditampilkan, Then format ID = "1.234,50" dan "14 Nov 2026", format EN = "1,234.50" dan "14 Nov 2026"; waktu ditampilkan dalam WIB dengan label "WIB" di kedua bahasa.
- AC-09.3 Given string terjemahan yang hilang, When build dijalankan, Then build gagal dengan nama key yang hilang.

States: success; error (build gagal, AC-09.3).

### US-10 Data basi dan sumber gagal

Sebagai Thomas, aku ingin peringatan bila data basi atau sumber gagal, supaya publik tidak melihat angka salah tanpa konteks.

Acceptance criteria:
- AC-10.1 Given `generated_at` lebih tua dari 6 jam saat halaman dibuka, When halaman ditampilkan, Then banner "Data terakhir diperbarui <waktu>, mungkin sudah tidak akurat" tampil (dihitung di browser dari `generated_at`; tanpa JavaScript, waktu "data per" tetap tampil).
- AC-10.2 Given run terakhir berstatus `partial` untuk sumber biaya, When halaman ditampilkan, Then banner menyebut "Biaya sebagian belum diperbarui" dan angka yang lama diberi tanda.
- AC-10.3 Given semua sumber ok dan data < 6 jam, When halaman dibuka, Then tidak ada banner.

States: error (AC-10.1, AC-10.2), success (AC-10.3).

### US-11 Mode pemilik lokal

Sebagai Thomas, aku ingin versi lengkap dashboard di mesinku, supaya aku melihat project private dan tautan thread tanpa memublikasikannya.

Acceptance criteria:
- AC-11.1 Given build mode pemilik, When dibuka secara lokal, Then project private tampil dengan nama, dan "Menunggu Thomas" memuat tautan thread Amp.
- AC-11.2 Given artefak mode pemilik, When perintah deploy/publish dijalankan terhadapnya, Then perintah gagal karena artefak bertanda `owner` (penanda di snapshot dan meta HTML), dan pemindai privasi melaporkannya.

States: sama dengan halaman publik.

### US-12 CI dan rilis (Should)

Sebagai Dimas, aku ingin status CI dan rilis terakhir repo public, supaya aku tahu project itu benar-benar dibangun.

Acceptance criteria:
- AC-12.1 Given GitHub API palsu mengembalikan run workflow terakhir branch default `success` dan rilis `v0.1.0`, When detail project dibuka, Then tampil "CI: lulus" (teks + ikon) dan "Rilis: v0.1.0 · <tanggal>" dengan tautan.
- AC-12.2 Given repo tanpa workflow, When ditampilkan, Then tampil "CI belum ada", bukan "gagal".
- AC-12.3 Given GitHub API mengembalikan 403 rate limit, When kolektor berjalan, Then nilai CI terakhir dipakai dengan tanda waktu dan run berstatus `partial`.

### US-13 Ekspor data terbuka (Should)

Sebagai peneliti/Dimas, aku ingin mengunduh snapshot publik, supaya aku bisa menganalisis sendiri.

Acceptance criteria:
- AC-13.1 Given build publik, When `/data/snapshot.v1.json` diunduh, Then file valid terhadap `contracts/snapshot.v1.schema.json` dan isinya sama dengan angka di halaman.
- AC-13.2 Given pemindai privasi, When dijalankan pada file ekspor, Then 0 temuan.

### US-14 Feed perubahan (Should)

Sebagai Dimas, aku ingin feed perubahan fase, supaya aku bisa mengikuti factory tanpa membuka dashboard tiap hari.

Acceptance criteria:
- AC-14.1 Given dua transisi fase baru sejak build sebelumnya, When `/id/feed.xml` dibaca, Then ada dua item dengan judul "<project> masuk Fase N" dan tanggal transisi.
- AC-14.2 Given transisi pada project private, When feed publik dibuat, Then item tersebut tidak ada.

## 6. Non-functional requirements

| Area | Requirement |
| --- | --- |
| Performance | Beranda dan detail: LCP ≤ 2,0 s, CLS < 0,1, INP ≤ 200 ms pada Lighthouse mobile (Moto G Power, 4G lambat simulasi); JavaScript ≤ 50 KB gzip per halaman; HTML beranda ≤ 100 KB gzip. Kolektor ≤ 5 menit untuk 135 project dan ≤ 300 thread |
| Reliability | Snapshot publik selalu dari run terakhir yang valid; run gagal tidak menimpa snapshot. Riwayat (fase, biaya per thread, run) disimpan permanen dan dibackup harian; restore diuji sekali di M3. Kolektor idempotent: menjalankan dua kali berturut-turut tidak menggandakan biaya |
| Security | Tidak ada endpoint tulis publik; situs publik statis. Kredensial Amp, GitHub dan Linear hanya di lingkungan kolektor privat, tidak pernah di secret workflow repo public. Token GitHub read-only (metadata, actions, contents read) untuk repo yang dibutuhkan. CSP tanpa script pihak ketiga; tanpa inline script kecuali hash. Pemindai privasi (US-03) dan audit dependency di CI. OWASP Top 10 yang relevan untuk situs statis: A01 (allowlist), A02 (tidak ada secret di artefak), A05 (header keamanan), A06 (dependency), A09 (log run tanpa secret) |
| Accessibility | WCAG 2.2 AA: kontras ≥ 4,5:1, status tidak hanya dengan warna, navigasi keyboard penuh, target sentuh ≥ 24×24 px, tabel dengan header, axe-core 0 pelanggaran serius di e2e |
| Privacy | Lihat inventaris data di bawah. Situs publik tidak memasang cookie atau analitik pihak ketiga. UU PDP: satu-satunya data pribadi adalah nama dan username GitHub Thomas, yang dipublikasikan atas keputusannya sendiri sebagai pemilik |
| Localization | `id` (default) dan `en`; URL `/id/…` dan `/en/…`; waktu dalam WIB (UTC+7) dengan label; mata uang US$ dua desimal, nilai < 0,005 tampil "< US$0,01" (EN "< US$0.01") |
| Maintainability | Schema snapshot dan schema DB di `contracts/`, berversi; perubahan lewat ADR. Fixture CLI Amp disimpan di repo untuk contract test parser |

**Inventaris data pribadi**

| Field | Kenapa | Disimpan di | Retensi | Publik? |
| --- | --- | --- | --- | --- |
| Nama Thomas, username GitHub | Identitas pemilik factory di halaman "tentang" | Konfigurasi situs | Selama situs ada | Ya, atas keputusan Thomas |
| ID dan URL thread Amp | Menghitung biaya, tautan mode pemilik | DB kolektor privat | Selama project ada | Tidak |
| Nama author commit/PR | Tidak dikumpulkan | — | — | Tidak |
| IP/user agent pengunjung | Tidak dikumpulkan oleh aplikasi; log akses penyedia hosting mengikuti kebijakan penyedia | Penyedia hosting | Kebijakan penyedia | Tidak |

Permintaan akses/penghapusan: karena tidak ada data pengunjung, permintaan dijawab lewat kontak di halaman "tentang". Kebocoran (misalnya nama repo private terpublikasi): rollback snapshot dalam 1 jam, catat insiden di docs/, beri tahu pihak terdampak bila ada data orang lain.

## 7. External dependencies

| Dependency | Purpose | Access needed | Cost | Fallback |
| --- | --- | --- | --- | --- |
| Repo factory (Amp-hosted git) | Registry dan riwayat fase | Kredensial baca git milik Thomas di lingkungan kolektor | 0 | Salinan registry terakhir; run `failed` |
| Amp CLI `amp threads usage` | Biaya per thread dan per rentang | Login/API key Amp milik Thomas di lingkungan kolektor | Kredit Amp per panggilan tidak diketahui; ASUMSI ≈ 0 | Angka terakhir + tanda basi; entri manual `cost_usd` registry |
| GitHub REST API | CI, rilis, aktivitas repo public (Should) | Token read-only | 0 | Tanpa CI; tampilkan "tidak diketahui" |
| Linear API | Progres issue (Could) | API key read-only | 0 | Tidak ditampilkan |
| Hosting statis | Situs publik | Akun penyedia (dipilih Phase 5, disetujui Phase 12) | Target US$0; batas < US$20/bulan | Artefak lokal (showcase offline) |
| Lingkungan kolektor terjadwal | Menjalankan kolektor + DB | Mesin/runner privat milik Thomas (dipilih Phase 5) | Target US$0–5/bulan | Dijalankan manual oleh Thomas |
| Field gate di registry | Sumber "Menunggu Thomas" | Persetujuan Thomas untuk menambah field opsional `gate` di schema registry | 0 | US-07 hanya menampilkan project dengan `status: blocked` |

## 8. Risks and assumptions

| Risk or assumption | Impact | Likelihood | Mitigation or validation |
| --- | --- | --- | --- |
| Format teks `amp threads usage` berubah (P6) | Tinggi: biaya salah/terlewat | Sedang | Parser ketat dengan fixture; teks tak dikenal → `partial`, bukan angka nol; contract test di CI; tanyakan Amp apakah ada output JSON |
| Kredensial Amp bocor dari kolektor (akses semua thread Thomas) | Tinggi | Rendah | Kolektor di lingkungan privat, tidak pernah di Actions repo public; log diredaksi; `amp orb redact-output` / redaksi sendiri |
| Data privat terpublikasi | Tinggi | Sedang | Allowlist + pemindai yang menggagalkan build (US-03); review keamanan di Phase 10 |
| Salah tafsir biaya (subscription vs kredit) | Sedang: reputasi | Tinggi | Dua angka terpisah, metodologi (US-08), tidak pernah dijumlahkan |
| Thomas tidak mau biaya tertentu publik | Sedang | Sedang | Konfigurasi `public_costs: true/false` per project; ditanyakan di gate Phase 4 |
| Registry diubah thread lain dengan format berbeda | Sedang | Sedang | Parser toleran field tambahan, validasi schema, run `partial` dengan nama project bermasalah |
| Transisi fase tidak di-commit terpisah (beberapa fase dalam satu commit) | Rendah: timeline kasar | Tinggi | Pakai waktu commit; tampilkan "beberapa fase dalam satu langkah"; biaya fase pendek bisa 0 |
| Field `gate` belum disetujui | Sedang: US-07 tidak lengkap | Sedang | Fallback `status: blocked`; ajukan di gate Phase 4 |
| Jadwal kolektor terlewat (mesin mati) | Sedang | Sedang | Banner basi (US-10), pemeriksa kesegaran (M1) |
| ASUMSI: publik peduli angka biaya factory AI | Sedang | Sedang | Validasi dengan M7; uji bacaan dengan 3 developer di M3 |
| ASUMSI: update per jam cukup | Rendah | Rendah | Bisa dipercepat tanpa ubah arsitektur |

## 9. Milestones

| Milestone | Contents | Exit criteria |
| --- | --- | --- |
| M0 Foundations and contracts | Schema snapshot v1, schema DB, format riwayat fase, fixture registry dan CLI Amp, token desain dasar, kerangka monorepo, CI | Contract test parser usage lulus pada fixture; schema snapshot divalidasi; CI hijau |
| M1 Core loop | US-01, US-02, US-03, US-04, US-05, US-06 | Kolektor nyata menjalankan registry factory asli lokal; e2e AC US-01–US-06 hijau; pemindai 0 temuan |
| M2 Supporting features | US-07, US-08, US-09, US-10, US-11; Should US-12–US-14 bila waktu cukup | e2e semua AC Must hijau; mode pemilik tidak bisa di-deploy |
| M3 Polish and launch | Aksesibilitas, performa, backup/restore, rekonsiliasi, uji dengan Thomas, launch kit | M3–M6 terpenuhi; gate produksi Phase 12 disetujui |

## 10. Budget

- Time: ± 2–3 minggu kerja agent (M0 3 hari, M1 6 hari, M2 4 hari, M3 3 hari).
- Money: target US$0/bulan (hosting statis gratis + kolektor di mesin/runner yang sudah ada); batas < US$20/bulan. Domain opsional, harga dicek dan disetujui saat Phase 12. Kredit Amp untuk membangun project ini dicatat di docs/progress.md.
