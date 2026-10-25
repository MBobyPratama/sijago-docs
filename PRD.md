**PRODUCT REQUIREMENTS DOCUMENT**

**SIJAGO**

**Sistem Jalan Bogor**

Platform Pelaporan Kerusakan Jalan Kota Bogor  \|  Civic-Tech & Weather-Road Intelligence

| Atribut | Keterangan |
| :---- | :---- |
| **Versi dokumen** | 1.0 (Final — Disesuaikan untuk Kompetisi MIA2026) |
| **Tanggal** | 25 Oktober 2026 |
| **Status** | Final (v1.0). Seluruh keputusan produk, cakupan Fase 1 (F1–F10), dan spesifikasi teknis telah dikunci untuk pengembangan proyek kompetisi MIA2026. |
| **Pemilik dokumen** | **Sabira** (Product Manager & UI/UX Designer)<br>**Boby** (System Architect & Fullstack Developer) |
| **Konteks proyek** | Proyek Kompetisi **MIA2026** (Independen, menggunakan layanan Free Tier & simulasi kemitraan instansi) |
| **Model operasi** | Hibrida: Warga melapor, Admin SIJAGO memverifikasi dan meneruskan, Instansi (disimulasikan) mengeksekusi |

| Cara membaca dokumen ini: PRD ini merinci kebutuhan fungsional, aturan bisnis sistem, skema data, penanganan edge case, dan arsitektur teknis SIJAGO. Seluruh fitur F1 hingga F10 berada di Fase 1 (MVP Utama) untuk kebutuhan demonstrasi kompetisi MIA2026. |
| :---- |

# **Riwayat Dokumen & Daftar Isi**

| Versi | Tanggal | Perubahan |
| :---- | :---- | :---- |
| 0.1 | 25 Okt 2026 | Draft awal PRD dari Product Summary & Scope. |
| 0.2 | 25 Okt 2026 | Rekomendasi default PM diterapkan sebagai keputusan dan asumsi kerja (Bagian 10). FR diselaraskan: OTP untuk tag, blur manual foto, email tanpa identitas pelapor, ambang sign-off. |
| 0.3 | 25 Okt 2026 | Nama produk SIJAGO ditetapkan (OQ-27). Bagian 12 Arsitektur dan Tech Stack ditambahkan. OQ-29 dan OQ-30 ditambahkan. |
| 1.0 | 25 Okt 2026 | Dokumen dikunci untuk kompetisi MIA2026 oleh Sabira & Boby. F9 (Weather-Road Intelligence) dan F10 (Community Sign-Off) ditarik ke Fase 1 (MVP Utama). Model AI ditetapkan menggunakan `openai gpt-6-luna` (*reasoning medium*). Peta menggunakan MapLibre GL JS + OpenStreetMap, Object Storage menggunakan Cloudinary, Email menggunakan Resend, daftar API cuaca gratis ditambahkan, dan arsitektur repositori ditetapkan menjadi Multirepo pada layanan Free Tier. |

1\. Overview & Problem Statement

2\. Goals & Success Metrics

3\. Pengguna, Peran, dan Cakupan MVP

4\. User Stories & Acceptance Criteria

5\. Functional Requirements per Fitur MVP

6\. Model Status Laporan

7\. Non-Functional Requirements

8\. Sketsa Data Model

9\. Edge Case & Failure Matrix

10\. Keputusan Produk yang Dikunci

11\. Risiko, Asumsi, dan Rencana Rilis

12\. Arsitektur dan Tech Stack

# **1\. Overview & Problem Statement**

## **1.1 Ringkasan Produk**

**SIJAGO (Sistem Jalan Bogor)** adalah platform web **mobile-first** untuk melaporkan kerusakan jalan di Kota Bogor yang dikembangkan untuk kompetisi **MIA2026**. Produk ini menggabungkan tiga pilar utama:
1. **Partisipasi Warga & Komunitas:** Lapor tanpa login di awal, dukung (+1), tambah tag kondisi, dan verifikasi hasil perbaikan (*Community Sign-Off*).
2. **Weather-Road Intelligence:** Analitik korelasi curah hujan dari sumber data cuaca terbuka dengan titik kerusakan jalan dan deteksi lubang kambuhan untuk membantu instansi memprioritaskan perbaikan.
3. **AI Drafting dengan Human-in-the-Loop:** Penyusunan judul dan deskripsi baku menggunakan model **OpenAI `gpt-6-luna` (*reasoning medium*)** yang aman, akuntabel, bebas halusinasi, dan sepenuhnya dapat diaudit.

Alur kerjanya hibrida. Warga melapor, Admin SIJAGO memverifikasi lalu meneruskan secara manual ke instansi yang berwenang, dan Dinas PUPR atau instansi terkait (disimulasikan di dalam sistem) mengeksekusi perbaikan serta melaporkan bukti pengerjaan.

## **1.2 Latar Belakang dan Problem Statement**

Kota Bogor dikenal sebagai Kota Hujan. Intensitas hujan yang tinggi mempercepat kerusakan permukaan jalan, terutama di titik dengan drainase buruk. Dari sisi warga dan instansi, masalah utama yang diselesaikan SIJAGO adalah sebagai berikut:

| Pihak | Masalah Utama | Dampak |
| :---- | :---- | :---- |
| **Warga / komuter** | Laporan sulit disampaikan dengan lengkap (lokasi, ukuran, dampak sering tidak jelas), tidak tahu laporan sampai ke instansi yang benar, dan tidak ada kejelasan status maupun bukti tindak lanjut | Warga enggan melapor ulang, kepercayaan pada kanal aduan turun |
| **Instansi (PUPR dan lainnya)** | Laporan tersebar dan formatnya tidak seragam, laporan salah alamat karena kewenangan ruas jalan berbeda (kota, provinsi, nasional), serta sulit menentukan prioritas karena tidak ada konteks frekuensi dan cuaca | Perbaikan reaktif, lubang kambuhan di titik yang sama |
| **Pengelola platform (Admin)** | Risiko spam, foto palsu, dan laporan duplikat, serta risiko teks laporan yang memuat tuduhan atau klaim yang tidak diverifikasi | Reputasi platform dan hubungan dengan instansi terganggu |

### **Problem statement**

Warga Kota Bogor yang menemukan jalan rusak kesulitan menyampaikan laporan yang lengkap, ke instansi yang tepat, dan memantau tindak lanjutnya. Di sisi lain, instansi kesulitan menentukan prioritas karena laporan tidak terstruktur dan tidak dilengkapi konteks cuaca. Akibatnya perbaikan bersifat reaktif dan kerusakan berulang di titik yang sama.

### **How Might We**

* Bagaimana membuat warga bisa melapor dalam hitungan menit dengan data yang cukup lengkap untuk ditindaklanjuti?
* Bagaimana memakai AI (`openai gpt-6-luna`) untuk merapikan teks tanpa risiko mengarang fakta dan tanpa mengambil keputusan atas nama manusia?
* Bagaimana membuat setiap laporan punya jejak yang bisa dipertanggungjawabkan, dari dibuat sampai diperbaiki dan diverifikasi warga?
* Bagaimana membantu instansi melihat titik kerusakan yang berulang di zona hujan ekstrem agar perbaikan lebih tahan lama?

## **1.3 Prinsip Produk**

| Prinsip | Artinya dalam produk |
| :---- | :---- |
| **Fakta Asal adalah sumber kebenaran** | Isian terstruktur warga (foto, GPS, form) adalah data utama. Draf AI hanyalah rangkuman teks atas data tersebut. |
| **AI membantu, manusia memutuskan** | AI tidak menganalisis foto, tidak memilih instansi, tidak menilai kebenaran, dan tidak mengubah status. |
| **Aksi eksternal hanya lewat konfirmasi manusia** | Email ke instansi dan perubahan status menjadi Diteruskan hanya terjadi saat Admin menekan tombol konfirmasi. |
| **Semua tercatat** | Setiap aksi penting masuk audit log yang tidak bisa diubah, lengkap dengan snapshot perubahan. |
| **Gagal dengan aman** | Jika AI atau layanan eksternal gagal, sistem turun ke perilaku deterministik, bukan berhenti atau menebak. |

## **1.4 Di Luar Lingkup (Out of Scope) MVP**

* AI yang menganalisis foto, mengukur kerusakan, atau mengklasifikasi otomatis.
* Pengiriman email otomatis ke instansi tanpa konfirmasi Admin.
* Integrasi langsung ke server internal pemerintah (kemitraan instansi disimulasikan pada proyek kompetisi MIA2026).
* Aplikasi native iOS/Android (MVP berupa web mobile-first).
* Notifikasi WhatsApp, sistem poin atau gamifikasi, dan pembayaran apa pun.
* Pelaporan di luar batas administratif Kota Bogor.

# **2\. Goals & Success Metrics**

## **2.1 Tujuan Produk**

| ID | Tujuan | Indikator keberhasilan utama |
| :---- | :---- | :---- |
| **G1** | Memudahkan warga membuat laporan yang lengkap dan terlacak | Completion rate form, waktu pengisian, kelengkapan data |
| **G2** | Meningkatkan kualitas dan ketepatan alamat laporan ke instansi | Persentase laporan terverifikasi, tingkat salah routing |
| **G3** | Membuktikan bahwa AI dapat membantu tanpa halusinasi dan tanpa efek samping | Insiden halusinasi, tingkat fallback, persetujuan draf |
| **G4** | Mendorong tindak lanjut instansi yang terlihat dan terverifikasi komunitas | Waktu respons simulasi, tingkat selesai, hasil sign-off |
| **G5** | Memberi instansi wawasan hujan vs kerusakan untuk prioritisasi | Visualisasi layer cuaca & kerusakan, titik kambuhan yang teridentifikasi |
| **G6** | Menjamin akuntabilitas dan kepatuhan lewat jejak audit | Cakupan audit log, hasil audit sampling |

## **2.2 Success Metrics**

| Metrik | Definisi | Target | Tujuan |
| :---- | :---- | :---- | :---- |
| **North Star: Laporan Tuntas Terverifikasi** | Jumlah laporan yang mencapai status Selesai dan tidak dibuka kembali selama masa pemantauan | Siklus penuh terbukti pada demo & pengujian | G4 |
| **Waktu pengisian laporan** | Median dari buka form sampai kirim | ≤ 3 menit | G1 |
| **Completion rate form** | Sesi yang memulai form dan berhasil mengirim | ≥ 60% | G1 |
| **Kelengkapan laporan** | Laporan dengan foto, GPS valid, kategori, dan ukuran terisi | ≥ 90% | G1 |
| **Waktu verifikasi Admin** | Median dari Menunggu Verifikasi sampai keputusan Admin | ≤ 2 hari kerja | G2 |
| **Tingkat salah routing** | Laporan yang dikembalikan instansi karena bukan kewenangannya | ≤ 10% | G2 |
| **Tingkat spam/tolak** | Laporan ditolak karena spam atau tidak valid dari total masuk | Terpantau di dashboard Admin | G2 |
| **Insiden halusinasi AI** | Draf yang memuat fakta tidak ada di Fakta Asal (audit sampling) | 0 kasus lolos ke laporan terkirim | G3 |
| **Tingkat fallback** | Draf yang jatuh ke Template dari total permintaan draf | ≤ 5% (di luar gangguan besar) | G3 |
| **Tingkat persetujuan draf** | Draf yang disetujui warga tanpa membuat ulang | ≥ 70% | G3 |
| **Partisipasi komunitas** | Rata-rata +1 per laporan terverifikasi, dan jumlah sign-off per laporan selesai | Terukur di halaman publik & dashboard | G4 |
| **Fungsi Dashboard Cuaca** | Layer curah hujan, zona merah, dan titik kambuhan tampil akurat | 100% skenario analitik berjalan | G5 |
| **Cakupan audit log** | Aksi wajib yang punya entri log | 100% | G6 |

### **Guardrail metrics (tidak boleh memburuk)**

* Laporan salah kirim ke instansi tanpa konfirmasi Admin harus 0.
* Latensi halaman form (LCP) tidak melewati 2,5 detik di jaringan 4G untuk 75% sesi (setelah *warm-up* layanan).
* Pelanggaran privasi (data pribadi terpapar publik) harus 0.

# **3\. Pengguna, Peran, dan Cakupan MVP**

## **3.1 Persona**

| Persona | Perilaku utama | Kebutuhan utama |
| :---- | :---- | :---- |
| **Warga / Komuter / Pelapor** | Menemukan kerusakan jalan di lapangan, sering saat bergerak, memakai ponsel dengan koneksi bervariasi | Form cepat tanpa login di awal, GPS otomatis, bantuan merapikan teks, bisa memantau status, dan bisa ikut mendukung atau memverifikasi hasil perbaikan |
| **Admin Pengelola SIJAGO** | Moderator independen yang menjaga kualitas laporan | Antrean moderasi, alat verifikasi, routing manual, dan kontrol atas pengiriman email resmi |
| **Mitra Instansi (Simulasi Dinas PUPR Kota Bogor / Pemprov / Balai Nasional)** | Petugas yang menindaklanjuti perbaikan | Laporan terkurasi, dashboard cuaca dan titik kambuhan, serta cara mudah memperbarui status dan mengunggah bukti |

## **3.2 Peran dan Hak Akses**

Tanda ✓ berarti diizinkan, ◐ berarti diizinkan dengan batasan yang dijelaskan di FR.

