**BUSINESS REQUIREMENTS DOCUMENT**

**SIJAGO**

**Sistem Jalan Bogor**

Platform Pelaporan Kerusakan Jalan Kota Bogor  \|  Civic-Tech & Weather-Road Intelligence

| Atribut | Keterangan |
| :---- | :---- |
| **Versi dokumen** | 1.0 (Final — Disesuaikan untuk Kompetisi MIA2026) |
| **Tanggal** | 25 Oktober 2026 |
| **Status** | Final (v1.0). Seluruh keputusan bisnis, ruang lingkup MVP, dan arsitektur telah dikunci untuk eksekusi pengembangan proyek kompetisi MIA2026. |
| **Dokumen pendamping** | PRD SIJAGO v1.0 (detail fungsional, data model, edge case, dan tech stack) |
| **Pemilik dokumen** | **Sabira** (Product Manager & UI/UX Designer)<br>**Boby** (System Architect & Fullstack Developer) |
| **Konteks proyek** | Proyek Kompetisi **MIA2026** (Independen, tanpa afiliasi organisasi/sponsor bisnis eksternal) |

| Cara membaca dokumen ini: BRD menjelaskan mengapa produk dibuat dan apa kebutuhan bisnisnya. Detail bagaimana sistem berperilaku ada di PRD. Setiap kebutuhan bisnis (BR) dipetakan ke fitur PRD pada Bagian 15. Dalam konteks kompetisi MIA2026, kemitraan instansi disimulasikan secara end-to-end dan seluruh infrastruktur berjalan pada layanan Free Tier. |
| :---- |

# **Riwayat Dokumen & Daftar Isi**

| Versi | Tanggal | Perubahan |
| :---- | :---- | :---- |
| 0.1 | 25 Okt 2026 | Draft awal BRD dari Product Summary & Scope. |
| 0.2 | 25 Okt 2026 | Rekomendasi default PM diterapkan sebagai keputusan dan asumsi kerja (Bagian 16). |
| 0.3 | 25 Okt 2026 | Nama produk SIJAGO ditetapkan. Platform teknologi ditambahkan (Bagian 12.4). |
| 1.0 | 25 Okt 2026 | Dokumen dikunci untuk kompetisi MIA2026. Pemilik dokumen ditetapkan (Sabira & Boby). F9 (Weather-Road Intelligence) dan F10 (Community Sign-Off) ditarik ke Fase 1 (MVP Utama). Simulasi kemitraan instansi, penggunaan layanan Free Tier, serta spesifikasi teknologi (OpenAI `gpt-6-luna`, Cloudinary, Resend, MapLibre + OSM, Multirepo) ditetapkan. |

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

16\. Keputusan Kunci dan Pengesahan Dokumen

# **1\. Ringkasan Eksekutif**

**SIJAGO (Sistem Jalan Bogor)** adalah layanan web **mobile-first** yang memudahkan warga Kota Bogor melaporkan kerusakan jalan, membantu pengelola platform memverifikasi dan meneruskan laporan ke instansi yang berwenang, dan memberi instansi (terutama Dinas PUPR) data untuk memprioritaskan perbaikan, termasuk hubungan antara curah hujan dengan titik kerusakan jalan.

Nilai bisnis utamanya ada pada tiga hal. Pertama, laporan warga menjadi lebih lengkap, terstruktur, dan sampai ke instansi yang benar. Kedua, tindak lanjut menjadi terlihat dan bisa diawasi komunitas melalui fitur *Community Sign-Off*. Ketiga, instansi mendapat dasar data *Weather-Road Intelligence* untuk beralih dari perbaikan reaktif ke perbaikan yang lebih tahan lama di zona hujan ekstrem.

Pembeda utama produk adalah **model hibrida dengan kendali manusia (*Human-in-the-Loop*)**. AI (`openai gpt-6-luna`) hanya membantu merapikan teks laporan dan tidak mengambil keputusan. Pengiriman resmi ke instansi hanya terjadi setelah Admin mengonfirmasi, dan seluruh aksi penting tercatat dalam audit log yang tidak dapat diubah (*append-only*). Pendekatan ini dipilih agar platform dapat dipercaya oleh warga dan oleh instansi.

### **Konteks Eksekusi Proyek (MIA2026)**

