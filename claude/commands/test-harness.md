# /test-harness — Suite Pengujian & Reliability

Buat atau perbaiki suite pengujian otomatis untuk modul/fitur yang sedang dikerjakan.

## Cakupan WAJIB

| Lapis | Isi | Perintah rujukan |
| :--- | :--- | :--- |
| **Unit** | happy-path, boundary value, error condition, edge-case (null, empty, timeout, payload korup) | sesuai stack |
| **Concurrency** | race detector aktif | `go test -race`, Tokio multi-thread test, worker pool mock |
| **DB Integration** | transactional rollback test atau Testcontainers | sesuai stack |
| **Benchmark** | throughput + alokasi memori | `go test -bench=. -benchmem`, `cargo bench`, skrip k6 |

## Aturan

- DILARANG menulis test yang selalu lulus tanpa menguji apa pun (assertion kosong, mock yang mem-bypass logic yang diuji).
- DILARANG meninggalkan flaky test. Test bergantung waktu/urutan WAJIB dibuat deterministik.
- WAJIB jalankan seluruh suite setelah dibuat dan laporkan outputnya APA ADANYA. Ada yang gagal → katakan gagal, jangan disembunyikan.
- Harness sudah ada → perluas, DILARANG menimpa test yang sudah lulus.

## Laporan WAJIB

Jumlah test ditambahkan, coverage sebelum → sesudah (bila tool coverage tersedia), dan output eksekusi penuh.
