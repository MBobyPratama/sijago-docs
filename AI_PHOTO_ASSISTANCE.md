# Addendum: bantuan pengisian dari foto

Perubahan cakupan atas permintaan pemilik proyek: analisis foto kini diizinkan sebatas **saran kategori kerusakan, perkiraan kelas ukuran, dan kondisi yang tampak**, atas tindakan eksplisit warga. Addendum ini memperbarui pembatasan analisis foto pada BRL-02 dan daftar di luar cakupan dalam dokumen awal. Larangan pengukuran otomatis yang pasti, penentuan instansi, keputusan verifikasi/status, dan pengiriman otomatis tetap berlaku.

Alur: unggah foto → klik Analisis foto dengan AI → tinjau → Terapkan saran ke formulir → warga memperbaiki/melengkapi Fakta Asal → tinjau dan kirim dengan template. Warga diberi tahu bahwa foto akan dikirim ke OpenAI. Pengisian manual tetap tersedia bila AI tidak tersedia. Hasil AI tidak disimpan sebagai Fakta Asal sampai warga menerapkannya dan menyimpan formulir melalui alur biasa.

AI dapat mengembalikan kategori/ukuran kosong bila bukti visual kurang. Tanpa skala yang jelas, kelas ukuran tidak disarankan. Tidak ada estimasi ukuran sentimeter atau kedalaman, identifikasi orang, penyalinan pelat kendaraan, inferensi lokasi/waktu, tuduhan atau penentuan penyebab. Saran selalu bisa diubah. Detail teknis dan konfigurasi terdapat di backend `sql/README-photo-analysis.md`.

## Pembaruan moderasi wajib

Keamanan dan relevansi foto diperiksa otomatis saat registrasi foto dan kembali ditegakkan saat penyimpanan fakta/pengiriman final, termasuk pengisian manual. Ini memperluas ketentuan sebelumnya bahwa foto hanya diproses AI setelah tombol analisis ditekan: tombol tersebut kini hanya mengendalikan pembuatan **saran isian**, sementara moderasi merupakan syarat penggunaan bukti. Foto yang ditolak tidak didaftarkan sebagai bukti laporan; aset direct-upload masih dapat tersimpan di Cloudinary dan memerlukan kebijakan cleanup tersendiri.

Backend memakai moderasi multimodal sebelum klasifikasi relevansi. Bila pemeriksaan gagal, foto tidak diloloskan. Cache konten singkat digunakan untuk menghindari pemeriksaan berulang. Hasil saran tetap harus diterapkan secara eksplisit oleh warga; pengukuran pasti dan keputusan status/instansi tetap di luar wewenang AI.

## Addendum: saran duplikat untuk admin

AI kini dapat membandingkan foto/keterangan laporan secara opsional untuk membantu admin menemukan duplikat. Ini memperluas batasan analisis foto pada dokumen awal, sebatas saran yang wajib ditinjau manusia. Tombol **Cari saran dengan AI** berada di moderasi admin; hasil dapat mengisi pilihan tiket induk, tetapi tidak memicu penggabungan. Penggabungan tetap melalui tombol dan modal konfirmasi yang ada.

Kandidat dibatasi maksimal 3 laporan aktif terdekat dalam radius 100 meter, bukan laporan selesai, ditutup, ditolak, duplikat, atau draf. Perbandingan mempertimbangkan waktu agar kerusakan baru setelah perbaikan tidak dianggap episode yang sama. Foto dimoderasi sebelum dipakai. Foto yang tidak tersedia/gagal diperiksa dilaporkan sebagai keterbatasan. Hasil kosong tidak membuktikan laporan unik. Tidak ada model yang menentukan status, instansi, atau pengiriman secara otomatis.