* Dikembangkan sebagai karya kompetisi **MIA2026** oleh tim pengembang independen (**Sabira** sebagai PM & UI/UX Designer, **Boby** sebagai System Architect & Fullstack Developer).
* Seluruh fitur utama (**F1 hingga F10**, termasuk *Weather-Road Intelligence* dan *Community Sign-Off*) dimasukkan ke dalam **Fase 1 (MVP Utama)** agar diferensiasi produk terlihat utuh saat demonstrasi dan evaluasi.
* Kemitraan dengan Dinas PUPR Kota Bogor dan instansi terkait **disimulasikan secara penuh** di dalam sistem (mulai dari eskalasi email via Resend, portal instansi, hingga penutupan laporan).
* Infrastruktur dibangun di atas layanan **Free Tier** dengan arsitektur **Multirepo** (repositori Frontend dan Backend terpisah).

# **2\. Latar Belakang Bisnis dan Peluang**

## **2.1 Konteks**

Kota Bogor dikenal sebagai Kota Hujan. Intensitas hujan yang tinggi mempercepat kerusakan permukaan jalan, terutama pada titik dengan drainase buruk. Warga menjadi pihak pertama yang mengetahui kerusakan, sedangkan perbaikan menjadi tanggung jawab instansi yang berbeda tergantung status ruas jalan (kota, provinsi, atau nasional).

## **2.2 Masalah Bisnis**

| Pihak | Masalah | Konsekuensi bisnis |
| :---- | :---- | :---- |
| **Warga** | Laporan sulit dibuat lengkap, tidak yakin laporan sampai ke instansi yang benar, dan tidak ada kepastian status | Partisipasi rendah dan kepercayaan pada kanal aduan menurun |
| **Instansi** | Laporan tersebar dan formatnya beragam, salah alamat karena kewenangan ruas berbeda, serta prioritas sulit ditentukan tanpa konteks frekuensi dan cuaca | Perbaikan reaktif, lubang kambuhan, dan pemborosan sumber daya |
| **Pengelola platform** | Risiko spam, foto palsu, dan laporan duplikat, serta risiko tuduhan atau data pribadi dalam teks laporan | Reputasi dan hubungan dengan instansi terganggu |

## **2.3 Peluang**

* Teknologi geotagging dan peta interaktif (MapLibre GL JS + OpenStreetMap) membuat laporan berlokasi akurat dengan usaha kecil dari warga.
* AI teks (`openai gpt-6-luna` dengan *reasoning medium*) dapat menyeragamkan bahasa laporan bila dibatasi ketat dan diawasi manusia.
* Integrasi data cuaca terbuka (BMKG, Open-Meteo, NASA POWER) membuka analisis hujan versus kerusakan yang jarang tersedia bagi instansi tingkat kota.
* Partisipasi komunitas (+1, tag kondisi, dan *Community Sign-Off*) memberi sinyal prioritas dan pengawasan kualitas tanpa biaya inspeksi tambahan.

## **2.4 Kondisi Alternatif yang Sudah Ada**

Warga saat ini dapat memakai kanal aduan nasional dan daerah serta media sosial. Posisi SIJAGO adalah **pelengkap yang berfokus khusus pada infrastruktur jalan**, dengan laporan terstruktur, routing ke instansi yang tepat, analitik korelasi cuaca, dan pelacakan terbuka.

# **3\. Tujuan Bisnis**

*Target adalah tolok ukur keberhasilan untuk evaluasi produk dan simulasi pilot.*

| ID | Tujuan bisnis | Ukuran keberhasilan | Target |
| :---- | :---- | :---- | :---- |
| **BO-1** | Meningkatkan kemudahan dan kelengkapan laporan warga | Median waktu pengisian, completion rate, kelengkapan laporan | ≤ 3 menit, ≥ 60%, ≥ 90% |
| **BO-2** | Memastikan laporan sampai ke instansi yang tepat dengan kualitas terjaga | Waktu verifikasi Admin, tingkat salah routing | ≤ 2 hari kerja, ≤ 10% |
| **BO-3** | Membangun kepercayaan terhadap pemakaian AI yang aman | Insiden halusinasi lolos, tingkat fallback, tingkat persetujuan draf | 0 kasus, ≤ 5%, ≥ 70% |
| **BO-4** | Mendorong tindak lanjut instansi yang terlihat dan terawasi komunitas | Waktu respons simulasi instansi, laporan Selesai yang terverifikasi lewat *Community Sign-Off* | Siklus penuh tercapai dari lapor hingga sign-off |
| **BO-5** | Memberi instansi dasar data untuk prioritisasi berbasis cuaca | Ketersediaan visualisasi *Weather-Road Intelligence*, titik kambuhan teridentifikasi | Layer cuaca & zona merah berfungsi penuh |
| **BO-6** | Menjamin akuntabilitas dan kepatuhan operasional | Cakupan audit log, insiden privasi | 100%, 0 insiden |

