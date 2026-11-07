# Pembahasan dan keputusan — Platform streaming video

## Status keputusan

Masuk ranking triage; belum dipilih untuk wave implementasi. Skor dan klaim pasar pada triage adalah **ASUMSI**, bukan riset pasar. Repo dibuat atas permintaan dokumentasi semua project pada 7 November 2026; bukan instruksi menjalankan wave atau membatalkan usulan merge/kill.

Prioritas sumber: keputusan Thomas terbaru → tabel stack revisi dan pendukung → catatan review historis → brainstorm. Catatan historis di bawah tidak boleh membatalkan koreksi terbaru.

## Keputusan dan batas draf

- **Arahan Thomas:** terapkan rekomendasi review pada dokumen, pertahankan diversitas, jangan coding. Kecocokan stack lebih penting daripada kuota; sedikit variasi yang bermakna dapat diterima. Batas lama dua penggunaan dan beda 4/8 bukan alasan otomatis mengganti stack. Bahasa/framework utama tetap bervariasi; tidak menambah komponen tanpa kebutuhan. Perubahan ini terbatas pada artifact, bukan sinkronisasi aturan repository atau registry stack final.
- Thomas membolehkan **Go core + TS interface** maupun **Rust core + TS interface**, dengan minat lebih besar pada Rust. Interface TS tidak harus berarti backend TS tambahan.
- Semua **135 ide tetap terlihat**, termasuk 7 usulan merge dan 11 usulan kill. Status itu belum keputusan final; kandidat standalone untuk mereka bersifat kondisional.
- Kategori 2/3/6 tetap memakai aplikasi utama TS/Go; Rust boleh menjadi core komputasi terbatas yang disebutkan eksplisit. Kategori 7/8 selalu mempunyai bahasa eksplorasi dengan tugas nyata. Ini interpretasi desain, bukan izin mengganti seluruh backend bisnis menjadi Rust.
- Pilihan utama = rekomendasi awal berdasarkan brief enam baris, belum berdasarkan PRD lengkap. Versi, lisensi, maintenance, biaya, kompatibilitas semua kombinasi, dan kelulusan e2e belum dibuktikan.
- Tidak ada keputusan wave, merge/kill, produksi, atau pengeluaran uang yang dibuat melalui dokumen ini. Registry final tidak diubah.

## Keputusan Thomas setelah pembahasan stack pendukung

Thomas menyetujui rekomendasi pembahasan pada 7 November 2026 dan meminta seluruh stack pendukung dilengkapi, tetap tanpa coding. Persetujuan mencakup:

- Palette merupakan daftar pilihan, bukan kewajiban memakai semua teknologi. Diversitas utama dipertahankan; auth, query tooling, monorepo, dan observability boleh berulang ketika cocok.
- Phoenix, Reitit, Giraffe, Wisp, Kemal dan SQLite standalone diizinkan sebagai kandidat. Ini bukan approval otomatis semua versi, dependency turunan, instalasi, atau biaya.
- 815: web memakai template tepercaya dan data tervalidasi; CLI boleh menjalankan kode milik operator. Hosted arbitrary user code tidak masuk baseline.
- 713: draft boleh offline; penerimaan final diputuskan server melalui finalisasi idempotent dan receipt. Submit setelah deadline menunggu keputusan pengawas, bukan otomatis diterima; jam perangkat bukan bukti waktu penyerahan. Keputusan pengawas harus tercatat, tanpa mengganti waktu penerimaan asli.
- 716/813/809: offline mencakup tutup dan buka ulang setelah data/aset siap tersimpan; storage browser tetap bisa hilang dan perlu pesan pemulihan.
- 212: PostgreSQL menjadi baseline dengan asumsi belum ada integrasi MySQL wajib; MySQL tetap alternatif dengan inventaris sync lengkap.
- Lengkapi pilihan auth/otorisasi, data/migrasi, runtime/kontrak, komponen khusus dan operasional seluruh project sebagai rencana, bukan kelulusan pengujian.

