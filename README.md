# Balapan Hybrid Cloud: ACO-SA vs DMFO-DE

Presentasi interaktif bergaya game balapan (ala Hill Climb) untuk membandingkan dua algoritma hybrid pada **penjadwalan tugas komputasi awan (cloud task scheduling)**:

1. **ACO-SA** (*Ant Colony Optimization – Simulated Annealing*), rujukan: Mensah et al. (2021); Kumar (2024).
2. **DMFO-DE** (*Discrete Moth-Flame Optimization – Differential Evolution*), rujukan: Ahmed et al. (2021).

Semua berjalan di peramban (client-side) dalam **satu berkas HTML** tanpa library dan tanpa backend. Materi, dataset, rumus, dan parameter algoritma mengikuti proyek *Hybrid-Algorithm (kelas B, kelompok 8)*.

---

## 1. Cara menjalankan

- Buka `balapan-hybrid-acosa-dmfode.html` di peramban modern (Chrome, Edge, Firefox, Safari).
- Klik di mana saja sekali supaya suara aktif. Tombol 🔊 di pojok kanan atas untuk mematikan suara.
- Berjalan offline. Satu-satunya sumber luar adalah font Google "Patrick Hand"; kalau tidak termuat, tampilan memakai font cadangan.

## 2. Alur 8 stage

Mobil berjalan dari satu bendera stage ke bendera berikutnya. Tombol **Lanjut** baru aktif setelah tugas di stage itu selesai.

| Stage | Isi | Syarat lanjut |
|---|---|---|
| 0 | Formulasi masalah, rumus, dataset Minggu 3, perkenalan dua algoritma (4 bagian yang bisa dibuka) | Langsung |
| 1 | Walkthrough ACO-SA, 5 langkah dengan penjelasan dan contoh angka | Lihat kelima langkah |
| 2 | Knowledge Check 1 (ACO-SA) | Jawab soal |
| 3 | **Simulator balapan** ACO-SA vs DMFO-DE | Simulasi selesai |
| 4 | Walkthrough DMFO-DE, 5 langkah | Lihat kelima langkah |
| 5 | Knowledge Check 2 (DMFO-DE) | Jawab soal |
| 6 | Benchmark 10 trial + CSV | Jalankan benchmark |
| 7 | Skor, tabel perbandingan, kesimpulan | Tombol "Main Lagi" mengulang dari Stage 0 |

## 3. Masalah dan dataset

Membagi **16 task** ke **4 VM** sehingga **makespan** sekecil mungkin. Ada 4^16 = 4.294.967.296 kemungkinan jadwal, sehingga dipakai metaheuristik.

| VM | MIPS | Biaya/detik |
|---|---|---|
| VM-1 | 1500 | $0,04 |
| VM-2 | 2000 | $0,06 |
| VM-3 | 2500 | $0,08 |
| VM-4 | 3000 | $0,10 |

Panjang task (MI): T01 4500, T02 7200, T03 3100, T04 8900, T05 5400, T06 6200, T07 2800, T08 9100, T09 4900, T10 6700, T11 3800, T12 7500, T13 5100, T14 8200, T15 4200, T16 6900. Total 94.500 MI, total kapasitas 9.000 MIPS.

### Metrik

- **Waktu eksekusi**: `ET = lengthMI / MIPS`. Task di satu VM dijalankan berurutan.
- **Makespan**: `Cmax = max_j (beban VM-j)`.
- **Degree of Imbalance**: `DI = (Lmax − Lmin) / Lavg`. Nilai 0 berarti beban merata.
- **Total cost**: `Σ beban VM × biaya per detik`.
- **Batas ideal makespan**: `ΣMI / ΣMIPS = 10,5 detik`. Task tidak bisa dipecah, sehingga jadwal nyata hanya bisa mendekatinya.

## 4. Algoritma

Posisi/jadwal berupa vektor 16 angka bulat 0–3 (nomor VM tiap task). Fitness = makespan.