**Indikator utama (North Star):** jumlah laporan yang mencapai status Selesai dan tidak dibuka kembali selama masa pemantauan komunitas.

# **4\. Ruang Lingkup Bisnis**

## **4.1 Dalam Lingkup (MVP Utama / Fase 1)**

* Pelaporan kerusakan jalan oleh warga di wilayah administratif Kota Bogor, dengan foto (diunggah ke Cloudinary), lokasi GPS, dan data terstruktur.
* Bantuan AI terbatas (`openai gpt-6-luna`, *reasoning medium*) untuk menyusun judul dan deskripsi, dengan fallback template deterministik dan persetujuan manusia.
* Moderasi, verifikasi, routing manual, dan eskalasi resmi via email (Resend) oleh Admin SIJAGO.
* Peta publik laporan terverifikasi (MapLibre GL JS + OpenStreetMap), dukungan komunitas (+1, tag kondisi), dan pelacakan status tanpa login.
* Portal sederhana bagi instansi (disimulasikan untuk Dinas PUPR/Pemprov/Balai Nasional) untuk memperbarui status dan mengunggah bukti sebelum-sesudah.
* Audit trail (*append-only*) dan *version history* untuk akuntabilitas.
* **Weather-Road Intelligence (F9):** Dashboard analitik korelasi curah hujan dengan frekuensi kerusakan jalan dan deteksi titik kambuhan.
* **Community Sign-Off (F10):** Penilaian kualitas hasil perbaikan oleh warga selama masa pemantauan setelah laporan ditandai Selesai.

## **4.2 Di Luar Lingkup**

* Pelaksanaan perbaikan fisik jalan dan pengambilan keputusan anggaran perbaikan (tanggung jawab instansi).
* Analisis foto oleh AI, pengukuran kerusakan otomatis, dan penentuan instansi oleh AI.
* Pengiriman otomatis ke instansi tanpa konfirmasi Admin.
* Integrasi dua arah langsung dengan sistem internal pemerintah atau kanal aduan nasional (kemitraan disimulasikan pada tahap kompetisi).
* Aplikasi native iOS/Android, notifikasi WhatsApp, gamifikasi, dan pelaporan di luar Kota Bogor.

## **4.3 Batas Tanggung Jawab**

Platform berperan sebagai **perantara dan pengawas sipil**, bukan pelaksana perbaikan. Platform tidak menjanjikan waktu perbaikan kepada warga dan dalam konteks kompetisi MIA2026 menyimulasikan alur kolaborasi dengan instansi pemerintah.

# **5\. Pemangku Kepentingan**

## **5.1 Analisis Pemangku Kepentingan**

| Pemangku kepentingan | Kepentingan | Pengaruh | Yang mereka butuhkan | Strategi |
| :---- | :---- | :---- | :---- | :---- |
| **Tim Inti SIJAGO (Sabira & Boby)** | Tinggi | Tinggi | Kebutuhan produk dan arsitektur yang jelas, eksekusi MVP tepat waktu untuk MIA2026 | Menyelaraskan desain UI/UX, PRD, dan implementasi multirepo |
| **Dewan Juri & Evaluator MIA2026** | Tinggi | Tinggi | Bukti inovasi, keandalan alur *end-to-end*, keamanan AI, dan dampak nyata solusi | Menampilkan demo utuh F1–F10 dengan data realistis Kota Bogor |
| **Dinas PUPR Kota Bogor (Mitra Simulasi)** | Tinggi | Tinggi | Laporan terkurasi, tidak membanjiri, dan dilengkapi analitik cuaca | Menyediakan simulasi Portal Instansi dan Dashboard Weather-Road Intelligence |
| **Instansi lain (Pemprov Jabar, Balai Nasional — Simulasi)** | Sedang | Tinggi | Laporan yang sesuai kewenangannya dengan data lokasi jelas | Menyediakan fitur routing akurat dan mekanisme pengembalian laporan |
| **Warga dan komunitas** | Tinggi | Sedang | Cara lapor yang cepat dan kepastian status | Antarmuka mobile-first tanpa paksaan login di awal |
| **Admin SIJAGO** | Tinggi | Sedang | Alat moderasi efisien dan kendali penuh atas eskalasi | Menyediakan antrean moderasi, perbandingan EXIF, dan pratinjau email |

