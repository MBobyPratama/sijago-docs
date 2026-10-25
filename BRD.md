**BUSINESS REQUIREMENTS DOCUMENT**

**SIJAGO**

**Sistem Jalan Bogor**

Platform Pelaporan Kerusakan Jalan Kota Bogor  \|  Civic-Tech & Weather-Road Intelligence

| Atribut | Keterangan |
| :---- | :---- |
| **Versi dokumen** | 0.3 (Draft, nama SIJAGO dan platform teknologi ditambahkan) |
| **Tanggal** | 25 Oktober 2026 |
| **Status** | Draft. Rekomendasi default PM sudah diterapkan sebagai keputusan (Bagian 16). Butir berstatus Asumsi kerja menunggu konfirmasi sponsor sebelum dikunci menjadi v1.0 |
| **Dokumen pendamping** | PRD SIJAGO v0.3 (detail fungsional, data model, edge case, dan tech stack) |
| **Pemilik dokumen** | Product Manager / Business Analyst |
| **Sponsor bisnis** | \[Diisi: nama dan jabatan sponsor\] |

| Cara membaca dokumen ini BRD menjelaskan mengapa produk dibuat dan apa kebutuhan bisnisnya. Detail bagaimana sistem berperilaku ada di PRD. Setiap kebutuhan bisnis (BR) dipetakan ke fitur PRD pada Bagian 15\. Angka target adalah usulan awal dan belum berbasis baseline. Kolom bertanda \[Diisi\] menunggu informasi dari sponsor. Tanda \[Usulan BA\] menandai hal yang saya tambahkan karena dibutuhkan agar bisnis dapat berjalan. |
| :---- |

# **Riwayat Dokumen & Daftar Isi**

| Versi | Tanggal | Perubahan |
| :---- | :---- | :---- |
| 0.1 | 25 Okt 2026 | Draft awal BRD dari Product Summary & Scope. |
| 0.2 | 25 Okt 2026 | Rekomendasi default PM diterapkan sebagai keputusan dan asumsi kerja (Bagian 16). |
| 0.3 | 25 Okt 2026 | Nama produk SIJAGO ditetapkan. Platform teknologi ditambahkan (Bagian 12.4). |

1\. Ringkasan Eksekutif

2\. Latar Belakang Bisnis dan Peluang

3\. Tujuan Bisnis

4\. Ruang Lingkup Bisnis

5\. Pemangku Kepentingan

6\. Proses Bisnis: Kondisi Saat Ini dan Target

7\. Kebutuhan Bisnis

8\. Aturan Bisnis

9\. Model Operasi dan Tata Kelola

10\. Analisis Biaya dan Manfaat

11\. Kriteria Keberhasilan dan KPI

12\. Asumsi, Batasan, dan Dependensi

13\. Risiko Bisnis

14\. Kepatuhan dan Aspek Hukum

15\. Implementasi, Adopsi, dan Matriks Keterlacakan

16\. Keputusan, Asumsi Kerja, dan Persetujuan

# **1\. Ringkasan Eksekutif**

**SIJAGO (Sistem Jalan Bogor)** adalah layanan web **mobile-first** yang memudahkan warga Kota Bogor melaporkan kerusakan jalan, membantu pengelola platform memverifikasi dan meneruskan laporan ke instansi yang berwenang, dan memberi instansi (terutama Dinas PUPR) data untuk memprioritaskan perbaikan, termasuk hubungan antara curah hujan BMKG dengan titik kerusakan.

Nilai bisnis utamanya ada pada tiga hal. Pertama, laporan warga menjadi lebih lengkap, terstruktur, dan sampai ke instansi yang benar. Kedua, tindak lanjut menjadi terlihat dan bisa diawasi komunitas. Ketiga, instansi mendapat dasar data untuk beralih dari perbaikan reaktif ke perbaikan yang lebih tahan lama di zona hujan ekstrem.

Pembeda utama produk adalah **model hibrida dengan kendali manusia**. AI hanya membantu merapikan teks laporan dan tidak mengambil keputusan. Pengiriman resmi ke instansi hanya terjadi setelah Admin mengonfirmasi, dan seluruh aksi penting tercatat dalam audit log yang tidak dapat diubah. Pendekatan ini dipilih agar platform dapat dipercaya oleh warga dan oleh instansi.

### **Hal yang perlu dikonfirmasi sponsor**

* Menyetujui tujuan bisnis dan ruang lingkup pada Bagian 3 dan 4\.

* Mengonfirmasi sponsor, pemilik operasional, dan tim Admin (Bagian 5 dan 9).

* Mengonfirmasi strategi kemitraan dengan Dinas PUPR Kota Bogor (OQ-B02).

* Menyetujui pendekatan bertahap: Discovery, pilot terbatas, lalu perluasan (Bagian 15).

# **2\. Latar Belakang Bisnis dan Peluang**

## **2.1 Konteks**

