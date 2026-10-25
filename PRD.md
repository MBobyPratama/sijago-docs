**PRODUCT REQUIREMENTS DOCUMENT**

**SIJAGO**

**Sistem Jalan Bogor**

Platform Pelaporan Kerusakan Jalan Kota Bogor  \|  Civic-Tech & Weather-Road Intelligence

| Atribut | Keterangan |
| :---- | :---- |
| **Versi dokumen** | 0.3 (Draft, nama SIJAGO dan tech stack ditambahkan) |
| **Tanggal** | 25 Oktober 2026 |
| **Status** | Draft. Rekomendasi default PM sudah diterapkan sebagai keputusan (Bagian 10). Butir berstatus Asumsi kerja menunggu konfirmasi fakta sebelum dikunci menjadi v1.0 |
| **Pemilik dokumen** | Product Manager (disusun berdasarkan Product Summary & Scope) |
| **Stakeholder utama** | Tim Produk SIJAGO, Tim Engineering, Admin Moderator, perwakilan Dinas PUPR Kota Bogor (calon mitra) |
| **Model operasi** | Hibrida: Warga melapor, Admin SIJAGO memverifikasi dan meneruskan, Instansi mengeksekusi |

| Cara membaca dokumen ini Semua isi yang berasal dari Product Summary dipertahankan sebagai keputusan produk. Bagian yang saya tambahkan sebagai PM karena dibutuhkan agar produk bisa berjalan (misalnya pelacakan status, aturan duplikat, dan validator output AI) ditandai \[Usulan PM\]. Angka target dan ambang batas adalah usulan awal yang perlu divalidasi dengan data dan stakeholder, bukan fakta. |
| :---- |

# **Riwayat Dokumen & Daftar Isi**

| Versi | Tanggal | Perubahan |
| :---- | :---- | :---- |
| 0.1 | 25 Okt 2026 | Draft awal PRD dari Product Summary & Scope. |
| 0.2 | 25 Okt 2026 | Rekomendasi default PM diterapkan sebagai keputusan dan asumsi kerja (Bagian 10). FR diselaraskan: OTP untuk tag, blur manual foto, email tanpa identitas pelapor, ambang sign-off. |
| 0.3 | 25 Okt 2026 | Nama produk SIJAGO ditetapkan (OQ-27). Bagian 12 Arsitektur dan Tech Stack ditambahkan. OQ-29 dan OQ-30 ditambahkan. |

1\. Overview & Problem Statement

2\. Goals & Success Metrics

3\. Pengguna, Peran, dan Cakupan MVP

4\. User Stories & Acceptance Criteria

5\. Functional Requirements per Fitur MVP

6\. Model Status Laporan

7\. Non-Functional Requirements

8\. Sketsa Data Model

9\. Edge Case & Failure Matrix

10\. Keputusan Produk dan Asumsi Kerja (eks Open Questions)

11\. Risiko, Asumsi, dan Rencana Rilis

12\. Arsitektur dan Tech Stack

# **1\. Overview & Problem Statement**

## **1.1 Ringkasan Produk**

**SIJAGO (Sistem Jalan Bogor)** adalah platform web **mobile-first** untuk melaporkan kerusakan jalan di Kota Bogor. Produk ini menggabungkan tiga hal: partisipasi warga (lapor, dukung, dan verifikasi hasil perbaikan), analitik cuaca ekstrem dari BMKG untuk membantu instansi memahami hubungan hujan dengan kerusakan jalan, serta **AI drafting dengan Human-in-the-Loop** yang akuntabel dan bisa diaudit.

Alur kerjanya hibrida. Warga melapor, Admin SIJAGO memverifikasi lalu meneruskan secara manual ke instansi yang berwenang, dan Dinas PUPR atau instansi terkait mengeksekusi perbaikan serta melaporkan bukti pengerjaan.

## **1.2 Latar Belakang dan Problem Statement**

Kota Bogor dikenal sebagai Kota Hujan. Intensitas hujan yang tinggi diduga mempercepat kerusakan permukaan jalan, terutama di titik dengan drainase buruk. Dari sisi warga dan instansi, masalah yang kami hipotesiskan adalah sebagai berikut.

| Pihak | Masalah yang dihipotesiskan | Dampak |
| :---- | :---- | :---- |
| **Warga / komuter** | Laporan sulit disampaikan dengan lengkap (lokasi, ukuran, dampak sering tidak jelas) Tidak tahu laporan sampai ke instansi yang benar Tidak ada kejelasan status dan bukti tindak lanjut | Warga enggan melapor ulang, kepercayaan pada kanal aduan turun |
| **Instansi (PUPR dan lainnya)** | Laporan tersebar dan formatnya tidak seragam Laporan salah alamat karena kewenangan ruas jalan berbeda (kota, provinsi, nasional) Sulit menentukan prioritas karena tidak ada konteks frekuensi dan cuaca | Perbaikan reaktif, lubang kambuhan di titik yang sama |
| **Pengelola platform (Admin)** | Risiko spam, foto palsu, dan laporan duplikat Risiko teks laporan yang memuat tuduhan atau klaim yang tidak diverifikasi | Reputasi platform dan hubungan dengan instansi terganggu |

| Catatan validasi Seluruh masalah di atas adalah hipotesis awal dari Product Summary. Sebelum PRD dikunci, perlu validasi lewat wawancara warga dan petugas PUPR serta analisis data kanal eksisting (misalnya SP4N-LAPOR\!, kanal aduan Pemkot, dan media sosial). Lihat Open Question OQ-26. |
| :---- |

### **Problem statement**

Warga Kota Bogor yang menemukan jalan rusak kesulitan menyampaikan laporan yang lengkap, ke instansi yang tepat, dan memantau tindak lanjutnya. Di sisi lain, instansi kesulitan menentukan prioritas karena laporan tidak terstruktur dan tidak dilengkapi konteks cuaca. Akibatnya perbaikan bersifat reaktif dan kerusakan berulang di titik yang sama.

### **How Might We**

* Bagaimana membuat warga bisa melapor dalam hitungan menit dengan data yang cukup lengkap untuk ditindaklanjuti?

* Bagaimana memakai AI untuk merapikan teks tanpa risiko mengarang fakta dan tanpa mengambil keputusan atas nama manusia?

* Bagaimana membuat setiap laporan punya jejak yang bisa dipertanggungjawabkan, dari dibuat sampai diperbaiki?

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

* Integrasi dua arah dengan sistem internal PUPR atau SP4N-LAPOR\! (dapat dipertimbangkan setelah MVP).

* Aplikasi native iOS/Android (MVP berupa web mobile-first, PWA sebagai opsi Fase 2).

* Notifikasi WhatsApp, sistem poin atau gamifikasi, dan pembayaran apa pun.

* Pelaporan di luar batas administratif Kota Bogor.

# **2\. Goals & Success Metrics**

## **2.1 Tujuan Produk**

| ID | Tujuan | Indikator keberhasilan utama |
| :---- | :---- | :---- |
| **G1** | Memudahkan warga membuat laporan yang lengkap dan terlacak | Completion rate form, waktu pengisian, kelengkapan data |
| **G2** | Meningkatkan kualitas dan ketepatan alamat laporan ke instansi | Persentase laporan terverifikasi, tingkat salah routing |
| **G3** | Membuktikan bahwa AI dapat membantu tanpa halusinasi dan tanpa efek samping | Insiden halusinasi, tingkat fallback, persetujuan draf |
| **G4** | Mendorong tindak lanjut instansi yang terlihat dan terverifikasi komunitas | Waktu respons, tingkat selesai, hasil sign-off |
| **G5** | Memberi instansi wawasan hujan vs kerusakan untuk prioritisasi | Penggunaan dashboard, titik kambuhan yang teridentifikasi |
| **G6** | Menjamin akuntabilitas dan kepatuhan lewat jejak audit | Cakupan audit log, hasil audit sampling |

## **2.2 Success Metrics**

*Target di bawah adalah **usulan** untuk 3 bulan setelah pilot dimulai. Baseline belum tersedia dan harus ditetapkan pada bulan pertama. Setelah baseline ada, target direvisi.*

| Metrik | Definisi | Target usulan | Tujuan |
| :---- | :---- | :---- | :---- |
| **North Star: Laporan Tuntas Terverifikasi** | Jumlah laporan yang mencapai status Selesai dan tidak dibuka kembali selama masa pemantauan | Dipantau tren naik tiap bulan | G4 |
| **Waktu pengisian laporan** | Median dari buka form sampai kirim | ≤ 3 menit | G1 |
| **Completion rate form** | Sesi yang memulai form dan berhasil mengirim | ≥ 60% | G1 |
| **Kelengkapan laporan** | Laporan dengan foto, GPS valid, kategori, dan ukuran terisi | ≥ 90% | G1 |
| **Waktu verifikasi Admin** | Median dari Menunggu Verifikasi sampai keputusan Admin | ≤ 2 hari kerja | G2 |
| **Tingkat salah routing** | Laporan yang dikembalikan instansi karena bukan kewenangannya | ≤ 10% | G2 |
| **Tingkat spam/tolak** | Laporan ditolak karena spam atau tidak valid dari total masuk | Dipantau (tanpa target di awal) | G2 |
| **Insiden halusinasi AI** | Draf yang memuat fakta tidak ada di Fakta Asal (audit sampling) | 0 kasus lolos ke laporan terkirim | G3 |
| **Tingkat fallback** | Draf yang jatuh ke Template dari total permintaan draf | ≤ 5% (di luar gangguan besar) | G3 |
| **Tingkat persetujuan draf** | Draf yang disetujui warga tanpa membuat ulang | ≥ 70% | G3 |
| **Waktu respons instansi** | Median dari Diteruskan sampai Diproses | Ditetapkan bersama PUPR (OQ-02) | G4 |
| **Partisipasi komunitas** | Rata-rata \+1 per laporan terverifikasi, dan jumlah sign-off per laporan selesai | Dipantau | G4 |
| **Penggunaan dashboard** | Petugas aktif mingguan di dashboard analitik | ≥ 3 petugas aktif/minggu | G5 |
| **Cakupan audit log** | Aksi wajib yang punya entri log | 100% | G6 |

### **Guardrail metrics (tidak boleh memburuk)**

* Laporan salah kirim ke instansi tanpa konfirmasi Admin harus 0\.

* Latensi halaman form (LCP) tidak melewati 2,5 detik di jaringan 4G untuk 75% sesi.

* Pelanggaran privasi (data pribadi terpapar publik) harus 0\.

# **3\. Pengguna, Peran, dan Cakupan MVP**

## **3.1 Persona**

| Persona | Perilaku utama | Kebutuhan utama |
| :---- | :---- | :---- |
| **Warga / Komuter / Pelapor** | Menemukan kerusakan jalan di lapangan, sering saat bergerak, memakai ponsel dengan koneksi bervariasi | Form cepat tanpa login di awal, GPS otomatis, bantuan merapikan teks, bisa memantau status, dan bisa ikut mendukung atau memverifikasi hasil perbaikan |
| **Admin Pengelola SIJAGO** | Moderator independen yang menjaga kualitas laporan | Antrean moderasi, alat verifikasi, routing manual, dan kontrol atas pengiriman email resmi |
| **Mitra Instansi (Dinas PUPR Kota Bogor)** | Petugas yang menindaklanjuti perbaikan | Laporan terkurasi, dashboard cuaca dan titik kambuhan, serta cara mudah memperbarui status dan mengunggah bukti |

## **3.2 Peran dan Hak Akses**

Peran Super Admin dan Auditor adalah **\[Usulan PM\]** karena dibutuhkan untuk konfigurasi dan kepatuhan. Tanda ✓ berarti diizinkan, ◐ berarti diizinkan dengan batasan yang dijelaskan di FR.

| Aksi | Warga (tanpa akun) | Admin | Petugas Instansi | Auditor | Super Admin |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Membuat laporan dan memakai AI draf** | ✓ | \- | \- | \- | \- |
| **Melihat peta dan laporan terverifikasi** | ✓ | ✓ | ◐ (miliknya) | ✓ | ✓ |
| **\+1 (dukungan)** | ✓ | \- | \- | \- | \- |
| **Tag kondisi** | ◐ (OTP email) | \- | \- | \- | \- |
| **Sign-off hasil perbaikan** | ◐ (eligible \+ OTP email) | \- | \- | \- | \- |
| **Verifikasi, tolak, gabung duplikat** | \- | ✓ | \- | \- | ✓ |
| **Memilih instansi dan mengirim eskalasi** | \- | ✓ | \- | \- | ✓ |
| **Mengubah Fakta Asal pelapor** | ◐ (sebelum kirim) | \- | \- | \- | \- |
| **Update status dan unggah bukti perbaikan** | \- | ◐ (override tercatat) | ✓ (miliknya) | \- | \- |
| **Dashboard cuaca dan analitik** | \- | ✓ | ✓ | ✓ | ✓ |
| **Membaca audit log dan version history** | \- | ✓ | \- | ✓ | ✓ |
| **Kelola instansi, pengguna, dan konfigurasi** | \- | \- | \- | \- | ✓ |