## **5.2 Matriks Peran (RACI)**

R = Responsible, A = Accountable, C = Consulted, I = Informed.

| Aktivitas | Sabira (PM & UI/UX) | Boby (System Architect & Fullstack) | Admin (Peran Sistem) | Instansi (Simulasi) |
| :---- | :---- | :---- | :---- | :---- |
| **Menetapkan kebutuhan produk, desain UI/UX, dan kebijakan konten** | R/A | C | I | I |
| **Merancang arsitektur sistem, database, dan integrasi API** | C | R/A | I | I |
| **Moderasi, verifikasi laporan, routing, dan eskalasi** | C | I | R/A | I |
| **Menindaklanjuti laporan dan memperbarui bukti perbaikan** | I | I | C | R/A |
| **Mengelola konfigurasi dan ambang sistem** | A | R | C | \- |
| **Audit sampling draf AI dan audit log** | R/A | R | C | \- |

# **6\. Proses Bisnis: Kondisi Saat Ini dan Target**

| Tahap | Kondisi saat ini (masalah umum) | Kondisi target di SIJAGO |
| :---- | :---- | :---- |
| **1\. Menemukan dan melapor** | Warga melapor lewat kanal berbeda dengan format bebas, sering tanpa lokasi presisi | Form terstruktur, GPS otomatis, foto, dan bantuan AI (`gpt-6-luna`) merapikan teks yang harus disetujui warga |
| **2\. Penyaringan** | Tidak ada penyaringan seragam, duplikat dan spam menumpuk | Admin memverifikasi keaslian, menolak spam, dan menggabungkan duplikat |
| **3\. Penentuan instansi** | Warga atau petugas menebak instansi, risiko salah alamat | Admin memilih instansi berdasarkan kewenangan, instansi dapat mengembalikan bila salah |
| **4\. Penyampaian** | Tidak seragam dan sulit dilacak | Email resmi berformat baku dikirim via Resend setelah konfirmasi Admin, tercatat di audit log |
| **5\. Tindak lanjut** | Status tidak terlihat oleh warga | Instansi memperbarui status dan bukti sebelum-sesudah di portal, warga melacak lewat kode tiket |
| **6\. Pengawasan kualitas** | Tidak ada umpan balik atas hasil perbaikan | Komunitas menilai kualitas tambalan ("Mulus" / "Cepat Rusak") selama masa pemantauan (*Community Sign-Off*) |
| **7\. Perencanaan** | Prioritas berdasarkan laporan masuk tanpa konteks cuaca | Dashboard hujan versus kerusakan dan titik kambuhan untuk instansi (*Weather-Road Intelligence*) |

# **7\. Kebutuhan Bisnis**

Seluruh fitur utama (F1 sampai F10) masuk dalam rilis **Fase 1 (MVP Utama)** untuk kompetisi MIA2026. Prioritas memakai MoSCoW: **Must** wajib berfungsi penuh pada MVP, **Should** mendukung pengalaman lengkap.

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
| **BR-14** | Instansi dapat melihat hubungan curah hujan dengan frekuensi kerusakan jalan | Prioritisasi berbasis data (BO-5) | Must | F9 |
| **BR-15** | Sistem menyorot titik kambuhan di zona hujan ekstrem dengan rekomendasi teknis indikatif | Perbaikan yang lebih tahan lama (BO-5) | Must | F9 |
| **BR-16** | Komunitas dapat menilai kualitas perbaikan selama masa pemantauan, dan laporan dapat dibuka kembali | Pengawasan kualitas hasil (BO-4) | Must | F10 |
| **BR-17** | Platform melindungi data pribadi dan mengikuti ketentuan pelindungan data yang berlaku | Kepatuhan dan kepercayaan (BO-6) | Must | NFR |
| **BR-18** | Admin dan instansi memiliki hak akses sesuai peran dan data antarinstansi terpisah | Keamanan dan tata kelola (BO-6) | Must | F5, F6, F7 |
| **BR-19** | Parameter bisnis (radius duplikat, masa pemantauan, ambang) dapat diubah tanpa pengembangan ulang | Kelincahan operasional (BO-2, BO-4) | Should | F7 (US-20) |
| **BR-20** | Layanan inti tetap berjalan bila layanan pendukung (AI, cuaca, peta) terganggu | Keandalan layanan (BO-1, BO-3) | Must | NFR |
| **BR-21** | Admin dapat mengirim ulang eskalasi dan mengirim pengingat bila instansi tidak merespons | Menjaga laporan tidak berhenti di tengah jalan (BO-4) | Should | F5 |
| **BR-22** | Metrik bisnis dapat diukur dan dipantau melalui sistem | Evaluasi dampak produk (semua BO) | Must | NFR (observabilitas) |

