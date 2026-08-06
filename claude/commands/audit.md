# /audit — Forensic Quality Gate Audit

Audit repo / direktori kerja saat ini dengan standar §6 `AGENTS.md` plus pemindaian forensik di bawah.

## Yang WAJIB dipindai

| Kategori | Cari |
| :--- | :--- |
| **Placeholder** | `// TODO`, `FIXME`, `TBD`, mock, stub, kode kosong, `/* omitted */` |
| **Type-safety** | `any` (TS), unchecked `unwrap()`/`expect()` (Rust), raw `interface{}` (Go), `mixed` tanpa guard (PHP) |
| **Secret** | credential & API key hardcode, `.env` ikut ter-commit, token di log |
| **OWASP Top 10** | SQL Injection, IDOR, Mass Assignment, broken auth, input validation hilang, hashing lemah (WAJIB Argon2id/bcrypt) |
| **Performa** | N+1 query, foreign key tanpa index, goroutine/promise leak, listener & stream tidak ditutup |
| **Konkurensi** | race condition, deadlock, shared state tanpa lock, missing timeout |

## Prosedur

1. Ada `system_map.md` → WAJIB dibaca lebih dulu.
2. Jalankan test + linter stack terkait. Laporkan output APA ADANYA. DILARANG mengklaim lulus tanpa menampilkan bukti.
3. Susun laporan dengan format di bawah.
4. **DILARANG memperbaiki temuan tanpa persetujuan user**, KECUALI user menulis "sekalian perbaiki". Perbaikan >3 berkas tetap tunduk pada RFC-Path §2 `AGENTS.md`.

## Format laporan WAJIB

```markdown
## Temuan Kritis
| # | Berkas:baris | Masalah | Dampak | Tingkat |

## Temuan Non-Kritis

## Hasil Test & Linter
<!-- perintah yang dijalankan + output apa adanya -->

## Rekomendasi Perbaikan
<!-- menyangkut arsitektur? WAJIB ≥3 opsi netral + trade-off, sesuai §4 AGENTS.md -->
```