## **3.3 Daftar Fitur dan Prioritas**

Prioritas: **P0** wajib untuk rilis pilot, **P1** penting dan menyusul segera, **P2** nilai tambah. Fase 1 adalah MVP Core untuk pilot terbatas. Fase 2 (MVP+) menambah analitik cuaca dan sign-off. Pembagian fase ini adalah **\[Usulan PM\]** dan menjadi keputusan di OQ-05.

| ID | Fitur | Sisi | Prioritas | Fase |
| :---- | :---- | :---- | :---- | :---- |
| **F1** | Form Pelaporan & Geotagging | Warga | P0 | 1 |
| **F2** | AI Draft Assistant & Fallback Template | Warga | P0 | 1 |
| **F3** | Panel Sanding & Persetujuan (Human-in-the-Loop) | Warga | P0 | 1 |
| **F4** | Peta Interaktif, Upvote (+1), dan Tag Kondisi | Komunitas | P0 | 1 |
| **F5** | Moderasi, Routing, dan Eskalasi Admin | Admin | P0 | 1 |
| **F6** | Portal Instansi: Status dan Bukti Perbaikan | Instansi | P0 | 1 |
| **F7** | Audit Trail & Version History | Sistem | P0 | 1 |
| **F8** | Pelacakan Status & Notifikasi **\[Usulan PM\]** | Warga | P0 (halaman), P1 (notifikasi) | 1 |
| **F9** | Weather-Road Intelligence (BMKG) | Instansi | P1 | 2 |
| **F10** | Community Sign-Off | Komunitas | P1 | 2 |

# **4\. User Stories & Acceptance Criteria**

Acceptance criteria memakai pola Given-When-Then yang dipadatkan. Setiap story dipetakan ke fitur (F) dan prioritas (P). Nilai ambang seperti radius dan durasi adalah usulan yang dapat dikonfigurasi (lihat Bagian 8, tabel system\_config).

| ID | Fitur · Prioritas | User story | Acceptance criteria |
| :---- | :---- | :---- | :---- |
| **US-01** | F1 · P0 | Sebagai **warga**, saya ingin melapor jalan rusak tanpa harus membuat akun, agar prosesnya cepat saat saya masih di lokasi. | Given saya membuka halaman utama, when saya menekan "Lapor", then form tampil tanpa login. Given form terbuka, when saya mengizinkan lokasi, then titik GPS terkunci otomatis dan akurasinya (meter) ditampilkan. Given koordinat berada di luar batas Kota Bogor, when saya lanjut, then sistem menolak dengan pesan yang jelas. Given izin lokasi ditolak, when saya lanjut, then saya dapat menandai titik secara manual di peta. Given foto atau kategori belum terisi, when saya lanjut, then field yang kurang ditandai dan tidak ada data yang hilang. |
| **US-02** | F2 · P0 | Sebagai **warga**, saya ingin AI merapikan isian saya menjadi judul dan deskripsi baku, agar laporan mudah dipahami petugas. | Given fakta sudah lengkap, when saya menekan "Buat Draf", then judul dan deskripsi tampil dengan label ✨ \[Draf Dibantu AI\] dalam batas waktu yang ditetapkan. Given draf tampil, then seluruh angka, ukuran, dan patokan di draf identik dengan isian saya, dan tidak ada nama instansi atau tuduhan baru. Given draf sedang dibuat, then tidak ada email terkirim dan status laporan tidak berubah. |
| **US-03** | F2 · P0 | Sebagai **warga**, saya tetap ingin mendapat draf yang rapi walau layanan AI sedang bermasalah. | Given API AI timeout, error, atau hasilnya gagal validasi, when draf diminta, then sistem menampilkan penjelasan singkat dan otomatis menampilkan 📄 \[Draf Template\]. Given fallback terjadi, then log AI\_FALLBACK\_TEMPLATE tercatat beserta alasannya. Given Draf Template tampil, then saya bisa lanjut ke tinjauan tanpa mengulang input. |
| **US-04** | F3 · P0 | Sebagai **warga**, saya ingin meninjau draf berdampingan dengan data asli saya sebelum mengirim, agar saya yakin isinya benar. | Given draf tersedia, when saya membuka tinjauan, then Fakta Asal dan Draf tampil berdampingan dengan label draf yang permanen. Given saya belum mencentang persetujuan, then tombol Kirim terkunci. Given saya mencentang persetujuan, then tombol Kirim aktif dan persetujuan tercatat bersama hash Fakta Asal. |
| **US-05** | F3 · P0 | Sebagai **warga**, saya ingin sistem memaksa saya meninjau ulang bila data saya berubah setelah draf dibuat, agar draf selalu sesuai fakta. | Given saya sudah menyetujui draf, when saya mengubah foto, GPS, atau field fakta apa pun, then persetujuan batal otomatis, tombol Kirim terkunci, dan banner menjelaskan alasannya. Given persetujuan batal, then draf lama ditandai kedaluwarsa dan tidak dapat dikirim. Given perubahan terjadi, then log SOURCE\_FACTS\_MODIFIED dan APPROVAL\_RESET tersimpan dengan snapshot diff. Given ada upaya mengirim lewat panggilan API langsung saat persetujuan tidak valid, then server menolak (bukan hanya UI). |
| **US-06** | F1/F3 · P0 | Sebagai **warga**, saya ingin menerima kode tiket setelah mengirim, agar saya bisa memantau laporan. | Given persetujuan valid, when saya menekan Kirim, then laporan berstatus Menunggu Verifikasi dan kode tiket serta tautan pelacakan ditampilkan. Given koneksi terputus saat mengirim, when saya mencoba lagi, then tidak terbentuk laporan ganda (idempotency key). |
| **US-07** | F4 · P0 | Sebagai **warga**, saya ingin melihat titik laporan di peta, agar saya tahu masalah yang sudah dilaporkan. | Given saya membuka peta, then hanya laporan terverifikasi yang tampil, berwarna sesuai status dan dikelompokkan (cluster) saat zoom jauh. Given saya menekan sebuah titik, then detail menampilkan foto, judul, deskripsi dengan label draf, status, dan jumlah \+1. Given saya memakai filter status, kategori, atau waktu, then peta diperbarui. |
| **US-08** | F4 · P0 | Sebagai **warga**, saya ingin menekan "+1 Saya Lewat Sini" pada laporan yang sudah ada, agar prioritasnya naik tanpa membuat laporan baru. | Given laporan aktif, when saya menekan \+1, then hitungan bertambah 1 dan tombol berubah menjadi status sudah mendukung. Given saya sudah memberi \+1, then saya tidak dapat memberi lagi (unik per perangkat atau akun) tetapi boleh membatalkan. Given batas laju terlampaui, then aksi ditolak dan dicatat. |
| **US-09** | F4 · P1 | Sebagai **warga**, saya ingin menambahkan tag kondisi seperti \[Tertutup Genangan Air\], agar petugas tahu kondisi nyata di lapangan. | Given laporan aktif, when saya memilih tag dari daftar yang tersedia, then tag tersimpan dan tampil sebagai hitungan. Given saya belum terverifikasi, when saya menambah tag, then saya diminta verifikasi OTP email ringan terlebih dahulu. Given saya sudah memilih tag yang sama pada laporan itu, then pilihan kedua ditolak. |
| **US-10** | F8 · P0 | Sebagai **warga**, saya ingin melihat perkembangan laporan saya, agar saya tidak perlu bertanya ke siapa pun. | Given saya membuka tautan pelacakan, then timeline status dengan waktu, instansi tujuan (setelah Diteruskan), dan bukti perbaikan (jika ada) tampil. Given saya mengisi email dan menyetujui notifikasi, when status utama berubah, then saya menerima email netral berisi status terbaru (P1). |
| **US-11** | F5 · P0 | Sebagai **Admin**, saya ingin memeriksa keaslian laporan sebelum diteruskan, agar instansi hanya menerima laporan yang layak. | Given antrean moderasi, when saya membuka sebuah laporan, then saya melihat Fakta Asal, draf beserta labelnya, foto, metadata EXIF, peta, dan laporan serupa di sekitarnya. Given saya menekan Verifikasi, then status menjadi Terverifikasi, log VERIFY\_REPORT tercatat, dan laporan tampil di peta publik. Given saya menolak laporan, then saya wajib memilih alasan, status menjadi Ditolak, dan pelapor dapat melihat alasannya. Admin tidak dapat mengubah Fakta Asal. Koreksi dilakukan lewat catatan Admin yang terpisah. |
| **US-12** | F5 · P0 | Sebagai **Admin**, saya ingin menggabungkan laporan duplikat, agar instansi tidak menerima laporan berulang untuk titik yang sama. | Given laporan baru berada dalam radius R meter dari laporan aktif berkategori sama, then sistem menandainya sebagai kandidat duplikat. Given saya menggabungkan, then \+1 dan tag dipindahkan ke laporan induk, laporan duplikat tertaut ke induk, dan log DUPLICATE\_MERGED tercatat. |
| **US-13** | F5 · P0 | Sebagai **Admin**, saya ingin menentukan instansi tujuan dan mengirim eskalasi secara manual, agar saya mengendalikan apa yang dikirim atas nama platform. | Given laporan Terverifikasi, when saya memilih instansi dari dropdown, then pilihan tersimpan tanpa mengirim email. Given saya membuka pratinjau email, then subjek, isi, label draf, dan lampiran atau tautan foto terlihat. Given saya menekan "Kirim ke Instansi" lalu mengonfirmasi dialog, then email terkirim, status menjadi Diteruskan, dan log ESCALATION\_EMAIL\_SENT tercatat. Given email gagal terkirim, then status tidak berubah menjadi Diteruskan, saya diberi tahu, dan saya dapat mencoba lagi. Sistem atau AI tidak pernah memilih instansi atau mengirim email sendiri. |
| **US-14** | F6 · P0 | Sebagai **petugas instansi**, saya ingin melihat dan memperbarui status laporan milik instansi saya, agar warga tahu perkembangannya. | Given saya login, then saya hanya melihat laporan yang diteruskan ke instansi saya. Given saya mengubah status menjadi Diproses, then timeline publik diperbarui dan log STATUS\_CHANGED tercatat. Given laporan bukan kewenangan instansi saya, when saya memilih "Kembalikan ke Admin" dengan alasan, then Admin menerima notifikasi untuk routing ulang. |
| **US-15** | F6 · P0 | Sebagai **petugas instansi**, saya ingin mengunggah foto bukti perbaikan, agar penyelesaian bisa diverifikasi. | Given saya menandai Selesai, then minimal 1 foto sesudah perbaikan wajib diunggah dan foto sebelum otomatis diambil dari laporan. Given lokasi EXIF foto sesudah lebih dari 100 meter dari titik laporan, then peringatan tidak memblokir tampil dan tanda flag tercatat. |
| **US-16** | F9 · P1 | Sebagai **petugas instansi**, saya ingin melihat curah hujan berdampingan dengan titik jalan berlubang, agar saya bisa memprioritaskan zona rawan. | Given saya membuka dashboard, then peta tumpang susun curah hujan dan frekuensi laporan tampil lengkap dengan waktu pembaruan data BMKG. Given saya mengubah filter periode atau kategori, then kedua layer diperbarui. Given data BMKG tidak tersedia, then data terakhir tetap tampil dengan penanda "Data usang". |
| **US-17** | F9 · P1 | Sebagai **petugas instansi**, saya ingin sistem menyorot titik kambuhan di zona hujan ekstrem, agar saya bisa merencanakan perbaikan yang lebih tahan lama. | Given terdapat titik kambuhan di zona hujan ekstrem, then titik disorot dengan aturan pemicu yang bisa dilihat. Given rekomendasi teknis tampil, then rekomendasi diberi label indikatif dan bukan pengganti survei teknis. Given saya menekan ekspor, then data dalam filter aktif terunduh sebagai CSV. |
| **US-18** | F10 · P1 | Sebagai **warga**, saya ingin menilai kualitas tambalan setelah perbaikan, agar kualitas pekerjaan ikut diawasi. | Given laporan Selesai dan masih dalam masa pemantauan, when pengguna yang eligible memilih "Mulus" atau "Cepat Rusak", then rating tersimpan (satu per pengguna per laporan). Given ambang "Cepat Rusak" tercapai, then status menjadi Dibuka Kembali dan Admin dinotifikasi. Given masa pemantauan berakhir tanpa ambang tercapai, then status menjadi Ditutup. |
| **US-19** | F7 · P0 | Sebagai **Auditor atau Admin**, saya ingin menelusuri riwayat lengkap sebuah laporan, agar akuntabilitas dapat dibuktikan. | Given saya membuka riwayat laporan, then semua event tampil berurutan waktu dengan diff versi Fakta Asal dan draf. Given ada upaya mengubah atau menghapus entri log, then ditolak dan upayanya tercatat. Given saya mengekspor log, then aksi ekspor itu sendiri tercatat. |
| **US-20** | F7 · P1 | Sebagai **Super Admin**, saya ingin mengatur parameter sistem tanpa mengubah kode, agar ambang dan aturan bisa disesuaikan. | Given saya mengubah parameter (radius duplikat, masa pemantauan, ambang Cepat Rusak, batas generate AI), then nilai baru berlaku untuk kejadian berikutnya dan perubahan tercatat di log. |