# **8\. Aturan Bisnis**

| ID | Aturan | Dasar |
| :---- | :---- | :---- |
| **BRL-01** | Fakta Asal dari pelapor adalah sumber kebenaran. Draf AI atau Template hanyalah rangkuman teks dan tidak boleh menambah fakta. | Mencegah misinformasi dan menjaga akuntabilitas |
| **BRL-02** | AI tidak boleh menganalisis foto, menentukan instansi tujuan, menilai kebenaran, mengubah ukuran, atau menyatakan status selesai. | Keputusan tetap pada manusia |
| **BRL-03** | Setiap draf wajib berlabel permanen: `[Draf Dibantu AI]` atau `[Draf Template]`. | Transparansi bagi pembaca dan instansi |
| **BRL-04** | Warga wajib menyetujui draf sebelum mengirim. Perubahan Fakta Asal setelah draf dibuat membatalkan persetujuan. | Draf selalu sesuai fakta |
| **BRL-05** | Pemanggilan AI tidak boleh mengirim email atau mengubah status penanganan. | Mencegah efek samping |
| **BRL-06** | Email resmi ke instansi dan perubahan status menjadi Diteruskan hanya terjadi setelah Admin menekan tombol konfirmasi. | Kendali atas komunikasi resmi |
| **BRL-07** | Penentuan instansi tujuan dilakukan Admin berdasarkan kewenangan ruas jalan. | Ketepatan routing |
| **BRL-08** | Admin tidak dapat mengubah Fakta Asal. Koreksi dilakukan lewat catatan Admin. | Integritas data pelapor |
| **BRL-09** | Hanya laporan terverifikasi yang tampil di peta publik. | Melindungi dari spam dan fitnah |
| **BRL-10** | Instansi dapat mengembalikan laporan yang bukan kewenangannya dengan alasan, dan Admin melakukan routing ulang. | Mengurangi salah alamat |
| **BRL-11** | Instansi baru dapat menandai Selesai dengan minimal satu foto bukti perbaikan. | Verifikasi hasil |
| **BRL-12** | Rekomendasi teknis pada dashboard berlabel indikatif dan bukan pengganti survei teknis. | Mencegah salah tafsir korelasi sebagai sebab-akibat |
| **BRL-13** | Audit log bersifat append-only dan tidak dapat diubah atau dihapus oleh siapa pun. | Bukti akuntabilitas |
| **BRL-14** | Identitas pelapor tidak diteruskan ke instansi kecuali pelapor menyetujui. | Privasi pelapor |

# **9\. Model Operasi dan Tata Kelola**

## **9.1 Model Hibrida (Simulasi End-to-End)**

Warga melapor, Admin SIJAGO memverifikasi dan meneruskan, instansi mengeksekusi. Untuk keperluan kompetisi MIA2026, peran Warga, Admin SIJAGO, dan Petugas Instansi (PUPR Kota Bogor, Pemprov Jabar, Balai Nasional) disimulasikan secara penuh di dalam aplikasi sehingga siklus hidup laporan dapat didemonstrasikan dari awal hingga akhir.

## **9.2 Operasional Sistem**

| Area | Ketentuan |
| :---- | :---- |
| **Moderasi** | Admin memeriksa antrean moderasi. Target SLA verifikasi adalah 2 hari kerja. |
| **Eskalasi** | Email resmi berformat baku dikirim melalui Resend, memuat label draf, koordinat, tautan peta, dan foto. |
| **Kebijakan konten** | Laporan dengan tuduhan, SARA, atau data pribadi pihak lain ditolak dengan alasan penolakan baku. |
| **Pengawasan AI** | Validator deterministik pascagenerasi dan audit sampling atas draf AI (`openai gpt-6-luna`). |

## **9.3 Kepemilikan Data**

Platform bertindak sebagai pengendali data pelapor dengan prinsip minimisasi data dan pseudonimisasi. Instansi hanya menerima informasi yang dibutuhkan untuk menindaklanjuti perbaikan jalan (tanpa identitas pribadi pelapor kecuali diizinkan).

# **10\. Analisis Biaya dan Manfaat**

