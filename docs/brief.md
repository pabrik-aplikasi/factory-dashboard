# Brief — Factory dashboard publik

Project `816-factory-dashboard` · kategori 8 (showcase) · Phase 0 Intake, 14 Nov 2026.
Status: Phase 0 dan Phase 1 selesai. Nama kerja: **Lantai Pabrik**.

## Ide dalam 5 baris

1. **Apa:** halaman web publik dwibahasa (ID/EN) yang menampilkan progres dan biaya *software factory* Thomas: project mana di fase mana, gate yang menunggu Thomas, status CI/rilis, dan biaya agent AI per project dan per fase.
2. **Untuk siapa:** Thomas (pemilik factory yang perlu satu layar untuk 135 project) dan publik (developer/calon klien yang ingin melihat factory AI bekerja secara jujur, termasuk angkanya).
3. **Masalah:** progres tersebar di registry YAML privat, 135 repo GitHub, Linear dan thread Amp; biaya hanya terlihat per thread di Amp. Tidak ada gambaran gabungan, dan tidak ada yang bisa dibagikan ke publik tanpa membocorkan data privat.
4. **Kenapa sekarang:** wave 1 (115, 403, 402, 601, 816) baru disetujui 14 Nov 2026; data fase dan biaya mulai mengalir hari ini. Dashboard yang hidup sejak wave 1 merekam sejarah factory dari awal.
5. **Batasan:** hosting < US$20/bulan, ID/EN, Indonesia lalu global, showcase yang juga harus berguna bagi factory; data publik hanya melalui allowlist, tanpa secret atau isi thread privat.

## Fakta yang sudah diperiksa (14 Nov 2026)

| Fakta | Bukti |
| --- | --- |
| Org GitHub `pabrik-aplikasi` plan Free, 66 repo public dan 69 private | `gh api orgs/pabrik-aplikasi` → `plan: free, public_repos: 66, total_private_repos: 69` |
| Repo factory (registry, ide, triage) tidak bisa dibuka publik | `curl https://ampcode.com/projects/@thomasbudiarjo/software-factory` tanpa login → HTTP 404 |
| Biaya per thread tersedia dari Amp CLI | `amp threads usage <thread> --details` mencetak `Cost` (kredit Amp), token, request, dan "estimated token list price" untuk inference lewat subscription milik user |
| Ada dua angka biaya yang berbeda jauh | Thread perencanaan stack: `Cost: $0.34` kredit Amp, tetapi "estimated token list price of $52.86" untuk inference lewat subscription ChatGPT |
| Daftar thread Amp berisi thread di luar factory | `amp threads list` menampilkan thread pribadi lain (misalnya "Portfolio 3D rebrand"), jadi dashboard tidak boleh mengambil semua thread |

## Asumsi

- ASUMSI: sumber kebenaran fase/status adalah `registry/projects.yaml` di repo factory; dashboard membacanya, tidak menulisnya.
- ASUMSI: biaya yang ditampilkan adalah biaya thread yang tercatat di `thread_url` registry (dan thread turunan yang tercatat), bukan seluruh akun Amp Thomas.
- ASUMSI: publik hanya melihat nama project dengan repo public (kategori 1, 3, 7, 8). Project private (kategori 2, 4, 5, 6) tampil sebagai jumlah agregat kecuali Thomas meng-allowlist namanya.
- ASUMSI: Thomas mau angka biaya nyata dipublikasikan, termasuk biaya yang ditanggung subscription, dengan label yang jelas.
- ASUMSI: data tidak perlu realtime; pembaruan tiap jam cukup untuk showcase dan kebutuhan Thomas.
- ASUMSI: target pertama adalah artefak showcase yang bisa dijalankan lokal; hosting publik murah dipilih di Phase 5/12 dengan approval biaya.

## Pertanyaan terbuka (dijawab di Phase 1 atau dibawa ke gate)

1. Biaya mana yang jadi angka utama: kredit Amp yang benar-benar ditagih, estimasi list price, atau keduanya?
2. Bolehkan nama project private, judul thread, atau isi gate (pertanyaan ke Thomas) tampil publik?
3. Apakah Linear (Phase 8 ke atas) dan GitHub CI masuk MVP, atau cukup registry + biaya Amp?
4. Bagaimana kredensial Amp/GitHub/Linear disimpan untuk pengumpul data tanpa masuk repo publik?
5. Apakah dashboard juga menjadi alat kerja Thomas (filter "menunggu aku"), atau murni etalase publik?
6. Domain dan hosting: pakai subdomain gratis (GitHub Pages/Cloudflare) atau domain sendiri (butuh approval biaya)?

## Setelah discovery (Phase 1, 14 Nov 2026)

Varian terpilih: V1, dashboard publik statis dari snapshot ber-allowlist, ditambah ekspor data terbuka dan halaman metodologi biaya ([research.md](research.md#variations)).

1. **Apa:** Lantai Pabrik, situs statis ID/EN yang menampilkan fase, gate menunggu, CI/rilis dan biaya agent per project dan per fase factory Thomas.
2. **Untuk siapa:** Thomas (cockpit pagi hari, mode pemilik lokal) dan Dimas, tech lead yang menilai factory AI ([personas.md](personas.md)).
3. **Masalah:** tidak ada produk publik yang menggabungkan fase + gate + biaya per project/fase; tampilan biaya yang ada privat atau per orang ([research.md](research.md#competitors)).
4. **Pendekatan:** kolektor privat terjadwal membaca registry, Amp API dan GitHub; build menyaring lewat allowlist; pemindai privasi menggagalkan build bila ada data terlarang.
5. **Batasan:** target US$0, maksimal < US$20/bulan; tanpa login; tanpa isi/judul thread.

**Asumsi yang terjawab:**

- Biaya bisa dibaca sebagai JSON dari Amp External API, bukan hanya teks CLI (P11).
- Biaya per fase bisa dihitung lewat `--start/--end` CLI (P4, P11b).
- Repo private tidak terlihat tanpa autentikasi; API GitHub tanpa token hanya mengembalikan 66 repo public (P16).
- Hosting statis gratis tersedia untuk repo public di org Free (GitHub Pages, Cloudflare Pages).

**Asumsi baru / tersisa:**

- Usage Amp hanya tersedia 90 hari (P12): kolektor harus berjalan sejak awal build; kalau tidak, biaya awal wave 1 hilang.
- Field gate tertunda belum ada di registry (P8); butuh persetujuan Thomas untuk menambah field opsional.
- Thomas setuju biaya nyata dan nama project public tampil (ditanyakan di gate Phase 4).

**Pertanyaan terbuka 1, 2, 3 dan 5** dijawab sebagai keputusan default di PRD; Thomas mengonfirmasi atau mengubahnya di gate Phase 4.
