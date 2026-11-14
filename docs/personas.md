# Personas — Lantai Pabrik (Factory dashboard publik)

Phase 1 · 14 Nov 2026. Kutipan verbatim dari halaman yang dibaca lengkap; sumber lain di [research.md](research.md).

## Persona 1: Thomas, pemilik factory

- **Context:** pemilik dan satu-satunya pengambil keputusan software factory berbasis agent Amp; 135 ide, 5 project aktif di wave 1, repo di org GitHub `pabrik-aplikasi`. Bekerja di laptop dan HP; nyaman dengan CLI, YAML dan GitHub. Memakai subscription model (ChatGPT) yang di-route lewat Amp (bukti primer P5).
- **Main job:** memutuskan ke mana perhatian dan uang pergi: project mana yang menunggu keputusannya di gate, mana yang macet, mana yang mahal.
- **Today they use:** membaca `registry/projects.yaml`, membuka thread Amp satu per satu, menunggu notifikasi gate, menjalankan `amp threads usage` per thread. Biayanya: tidak ada agregat, waktu terbuang, dan biaya per fase tidak pernah terlihat.
- **Pains:**
  1. Posisi lima project paralel hanya bisa dilihat dengan membuka banyak file dan thread (bukti primer P8; skill factory-orchestrator: tiap project melapor di thread sendiri).
  2. Gate yang menunggu bercampur dengan thread non-factory di daftar thread Amp (P7).
  3. Biaya per project dan per fase tidak ada; per thread pun ada dua angka yang mudah tertukar (P3, P5). Keluhan serupa di pasar: "I still can't tell which project is the money pit." — https://www.reddit.com/r/claude/comments/1vftn8s/anyone_else_lost_track_of_what_theyre_actually
  4. Data usage thread hilang setelah 90 hari (P12), sehingga retro Phase 13 bisa kehilangan angka bila tidak dicatat sejak awal.
  5. Showcase manual cepat basi: "it feels like a second full-time job." — http://www.indiehackers.com:2082/post/i-m-terrible-at-building-in-public-how-do-you-do-it-without-it-becoming-a-second-job-5adb379349
- **Gains:** satu layar pagi hari: "2 gate menunggu aku, project X paling mahal di Phase 5, data per 1 jam lalu"; satu link yang bisa dibagikan tanpa khawatir bocor; angka per fase untuk memperbaiki skill factory.
- **Quote:** "Masalah: Progres dan biaya factory tidak terlihat." — file ide `ideas/816-factory-dashboard.md` di repo factory (ditulis Thomas)
- **Will pay?** Tidak membayar langganan; budget hosting < US$20/bulan (file ide, baris Batasan). Membayar dengan waktu agent untuk membangunnya.

## Persona 2: Dimas, tech lead yang mengevaluasi agent AI

- **Context:** 28–38 tahun, tech lead atau engineering manager di startup/software house di Jakarta, Bandung atau Yogyakarta; membuka link dari X/LinkedIn/grup Telegram di HP Android lewat 4G, lalu membaca detail di laptop. Membaca Inggris teknis, lebih nyaman Indonesia untuk penjelasan. ASUMSI: usia dan kota; bukti adopsi: 67,3% developer Indonesia memakai AI setiap hari (https://ido.dicoding.com/).
- **Main job:** menilai apakah factory software berbasis agent AI nyata dan berapa biayanya, sebelum meyakinkan atasan/klien atau mencoba pola serupa di timnya.
- **Today they use:** postingan blog satu kali, leaderboard spend per orang (viberank), demo vendor, dan pengalaman sendiri; tidak ada contoh factory nyata dengan angka per project.
- **Pains:**
  1. Klaim AI sulit diverifikasi; atasan meminta bukti: "Fine - but show me the ROI on what we're spending on AI. Don't tell me it's helping, show me." — https://www.reddit.com/r/ClaudeAI/comments/1tslzaq/followup_i_talked_my_manager_out_of_ranking/
  2. Tidak tahu biaya nyata per pekerjaan: "Finance is starting to ask the obvious question and I dont have a good answer." — https://www.reddit.com/r/EngineeringManagers/comments/1w73h5x/our_ai_coding_bill_keeps_climbing_and_i_honestly
  3. Biaya dalam dolar terasa berat di Indonesia: "gw ga pengen subscribe, lumayan mahal since bayarnya harus make dollar" — https://www.reddit.com/r/indotech/comments/1tvufbq/beli_claude_token_lewat_taobao_ada_yang_pernah
  4. Tool observability terlalu berat untuk sekadar tahu biaya: "they feel like massive overkill for a solo dev" — https://www.reddit.com/r/LangChain/comments/1sslq5g
- **Gains:** melihat dalam 1 menit berapa project, di fase apa, berapa biaya per fase, dan bisa membuka repo public untuk memeriksa hasilnya; bisa mengutip angka dengan definisi yang benar.
- **Quote:** "How are you all proving whether AI coding spend is actually paying off?" — https://www.reddit.com/r/EngineeringManagers/comments/1w73h5x/our_ai_coding_bill_keeps_climbing_and_i_honestly
- **Will pay?** Tidak untuk melihat dashboard (semua pembanding publik gratis). Mungkin membayar jasa/template di masa depan (ASUMSI, tidak divalidasi).

## Anti-persona: Rani, platform engineer yang butuh observability LLM

- **Who:** engineer di tim yang membangun aplikasi LLM sendiri dan butuh trace per request, prompt, latency, eval dan alert real-time (pengguna LangSmith, Langfuse, Helicone).
- **Why we do not serve them:** kebutuhan mereka menuntut SDK/proxy di aplikasi, penyimpanan prompt dan isi percakapan, serta realtime, yang bertentangan dengan batas privasi (tanpa isi thread) dan budget < US$20/bulan. Pasar itu sudah dilayani dengan baik oleh produk khusus (lihat tabel kompetitor kelompok 3).
- **What that means for the product:** tidak ada trace, prompt, isi pesan atau judul thread; tidak ada alert real-time; tidak ada SDK atau proxy; tidak ada eval. Biaya hanya agregat per thread, project dan fase.