| Aksi | Warga (tanpa akun) | Admin | Petugas Instansi | Auditor | Super Admin |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Membuat laporan dan memakai AI draf** | ✓ | \- | \- | \- | \- |
| **Melihat peta dan laporan terverifikasi** | ✓ | ✓ | ◐ (miliknya) | ✓ | ✓ |
| **+1 (dukungan)** | ✓ | \- | \- | \- | \- |
| **Tag kondisi** | ◐ (OTP email) | \- | \- | \- | \- |
| **Sign-off hasil perbaikan** | ◐ (eligible + OTP email) | \- | \- | \- | \- |
| **Verifikasi, tolak, gabung duplikat** | \- | ✓ | \- | \- | ✓ |
| **Memilih instansi dan mengirim eskalasi** | \- | ✓ | \- | \- | ✓ |
| **Mengubah Fakta Asal pelapor** | ◐ (sebelum kirim) | \- | \- | \- | \- |
| **Update status dan unggah bukti perbaikan** | \- | ◐ (override tercatat) | ✓ (miliknya) | \- | \- |
| **Dashboard cuaca dan analitik** | \- | ✓ | ✓ | ✓ | ✓ |
| **Membaca audit log dan version history** | \- | ✓ | \- | ✓ | ✓ |
| **Kelola instansi, pengguna, dan konfigurasi** | \- | \- | \- | \- | ✓ |

## **3.3 Daftar Fitur dan Prioritas**

Untuk kompetisi **MIA2026**, seluruh fitur **F1 sampai F10** ditarik masuk ke dalam **Fase 1 (MVP Utama)**. Prioritas: **P0** adalah alur inti, **P1** adalah fitur pembeda utama & pengayaan pengalaman yang juga diimplementasikan pada Fase 1.

| ID | Fitur | Sisi | Prioritas | Fase |
| :---- | :---- | :---- | :---- | :---- |
| **F1** | Form Pelaporan & Geotagging | Warga | P0 | 1 |
| **F2** | AI Draft Assistant (`openai gpt-6-luna`) & Fallback Template | Warga | P0 | 1 |
| **F3** | Panel Sanding & Persetujuan (Human-in-the-Loop) | Warga | P0 | 1 |
| **F4** | Peta Interaktif, Upvote (+1), dan Tag Kondisi | Komunitas | P0 | 1 |
| **F5** | Moderasi, Routing, dan Eskalasi Admin | Admin | P0 | 1 |
| **F6** | Portal Instansi (Simulasi): Status dan Bukti Perbaikan | Instansi | P0 | 1 |
| **F7** | Audit Trail & Version History | Sistem | P0 | 1 |
| **F8** | Pelacakan Status & Notifikasi | Warga | P0 (halaman), P1 (notifikasi) | 1 |
| **F9** | Weather-Road Intelligence (Korelasi Cuaca & Kerusakan) | Instansi | P0 | 1 |
| **F10** | Community Sign-Off (Verifikasi Hasil Perbaikan) | Komunitas | P0 | 1 |

# **4\. User Stories & Acceptance Criteria**

Acceptance criteria memakai pola Given-When-Then yang dipadatkan. Setiap story dipetakan ke fitur (F) dan prioritas (P). Nilai ambang seperti radius dan durasi dapat dikonfigurasi (lihat Bagian 8, tabel `system_config`).

| ID | Fitur · Prioritas | User story | Acceptance criteria |
| :---- | :---- | :---- | :---- |
| **US-01** | F1 · P0 | Sebagai **warga**, saya ingin melapor jalan rusak tanpa harus membuat akun, agar prosesnya cepat saat saya masih di lokasi. | Given saya membuka halaman utama, when saya menekan "Lapor", then form tampil tanpa login. Given form terbuka, when saya mengizinkan lokasi, then titik GPS terkunci otomatis dan akurasinya (meter) ditampilkan. Given koordinat berada di luar batas Kota Bogor, when saya lanjut, then sistem menolak dengan pesan yang jelas. Given izin lokasi ditolak, when saya lanjut, then saya dapat menandai titik secara manual di peta. Given foto atau kategori belum terisi, when saya lanjut, then field yang kurang ditandai dan tidak ada data yang hilang. |
| **US-02** | F2 · P0 | Sebagai **warga**, saya ingin AI merapikan isian saya menjadi judul dan deskripsi baku, agar laporan mudah dipahami petugas. | Given fakta sudah lengkap, when saya menekan "Buat Draf", then judul dan deskripsi dari model `openai gpt-6-luna` (*reasoning medium*) tampil dengan label `✨ [Draf Dibantu AI]` dalam batas waktu yang ditetapkan. Given draf tampil, then seluruh angka, ukuran, dan patokan di draf identik dengan isian saya, dan tidak ada nama instansi atau tuduhan baru. Given draf sedang dibuat, then tidak ada email terkirim dan status laporan tidak berubah. |
| **US-03** | F2 · P0 | Sebagai **warga**, saya tetap ingin mendapat draf yang rapi walau layanan AI sedang bermasalah. | Given API AI timeout, error, atau hasilnya gagal validasi, when draf diminta, then sistem menampilkan penjelasan singkat dan otomatis menampilkan `📄 [Draf Template]`. Given fallback terjadi, then log `AI_FALLBACK_TEMPLATE` tercatat beserta alasannya. Given Draf Template tampil, then saya bisa lanjut ke tinjauan tanpa mengulang input. |
| **US-04** | F3 · P0 | Sebagai **warga**, saya ingin meninjau draf berdampingan dengan data asli saya sebelum mengirim, agar saya yakin isinya benar. | Given draf tersedia, when saya membuka tinjauan, then Fakta Asal dan Draf tampil berdampingan dengan label draf yang permanen. Given saya belum mencentang persetujuan, then tombol Kirim terkunci. Given saya mencentang persetujuan, then tombol Kirim aktif dan persetujuan tercatat bersama hash Fakta Asal. |
| **US-05** | F3 · P0 | Sebagai **warga**, saya ingin sistem memaksa saya meninjau ulang bila data saya berubah setelah draf dibuat, agar draf selalu sesuai fakta. | Given saya sudah menyetujui draf, when saya mengubah foto, GPS, atau field fakta apa pun, then persetujuan batal otomatis, tombol Kirim terkunci, dan banner menjelaskan alasannya. Given persetujuan batal, then draf lama ditandai kedaluwarsa dan tidak dapat dikirim. Given perubahan terjadi, then log `SOURCE_FACTS_MODIFIED` dan `APPROVAL_RESET` tersimpan dengan snapshot diff. Given ada upaya mengirim lewat panggilan API langsung saat persetujuan tidak valid, then server menolak (bukan hanya UI). |
| **US-06** | F1/F3 · P0 | Sebagai **warga**, saya ingin menerima kode tiket setelah mengirim, agar saya bisa memantau laporan. | Given persetujuan valid, when saya menekan Kirim, then laporan berstatus Menunggu Verifikasi dan kode tiket serta tautan pelacakan ditampilkan. Given koneksi terputus saat mengirim, when saya mencoba lagi, then tidak terbentuk laporan ganda (idempotency key). |
| **US-07** | F4 · P0 | Sebagai **warga**, saya ingin melihat titik laporan di peta, agar saya tahu masalah yang sudah dilaporkan. | Given saya membuka peta (MapLibre GL JS + OpenStreetMap), then hanya laporan terverifikasi yang tampil, berwarna sesuai status dan dikelompokkan (cluster) saat zoom jauh. Given saya menekan sebuah titik, then detail menampilkan foto, judul, deskripsi dengan label draf, status, dan jumlah +1. Given saya memakai filter status, kategori, atau waktu, then peta diperbarui. |
| **US-08** | F4 · P0 | Sebagai **warga**, saya ingin menekan "+1 Saya Lewat Sini" pada laporan yang sudah ada, agar prioritasnya naik tanpa membuat laporan baru. | Given laporan aktif, when saya menekan +1, then hitungan bertambah 1 dan tombol berubah menjadi status sudah mendukung. Given saya sudah memberi +1, then saya tidak dapat memberi lagi (unik per perangkat atau akun) tetapi boleh membatalkan. Given batas laju terlampaui, then aksi ditolak dan dicatat. |
| **US-09** | F4 · P1 | Sebagai **warga**, saya ingin menambahkan tag kondisi seperti `[Tertutup Genangan Air]`, agar petugas tahu kondisi nyata di lapangan. | Given laporan aktif, when saya memilih tag dari daftar yang tersedia, then tag tersimpan dan tampil sebagai hitungan. Given saya belum terverifikasi, when saya menambah tag, then saya diminta verifikasi OTP email ringan terlebih dahulu. Given saya sudah memilih tag yang sama pada laporan itu, then pilihan kedua ditolak. |
| **US-10** | F8 · P0 | Sebagai **warga**, saya ingin melihat perkembangan laporan saya, agar saya tidak perlu bertanya ke siapa pun. | Given saya membuka tautan pelacakan, then timeline status dengan waktu, instansi tujuan (setelah Diteruskan), dan bukti perbaikan (jika ada) tampil. Given saya mengisi email dan menyetujui notifikasi, when status utama berubah, then saya menerima email netral via Resend berisi status terbaru (P1). |
| **US-11** | F5 · P0 | Sebagai **Admin**, saya ingin memeriksa keaslian laporan sebelum diteruskan, agar instansi hanya menerima laporan yang layak. | Given antrean moderasi, when saya membuka sebuah laporan, then saya melihat Fakta Asal, draf beserta labelnya, foto, metadata EXIF, peta, dan laporan serupa di sekitarnya. Given saya menekan Verifikasi, then status menjadi Terverifikasi, log `VERIFY_REPORT` tercatat, dan laporan tampil di peta publik. Given saya menolak laporan, then saya wajib memilih alasan, status menjadi Ditolak, dan pelapor dapat melihat alasannya. Admin tidak dapat mengubah Fakta Asal. Koreksi dilakukan lewat catatan Admin yang terpisah. |
| **US-12** | F5 · P0 | Sebagai **Admin**, saya ingin menggabungkan laporan duplikat, agar instansi tidak menerima laporan berulang untuk titik yang sama. | Given laporan baru berada dalam radius R meter dari laporan aktif berkategori sama, then sistem menandainya sebagai kandidat duplikat. Given saya menggabungkan, then +1 dan tag dipindahkan ke laporan induk, laporan duplikat tertaut ke induk, dan log `DUPLICATE_MERGED` tercatat. |
| **US-13** | F5 · P0 | Sebagai **Admin**, saya ingin menentukan instansi tujuan dan mengirim eskalasi secara manual, agar saya mengendalikan apa yang dikirim atas nama platform. | Given laporan Terverifikasi, when saya memilih instansi dari dropdown, then pilihan tersimpan tanpa mengirim email. Given saya membuka pratinjau email, then subjek, isi, label draf, dan lampiran atau tautan foto terlihat. Given saya menekan "Kirim ke Instansi" lalu mengonfirmasi dialog, then email terkirim via Resend, status menjadi Diteruskan, dan log `ESCALATION_EMAIL_SENT` tercatat. Given email gagal terkirim, then status tidak berubah menjadi Diteruskan, saya diberi tahu, dan saya dapat mencoba lagi. Sistem atau AI tidak pernah memilih instansi atau mengirim email sendiri. |
| **US-14** | F6 · P0 | Sebagai **petugas instansi (simulasi)**, saya ingin melihat dan memperbarui status laporan milik instansi saya, agar warga tahu perkembangannya. | Given saya login, then saya hanya melihat laporan yang diteruskan ke instansi saya. Given saya mengubah status menjadi Diproses, then timeline publik diperbarui dan log `STATUS_CHANGED` tercatat. Given laporan bukan kewenangan instansi saya, when saya memilih "Kembalikan ke Admin" dengan alasan, then Admin menerima notifikasi untuk routing ulang. |
| **US-15** | F6 · P0 | Sebagai **petugas instansi (simulasi)**, saya ingin mengunggah foto bukti perbaikan, agar penyelesaian bisa diverifikasi. | Given saya menandai Selesai, then minimal 1 foto sesudah perbaikan wajib diunggah ke Cloudinary dan foto sebelum otomatis diambil dari laporan. Given lokasi EXIF foto sesudah lebih dari 100 meter dari titik laporan, then peringatan tidak memblokir tampil dan tanda flag tercatat. |
| **US-16** | F9 · P0 | Sebagai **petugas instansi / Admin**, saya ingin melihat curah hujan berdampingan dengan titik jalan berlubang, agar saya bisa memprioritaskan zona rawan. | Given saya membuka dashboard, then peta tumpang susun curah hujan dan frekuensi laporan tampil lengkap dengan waktu pembaruan data cuaca. Given saya mengubah filter periode atau kategori, then kedua layer diperbarui. Given API cuaca eksternal sedang gangguan, then data snapshot terakhir tetap tampil dengan penanda "Data usang". |
| **US-17** | F9 · P0 | Sebagai **petugas instansi / Admin**, saya ingin sistem menyorot titik kambuhan di zona hujan ekstrem, agar saya bisa merencanakan perbaikan yang lebih tahan lama. | Given terdapat titik kambuhan di zona hujan ekstrem, then titik disorot dengan aturan pemicu yang bisa dilihat. Given rekomendasi teknis tampil, then rekomendasi diberi label indikatif dan bukan pengganti survei teknis. Given saya menekan ekspor, then data dalam filter aktif terunduh sebagai CSV. |
| **US-18** | F10 · P0 | Sebagai **warga**, saya ingin menilai kualitas tambalan setelah perbaikan, agar kualitas pekerjaan ikut diawasi. | Given laporan Selesai dan masih dalam masa pemantauan, when pengguna yang eligible memilih "Mulus" atau "Cepat Rusak", then rating tersimpan (satu per pengguna per laporan). Given ambang "Cepat Rusak" tercapai, then status menjadi Dibuka Kembali dan Admin dinotifikasi. Given masa pemantauan berakhir tanpa ambang tercapai, then status menjadi Ditutup. |
| **US-19** | F7 · P0 | Sebagai **Auditor atau Admin**, saya ingin menelusuri riwayat lengkap sebuah laporan, agar akuntabilitas dapat dibuktikan. | Given saya membuka riwayat laporan, then semua event tampil berurutan waktu dengan diff versi Fakta Asal dan draf. Given ada upaya mengubah atau menghapus entri log, then ditolak dan upayanya tercatat. Given saya mengekspor log, then aksi ekspor itu sendiri tercatat. |
| **US-20** | F7 · P1 | Sebagai **Super Admin**, saya ingin mengatur parameter sistem tanpa mengubah kode, agar ambang dan aturan bisa disesuaikan. | Given saya mengubah parameter (radius duplikat, masa pemantauan, ambang Cepat Rusak, batas generate AI), then nilai baru berlaku untuk kejadian berikutnya dan perubahan tercatat di log. |

