# ChessLens Realtime v0.2 — Aplikasi Android Stockfish untuk latihan catur

**Status saat dikirim: proyek source code Android Studio, BUKAN APK yang telah diuji.**

ChessLens Realtime adalah prototipe native Android (Java) untuk mempelajari posisi catur dengan **Stockfish 18**, menampilkan label langkah `!!`, `★`, `!`, `!?`, `?!`, `?`, `??`, serta memberi **panah langkah terakhir dan panah rekomendasi engine**. Ada dua mode: papan latihan **di dalam aplikasi**, atau **overlay Android** yang bekerja di atas aplikasi lain ketika pengguna memberikan izin merekam layar.

## Fitur yang dibuat

**Mode Papan Latihan (berfungsi tanpa screen capture)**

- Papan 8×8 interaktif; pilih bidak lalu petak tujuan. Semua langkah disaring menggunakan chesslib.
- Bisa membalik orientasi, mengulang ke posisi awal, membatalkan langkah, menguji posisi lain melalui editor FEN.
- Analisis langkah terakhir menggunakan evaluasi Stockfish sebelum dan sesudah langkah.
- Panah hijau untuk langkah dimainkan, panah biru putus-putus untuk saran berikutnya.
- Menyimpan hingga 30 posisi FEN di Riwayat lokal, lalu dapat memuatnya kembali.
- Tombol **Stockfish** untuk menganalisis posisi kapan saja.

**Mode Floating Overlay Android**

- Izin `SYSTEM_ALERT_WINDOW` + `MediaProjection` yang harus diminta secara eksplisit. Android meminta izin tiap sesi.
- Foreground Service dengan notifikasi dan tombol **Hentikan**, overlay panel dapat dipindah.
- Tombol panel **Kalibrasi**, **Jeda/Lanjut**, dan **Stop**.
- Deteksi papan dari screenshot, orientasi putih/hitam di bawah, pengambilan template visual dari **posisi awal 32 buah**.
- Pelacakan langkah memakai `Board.legalMoves()`: rokade, en passant, dan promosi mengikuti chesslib.
- Ketika langkah terdeteksi, engine mengukur perbedaan evaluasi sebelum dan sesudah langkah, mengeluarkan label, dan menggambar panah.
- Setelah kalibrasi, aplikasi juga mengevaluasi posisi awal.
- Antarmuka menganalisis foto di RAM dan **tidak mengunggah screenshot ke server**.

### Makna panah dan simbol

- **Panah hijau penuh**: langkah terakhir dinilai baik.
- **Panah oranye/merah penuh**: langkah terakhir kehilangan evaluasi.
- **Panah biru putus-putus**: rekomendasi Stockfish untuk **giliran berikutnya pada posisi sekarang** (bukan pengganti langkah yang sudah dimainkan).
- **`!!`**: tebakan *brilliant* berdasarkan pengorbanan buah + perbandingan evaluasi, bukan label Chess.com.
- **`★`, `!`, `!?`, `?!`, `?`, `??`**: klasifikasi heuristik berdasarkan selisih centipawn; klasifikasi dapat berbeda dari Chess.com.

## Cara membuat APK (tanpa menulis kode)

**Opsi paling mudah: GitHub Actions**

1. Buat repo GitHub baru (`chesslens-android`).
2. Ekstrak file ZIP proyek ini dan unggah **seluruh ISI folder** `ChessLens-Realtime-v2/` ke bagian root repo. Pastikan folder tersembunyi `.github/` ikut terunggah. `settings.gradle`, `app/`, dan `.github/` harus di root.
3. Buka tab **Actions** → **Build ChessLens Android (Stockfish 18)** → **Run workflow**.
4. Workflow mengunduh dan **memeriksa SHA256** binary ARM64 dan ARMv7 Stockfish 18 resmi, lalu memasang Android SDK dan melakukan build.
5. Jika berhasil, unduh artifact `chesslens-android-debug`, ekstrak, dan cari `app-debug.apk`.
6. Pasang APK secara manual di ponsel Android (izinkan instalasi aplikasi dari sumber tepercaya saat diminta).

**Opsi Android Studio**

- Buka folder proyek ini di Android Studio, gunakan JDK 17, SDK 34, Android Gradle Plugin 8.5.2, Gradle 8.7.
- Sebelum build, taruh binary Stockfish 18 yang sesuai ABI perangkat dalam `app/src/main/jniLibs/<abi>/libstockfish.so`. Lihat contoh unduhan dan SHA-256 pada `.github/workflows/build-android.yml`.
- Build > Build APK(s). Aplikasi tidak memerlukan layanan cloud pada runtime.
- Proyek belum menyertakan Gradle wrapper JAR; di CI digunakan Gradle yang dipasang oleh workflow.