Kota Bogor dikenal sebagai Kota Hujan. Intensitas hujan yang tinggi diduga mempercepat kerusakan permukaan jalan, terutama pada titik dengan drainase buruk. Warga menjadi pihak pertama yang mengetahui kerusakan, sedangkan perbaikan menjadi tanggung jawab instansi yang berbeda tergantung status ruas jalan (kota, provinsi, atau nasional).

| Catatan validasi Masalah pada tabel di bawah adalah hipotesis awal dari Product Summary dan belum divalidasi dengan data. Validasi dilakukan pada fase Discovery lewat wawancara warga dan petugas serta analisis kanal aduan yang sudah ada. |
| :---- |

## **2.2 Masalah Bisnis**

| Pihak | Masalah | Konsekuensi bisnis |
| :---- | :---- | :---- |
| **Warga** | Laporan sulit dibuat lengkap Tidak yakin laporan sampai ke instansi yang benar Tidak ada kepastian status | Partisipasi rendah dan kepercayaan pada kanal aduan menurun |
| **Instansi** | Laporan tersebar dan formatnya beragam Salah alamat karena kewenangan ruas berbeda Prioritas sulit ditentukan tanpa konteks frekuensi dan cuaca | Perbaikan reaktif, lubang kambuhan, dan pemborosan sumber daya |
| **Pengelola platform** | Risiko spam, foto palsu, dan laporan duplikat Risiko tuduhan atau data pribadi dalam teks laporan | Reputasi dan hubungan dengan instansi terganggu |

## **2.3 Peluang**

* Teknologi geotagging dan peta membuat laporan berlokasi akurat dengan usaha kecil dari warga.

* AI teks dapat menyeragamkan bahasa laporan bila dibatasi ketat dan diawasi manusia.

* Data cuaca BMKG membuka analisis hujan versus kerusakan yang jarang tersedia bagi instansi tingkat kota. Ketersediaan datanya masih harus diverifikasi (OQ-B07).

* Partisipasi komunitas (+1 dan sign-off) memberi sinyal prioritas dan pengawasan kualitas tanpa biaya inspeksi tambahan.

## **2.4 Kondisi Alternatif yang Sudah Ada**

Warga saat ini dapat memakai kanal aduan nasional dan daerah serta media sosial. Posisi platform ini adalah **pelengkap yang berfokus pada jalan**, dengan laporan terstruktur, routing ke instansi yang tepat, dan pelacakan terbuka. Pemetaan kanal eksisting dan celahnya menjadi pekerjaan Discovery (OQ-B09).

# **3\. Tujuan Bisnis**

*Target adalah **usulan** untuk 3 bulan setelah pilot dimulai dan akan direvisi setelah baseline ditetapkan.*

| ID | Tujuan bisnis | Ukuran keberhasilan | Target usulan |
| :---- | :---- | :---- | :---- |
| **BO-1** | Meningkatkan kemudahan dan kelengkapan laporan warga | Median waktu pengisian, completion rate, kelengkapan laporan | ≤ 3 menit, ≥ 60%, ≥ 90% |
| **BO-2** | Memastikan laporan sampai ke instansi yang tepat dengan kualitas terjaga | Waktu verifikasi Admin, tingkat salah routing | ≤ 2 hari kerja, ≤ 10% |
| **BO-3** | Membangun kepercayaan terhadap pemakaian AI yang aman | Insiden halusinasi lolos, tingkat fallback, tingkat persetujuan draf | 0 kasus, ≤ 5%, ≥ 70% |
| **BO-4** | Mendorong tindak lanjut instansi yang terlihat dan terawasi | Waktu respons instansi, laporan Selesai yang tidak dibuka kembali | Disepakati bersama PUPR |
| **BO-5** | Memberi instansi dasar data untuk prioritisasi berbasis cuaca | Petugas aktif di dashboard, titik kambuhan teridentifikasi | ≥ 3 petugas aktif per minggu |
| **BO-6** | Menjamin akuntabilitas dan kepatuhan operasional | Cakupan audit log, insiden privasi | 100%, 0 insiden |

**Indikator utama (North Star):** jumlah laporan yang mencapai status Selesai dan tidak dibuka kembali selama masa pemantauan.

# **4\. Ruang Lingkup Bisnis**

## **4.1 Dalam Lingkup**

* Pelaporan kerusakan jalan oleh warga di wilayah administratif Kota Bogor, dengan foto, lokasi GPS, dan data terstruktur.

* Bantuan AI terbatas untuk menyusun judul dan deskripsi, dengan fallback template dan persetujuan manusia.

* Moderasi, verifikasi, routing manual, dan eskalasi resmi via email oleh Admin SIJAGO.

* Peta publik laporan terverifikasi, dukungan komunitas (+1, tag kondisi), dan pelacakan status.

* Portal sederhana bagi instansi untuk memperbarui status dan mengunggah bukti sebelum-sesudah.

* Audit trail dan version history untuk akuntabilitas.

* Fase 2: dashboard analitik cuaca BMKG dan Community Sign-Off.

## **4.2 Di Luar Lingkup**

* Pelaksanaan perbaikan fisik jalan dan pengambilan keputusan anggaran perbaikan (tanggung jawab instansi).

