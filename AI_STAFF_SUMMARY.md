# Ringkasan petugas dengan bantuan AI

Fitur ini melanjutkan bantuan AI untuk admin dan instansi. Petugas meminta ringkasan secara eksplisit pada detail laporan. Fakta inti berasal dari versi Fakta Asal aktif; AI hanya memilih maksimal tiga kutipan penting dari keterangan yang panjang. AI tidak menulis fakta baru, menyatakan sebab kerusakan, menentukan prioritas, status, instansi, atau tindakan teknis. Deskripsi asli tetap tersedia sebagai rujukan.

Keterangan singkat ditampilkan tanpa AI. Jika AI gagal, tersedia ringkasan dasar dengan label dan penjelasan yang berbeda. Ringkasan tidak disimpan menggantikan Fakta Asal dan tidak memicu email maupun perubahan status. Petugas instansi dibatasi pada laporan penugasannya; akses dan revisi laporan diperiksa kembali setelah pemrosesan. Token, kontak pelapor, catatan internal dan foto tidak dimasukkan ke prompt ringkasan; pola informasi kontak dalam teks disamarkan, dengan keterbatasan penyaringan pola yang tetap perlu dipahami.

Efisiensi: input keterangan dibatasi, keluaran hanya indeks kutipan (maksimal 256 token), cache 10 menit dengan validasi revisi, tanpa retry otomatis. Tidak diperlukan migrasi database. Rincian konfigurasi dan pengujian terdapat pada backend `sql/README-report-summary.md`.