## **10.1 Kebijakan Biaya Infrastruktur (Free Tier)**

Karena SIJAGO dikembangkan sebagai proyek kompetisi **MIA2026**, proyek ini dirancang untuk berjalan sepenuhnya menggunakan **layanan Free Tier (Rp 0 untuk infrastruktur hosting dan layanan pendukung)**:

* **Frontend Hosting:** Vercel (Hobby / Free Tier).
* **Backend Hosting:** Render (Free Web Service).
* **Database:** Neon PostgreSQL + PostGIS (Free Tier).
* **Penyimpanan Foto:** Cloudinary (Free Tier).
* **Email Transaksional:** Resend (Free Tier).
* **Peta & Geocoding:** MapLibre GL JS + OpenStreetMap & Nominatim (Gratis & Terbuka).
* **Data Cuaca:** API Terbuka Gratis (BMKG, Open-Meteo, NASA POWER).

## **10.2 Komponen Manfaat**

| Manfaat | Jenis | Cara mengukur |
| :---- | :---- | :---- |
| **Waktu pelaporan dan verifikasi lebih singkat** | Efisiensi | Median waktu pengisian dan verifikasi |
| **Laporan tepat sasaran, salah routing berkurang** | Efisiensi instansi | Tingkat pengembalian oleh instansi |
| **Jalan yang diperbaiki lebih cepat atau lebih tahan lama** | Dampak publik | Waktu respons, laporan Selesai tanpa dibuka kembali, titik kambuhan |
| **Kepercayaan publik pada kanal aduan** | Dampak sosial | Transparansi tiket pelacakan dan partisipasi *Community Sign-Off* |
| **Prioritisasi berbasis data cuaca** | Kualitas keputusan | Penggunaan dashboard *Weather-Road Intelligence* |

# **11\. Kriteria Keberhasilan dan KPI**

Detail definisi metrik ada di PRD Bagian 2.2. Berikut kriteria keberhasilan untuk pengiriman dan demonstrasi proyek MIA2026:

| Tahap | Kriteria keberhasilan |
| :---- | :---- |
| **Kesiapan Dokumen & Desain** | BRD v1.0 dan PRD v1.0 terkunci, desain UI/UX selesai oleh Sabira, skema database & API siap oleh Boby. |
| **Kesiapan MVP Utama (Fase 1 — F1 s/d F10)** | Seluruh alur dari pelaporan warga (F1–F3), peta publik (F4), moderasi & eskalasi Admin (F5), portal instansi (F6), audit log (F7), pelacakan tiket (F8), analitik cuaca (F9), hingga *Community Sign-Off* (F10) berfungsi ujung-ke-ujung tanpa halusinasi AI yang lolos. |

# **12\. Asumsi, Batasan, dan Dependensi**

## **12.1 Asumsi**

* Kemitraan dengan Dinas PUPR Kota Bogor dan instansi lain disimulasikan melalui akun peran Petugas Instansi dan email pengujian untuk kebutuhan demonstrasi MIA2026.
* Warga memiliki ponsel berkamera dan akses internet seluler.
* API cuaca gratis (BMKG, Open-Meteo, dll.) dan tile peta OpenStreetMap dapat diakses sesuai kebijakan penggunaan wajar (*fair use*).

## **12.2 Batasan**

* Berjalan pada kuota dan batasan layanan Free Tier (misalnya potensi *cold start* pada Render Free Tier dan batas kuota harian Resend/Cloudinary).
* Platform tidak memiliki kewenangan memaksa instansi bertindak secara hukum.
* Rekomendasi teknis pada dashboard cuaca diposisikan sebagai informasi indikatif, bukan pengganti survei teknis lapangan.

## **12.3 Dependensi**

| Dependensi | Pemilik | Dampak bila bermasalah |
| :---- | :---- | :---- |
| **Desain UI/UX & Spesifikasi Produk** | Sabira | Pengembangan frontend tertunda |
| **Arsitektur Backend, Database & Integrasi API** | Boby | Fitur inti tidak terhubung |
| **API OpenAI (`gpt-6-luna`), Cloudinary, Resend** | Boby | Sistem otomatis menggunakan mekanisme fallback (Draf Template) |
| **API Cuaca Gratis (BMKG / Open-Meteo)** | Boby | Dashboard menampilkan snapshot data terakhir atau data seed Bogor |

## **12.4 Platform Teknologi**

Rincian lengkap arsitektur dan *tech stack* terdapat pada PRD Bagian 12.