* Analisis foto oleh AI, pengukuran kerusakan otomatis, dan penentuan instansi oleh AI.

* Pengiriman otomatis ke instansi tanpa konfirmasi Admin.

* Integrasi dua arah dengan sistem internal pemerintah atau kanal aduan nasional (dapat dipertimbangkan kemudian).

* Aplikasi native, notifikasi WhatsApp, gamifikasi, dan pelaporan di luar Kota Bogor.

## **4.3 Batas Tanggung Jawab**

Platform berperan sebagai **perantara dan pengawas sipil**, bukan pelaksana perbaikan. Platform tidak menjanjikan waktu perbaikan kepada warga dan tidak mewakili instansi pemerintah, kecuali ada kesepakatan resmi (OQ-B02).

# **5\. Pemangku Kepentingan**

## **5.1 Analisis Pemangku Kepentingan**

| Pemangku kepentingan | Kepentingan | Pengaruh | Yang mereka butuhkan | Strategi |
| :---- | :---- | :---- | :---- | :---- |
| **Sponsor bisnis** | Tinggi | Tinggi | Bukti dampak dan risiko terkendali | Laporan kemajuan berkala dan keputusan pada gerbang fase |
| **Dinas PUPR Kota Bogor** | Tinggi | Tinggi | Laporan terkurasi, tidak membanjiri, dan tidak mempermalukan | Libatkan sejak Discovery, sediakan jalur ringan, tampilkan nilai lewat dashboard |
| **Instansi lain (Pemprov Jabar, Balai Nasional)** | Sedang | Tinggi | Laporan yang memang kewenangannya dengan data lokasi jelas | Routing akurat dan mekanisme pengembalian |
| **Warga dan komunitas** | Tinggi | Sedang | Cara lapor yang cepat dan kepastian status | Uji dengan pengguna, komunikasi status yang jujur |
| **Admin SIJAGO** | Tinggi | Sedang | Alat moderasi efisien dan SLA yang realistis | Pelatihan, panduan moderasi, dan kapasitas yang jelas |
| **Tim produk dan engineering** | Tinggi | Sedang | Kebutuhan yang jelas dan stabil | PRD sebagai acuan dan kontrol perubahan |
| **Penegak kepatuhan, hukum, dan privasi** | Sedang | Tinggi | Kepatuhan data pribadi dan kebijakan konten | Tinjau kebijakan sebelum pilot |
| **BMKG (penyedia data)** | Rendah | Sedang | Penggunaan data sesuai ketentuan | Pelajari syarat penggunaan dan kuota |
| **Media dan publik luas** | Sedang | Sedang | Transparansi dan keandalan | Komunikasi hati-hati dan kebijakan konten |

## **5.2 Matriks Peran (RACI) \[Usulan BA\]**

R \= Responsible, A \= Accountable, C \= Consulted, I \= Informed.

| Aktivitas | Sponsor | Product | Admin | Instansi | Hukum/Privasi |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Menetapkan kebijakan konten dan privasi** | A | R | C | I | C |
| **Moderasi dan verifikasi laporan** | I | C | R/A | \- | \- |
| **Routing dan eskalasi ke instansi** | I | C | R/A | I | \- |
| **Menindaklanjuti dan memperbarui status** | I | \- | C | R/A | \- |
| **Mengelola konfigurasi dan ambang sistem** | I | A | C | \- | \- |
| **Audit sampling draf AI dan audit log** | A | R | C | \- | C |
| **Meninjau aturan rekomendasi teknis (Fase 2\)** | I | A | \- | C | \- |

# **6\. Proses Bisnis: Kondisi Saat Ini dan Target**

*Kondisi saat ini adalah **hipotesis** yang perlu dikonfirmasi pada fase Discovery.*

| Tahap | Kondisi saat ini (hipotesis) | Kondisi target |
| :---- | :---- | :---- |
| **1\. Menemukan dan melapor** | Warga melapor lewat kanal berbeda dengan format bebas, sering tanpa lokasi presisi | Form terstruktur, GPS otomatis, foto, dan bantuan merapikan teks yang harus disetujui warga |
| **2\. Penyaringan** | Tidak ada penyaringan seragam, duplikat dan spam menumpuk | Admin memverifikasi keaslian, menolak spam, dan menggabungkan duplikat |
| **3\. Penentuan instansi** | Warga atau petugas menebak instansi, risiko salah alamat | Admin memilih instansi berdasarkan kewenangan, instansi dapat mengembalikan bila salah |
| **4\. Penyampaian** | Tidak seragam dan sulit dilacak | Email resmi berformat baku dikirim setelah konfirmasi Admin, tercatat di audit log |
| **5\. Tindak lanjut** | Status tidak terlihat oleh warga | Instansi memperbarui status dan bukti sebelum-sesudah, warga melacak lewat tiket |
| **6\. Pengawasan kualitas** | Tidak ada umpan balik atas hasil perbaikan | Komunitas menilai kualitas tambalan selama masa pemantauan (Fase 2\) |
| **7\. Perencanaan** | Prioritas berdasarkan laporan masuk tanpa konteks cuaca | Dashboard hujan versus kerusakan dan titik kambuhan untuk instansi (Fase 2\) |

