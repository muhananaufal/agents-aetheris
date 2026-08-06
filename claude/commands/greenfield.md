# /greenfield — Inisiasi Proyek Day-0

WAJIB ikuti urutan ini. DILARANG melompati langkah.

0. **Cek dulu apakah ini proyek kecil** (definisi §2 `AGENTS.md`: tanpa DB ter-provision, tanpa proses hidup terus, tanpa integrasi pihak ketiga, <10 berkas sumber). **Kecil → macro ini GUGUR.** Langsung eksekusi tanpa Day-0 Quintet, tanpa swarm, tanpa `implementation_plan.md`. Memaksakan `docker-compose` 3-tier untuk skrip 100 baris = pelanggaran KISS.
1. **Tentukan stack.** Belum jelas → tanya user. DILARANG menebak.
2. **Baca protokol stack-nya sampai tuntas.** Versi ini lebih spesifik daripada ringkasan mana pun — DILARANG memakai 5 Pilar dari ingatan.

   | Stack | Protokol WAJIB dibaca |
   | :--- | :--- |
   | Go | `~/.gemini/config/skills/golang-mastery/references/_protocol/greenfield.md` |
   | Rust | `~/.gemini/config/skills/rust-mastery/references/_protocol/greenfield.md` |
   | Laravel | `~/.gemini/config/skills/laravel-mastery/references/_protocol/greenfield.md` |
   | NestJS | `~/.gemini/config/skills/nestjs-mastery/references/_protocol/greenfield.md` |

3. **Riset referensi domain** sesuai §7 `AGENTS.md` (Claude Code: CC-1).
4. **Tulis `implementation_plan.md`** dengan template §5 `AGENTS.md`.
5. **BERHENTI.** Tunggu user mengetik "Gasskan". DILARANG menulis kode sebelum itu.

**Stack di luar keempat di atas** — tidak ada protokol khusus, WAJIB tetap hasilkan 5 Pilar Day-0 ini:
1. Modular 3-Tier Container Topology — Tier 1 core DB/cache, profile `telemetry`, profile `stress` (k6). DILARANG auto-run `docker build`/`up`.
2. Fail-Fast Type-Safe Configuration — `.env.example` + validator yang crash saat boot bila env kurang/salah tipe.
3. Graceful Shutdown — tangkap `SIGINT`/`SIGTERM`, drain in-flight 5–15 detik, flush log & trace sebelum exit.
4. Migration & Idempotent Seeder — folder `migrations/` + seeder yang aman dijalankan berulang.
5. Unified Task Runner — target baku `dev`, `test`, `infra-up`, `telemetry-up`, `migrate`, `seed`, `stress`.