| Lapisan | Teknologi | Catatan |
| :---- | :---- | :---- |
| **Manajemen Kode** | Multirepo (Repositori Frontend & Backend terpisah) | Memudahkan pemisahan deployment ke Vercel dan Render |
| **Frontend** | Next.js (App Router), di-deploy di Vercel (Free Tier) | Antarmuka mobile-first, cepat, dan responsif |
| **Backend** | Express.js, di-deploy di Render (Free Tier) | Menangani seluruh logika bisnis, validasi hash, dan audit log |
| **Database** | Neon PostgreSQL + PostGIS dengan Drizzle ORM (Free Tier) | Menyimpan data relasional, geospasial, dan audit log append-only |
| **Model AI** | OpenAI `gpt-6-luna` (*reasoning: medium*) | Merapikan judul dan deskripsi laporan dengan kendali validator |
| **Peta & Geocoding** | MapLibre GL JS + OpenStreetMap (Nominatim) | Peta interaktif dan reverse geocoding gratis |
| **Penyimpanan Foto & Email** | Cloudinary (Foto) & Resend (Email Transaksional) | Mengelola varian foto tanpa EXIF dan pengiriman email eskalasi |

# **13\. Risiko Bisnis**

| ID | Risiko | Peluang | Dampak | Mitigasi |
| :---- | :---- | :---- | :---- | :---- |
| **BR-R1** | Keterbatasan layanan Free Tier (Render *cold start*, kuota rate-limit OSM/Resend) saat penjurian | Sedang | Tinggi | Lakukan *warm-up* server sebelum demo, tambahkan cache pada query cuaca/geocoding, dan siapkan fallback |
| **BR-R2** | Draf AI memuat fakta yang tidak ada di input | Sedang | Tinggi | Validator deterministik pascagenerasi, fallback template, sanding wajib, dan persetujuan warga |
| **BR-R3** | Spam, foto palsu, atau laporan fitnah | Sedang | Tinggi | Moderasi sebelum publik, rate limit, pemeriksaan EXIF, dan kebijakan konten |
| **BR-R4** | API cuaca eksternal sedang gangguan atau lambat | Sedang | Sedang | Simpan snapshot cuaca secara lokal di database dan gunakan sumber cadangan dari daftar API gratis |
| **BR-R5** | Pelanggaran privasi (wajah/pelat nomor atau EXIF bocor ke publik) | Rendah | Tinggi | Hapus EXIF otomatis lewat transformasi Cloudinary dan sediakan fitur blur bagi Admin |

# **14\. Kepatuhan dan Aspek Hukum**

| Topik | Isu bisnis | Tindakan di dalam sistem |
| :---- | :---- | :---- |
| **Pelindungan data pribadi (UU No. 27 Tahun 2022)** | Foto, lokasi, dan kontak dapat memuat data pribadi. Transfer data ke penyedia AI | Minimisasi data (kirim hanya teks non-PII ke AI), persetujuan eksplisit untuk email kontak |
| **Konten dan pencemaran nama baik** | Laporan dapat memuat tuduhan terhadap pihak tertentu | Moderasi sebelum tampil publik, instruksi AI netral, dan validator kata terlarang |
| **Hak atas foto dan privasi pihak ketiga** | Foto bisa memuat wajah, pelat nomor, dan metadata lokasi perangkat | Hapus EXIF pada foto publik via Cloudinary dan sediakan alat blur manual untuk Admin |
| **Retensi dan hak subjek data** | Hak hapus vs audit log yang tidak dapat diubah | Aktor disimpan secara pseudonim di audit log sehingga PII kontak dapat dihapus terpisah |

# **15\. Implementasi, Adopsi, dan Matriks Keterlacakan**

## **15.1 Pendekatan Rilis (Satu Fase Terpadu untuk MIA2026)**

Untuk memaksimalkan dampak produk pada kompetisi **MIA2026**, seluruh fitur **F1 sampai F10** dibangun dalam **Fase 1 (MVP Utama)**:

| Tahap | Tujuan | Cakupan Fitur |
| :---- | :---- | :---- |
| **Fase 1: MVP Utama (MIA2026)** | Membuktikan alur pelaporan warga, bantuan AI aman, moderasi Admin, tindak lanjut instansi, analitik cuaca, dan verifikasi komunitas secara utuh | **F1** (Form & Geotagging), **F2** (AI Draft & Fallback), **F3** (Panel Sanding & Persetujuan), **F4** (Peta Publik, +1, Tag), **F5** (Moderasi & Eskalasi Admin), **F6** (Portal Instansi), **F7** (Audit Trail), **F8** (Pelacakan & Notifikasi), **F9** (Weather-Road Intelligence), dan **F10** (Community Sign-Off) |