# **5\. Functional Requirements per Fitur MVP**

Seluruh fitur **F1 hingga F10** diimplementasikan pada **Fase 1 (MVP Utama)** untuk kompetisi MIA2026.

## **F1. Form Pelaporan & Geotagging**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Warga |
| **Tujuan** | Membuat Fakta Asal yang lengkap, terstruktur, dan berlokasi akurat dengan usaha pengguna sekecil mungkin. |
| **Aturan bisnis kunci** | Fakta Asal terdiri dari foto, titik GPS, field terstruktur, dan uraian bebas. Setiap perubahan Fakta Asal membuat versi baru dan menghitung ulang `facts_hash`. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-1.1** | Form dapat dipakai tanpa login dan nyaman di layar sempit (mulai 320 px). | P0 |
| **FR-1.2** | Unggah foto minimal 1 dan maksimal 3 ke **Cloudinary**. Format JPG, PNG, HEIC, maks 10 MB per foto. Kompresi dilakukan di sisi klien sebelum unggah. | P0 |
| **FR-1.3** | Geotagging otomatis lewat Geolocation API. Tampilkan akurasi dalam meter dan peta mini (MapLibre GL JS + OpenStreetMap) untuk konfirmasi. Titik terkunci setelah dikonfirmasi. | P0 |
| **FR-1.4** | Validasi titik terhadap poligon batas Kota Bogor. Di luar poligon, laporan ditolak dengan pesan jelas. | P0 |
| **FR-1.5** | Jika izin lokasi ditolak atau akurasi buruk (di atas 50 m), pengguna dapat menandai titik manual di peta dan sistem mencatat bahwa lokasi dipilih manual. | P0 |
| **FR-1.6** | Field terstruktur: kategori (lubang, retak, amblas, bergelombang, genangan atau drainase buruk, lainnya), estimasi ukuran (panjang, lebar, kedalaman atau kelas kecil, sedang, besar), patokan lokasi, waktu ditemukan (default sekarang), dampak (pilihan ganda, misalnya membahayakan pengendara roda dua, menimbulkan macet, pernah terjadi kecelakaan), dan uraian bebas maks 1.000 karakter. | P0 |
| **FR-1.7** | Reverse geocoding menggunakan **Nominatim (OpenStreetMap)** menyarankan nama jalan sebagai **saran terpisah**. Nilai ini bukan bagian Fakta Asal kecuali pelapor memasukkannya sebagai patokan. | P1 |
| **FR-1.8** | Kontak pelapor (email) opsional dengan persetujuan eksplisit untuk notifikasi. Laporan tetap bisa dikirim tanpa kontak. | P1 |
| **FR-1.9** | Sebelum lanjut, tampilkan laporan aktif dalam radius R (default 30 m) dengan kategori mirip dan tawarkan "+1" sebagai ganti membuat laporan baru. Pengguna tetap boleh lanjut membuat laporan baru. | P0 |
| **FR-1.10** | Isian form tersimpan lokal selama 24 jam agar tidak hilang bila halaman tertutup atau sinyal putus. | P1 |
| **FR-1.11** | Perlindungan penyalahgunaan: batas laju per perangkat dan per IP, serta Turnstile atau CAPTCHA pada pengiriman. | P0 |
| **FR-1.12** | Metadata EXIF dibaca untuk keperluan verifikasi Admin. EXIF dihapus dari varian foto yang tampil publik menggunakan transformasi Cloudinary / pemrosesan server. | P0 |

## **F2. AI Draft Assistant & Fallback Template**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Warga, Sistem |
| **Tujuan** | Menghasilkan judul dan deskripsi baku dari Fakta Asal menggunakan model **OpenAI `gpt-6-luna` (*reasoning: medium*)** tanpa menambah fakta, dan selalu punya jalan keluar deterministik. |
| **Aturan bisnis kunci** | AI hanya merangkum teks. AI tidak menganalisis foto dan tidak menentukan instansi, ukuran, kebenaran, kepatuhan, atau status selesai. Pemanggilan AI tidak boleh memicu email atau perubahan status (*zero side-effect*). Setiap draf memiliki label permanen: `✨ [Draf Dibantu AI]` atau `📄 [Draf Template]`. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-2.1** | Input ke model **OpenAI `gpt-6-luna` (konfigurasi `reasoning_effort: "medium"`)** hanya berupa field terstruktur dan uraian bebas dalam bentuk teks. Foto, kontak pelapor, dan data pribadi tidak dikirim. | P0 |
| **FR-2.2** | Output berupa Judul (maks 80 karakter) dan Deskripsi (maks 600 karakter) dalam Bahasa Indonesia baku dan netral. | P0 |
| **FR-2.3** | Instruksi sistem melarang AI menambah fakta, mengubah angka atau ukuran, menyebut atau menyimpulkan instansi, menilai kebenaran, menuduh pihak tertentu, atau menyatakan status. | P0 |
| **FR-2.4** | **Validator pascagenerasi**: periksa bahwa angka dan satuan di output ada di input, tidak ada nama instansi, tidak ada kata terlarang (misalnya tuduhan kelalaian), dan panjang sesuai batas. Jika gagal, jatuh ke Draf Template dan catat alasan `VALIDATION_FAILED`. | P0 |
| **FR-2.5** | Batas waktu panggilan AI (default 8–10 detik menyesuaikan *reasoning medium*) dengan maksimal satu percobaan ulang. Setelah itu otomatis ke Draf Template. | P0 |
| **FR-2.6** | Draf Template deterministik: kalimat baku dengan placeholder yang diisi dari Fakta Asal tanpa model bahasa. Pengguna melihat penjelasan singkat saat fallback terjadi. | P0 |
| **FR-2.7** | Label draf tampil permanen di UI warga, UI Admin, halaman pelacakan, dan email eskalasi. | P0 |
| **FR-2.8** | Endpoint pembuatan draf hanya menulis ke tabel draf dan audit log. Endpoint tidak memiliki akses ke layanan email dan tidak mengubah status laporan. | P0 |
| **FR-2.9** | Batas pembuatan ulang draf (default 5 kali per laporan dan kuota harian per perangkat) untuk mengendalikan kuota dan penyalahgunaan. | P1 |
| **FR-2.10** | Uraian bebas diperlakukan sebagai data tidak tepercaya. Instruksi di dalam uraian (*prompt injection*) tidak boleh diikuti dan dibatasi lewat pemisahan konteks serta validator FR-2.4. | P0 |
| **FR-2.11** | Setiap pembuatan draf mencatat versi model (`gpt-6-luna`), level reasoning (`medium`), versi prompt, latensi, hasil validasi, dan teks input serta output pada snapshot audit. | P0 |
| **FR-2.12** | Draf bersifat baca saja untuk warga. Untuk mengubah isi, warga mengubah Fakta Asal lalu membuat draf baru. | P0 |

## **F3. Panel Sanding & Persetujuan (Human-in-the-Loop)**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Warga |
| **Tujuan** | Memastikan manusia meninjau dan menyetujui teks yang akan dikirim, dan persetujuan itu hanya berlaku untuk fakta yang sama. |
| **Aturan bisnis kunci** | Persetujuan hanya sah bila `approved_facts_hash` sama dengan `facts_hash` terkini. Aturan invalidasi ditegakkan di server, bukan hanya di UI. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-3.1** | Panel sanding menampilkan Fakta Asal dan Draf berdampingan (desktop) atau berurutan dalam tab (mobile), dengan label draf permanen. | P0 |
| **FR-3.2** | Persetujuan berupa kotak centang eksplisit: "Saya sudah meninjau dan menyetujui draf ini." | P0 |
| **FR-3.3** | Tombol Kirim terkunci sampai ada draf yang tidak kedaluwarsa dan persetujuan yang valid. | P0 |
| **FR-3.4** | **Aturan invalidasi**: setiap perubahan Fakta Asal (foto, GPS, field, uraian) setelah draf dibuat mereset persetujuan, mengunci tombol Kirim, menandai draf lama kedaluwarsa, dan menampilkan banner penjelasan. Warga wajib meninjau ulang draf. | P0 |
| **FR-3.5** | Server menolak pengiriman (kode 409) bila `approved_facts_hash` tidak sama dengan `facts_hash` terkini, apa pun yang dikirim klien. | P0 |
| **FR-3.6** | Saat dikirim, sistem menyimpan snapshot final yang tidak bisa diubah: Fakta Asal, draf, label, versi model dan template, dan hash. | P0 |
| **FR-3.7** | Event `SOURCE_FACTS_MODIFIED`, `APPROVAL_RESET`, dan `APPROVAL_GRANTED` dicatat di audit log dengan snapshot diff. | P0 |

## **F4. Peta Interaktif, Upvote (+1), dan Tag Kondisi**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Warga, Admin |
| **Tujuan** | Menampilkan laporan terverifikasi secara terbuka di atas peta **MapLibre GL JS + OpenStreetMap** dan membiarkan komunitas memberi sinyal prioritas tanpa membuat laporan ganda. |
| **Aturan bisnis kunci** | Hanya laporan berstatus Terverifikasi ke atas yang tampil publik. Skor prioritas adalah sinyal bantu bagi Admin dan instansi, bukan keputusan otomatis. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-4.1** | Peta publik menggunakan **MapLibre GL JS + OpenStreetMap** dengan cluster marker, warna per status, dan batas zoom agar tetap ringan di ponsel. | P0 |
| **FR-4.2** | Filter berdasarkan status, kategori, rentang waktu, dan instansi tujuan. | P0 |
| **FR-4.3** | Detail laporan: foto (tanpa EXIF dari Cloudinary), judul, deskripsi, label draf, timeline status, jumlah +1, dan tag. | P0 |
| **FR-4.4** | "+1 Saya Lewat Sini" satu kali per perangkat atau akun per laporan, dapat dibatalkan. Anti-penyalahgunaan lewat batas laju dan sinyal perangkat ringan. | P0 |
| **FR-4.5** | Skor prioritas dihitung dari jumlah +1, kategori, dampak, umur laporan, dan konteks hujan (F9). Formula dan bobot dapat dikonfigurasi dan dijelaskan di UI Admin. | P1 |
| **FR-4.6** | Tag kondisi dari daftar tertutup (misalnya `[Tertutup Genangan Air]`). Satu pengguna boleh memberi tag yang sama sekali per laporan. Tag tampil sebagai hitungan. Pemberi tag perlu verifikasi OTP email ringan via Resend. | P1 |
| **FR-4.7** | Saat Admin menggabungkan duplikat, +1 dan tag dipindahkan ke laporan induk tanpa menggandakan pengguna yang sama. | P0 |

## **F5. Moderasi, Routing, dan Eskalasi Admin**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Admin |
| **Tujuan** | Memberi Admin kendali penuh atas kualitas laporan dan atas setiap pengiriman resmi ke instansi. |
| **Aturan bisnis kunci** | Pengiriman email ke instansi (via Resend) dan perubahan status menjadi Diteruskan terjadi hanya setelah Admin menekan tombol konfirmasi. Admin tidak dapat mengubah Fakta Asal. Koreksi memakai catatan Admin yang terpisah. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-5.1** | Antrean moderasi dengan filter status dan pengurutan berdasarkan umur, skor, dan kandidat duplikat. | P0 |
| **FR-5.2** | Tampilan detail: Fakta Asal, draf dengan label, foto asli, perbandingan lokasi EXIF dengan GPS, peta, dan laporan serupa. | P0 |
| **FR-5.3** | Checklist verifikasi: foto autentik, lokasi sesuai, bukan spam, bukan duplikat, kategori masuk akal. | P0 |
| **FR-5.4** | Aksi: Verifikasi, Tolak (alasan wajib dari daftar dan catatan opsional), Tandai atau Gabung Duplikat, Minta Info Tambahan (P1, jika pelapor memberi kontak). | P0 |
| **FR-5.5** | Routing manual lewat dropdown instansi simulasi (PUPR Kota Bogor, Pemprov Jabar, Balai Nasional). Sistem menampilkan petunjuk wewenang ruas jalan dari data seed Kota Bogor, namun keputusan tetap pada Admin. | P0 |
| **FR-5.6** | Pratinjau email resmi: subjek, isi (judul, deskripsi, label draf, koordinat, tautan peta, foto), dan penerima. Email tidak memuat identitas pelapor kecuali diizinkan. | P0 |
| **FR-5.7** | Tombol "Kirim ke Instansi" dengan dialog konfirmasi. Setelah email diterima **Resend** dengan sukses, status berubah menjadi Diteruskan dan log `ESCALATION_EMAIL_SENT` dicatat. | P0 |
| **FR-5.8** | Jika pengiriman gagal atau memantul (bounce), status tetap Terverifikasi, Admin diberi tahu, dan dapat mengirim ulang. Log `ESCALATION_EMAIL_FAILED` dicatat. | P0 |
| **FR-5.9** | Laporan yang dikembalikan instansi muncul kembali di antrean Admin dengan alasan pengembalian. | P0 |
| **FR-5.10** | Pengingat eskalasi ulang bila tidak ada respons dalam N hari (N dapat dikonfigurasi). Pengiriman ulang tetap manual. | P1 |
| **FR-5.11** | Admin dapat menimpa (override) status dengan alasan wajib bila instansi tidak memakai portal. Override ditandai jelas di timeline dan audit log. | P1 |
| **FR-5.12** | Admin dapat memburamkan area foto yang memuat wajah atau pelat nomor (menggunakan transformasi Cloudinary atau pengolahan server) sebelum laporan tampil publik. Foto asli tetap tersimpan terbatas untuk Admin. | P1 |