## Cara menggunakan

1. Jalankan ChessLens. Untuk belajar tanpa kamera layar, pilih **Papan Latihan + FEN + Riwayat**.
2. Untuk overlay, aktifkan **Izin overlay sistem**, kemudian pilih **Mulai tangkap layar + analisis**.
3. Berikan persetujuan capture layar penuh yang diminta Android. Kembali ke aplikasi catur yang menampilkan **posisi awal standar**.
4. Tekan **Kalibrasi** di HUD. Setelah kalibrasi berhasil, setiap langkah yang dikenali selama papan tetap tampak akan dinilai otomatis.
5. Gunakan **Jeda** saat tidak ingin memantau papan. Gunakan **Stop** atau tombol notifikasi untuk mengakhiri tangkap layar.
6. Jika bidak tidak dikenali, tema papan berubah, atau orientasi/ukuran layar berganti, tampilkan posisi awal dan kalibrasi ulang.

## Batasan dan keamanan

1. **Belum dikompilasi menjadi APK atau diuji pada ponsel** dalam lingkungan penyusunan ini. Pemeriksaan lokal berupa parsing sintaks Java dan konsistensi file, bukan build Android lengkap. Kesalahan integrasi perangkat/Gradle masih mungkin terjadi.
2. Deteksi gambar menggunakan pendekatan ringan (pola papan + template yang dipelajari saat papan awal), **bukan model computer vision universal**. Papan 3D, animasi, highlight, modifikasi tema, orientasi berubah, serta screenshot yang tidak jelas dapat menghasilkan false-positive atau gagal deteksi.
3. **Tidak dapat membaca posisi sembarang di tengah permainan tanpa kalibrasi dari posisi awal standar.** Mode FEN di dalam aplikasi bisa menganalisis posisi yang dimasukkan secara manual.
4. MediaProjection tidak boleh dan tidak dapat mengabaikan perlindungan screenshot `FLAG_SECURE` dari aplikasi lain. Android dapat menghentikan capture saat layar terkunci; persetujuan sesi harus diminta lagi.
5. Pada beberapa ROM/vendor, proses bisa dihentikan karena optimasi baterai. Jika capture berhenti, buka aplikasi dan jalankan kembali.
6. Indikator `!!` bersifat perkiraan algoritmik, **tidak setara dengan Chess.com Brilliant**. Centipawn loss tidak selalu menyatakan tingkat kesulitan atau kemampuan melihat pengorbanan.
7. Capture interval kira-kira 1,15 detik ditambah waktu analisis engine dua posisi. **Bukan zero-latency**; langkah yang terjadi terlalu cepat dapat terlewat.
8. Gunakan untuk studi, puzzle, game melawan engine, atau analisis yang diperbolehkan. Jangan gunakan untuk mendapatkan bantuan engine dalam pertandingan dengan manusia yang melarang bantuan eksternal.

## Struktur kode

```
app/src/main/java/id/chesslens/realtime/
├── MainActivity.java                    # izin & launcher
├── ScreenService.java                   # MediaProjection, capture, engine
├── practice/PracticeActivity.java       # papan latihan + FEN + Riwayat
├── engine/UciEngine.java                # komunikasi UCI Stockfish
├── engine/MoveJudge.java                # klasifikasi !!/★/!/!?/?!/?/??
├── vision/BoardFinder.java              # deteksi area persegi 8x8
├── vision/PositionTracker.java          # kalibrasi / pelacakan langkah legal
├── overlay/ArrowCanvas.java             # panah & label di atas papan
└── overlay/FloatingOverlay.java         # HUD dapat digeser, jeda, stop
```

## Lisensi dan rujukan

- Kode yang ditulis untuk proyek ini menggunakan lisensi MIT (lihat `LICENSE`). Distribusi APK yang menggabungkan Stockfish perlu memenuhi kewajiban **GPL-3.0** untuk karya gabungan dan kode sumber yang sesuai.
- Stockfish 18 (GPLv3): https://github.com/official-stockfish/Stockfish/releases/tag/sf_18
- Source Stockfish: https://github.com/official-stockfish/Stockfish/tree/sf_18
- chesslib (Apache-2.0): https://github.com/bhlangonijr/chesslib
- Android MediaProjection: https://developer.android.com/media/grow/media-projection

ChessLens tidak berafiliasi dengan Chess.com dan tidak mengklaim menggunakan algoritma Game Review resmi mereka.


## GitHub Actions build

The workflow at `.github/workflows/build-android.yml` installs Android API 34 and Build Tools 34 using the SDK command-line tools already present on the GitHub-hosted runner. It intentionally does not call `android-actions/setup-android`, whose default request for the retired SDK package `tools` caused the first build attempt to fail.