# **7\. Kebutuhan Bisnis**

Prioritas memakai MoSCoW: **Must** wajib untuk pilot, **Should** penting dan menyusul segera, **Could** nilai tambah. Setiap kebutuhan dipetakan ke fitur PRD (F1 sampai F10).

| ID | Kebutuhan bisnis | Alasan dan tujuan terkait | Prioritas | Fitur PRD |
| :---- | :---- | :---- | :---- | :---- |
| **BR-01** | Warga dapat melaporkan kerusakan jalan tanpa hambatan pendaftaran awal | Menurunkan hambatan partisipasi (BO-1) | Must | F1 |
| **BR-02** | Laporan harus memuat lokasi akurat di dalam wilayah Kota Bogor, foto, dan data terstruktur | Instansi dapat menindaklanjuti tanpa survei ulang (BO-1, BO-2) | Must | F1 |
| **BR-03** | Teks laporan dapat dirapikan dengan bantuan AI tanpa menambah atau mengubah fakta | Keseragaman laporan dengan risiko minimal (BO-3) | Must | F2 |
| **BR-04** | Layanan harus tetap menghasilkan draf bila AI gagal, dengan penanda jelas jenis draf | Kontinuitas layanan dan transparansi (BO-3) | Must | F2 |
| **BR-05** | Warga harus meninjau dan menyetujui teks sebelum dikirim, dan persetujuan batal bila fakta berubah | Akuntabilitas isi laporan (BO-3, BO-6) | Must | F3 |
| **BR-06** | Hanya laporan terverifikasi yang tampil publik, dengan peta, filter, dan detail | Transparansi dengan perlindungan dari spam dan fitnah (BO-2) | Must | F4 |
| **BR-07** | Komunitas dapat memberi dukungan (+1) dan tag kondisi pada laporan yang ada | Sinyal prioritas dan pengurangan duplikat (BO-4) | Must | F4 |
| **BR-08** | Admin dapat memverifikasi, menolak, dan menggabungkan duplikat dengan alasan tercatat | Kualitas laporan yang diteruskan (BO-2) | Must | F5 |
| **BR-09** | Admin menentukan instansi tujuan secara manual dan mengirim eskalasi hanya lewat konfirmasi | Kendali manusia atas komunikasi resmi (BO-2, BO-6) | Must | F5 |
| **BR-10** | Instansi dapat memperbarui status, mengembalikan laporan yang bukan kewenangannya, dan mengunggah bukti | Menutup siklus tindak lanjut (BO-4) | Must | F6 |
| **BR-11** | Semua aksi penting tercatat permanen dan dapat ditelusuri, termasuk riwayat versi fakta dan draf | Akuntabilitas hukum dan operasional (BO-6) | Must | F7 |
| **BR-12** | Warga dapat melacak status laporan melalui tiket tanpa login | Menjawab masalah ketidakpastian status (BO-1, BO-4) | Must | F8 |
| **BR-13** | Warga dapat menerima notifikasi perubahan status bila memberi persetujuan | Retensi dan kepercayaan (BO-4) | Should | F8 |
| **BR-14** | Instansi dapat melihat hubungan curah hujan BMKG dengan frekuensi kerusakan | Prioritisasi berbasis data (BO-5) | Should | F9 |
| **BR-15** | Sistem menyorot titik kambuhan di zona hujan ekstrem dengan rekomendasi teknis indikatif | Perbaikan yang lebih tahan lama (BO-5) | Should | F9 |
| **BR-16** | Komunitas dapat menilai kualitas perbaikan selama masa pemantauan, dan laporan dapat dibuka kembali | Pengawasan kualitas hasil (BO-4) | Should | F10 |
| **BR-17** | Platform melindungi data pribadi dan mengikuti ketentuan pelindungan data yang berlaku | Kepatuhan dan kepercayaan (BO-6) | Must | NFR |
| **BR-18** | Admin dan instansi memiliki hak akses sesuai peran dan data antarinstansi terpisah | Keamanan dan tata kelola (BO-6) | Must | F5, F6, F7 |
| **BR-19** | Parameter bisnis (radius duplikat, masa pemantauan, ambang) dapat diubah tanpa pengembangan ulang | Kelincahan operasional (BO-2, BO-4) | Should | F7 (US-20) |
| **BR-20** | Layanan inti tetap berjalan bila layanan pendukung (AI, BMKG, peta) terganggu | Keandalan layanan (BO-1, BO-3) | Must | NFR |
| **BR-21** | Admin dapat mengirim ulang eskalasi dan mengirim pengingat bila instansi tidak merespons | Menjaga laporan tidak berhenti di tengah jalan (BO-4) | Should | F5 |
| **BR-22** | Metrik bisnis dapat diukur dan dilaporkan secara berkala kepada sponsor \[Usulan BA\] | Dasar keputusan gerbang fase (semua BO) | Must | NFR (observabilitas) |