## **F6. Portal Instansi (Simulasi): Status dan Bukti Perbaikan**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Petugas Instansi (Simulasi) |
| **Tujuan** | Memberi instansi cara sederhana menerima laporan, memperbarui status, dan mengunggah bukti sebelum dan sesudah untuk mendemonstrasikan siklus penuh di MIA2026. |
| **Aturan bisnis kunci** | Petugas hanya melihat laporan milik instansinya (isolasi tenant). Mendukung login akun petugas simulasi maupun tautan magic link per laporan. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-6.1** | Autentikasi petugas dengan email dan kata sandi (serta dukungan MFA/TOTP atau tautan magic link sekali pakai per laporan). | P0 |
| **FR-6.2** | Daftar laporan terbatas pada instansi petugas (isolasi tenant) dengan filter status dan pencarian. | P0 |
| **FR-6.3** | Perubahan status: Diterima, Diproses (dengan perkiraan tanggal opsional), Selesai. Setiap perubahan wajib punya catatan singkat opsional dan tercatat di log. | P0 |
| **FR-6.4** | Status Selesai mensyaratkan minimal satu foto sesudah perbaikan yang diunggah ke Cloudinary. Foto sebelum diambil otomatis dari laporan. | P0 |
| **FR-6.5** | Cek lokasi EXIF foto sesudah terhadap titik laporan. Jarak lebih dari 100 m memunculkan peringatan dan flag, tidak memblokir. | P1 |
| **FR-6.6** | Aksi "Kembalikan ke Admin" dengan alasan (misalnya bukan kewenangan) agar routing ulang dapat dilakukan. | P0 |
| **FR-6.7** | Notifikasi email via Resend ke petugas saat ada laporan baru diteruskan atau pengingat tertunda. | P1 |

## **F7. Audit Trail & Version History**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Sistem, Admin, Auditor |
| **Tujuan** | Menyediakan catatan yang tidak dapat diubah atas semua aksi penting agar akuntabilitas hukum dan operasional dapat dibuktikan. |
| **Aturan bisnis kunci** | Log bersifat append-only. Tidak ada operasi ubah atau hapus pada tabel log di level database. Aksi pengguna anonim dicatat dengan pengenal pseudonim (bukan identitas asli). |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-7.1** | Tabel `report_audit_logs` append-only, dijaga lewat hak akses database dan trigger penolak UPDATE dan DELETE. | P0 |
| **FR-7.2** | Setiap entri berisi timestamp UTC, actor ID dan tipe aktor, action type, report ID, snapshot diff (sebelum dan sesudah), request ID, serta hash IP dan user agent. | P0 |
| **FR-7.3** | Action type wajib: `AI_GENERATE_SUCCESS`, `AI_FALLBACK_TEMPLATE`, `SOURCE_FACTS_MODIFIED`, `APPROVAL_GRANTED`, `APPROVAL_RESET`, `REPORT_SUBMITTED`, `VERIFY_REPORT`, `REJECT_REPORT`, `ROUTE_ASSIGNED`, `ESCALATION_EMAIL_SENT`, `ESCALATION_EMAIL_FAILED`, `STATUS_CHANGED`, `DUPLICATE_MERGED`, `SIGNOFF_SUBMITTED`, `REOPENED`, `ADMIN_OVERRIDE`, `LOG_EXPORTED`. | P0 |
| **FR-7.4** | Version history menyimpan setiap versi Fakta Asal dan draf, dengan tampilan diff berdampingan di timeline laporan. | P0 |
| **FR-7.5** | Pengaman tambahan antiubah: setiap entri menyimpan hash entri sebelumnya (*hash chain*) sehingga manipulasi dapat terdeteksi. | P1 |
| **FR-7.6** | Akses baca log hanya untuk Admin, Auditor, dan Super Admin. Ekspor CSV atau JSON tersedia dan ekspor itu sendiri dicatat. | P0 |
| **FR-7.7** | Kebijakan retensi dan pseudonimisasi memastikan penghapusan PII kontak tidak merusak integritas log yang tidak dapat diubah. | P0 |

## **F8. Pelacakan Status & Notifikasi**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 (halaman) · P1 (notifikasi) · Fase 1 |
| **Aktor** | Warga |
| **Tujuan** | Menutup siklus pelaporan: warga selalu bisa melihat apa yang terjadi pada laporannya. |
| **Aturan bisnis kunci** | Tautan pelacakan memakai token acak yang sulit ditebak. Pelacakan tidak memerlukan login. Teks notifikasi netral dan tidak menjanjikan waktu perbaikan. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-8.1** | Setiap laporan memiliki kode tiket dan tautan pelacakan berisi token acak. | P0 |
| **FR-8.2** | Halaman pelacakan menampilkan timeline status, alasan bila Ditolak atau Duplikat, instansi tujuan setelah Diteruskan, dan bukti sebelum-sesudah bila ada. | P0 |
| **FR-8.3** | Notifikasi email via **Resend** pada perubahan status utama (Terverifikasi, Diteruskan, Diproses, Selesai, Ditolak) hanya bila pelapor memberi kontak dan persetujuan. | P1 |
| **FR-8.4** | Pelapor dapat berhenti berlangganan notifikasi lewat tautan di email. | P1 |

## **F9. Weather-Road Intelligence**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 (Ditarik ke Fase 1 untuk MVP Utama MIA2026) |
| **Aktor** | Petugas Instansi, Admin |
| **Tujuan** | Membantu instansi melihat hubungan curah hujan dengan frekuensi kerusakan dan titik kambuhan untuk prioritisasi serta rekomendasi konstruksi. |
| **Aturan bisnis kunci** | Rekomendasi teknis berbasis aturan yang transparan, bukan hasil AI generatif. Korelasi tidak dinyatakan sebagai sebab-akibat. Semua rekomendasi berlabel indikatif. Sistem menggunakan daftar API cuaca gratis yang tersedia (lihat Bagian 12.2.1). |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-9.1** | Ingest data cuaca terjadwal (harian) lewat adapter API cuaca gratis (BMKG Open Data / Open-Meteo / NASA POWER), disimpan sebagai snapshot dengan waktu pengambilan dan sumber. | P0 |
| **FR-9.2** | Peta tumpang susun dua lapis di atas MapLibre GL JS: curah hujan (per kecamatan/kelurahan atau grid koordinat Bogor) dan frekuensi atau kepadatan laporan terverifikasi. | P0 |
| **FR-9.3** | Filter periode, kategori kerusakan, dan wilayah (kecamatan atau kelurahan di Kota Bogor). | P0 |
| **FR-9.4** | "Zona merah" didefinisikan lewat aturan yang dapat dikonfigurasi, misalnya minimal N laporan terverifikasi dalam radius R dan periode T di area dengan hujan di atas ambang. | P0 |
| **FR-9.5** | "Titik kambuhan" ditandai bila ada laporan baru dalam radius R dari laporan yang sudah Selesai dalam T hari terakhir (nilai dapat dikonfigurasi). | P0 |
| **FR-9.6** | Rekomendasi indikatif berbasis aturan (misalnya tinjau drainase atau pertimbangkan konstruksi beton *rigid pavement* pada titik kambuhan di zona hujan ekstrem). | P0 |
| **FR-9.7** | Ekspor data dashboard sebagai CSV. | P1 |
| **FR-9.8** | Jika API cuaca sedang tidak dapat diakses, tampilkan data snapshot terakhir dengan penanda "Data usang" dan waktu pembaruan terakhir. Dashboard tetap berfungsi penuh. | P0 |
| **FR-9.9** | Banner peringatan dini cuaca ekstrem di dashboard. | P1 |

## **F10. Community Sign-Off**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 (Ditarik ke Fase 1 untuk MVP Utama MIA2026) |
| **Aktor** | Warga |
| **Tujuan** | Membiarkan komunitas memverifikasi kualitas perbaikan selama masa pemantauan. |
| **Aturan bisnis kunci** | Sign-off hanya dibuka setelah instansi menandai Selesai. Ambang pembukaan kembali bersifat konfigurasi dan memberi sinyal ke Admin, bukan eskalasi otomatis ke instansi. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-10.1** | Masa pemantauan dimulai saat status Selesai. Durasi default 30 hari dan dapat dikonfigurasi. | P0 |
| **FR-10.2** | Rating dua pilihan: "Mulus" atau "Cepat Rusak", dengan foto (diunggah ke Cloudinary) dan komentar opsional. | P0 |
| **FR-10.3** | Yang berhak memberi rating: pelapor asli dan pengguna yang pernah memberi +1 pada laporan tersebut, dengan verifikasi ringan OTP email via Resend. | P0 |
| **FR-10.4** | Satu rating per pengguna per laporan. Rating dapat diubah selama masa pemantauan, setiap perubahan tercatat. | P0 |
| **FR-10.5** | Bila jumlah dan proporsi "Cepat Rusak" melewati ambang (default minimal 3 suara dan lebih dari 50%), status menjadi Dibuka Kembali dan Admin dinotifikasi untuk memutuskan tindak lanjut. | P0 |
| **FR-10.6** | Setelah masa pemantauan berakhir tanpa ambang tercapai, status menjadi Ditutup dan hasil rating diagregasi ke dashboard instansi. | P0 |

# **6\. Model Status Laporan**

Perpindahan status selalu menghasilkan entri audit log `STATUS_CHANGED`.

| Status | Arti | Pemicu | Aktor |
| :---- | :---- | :---- | :---- |
| **Draft** | Laporan sedang disusun dan belum dikirim. Hanya ada di sisi pelapor | Warga mulai mengisi form | Warga |
| **Menunggu Verifikasi** | Laporan terkirim dan masuk antrean moderasi | Warga menekan Kirim dengan persetujuan valid | Warga |
| **Terverifikasi** | Laporan dinilai valid dan tampil di peta publik | Admin menekan Verifikasi | Admin |
| **Ditolak** | Laporan tidak valid, misalnya spam atau tidak sesuai | Admin menolak dengan alasan | Admin |
| **Duplikat** | Laporan digabung ke laporan induk | Admin menggabungkan | Admin |
| **Diteruskan** | Eskalasi resmi sudah terkirim ke instansi | Admin menekan "Kirim ke Instansi" dan email sukses terkirim via Resend | Admin |
| **Diproses** | Instansi menyatakan sedang atau akan menangani | Petugas memperbarui status | Instansi (atau Admin override) |
| **Selesai (Dalam Pemantauan)** | Instansi menandai selesai dengan bukti. Masa pemantauan komunitas berjalan | Petugas menandai Selesai dengan foto sesudah | Instansi (atau Admin override) |
| **Dibuka Kembali** | Komunitas menilai perbaikan cepat rusak | Ambang "Cepat Rusak" tercapai | Sistem lalu Admin |
| **Ditutup** | Masa pemantauan selesai tanpa masalah | Masa pemantauan berakhir | Sistem |

**Catatan alur khusus:** Bila instansi memilih "Kembalikan ke Admin", laporan kembali ke status Terverifikasi dengan penanda Dikembalikan dan alasan, lalu Admin melakukan routing ulang. Bila email eskalasi gagal, status tetap Terverifikasi dan tidak berubah menjadi Diteruskan.

# **7\. Non-Functional Requirements**

Diselaraskan dengan penggunaan layanan **Free Tier** untuk kompetisi **MIA2026**.