# **5\. Functional Requirements per Fitur MVP**

Setiap requirement punya ID unik agar bisa dilacak ke desain, tiket engineering, dan test case. Kolom Prioritas memakai P0, P1, P2 seperti pada Bagian 3.3.

## **F1. Form Pelaporan & Geotagging**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Warga |
| **Tujuan** | Membuat Fakta Asal yang lengkap, terstruktur, dan berlokasi akurat dengan usaha pengguna sekecil mungkin. |
| **Aturan bisnis kunci** | Fakta Asal terdiri dari foto, titik GPS, field terstruktur, dan uraian bebas. Setiap perubahan Fakta Asal membuat versi baru dan menghitung ulang facts\_hash. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-1.1** | Form dapat dipakai tanpa login dan nyaman di layar sempit (mulai 320 px). | P0 |
| **FR-1.2** | Unggah foto minimal 1 dan maksimal 3 **\[Usulan PM\]**. Format JPG, PNG, HEIC, maks 10 MB per foto. Kompresi dilakukan di sisi klien sebelum unggah. | P0 |
| **FR-1.3** | Geotagging otomatis lewat Geolocation API. Tampilkan akurasi dalam meter dan peta mini untuk konfirmasi. Titik terkunci setelah dikonfirmasi. | P0 |
| **FR-1.4** | Validasi titik terhadap poligon batas Kota Bogor. Di luar poligon, laporan ditolak dengan pesan jelas. | P0 |
| **FR-1.5** | Jika izin lokasi ditolak atau akurasi buruk (di atas 50 m), pengguna dapat menandai titik manual di peta dan sistem mencatat bahwa lokasi dipilih manual. | P0 |
| **FR-1.6** | Field terstruktur: kategori (lubang, retak, amblas, bergelombang, genangan atau drainase buruk, lainnya), estimasi ukuran (panjang, lebar, kedalaman atau kelas kecil, sedang, besar), patokan lokasi, waktu ditemukan (default sekarang), dampak (pilihan ganda, misalnya membahayakan pengendara roda dua, menimbulkan macet, pernah terjadi kecelakaan), dan uraian bebas maks 1.000 karakter. | P0 |
| **FR-1.7** | Reverse geocoding menyarankan nama jalan sebagai **saran terpisah**. Nilai ini bukan bagian Fakta Asal kecuali pelapor memasukkannya sebagai patokan. | P1 |
| **FR-1.8** | Kontak pelapor (email) opsional dengan persetujuan eksplisit untuk notifikasi. Laporan tetap bisa dikirim tanpa kontak. | P1 |
| **FR-1.9** | Sebelum lanjut, tampilkan laporan aktif dalam radius R (usulan 30 m) dengan kategori mirip dan tawarkan "+1" sebagai ganti membuat laporan baru. Pengguna tetap boleh lanjut membuat laporan baru. | P0 |
| **FR-1.10** | Isian form tersimpan lokal selama 24 jam agar tidak hilang bila halaman tertutup atau sinyal putus. | P1 |
| **FR-1.11** | Perlindungan penyalahgunaan: batas laju per perangkat dan per IP, serta Turnstile atau CAPTCHA pada pengiriman. | P0 |
| **FR-1.12** | Metadata EXIF dibaca untuk keperluan verifikasi Admin. EXIF dihapus dari foto yang tampil publik. | P0 |

## **F2. AI Draft Assistant & Fallback Template**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Warga, Sistem |
| **Tujuan** | Menghasilkan judul dan deskripsi baku dari Fakta Asal tanpa menambah fakta, dan selalu punya jalan keluar deterministik. |
| **Aturan bisnis kunci** | AI hanya merangkum teks. AI tidak menganalisis foto dan tidak menentukan instansi, ukuran, kebenaran, kepatuhan, atau status selesai. Pemanggilan AI tidak boleh memicu email atau perubahan status (zero side-effect). Setiap draf memiliki label permanen: ✨ \[Draf Dibantu AI\] atau 📄 \[Draf Template\]. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-2.1** | Input ke AI hanya berupa field terstruktur dan uraian bebas dalam bentuk teks. Foto, kontak pelapor, dan data pribadi tidak dikirim. | P0 |
| **FR-2.2** | Output berupa Judul (maks 80 karakter) dan Deskripsi (maks 600 karakter) dalam Bahasa Indonesia baku dan netral. | P0 |
| **FR-2.3** | Instruksi sistem melarang AI menambah fakta, mengubah angka atau ukuran, menyebut atau menyimpulkan instansi, menilai kebenaran, menuduh pihak tertentu, atau menyatakan status. | P0 |
| **FR-2.4** | **Validator pascagenerasi \[Usulan PM\]**: periksa bahwa angka dan satuan di output ada di input, tidak ada nama instansi, tidak ada kata terlarang (misalnya tuduhan kelalaian), dan panjang sesuai batas. Jika gagal, jatuh ke Draf Template dan catat alasan VALIDATION\_FAILED. | P0 |
| **FR-2.5** | Batas waktu panggilan AI (usulan 8 detik) dengan maksimal satu percobaan ulang. Setelah itu otomatis ke Draf Template. | P0 |
| **FR-2.6** | Draf Template deterministik: kalimat baku dengan placeholder yang diisi dari Fakta Asal tanpa model bahasa. Pengguna melihat penjelasan singkat saat fallback terjadi. | P0 |
| **FR-2.7** | Label draf tampil permanen di UI warga, UI Admin, halaman pelacakan, dan email eskalasi. | P0 |
| **FR-2.8** | Endpoint pembuatan draf hanya menulis ke tabel draf dan audit log. Endpoint tidak memiliki akses ke layanan email dan tidak mengubah status laporan. | P0 |
| **FR-2.9** | Batas pembuatan ulang draf (usulan 5 kali per laporan dan kuota harian per perangkat) untuk mengendalikan biaya dan penyalahgunaan. | P1 |
| **FR-2.10** | Uraian bebas diperlakukan sebagai data tidak tepercaya. Instruksi di dalam uraian (prompt injection) tidak boleh diikuti dan dibatasi lewat pemisahan konteks serta validator FR-2.4. | P0 |
| **FR-2.11** | Setiap pembuatan draf mencatat versi model, versi prompt, latensi, hasil validasi, dan teks input serta output pada snapshot audit. | P0 |
| **FR-2.12** | Draf bersifat baca saja untuk warga. Untuk mengubah isi, warga mengubah Fakta Asal lalu membuat draf baru (keputusan OQ-13). | P0 |

## **F3. Panel Sanding & Persetujuan (Human-in-the-Loop)**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Warga |
| **Tujuan** | Memastikan manusia meninjau dan menyetujui teks yang akan dikirim, dan persetujuan itu hanya berlaku untuk fakta yang sama. |
| **Aturan bisnis kunci** | Persetujuan hanya sah bila approved\_facts\_hash sama dengan facts\_hash terkini. Aturan invalidasi ditegakkan di server, bukan hanya di UI. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-3.1** | Panel sanding menampilkan Fakta Asal dan Draf berdampingan (desktop) atau berurutan dalam tab (mobile), dengan label draf permanen. | P0 |
| **FR-3.2** | Persetujuan berupa kotak centang eksplisit: "Saya sudah meninjau dan menyetujui draf ini." | P0 |
| **FR-3.3** | Tombol Kirim terkunci sampai ada draf yang tidak kedaluwarsa dan persetujuan yang valid. | P0 |
| **FR-3.4** | **Aturan invalidasi**: setiap perubahan Fakta Asal (foto, GPS, field, uraian) setelah draf dibuat mereset persetujuan, mengunci tombol Kirim, menandai draf lama kedaluwarsa, dan menampilkan banner penjelasan. Warga wajib meninjau ulang draf. | P0 |
| **FR-3.5** | Server menolak pengiriman (kode 409\) bila approved\_facts\_hash tidak sama dengan facts\_hash terkini, apa pun yang dikirim klien. | P0 |
| **FR-3.6** | Saat dikirim, sistem menyimpan snapshot final yang tidak bisa diubah: Fakta Asal, draf, label, versi model dan template, dan hash. | P0 |
| **FR-3.7** | Event SOURCE\_FACTS\_MODIFIED, APPROVAL\_RESET, dan APPROVAL\_GRANTED dicatat di audit log dengan snapshot diff. | P0 |

## **F4. Peta Interaktif, Upvote (+1), dan Tag Kondisi**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 (tag P1) |
| **Aktor** | Warga, Admin |
| **Tujuan** | Menampilkan laporan terverifikasi secara terbuka dan membiarkan komunitas memberi sinyal prioritas tanpa membuat laporan ganda. |
| **Aturan bisnis kunci** | Hanya laporan berstatus Terverifikasi ke atas yang tampil publik (default, OQ-20). Skor prioritas adalah sinyal bantu bagi Admin dan instansi, bukan keputusan otomatis. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-4.1** | Peta publik dengan cluster marker, warna per status, dan batas zoom agar tetap ringan di ponsel. | P0 |
| **FR-4.2** | Filter berdasarkan status, kategori, rentang waktu, dan instansi tujuan. | P0 |
| **FR-4.3** | Detail laporan: foto (tanpa EXIF), judul, deskripsi, label draf, timeline status, jumlah \+1, dan tag. | P0 |
| **FR-4.4** | "+1 Saya Lewat Sini" satu kali per perangkat atau akun per laporan, dapat dibatalkan. Anti-penyalahgunaan lewat batas laju dan sinyal perangkat ringan. | P0 |
| **FR-4.5** | Skor prioritas dihitung dari jumlah \+1, kategori, dampak, umur laporan, dan konteks hujan (setelah F9). Formula dan bobot dapat dikonfigurasi dan dijelaskan di UI Admin (OQ-19). | P1 |
| **FR-4.6** | Tag kondisi dari daftar tertutup (misalnya \[Tertutup Genangan Air\]). Satu pengguna boleh memberi tag yang sama sekali per laporan. Tag tampil sebagai hitungan. Pemberi tag perlu verifikasi OTP email ringan (keputusan OQ-08). | P1 |
| **FR-4.7** | Saat Admin menggabungkan duplikat, \+1 dan tag dipindahkan ke laporan induk tanpa menggandakan pengguna yang sama. | P0 |

## **F5. Moderasi, Routing, dan Eskalasi Admin**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Admin |
| **Tujuan** | Memberi Admin kendali penuh atas kualitas laporan dan atas setiap pengiriman resmi ke instansi. |
| **Aturan bisnis kunci** | Pengiriman email ke instansi dan perubahan status menjadi Diteruskan terjadi hanya setelah Admin menekan tombol konfirmasi. Admin tidak dapat mengubah Fakta Asal. Koreksi memakai catatan Admin yang terpisah. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-5.1** | Antrean moderasi dengan filter status dan pengurutan berdasarkan umur, skor, dan kandidat duplikat. | P0 |
| **FR-5.2** | Tampilan detail: Fakta Asal, draf dengan label, foto asli, perbandingan lokasi EXIF dengan GPS, peta, dan laporan serupa. | P0 |
| **FR-5.3** | Checklist verifikasi: foto autentik, lokasi sesuai, bukan spam, bukan duplikat, kategori masuk akal. | P0 |
| **FR-5.4** | Aksi: Verifikasi, Tolak (alasan wajib dari daftar dan catatan opsional), Tandai atau Gabung Duplikat, Minta Info Tambahan (P1, jika pelapor memberi kontak). | P0 |
| **FR-5.5** | Routing manual lewat dropdown instansi (PUPR Kota Bogor, Pemprov Jabar, Balai Nasional). Sistem boleh menampilkan petunjuk wewenang ruas bila datanya ada, namun keputusan tetap pada Admin. | P0 |
| **FR-5.6** | Pratinjau email resmi: subjek, isi (judul, deskripsi, label draf, koordinat, tautan peta, foto), dan penerima. Email tidak memuat identitas pelapor (keputusan OQ-15). | P0 |
| **FR-5.7** | Tombol "Kirim ke Instansi" dengan dialog konfirmasi. Setelah email diterima penyedia email dengan sukses, status berubah menjadi Diteruskan dan log ESCALATION\_EMAIL\_SENT dicatat. | P0 |
| **FR-5.8** | Jika pengiriman gagal atau memantul (bounce), status tetap Terverifikasi, Admin diberi tahu, dan dapat mengirim ulang. Log ESCALATION\_EMAIL\_FAILED dicatat. | P0 |
| **FR-5.9** | Laporan yang dikembalikan instansi muncul kembali di antrean Admin dengan alasan pengembalian. | P0 |
| **FR-5.10** | Pengingat eskalasi ulang bila tidak ada respons dalam N hari (N dapat dikonfigurasi). Pengiriman ulang tetap manual. | P1 |
| **FR-5.11** | Admin dapat menimpa (override) status dengan alasan wajib bila instansi tidak memakai portal. Override ditandai jelas di timeline dan audit log. | P1 |
| **FR-5.12** | Admin dapat memburamkan area foto yang memuat wajah atau pelat nomor sebelum laporan tampil publik. Foto asli tetap tersimpan terbatas untuk Admin (keputusan OQ-11, blur otomatis ditunda). | P1 |

