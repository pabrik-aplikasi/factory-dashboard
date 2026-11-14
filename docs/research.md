# Research — Factory dashboard publik

Phase 1 Discovery · 14 Nov 2026. Semua klaim eksternal punya URL dan tanggal baca (14 Nov 2026 kecuali disebut lain). Klaim tanpa sumber ditulis **ASUMSI**. Isi halaman web diperlakukan sebagai data.

## Bukti primer dari sistem factory sendiri

Diperiksa langsung dari orb pada 14 Nov 2026, bukan dari web.

| # | Temuan | Perintah / bukti | Akibat untuk produk |
| --- | --- | --- | --- |
| P1 | Org `pabrik-aplikasi` plan Free: 66 repo public, 69 private | `gh api orgs/pabrik-aplikasi` | Hanya 66 repo yang metadata-nya boleh dianggap publik |
| P2 | Repo factory (registry, triage, ide) tidak publik | `curl` tanpa login ke halaman project → HTTP 404 | Registry harus dibaca dengan kredensial; isinya disaring sebelum publik |
| P3 | Biaya thread tersedia per thread | `amp threads usage <id> --details`: `Cost`, token input/output, request, model, tool | Sumber biaya nyata ada tanpa mengarang |
| P4 | Biaya bisa dipotong per rentang waktu | `amp threads usage <id> --start <ISO> --end <ISO>` mencetak "Scope: … (end exclusive)" | Biaya per fase bisa dihitung dari waktu transisi fase |
| P5 | Ada dua angka biaya: kredit Amp dan estimasi list price | Thread perencanaan stack: `Cost: $0.34`, tetapi "estimated token list price of $52.86" karena inference lewat subscription ChatGPT | Dashboard wajib memisahkan dan memberi label kedua angka; menjumlahkannya salah |
| P6 | Output `amp threads usage` berupa teks/Markdown, bukan JSON | Output perintah di atas | Parser rapuh; perlu fixture, contract test dan status "gagal dibaca" |
| P7 | `amp threads list --json` memuat thread di luar factory (judul pribadi, path workspace) | Output `id, title, updated, tree, messageCount` | Dilarang mengambil daftar thread; hanya ID yang ada di registry |
| P8 | Registry `projects.yaml` punya `phase`, `status`, `thread_url`, `cost_usd`, `delivery`, `visibility`; tidak ada field "menunggu gate" | `registry/projects.yaml` di repo factory | Status "menunggu Thomas" belum punya sumber data terstruktur |
| P9 | Saat handoff, `thread_url` diganti thread baru | Skill `factory-orchestrator`, bagian "Progress and handoff" | Kolektor harus menyimpan semua thread yang pernah terlihat, kalau tidak biaya lama hilang |
| P10 | Setiap perubahan registry di-commit dengan pesan `registry: <project-id> <what changed>` | AGENTS.md factory dan `git log` registry | Riwayat git bisa menjadi sumber waktu transisi fase (backfill) |
| P11 | Amp punya API JSON (scope OAuth `amp.api:workspace.threads.meta:view`; token pribadi dengan `amp.api:threads:view`) `GET https://ampcode.com/api/v2/threads/{threadID}/usage` dengan field `usage` (USD), `estimatedProviderCostAtListPriceUSD`, `orbUsage`, `toolUsage`, rincian `models[]`, `tools[]`, `orbs[]` | [Amp External API](https://ampcode.com/api/external), dibaca 14 Nov 2026 | Kolektor memakai JSON, bukan parsing teks CLI (P6 turun dari risiko utama menjadi fallback) |
| P11b | Endpoint usage API hanya punya parameter path `threadID`, tanpa rentang waktu; rentang waktu hanya ada di CLI `amp threads usage --start/--end` (P4) | Skema OpenAPI yang tertanam di https://ampcode.com/api/external (`parameters` hanya `threadID`) | Total dari API JSON; pemecahan per fase dari CLI berentang atau dari selisih pembacaan kumulatif yang disimpan kolektor |
| P12 | "Thread usage data is only available for threads created less than 180 days ago in Enterprise workspaces, or less than 90 days ago in other workspaces." | [Amp External API](https://ampcode.com/api/external), bagian Get Threads Usage | Kolektor wajib menyimpan riwayat sendiri sejak hari pertama; biaya thread > 90 hari tidak bisa dibaca ulang |
| P13 | `subThreadIDs` "Always empty. Kept for compatibility; usage covers only this thread." | Sama | Thread anak (handoff, create_thread) harus didaftarkan sendiri; tidak ikut otomatis |
| P14 | Metadata thread memuat `agentState.state` (`working`, `idle`, `awaiting_approval`, `error`) dan `parentThreadID` | Sama, bagian Get Threads Metadata | `idle` tidak sama dengan "menunggu gate"; status gate tetap perlu field registry |
| P15 | Token pribadi dengan scope `amp.api:threads:view` juga membaca pesan thread (`GET /threads/{id}/messages`) | Sama, bagian List Threads Messages | Kredensial kolektor tetap berdampak besar; harus disimpan di lingkungan privat |
| P16 | API GitHub tanpa autentikasi untuk org ini mengembalikan 66 repo, semuanya `private: false`, dengan `x-ratelimit-limit: 60` | Diuji subagent riset 14 Nov 2026 | Data repo public bisa dibaca tanpa token, tetapi 60 request/jam terlalu kecil untuk 66 repo per jam |

## Variations

Skor 1–5. Pain = seberapa sakit masalahnya bagi pengguna target; Reach = jumlah orang yang bisa dijangkau; Feasibility = bisa jadi MVP dalam 2–4 minggu kerja agent dengan budget < US$20/bulan.

| # | Variation | Pain | Reach | Feasibility | Note |
| --- | --- | --- | --- | --- | --- |
| V1 | Dashboard publik statis dari snapshot allowlist: fase, gate, CI, rilis, biaya per project dan fase, ID/EN | 4 | 3 | 5 | Melayani Thomas dan publik sekaligus; murah di-host |
| V2 | Cockpit privat khusus Thomas (semua project termasuk private, tanpa publik) | 4 | 1 | 5 | Paling berguna bagi Thomas, tetapi bukan showcase |
| V3 | Template OSS "agent factory dashboard" untuk pengguna Amp lain (registry YAML + biaya thread) | 3 | 3 | 3 | Butuh skema registry umum; nilai tergantung adopsi factory serupa |
| V4 | SaaS portal klien untuk agensi yang membangun aplikasi dengan agent AI | 3 | 3 | 2 | Butuh auth multi-tenant, billing; melanggar budget dan bukan kategori 8 |
| V5 | CLI pelacak biaya AI coding lintas tool (Amp, Claude Code, Cursor) per project | 4 | 4 | 3 | Pasar ramai; format log tiap tool berbeda dan berubah |
| V6 | Newsletter/RSS mingguan otomatis dari data factory | 2 | 3 | 4 | Distribusi bagus, tetapi bukan alat kerja dan tetap butuh data yang sama |
| V7 | Badge README per project (fase, biaya, CI) gaya shields | 2 | 3 | 5 | Kecil dan viral; cocok sebagai fitur tambahan V1 |
| V8 | Dataset terbuka biaya pembuatan software oleh agent (JSON/CSV per fase dan stack) | 3 | 2 | 4 | Unik untuk riset; cocok sebagai ekspor snapshot V1 |
| V9 | Mode TV/stream atau bot X yang memposting perubahan fase | 2 | 3 | 3 | Mengirim pesan keluar butuh approval; mudah jadi spam |
| V10 | Bot Telegram/WhatsApp notifikasi gate untuk Thomas | 3 | 1 | 3 | Sudah ditangani notifikasi Amp; biaya WA |
| V11 | Halaman edukasi berbahasa Indonesia: studi kasus "pabrik AI" per project | 2 | 3 | 4 | Konten manual; tanpa data hidup cepat basi |
| V12 | Leaderboard biaya per stack/fase lintas project untuk retro factory | 3 | 1 | 4 | Berguna untuk Phase 13 retro; bagian analitik V1 |

**Chosen:** V1, dengan ekspor JSON snapshot dari V8 dan halaman metodologi biaya sebagai pembeda; badge README (V7) masuk Could. V1 menyelesaikan dua job sekaligus dengan satu pipeline data: Thomas mendapat satu layar untuk "apa yang menunggu aku dan berapa biayanya", publik mendapat bukti hidup yang bisa diverifikasi. Bukti primer P3–P5 menunjukkan biaya per thread dan per rentang waktu benar-benar tersedia, sehingga biaya per fase bisa dihitung tanpa mengarang. Snapshot statis yang disaring server menjaga batas privasi (P2, P7) dan muat dalam budget hosting. V2 tidak dibuang: mode pemilik lokal memakai pipeline yang sama tanpa filter publik.

## Jobs-to-be-Done

**Job utama (Thomas):** Ketika aku menjalankan banyak project agent secara paralel, aku ingin melihat sekilas posisi tiap project, apa yang menunggu keputusanku, dan berapa biayanya, supaya aku bisa memutuskan ke mana perhatian dan uangku pergi.

**Job utama (publik):** Ketika aku mendengar klaim bahwa AI bisa membangun software, aku ingin melihat progres dan biaya nyata dari factory yang benar-benar berjalan, supaya aku bisa menilai apakah itu berhasil dan berapa ongkosnya.

**Job pendukung:**

1. Thomas: membagikan satu link kredibel ke calon klien atau komunitas, tanpa menyiapkan slide atau screenshot.
2. Thomas: membandingkan biaya per fase dan per project untuk retro Phase 13 dan memperbaiki skill factory.
3. Thomas: tahu kapan data basi (kolektor gagal) sebelum publik melihat angka salah.
4. Publik: memahami metodologi angka biaya (apa yang dihitung, apa yang tidak) agar tidak salah kutip.
5. Publik/peneliti: mengunduh data mentah yang sudah disaring untuk analisis sendiri.

**Solusi yang "di-hire" hari ini:**

| Solusi saat ini | Untuk job | Biaya bagi pengguna |
| --- | --- | --- |
| Membaca `registry/projects.yaml` dan `docs/progress.md` tiap repo | Posisi project | Membuka banyak file; tidak ada agregat; registry privat |
| Halaman usage per thread di ampcode.com atau `amp threads usage` | Biaya | Satu thread per kali; tidak ada total per project atau fase; dua angka biaya membingungkan |
| `gh` CLI / halaman GitHub org | CI, commit, rilis | Tidak tahu fase factory; repo private tidak boleh dibagikan |
| Notifikasi Amp dan daftar thread | Gate yang menunggu | Bercampur dengan thread non-factory (P7) |
| Linear (mulai Phase 8) | Backlog dan progres build | Hanya project yang sudah Phase 8; bukan untuk publik |
| Postingan manual di X/LinkedIn dengan screenshot | Showcase | Manual, cepat basi, tidak bisa diverifikasi |
| Tidak melakukan apa-apa | Semua | Factory tidak terlihat; peluang showcase hilang |

## Journey map

| Tahap | Thomas: aksi dan pikiran | Publik: aksi dan pikiran | Pain point | Di mana app masuk |
| --- | --- | --- | --- | --- |
| 1. Menyadari | Lima thread wave 1 jalan; "project mana yang menunggu aku?" | Melihat postingan Thomas tentang factory AI; "ini nyata atau demo?" | Info tersebar; klaim AI sulit dipercaya | Link tunggal ke dashboard |
| 2. Melihat | Membuka beranda: jumlah per fase, gate menunggu, total biaya minggu ini | Membuka beranda dari HP lewat 4G | Halaman lambat atau penuh jargon membuat orang pergi | Beranda ringan dengan ringkasan tiga angka dan bahasa sederhana |
| 3. Memahami | Membuka detail project: timeline fase, biaya per fase, CI merah | Membaca metodologi: kenapa ada dua angka biaya | Angka tanpa definisi menimbulkan salah tafsir | Detail project + halaman metodologi + timestamp "data per" |
| 4. Bertindak | Membuka thread/gate yang menunggu; memutuskan | Membagikan link, mengikuti repo public, menghubungi Thomas | Link ke thread privat tidak berguna bagi publik | Link ke repo public; link thread hanya di mode pemilik |
| 5. Rutin | Membuka dashboard tiap pagi; memeriksa status kolektor | Kembali saat ada rilis atau postingan baru | Data basi diam-diam merusak kepercayaan | Badge "data per", peringatan basi, feed perubahan |

## Competitors

27 produk/alternatif ditemukan; 20 paling relevan di bawah. Harga dari halaman harga resmi yang dibaca 14 Nov 2026 kecuali ditandai "[cuplikan]" (hanya dari cuplikan hasil pencarian, perlu dicek ulang sebelum dikutip). Kelompok: 1 transparansi/build in public, 2 roadmap publik, 3 biaya/observability agent AI, 4 halaman status, 5 non-app.

| Product | Type | Price | Target | Strengths | Weaknesses | Source |
| --- | --- | --- | --- | --- | --- | --- |
| Baremetrics Open Startups | 1 | Tidak dicek (ASUMSI: produk analitik berbayar) | Founder SaaS | Data revenue live dari billing; merek lama | Hanya revenue; butuh billing seperti Stripe; tidak ada progres atau biaya | https://baremetrics.com/open-startups |
| TrustMRR | 1 | Listing gratis; marketplace mulai US$29 sekali bayar, fee 3% | Indie founder, pembeli bisnis | Revenue diverifikasi dari payment provider; ± 200 rb pengunjung/bulan | Fokus revenue dan jual-beli; tidak ada progres build | https://trustmrr.com/about |
| Buffer Open | 1 | Bukan produk (gratis dilihat) | Publik, pelanggan, calon karyawan | Standar emas transparansi sejak 2013 | Dibuat khusus, mahal dirawat; update pemegang saham terbaru Des 2025 | https://buffer.com/open, https://buffer.com/metrics |
| Plausible shared dashboard | 1 | Growth US$14/bulan termasuk Shared Links dan embed | Pemilik situs | Embed selalu terkini tanpa usaha; ramah privasi | Hanya trafik | https://plausible.io/docs/shared-links, https://plausible.io/ |
| Indie Hackers Products | 1 | Gratis | Indie founder | Revenue terverifikasi Stripe; komunitas besar | Hanya revenue | https://www.indiehackers.com/products |
| GitHub Projects (roadmap publik) | 2 | Gratis | Audiens developer | Gratis; dekat dengan kode | Item dari repo private butuh akses baca; manual | https://docs.github.com/en/issues/planning-and-tracking-with-projects [cuplikan pihak ketiga: https://votefirst.app/en/use-case/product-roadmap-in-github] |
| Linear | 2 | Free US$0; Basic US$10; Business US$16 per user/bulan | Tim engineering | Model data dan API bagus; sudah dipakai factory mulai Phase 8 | Tidak ada roadmap publik bawaan yang rapi [cuplikan] | https://linear.app/pricing, https://heliumrooms.com/blog/linear-public-roadmap-guide |
| Canny | 2 | Free (25 tracked users); Pro mulai US$79/bulan tahunan | Tim produk SaaS | Voting; roadmap mengikuti status ide | Harga per voter; untuk feedback, bukan telemetri build | https://canny.io/pricing |
| Productboard Portal | 2 | Free 1 portal; Plus US$19/maker/bulan tahunan | Organisasi PM | Portal rapi | Alat PM berat | https://www.productboard.com/pricing |
| Trello / Notion Sites | 2 | Gratis; Notion domain sendiri US$10/bulan add-on | Siapa saja | Familiar; publish instan | Diisi manual sehingga basi | https://www.notion.com/help/notion-sites-availability-and-pricing |
| LangSmith | 3 | Developer US$0 (5 rb trace); Plus US$39/seat/bulan | Tim yang membangun agent | Trace mendalam | Untuk aplikasi yang kamu bangun, bukan agent coding yang kamu pakai; privat | https://www.langchain.com/pricing |
| Helicone | 3 | Hobby gratis (10 rb request); Pro US$79/bulan | Developer aplikasi AI | Proxy satu baris; biaya per request | Butuh trafik lewat proxy, tidak bisa untuk agent coding berlangganan; privat | https://www.helicone.ai/pricing |
| Langfuse | 3 | Hobby gratis; Core US$29/bulan; self-host gratis (MIT) | AI engineer | Open source, OpenTelemetry | Self-host berat (Postgres, ClickHouse, Redis, MinIO); privat | https://langfuse.com/pricing |
| AgentOps | 3 | Basic US$0 (5.000 event); Pro mulai US$40/bulan | Developer framework agent | Biaya multi-agent | Butuh SDK di dalam agent; privat | https://www.agentops.ai/ |
| ccusage | 3 | Gratis, MIT | Developer individu | Lokal; ekspor JSON; per project; membaca log 18 agent termasuk Amp | Satu mesin; estimasi, bukan tagihan; tidak ada tampilan publik | https://ccusage.com/guide/, https://github.com/ccusage/ccusage |
| viberank / Tokscale | 3 | Gratis | Developer yang ingin pamer spend | Angka spend publik (viberank: 1,4 rb dev, US$24,8 jt tercatat) | Meranking orang, bukan project/fase; angka dikirim sendiri | https://www.viberank.app/, https://tokscale.ai/ |
| Factory.ai Analytics | 3 | Pro US$20, Plus US$100, Max US$200 | Organisasi engineering | Biaya per PR, tiket dan project Linear | Atribusi masih Private Preview; privat; hanya Factory | https://docs.factory.ai/api-reference/analytics, https://docs.factory.ai/pricing |
| Amp usage (workspace + thread) | 3 | Hobby gratis; Individual US$20/bulan | Pengguna Amp | Biaya pasti per thread; API JSON | Privat; usage thread hanya 90 hari (P12); tidak ada agregat per project/fase | https://ampcode.com/api/external, https://ampcode.com/pricing |
| SaaS Factory AI Cost Dashboard | 3 | Tidak dicek | Pengguna platform SaaS Factory | Konsep hampir sama dengan sisi biaya 816: spend per project, alert budget | Internal, bukan publik; terikat platformnya | https://saas-factory.ai/docs/ai-cost-dashboard |
| AgentsRoom | 3 | Unduh gratis | Developer multi-agent | Biaya per tiket yang selesai | Aplikasi desktop; privat | https://agentsroom.dev/features/project-statistics |
| Atlassian Statuspage | 4 | Free (100 subscriber, 2 metrik); Hobby US$29/bulan | Operasional SaaS | Format tepercaya | Fokus insiden; metrik sedikit | https://www.atlassian.com/software/statuspage/pricing |
| Badge GitHub Actions / shields.io | 4 | Gratis | README open source | Live, bisa ditanam di mana saja | Badge repo private "not accessible externally"; satu bit info | https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/adding-a-workflow-status-badge |
| Postingan blog biaya AI bulanan | 5 | Gratis | Audiens developer | Angka konkret menarik pembaca | Sekali tulis lalu basi; angka estimasi list price [cuplikan] | https://promptconduit.dev/blog/what-a-month-of-claude-code-costs, https://dev.to/deibygs/i-spent-514-on-claude-code-in-30-days-heres-what-i-learned-1p32 |
| Spreadsheet manual / tidak melakukan apa-apa | 5 | Gratis | Semua | Fleksibel | "gotten way out of hand" (lihat tema A) | https://www.reddit.com/r/claude/comments/1vftn8s/anyone_else_lost_track_of_what_theyre_actually |

**Celah:** tidak ada produk yang secara publik menggabungkan fase project + gate manusia + CI/rilis + biaya agent per project dan per fase. Bagian-bagiannya ada terpisah: halaman transparansi revenue (Baremetrics, TrustMRR, Buffer), roadmap (GitHub Projects, Canny), tampilan biaya agent yang privat (ccusage, Amp, Factory.ai), dan leaderboard spend yang meranking orang (viberank). Analog terdekat, SaaS Factory dan AgentsRoom, keduanya privat.

## Complaint themes

Jumlah = keluhan berbeda yang dibaca lengkap, bukan frekuensi masalah. Kutipan verbatim; X dan G2 tidak ditambang.

| Theme | Count | Example quote | Source |
| --- | --- | --- | --- |
| A. Tidak ada tampilan biaya agent per project/sesi | 5 | "The built-in `/cost` command only shows the current session. There's no way to see: Per-project history" · "I track mine in a spreadsheet that's gotten way out of hand, and the worst part is I still can't tell which project is the money pit." | https://www.reddit.com/r/ClaudeAI/comments/1rwdw3m/i_analyzed_77_claude_code_sessions_233_ghost · https://www.reddit.com/r/claude/comments/1vftn8s/anyone_else_lost_track_of_what_theyre_actually |
| B. Biaya tersebar di banyak dashboard vendor, hanya total, setelah kejadian | 4 | "Copilot's admin view gives me a total but nothing per repo or per person" · "Anthropic dashboard, Reducto dashboard, Voyage AI dashboard, all separate, all after the fact." | https://www.reddit.com/r/ClaudeAI/comments/1vgt027/how_are_you_tracking_ai_coding_spend_per_team/ · https://www.reddit.com/r/ClaudeCode/comments/1rrlj6y/got_hit_with_a_55_bill_on_a_single_run_didnt_see |
| C. Merawat update build in public terasa seperti pekerjaan kedua, lalu basi | 4 | "it feels like a second full-time job." · "the last internal tool I built like this quietly broke about six months after I stopped looking at it." | http://www.indiehackers.com:2082/post/i-m-terrible-at-building-in-public-how-do-you-do-it-without-it-becoming-a-second-job-5adb379349 · https://www.reddit.com/r/EngineeringManagers/comments/1w9onm4/we_built_the_nightly_job_to_track_coding_agent |
| D. Angka publik bisa berbalik merugikan (disalin kompetitor, "perform" untuk audiens) | 6 | "Two competitors started monitoring my updates. Features I posted about building appeared in their products within weeks." · "The transparency didn't keep me honest it made me perform." | https://www.reddit.com/r/SaaS/comments/1s5twuh/build_in_public_almost_killed_my_startup_nobody/ · https://www.producthunt.com/p/general/i-posted-product-metrics-publicly-for-3-months-it-made-me-build-for-the-wrong-people |
| E. Tool observability agent terlalu berat/mahal untuk solo dev | 4 | "they feel like massive overkill for a solo dev" · "Langfuse = nightmare to self host (postgres, click house, redis, minio)." | https://www.reddit.com/r/LangChain/comments/1sslq5g · https://www.reddit.com/r/LangChain/comments/1rjktte/langsmith_vs_langfuse |
| F. Roadmap publik basi | 10 + 2 pengakuan vendor | "It was never up to date and only got semi regular updates once or twice a year" · "The roadmap is outdated, many features that are listed for Juli to September are not shipped" | https://discussions.unity.com/t/interactive-public-roadmap-changes/1693017 · https://github.com/orgs/community/discussions/145255 |

Data pendukung tema C: GTM for One menemukan 76% founder yang memposting update progres tidak pernah memposting update kedua (https://www.gtmforone.com/blog/does-building-in-public-work).

**Implikasi produk:**

- Tema A, B → biaya per project dan per fase dari satu sumber (Amp) adalah nilai inti, bukan total akun.
- Tema C, F → update harus otomatis dari sumber kerja (registry, Amp, GitHub) dan menampilkan kesegaran data; tidak ada input manual.
- Tema D → yang dipublikasikan adalah proses dan biaya, bukan fitur yang sedang dibangun di project private; allowlist melindungi ide private.
- Tema E → tanpa SDK, proxy, trace atau stack observability; satu kolektor kecil.

**Bukti bahwa angka publik menarik audiens:** tweet peluncuran TrustMRR "got 3,000+ likes, 2M+ views" (https://newsletter.marclou.com/p/i-made-an-app-in-24-hours-and-20-378-the-next-day); TrustMRR ± 200.000 pengunjung unik per bulan (https://trustmrr.com/about). Kontra: founder yang memposting update progres di Reddit tidak lebih sering mendapat pelanggan pertama (https://www.gtmforone.com/blog/does-building-in-public-work), dan audiens cenderung sesama founder (r/SaaS di atas). Jadi M7 (jangkauan) adalah hipotesis, bukan janji.

**Konteks Indonesia:** Octoverse 2026 mencatat "Indonesia | 6.34 M" developer per Sept 2026, peringkat 7 dunia, dan "By September 2026, more than 40% of pull requests on GitHub started with an agent's first draft" (https://github.blog/news-insights/octoverse/octoverse-commits-pushes-reach-new-highs-in-2026-as-developers-build-with-agents). Dicoding Indonesia Developer Outlook 2026 (3.163 responden): "Sebanyak 67,3% responden telah mengintegrasikan AI ke dalam alur kerja mereka setiap hari" dan asisten coding utama "GitHub Copilot 50,0% / Claude Code 33,2% / Cursor 14,2% / Codex 12,2%" (https://ido.dicoding.com/).

## Positioning

Untuk developer dan tech lead (Indonesia lalu global) yang ingin tahu apakah factory software berbasis agent AI benar-benar bekerja dan berapa biayanya, **Lantai Pabrik** adalah dashboard publik yang menampilkan fase, keputusan manusia, dan biaya agent per project dan per fase langsung dari sumber kerja factory, tidak seperti leaderboard spend (viberank) atau tampilan biaya privat (ccusage, Amp, Factory.ai) yang hanya menunjukkan total per orang atau per akun.

## Pricing

**Model: gratis, tanpa monetisasi langsung.** Triage memberi skor Money 1 (registry/triage.md), dan semua pembanding publik sejenis gratis dilihat (Buffer Open, TrustMRR listing, viberank, GitHub Projects). Nilai bagi Thomas tidak langsung: kredibilitas untuk klien/jasa, konten build in public, dan data retro factory.

| Opsi di masa depan | Pembanding harga | Catatan |
| --- | --- | --- |
| Template OSS gratis (V3) | ccusage, Langfuse self-host: gratis, MIT | Menambah reputasi; bukan MVP |
| Versi hosted untuk factory lain | Plausible Growth US$14/bulan; Langfuse Core US$29/bulan; Statuspage Hobby US$29/bulan | ASUMSI: US$9–19/bulan per factory masuk akal; butuh auth/multi-tenant, di luar kategori 8 |
| Laporan biaya per project untuk klien | — | ASUMSI; hanya bila Thomas menjual jasa pembuatan app |

Biaya operasional target: US$0/bulan. GitHub Pages gratis untuk repo public di org Free ("If the account that owns the repository uses GitHub Free or GitHub Free for organizations, the repository must be public"), batas situs 1 GB dan "soft bandwidth limit of 100 GB per month" (https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits). Alternatif Cloudflare Pages: "requests to static assets are free and unlimited" (https://developers.cloudflare.com/pages/functions/pricing/).

## Names and domains

RDAP lewat `curl https://rdap.org/domain/<nama>`, 14 Nov 2026. 404 berarti kemungkinan besar tersedia; 200 berarti sudah terdaftar. Hanya dicek, tidak dibeli. Domain sendiri bersifat opsional: MVP bisa memakai subdomain gratis penyedia hosting.

| Name | .com | .id / .co.id | Note |
| --- | --- | --- | --- |
| **Lantai Pabrik** (dipilih sebagai nama kerja) | lantaipabrik.com 404 | lantaipabrik.id 404; lantaipabrik.co.id 404 | "Factory floor": lantai produksi yang terlihat publik; mudah diucapkan ID/EN ("Lantai Pabrik — the factory floor") |
| Papan Pabrik | papanpabrik.com 404 | papanpabrik.id 404 | "Papan" = papan informasi/board |
| Pabrik Terbuka | pabrikterbuka.com 404 | pabrikterbuka.id 404 | Menekankan transparansi |
| Pabrik Aplikasi | pabrikaplikasi.com 200 | pabrikaplikasi.id 200 | Sama dengan nama org GitHub; domain sudah dipakai orang lain |
| pabrik-aplikasi.com | 404 | — | Bertanda hubung; cocok dengan nama org, tetapi kurang enak diucapkan |
| Factory Floor | factoryfloor.dev 200 | — | Sudah terdaftar |
| Open Factory | openfactory.dev 429 | — | RDAP membatasi request; tidak dianggap tersedia |
| Jejak Pabrik | jejakpabrik.com 404 | jejakpabrik.id 404 | "Jejak" = rekam jejak |

## Sources

Semua dibaca 14 Nov 2026.

**Sumber data dan batas teknis**

- Amp External API, thread usage/metadata/messages: https://ampcode.com/api/external
- Amp workspaces, ekspor usage CSV maksimal 30 hari: http://ampcode.com/docs/collaborate/workspaces
- Amp `AMP_API_KEY` untuk CI: https://ampcode.com/docs/cli/execute-mode ; visibilitas thread (private/unlisted/workspace): https://ampcode.com/docs/threads ; harga: https://ampcode.com/docs/pricing ; `amp threads usage`: https://ampcode.com/news/explain-usage
- GitHub REST rate limit ("60 requests per hour" tanpa auth, 5.000 dengan token): https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api
- Izin fine-grained token (Metadata, Contents, Actions read): https://docs.github.com/en/rest/authentication/permissions-required-for-fine-grained-personal-access-tokens
- Repo private → 404 tanpa auth: https://docs.github.com/en/rest/using-the-rest-api/troubleshooting-the-rest-api
- GitHub Pages limits dan larangan SaaS komersial: https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits
- GitHub Actions: runner gratis di repo public; private org Free 2.000 menit/bulan: https://docs.github.com/en/billing/concepts/product-billing/github-actions , https://github.com/pricing
- Cron Actions: minimal 5 menit, bisa tertunda/terbuang saat beban tinggi, nonaktif setelah 60 hari tanpa aktivitas di repo public: https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows
- Linear API: 2.500 request/jam per API key; key bisa dibatasi Read: https://linear.app/developers/rate-limiting , https://linear.app/docs/api-and-webhooks
- Cloudflare Pages limits: https://developers.cloudflare.com/pages/platform/limits/
- UU 27/2022 PDP, Pasal 1 angka 1 dan Pasal 4(3): https://jdih.komdigi.go.id/produk_hukum/view/id/832/t/undangundang+nomor+27+tahun+2022

**Kompetitor, keluhan dan pasar:** lihat kolom Source di tabel di atas. Statistik Indonesia: https://github.blog/news-insights/octoverse/octoverse-commits-pushes-reach-new-highs-in-2026-as-developers-build-with-agents , https://ido.dicoding.com/

**Tidak dipakai sebagai bukti:** X dan G2 (tidak ditambang); harga Baremetrics dan SaaS Factory (tidak dicek); cuplikan hasil pencarian ditandai [cuplikan].