| Kategori | Requirement | Target |
| :---- | :---- | :---- |
| **Performa** | Halaman form dan peta dimuat cepat di jaringan seluler, respons API pengiriman laporan di luar proses unggah foto, dan peta tetap ringan dengan cluster marker | LCP ≤ 2,5 detik pada 75% sesi 4G, API p95 ≤ 1 detik (setelah *warm-up* instance Render) |
| **AI** | Batas waktu panggilan OpenAI `gpt-6-luna` (*reasoning medium*) dan jalur fallback deterministik | Timeout 8–10 detik, satu retry, batas generate ulang 5x per laporan (FR-2.9) |
| **Ketersediaan** | Layanan pelaporan harus tetap bisa dipakai saat layanan pendukung (AI, cuaca, peta) bermasalah | Degradasi anggun (*graceful degradation*) untuk AI (ke Draf Template) dan API cuaca (ke snapshot lokal) |
| **Keamanan** | Enkripsi TLS, RBAC dan isolasi data antarinstansi, validasi tipe file gambar, perlindungan dari OWASP Top 10 dan *prompt injection*, manajemen rahasia lewat environment variables | Tidak ada rahasia di kode klien, seluruh mutasi kritis diverifikasi di backend |
| **Privasi & kepatuhan** | Kepatuhan pada prinsip UU No. 27 Tahun 2022 (UU PDP): minimisasi data, persetujuan eksplisit untuk kontak, hapus EXIF pada foto publik via Cloudinary, fitur blur wajah/pelat nomor, dan pseudonimisasi aktor anonim di log | Tidak ada data pribadi pelapor terpapar di halaman publik |
| **Auditabilitas** | Log append-only, sinkronisasi waktu server, dan pemeriksaan integritas *hash chain* | 100% aksi wajib tercatat dalam transaksi atomik |
| **Aksesibilitas** | Antarmuka dapat dipakai pengguna dengan keterbatasan penglihatan dan motorik | Mengacu WCAG 2.1 level AA untuk alur utama |
| **Kompatibilitas** | Peramban seluler dan desktop utama, tahan koneksi buruk (kompresi foto di klien, penyimpanan draf lokal 24 jam) | Dua versi mayor terakhir Chrome dan Safari, serta peramban bawaan Android |
| **Lokalisasi** | Bahasa Indonesia, zona waktu WIB, format tanggal Indonesia | Seluruh antarmuka dan email berbahasa Indonesia |
| **Observabilitas** | Log terstruktur (`pino`), metrik khusus AI (latensi, tingkat fallback, kegagalan validasi), dan metrik bisnis | Tercatat dan dapat dipantau oleh Admin / Super Admin |
| **Tata kelola AI** | Versi prompt dan model (`gpt-6-luna`) tercatat, set uji regresi berisi kasus halusinasi dan injeksi | Uji regresi wajib lulus sebelum perubahan prompt dirilis |

# **8\. Sketsa Data Model**

Skema relasional dan geospasial pada **Neon PostgreSQL (PostGIS)** menggunakan **Drizzle ORM**. Field yang berisi data pribadi ditandai `(PII)`.

## **8.1 Ringkasan Entitas dan Relasi**

| Entitas | Fungsi | Relasi utama |
| :---- | :---- | :---- |
| **reporters** | Identitas pseudonim pelapor atau pengguna komunitas | 1 reporter ke banyak reports, upvotes, signoffs |
| **users** | Akun Admin, Auditor, Super Admin, dan Petugas Instansi (Simulasi) | Petugas terkait ke 1 agency |
| **agencies** | Daftar instansi tujuan simulasi dan kontak resminya | 1 agency ke banyak escalations dan agency_updates |
| **reports** | Entitas inti laporan dan statusnya | Pusat dari hampir semua entitas lain |
| **report_fact_versions** | Versi Fakta Asal (sumber kebenaran) | Banyak versi per report |
| **report_photos** | Foto laporan, sebelum, sesudah, dan sign-off di Cloudinary | Banyak foto per report |
| **report_drafts** | Draf AI (`gpt-6-luna`) atau Template per versi fakta | Terkait ke 1 fact_version |
| **report_approvals** | Persetujuan warga atas draf dan hash fakta | Terkait ke 1 draft |
| **report_audit_logs** | Log append-only semua aksi penting | Banyak log per report |
| **report_upvotes dan report_condition_tags** | Sinyal komunitas | Banyak per report |
| **admin_notes** | Catatan Admin terpisah dari Fakta Asal | Banyak per report |
| **escalations** | Rekam pengiriman email resmi ke instansi via Resend | Banyak per report, 1 agency |
| **agency_updates** | Riwayat pembaruan status dari instansi | Banyak per report |
| **signoffs** | Rating komunitas setelah perbaikan | Banyak per report |
| **weather_observations** | Snapshot data dari API cuaca gratis | Dipakai oleh hotspot_zones |
| **hotspot_zones** | Hasil turunan zona merah dan titik kambuhan | Dihitung dari reports dan weather_observations |
| **system_config** | Parameter yang dapat dikonfigurasi | Dibaca oleh seluruh modul |

## **8.2 Entitas dan Field Kunci**

### **reports**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id** | UUID (PK) | Pengenal internal |
| **ticket_code** | string (unik) | Kode tiket yang mudah dibaca warga |
| **tracking_token** | string (unik) | Token acak untuk tautan pelacakan |
| **reporter_id** | FK reporters | Pelapor pseudonim |
| **status** | enum | Lihat Bagian 6 |
| **location** | geometry(Point) | Titik GPS final yang terkunci |
| **location_source** | enum | `GPS` atau `MANUAL_PIN` |
| **current_facts_version_id** | FK | Versi Fakta Asal terbaru |
| **current_facts_hash** | string | Hash versi Fakta Asal terbaru |
| **approved_draft_id** | FK nullable | Draf yang disetujui dan valid |
| **assigned_agency_id** | FK nullable | Instansi tujuan hasil routing Admin |
| **duplicate_of_id** | FK nullable | Laporan induk bila Duplikat |
| **priority_score** | numeric | Skor prioritas turunan |
| **upvote_count** | int | Cache hitungan +1 |
| **submitted_at / verified_at / forwarded_at / resolved_at / closed_at** | timestamptz | Penanda waktu tiap tahap |
| **returned_by_agency** | boolean | Penanda laporan dikembalikan instansi |
| **idempotency_key** | string (unik) | Mencegah laporan ganda saat kirim ulang |

### **report_fact_versions (Fakta Asal)**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id, report_id, version_no** | UUID, FK, int | Versi berurutan per laporan |
| **category** | enum | Lubang, retak, amblas, bergelombang, genangan, lainnya |
| **size_length_cm / width_cm / depth_cm / size_class** | numeric / enum | Estimasi ukuran dari pelapor |
| **landmark** | text | Patokan lokasi dari pelapor |
| **observed_at** | timestamptz | Waktu ditemukan |
| **impact_tags** | string[] | Dampak yang dipilih |
| **free_text** | text | Uraian bebas, maks 1.000 karakter |
| **lat, lng, gps_accuracy_m** | numeric | Koordinat dan akurasi saat itu |
| **photo_ids** | UUID[] | Foto yang menjadi bagian versi ini |
| **facts_hash** | string | Hash dari seluruh isi versi, dasar invalidasi persetujuan |
| **created_by_actor, created_at** | string, timestamptz | Pembuat versi |

### **report_drafts**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id, report_id, facts_version_id** | UUID, FK, FK | Draf terikat ke versi fakta tertentu |
| **source_type** | enum | `AI` atau `TEMPLATE` (menentukan label permanen) |
| **title, body** | text | Judul dan deskripsi |
| **model_name, model_version, prompt_version** | string | Terisi `openai/gpt-6-luna` (reasoning medium) bila `source_type = AI` |
| **fallback_reason** | enum nullable | `TIMEOUT`, `API_ERROR`, `VALIDATION_FAILED`, `DISABLED` |
| **validator_result** | jsonb | Hasil pemeriksaan FR-2.4 |
| **latency_ms** | int | Waktu pembuatan |
| **facts_hash** | string | Hash fakta saat draf dibuat |
| **is_stale** | boolean | Menjadi true bila fakta berubah |
| **created_at** | timestamptz | Waktu pembuatan |

### **report_approvals**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id, report_id, draft_id** | UUID, FK, FK | Persetujuan atas satu draf |
| **approved_facts_hash** | string | Hash fakta saat disetujui. Harus sama dengan `facts_hash` terkini saat kirim |
| **approved_at** | timestamptz | Waktu persetujuan |
| **reset_at, reset_reason** | timestamptz, enum | Terisi bila persetujuan dibatalkan (misalnya `SOURCE_FACTS_MODIFIED`) |

### **report_audit_logs (append-only)**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id** | bigserial (PK) | Urutan monoton |
| **occurred_at** | timestamptz (UTC) | Waktu kejadian |
| **report_id** | FK nullable | Laporan terkait |
| **actor_id, actor_type** | string, enum | `WARGA_PSEUDONIM`, `ADMIN`, `INSTANSI`, `SISTEM`, `AI` |
| **action_type** | enum | Daftar pada FR-7.3 |
| **snapshot_before, snapshot_after** | jsonb | Diff perubahan |
| **request_id, ip_hash, user_agent** | string | Penelusuran teknis. IP disimpan dalam bentuk hash |
| **prev_entry_hash, entry_hash** | string | Hash chain untuk deteksi manipulasi |

### **escalations**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id, report_id, agency_id** | UUID, FK, FK | Satu pengiriman eskalasi |
| **sent_by_user_id, sent_at** | FK, timestamptz | Admin yang menekan tombol konfirmasi |
| **recipient_emails** | string[] | Alamat email tujuan saat pengiriman |
| **email_subject, email_body_snapshot** | text | Isi email persis seperti terkirim, termasuk label draf |
| **provider_message_id** | string | ID referensi dari Resend |
| **delivery_status** | enum | `QUEUED`, `SENT`, `DELIVERED`, `BOUNCED`, `FAILED` |
| **failure_reason** | text nullable | Alasan gagal atau bounce |

### **agency_updates**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id, report_id, agency_id** | UUID, FK, FK | Satu pembaruan status |
| **new_status** | enum | `DITERIMA`, `DIPROSES`, `SELESAI`, `DIKEMBALIKAN` |
| **planned_date** | date nullable | Perkiraan pengerjaan |
| **note** | text | Catatan petugas atau alasan pengembalian |
| **updated_by, updated_at** | FK, timestamptz | Petugas atau Admin override |
| **is_admin_override** | boolean | Menandai penimpaan oleh Admin |

### **report_photos**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id, report_id** | UUID, FK | Foto milik laporan |
| **kind** | enum | `REPORT`, `BEFORE`, `AFTER`, `SIGNOFF` |
| **cloudinary_public_id, sha256** | string | Public ID objek di Cloudinary dan hash isi |
| **public_url** | string | URL varian publik tanpa EXIF (dan sudah di-blur jika ditandai Admin) |
| **exif_taken_at, exif_lat, exif_lng** | nullable | Hanya untuk verifikasi Admin, tidak publik |
| **uploaded_by_actor, uploaded_at** | string, timestamptz | Asal unggahan |

### **report_upvotes, report_condition_tags, signoffs**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **report_upvotes: report_id, voter_key, created_at** | FK, string | Unik pada `(report_id, voter_key)` |
| **report_condition_tags: report_id, tag_code, voter_key** | FK, enum, string | Unik pada `(report_id, tag_code, voter_key)` |
| **signoffs: report_id, voter_key, rating, comment, photo_id, monitoring_window_id** | FK, enum | rating: `MULUS` atau `CEPAT_RUSAK`. Unik pada `(report_id, voter_key)` |

### **weather_observations dan hotspot_zones**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **weather_observations: source, area_code, area_level** | string, enum | `source`: `BMKG`, `OPEN_METEO`, `NASA_POWER`, dll. |
| **weather_observations: period_start, period_end, rainfall_mm, is_forecast, fetched_at, raw_payload_ref** | timestamptz, numeric | Snapshot yang bisa ditelusuri |
| **hotspot_zones: geometry, period, report_count, rainfall_mm, recurrence_count, rule_version, computed_at** | geometry, numeric | Hasil turunan. `rule_version` menjaga hasil dapat direproduksi |

### **agencies, users, reporters, admin_notes, system_config**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **agencies: name, level, official_emails, jurisdiction_note, is_active** | string, enum | level: `KOTA`, `PROVINSI`, `NASIONAL` |
| **users: role, email, password_hash, agency_id, mfa_enabled, status** | enum | role: `ADMIN`, `AUDITOR`, `SUPER_ADMIN`, `PETUGAS_INSTANSI` |
| **reporters: pseudonym_id, device_hash, contact_email (PII, terenkripsi, opsional), consent_at, email_verified_at** | string | Kontak hanya ada bila pelapor setuju |
| **admin_notes: report_id, author_id, note, created_at** | text | Tempat koreksi Admin tanpa mengubah Fakta Asal |
| **system_config: key, value, updated_by, updated_at** | jsonb | Radius duplikat, masa pemantauan, ambang Cepat Rusak, batas generate AI, timeout AI, dan sebagainya |

# **9\. Edge Case & Failure Matrix**

Severity menunjukkan dampak bila perilaku yang diharapkan tidak diterapkan: **Tinggi** (merusak integritas, keamanan, atau kepercayaan), **Sedang** (mengganggu pengalaman atau operasional), **Rendah** (kosmetik atau jarang).

