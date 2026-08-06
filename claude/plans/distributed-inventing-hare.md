# SSOT Aturan & Mastery Skills: Antigravity ↔ Claude Code

## Context

Anda menjalankan dua agent coding dengan **dua salinan manual** dari konstitusi engineering yang sama:

| Berkas | Ukuran | Dibaca oleh |
| :--- | :--- | :--- |
| `~/.gemini/config/AGENTS.md` | 3.652 B / 38 baris | Antigravity (workspace utama) |
| `~/.claude/CLAUDE.md` | 3.427 B / 37 baris | Claude Code (hilir) |

Keduanya ~85% identik tapi **sudah drift**. Bukti konkret:

| Aturan | AGENTS.md | CLAUDE.md |
| :--- | :--- | :--- |
| Perintah tes konkret (`go test -race`, `cargo test`, …) | ❌ tidak ada | ✅ ada |
| `staleness check` di Protokol Lokal | ✅ ada | ❌ hilang |
| Contoh untyped bypass (`any`, `unwrap()`, `interface{}`) | ❌ tidak ada | ✅ ada |
| Fallback Search (`search_web` hanya bila referensi habis) | ✅ ada | ❌ hilang |

Tiap perbaikan harus diketik dua kali → drift makin lebar tiap minggu.

Sementara **mastery skills** (5 bundle, 668 berkas referensi, 7,4 MB) hanya hidup di `~/.gemini/config/skills/`. Claude Code **tidak melihatnya sama sekali** (`~/.claude/skills/` belum ada). CLAUDE.md cuma menyebut path-nya sebagai teks biasa, jadi pemanfaatannya bergantung kepatuhan model — bukan mekanisme harness.

**Tujuan:** satu tempat edit, dua tool langsung dapat manfaatnya. **Batasan mutlak: kualitas Antigravity tidak boleh turun.**

---

## Rekomendasi: "Satu Inti + Dua Adaptor Tipis"

Aturan dipecah tiga berkas di dalam folder baru `~/.gemini/config/shared/`:

| Berkas | Isi | Dibaca |
| :--- | :--- | :--- |
| `core-rules.md` | ~85% aturan yang berlaku universal | **kedua** tool |
| `tool-antigravity.md` | khusus Antigravity: `invoke_subagent`, tier `pro`/`flash`, `replace_file_content` | Antigravity saja |
| `tool-claude-code.md` | khusus Claude Code: `/compact`, `/greenfield`, `/audit`, penerjemah protokol swarm | Claude Code saja |

`AGENTS.md` dan `CLAUDE.md` menyusut jadi **berkas penunjuk 2 baris**:

```text
~/.gemini/config/AGENTS.md          ~/.claude/CLAUDE.md
  @shared/core-rules.md               @~/.gemini/config/shared/core-rules.md
  @shared/tool-antigravity.md         @~/.gemini/config/shared/tool-claude-code.md
```

Mulai sekarang **`core-rules.md` adalah satu-satunya tempat mengedit aturan bersama.**

### Kenapa `@import`, bukan skrip sinkronisasi
Keduanya resmi didukung — bukan tebakan:
- **Antigravity** — `antigravity.google/docs/rules-workflows`: *"Rules support referencing other files using `@filename` syntax."* Path relatif diselesaikan relatif terhadap lokasi berkas rules.
- **Claude Code** — `code.claude.com/docs/en/memory`: `@path/to/import`, maks 4 hop. Dokumentasinya bahkan **menganjurkan pola ini persis** untuk interop `AGENTS.md`, dan import di lingkup user dimuat **tanpa dialog approval**.

Konsekuensinya: **nol build step, nol perintah sync, drift mustahil secara struktural.** Skrip generator akan menambah satu langkah manual yang bisa lupa dijalankan — justru sumber drift baru.