### ACO-SA

| Parameter | Nilai |
|---|---|
| α (feromon), β (heuristik) | 1,0 dan 1,5 |
| ρ (evaporasi) | 0,25 |
| Suhu SA | `T = 25 × 0,05^(iter / (maxIter−1))` |
| Langkah SA per semut | 5 |

Siklus tiap iterasi:

1. Tiap semut memilih VM untuk setiap task dengan peluang ∝ `τ^α × η^β`, di mana `η = MIPS VM / panjang task`.
2. Jadwal semut diperhalus 5 langkah SA: ubah satu task ke VM acak; terima jika membaik, atau dengan peluang `exp(−Δ/T)` jika memburuk.
3. Feromon menguap `τ ← (1−ρ)τ`, lalu jadwal terbaik iterasi menyetor `1/makespan`.
4. Jadwal terbaik global diperbarui bila lebih baik.

### DMFO-DE

| Parameter | Nilai |
|---|---|
| F (faktor mutasi) | 0,6 |
| CR (peluang crossover) | 0,7 |
| b (bentuk spiral) | 1 |
| Jumlah api | `max(1, popSize − iter × (popSize−1)/maxIter)` |

Siklus tiap iterasi untuk setiap ngengat:

1. Pilih api (api ke-i, atau api terakhir bila api lebih sedikit).
2. Gerak spiral: `next = dist × e^(b·t) × cos(2πt) + api`, `t` acak di [−1, 1], dibulatkan lalu modulo 4.
3. DE/rand/1/bin: `trial = best + F × (a − c)` dengan peluang CR (satu dimensi acak selalu dimutasi).
4. Seleksi greedy: trial diterima jika makespan ≤ milik ngengat.

### Biaya komputasi (NFE)

ACO-SA mengevaluasi 6 jadwal per semut per iterasi (144 per iterasi untuk populasi 24), DMFO-DE 1 per ngengat (24 per iterasi). Kolom **NFE** di tabel simulator menampilkan ini supaya perbandingan adil dibaca.

## 5. Simulator balapan (Stage 3)

Dua algoritma dijalankan bersamaan dengan dataset dan seed yang sama.

**Kontrol**

- Populasi: 8–40 (default 24)
- Max iterasi: 20–100 (default 60)
- Seed: default 2026
- Play/Pause, Step (satu iterasi), Reset

**Cara membaca visual**

- **Mobil**: posisi = `1 − (makespan − batas ideal) / 4`, dibatasi 0–1. Makin dekat batas ideal, makin jauh melaju.
- **Gantt**: satu baris = satu VM, satu blok = satu task, lebar = waktu eksekusi. Garis merah putus-putus = batas ideal.
- **Grafik konvergensi**: makespan terbaik per iterasi. Tidak pernah naik karena hanya menyimpan yang terbaik.
- **Tabel**: iterasi, makespan, cost, DI, Gap ke batas ideal, NFE. Setelah selesai muncul analisis otomatis (pemenang, beban per VM, perbandingan NFE).

Generator acak memakai LCG berseed (`state × 1664525 + 1013904223 mod 2^32`), sehingga hasil bisa diulang dengan seed yang sama.

## 6. Benchmark multi-trial (Stage 6)

- 10 trial independen, seed = `2026 + trial × 37`, populasi dan iterasi mengikuti simulator.
- Tabel per trial: makespan kedua algoritma, `Δ = ACO-SA − DMFO-DE` (negatif berarti ACO-SA unggul), dan pemenang.
- Ringkasan: rata-rata, simpangan baku, terbaik, dan terburuk.
- **CSV** ditampilkan di kotak teks dengan tombol **Salin CSV** (tombol unduh file tidak dipakai karena tidak andal di halaman yang di-host). Kolom: `Trial,Seed,ACO_SA_Makespan,DMFO_DE_Makespan,Delta,Winner`.