| ID | Area | Skenario | Perilaku yang diharapkan | Severity |
| :---- | :---- | :---- | :---- | :---- |
| **E-01** | Pelaporan | Izin lokasi ditolak atau GPS tidak akurat (di atas 50 m) | Tawarkan penandaan titik manual di peta. Catat `location_source = MANUAL_PIN` dan tampilkan petunjuk ke Admin. | Sedang |
| **E-02** | Pelaporan | Koordinat di luar batas Kota Bogor | Tolak dengan pesan jelas. Sarankan melapor ke kanal wilayah terkait bila tersedia. | Sedang |
| **E-03** | Pelaporan | Foto terlalu besar, format tidak didukung, atau file bukan gambar sebenarnya (ekstensi dipalsukan) | Validasi tipe asli di server/Cloudinary, tolak dengan pesan spesifik. Kompres di klien bila memungkinkan. | Sedang |
| **E-04** | Pelaporan | Koneksi terputus saat mengirim atau tombol Kirim ditekan dua kali | Idempotency key mencegah laporan ganda. Isian tersimpan lokal agar bisa dikirim ulang. | Sedang |
| **E-05** | Pelaporan | EXIF foto menunjukkan lokasi atau waktu jauh berbeda dari GPS dan waktu laporan | Jangan memblokir. Tandai pada tampilan Admin sebagai sinyal verifikasi. | Sedang |
| **E-06** | Pelaporan | Foto memuat wajah, pelat nomor, atau data pribadi | EXIF dihapus untuk publik via Cloudinary. Admin dapat memburamkan area sensitif, menolak, atau meminta foto ulang. | Tinggi |
| **E-07** | Pelaporan | Pelapor mengirim banyak laporan dalam waktu singkat (spam atau bot) | Batas laju, Turnstile atau CAPTCHA, dan penandaan otomatis di antrean Admin. | Sedang |
| **E-08** | Pelaporan | Uraian bebas memuat tuduhan, ujaran kebencian, atau data pribadi pihak lain | Moderasi Admin sebelum tayang publik dan kebijakan konten. Draf AI tidak boleh mengulang tuduhan. | Tinggi |
| **E-09** | AI | API OpenAI `gpt-6-luna` timeout atau mengembalikan error | Retry sekali lalu fallback ke Draf Template dengan penjelasan. Log `AI_FALLBACK_TEMPLATE`. | Sedang |
| **E-10** | AI | AI menambahkan angka, ukuran, atau instansi yang tidak ada di input | Validator mendeteksi dan jatuh ke Draf Template dengan alasan `VALIDATION_FAILED`. Kasus dimasukkan ke set uji regresi. | Tinggi |
| **E-11** | AI | Uraian bebas berisi instruksi seperti "abaikan aturan sebelumnya" | Diperlakukan sebagai data. Validator menahan output yang melanggar. Kasus masuk set uji injeksi. | Tinggi |
| **E-12** | AI | Output kosong, terpotong, atau bukan Bahasa Indonesia | Dianggap gagal validasi dan fallback ke Draf Template. | Sedang |
| **E-13** | AI | Kuota AI habis, atau fitur AI dimatikan lewat feature flag | Semua draf memakai Draf Template dengan label yang benar. Pelaporan tetap berjalan. | Sedang |
| **E-14** | Sanding | Warga mengubah Fakta Asal setelah menyetujui draf | Persetujuan reset, tombol Kirim terkunci, draf kedaluwarsa, banner penjelasan. Log `SOURCE_FACTS_MODIFIED` dan `APPROVAL_RESET`. | Tinggi |
| **E-15** | Sanding | Klien memanipulasi request untuk mengirim dengan persetujuan tidak valid | Server membandingkan `approved_facts_hash` dengan `facts_hash` terkini dan menolak (409). | Tinggi |
| **E-16** | Sanding | Warga mengubah fakta di dua tab atau perangkat sekaligus | Versi fakta terbaru menang. Persetujuan yang tidak cocok hash otomatis tidak sah dan tab lain diberi tahu saat sinkron. | Sedang |
| **E-17** | Sanding | Warga menutup halaman setelah menyetujui tetapi sebelum kirim | Persetujuan lokal tidak dianggap sah setelah sesi baru. Warga meninjau ulang sebelum kirim. | Rendah |
| **E-18** | Komunitas | Satu orang menekan +1 berkali-kali atau memakai banyak perangkat | Unik per `voter_key`, batas laju, dan sinyal perangkat ringan. Pola mencurigakan ditandai di antrean Admin. | Sedang |
| **E-19** | Komunitas | Dua laporan dari titik sama muncul hampir bersamaan | Kandidat duplikat ditandai. Admin menggabungkan dan +1 serta tag dipindahkan tanpa menghitung ganda. | Sedang |
| **E-20** | Komunitas | Brigading: banyak akun memberi "Cepat Rusak" untuk menjatuhkan instansi | Ambang memakai jumlah minimum dan proporsi. Dibuka Kembali hanya memberi sinyal ke Admin, bukan eskalasi otomatis. | Sedang |
| **E-21** | Komunitas | Pengguna memberi rating tanpa pernah berada di lokasi | Hanya pengguna eligible (pelapor dan pemberi +1) yang boleh memberi rating dengan verifikasi OTP email. | Sedang |
| **E-22** | Eskalasi | Email eskalasi Resend gagal, ditolak, atau memantul | Status tidak berubah menjadi Diteruskan. Admin diberi tahu, bisa mengirim ulang. Log `ESCALATION_EMAIL_FAILED`. | Tinggi |
| **E-23** | Eskalasi | Admin salah memilih instansi | Instansi memakai "Kembalikan ke Admin". Admin routing ulang, riwayat tersimpan. Metrik salah routing dipantau. | Sedang |
| **E-24** | Eskalasi | Admin menekan Kirim dua kali atau dua Admin memproses laporan yang sama | Kunci pada laporan (optimistic locking) dan pencegahan pengiriman ganda. Antrean menampilkan siapa yang sedang membuka. | Sedang |
| **E-25** | Eskalasi | Ruas jalan berada di batas kewenangan atau tidak jelas pemiliknya | Admin dapat memilih lebih dari satu instansi dengan catatan atau menandai "perlu klarifikasi". Keputusan tercatat. | Sedang |
| **E-26** | Admin | Antrean moderasi menumpuk | Pengurutan berdasarkan umur dan skor prioritas, serta peringatan SLA. | Sedang |
| **E-27** | Instansi | Instansi tidak merespons dalam waktu lama | Pengingat ke Admin untuk eskalasi ulang manual (FR-5.10). Status publik tetap jujur (Diteruskan, belum ada tanggapan). | Sedang |
| **E-28** | Instansi | Instansi memilih tidak login dengan password | Jalur magic link atau override Admin dengan penanda jelas di timeline (FR-5.11). | Sedang |
| **E-29** | Instansi | Foto sesudah perbaikan diambil di lokasi berbeda atau tidak relevan | Peringatan EXIF dan flag untuk Admin. Komunitas dapat menilai lewat sign-off. | Sedang |
| **E-30** | Instansi | Perbaikan sementara (tambalan) lalu rusak lagi di titik sama | Dibuka Kembali (sign-off) atau terdeteksi sebagai titik kambuhan pada dashboard (FR-9.5). | Sedang |
| **E-31** | Cuaca | API cuaca utama tidak tersedia, berubah format, atau kuota habis | Beralih ke API cuaca gratis cadangan atau tampilkan data snapshot terakhir dengan penanda "Data usang". Dashboard laporan tetap jalan. | Sedang |
| **E-32** | Cuaca | Data cuaca terlalu kasar untuk membedakan kelurahan | Tampilkan pada granularitas yang tersedia (kecamatan / grid koordinat) dengan catatan keterbatasan. Jangan menyiratkan presisi palsu. | Sedang |
| **E-33** | Cuaca | Titik kerusakan banyak di area tanpa hujan ekstrem | Dashboard tidak memaksa korelasi. Rekomendasi hanya muncul bila aturan terpenuhi dan tetap berlabel indikatif. | Rendah |
| **E-34** | Audit | Penulisan audit log gagal saat aksi penting berlangsung | Aksi dan log ditulis dalam satu transaksi. Bila log gagal, aksi dibatalkan agar tidak ada aksi tanpa jejak. | Tinggi |
| **E-35** | Audit | Permintaan penghapusan data pribadi dari subjek data | Hapus atau anonimkan PII (kontak, foto bila relevan) dan pertahankan log dalam bentuk pseudonim. | Tinggi |
| **E-36** | Sistem | Penyimpanan foto (Cloudinary) atau basis data (Neon) tidak tersedia sementara | Tampilkan pesan jelas dan simpan isian lokal. Jangan menampilkan keberhasilan palsu. | Tinggi |
| **E-37** | Sistem | Jam perangkat pengguna salah | Timestamp otoritatif selalu dari server. Waktu dari klien hanya disimpan sebagai informasi tambahan. | Rendah |

# **10\. Keputusan Produk yang Dikunci**

Seluruh pertanyaan terbuka pada draf sebelumnya telah dikunci menjadi **Keputusan Tetap (Diputuskan)** untuk eksekusi proyek kompetisi **MIA2026**:

| No | Topik | Keputusan Final | Alasan | Status |
| :---- | :---- | :---- | :---- | :---- |
| **OQ-01** | Pemilik dokumen dan organisasi di balik SIJAGO | Proyek independen untuk kompetisi **MIA2026** tanpa organisasi eksternal. Pemilik dokumen: **Sabira** (PM & UI/UX Designer) dan **Boby** (System Architect & Fullstack Developer). | Kejelasan peran dan tanggung jawab tim kompetisi. | Diputuskan |
| **OQ-02** | Kemitraan dengan Dinas PUPR Kota Bogor | Kemitraan instansi **disimulasikan secara penuh** di dalam aplikasi (mulai dari eskalasi email Resend hingga tindak lanjut di Portal Instansi). | Memastikan seluruh alur produk dapat didemonstrasikan secara utuh saat penjurian. | Diputuskan |
| **OQ-03** | Cara akses petugas instansi ke portal | Sediakan login akun simulasi petugas, dukungan magic link per laporan, serta fitur override Admin dengan penanda transparan. | Fleksibilitas demonstrasi dan ketahanan operasional. | Diputuskan |
| **OQ-04** | Cakupan geografis laporan | Poligon batas administratif Kota Bogor, mencakup semua kelas jalan di dalamnya (kota, provinsi, nasional). | Menentukan validasi lokasi PostGIS dan daftar instansi tujuan. | Diputuskan |
| **OQ-05** | Pembagian fase untuk Weather-Road Intelligence (F9) dan Community Sign-Off (F10) | **F9 dan F10 ditarik masuk ke Fase 1 (MVP Utama)** bersama F1 sampai F8. | Merupakan nilai jual dan pembeda utama (*unique value proposition*) SIJAGO pada kompetisi MIA2026. | Diputuskan |
| **OQ-06** | Anggaran dan paket infrastruktur | Menggunakan **100% layanan Free Tier** (Vercel, Render, Neon, Cloudinary, Resend, MapLibre + OSM, API Cuaca Gratis). | Efisiensi biaya untuk proyek kompetisi tanpa mengorbankan kualitas arsitektur. | Diputuskan |
| **OQ-07** | Tujuan akhir pengembangan produk | Digunakan untuk kompetisi **MIA2026** dengan standar arsitektur siap produksi (*production-grade*). | Menunjukkan kematangan teknis dan produk kepada dewan juri. | Diputuskan |
| **OQ-08** | Aksi tanpa login vs dengan verifikasi | Lapor dan +1 tanpa login (berbasis perangkat). Tag kondisi dan Community Sign-Off butuh verifikasi ringan via OTP email (Resend). | Keseimbangan kemudahan pengguna dan pencegahan spam/brigading. | Diputuskan |
| **OQ-09** | Kanal kontak pelapor | Email saja (dikirim via Resend). | Gratis pada tier awal dan mudah diintegrasikan. | Diputuskan |
| **OQ-10** | Durasi masa pemantauan sign-off dan ambang "Cepat Rusak" | Durasi default 30 hari. Ambang default minimal 3 suara dan lebih dari 50% menyatakan Cepat Rusak (dapat dikonfigurasi di `system_config`). | Menentukan perilaku otomatis F10. | Diputuskan |
| **OQ-11** | Penanganan wajah dan pelat nomor pada foto | Blur manual oleh Admin pada MVP (didukung fitur transformasi Cloudinary / pengolahan gambar). | Melindungi privasi pihak ketiga. | Diputuskan |
| **OQ-12** | Penyedia dan model AI yang digunakan | Menggunakan model **OpenAI `gpt-6-luna` dengan level reasoning `medium`**. Hanya teks fakta non-PII yang dikirim ke API AI. | Menghasilkan rangkuman Bahasa Indonesia baku yang akurat dengan tetap menjaga privasi. | Diputuskan |
| **OQ-13** | Apakah warga boleh mengedit draf secara manual? | Tidak boleh. Warga mengubah Fakta Asal lalu membuat draf baru. | Menjaga aturan invalidasi hash dan akuntabilitas teks. | Diputuskan |
| **OQ-14** | Batas pembuatan ulang draf AI | Maksimal 5 pembuatan ulang per laporan dan kuota harian per perangkat. | Mengendalikan penggunaan token AI dan mencegah penyalahgunaan. | Diputuskan |
| **OQ-15** | Apakah identitas pelapor diteruskan ke instansi? | Tidak diteruskan (hanya kode tiket), kecuali pelapor memberikan persetujuan eksplisit. | Melindungi privasi pelapor. | Diputuskan |
| **OQ-16** | SLA verifikasi Admin | Target SLA verifikasi 2 hari kerja. | Standar operasional moderasi. | Diputuskan |
| **OQ-17** | Daftar instansi dan data kewenangan ruas jalan | Tabel instansi simulasi (PUPR Kota Bogor, Pemprov Jabar, Balai Nasional) dilengkapi data *seed* ruas jalan utama Kota Bogor sebagai petunjuk bagi Admin. | Memudahkan Admin melakukan routing saat demonstrasi. | Diputuskan |
| **OQ-18** | Format eskalasi ke instansi | Email berformat baku via Resend lengkap dengan label draf, koordinat, tautan peta, dan foto. | Bukti penyampaian resmi yang terstruktur. | Diputuskan |
| **OQ-19** | Formula skor prioritas | Formula transparan berbasis bobot (+1, dampak, kategori, umur, dan curah hujan) yang dapat dikonfigurasi di `system_config`. | Membantu pengurutan kerja Admin dan instansi. | Diputuskan |
| **OQ-20** | Visibilitas laporan berstatus Menunggu Verifikasi | Tidak tampil publik. Hanya laporan berstatus **Terverifikasi** ke atas yang tampil di peta publik. | Mencegah spam dan laporan palsu tampil di peta publik. | Diputuskan |
| **OQ-21** | Sumber API data cuaca untuk F9 | Menyediakan daftar lengkap API cuaca gratis (BMKG Open Data, Open-Meteo, NASA POWER, OpenWeatherMap Free, CHIRPS/VisualCrossing Free) dengan arsitektur *adapter* agar dapat dipilih/dikombinasikan saat implementasi. | Memastikan ketersediaan data prakiraan, peringatan dini, maupun curah hujan historis harian secara gratis. | Diputuskan |
| **OQ-22** | Label rekomendasi teknis pada dashboard F9 | Seluruh rekomendasi teknis berbasis aturan (*rule-based*) diberi label "Indikatif — Bukan Pengganti Survei Teknis Lapangan". | Mencegah salah tafsir korelasi cuaca sebagai kepastian struktur jalan. | Diputuskan |
| **OQ-23** | Definisi "zona merah" dan "titik kambuhan" | Parameter jumlah laporan, radius (meter), periode (hari), dan ambang curah hujan (mm) disimpan di `system_config` agar mudah dikalibrasi. | Fleksibilitas pengujian dan demonstrasi analitik. | Diputuskan |
| **OQ-24** | Kebijakan retensi dan hak hapus data vs audit log | Aktor disimpan secara pseudonim di audit log; PII kontak disimpan terpisah di tabel `reporters` sehingga dapat dihapus/dianonimkan tanpa mengubah audit log. | Kepatuhan desain privasi dan integritas *append-only log*. | Diputuskan |
| **OQ-25** | Kebijakan konten laporan | Moderasi sebelum tayang dan daftar alasan penolakan baku untuk laporan yang memuat tuduhan, SARA, atau PII pihak ketiga. | Menjaga keamanan hukum dan etika platform. | Diputuskan |
| **OQ-26** | Data awal untuk pengujian dan demo | Menyiapkan *seed data* realistis di wilayah Kota Bogor (mencakup berbagai kecamatan, status laporan, dan riwayat curah hujan) untuk keperluan demo MIA2026. | Memastikan peta publik dan dashboard F9 langsung kaya informasi saat dipresentasikan. | Diputuskan |
| **OQ-27** | Nama produk resmi | **SIJAGO (Sistem Jalan Bogor)**. | Identitas produk yang kuat dan relevan dengan Kota Bogor. | Diputuskan |
| **OQ-28** | Penyedia peta dasar dan geocoding | **MapLibre GL JS + OpenStreetMap** (dengan **Nominatim** untuk *reverse geocoding*). | 100% gratis, *open-source*, dan mendukung clustering GeoJSON dengan performa tinggi. | Diputuskan |
| **OQ-29** | Arsitektur repositori dan hosting | **Multirepo** (repositori Frontend dan Backend terpisah). Frontend di-deploy di **Vercel (Free Tier)**, Backend di **Render (Free Tier)**, dan Database di **Neon (Free Tier)**. | Sesuai preferensi tim pengembang dan gratis untuk kompetisi. | Diputuskan |
| **OQ-30** | Penyimpanan foto dan layanan email transaksional | Foto disimpan di **Cloudinary (Free Tier)**; email transaksional dan eskalasi dikirim menggunakan **Resend (Free Tier)**. | Cloudinary memudahkan kompresi, penghapusan EXIF, dan blur; Resend menyediakan DX modern dan webhook status pengiriman. | Diputuskan |