### Kenapa `shared/`, bukan `rules/`
Forensik `language_server.exe` menunjukkan Antigravity **otomatis memuat** apa pun di `rules/` relatif terhadap customization root. Kalau ketiga berkas ditaruh di situ, `tool-claude-code.md` ikut terbaca Antigravity sebagai sampah context. Folder netral `shared/` membuat pemuatan sepenuhnya eksplisit lewat `@import`.

---

## Jaminan Anti-Regresi Antigravity

Ini kekhawatiran utama Anda, jadi ditangani berlapis:

1. **Baseline commit.** Repo `~/.gemini/config` sudah ber-git (tag `v1.0`→`v1.5`) tapi ada **8 berkas modified + 4 untracked (termasuk seluruh `nestjs-mastery/` yang belum pernah masuk git)**. Commit dulu sebagai `v1.6-baseline` → rollback instan kapan pun.
2. **Aturan hasil gabungan wajib superset.** `core-rules.md` + `tool-antigravity.md` harus memuat **seluruh** isi AGENTS.md sekarang, ditambah 4 butir yang selama ini hanya ada di sisi Claude Code. Tidak ada satu baris pun yang hilang.
3. **Uji sebelum dianggap beres.** Buka chat baru Antigravity → *"Sebutkan verbatim §4 PM Gate dan §5 Subagent Protocol dari aturan aktifmu."* Kalau tidak bisa menyebutkan → `git checkout AGENTS.md`, pindah ke rencana cadangan.
4. **Rencana cadangan siap.** Bila `@import` ternyata tidak diekspansi di `AGENTS.md`, tempel isi `core-rules.md` + `tool-antigravity.md` secara literal ke `AGENTS.md`. SSOT sisi Claude Code tetap jalan lewat `@import`.
5. **Mastery skills tidak disentuh sama sekali** (lihat bagian berikut).

---

## Mastery Skills: Symlink, Tanpa Mengubah Isi

Antigravity **tidak perlu diubah apa pun** — `~/.gemini/config/skills/` sudah jadi lokasi global skills resminya (dikonfirmasi `antigravity.google/docs/skills`).

Sisi Claude Code cukup 5 symlink direktori:

```text
~/.claude/skills/golang-mastery            → ~/.gemini/config/skills/golang-mastery
~/.claude/skills/rust-mastery              → ~/.gemini/config/skills/rust-mastery
~/.claude/skills/laravel-mastery           → ~/.gemini/config/skills/laravel-mastery
~/.claude/skills/nestjs-mastery            → ~/.gemini/config/skills/nestjs-mastery
~/.claude/skills/skill-engineering-mastery → ~/.gemini/config/skills/skill-engineering-mastery
```

Dokumentasi Claude Code menyatakan entri skill **boleh berupa symlink** ke direktori mana pun, dan `~/.claude/skills/` **diawasi live** sehingga edit langsung terbaca tanpa restart. Frontmatter kedua tool kompatibel 1:1 (`name` + `description`). Mesin ini sudah **Developer Mode ON + Administrator + NTFS**, jadi symlink tersedia.

Symlink **per-skill**, bukan seluruh folder `skills/`, karena: berkas lepas `quality_tracker.md` dan `rust.txt` di sana bukan skill dan akan otomatis terlewat, dan Anda tetap bisa menaruh skill khusus Claude Code berdampingan.

### Masalah tersembunyi & solusinya
Kelima `SKILL.md` memerintahkan *"Dynamic Swarm Subagent"* dengan tier model `pro`/`flash` — semantik Antigravity yang **tidak eksis** di Claude Code. Frontmatter `skill-engineering-mastery` bahkan menyebut *"di ekosistem Antigravity"*.

**Solusinya: jangan sentuh kelima `SKILL.md` itu.** Cukup pasang tabel penerjemah di `tool-claude-code.md`:

| Ditulis di SKILL.md | Arti di Claude Code |
| :--- | :--- |
| `invoke_subagent` tier `pro` | baca langsung via Read/Grep; naik ke subagent `Explore` paralel hanya bila diminta |
| `invoke_subagent` tier `flash` | Grep/Glob langsung |
| `view_file` / `replace_file_content` | Read / Edit |
| "ekosistem Antigravity" | berlaku untuk agent coding mana pun |