## 7. Validasi

- Hasil ACO-SA dan DMFO-DE dibandingkan langsung dengan `aco-sa.js` dan `dmfo-de.js` dari proyek sumber: riwayat makespan **identik** pada seed 2026, 2063, dan 5 (populasi 24, 60 iterasi). Contoh seed 2026: ACO-SA 10,64 dan DMFO-DE 10,73.
- Seluruh alur 8 stage (walkthrough, kuis, Play/Step/Reset, benchmark, Main Lagi) diuji di peramban headless tanpa error.

## 8. Catatan penting

- **Perbaikan bug DMFO-DE**: pada `dmfo-de.js` sumber, `currentIter` tidak diberi nilai awal 0 di `reset()`. Akibatnya loop iterasi tidak pernah berjalan dan hanya populasi acak awal yang terlihat. Di sini sudah diperbaiki. Bila ingin memperbaiki proyek sumber, tambahkan `this.currentIter = 0;` di `reset()`.
- Perbedaan dengan proyek sumber: simulator menjalankan kedua algoritma sekaligus (sumber memilih satu algoritma aktif), ada kolom NFE, ada gamifikasi balapan, dan rumus ditulis dengan teks biasa (tanpa KaTeX).
- Penjelasan konsep dasar (analogi pendinginan logam, spiral logaritmik, dan sejenisnya) ditulis dari pengetahuan umum. Rumus, parameter, dataset, soal kuis, dan langkah walkthrough diambil dari proyek sumber.
- Dataset hanya 16 task dan 4 VM dengan eksekusi berurutan per VM (bukan model time-shared ala CloudSim). Kesimpulan dari data sekecil ini tidak otomatis berlaku untuk skala besar.
- Referensi dicantumkan sebagai nama dan tahun saja, sesuai proyek sumber.

## 9. Struktur kode (dalam satu berkas)

| Bagian | Fungsi |
|---|---|
| `VMS`, `TK`, `ev()` | Dataset Minggu 3 dan evaluasi jadwal (makespan, cost, DI, blok Gantt) |
| `LCG` | Generator acak berseed |
| `ACO`, `DMFO`, `mkPair()` | Implementasi dua algoritma dan pembuat pasangan dengan seed sama |
| `QZ` | Soal, pilihan, dan penjelasan jawaban Knowledge Check |
| `ST`, `WT` | Teks tiap stage dan langkah walkthrough |
| `wt()`, `qz()`, `simUI()`, `bmUI()`, `finUI()` | Antarmuka tiap jenis stage |
| `dg()`, `dch()`, `dst()`, `ana()` | Gantt, grafik konvergensi, tabel, dan analisis hasil |
| `scene()`, `drawCar()`, `bubble()` | Adegan balapan di canvas |
| `snd()`, `bl()`, `yay()` | Suara mesin, klakson, dan efek selesai (WebAudio) |

## 10. Kustomisasi

- **Ganti dataset**: ubah array `VMS` (id, mips, c) dan `TK` (panjang task dalam MI). Jumlah VM otomatis mengikuti `NV`. Jika jumlah VM berubah, sesuaikan juga warna di `GC` dan teks yang menyebut "4 VM".
- **Ganti parameter algoritma**: ubah nilai di kelas `ACO` (`Math.pow(...,1.5)`, `.75` untuk 1−ρ, `25` dan `.05` untuk suhu) dan `DMFO` (`.7` untuk CR, `.6` untuk F).
- **Ganti default simulator**: variabel `pop`, `mxi`, dan `sd`.

## 11. Rujukan

- Mensah, Boateng, & Asare (2021): ACO-SA.
- Anoop Kumar (2024): ACO-SA.
- Ahmed et al. (2021): DMFO-DE.
- Proyek sumber: *Hybrid-Algorithm (kelas B, kelompok 8)*, mata kuliah Strategi Optimasi Komputasi Awan (SOKA).