# **8\. Aturan Bisnis**

| ID | Aturan | Dasar |
| :---- | :---- | :---- |
| **BRL-01** | Fakta Asal dari pelapor adalah sumber kebenaran. Draf AI atau Template hanyalah rangkuman teks dan tidak boleh menambah fakta. | Mencegah misinformasi dan menjaga akuntabilitas |
| **BRL-02** | AI tidak boleh menganalisis foto, menentukan instansi tujuan, menilai kebenaran, mengubah ukuran, atau menyatakan status selesai. | Keputusan tetap pada manusia |
| **BRL-03** | Setiap draf wajib berlabel permanen: \[Draf Dibantu AI\] atau \[Draf Template\]. | Transparansi bagi pembaca dan instansi |
| **BRL-04** | Warga wajib menyetujui draf sebelum mengirim. Perubahan Fakta Asal setelah draf dibuat membatalkan persetujuan. | Draf selalu sesuai fakta |
| **BRL-05** | Pemanggilan AI tidak boleh mengirim email atau mengubah status penanganan. | Mencegah efek samping |
| **BRL-06** | Email resmi ke instansi dan perubahan status menjadi Diteruskan hanya terjadi setelah Admin menekan tombol konfirmasi. | Kendali atas komunikasi resmi |
| **BRL-07** | Penentuan instansi tujuan dilakukan Admin berdasarkan kewenangan ruas jalan. | Ketepatan routing |
| **BRL-08** | Admin tidak dapat mengubah Fakta Asal. Koreksi dilakukan lewat catatan Admin. | Integritas data pelapor |
| **BRL-09** | Hanya laporan terverifikasi yang tampil di peta publik (keputusan OQ-B05). | Melindungi dari spam dan fitnah |
| **BRL-10** | Instansi dapat mengembalikan laporan yang bukan kewenangannya dengan alasan, dan Admin melakukan routing ulang. | Mengurangi salah alamat |
| **BRL-11** | Instansi baru dapat menandai Selesai dengan minimal satu foto bukti perbaikan. | Verifikasi hasil |
| **BRL-12** | Rekomendasi teknis pada dashboard berlabel indikatif dan bukan pengganti survei teknis. | Mencegah salah tafsir korelasi sebagai sebab-akibat |
| **BRL-13** | Audit log bersifat append-only dan tidak dapat diubah atau dihapus oleh siapa pun. | Bukti akuntabilitas |
| **BRL-14** | Identitas pelapor tidak diteruskan ke instansi kecuali pelapor menyetujui (keputusan OQ-B06). | Privasi pelapor |

# **9\. Model Operasi dan Tata Kelola**

## **9.1 Model Hibrida**

Warga melapor, Admin SIJAGO memverifikasi dan meneruskan, instansi mengeksekusi. Model ini dipilih karena platform belum tentu terhubung langsung dengan sistem pemerintah, dan peran moderator independen menjaga kualitas serta netralitas.

## **9.2 Operasional Harian \[Usulan BA\]**

| Area | Ketentuan usulan |
| :---- | :---- |
| **Moderasi** | Admin memeriksa antrean pada hari kerja. SLA verifikasi usulan 2 hari kerja (OQ-B04). |
| **Eskalasi** | Email resmi berformat baku, memuat label draf, koordinat, tautan peta, dan foto. Pengingat manual bila tidak ada respons. |
| **Kebijakan konten** | Laporan dengan tuduhan, SARA, atau data pribadi pihak lain ditolak atau diminta diperbaiki. Alasan penolakan baku. |
| **Penanganan keluhan** | Saluran bagi pihak yang laporannya ditolak atau yang merasa dirugikan oleh isi laporan, dengan jawaban dalam waktu yang ditetapkan. |
| **Pengawasan AI** | Audit sampling berkala atas draf AI. Kasus halusinasi masuk set uji regresi. |
| **Pelaporan ke sponsor** | Ringkasan metrik bulanan dan tinjauan pada gerbang fase. |

## **9.3 Kepemilikan Data**

Pemilik data laporan, tata kelola retensi, dan hak akses publik ditetapkan oleh sponsor bersama pihak hukum (OQ-B08). Selama belum ada keputusan, asumsi kerja adalah platform bertindak sebagai pengendali data pelapor dan instansi hanya menerima data yang dibutuhkan untuk menindaklanjuti.

# **10\. Analisis Biaya dan Manfaat**

Dokumen ini belum memuat angka rupiah karena anggaran dan biaya belum ditetapkan. Tabel berikut menyiapkan kerangka agar sponsor dapat melengkapinya. Saya tidak mengisi angka tanpa dasar.

## **10.1 Komponen Biaya**