# **11\. Risiko, Asumsi, dan Rencana Rilis**

## **11.1 Risiko Utama**

| ID | Risiko | Peluang | Dampak | Mitigasi |
| :---- | :---- | :---- | :---- | :---- |
| **R1** | Instance gratis Render mengalami *cold start* (tertidur saat tidak ada trafik) ketika diuji juri | Tinggi | Sedang | Lakukan *warm-up* / ping otomatis menjelang sesi penjurian dan tampilkan indikator loading yang jelas di UI |
| **R2** | Draf AI (`gpt-6-luna`) memuat fakta yang tidak ada di input | Sedang | Tinggi | Validator deterministik pascagenerasi, fallback template, sanding wajib, dan audit sampling |
| **R3** | Spam, foto palsu, atau laporan fitnah | Sedang | Tinggi | Moderasi sebelum tayang publik, rate limit, Turnstile/CAPTCHA, pemeriksaan EXIF, kebijakan konten |
| **R4** | Batas rate-limit Nominatim OSM (1 req/detik) atau API cuaca gratis terlampaui | Sedang | Sedang | Terapkan *debouncing* di klien, wajib sertakan `User-Agent` resmi aplikasi, dan simpan *cache*/snapshot di database |
| **R5** | Batas pengiriman email Resend Free Tier pada mode sandbox (hanya ke email terverifikasi jika tanpa custom domain) | Tinggi | Sedang | Gunakan alamat email penerima yang sudah didaftarkan/diverifikasi untuk simulasi instansi dan pelapor, atau hubungkan domain bila tersedia |
| **R6** | Korelasi hujan dan kerusakan dibaca sebagai sebab-akibat pasti | Sedang | Sedang | Label "Indikatif" permanen dan penjelasan transparan aturan pemicu di UI dashboard |
| **R7** | Kebocoran data pribadi (foto, lokasi, kontak) | Rendah | Tinggi | Minimisasi data, strip EXIF publik via Cloudinary, dan pembatasan akses RBAC |

## **11.2 Asumsi dan Dependensi**

* Peran Warga, Admin SIJAGO, dan Petugas Instansi disimulasikan oleh tim selama demonstrasi MIA2026.
* API OpenAI (`gpt-6-luna`), Cloudinary, Resend, Neon, dan salah satu API cuaca gratis dari daftar (Bagian 12.2.1) dapat diakses dari environment backend.

## **11.3 Rencana Eksekusi Satu Fase (MVP Utama MIA2026)**

Seluruh pengembangan difokuskan ke dalam **Fase 1 (MVP Utama)** yang dibagi menjadi tiga *milestone* teknis:

| Milestone | Fokus Pengerjaan | Kriteria Selesai (Exit Criteria) |
| :---- | :---- | :---- |
| **Milestone 1: Fondasi, Pelaporan Warga & AI Drafting (F1, F2, F3, F7)** | Setup multirepo (Frontend Next.js & Backend Express), skema Drizzle + PostGIS di Neon, unggah foto Cloudinary, integrasi OpenAI `gpt-6-luna` (*reasoning medium*) + validator + fallback template, panel sanding hash, dan audit log append-only | Warga dapat mengirim laporan dengan titik GPS Kota Bogor, menyetujui draf AI/Template, validasi hash 409 berfungsi, dan audit log tercatat 100% |
| **Milestone 2: Peta Publik, Moderasi Admin, Portal Instansi & Pelacakan (F4, F5, F6, F8)** | Peta interaktif MapLibre + OSM, fitur +1 & tag kondisi (OTP email), antrean moderasi Admin, eskalasi email via Resend, Portal Instansi simulasi (update status & bukti foto), serta halaman lacak tiket | Siklus laporan dari Menunggu Verifikasi -> Terverifikasi -> Diteruskan -> Diproses -> Selesai berjalan ujung-ke-ujung |
| **Milestone 3: Weather-Road Intelligence, Community Sign-Off & Pemolesan Demo (F9, F10)** | Ingest data dari API cuaca gratis, layer peta tumpang susun hujan vs kerusakan, deteksi zona merah & titik kambuhan, fitur *Community Sign-Off* ("Mulus" / "Cepat Rusak"), serta *seeding* data demo Kota Bogor | Seluruh fitur F1–F10 berjalan mulus di Vercel dan Render Free Tier, siap didemonstrasikan untuk MIA2026 |

# **12\. Arsitektur dan Tech Stack**

Bagian ini menerjemahkan kebutuhan produk menjadi spesifikasi arsitektur yang dikelola oleh **Sabira** (Product Manager & UI/UX Designer) dan **Boby** (System Architect & Fullstack Developer) untuk kompetisi **MIA2026**.

## **12.1 Keputusan Arsitektur**

| Keputusan | Pilihan | Alasan |
| :---- | :---- | :---- |
| **Manajemen repositori** | **Multirepo** (Repositori Frontend dan Backend terpisah) | Memisahkan siklus build/deploy antara Next.js di Vercel dan Express.js di Render secara bersih |
| **Pemisahan frontend dan backend** | Next.js hanya untuk antarmuka. Seluruh logika bisnis ada di Express API | Aturan inti (invalidasi persetujuan, audit log, routing, eskalasi) ditegakkan di satu tempat di server |
| **Bentuk API** | REST JSON dengan prefix `/v1`, validasi Zod, kontrak didokumentasikan dengan OpenAPI | Sederhana, mudah diuji, dan menjaga konsistensi kontrak antar-repositori (multirepo) |
| **Akses data** | Drizzle ORM dengan driver `pg` ke connection string pooled milik Neon | Type-safe, migrasi eksplisit lewat `drizzle-kit`, dan SQL tetap bisa dipakai langsung untuk query PostGIS |
| **Data geospasial** | Ekstensi PostGIS di Neon, kolom `geometry` untuk titik laporan dan poligon batas kota | Dibutuhkan untuk validasi batas wilayah Kota Bogor, deteksi duplikat radius, dan analitik zona merah |
| **Model AI** | **OpenAI `gpt-6-luna`** dengan konfigurasi `reasoning_effort: "medium"` | Menghasilkan rangkuman teks laporan yang terstruktur, netral, dan patuh pada instruksi sistem |
| **Penyimpanan foto** | **Cloudinary (Free Tier)** | Mendukung unggah gambar, kompresi otomatis, penghapusan EXIF untuk varian publik, dan transformasi blur |
| **Email transaksional** | **Resend (Free Tier)** | API modern untuk mengirim email eskalasi instansi, OTP verifikasi ringan, dan notifikasi status |
| **Peta & Geocoding** | **MapLibre GL JS + OpenStreetMap** (dan **Nominatim** untuk *reverse geocoding*) | Gratis, *open-source*, tanpa biaya per *map load*, dan mendukung clustering GeoJSON |
| **Pekerjaan latar belakang** | Antrean berbasis Postgres (`pg-boss`) atau job terjadwal di backend | Tidak perlu menambah Redis berbayar untuk pengiriman email retry dan ingest data cuaca |

## **12.2 Tech Stack**

| Area | Lapisan | Pilihan Teknologi | Status | Catatan |
| :---- | :---- | :---- | :---- | :---- |
| **Repositori** | **Struktur Kode** | **Multirepo** (`sijago-frontend` & `sijago-backend`) | Ditetapkan | Dua repositori Git terpisah untuk Frontend dan Backend |
| Frontend | **Framework** | Next.js (App Router) + TypeScript | Ditetapkan | Di-deploy di Vercel (Free Tier) |
| Frontend | **UI dan styling** | Tailwind CSS + shadcn/ui (Radix UI) | Ditetapkan | Komponen aksesibel untuk target WCAG 2.1 AA dan cepat dibangun |
| Frontend | **Form dan validasi** | React Hook Form + Zod | Ditetapkan | Validasi input sisi klien yang sinkron dengan skema backend |
| Frontend | **Data fetching** | TanStack Query | Ditetapkan | Cache, retry, dan optimistic update untuk +1 |
| Frontend | **Peta & Geocoding** | **MapLibre GL JS** (`react-map-gl`) + **OpenStreetMap** & **Nominatim** | Ditetapkan | Cluster marker bawaan untuk sumber GeoJSON dan reverse geocoding gratis |
| Frontend | **Grafik dashboard (F9)** | Recharts | Ditetapkan | Visualisasi grafik curah hujan versus frekuensi laporan |
| Frontend | **Foto di sisi klien** | `browser-image-compression`, `exifr` | Ditetapkan | Kompresi sebelum unggah ke Cloudinary dan pembacaan EXIF untuk verifikasi (FR-1.2, FR-1.12) |
| Frontend | **Anti-bot** | Cloudflare Turnstile (Free) | Ditetapkan | Memenuhi FR-1.11, token diverifikasi di server |
| Frontend | **Isian lokal** | Zustand + `idb-keyval` | Ditetapkan | Simpan isian 24 jam (FR-1.10) dan antrean kirim ulang saat sinyal putus |
| Frontend | **Tanggal** | `date-fns` (locale `id`), zona WIB | Ditetapkan | Format tanggal Indonesia |
| Backend | **Runtime dan framework** | Node.js LTS + TypeScript + Express.js | Ditetapkan | Di-deploy sebagai Free Web Service di Render |
| Backend | **Validasi dan dokumen API** | Zod + `zod-to-openapi` | Ditetapkan | Dokumentasi OpenAPI/Swagger otomatis agar mudah dikonsumsi repo frontend |
| Backend | **ORM dan migrasi** | Drizzle ORM + `drizzle-kit`, driver `pg` | Ditetapkan | Terhubung ke Neon PostgreSQL + PostGIS |
| Backend | **Autentikasi dan RBAC** | Better Auth / JWT / Cookie Session + Middleware RBAC | Ditetapkan | Berlaku untuk Admin, Auditor, Super Admin, dan Petugas Instansi (Simulasi) |
| Backend | **Keamanan HTTP** | `helmet`, `cors`, `express-rate-limit`, `hpp` | Ditetapkan | Konfigurasi CORS mengizinkan origin domain Vercel frontend |
| Backend | **Logging** | `pino` + `pino-http` dengan request ID | Ditetapkan | Request ID ikut ke audit log (FR-7.2) |
| Backend | **Penyimpanan objek (Foto)** | **Cloudinary** (`cloudinary` SDK) | Ditetapkan | Menyimpan foto asli dan menghasilkan URL publik tanpa EXIF serta transformasi blur (FR-1.12, FR-5.12) |
| Backend | **Email transaksional** | **Resend** (`resend` SDK + React Email / HTML Template) | Ditetapkan | Mengirim email eskalasi instansi, OTP, dan notifikasi status beserta webhook status pengiriman |
| Backend | **Modul AI** | **OpenAI SDK** — Model **`gpt-6-luna`** (`reasoning_effort: "medium"`) | Ditetapkan | Dilengkapi `AbortController` untuk timeout, validasi output terstruktur Zod, validator FR-2.4, dan fallback template |
| Backend | **Geospasial** | PostGIS, `turf.js` | Ditetapkan | `ST_Contains` untuk poligon batas Kota Bogor dan `ST_DWithin` untuk deteksi duplikat serta titik kambuhan |
| Backend | **Hash dan audit** | `node:crypto` (SHA-256), trigger Postgres | Ditetapkan | `facts_hash`, *hash chain* log, dan penolakan UPDATE atau DELETE pada `report_audit_logs` |
| Database | **Database** | **Neon PostgreSQL** (Free Tier, ekstensi `postgis` dan `pgcrypto`) | Ditetapkan | Menggunakan connection string pooled |
| Kualitas | **Pengujian** | Vitest, Supertest, Playwright | Ditetapkan | Unit test validator AI, integrasi API, dan E2E alur utama |