## **F6. Portal Instansi: Status dan Bukti Perbaikan**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Petugas Instansi |
| **Tujuan** | Memberi instansi cara sederhana menerima laporan, memperbarui status, dan mengunggah bukti sebelum dan sesudah. |
| **Aturan bisnis kunci** | Petugas hanya melihat laporan milik instansinya. Bentuk akses (login penuh atau magic link) bergantung pada OQ-03. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-6.1** | Autentikasi petugas dengan email dan kata sandi serta MFA. Alternatif tautan aman sekali pakai per laporan bila login tidak diterima (OQ-03). | P0 |
| **FR-6.2** | Daftar laporan terbatas pada instansi petugas (isolasi tenant) dengan filter status dan pencarian. | P0 |
| **FR-6.3** | Perubahan status: Diterima, Diproses (dengan perkiraan tanggal opsional), Selesai. Setiap perubahan wajib punya catatan singkat opsional dan tercatat di log. | P0 |
| **FR-6.4** | Status Selesai mensyaratkan minimal satu foto sesudah perbaikan. Foto sebelum diambil otomatis dari laporan. | P0 |
| **FR-6.5** | Cek lokasi EXIF foto sesudah terhadap titik laporan. Jarak lebih dari 100 m memunculkan peringatan dan flag, tidak memblokir. | P1 |
| **FR-6.6** | Aksi "Kembalikan ke Admin" dengan alasan (misalnya bukan kewenangan) agar routing ulang dapat dilakukan. | P0 |
| **FR-6.7** | Notifikasi email ke petugas saat ada laporan baru diteruskan atau pengingat tertunda. | P1 |

## **F7. Audit Trail & Version History**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 · Fase 1 |
| **Aktor** | Sistem, Admin, Auditor |
| **Tujuan** | Menyediakan catatan yang tidak dapat diubah atas semua aksi penting agar akuntabilitas hukum dan operasional dapat dibuktikan. |
| **Aturan bisnis kunci** | Log bersifat append-only. Tidak ada operasi ubah atau hapus pada tabel log di level database. Aksi pengguna anonim dicatat dengan pengenal pseudonim (bukan identitas asli). |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-7.1** | Tabel report\_audit\_logs append-only, dijaga lewat hak akses database dan trigger penolak UPDATE dan DELETE. | P0 |
| **FR-7.2** | Setiap entri berisi timestamp UTC, actor ID dan tipe aktor, action type, report ID, snapshot diff (sebelum dan sesudah), request ID, serta hash IP dan user agent. | P0 |
| **FR-7.3** | Action type wajib: AI\_GENERATE\_SUCCESS, AI\_FALLBACK\_TEMPLATE, SOURCE\_FACTS\_MODIFIED, APPROVAL\_GRANTED, APPROVAL\_RESET, REPORT\_SUBMITTED, VERIFY\_REPORT, REJECT\_REPORT, ROUTE\_ASSIGNED, ESCALATION\_EMAIL\_SENT, ESCALATION\_EMAIL\_FAILED, STATUS\_CHANGED, DUPLICATE\_MERGED, SIGNOFF\_SUBMITTED, REOPENED, ADMIN\_OVERRIDE, LOG\_EXPORTED. | P0 |
| **FR-7.4** | Version history menyimpan setiap versi Fakta Asal dan draf, dengan tampilan diff berdampingan di timeline laporan. | P0 |
| **FR-7.5** | Pengaman tambahan antiubah **\[Usulan PM\]**: setiap entri menyimpan hash entri sebelumnya (hash chain) sehingga manipulasi dapat terdeteksi. | P1 |
| **FR-7.6** | Akses baca log hanya untuk Admin, Auditor, dan Super Admin. Ekspor CSV atau JSON tersedia dan ekspor itu sendiri dicatat. | P0 |
| **FR-7.7** | Kebijakan retensi dan pseudonimisasi mengikuti jawaban OQ-24 agar hak subjek data tidak bertabrakan dengan sifat log yang tidak dapat diubah. | P0 |

## **F8. Pelacakan Status & Notifikasi \[Usulan PM\]**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P0 (halaman) · P1 (notifikasi) |
| **Aktor** | Warga |
| **Tujuan** | Menutup siklus pelaporan: warga selalu bisa melihat apa yang terjadi pada laporannya. Fitur ini tidak ada eksplisit di Product Summary namun dibutuhkan agar G1 dan G4 tercapai. |
| **Aturan bisnis kunci** | Tautan pelacakan memakai token acak yang sulit ditebak. Pelacakan tidak memerlukan login. Teks notifikasi netral dan tidak menjanjikan waktu perbaikan. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-8.1** | Setiap laporan memiliki kode tiket dan tautan pelacakan berisi token acak. | P0 |
| **FR-8.2** | Halaman pelacakan menampilkan timeline status, alasan bila Ditolak atau Duplikat, instansi tujuan setelah Diteruskan, dan bukti sebelum-sesudah bila ada. | P0 |
| **FR-8.3** | Notifikasi email pada perubahan status utama (Terverifikasi, Diteruskan, Diproses, Selesai, Ditolak) hanya bila pelapor memberi kontak dan persetujuan. | P1 |
| **FR-8.4** | Pelapor dapat berhenti berlangganan notifikasi lewat tautan di email. | P1 |

## **F9. Weather-Road Intelligence (BMKG)**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P1 · Fase 2 |
| **Aktor** | Petugas Instansi, Admin |
| **Tujuan** | Membantu instansi melihat hubungan curah hujan dengan frekuensi kerusakan dan titik kambuhan untuk prioritisasi serta rekomendasi konstruksi. |
| **Aturan bisnis kunci** | Rekomendasi teknis berbasis aturan yang transparan, bukan hasil AI generatif. Korelasi tidak dinyatakan sebagai sebab-akibat. Semua rekomendasi berlabel indikatif. Kelayakan fitur bergantung pada hasil spike akses data BMKG (OQ-21). |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-9.1** | Ingest data BMKG terjadwal (harian) lewat API, disimpan sebagai snapshot dengan waktu pengambilan dan sumber. | P1 |
| **FR-9.2** | Peta tumpang susun dua lapis: curah hujan (pada granularitas yang tersedia) dan frekuensi atau kepadatan laporan terverifikasi. | P1 |
| **FR-9.3** | Filter periode, kategori kerusakan, dan wilayah (kecamatan atau kelurahan). | P1 |
| **FR-9.4** | "Zona merah" didefinisikan lewat aturan yang dapat dikonfigurasi, misalnya minimal N laporan terverifikasi dalam radius R dan periode T di area dengan hujan di atas ambang (OQ-23). | P1 |
| **FR-9.5** | "Titik kambuhan" ditandai bila ada laporan baru dalam radius R dari laporan yang sudah Selesai dalam T hari terakhir (nilai dapat dikonfigurasi). | P1 |
| **FR-9.6** | Rekomendasi indikatif berbasis aturan (misalnya tinjau drainase atau pertimbangkan konstruksi beton rigid pavement pada titik kambuhan di zona hujan ekstrem). Aturan ditinjau oleh ahli perkerasan jalan sebelum rilis (OQ-22). | P1 |
| **FR-9.7** | Ekspor data dashboard sebagai CSV dan ringkasan PDF. | P2 |
| **FR-9.8** | Jika BMKG tidak dapat diakses, tampilkan data terakhir dengan penanda "Data usang" dan waktu pembaruan terakhir. Dashboard tetap berfungsi untuk data laporan. | P1 |
| **FR-9.9** | Banner peringatan dini cuaca ekstrem dari BMKG di dashboard. | P2 |

## **F10. Community Sign-Off**

| Atribut | Keterangan |
| :---- | :---- |
| **Prioritas dan fase** | P1 · Fase 2 |
| **Aktor** | Warga |
| **Tujuan** | Membiarkan komunitas memverifikasi kualitas perbaikan selama masa pemantauan. |
| **Aturan bisnis kunci** | Sign-off hanya dibuka setelah instansi menandai Selesai. Ambang pembukaan kembali bersifat konfigurasi dan memberi sinyal ke Admin, bukan eskalasi otomatis ke instansi. |

| ID | Requirement | Prioritas |
| :---- | :---- | :---- |
| **FR-10.1** | Masa pemantauan dimulai saat status Selesai. Durasi default 30 hari **\[Usulan PM\]** dan dapat dikonfigurasi (keputusan OQ-10). | P1 |
| **FR-10.2** | Rating dua pilihan: "Mulus" atau "Cepat Rusak", dengan foto dan komentar opsional. | P1 |
| **FR-10.3** | Yang berhak memberi rating: pelapor asli dan pengguna yang pernah memberi \+1 pada laporan tersebut, dengan verifikasi ringan (OQ-08). | P1 |
| **FR-10.4** | Satu rating per pengguna per laporan. Rating dapat diubah selama masa pemantauan, setiap perubahan tercatat. | P1 |
| **FR-10.5** | Bila jumlah dan proporsi "Cepat Rusak" melewati ambang (default minimal 3 suara dan lebih dari 50%, keputusan OQ-10), status menjadi Dibuka Kembali dan Admin dinotifikasi untuk memutuskan tindak lanjut. | P1 |
| **FR-10.6** | Setelah masa pemantauan berakhir tanpa ambang tercapai, status menjadi Ditutup dan hasil rating diagregasi ke dashboard instansi. | P1 |

# **6\. Model Status Laporan**

Model status ini adalah **\[Usulan PM\]** untuk menyatukan alur pada Product Summary. Perpindahan status selalu menghasilkan entri audit log STATUS\_CHANGED.

| Status | Arti | Pemicu | Aktor |
| :---- | :---- | :---- | :---- |
| **Draft** | Laporan sedang disusun dan belum dikirim. Hanya ada di sisi pelapor | Warga mulai mengisi form | Warga |
| **Menunggu Verifikasi** | Laporan terkirim dan masuk antrean moderasi | Warga menekan Kirim dengan persetujuan valid | Warga |
| **Terverifikasi** | Laporan dinilai valid dan tampil di peta publik | Admin menekan Verifikasi | Admin |
| **Ditolak** | Laporan tidak valid, misalnya spam atau tidak sesuai | Admin menolak dengan alasan | Admin |
| **Duplikat** | Laporan digabung ke laporan induk | Admin menggabungkan | Admin |
| **Diteruskan** | Eskalasi resmi sudah terkirim ke instansi | Admin menekan "Kirim ke Instansi" dan email sukses terkirim | Admin |
| **Diproses** | Instansi menyatakan sedang atau akan menangani | Petugas memperbarui status | Instansi (atau Admin override) |
| **Selesai (Dalam Pemantauan)** | Instansi menandai selesai dengan bukti. Masa pemantauan komunitas berjalan | Petugas menandai Selesai dengan foto sesudah | Instansi (atau Admin override) |
| **Dibuka Kembali** | Komunitas menilai perbaikan cepat rusak | Ambang "Cepat Rusak" tercapai | Sistem lalu Admin |
| **Ditutup** | Masa pemantauan selesai tanpa masalah | Masa pemantauan berakhir | Sistem |

**Catatan alur khusus.** Bila instansi memilih "Kembalikan ke Admin", laporan kembali ke status Terverifikasi dengan penanda Dikembalikan dan alasan, lalu Admin melakukan routing ulang. Bila email eskalasi gagal, status tetap Terverifikasi dan tidak berubah menjadi Diteruskan.

# **7\. Non-Functional Requirements**

Target adalah **usulan awal** dan perlu disesuaikan dengan anggaran, infrastruktur, dan komitmen mitra.