| Kategori | Contoh komponen | Perkiraan |
| :---- | :---- | :---- |
| **Pengembangan** | Tim produk, desain, engineering, pengujian | \[Diisi\] |
| **Infrastruktur** | Vercel (frontend), Render (backend), Neon PostgreSQL, penyimpanan objek untuk foto, email transaksional, peta dasar dan geocoding | \[Diisi\] |
| **AI** | Biaya per permintaan draf dan kuota harian | \[Diisi, tergantung penyedia dan volume\] |
| **Operasional** | Admin moderator, dukungan, penanganan keluhan | \[Diisi\] |
| **Kepatuhan** | Tinjauan hukum, kebijakan privasi, uji keamanan | \[Diisi\] |
| **Adopsi** | Sosialisasi ke warga, pelatihan petugas instansi | \[Diisi\] |

## **10.2 Komponen Manfaat**

| Manfaat | Jenis | Cara mengukur |
| :---- | :---- | :---- |
| **Waktu pelaporan dan verifikasi lebih singkat** | Efisiensi | Median waktu pengisian dan verifikasi |
| **Laporan tepat sasaran, salah routing berkurang** | Efisiensi instansi | Tingkat pengembalian oleh instansi |
| **Jalan yang diperbaiki lebih cepat atau lebih tahan lama** | Dampak publik | Waktu respons, laporan Selesai tanpa dibuka kembali, titik kambuhan |
| **Kepercayaan publik pada kanal aduan** | Dampak sosial | Survei kepuasan pelapor dan jumlah pelapor berulang |
| **Prioritisasi berbasis data** | Kualitas keputusan | Penggunaan dashboard dan umpan balik petugas |

Sebagian besar manfaat bersifat publik dan sulit diubah menjadi pendapatan. Karena itu model pendanaan dan keberlanjutan operasional adalah keputusan penting sponsor (OQ-B03).

# **11\. Kriteria Keberhasilan dan KPI**

Detail definisi metrik ada di PRD Bagian 2.2. Berikut kriteria untuk keputusan gerbang fase.

| Gerbang | Kriteria untuk lanjut |
| :---- | :---- |
| **Akhir Discovery** | Asumsi kerja berstatus Blocker terkonfirmasi, ada komitmen awal dari instansi (OQ-B02), kebijakan privasi dan konten disiapkan, baseline metrik ditetapkan. |
| **Akhir Pilot** | Alur dari laporan sampai Selesai berjalan minimal satu siklus bersama instansi, tidak ada halusinasi AI yang lolos ke laporan terkirim, tidak ada insiden privasi, SLA verifikasi tercapai. |
| **Akhir Fase 2** | Dashboard dipakai petugas, aturan rekomendasi ditinjau ahli, data BMKG terbukti memadai, hasil sign-off terbaca. |

# **12\. Asumsi, Batasan, dan Dependensi**

## **12.1 Asumsi**

* Ada tim Admin yang bisa memoderasi pada jam kerja.

* Instansi tujuan menyediakan alamat email resmi dan bersedia menerima laporan dari platform.

* Warga memiliki ponsel berkamera dan akses internet seluler.

* Akses data BMKG dengan granularitas yang cukup tersedia untuk Bogor (belum diverifikasi).

* Penyedia AI dapat dipakai dengan kebijakan data yang dapat diterima secara hukum.

## **12.2 Batasan**

* Platform tidak memiliki kewenangan memaksa instansi bertindak.

* Anggaran, tim, dan tenggat belum ditetapkan.

* Pilot dibatasi pada kapasitas moderasi Admin.

* Rekomendasi teknis tidak boleh diposisikan sebagai pengganti survei teknis.

## **12.3 Dependensi**

| Dependensi | Pemilik | Dampak bila terlambat |
| :---- | :---- | :---- |
| **Komitmen dan kontak resmi instansi** | Sponsor | Siklus tindak lanjut tidak terbukti |
| **Kebijakan privasi, syarat layanan, dan kebijakan konten** | Sponsor dan hukum | Pilot tidak dapat dibuka ke publik |
| **Akses dan syarat penggunaan data BMKG** | Product dan engineering | Fase 2 tertunda atau berubah bentuk |
| **Penyedia AI, peta, email, dan penyimpanan** | Engineering | Fitur terkait memakai mode fallback |
| **Daftar ruas jalan dan kewenangannya** | Instansi dan Admin | Salah routing meningkat |
| **Akun dan paket layanan Vercel, Render, Neon, penyimpanan objek, dan email** | Sponsor dan engineering | Pilot berjalan di paket terbatas atau tertunda |

## **12.4 Platform Teknologi**

Pilihan teknologi inti ditetapkan oleh pemilik produk. Rincian lengkap beserta library pendamping ada di PRD Bagian 12\.

| Lapisan | Teknologi | Implikasi bisnis |
| :---- | :---- | :---- |
| **Frontend** | Next.js, di-deploy di Vercel | Biaya berlangganan bertambah seiring trafik. Halaman publik cepat dan ramah pencarian |
| **Backend** | Express.js, di-deploy di Render | Instance gratis dapat tertidur saat sepi sehingga pilot sebaiknya memakai paket berbayar |
| **Database** | Neon PostgreSQL dengan Drizzle ORM | Standar terbuka (PostgreSQL) sehingga risiko ketergantungan pada satu vendor relatif rendah |
| **Penyimpanan foto dan email** | Object storage dan layanan email terpisah (usulan) | Menambah komponen biaya dan akun yang harus dikelola |

