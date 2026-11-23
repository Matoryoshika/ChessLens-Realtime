# Dapatkan APK ChessLens (tanpa Android Studio)

**ZIP ini adalah source project, bukan APK.** Workflow GitHub Actions sudah disesuaikan agar memakai Android SDK yang tersedia di runner GitHub (tanpa memanggil paket SDK lama bernama `tools`).

## Cara memasukkan semua source ke repository

1. Buka repository `Matoryoshika/ChessLens-Realtime`.
2. Ekstrak ZIP ini di ponsel atau komputer.
3. Unggah **seluruh isi folder hasil ekstrak** ke root repository, bukan ZIP sebagai satu file. Pertahankan struktur folder: `app/`, `.github/workflows/`, serta file `settings.gradle`, `build.gradle`, dan `gradle.properties`.
4. Jika GitHub bertanya apakah file `.github/workflows/build-android.yml` akan ditimpa, pilih untuk mengganti file tersebut.
5. Commit perubahan. Buka tab **Actions** dan pilih workflow **Build ChessLens APK (Stockfish 18)**.
6. Jika workflow selesai dengan status hijau, buka halaman run dan unduh artifact `ChessLens-Realtime-APK`, atau buka **Releases** untuk file `ChessLens-Realtime-debug.apk`.

## Catatan penting

- Jangan hanya mengunggah folder `.github`; source Android utama berada di folder `app/` dan file Gradle pada root.
- APK belum dianggap berhasil sampai workflow hijau dan artifact APK benar-benar tersedia.
- APK debug ditujukan untuk pengujian. Pengujian di perangkat fisik tetap diperlukan, terutama untuk izin overlay, MediaProjection, pengenalan papan, dan performa engine.
- Overlay/engine digunakan untuk latihan, puzzle, dan analisis setelah pertandingan atau konteks lain yang memperbolehkannya; bukan untuk bantuan saat pertandingan online yang melarang engine.