| Kategori | Requirement | Target usulan |
| :---- | :---- | :---- |
| **Performa** | Halaman form dan peta dimuat cepat di jaringan seluler Respons API pengiriman laporan di luar proses unggah foto Peta tetap ringan dengan ribuan titik lewat cluster | LCP ≤ 2,5 detik pada 75% sesi 4G p95 ≤ 1 detik Interaksi peta tetap lancar pada ponsel kelas menengah |
| **AI** | Batas waktu panggilan AI dan jalur fallback Biaya terkendali | Timeout 8 detik, satu retry Kuota harian dan batas generate ulang (FR-2.9) |
| **Ketersediaan** | Layanan pelaporan harus tetap bisa dipakai saat layanan pendukung (AI, BMKG, peta) bermasalah | Ketersediaan 99,5% per bulan Degradasi anggun untuk AI dan BMKG |
| **Skalabilitas** | Lonjakan trafik saat hujan besar atau laporan viral | Mampu menangani 10 kali beban normal dengan autoscaling dan antrean |
| **Keamanan** | Enkripsi dalam perjalanan dan penyimpanan RBAC dan isolasi data antarinstansi MFA untuk Admin dan petugas instansi Validasi file (tipe sebenarnya), pemindaian malware, batas ukuran Perlindungan dari OWASP Top 10 dan prompt injection Manajemen rahasia dan rotasi kunci | TLS 1.2 atau lebih baru Uji penetrasi sebelum peluncuran publik Tidak ada rahasia di klien |
| **Privasi & kepatuhan** | Kepatuhan pada UU No. 27 Tahun 2022 tentang Pelindungan Data Pribadi (perlu konfirmasi hukum) Minimisasi data dan persetujuan eksplisit untuk kontak Hapus EXIF pada foto publik Pertimbangan blur wajah dan pelat nomor Pseudonimisasi aktor anonim di log | Tidak ada data pribadi di publik Kebijakan privasi dan syarat layanan tersedia sebelum rilis Retensi sesuai OQ-24 |
| **Auditabilitas** | Log append-only, sinkronisasi waktu server, dan pemeriksaan integritas | 100% aksi wajib tercatat Pemeriksaan integritas berkala |
| **Aksesibilitas** | Antarmuka dapat dipakai pengguna dengan keterbatasan penglihatan dan motorik | Mengacu WCAG 2.1 level AA untuk alur utama |
| **Kompatibilitas** | Peramban seluler dan desktop utama Tahan koneksi buruk (kompresi foto, antrean pengiriman ulang) | Dua versi mayor terakhir Chrome dan Safari, serta peramban bawaan Android PWA sebagai opsi Fase 2 |
| **Lokalisasi** | Bahasa Indonesia, zona waktu WIB, format tanggal Indonesia | Seluruh antarmuka dan email berbahasa Indonesia |
| **Observabilitas** | Log terstruktur, metrik, dan alert Metrik khusus AI (latensi, tingkat fallback, kegagalan validasi) Metrik bisnis dari Bagian 2.2 | Dashboard dan alert untuk kegagalan email eskalasi, antrean, dan ingest BMKG |
| **Tata kelola AI** | Versi prompt dan model tercatat Set uji regresi berisi kasus halusinasi dan injeksi Kebijakan data ke penyedia AI (OQ-12) | Uji regresi wajib lulus sebelum perubahan prompt atau model dirilis |
| **Cadangan & pemulihan** | Cadangan basis data dan penyimpanan foto, serta uji pemulihan | RPO 24 jam dan RTO 4 jam (usulan) |
| **Maintainability** | CI/CD, feature flag (khususnya AI dan BMKG), dan parameter konfigurasi tanpa deploy ulang | Fitur eksternal dapat dimatikan tanpa mengganggu pelaporan inti |

# **8\. Sketsa Data Model**

Ini adalah sketsa konseptual untuk menyamakan pemahaman antara produk dan engineering, bukan skema final. Nama tabel dan tipe data dapat disesuaikan dengan teknologi yang dipilih. Field yang berisi data pribadi ditandai (PII).

## **8.1 Ringkasan Entitas dan Relasi**

| Entitas | Fungsi | Relasi utama |
| :---- | :---- | :---- |
| **reporters** | Identitas pseudonim pelapor atau pengguna komunitas | 1 reporter ke banyak reports, upvotes, signoffs |
| **users** | Akun Admin, Auditor, Super Admin, dan Petugas Instansi | Petugas terkait ke 1 agency |
| **agencies** | Daftar instansi tujuan dan kontak resminya | 1 agency ke banyak escalations dan agency\_updates |
| **reports** | Entitas inti laporan dan statusnya | Pusat dari hampir semua entitas lain |
| **report\_fact\_versions** | Versi Fakta Asal (sumber kebenaran) | Banyak versi per report |
| **report\_photos** | Foto laporan, sebelum, sesudah, dan sign-off | Banyak foto per report |
| **report\_drafts** | Draf AI atau Template per versi fakta | Terkait ke 1 fact\_version |
| **report\_approvals** | Persetujuan warga atas draf dan hash fakta | Terkait ke 1 draft |
| **report\_audit\_logs** | Log append-only semua aksi penting | Banyak log per report |
| **report\_upvotes dan report\_condition\_tags** | Sinyal komunitas | Banyak per report |
| **admin\_notes** | Catatan Admin terpisah dari Fakta Asal | Banyak per report |
| **escalations** | Rekam pengiriman email resmi ke instansi | Banyak per report, 1 agency |
| **agency\_updates** | Riwayat pembaruan status dari instansi | Banyak per report |
| **signoffs** | Rating komunitas setelah perbaikan | Banyak per report |
| **weather\_observations** | Snapshot data BMKG | Dipakai oleh hotspot\_zones |
| **hotspot\_zones** | Hasil turunan zona merah dan titik kambuhan | Dihitung dari reports dan weather\_observations |
| **system\_config** | Parameter yang dapat dikonfigurasi | Dibaca oleh seluruh modul |

## **8.2 Entitas dan Field Kunci**

### **reports**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id** | UUID (PK) | Pengenal internal |
| **ticket\_code** | string (unik) | Kode tiket yang mudah dibaca warga |
| **tracking\_token** | string (unik) | Token acak untuk tautan pelacakan |
| **reporter\_id** | FK reporters | Pelapor pseudonim |
| **status** | enum | Lihat Bagian 6 |
| **location** | geometry(Point) | Titik GPS final yang terkunci |
| **location\_source** | enum | GPS atau MANUAL\_PIN |
| **current\_facts\_version\_id** | FK | Versi Fakta Asal terbaru |
| **current\_facts\_hash** | string | Hash versi Fakta Asal terbaru |
| **approved\_draft\_id** | FK nullable | Draf yang disetujui dan valid |
| **assigned\_agency\_id** | FK nullable | Instansi tujuan hasil routing Admin |
| **duplicate\_of\_id** | FK nullable | Laporan induk bila Duplikat |
| **priority\_score** | numeric | Skor prioritas turunan |
| **upvote\_count** | int | Cache hitungan \+1 |
| **submitted\_at / verified\_at / forwarded\_at / resolved\_at / closed\_at** | timestamptz | Penanda waktu tiap tahap |
| **returned\_by\_agency** | boolean | Penanda laporan dikembalikan instansi |
| **idempotency\_key** | string (unik) | Mencegah laporan ganda saat kirim ulang |

### **report\_fact\_versions (Fakta Asal)**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id, report\_id, version\_no** | UUID, FK, int | Versi berurutan per laporan |
| **category** | enum | Lubang, retak, amblas, bergelombang, genangan, lainnya |
| **size\_length\_cm / width\_cm / depth\_cm / size\_class** | numeric / enum | Estimasi ukuran dari pelapor |
| **landmark** | text | Patokan lokasi dari pelapor |
| **observed\_at** | timestamptz | Waktu ditemukan |
| **impact\_tags** | string\[\] | Dampak yang dipilih |
| **free\_text** | text | Uraian bebas, maks 1.000 karakter |
| **lat, lng, gps\_accuracy\_m** | numeric | Koordinat dan akurasi saat itu |
| **photo\_ids** | UUID\[\] | Foto yang menjadi bagian versi ini |
| **facts\_hash** | string | Hash dari seluruh isi versi, dasar invalidasi persetujuan |
| **created\_by\_actor, created\_at** | string, timestamptz | Pembuat versi |

### **report\_drafts**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id, report\_id, facts\_version\_id** | UUID, FK, FK | Draf terikat ke versi fakta tertentu |
| **source\_type** | enum | AI atau TEMPLATE (menentukan label permanen) |
| **title, body** | text | Judul dan deskripsi |
| **model\_name, model\_version, prompt\_version** | string | Terisi bila source\_type AI |
| **fallback\_reason** | enum nullable | TIMEOUT, API\_ERROR, VALIDATION\_FAILED, DISABLED |
| **validator\_result** | jsonb | Hasil pemeriksaan FR-2.4 |
| **latency\_ms** | int | Waktu pembuatan |
| **facts\_hash** | string | Hash fakta saat draf dibuat |
| **is\_stale** | boolean | Menjadi true bila fakta berubah |
| **created\_at** | timestamptz | Waktu pembuatan |

### **report\_approvals**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id, report\_id, draft\_id** | UUID, FK, FK | Persetujuan atas satu draf |
| **approved\_facts\_hash** | string | Hash fakta saat disetujui. Harus sama dengan facts\_hash terkini saat kirim |
| **approved\_at** | timestamptz | Waktu persetujuan |
| **reset\_at, reset\_reason** | timestamptz, enum | Terisi bila persetujuan dibatalkan (misalnya SOURCE\_FACTS\_MODIFIED) |

### **report\_audit\_logs (append-only)**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id** | bigserial (PK) | Urutan monoton |
| **occurred\_at** | timestamptz (UTC) | Waktu kejadian |
| **report\_id** | FK nullable | Laporan terkait |
| **actor\_id, actor\_type** | string, enum | WARGA\_PSEUDONIM, ADMIN, INSTANSI, SISTEM, AI |
| **action\_type** | enum | Daftar pada FR-7.3 |
| **snapshot\_before, snapshot\_after** | jsonb | Diff perubahan |
| **request\_id, ip\_hash, user\_agent** | string | Penelusuran teknis. IP disimpan dalam bentuk hash |
| **prev\_entry\_hash, entry\_hash** | string | Hash chain untuk deteksi manipulasi (P1) |

### **escalations**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id, report\_id, agency\_id** | UUID, FK, FK | Satu pengiriman eskalasi |
| **sent\_by\_user\_id, sent\_at** | FK, timestamptz | Admin yang menekan tombol konfirmasi |
| **recipient\_emails** | string\[\] | Alamat resmi saat pengiriman |
| **email\_subject, email\_body\_snapshot** | text | Isi email persis seperti terkirim, termasuk label draf |
| **provider\_message\_id** | string | Referensi penyedia email |
| **delivery\_status** | enum | QUEUED, SENT, DELIVERED, BOUNCED, FAILED |
| **failure\_reason** | text nullable | Alasan gagal atau bounce |

### **agency\_updates**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id, report\_id, agency\_id** | UUID, FK, FK | Satu pembaruan status |
| **new\_status** | enum | DITERIMA, DIPROSES, SELESAI, DIKEMBALIKAN |
| **planned\_date** | date nullable | Perkiraan pengerjaan |
| **note** | text | Catatan petugas atau alasan pengembalian |
| **updated\_by, updated\_at** | FK, timestamptz | Petugas atau Admin override |
| **is\_admin\_override** | boolean | Menandai penimpaan oleh Admin |

### **report\_photos**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **id, report\_id** | UUID, FK | Foto milik laporan |
| **kind** | enum | REPORT, BEFORE, AFTER, SIGNOFF |
| **storage\_key, sha256** | string | Lokasi objek dan hash isi |
| **public\_variant\_key** | string | Versi tanpa EXIF untuk tampilan publik |
| **exif\_taken\_at, exif\_lat, exif\_lng** | nullable | Hanya untuk verifikasi Admin, tidak publik |
| **uploaded\_by\_actor, uploaded\_at** | string, timestamptz | Asal unggahan |

### **report\_upvotes, report\_condition\_tags, signoffs**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **report\_upvotes: report\_id, voter\_key, created\_at** | FK, string | Unik pada (report\_id, voter\_key) |
| **report\_condition\_tags: report\_id, tag\_code, voter\_key** | FK, enum, string | Unik pada (report\_id, tag\_code, voter\_key) |
| **signoffs: report\_id, voter\_key, rating, comment, photo\_id, monitoring\_window\_id** | FK, enum | rating: MULUS atau CEPAT\_RUSAK. Unik pada (report\_id, voter\_key) |

### **weather\_observations dan hotspot\_zones**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **weather\_observations: source, area\_code, area\_level** | string, enum | Granularitas bergantung data BMKG yang tersedia (OQ-21) |
| **weather\_observations: period\_start, period\_end, rainfall\_mm, is\_forecast, fetched\_at, raw\_payload\_ref** | timestamptz, numeric | Snapshot yang bisa ditelusuri |
| **hotspot\_zones: geometry, period, report\_count, rainfall\_mm, recurrence\_count, rule\_version, computed\_at** | geometry, numeric | Hasil turunan. rule\_version menjaga hasil dapat direproduksi |

### **agencies, users, reporters, admin\_notes, system\_config**

| Field | Tipe | Keterangan |
| :---- | :---- | :---- |
| **agencies: name, level, official\_emails, jurisdiction\_note, is\_active** | string, enum | level: KOTA, PROVINSI, NASIONAL |
| **users: role, email, password\_hash atau sso\_id, agency\_id, mfa\_enabled, status** | enum | role: ADMIN, AUDITOR, SUPER\_ADMIN, PETUGAS\_INSTANSI |
| **reporters: pseudonym\_id, device\_hash, contact\_email (PII, terenkripsi, opsional), consent\_at, email\_verified\_at** | string | Kontak hanya ada bila pelapor setuju |
| **admin\_notes: report\_id, author\_id, note, created\_at** | text | Tempat koreksi Admin tanpa mengubah Fakta Asal |
| **system\_config: key, value, updated\_by, updated\_at** | jsonb | Radius duplikat, masa pemantauan, ambang Cepat Rusak, batas generate AI, timeout AI, dan sebagainya |