Harga dan kuota tiap layanan belum diverifikasi. Angka biaya diisi sponsor pada Bagian 10 setelah paket dipilih.

# **13\. Risiko Bisnis**

| ID | Risiko | Peluang | Dampak | Mitigasi |
| :---- | :---- | :---- | :---- | :---- |
| **BR-R1** | Instansi tidak merespons atau tidak mau memakai portal | Tinggi | Tinggi | Kemitraan sejak awal, jalur ringan, override Admin yang transparan, nilai dashboard |
| **BR-R2** | Ekspektasi publik melebihi kapasitas perbaikan sehingga muncul kekecewaan | Tinggi | Sedang | Komunikasi status yang jujur, tanpa janji waktu perbaikan |
| **BR-R3** | Platform dianggap menyudutkan instansi atau pejabat | Sedang | Tinggi | Bahasa laporan netral, validator AI, moderasi, dan kebijakan konten |
| **BR-R4** | Draf AI memuat fakta yang tidak ada | Sedang | Tinggi | Validator, fallback, persetujuan warga, audit sampling |
| **BR-R5** | Spam, foto palsu, atau laporan fitnah | Sedang | Tinggi | Moderasi sebelum publik, rate limit, pemeriksaan EXIF |
| **BR-R6** | Beban moderasi melebihi kapasitas | Sedang | Sedang | Pilot terbatas, SLA realistis, antrean berprioritas |
| **BR-R7** | Pelanggaran atau kebocoran data pribadi | Rendah | Tinggi | Minimisasi data, enkripsi, kebijakan privasi, tinjauan hukum |
| **BR-R8** | Data BMKG tidak memadai untuk analisis | Sedang | Tinggi | Spike teknis sebelum komitmen Fase 2, rencana sumber alternatif |
| **BR-R9** | Biaya operasional (terutama AI dan moderasi) tidak berkelanjutan | Sedang | Sedang | Kuota, fallback template, model pendanaan jelas (OQ-B03) |
| **BR-R10** | Ketergantungan pada satu atau dua orang Admin | Sedang | Sedang | Panduan moderasi dan pelatihan cadangan |
| **BR-R11** | Keterbatasan paket hosting (layanan tertidur saat sepi, kuota terbatas) mengganggu pengalaman pilot | Sedang | Sedang | Pakai paket berbayar minimal selama pilot, uji beban, dan pantau uptime |

# **14\. Kepatuhan dan Aspek Hukum**

| Penting Bagian ini adalah daftar hal yang perlu ditinjau, bukan nasihat hukum. Penafsiran dan kewajiban spesifik perlu dikonfirmasi oleh penasihat hukum. |
| :---- |

| Topik | Isu bisnis | Tindakan |
| :---- | :---- | :---- |
| **Pelindungan data pribadi (UU No. 27 Tahun 2022\)** | Foto, lokasi, dan kontak dapat memuat data pribadi. Transfer data ke penyedia AI atau layanan luar negeri | Minimisasi data, persetujuan eksplisit, kebijakan privasi, tinjauan transfer data (OQ-B08) |
| **Pelayanan publik (UU No. 25 Tahun 2009\)** | Hubungan platform dengan penyelenggara layanan publik dan ekspektasi respons | Konfirmasi posisi platform dan bentuk kerja sama (OQ-B02) |
| **Konten dan pencemaran nama baik** | Laporan dapat memuat tuduhan terhadap pihak tertentu | Kebijakan konten, moderasi, bahasa netral, saluran keluhan |
| **Hak atas foto dan privasi pihak ketiga** | Foto bisa memuat wajah dan pelat nomor | Blur dan prosedur penolakan (PRD OQ-11) |
| **Retensi dan hak subjek data** | Hak hapus vs audit log yang tidak dapat diubah | Pseudonimisasi dan retensi yang ditetapkan hukum (PRD OQ-24) |
| **Syarat penggunaan data pihak ketiga** | Data BMKG, peta, dan layanan AI memiliki lisensi dan kuota | Tinjau syarat sebelum dipakai (OQ-B07) |

# **15\. Implementasi, Adopsi, dan Matriks Keterlacakan**

## **15.1 Pendekatan Bertahap \[Usulan BA\]**

| Tahap | Tujuan bisnis | Isi |
| :---- | :---- | :---- |
| **Fase 0: Discovery** | Memvalidasi masalah dan kemitraan | Wawancara, audit kanal, jawaban pertanyaan terbuka, spike BMKG dan AI, kebijakan privasi dan konten |
| **Fase 1: Pilot MVP Core** | Membuktikan alur laporan sampai Selesai | F1 sampai F8 pada wilayah terbatas dengan kapasitas Admin yang jelas |
| **Fase 2: MVP+** | Menambah nilai analitik dan pengawasan | F9 dan F10, perluasan wilayah |