## **15.2 Matriks Keterlacakan Tujuan Bisnis ke Kebutuhan dan Fitur**

| Tujuan bisnis | Kebutuhan bisnis | Fitur PRD |
| :---- | :---- | :---- |
| **BO-1 Kemudahan dan kelengkapan laporan** | BR-01, BR-02, BR-12, BR-13, BR-20 | F1, F8 |
| **BO-2 Laporan tepat sasaran dan berkualitas** | BR-06, BR-08, BR-09, BR-19, BR-21 | F4, F5 |
| **BO-3 Kepercayaan pada AI yang aman** | BR-03, BR-04, BR-05 | F2, F3 |
| **BO-4 Tindak lanjut terlihat dan terawasi** | BR-07, BR-10, BR-12, BR-16, BR-21 | F4, F6, F8, F10 |
| **BO-5 Prioritisasi berbasis cuaca** | BR-14, BR-15 | F9 |
| **BO-6 Akuntabilitas dan kepatuhan** | BR-05, BR-09, BR-11, BR-17, BR-18, BR-22 | F3, F5, F6, F7 |

# **16\. Keputusan Kunci dan Pengesahan Dokumen**

Seluruh pertanyaan bisnis pada versi draf telah dikunci menjadi keputusan tetap untuk proyek **MIA2026**:

| No | Topik Keputusan | Keputusan Final | Status |
| :---- | :---- | :---- | :---- |
| **OQ-B01** | Pemilik dokumen, tim pengembang, dan sponsor bisnis | Proyek independen untuk kompetisi **MIA2026** tanpa sponsor/organisasi eksternal. Pemilik dokumen: **Sabira** (PM & UI/UX Designer) dan **Boby** (System Architect & Fullstack Developer). | Diputuskan |
| **OQ-B02** | Bentuk kerja sama dengan Dinas PUPR Kota Bogor | Kemitraan instansi **disimulasikan secara penuh** di dalam platform untuk mendemonstrasikan alur *end-to-end*. | Diputuskan |
| **OQ-B03** | Anggaran dan model infrastruktur | Menggunakan **100% layanan Free Tier** (Vercel, Render, Neon, Cloudinary, Resend, OSM, API Cuaca Gratis) tanpa tabel anggaran biaya. | Diputuskan |
| **OQ-B04** | Kapasitas Admin dan SLA verifikasi | SLA verifikasi ditetapkan 2 hari kerja dalam simulasi operasional. | Diputuskan |
| **OQ-B05** | Visibilitas laporan yang belum diverifikasi | Tidak tampil publik. Hanya laporan berstatus **Terverifikasi** ke atas yang tampil di peta publik. | Diputuskan |
| **OQ-B06** | Kerahasiaan identitas pelapor | Identitas pelapor tidak diteruskan ke instansi kecuali pelapor menyetujui secara eksplisit. | Diputuskan |
| **OQ-B07** | Sumber data cuaca untuk *Weather-Road Intelligence* | Menggunakan daftar API cuaca gratis (BMKG Open Data, Open-Meteo, NASA POWER, OpenWeatherMap Free) yang di-ingest dan disimpan sebagai snapshot. | Diputuskan |
| **OQ-B08** | Kebijakan privasi, penyimpanan foto, dan AI | Teks non-PII diproses oleh `openai gpt-6-luna` (*reasoning medium*). Foto disimpan di **Cloudinary** dengan penghapusan EXIF untuk publik. | Diputuskan |
| **OQ-B09** | Cakupan fitur Fase 1 (MVP Utama) | **F9 (Weather-Road Intelligence)** dan **F10 (Community Sign-Off)** ditarik masuk ke **Fase 1** bersama F1–F8. | Diputuskan |
| **OQ-B10** | Konteks penggunaan produk | Digunakan untuk kompetisi **MIA2026** dengan kualitas arsitektur setara produk siap rilis (*production-grade architecture*). | Diputuskan |

## **Pengesahan Tim Proyek**

| Peran | Nama | Tanggal |
| :---- | :---- | :---- |
| **Product Manager & UI/UX Designer** | **Sabira** | 25 Oktober 2026 |
| **System Architect & Fullstack Developer** | **Boby** | 25 Oktober 2026 |