Nol edit pada konten mastery yang sudah matang ⇒ **nol risiko regresi Antigravity**, dan penerjemahnya hanya terbaca Claude Code.

---

## Kenaikan Kualitas untuk Antigravity

Sesuai permintaan "kalau bisa naik". Semuanya aditif:

1. **4 butir yang selama ini hanya di CLAUDE.md masuk ke `core-rules.md`** ⇒ otomatis dinikmati Antigravity: perintah tes konkret per stack, contoh untyped bypass, dan dua butir lain di tabel drift di atas.
2. **668 berkas referensi mastery jadi bisa dipakai Claude Code** ⇒ hilir tidak lagi bekerja dengan standar lebih rendah dari hulu.
3. **`~/.gemini/config/shared/` ikut ter-versioning git** ⇒ sejarah perubahan aturan bisa ditelusuri dan di-rollback, sesuatu yang sekarang belum ada.

---

## Berkas yang Disentuh

**Dibuat**
- `~/.gemini/config/shared/core-rules.md`
- `~/.gemini/config/shared/tool-antigravity.md`
- `~/.gemini/config/shared/tool-claude-code.md`
- 5 symlink di `~/.claude/skills/`

**Diubah**
- `~/.gemini/config/AGENTS.md` → jadi 2 baris `@import`
- `~/.claude/CLAUDE.md` → jadi 2 baris `@import`

**Tidak disentuh:** seluruh `~/.gemini/config/skills/`, `~/.claude/commands/`, `~/.claude/settings.json`.

⚠️ **Jebakan teknis:** semua berkas ini **UTF-8 tanpa BOM** dan penuh emoji serta em-dash. `Set-Content`/`Out-File` bawaan PowerShell 5.1 akan merusaknya. Penulisan wajib lewat `UTF8Encoding($false)`. *(Mojibake yang sempat muncul saat `Get-Content` hanya artefak konsol, bukan korupsi berkas.)*

---

## Verifikasi

1. **Baseline aman** — `git -C ~/.gemini/config log --oneline -1` menampilkan commit `v1.6-baseline`, `git status` bersih.
2. **Antigravity tidak turun** — chat baru: *"Sebutkan verbatim §4 PM Gate dan §5 Subagent Protocol dari aturan aktifmu."* Harus lengkap. Lalu *"Sebutkan perintah tes wajib untuk Go."* Harus menjawab `go test -race` (butir baru hasil merge).
3. **Batas 12.000 karakter** — hitung total `core-rules.md` + `tool-antigravity.md`; harus di bawah 12.000 (estimasi ±4.500, aman). Ini batas resmi berkas rules Antigravity.
4. **Claude Code memuat aturan** — `/context`, cek bagian **Memory files** memuat CLAUDE.md beserta kedua berkas hasil import.
5. **Skills terbaca dua sisi** — `/skills` di Claude Code menampilkan kelima mastery; di Antigravity kelimanya masih muncul seperti semula.
6. **Uji SSOT sungguhan** — tambah baris uji di `core-rules.md`, tanyakan ke **kedua** tool. Dua-duanya harus menyebutkannya. Lalu hapus lagi.

---

## Belum Termasuk (tahap berikutnya)

`/greenfield`, `/audit`, `/test-harness` sekarang eksklusif Claude Code — Antigravity punya 0 padanan. Antigravity mendukung **Workflows** (`.agents/workflows/`, dipanggil `/nama`, batas 12.000 karakter), tapi **lokasi workflow global belum terdokumentasi resmi** sehingga perlu diverifikasi dulu di mesin Anda. Dikeluarkan dari batch ini supaya fondasi SSOT bisa dibuktikan jalan dulu; ditawarkan sebagai tahap lanjutan.