Wave, merge/kill, pembelian, aktivasi layanan dan deployment tetap tidak disetujui melalui keputusan ini. Aturan lama di repository/skills belum disinkronkan; registry final tetap tidak diubah.

## Kutipan triage khusus project

## Ranking

| Rank | Idea | Cat | Pain | User | Diff | Feas | Money | Learn | Total | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 115 | `708-streaming-video` Platform streaming video | 7 | 2 | 2 | 1 | 2 | 1 | 5 | 13 | Biaya storage dan bandwidth |

## Pertimbangan percakapan awal

Tidak ada PRD final atau keputusan desain per project yang ditemukan.

## Ringkasan brainstorm awal

Upload/transcode/HLS/CDN/rekomendasi; media pipeline; baseline terbaru HLS VOD.

Sumber: artifact brainstorm revisi 1 (6 November 2026), ditautkan di sources.md. Ini bahan ide, bukan scope MVP atau riset terverifikasi; klaim pasar, kompetitor, tarif dan regulasi adalah **ASUMSI**. Koreksi keputusan terbaru tetap berlaku.

## Hasil review yang menyebut project ini

| Temuan | Keputusan yang diterapkan dalam dokumen | Yang masih harus diputuskan atau dibuktikan |
| --- | --- | --- |
| R06 · 708 | Kontrak HLS VOD dengan ≥2 rendition, master playlist/segments dan player sesuai browser | Package codec, packager dan player beserta lisensinya belum dikunci |

## Riwayat review (bukan instruksi mengganti keputusan terbaru)

**R01 — NATS: nyatakan durability dan hubungan dengan transaksi DB.**

Cakupan temuan: utama 402, 703, 707, 708, 717, 810, 815; alternatif 101, 401, 414, 502, 505. Pada alternatif 705/709, bedakan juga fanout/presence dari penyimpanan order/pesan.

- Masalah: label NATS saja belum membuktikan pekerjaan yang diterima akan bertahan ketika worker mati. Mengganti River dengan broker juga tidak otomatis mempertahankan atomic enqueue bersama transaksi PostgreSQL.
- Rekomendasi: untuk pekerjaan yang tidak boleh hilang, tentukan JetStream dengan storage persisten, durable consumer, acknowledgement setelah hasil tersimpan, dan consumer idempotent; gunakan transactional outbox atau rekonsiliasi eksplisit untuk celah commit DB–publish. Untuk pekerjaan kecil, tabel job/outbox dengan worker Rust cukup. Core NATS tetap sah untuk update sementara yang boleh hilang.
- Bukti: [NATS JetStream](https://docs.nats.io/nats-concepts/jetstream) menjelaskan Core NATS sebagai “at most once, never replayed”; [River transactional enqueueing](https://riverqueue.com/docs/transactional-enqueueing) memungkinkan enqueue dalam transaksi perubahan bisnis. JetStream bukan transaksi atomik dengan PostgreSQL dan bukan jaminan pembayaran exactly-once.
- Batas kesimpulan: belum ada implementasi yang diketahui memakai Core NATS untuk pekerjaan durable; ini kekurangan spesifikasi, bukan alasan membuang NATS atau Rust.
**R06 — 708 Streaming video: adaptive streaming membutuhkan packaging dan player.**

Satu MP4 hasil transcode belum memenuhi brief adaptive streaming. Rekomendasi kontrak pertama: **HLS VOD**, sedikitnya dua rendition, master playlist dan segments di object storage, serta player HLS sesuai browser target. [FFmpeg HLS muxer](https://ffmpeg.org/ffmpeg-formats.html#hls-2) mendokumentasikan playlist/segment, variant streams, dan master playlist. Pertahankan Rust/Rocket atau alternatif Go; codec/packager/player masih kandidat yang perlu persetujuan dan verifikasi versi, bukan dependency yang sudah dipilih. Provider transcoder bisa saja sudah menyediakan semuanya—jika demikian, cukup tegaskan output tersebut.