# **9\. Edge Case & Failure Matrix**

Severity menunjukkan dampak bila perilaku yang diharapkan tidak diterapkan: **Tinggi** (merusak integritas, keamanan, atau kepercayaan), **Sedang** (mengganggu pengalaman atau operasional), **Rendah** (kosmetik atau jarang).

| ID | Area | Skenario | Perilaku yang diharapkan | Severity |
| :---- | :---- | :---- | :---- | :---- |
| **E-01** | Pelaporan | Izin lokasi ditolak atau GPS tidak akurat (di atas 50 m) | Tawarkan penandaan titik manual di peta. Catat location\_source \= MANUAL\_PIN dan tampilkan petunjuk ke Admin. | Sedang |
| **E-02** | Pelaporan | Koordinat di luar batas Kota Bogor | Tolak dengan pesan jelas. Sarankan melapor ke kanal wilayah terkait bila tersedia. | Sedang |
| **E-03** | Pelaporan | Foto terlalu besar, format tidak didukung, atau file bukan gambar sebenarnya (ekstensi dipalsukan) | Validasi tipe asli di server, tolak dengan pesan spesifik. Kompres di klien bila memungkinkan. | Sedang |
| **E-04** | Pelaporan | Koneksi terputus saat mengirim atau tombol Kirim ditekan dua kali | Idempotency key mencegah laporan ganda. Isian tersimpan lokal agar bisa dikirim ulang. | Sedang |
| **E-05** | Pelaporan | EXIF foto menunjukkan lokasi atau waktu jauh berbeda dari GPS dan waktu laporan | Jangan memblokir. Tandai pada tampilan Admin sebagai sinyal verifikasi. | Sedang |
| **E-06** | Pelaporan | Foto memuat wajah, pelat nomor, atau data pribadi | EXIF dihapus untuk publik. Admin dapat menolak atau meminta foto ulang. Blur otomatis dipertimbangkan (OQ-11). | Tinggi |
| **E-07** | Pelaporan | Pelapor mengirim banyak laporan dalam waktu singkat (spam atau bot) | Batas laju, Turnstile atau CAPTCHA, dan penandaan otomatis di antrean Admin. | Sedang |
| **E-08** | Pelaporan | Uraian bebas memuat tuduhan, ujaran kebencian, atau data pribadi pihak lain | Moderasi Admin sebelum tayang publik dan kebijakan konten (OQ-25). Draf AI tidak boleh mengulang tuduhan. | Tinggi |
| **E-09** | AI | API AI timeout atau mengembalikan error | Retry sekali lalu fallback ke Draf Template dengan penjelasan. Log AI\_FALLBACK\_TEMPLATE. | Sedang |
| **E-10** | AI | AI menambahkan angka, ukuran, atau instansi yang tidak ada di input | Validator mendeteksi dan jatuh ke Draf Template dengan alasan VALIDATION\_FAILED. Kasus dimasukkan ke set uji regresi. | Tinggi |
| **E-11** | AI | Uraian bebas berisi instruksi seperti "abaikan aturan sebelumnya" | Diperlakukan sebagai data. Validator menahan output yang melanggar. Kasus masuk set uji injeksi. | Tinggi |
| **E-12** | AI | Output kosong, terpotong, atau bukan Bahasa Indonesia | Dianggap gagal validasi dan fallback ke Draf Template. | Sedang |
| **E-13** | AI | Biaya atau kuota AI habis, atau fitur AI dimatikan lewat feature flag | Semua draf memakai Draf Template dengan label yang benar. Pelaporan tetap berjalan. | Sedang |
| **E-14** | Sanding | Warga mengubah Fakta Asal setelah menyetujui draf | Persetujuan reset, tombol Kirim terkunci, draf kedaluwarsa, banner penjelasan. Log SOURCE\_FACTS\_MODIFIED dan APPROVAL\_RESET. | Tinggi |
| **E-15** | Sanding | Klien memanipulasi request untuk mengirim dengan persetujuan tidak valid | Server membandingkan approved\_facts\_hash dengan facts\_hash terkini dan menolak (409). | Tinggi |
| **E-16** | Sanding | Warga mengubah fakta di dua tab atau perangkat sekaligus | Versi fakta terbaru menang. Persetujuan yang tidak cocok hash otomatis tidak sah dan tab lain diberi tahu saat sinkron. | Sedang |
| **E-17** | Sanding | Warga menutup halaman setelah menyetujui tetapi sebelum kirim | Persetujuan lokal tidak dianggap sah setelah sesi baru. Warga meninjau ulang sebelum kirim. | Rendah |
| **E-18** | Komunitas | Satu orang menekan \+1 berkali-kali atau memakai banyak perangkat | Unik per voter\_key, batas laju, dan sinyal perangkat ringan. Pola mencurigakan ditandai di antrean Admin. | Sedang |
| **E-19** | Komunitas | Dua laporan dari titik sama muncul hampir bersamaan | Kandidat duplikat ditandai. Admin menggabungkan dan \+1 serta tag dipindahkan tanpa menghitung ganda. | Sedang |
| **E-20** | Komunitas | Brigading: banyak akun memberi "Cepat Rusak" untuk menjatuhkan instansi | Ambang memakai jumlah minimum dan proporsi. Dibuka Kembali hanya memberi sinyal ke Admin, bukan eskalasi otomatis. | Sedang |
| **E-21** | Komunitas | Pengguna memberi rating tanpa pernah berada di lokasi | Hanya pengguna eligible (pelapor dan pemberi \+1) yang boleh memberi rating. Verifikasi ringan sesuai OQ-08. | Sedang |
| **E-22** | Eskalasi | Email eskalasi gagal, ditolak, atau memantul | Status tidak berubah menjadi Diteruskan. Admin diberi tahu, bisa mengirim ulang. Log ESCALATION\_EMAIL\_FAILED. | Tinggi |
| **E-23** | Eskalasi | Admin salah memilih instansi | Instansi memakai "Kembalikan ke Admin". Admin routing ulang, riwayat tersimpan. Metrik salah routing dipantau. | Sedang |
| **E-24** | Eskalasi | Admin menekan Kirim dua kali atau dua Admin memproses laporan yang sama | Kunci pada laporan (optimistic locking) dan pencegahan pengiriman ganda. Antrean menampilkan siapa yang sedang membuka. | Sedang |
| **E-25** | Eskalasi | Ruas jalan berada di batas kewenangan atau tidak jelas pemiliknya | Admin dapat memilih lebih dari satu instansi dengan catatan atau menandai "perlu klarifikasi". Keputusan tercatat. | Sedang |
| **E-26** | Admin | Antrean moderasi menumpuk melebihi kapasitas | Pengurutan berdasarkan umur dan skor, peringatan SLA, dan batas pilot per wilayah (R4). | Sedang |
| **E-27** | Instansi | Instansi tidak merespons dalam waktu lama | Pengingat ke Admin untuk eskalasi ulang manual (FR-5.10). Status publik tetap jujur (Diteruskan, belum ada tanggapan). | Sedang |
| **E-28** | Instansi | Instansi tidak mau memakai portal | Jalur magic link atau override Admin dengan penanda jelas di timeline (FR-5.11). | Sedang |
| **E-29** | Instansi | Foto sesudah perbaikan diambil di lokasi berbeda atau tidak relevan | Peringatan EXIF dan flag untuk Admin. Komunitas dapat menilai lewat sign-off. | Sedang |
| **E-30** | Instansi | Perbaikan sementara (tambalan) lalu rusak lagi di titik sama | Dibuka Kembali (sign-off) atau terdeteksi sebagai titik kambuhan pada dashboard (FR-9.5). | Sedang |
| **E-31** | BMKG | API BMKG tidak tersedia, berubah format, atau kuota habis | Tampilkan data terakhir dengan penanda "Data usang". Alert ke tim. Dashboard laporan tetap jalan. | Sedang |
| **E-32** | BMKG | Data cuaca terlalu kasar untuk membedakan kelurahan atau tidak ada riwayat yang cukup | Tampilkan pada granularitas yang tersedia dengan catatan keterbatasan. Jangan menyiratkan presisi palsu. | Sedang |
| **E-33** | BMKG | Titik kerusakan banyak di area tanpa hujan ekstrem | Dashboard tidak memaksa korelasi. Rekomendasi hanya muncul bila aturan terpenuhi dan tetap berlabel indikatif. | Rendah |
| **E-34** | Audit | Penulisan audit log gagal saat aksi penting berlangsung | Aksi dan log ditulis dalam satu transaksi. Bila log gagal, aksi dibatalkan agar tidak ada aksi tanpa jejak. | Tinggi |
| **E-35** | Audit | Permintaan penghapusan data pribadi dari subjek data | Hapus atau anonimkan PII (kontak, foto bila relevan) dan pertahankan log dalam bentuk pseudonim sesuai kebijakan OQ-24. | Tinggi |
| **E-36** | Sistem | Penyimpanan foto atau basis data tidak tersedia sementara | Tampilkan pesan jelas dan simpan isian lokal. Jangan menampilkan keberhasilan palsu. | Tinggi |
| **E-37** | Sistem | Jam perangkat pengguna salah | Timestamp otoritatif selalu dari server. Waktu dari klien hanya disimpan sebagai informasi tambahan. | Rendah |

# **10\. Keputusan Produk dan Asumsi Kerja**

Semua pertanyaan terbuka pada draf sebelumnya kini memakai **rekomendasi default PM sebagai keputusan**. Status **Diputuskan** berarti keputusan produk atau desain yang sekarang berlaku. Status **Asumsi kerja** berarti rekomendasi bergantung pada fakta di luar kendali tim produk (kemitraan, anggaran, kapasitas, akses data), jadi dipakai sementara dan tetap perlu dikonfirmasi. Perubahan keputusan diajukan lewat revisi dokumen.