## **12.2.1 Daftar Referensi API Cuaca Gratis untuk Weather-Road Intelligence (F9)**

Berikut adalah daftar lengkap API cuaca dan curah hujan yang **dapat diakses secara gratis** untuk diintegrasikan (dipilih salah satu atau dikombinasikan melalui *Weather Provider Adapter*) pada saat implementasi fitur **F9 (Weather-Road Intelligence)**:

| No | Nama API / Layanan | Jenis Data yang Tersedia | Keunggulan untuk SIJAGO (Kota Bogor) | Batasan Gratis & Persyaratan |
| :---- | :---- | :---- | :---- | :---- |
| **1** | **BMKG Open Data API**<br>(`api.bmkg.go.id` / `data.bmkg.go.id`) | Prakiraan cuaca 3 harian per kelurahan/kecamatan (`adm4`), suhu, kelembapan, kode cuaca, dan peringatan dini cuaca ekstrem (*nowcasting*). | Sumber resmi pemerintah Indonesia, memiliki kode wilayah administratif (`adm3`/`adm4`) spesifik untuk seluruh kecamatan dan kelurahan di Kota Bogor. | **100% Gratis** (tanpa API key). Wajib mencantumkan atribusi BMKG. *Catatan: Hanya menyediakan prakiraan & peringatan dini, bukan riwayat curah hujan harian (mm) jangka panjang.* |
| **2** | **Open-Meteo API**<br>(`open-meteo.com`) | **Curah hujan historis harian/jam-jaman (`precipitation_sum`, `rain_sum` dalam mm)** hingga puluhan tahun ke belakang, serta prakiraan 7–16 hari berdasarkan titik koordinat (`lat`, `lng`). | Sangat mudah mengambil curah hujan (mm) spesifik per koordinat kecamatan di Kota Bogor (Bogor Tengah, Bogor Utara, Tanah Sareal, dll.) untuk menghitung zona merah dan korelasi lubang jalan. | **100% Gratis tanpa API Key** untuk penggunaan non-komersial/akademik. Kuota hingga **10.000 request/hari**. |
| **3** | **NASA POWER API**<br>(`power.larc.nasa.gov`) | Data klimatologi dan **curah hujan historis harian (`PRECTOTCORR` dalam mm/hari)** berbasis satelit & model global per titik koordinat. | Sangat andal untuk analisis tren curah hujan bulanan/tahunan di Bogor tanpa batasan kunci API. | **100% Gratis tanpa API Key**. Memiliki jeda pembaruan data (*near real-time lag* sekitar 2–3 hari). |
| **4** | **OpenWeatherMap (Free Tier)**<br>(`openweathermap.org/api`) | Cuaca saat ini (*current weather*) dan prakiraan 5 hari / 3 jam (termasuk volume hujan `rain.3h` dalam mm) per koordinat. | Populer, dokumentasi sangat lengkap, respons JSON cepat untuk status hujan *real-time*. | **Gratis dengan API Key** (hingga 1.000 panggilan/hari atau 60 panggilan/menit). Data historis memerlukan paket berbayar. |
| **5** | **WeatherAPI.com (Free Tier)**<br>(`weatherapi.com`) | Cuaca *real-time*, prakiraan 3 hari, peringatan cuaca, dan riwayat cuaca 7 hari terakhir (`precip_mm`). | Menyediakan `precip_mm` harian untuk 7 hari terakhir secara gratis dalam satu endpoint. | **Gratis dengan API Key** (hingga 1.000.000 panggilan/bulan pada tier gratis). |
| **6** | **Visual Crossing Weather API (Free Tier)**<br>(`visualcrossing.com`) | Riwayat cuaca harian lengkap (curah hujan dalam mm, durasi hujan) dan prakiraan 15 hari berdasarkan lokasi/koordinat. | Memudahkan penarikan tabel riwayat hujan harian Kota Bogor dalam satu panggilan query. | **Gratis dengan API Key** hingga **1.000 *records*/hari**. |

> **Strategi Rekomendasi Implementasi nanti:** Gunakan **BMKG Open Data API** untuk menampilkan status cuaca resmi per kelurahan dan *banner* peringatan dini (FR-9.9), lalu padukan dengan **Open-Meteo API** (tanpa API key) untuk menarik angka **akumulasi curah hujan historis harian (`mm`)** di titik koordinat Kota Bogor guna menghidupkan kalkulasi Zona Merah dan Titik Kambuhan (FR-9.1 s/d FR-9.5).

## **12.3 Alur Utama Sistem**

### **Mengirim laporan (F1 sampai F3)**

* Browser mengompres foto, membaca metadata EXIF awal, dan mengunggah foto ke **Cloudinary** (melalui endpoint backend atau *signed upload*).
* Klien menyimpan Fakta Asal ke API Express. API membuat versi fakta baru dan menghitung `facts_hash` di server.
* Klien meminta draf. Modul AI memanggil **OpenAI `gpt-6-luna` (`reasoning_effort: "medium"`)** dengan timeout, memvalidasi hasil dengan validator deterministik, dan jatuh ke Draf Template bila gagal.
* Warga menyetujui draf. API menyimpan `approved_facts_hash`.
* Klien memanggil kirim dengan header `Idempotency-Key`. API membandingkan hash dan menolak dengan 409 bila berbeda. Bila cocok, snapshot final dan entri audit log ditulis dalam satu transaksi database.

### **Eskalasi ke instansi (F5)**

* Admin memilih instansi tujuan simulasi, membuka pratinjau, lalu menekan konfirmasi.
* API membuat catatan `escalations` berstatus `QUEUED` dan mengirim email melalui **Resend**.
* Hanya bila Resend menerima email, status laporan menjadi Diteruskan dan `ESCALATION_EMAIL_SENT` dicatat. Bila gagal, status tetap Terverifikasi dan Admin dapat mengirim ulang.
* Webhook Resend memperbarui `delivery_status` (misalnya `DELIVERED` atau `BOUNCED`).

### **Peta publik dan dashboard cuaca (F4 dan F9)**

* Peta publik (**MapLibre GL JS + OpenStreetMap**) memanggil endpoint publik berparameter bounding box dan filter, hasilnya GeoJSON dari PostGIS dengan cache HTTP singkat.
* Job terjadwal / pemicu ingest mengambil data dari API cuaca gratis (BMKG / Open-Meteo), memvalidasi dengan Zod, menyimpan snapshot ke `weather_observations`, lalu menghitung ulang `hotspot_zones`.

## **12.4 Struktur Multirepo**

Proyek dibagi menjadi dua repositori mandiri:

1. **Repositori Frontend (`sijago-frontend` / Web — di-deploy ke Vercel Free Tier):**
   * Aplikasi Next.js (App Router) yang memuat antarmuka Warga (Form Lapor, Peta Publik, Lacak Tiket, Community Sign-Off), Dashboard Admin (Moderasi, Routing, Audit Log), dan Portal Instansi (Update Status & Dashboard Weather-Road Intelligence).
2. **Repositori Backend (`sijago-backend` / API — di-deploy ke Render Free Tier):**
   * Aplikasi Express.js + TypeScript yang memuat skema Drizzle ORM + migrasi PostGIS, modul OpenAI `gpt-6-luna`, integrasi Cloudinary & Resend, adapter API cuaca gratis, engine kalkulasi zona merah, serta manajemen audit log *append-only*.
   * Menyediakan spesifikasi OpenAPI (`/v1/openapi.json`) agar tipe data dan kontrak request/response tetap sinkron dengan repositori Frontend.

Konvensi API: awalan **`/v1`**, galat memakai format `problem+json`, paginasi berbasis cursor, dan header **`Idempotency-Key`** pada endpoint pengiriman.

## **12.5 Lingkungan dan Deployment (Free Tier)**

| Lingkungan | Frontend (Repo 1) | Backend (Repo 2) | Database |
| :---- | :---- | :---- | :---- |
| **Lokal** | `localhost:3000` (Next.js dev) | `localhost:4000` (Express dev) | Neon Development Branch / Docker Postgres + PostGIS |
| **Production / Demo MIA2026** | **Vercel (Free Tier)** | **Render (Free Web Service)** | **Neon PostgreSQL (Free Tier)** dengan pooled connection |

Rahasia (API key OpenAI, Cloudinary, Resend, Turnstile, JWT/Session secret) disimpan sebagai environment variable di Vercel dan Render, serta divalidasi dengan Zod saat aplikasi mulai.

## **12.6 Pemetaan Aturan Kritis ke Implementasi**

| Aturan produk | Cara menegakkan di stack |
| :---- | :---- |
| **Persetujuan batal bila Fakta Asal berubah (FR-3.4, FR-3.5)** | `facts_hash` dihitung di server. Endpoint kirim membandingkan `approved_facts_hash` dengan `facts_hash` terkini dan menolak dengan 409 |
| **AI tanpa efek samping (FR-2.8)** | Modul AI (`gpt-6-luna`) tidak boleh mengimpor modul Resend atau modul pengubah status. Dijaga dengan aturan ESLint `no-restricted-imports` dan unit test |
| **Fallback AI (FR-2.4 sampai FR-2.6)** | Timeout lewat `AbortController`, satu retry, validator deterministik, lalu Draf Template tanpa model bahasa |
| **Audit log tidak bisa diubah (FR-7.1)** | Role database aplikasi tidak punya hak UPDATE atau DELETE pada tabel log, ditambah trigger penolak dan *hash chain* |
| **Aksi dan log satu transaksi (E-34)** | Setiap perubahan status dan penulisan audit log berada dalam satu transaksi Drizzle |
| **Tidak ada pengiriman ganda (US-06, E-24)** | `Idempotency-Key` unik di database dan penguncian optimistik pada laporan |
| **Isolasi data antarinstansi (BR-18)** | Middleware RBAC dan filter `agency_id` wajib pada seluruh query portal instansi |
| **EXIF tidak tampil publik (FR-1.12)** | Cloudinary menghasilkan varian publik tanpa EXIF. Metadata EXIF asli hanya dapat diakses Admin |
| **Email eskalasi hanya setelah konfirmasi (BRL-06)** | Endpoint eskalasi Resend hanya dapat dipanggil dengan peran Admin dan tindakan eksplisit dari UI |

## **12.7 Checklist Kesiapan Teknis Sebelum Demo MIA2026**

* **Antisipasi Cold Start Render Free Tier:** Sediakan endpoint `/v1/health` yang ringan dan lakukan ping berkala (misalnya via UptimeRobot gratis) selama masa penjurian agar backend Express tidak tertidur.
* **Kepatuhan Nominatim & OSM:** Sertakan header `User-Agent: SIJAGO-MIA2026/1.0` pada setiap panggilan *reverse geocoding* Nominatim dan terapkan *debounce* di frontend.
* **Konfigurasi Email Resend Free Tier:** Pastikan alamat email tujuan simulasi instansi telah diverifikasi di dashboard Resend (atau gunakan domain terverifikasi) agar pengiriman email eskalasi selalu berstatus `DELIVERED` saat demo.
* **Seed Data Kota Bogor:** Jalankan script *seeder* yang mengisi poligon batas Kota Bogor, daftar ruas jalan utama, sampel laporan di berbagai kecamatan, serta snapshot curah hujan dari Open-Meteo/BMKG agar peta dan dashboard F9 langsung siap didemonstrasikan.