## **15.2 Adopsi dan Manajemen Perubahan**

* Libatkan petugas PUPR sejak Discovery agar platform dirasakan sebagai alat bantu, bukan sorotan.

* Siapkan panduan singkat dan sesi pelatihan untuk petugas dan Admin.

* Komunikasikan kepada warga apa yang dapat dan tidak dapat dijanjikan platform.

* Mulai dari wilayah pilot dan kumpulkan umpan balik sebelum meluas.

* Tentukan pemilik komunikasi publik dan penanganan isu bila ada laporan viral atau sengketa.

## **15.3 Matriks Keterlacakan Tujuan Bisnis ke Kebutuhan dan Fitur**

| Tujuan bisnis | Kebutuhan bisnis | Fitur PRD |
| :---- | :---- | :---- |
| **BO-1 Kemudahan dan kelengkapan laporan** | BR-01, BR-02, BR-12, BR-13, BR-20 | F1, F8 |
| **BO-2 Laporan tepat sasaran dan berkualitas** | BR-06, BR-08, BR-09, BR-19, BR-21 | F4, F5 |
| **BO-3 Kepercayaan pada AI yang aman** | BR-03, BR-04, BR-05 | F2, F3 |
| **BO-4 Tindak lanjut terlihat dan terawasi** | BR-07, BR-10, BR-12, BR-16, BR-21 | F4, F6, F8, F10 |
| **BO-5 Prioritisasi berbasis cuaca** | BR-14, BR-15 | F9 |
| **BO-6 Akuntabilitas dan kepatuhan** | BR-05, BR-09, BR-11, BR-17, BR-18, BR-22 | F3, F5, F6, F7 |

# **16\. Keputusan, Asumsi Kerja, dan Persetujuan**

Pertanyaan teknis dan fungsional ada di PRD Bagian 10\. Pertanyaan **tingkat bisnis** berikut kini memakai **rekomendasi default PM sebagai keputusan**. Status **Diputuskan** berarti berlaku sekarang. Status **Asumsi kerja** berarti bergantung pada fakta di luar kendali tim produk (kemitraan, pendanaan, kapasitas, hukum, akses data) sehingga dipakai sementara dan perlu dikonfirmasi sponsor.

| No | Pertanyaan | Keputusan (rekomendasi PM) | Prioritas | Status |
| :---- | :---- | :---- | :---- | :---- |
| **OQ-B01** | Siapa sponsor, pemilik operasional, dan badan hukum atau organisasi di balik SIJAGO? | Sponsor menunjuk pemilik dan menyiapkan kebijakan dasar sebelum pilot. | Blocker | Asumsi kerja |
| **OQ-B02** | Apakah sudah ada komitmen atau MoU dengan Dinas PUPR Kota Bogor dan apa bentuk kerja samanya? | Mulai tanpa MoU lewat email eskalasi sambil menjajaki kerja sama formal. | Blocker | Asumsi kerja |
| **OQ-B03** | Apa sumber pendanaan dan model keberlanjutan setelah pilot? | Pilot berskala kecil dengan biaya per laporan dihitung dahulu. | Blocker | Asumsi kerja |
| **OQ-B04** | Berapa kapasitas Admin dan SLA verifikasi yang dapat dijamin? | SLA 2 hari kerja dan wilayah pilot disesuaikan kapasitas. | High | Asumsi kerja |
| **OQ-B05** | Apakah laporan yang belum diverifikasi boleh tampil publik? | Tidak, hanya yang terverifikasi. | High | Diputuskan |
| **OQ-B06** | Apakah identitas pelapor boleh diteruskan ke instansi? | Tidak, kecuali pelapor setuju. | High | Diputuskan |
| **OQ-B07** | Apakah akses dan lisensi data BMKG sudah dipastikan untuk penggunaan ini? | Lakukan spike dan tinjau syarat sebelum komitmen Fase 2\. | High | Asumsi kerja |
| **OQ-B08** | Siapa pemilik data dan bagaimana kebijakan retensi serta transfer data ke penyedia pihak ketiga? | Konsultasi hukum, minimisasi data, dan pseudonimisasi. | High | Asumsi kerja |
| **OQ-B09** | Adakah data aduan jalan rusak dari kanal eksisting untuk baseline? | Kumpulkan pada Fase 0\. | Medium | Asumsi kerja |
| **OQ-B10** | Apakah ini produk yang akan diluncurkan nyata atau proyek akademik atau prototipe? | Susun untuk peluncuran nyata dan tandai bagian yang cukup berupa prototipe. | High | Asumsi kerja |

## **Persetujuan**

| Peran | Nama | Tanda tangan | Tanggal |
| :---- | :---- | :---- | :---- |
| **Sponsor bisnis** |  |  |  |
| **Perwakilan Dinas PUPR (jika terlibat)** |  |  |  |
| **Product Manager / Business Analyst** |  |  |  |
| **Pemimpin Engineering** |  |  |  |
| **Penasihat hukum / privasi** |  |  |  |