| No | Pertanyaan | Keputusan (rekomendasi PM) | Alasan | Prioritas | Status |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **OQ-01** | Siapa pemilik dan operator SIJAGO (komunitas, LSM, institusi pendidikan, atau badan hukum lain)? Apakah ada kebijakan privasi dan syarat layanan? | Anggap organisasi sipil yang akan menyiapkan kebijakan privasi dan syarat layanan sebelum pilot. | Menentukan legitimasi di mata instansi dan siapa yang bertanggung jawab atas data. | Blocker | Asumsi kerja |
| **OQ-02** | Apakah sudah ada komitmen atau MoU dengan Dinas PUPR Kota Bogor, dan target waktu respons apa yang disepakati? | Pilot tetap jalan lewat email eskalasi. Target respons ditetapkan bersama setelah pilot dimulai. | Tanpa mitra yang aktif, nilai produk berhenti di Diteruskan. | Blocker | Asumsi kerja |
| **OQ-03** | Apakah petugas PUPR bersedia login ke portal? Bila tidak, siapa yang memperbarui status dan mengunggah bukti? | Sediakan login ringan dan magic link per laporan. Admin boleh override dengan penanda. | Menentukan bentuk F6 dan beban kerja Admin. | Blocker | Diputuskan |
| **OQ-04** | Apakah cakupan hanya di dalam batas administratif Kota Bogor, termasuk jalan nasional dan provinsi yang melewatinya? | Poligon Kota Bogor, semua kelas jalan di dalamnya. | Menentukan validasi lokasi dan daftar instansi. | High | Diputuskan |
| **OQ-05** | Apakah Weather-Road Intelligence (F9) dan Community Sign-Off (F10) masuk rilis pertama atau Fase 2? | Fase 2, setelah F1 sampai F8 terbukti di pilot. | Memengaruhi waktu, anggaran, dan risiko. | High | Diputuskan |
| **OQ-06** | Berapa ukuran tim, anggaran (AI, hosting, peta), dan tanggal target pilot dan peluncuran? | Rencanakan pilot kecil dulu dan hitung biaya per laporan sebelum membuka luas. | Menentukan potongan scope dan pilihan teknologi. | High | Asumsi kerja |
| **OQ-07** | Apakah produk ini akan diluncurkan untuk pengguna nyata, atau berupa proyek akademik atau prototipe? | Susun seolah akan dirilis nyata, lalu tandai bagian yang cukup berupa prototipe. | Memengaruhi kedalaman NFR, kepatuhan hukum, dan kebutuhan operasional. | High | Asumsi kerja |
| **OQ-08** | Aksi apa saja yang boleh tanpa login (lapor, \+1, tag, sign-off)? Apakah sign-off butuh verifikasi seperti OTP email atau WhatsApp? | Lapor dan \+1 tanpa login (berbasis perangkat). Tag dan sign-off butuh verifikasi ringan via OTP email. | Keseimbangan kemudahan dan ketahanan terhadap penyalahgunaan. | High | Diputuskan |
| **OQ-09** | Kanal kontak pelapor: email saja, atau juga WhatsApp? | Email saja pada MVP. | WhatsApp Business API berbiaya dan butuh persetujuan template. | Medium | Diputuskan |
| **OQ-10** | Berapa lama masa pemantauan sign-off dan apa ambang "Cepat Rusak" yang membuka kembali laporan? | 30 hari. Minimal 3 suara dan lebih dari 50% menyatakan Cepat Rusak. | Menentukan perilaku F10 dan beban Admin. | Medium | Diputuskan |
| **OQ-11** | Apakah foto yang memuat wajah atau pelat nomor perlu diburamkan, otomatis atau manual? | Blur manual oleh Admin pada MVP, otomatis dipertimbangkan kemudian. | Privasi dan risiko hukum. | High | Diputuskan |
| **OQ-12** | Penyedia dan model AI apa yang dipakai, dan apakah boleh mengirim teks ke layanan di luar negeri? Apa kebijakan retensi data oleh penyedia? | Kirim hanya teks fakta non-PII. Pilih penyedia dengan opsi tanpa pelatihan dari data pelanggan dan tinjau ketentuan transfer data. | Kepatuhan UU PDP dan risiko kebocoran. | Blocker | Diputuskan |
| **OQ-13** | Apakah warga boleh mengedit draf secara manual? Bila boleh, labelnya menjadi apa? | Tidak boleh. Warga mengubah Fakta Asal lalu membuat draf baru. | Memengaruhi aturan invalidasi dan akuntabilitas teks. | High | Diputuskan |
| **OQ-14** | Berapa anggaran AI per laporan dan kuota harian yang dapat diterima? | Maks 5 pembuatan ulang per laporan dan kuota harian per perangkat. | Mengendalikan biaya dan penyalahgunaan. | Medium | Diputuskan |
| **OQ-15** | Apakah identitas pelapor boleh diteruskan ke instansi? | Tidak diteruskan. Hanya kode tiket, kecuali pelapor setuju. | Privasi pelapor dan kemampuan instansi menghubungi. | High | Diputuskan |
| **OQ-16** | Siapa Admin, berapa orang, jam operasional, dan SLA verifikasi yang realistis? | Mulai dengan SLA 2 hari kerja dan batasi wilayah pilot sesuai kapasitas. | Kapasitas moderasi menentukan skala pilot. | Blocker | Asumsi kerja |
| **OQ-17** | Daftar resmi instansi dan alamat email penerima, serta sumber data kewenangan ruas jalan (kota, provinsi, nasional)? | Tabel instansi manual dan daftar ruas sederhana. GIS wewenang dipertimbangkan setelah MVP. | Menentukan akurasi routing dan isi dropdown. | High | Asumsi kerja |
| **OQ-18** | Apakah email dari platform cukup sebagai bukti penyampaian, atau instansi butuh surat resmi atau format tertentu? | Email dengan format baku dan lampiran, sambil menjajaki format yang diminta instansi. | Menentukan format eskalasi dan nilai hukumnya. | High | Diputuskan |
| **OQ-19** | Bagaimana formula skor prioritas (bobot \+1, dampak, kategori, umur, hujan)? | Formula sederhana yang transparan dan dapat dikonfigurasi, ditinjau setelah ada data pilot. | Skor memengaruhi urutan kerja Admin dan instansi. | Medium | Diputuskan |
| **OQ-20** | Apakah laporan berstatus Menunggu Verifikasi boleh tampil publik? | Tidak. Hanya yang Terverifikasi tampil di peta publik. | Menyeimbangkan transparansi dengan risiko spam dan fitnah. | High | Diputuskan |
| **OQ-21** | Endpoint BMKG mana yang dipakai, apa syarat penggunaannya dan kuotanya, pada granularitas apa data tersedia untuk Bogor, dan apakah ada data curah hujan historis harian? | Lakukan spike teknis sebelum komitmen Fase 2\. Siapkan sumber alternatif bila perlu. | Menentukan kelayakan F9. Ketersediaan dan granularitas belum saya verifikasi. | Blocker | Asumsi kerja |
| **OQ-22** | Siapa ahli perkerasan jalan atau insinyur sipil yang dapat meninjau aturan rekomendasi (drainase, rigid pavement)? | Rekomendasi berlabel indikatif dan ditinjau ahli sebelum rilis Fase 2\. | Rekomendasi teknis tanpa review ahli berisiko menyesatkan. | High | Asumsi kerja |
| **OQ-23** | Bagaimana definisi "zona merah" dan "titik kambuhan" (jumlah laporan, radius, periode, ambang hujan)? | Parameter dapat dikonfigurasi dan dikalibrasi dengan data pilot dan masukan PUPR. | Menentukan logika inti dashboard. | Medium | Diputuskan |
| **OQ-24** | Berapa lama data dan audit log disimpan, dan bagaimana menyelaraskan hak hapus data dengan log yang tidak dapat diubah? | Pseudonimisasi aktor dan penghapusan PII terpisah dari log. Retensi ditetapkan bersama pihak hukum. | Kepatuhan hukum dan desain basis data. | High | Diputuskan |
| **OQ-25** | Apa kebijakan konten untuk laporan yang memuat tuduhan, SARA, atau data pribadi pihak lain? | Moderasi sebelum tayang dan alasan penolakan baku. Kebijakan tertulis sebelum pilot. | Melindungi pengguna dan platform. | High | Diputuskan |
| **OQ-26** | Apakah tersedia data aduan jalan rusak historis dari kanal eksisting untuk baseline metrik dan validasi masalah? | Kumpulkan data dan lakukan wawancara pada Fase 0\. | Target metrik dan hipotesis masalah masih belum berbasis data. | High | Asumsi kerja |
| **OQ-27** | Apa nama produk resmi dan identitas merek? Ditetapkan: SIJAGO (Sistem Jalan Bogor). | SIJAGO dipakai untuk seluruh platform termasuk sisi Admin. Cek ketersediaan domain, akun media sosial, dan pendaftaran merek sebelum rilis publik. | Konsistensi teks, email, dan domain. | Medium | Diputuskan |
| **OQ-28** | Penyedia peta dasar dan geocoding apa yang dipakai, dan berapa anggarannya? | Pilih penyedia peta terbuka untuk pilot dan evaluasi ulang saat beban naik. | Biaya, batas penggunaan, dan syarat atribusi berbeda antarpenyedia. | Medium | Diputuskan |
| **OQ-29** | Apakah tersedia domain kustom dan anggaran paket hosting berbayar (Vercel, Render, Neon) untuk pilot? | Pakai domain kustom dengan subdomain terpisah untuk web dan API, serta paket berbayar minimal untuk backend dan database selama pilot. | Cookie lintas situs, instance gratis Render yang tertidur, dan batas paket gratis dapat merusak pengalaman pilot. | High | Asumsi kerja |
| **OQ-30** | Di mana foto laporan disimpan? | Cloudflare R2 dengan bucket privat untuk foto asli dan varian publik tanpa EXIF. Alternatif Vercel Blob atau AWS S3. | Neon hanya menyimpan data relasional. Foto butuh object storage dengan akses privat dan varian publik. | Medium | Diputuskan |

# **11\. Risiko, Asumsi, dan Rencana Rilis**

## **11.1 Risiko Utama**

| ID | Risiko | Peluang | Dampak | Mitigasi |
| :---- | :---- | :---- | :---- | :---- |
| R1 | Instansi tidak merespons atau tidak mau memakai portal | Tinggi | Tinggi | Jalin komitmen awal (OQ-02, OQ-03), siapkan jalur magic link dan override Admin, tunjukkan nilai lewat dashboard |
| R2 | Draf AI memuat fakta yang tidak ada di input | Sedang | Tinggi | Validator deterministik pascagenerasi, fallback template, sanding wajib, audit sampling berkala |
| R3 | Spam, foto palsu, atau laporan fitnah | Sedang | Tinggi | Moderasi sebelum tayang publik, rate limit, Turnstile atau CAPTCHA, pemeriksaan EXIF, kebijakan konten |
| R4 | Beban moderasi melebihi kapasitas Admin | Sedang | Sedang | Antrean berprioritas, SLA realistis, batasi pilot per wilayah |
| R5 | Data BMKG tidak tersedia pada granularitas atau riwayat yang dibutuhkan | Sedang | Tinggi | Spike teknis sebelum komitmen Fase 2 (OQ-21), siapkan sumber alternatif |
| R6 | Korelasi hujan dan kerusakan dibaca sebagai sebab-akibat pasti | Sedang | Sedang | Label "indikatif", aturan transparan, review ahli perkerasan jalan (OQ-22) |
| R7 | Kebocoran atau penyalahgunaan data pribadi (foto, lokasi, kontak) | Rendah | Tinggi | Minimisasi data, strip EXIF publik, kepatuhan UU PDP, enkripsi, pembatasan akses |
| R8 | Ekspektasi publik melebihi kapasitas perbaikan, memicu kekecewaan | Tinggi | Sedang | Komunikasi status yang jujur, tampilkan alasan Ditolak atau Dikembalikan, jangan menjanjikan SLA perbaikan |
| R9 | Salah routing karena kewenangan ruas tidak jelas | Sedang | Sedang | Tabel ruas jalan dan kewenangan, fitur Kembalikan ke Admin, metrik salah routing |

## **11.2 Asumsi dan Dependensi**

* SIJAGO memiliki tim Admin yang bisa memoderasi secara rutin pada jam kerja (OQ-16).

* Alamat email resmi tiap instansi tujuan tersedia dan bersedia menerima laporan dari platform (OQ-17, OQ-18).

* BMKG menyediakan akses data yang cukup untuk wilayah Bogor dengan syarat penggunaan yang memungkinkan (OQ-21).

* Penyedia layanan AI dapat dipakai dengan kebijakan data yang sesuai (OQ-12).

* Penyedia peta dasar, geocoding, penyimpanan objek, dan email transaksional tersedia dengan anggaran yang disetujui (OQ-28).

## **11.3 Rencana Rilis Bertahap \[Usulan PM\]**

| Tahap | Isi | Kriteria keluar (exit criteria) |
| :---- | :---- | :---- |
| **Fase 0: Discovery** | Wawancara warga dan petugas, audit kanal eksisting, spike teknis BMKG dan AI, jawaban Open Questions | Asumsi kerja berstatus Blocker terkonfirmasi, baseline metrik ditetapkan, PRD v1.0 dikunci |
| **Fase 1: MVP Core (pilot terbatas)** | F1 sampai F8 pada 1 sampai 2 kecamatan atau jalur tertentu | Alur ujung ke ujung berjalan, 0 insiden halusinasi lolos, audit log lengkap, minimal satu siklus laporan sampai Selesai bersama instansi |
| **Fase 2: MVP+** | F9 Weather-Road Intelligence dan F10 Community Sign-Off, perluasan wilayah | Dashboard dipakai petugas, validasi aturan rekomendasi oleh ahli, hasil sign-off terbaca di dashboard |

# **12\. Arsitektur dan Tech Stack**

Bagian ini menerjemahkan kebutuhan produk menjadi pilihan teknis untuk SIJAGO. Pilihan inti **ditetapkan oleh pemilik produk**: Next.js (frontend), Express.js (backend), Neon PostgreSQL (database), Drizzle (ORM), Vercel (deployment frontend), dan Render (deployment backend). Library dan tool pendamping adalah **\[Usulan PM\]** dan dapat diganti selama memenuhi requirement dan NFR. Saya tidak memverifikasi harga, kuota, atau batas paket layanan, sehingga hal itu masuk daftar verifikasi pada Bagian 12.7.

## **12.1 Keputusan Arsitektur**

| Keputusan | Pilihan | Alasan |
| :---- | :---- | :---- |
| **Pemisahan frontend dan backend** | Next.js hanya untuk antarmuka. Seluruh logika bisnis ada di Express API | Aturan inti (invalidasi persetujuan, audit log, routing, eskalasi) harus ditegakkan di satu tempat di server dan tidak terduplikasi di Route Handlers Next.js |
| **Bentuk API** | REST JSON dengan prefix /v1, validasi Zod, kontrak didokumentasikan dengan OpenAPI | Sederhana, mudah diuji, dan mudah dipakai klien lain di masa depan |
| **Akses data** | Drizzle ORM dengan driver pg ke connection string pooled milik Neon | Type-safe, migrasi eksplisit lewat drizzle-kit, dan SQL tetap bisa dipakai langsung untuk query PostGIS |
| **Data geospasial** | Ekstensi PostGIS di Neon, kolom geometry untuk titik laporan dan poligon batas kota | Dibutuhkan untuk validasi batas wilayah, deteksi duplikat radius, dan analitik zona merah |
| **Pekerjaan latar belakang** | Antrean berbasis Postgres (pg-boss) dan Render Cron Job | Tidak perlu menambah Redis. Cocok untuk kirim email, retry, dan ingest BMKG berkala |
| **Penyimpanan foto** | Object storage terpisah (usulan Cloudflare R2) dengan unggah langsung memakai presigned URL | Neon menyimpan data relasional saja. Foto asli privat, varian publik tanpa EXIF |
| **Sesi dan cookie lintas domain** | Domain kustom dengan subdomain sama untuk web dan API, atau proxy lewat rewrites Next.js | Cookie lintas situs antara domain vercel.app dan onrender.com rawan diblokir peramban, terutama Safari |
| **Wilayah layanan** | Wilayah Singapura untuk Vercel Functions, Render, dan Neon bila tersedia pada paket yang dipakai | Latensi paling dekat untuk pengguna di Indonesia |

## **12.2 Tech Stack**

*Kolom Status: **Ditetapkan** berasal dari pemilik produk, **Usulan** berasal dari PM.*

| Area | Lapisan | Pilihan | Status | Catatan |
| :---- | :---- | :---- | :---- | :---- |
| Frontend | **Framework** | Next.js (App Router) \+ TypeScript | Ditetapkan | Halaman publik seperti peta dan pelacakan memakai rendering server agar cepat di jaringan seluler |
| Frontend | **UI dan styling** | Tailwind CSS \+ shadcn/ui (Radix UI) | Usulan | Komponen aksesibel untuk target WCAG 2.1 AA dan cepat dibangun |
| Frontend | **Form dan validasi** | React Hook Form \+ Zod | Usulan | Skema Zod dibagikan ke backend lewat package shared |
| Frontend | **Data fetching** | TanStack Query | Usulan | Cache, retry, dan optimistic update untuk \+1 |
| Frontend | **Peta** | MapLibre GL JS (react-map-gl) | Usulan | Cluster marker bawaan untuk sumber GeoJSON. Penyedia tile ditentukan lewat OQ-28 dan harus mengizinkan pemakaian produksi |
| Frontend | **Grafik dashboard (Fase 2\)** | Recharts | Usulan | Cukup untuk grafik hujan versus laporan |
| Frontend | **Foto di sisi klien** | browser-image-compression, exifr | Usulan | Kompresi sebelum unggah dan pembacaan EXIF untuk verifikasi (FR-1.2, FR-1.12) |
| Frontend | **Anti-bot** | Cloudflare Turnstile | Usulan | Memenuhi FR-1.11, token diverifikasi di server |
| Frontend | **Isian lokal dan PWA** | Zustand \+ idb-keyval, Serwist (Fase 2\) | Usulan | Simpan isian 24 jam (FR-1.10) dan antrean kirim ulang saat sinyal putus |
| Frontend | **Tanggal** | date-fns (locale id), zona WIB | Usulan | Format tanggal Indonesia |
| Backend | **Runtime dan framework** | Node.js LTS \+ TypeScript \+ Express.js | Ditetapkan | Dijalankan sebagai Web Service di Render |
| Backend | **Validasi dan dokumen API** | Zod \+ zod-to-openapi | Usulan | Satu sumber skema untuk validasi dan dokumentasi |
| Backend | **ORM dan migrasi** | Drizzle ORM \+ drizzle-kit, driver pg | Ditetapkan | Tipe geometry dan query PostGIS lewat helper SQL. Pastikan didukung pada versi Drizzle yang dipakai (spike) |
| Backend | **Autentikasi dan RBAC** | Better Auth (adapter Drizzle, 2FA TOTP), middleware RBAC | Usulan | Alternatif manual: argon2 \+ otplib \+ cookie session. Berlaku untuk Admin, Auditor, Super Admin, dan Petugas Instansi |
| Backend | **Keamanan HTTP** | helmet, cors (allowlist), express-rate-limit, hpp | Usulan | Batas laju ke FR-1.11 dan FR-4.4. Bila lebih dari satu instance, simpan hitungan di penyimpanan bersama |
| Backend | **Logging** | pino \+ pino-http dengan request ID | Usulan | Request ID ikut ke audit log (FR-7.2) |
| Backend | **Antrean dan jadwal** | pg-boss, Render Cron Job | Usulan | Kirim email, retry, ingest BMKG. Polling pg-boss dapat menjaga compute Neon tetap aktif sehingga perlu dipantau |
| Backend | **Pemrosesan gambar** | sharp, file-type | Usulan | Cek tipe asli file, hapus EXIF, ubah ukuran, dan buramkan area (FR-5.12). Re-encode juga menetralkan file berbahaya |
| Backend | **Penyimpanan objek** | Cloudflare R2 lewat @aws-sdk/client-s3 dan presigner | Usulan | Bucket privat untuk foto asli dan bucket atau path publik untuk varian tanpa EXIF. Alternatif Vercel Blob atau AWS S3 |
| Backend | **Email transaksional** | Resend \+ React Email | Usulan | Webhook bounce memperbarui delivery\_status eskalasi (FR-5.8). Alternatif Brevo atau Amazon SES |
| Backend | **Modul AI** | Adapter penyedia AI yang dapat diganti, AbortController untuk timeout, output terstruktur divalidasi Zod | Usulan | Penyedia dan kebijakan data ditentukan lewat OQ-12. Validator FR-2.4 dan fallback template ada di modul ini |
| Backend | **Geospasial** | PostGIS, turf.js untuk utilitas ringan | Usulan | ST\_Contains untuk batas kota dan ST\_DWithin untuk duplikat dan titik kambuhan |
| Backend | **Hash dan audit** | node:crypto (SHA-256), trigger Postgres | Usulan | facts\_hash, hash chain log, dan penolakan UPDATE atau DELETE pada report\_audit\_logs (FR-7.1, FR-7.5) |
| Database | **Database** | Neon PostgreSQL (ekstensi postgis dan pgcrypto) | Ditetapkan | Pakai connection string pooled. Manfaatkan branching untuk lingkungan preview dan staging |
| Kualitas | **Pengujian** | Vitest, Supertest, Playwright, MSW | Usulan | Unit dan integrasi API, E2E alur lapor sampai kirim, serta mock layanan AI, BMKG, dan email |
| Kualitas | **Set uji AI** | Kumpulan kasus halusinasi dan injeksi dalam Vitest | Usulan | Wajib lulus sebelum prompt atau model berubah (NFR Tata kelola AI) |
| Kualitas | **Kualitas kode** | ESLint, Prettier, Husky \+ lint-staged, commitlint | Usulan | Termasuk aturan no-restricted-imports untuk menjaga modul AI bebas efek samping |
| Kualitas | **Keamanan rantai pasok** | Dependabot, pemindaian rahasia (gitleaks) | Usulan | Pin versi, pilih rilis yang sudah matang |
| Operasional | **Monorepo dan build** | pnpm workspaces \+ Turborepo | Usulan | Berbagi skema Zod dan konfigurasi antar aplikasi |
| Operasional | **CI/CD** | GitHub Actions, Vercel dan Render auto deploy | Usulan | Pipeline: lint, typecheck, test, cek migrasi Drizzle, lalu deploy |
| Operasional | **Pemantauan galat** | Sentry (frontend dan backend) | Usulan | Termasuk alert kegagalan eskalasi dan ingest |
| Operasional | **Uptime** | UptimeRobot atau Better Stack | Usulan | Pantau endpoint kesehatan API dan halaman publik |
| Operasional | **Analitik produk** | Umami atau Plausible (tanpa cookie) | Usulan | Mengukur completion rate dan waktu pengisian tanpa melacak individu |
| Operasional | **Deployment** | Vercel (frontend), Render (backend dan cron) | Ditetapkan | Lihat Bagian 12.5 |

## **12.3 Alur Utama Sistem**

### **Mengirim laporan (F1 sampai F3)**

* Browser mengompres foto lalu meminta presigned URL dari API dan mengunggah langsung ke object storage.

* Klien menyimpan Fakta Asal ke API. API membuat versi fakta baru dan menghitung facts\_hash di server.

* Klien meminta draf. Modul AI memanggil penyedia dengan timeout, memvalidasi hasil, dan jatuh ke Draf Template bila gagal.

* Warga menyetujui draf. API menyimpan approved\_facts\_hash.

* Klien memanggil kirim dengan header Idempotency-Key. API membandingkan hash dan menolak dengan 409 bila berbeda. Bila cocok, snapshot final dan entri audit log ditulis dalam satu transaksi database.

### **Eskalasi ke instansi (F5)**

* Admin memilih instansi, membuka pratinjau, lalu menekan konfirmasi.

* API membuat catatan escalations berstatus QUEUED dan mengirim email melalui penyedia email dengan timeout pendek.

* Hanya bila penyedia menerima email, status laporan menjadi Diteruskan dan ESCALATION\_EMAIL\_SENT dicatat. Bila gagal, status tetap Terverifikasi dan Admin dapat mengirim ulang.

* Webhook penyedia email memperbarui delivery\_status (misalnya BOUNCED).

### **Peta publik dan dashboard (F4 dan F9)**

* Peta memanggil endpoint publik berparameter bounding box dan filter, hasilnya GeoJSON dari PostGIS dengan cache HTTP singkat.

* Job terjadwal mengambil data BMKG, memvalidasi dengan Zod, menyimpan snapshot, lalu menghitung ulang hotspot\_zones.

## **12.4 Struktur Repositori yang Diusulkan**

* **apps/web**: aplikasi Next.js (UI warga, Admin, dan portal instansi).

* **apps/api**: aplikasi Express (route, service, modul AI, modul audit, job).

* **packages/db**: skema Drizzle, migrasi, dan helper query PostGIS.

* **packages/shared**: skema Zod, enum status dan action type, serta tipe bersama.

* **packages/config**: konfigurasi ESLint, TypeScript, dan Prettier.

Konvensi API: awalan **/v1**, galat memakai format problem+json, paginasi berbasis cursor, dan header **Idempotency-Key** pada endpoint pengiriman.

## **12.5 Lingkungan dan Deployment**

| Lingkungan | Frontend | Backend | Database |
| :---- | :---- | :---- | :---- |
| **Lokal** | localhost (Next.js dev) | localhost (Express) | Docker dengan image Postgres \+ PostGIS |
| **Preview (per pull request)** | Vercel Preview Deployment | Render preview atau service staging bersama | Neon branch per PR |
| **Staging** | Vercel (branch staging) | Render service staging | Neon branch staging |
| **Production** | Vercel Production | Render service production \+ Cron Job | Neon branch utama dengan pooled connection |

Rahasia (kunci AI, email, storage, Turnstile) disimpan sebagai environment variable di Vercel dan Render. Variabel divalidasi dengan Zod saat aplikasi mulai agar salah konfigurasi langsung terdeteksi. Tidak ada rahasia di kode klien.

## **12.6 Pemetaan Aturan Kritis ke Implementasi**

| Aturan produk | Cara menegakkan di stack |
| :---- | :---- |
| **Persetujuan batal bila Fakta Asal berubah (FR-3.4, FR-3.5)** | facts\_hash dihitung di server. Endpoint kirim membandingkan approved\_facts\_hash dengan facts\_hash terkini dan menolak dengan 409 |
| **AI tanpa efek samping (FR-2.8)** | Modul AI tidak boleh mengimpor modul email atau modul status. Dijaga dengan aturan no-restricted-imports dan dicakup test |
| **Fallback AI (FR-2.4 sampai FR-2.6)** | Timeout lewat AbortController, satu retry, validator deterministik, lalu Draf Template tanpa model bahasa |
| **Audit log tidak bisa diubah (FR-7.1)** | Role database aplikasi tidak punya hak UPDATE atau DELETE pada tabel log, ditambah trigger penolak dan hash chain |
| **Aksi dan log satu transaksi (E-34)** | Setiap perubahan status dan penulisan audit log berada dalam satu transaksi Drizzle |
| **Tidak ada pengiriman ganda (US-06, E-24)** | Idempotency-Key unik di database dan penguncian optimistik pada laporan |
| **Isolasi data antarinstansi (BR-18)** | Middleware RBAC dan filter agency\_id wajib pada seluruh query portal instansi |
| **EXIF tidak tampil publik (FR-1.12)** | sharp membuat varian publik tanpa EXIF. Foto asli hanya diakses Admin lewat URL bertanda tangan berumur pendek |
| **Email eskalasi hanya setelah konfirmasi (BRL-06)** | Endpoint eskalasi hanya dapat dipanggil dengan peran Admin dan tindakan eksplisit dari UI. Tidak ada job yang memicunya otomatis |

## **12.7 Hal yang Harus Diverifikasi Sebelum Build**

| Belum saya verifikasi Paket dan kuota. Harga dan batas Vercel, Render, Neon, dan object storage untuk beban pilot. Instance gratis Render. Umumnya layanan tertidur saat sepi sehingga request pertama lambat dan bisa melewati timeout AI. Gunakan instance berbayar saat pilot (OQ-29). Neon. Perilaku scale-to-zero, pengaruh polling pg-boss, dan ketersediaan PostGIS serta wilayah Singapura pada paket yang dipakai. Drizzle dan PostGIS. Lakukan spike singkat untuk tipe geometry, migrasi, dan query ST\_DWithin pada versi yang dipakai. Domain. Domain kustom dengan subdomain sama untuk web dan API, atau proxy lewat rewrites, agar cookie sesi bekerja di semua peramban. Lisensi data. Syarat penggunaan tile peta, sumber poligon batas kota, dan data BMKG (OQ-21, OQ-28). Penyedia AI. Kebijakan data dan lokasi pemrosesan (OQ-12). Versi library. Pilih versi stabil yang sudah matang saat proyek dimulai dan kunci dengan lockfile. |
| :---- |

