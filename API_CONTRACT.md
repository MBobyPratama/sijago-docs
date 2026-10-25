# **REST API CONTRACT (`/v1`)**

**SIJAGO (Sistem Jalan Bogor)** — Spesifikasi Antarmuka Pemrograman Aplikasi (Multirepo Frontend <-> Backend)

| Atribut | Keterangan |
| :---- | :---- |
| **Versi kontrak** | 1.0 (Selaras dengan PRD v1.0 & DATABASE_SCHEMA v1.0) |
| **Tanggal** | 25 Oktober 2026 |
| **Pemilik dokumen** | **Boby** (System Architect & Fullstack Developer) & **Sabira** (PM & UI/UX Designer) |
| **Base URL (Lokal)** | `http://localhost:4000/v1` |
| **Base URL (Produksi)** | `https://<sijago-api>.onrender.com/v1` |
| **Format Pertukaran** | `application/json` (Error mengikuti standar RFC 7807 `application/problem+json`) |

---

## **1. Konvensi Umum & Header Wajib**

Karena SIJAGO menggunakan arsitektur **Multirepo** (`sijago-frontend` di Vercel dan `sijago-backend` di Render), seluruh komunikasi API mengikuti konvensi berikut:

### **1.1 Header Permintaan (Request Headers)**
| Header | Kapan Wajib | Keterangan |
| :---- | :---- | :---- |
| `Content-Type: application/json` | Semua `POST`, `PUT`, `PATCH` | Kecuali endpoint unggah multipart jika tidak memakai *signed upload* langsung |
| `X-Device-Id: <string>` | Semua aksi Warga (tanpa login) | Fingerprint/UUID lokal perangkat untuk menghasilkan `reporter.pseudonym_id` dan mencegah duplikasi upvote (+1) |
| `Idempotency-Key: <uuid>` | Saat menekan **Kirim Laporan** (`POST /v1/reports/:id/submit`) | Mencegah terbentuknya laporan ganda bila sinyal warga terputus (US-06, E-04) |
| `Authorization: Bearer <token>` | Semua endpoint Admin, Auditor, Super Admin, & Petugas Instansi | Token sesi JWT (atau dikirim via `HttpOnly` Cookie) |

### **1.2 Format Standar Respons Sukses**
```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "requestId": "req_01jc8x92k",
    "timestamp": "2026-10-25T16:30:00.000Z"
  }
}
```

### **1.3 Format Standar Respons Error (`problem+json`)**
```json
{
  "success": false,
  "error": {
    "type": "https://sijago.id/errors/approval-hash-mismatch",
    "title": "Persetujuan Draf Tidak Valid",
    "status": 409,
    "code": "APPROVAL_HASH_MISMATCH",
    "detail": "Fakta Asal telah berubah setelah draf disetujui. Silakan tinjau ulang draf sebelum mengirim.",
    "requestId": "req_01jc8x92k"
  }
}
```

---

## **2. Ringkasan Endpoint per Fitur PRD (F1 – F10)**

| Modul / Fitur | Method & Path | Aktor | Deskripsi Singkat |
| :---- | :---- | :---- | :---- |
| **Health & Warm-up** | `GET /v1/health` | Publik | Ping ringan untuk menjaga instance Render Free Tier tetap bangun |
| **F1: Geotagging & Upload** | `POST /v1/geo/validate-point` | Warga | Cek apakah titik berada di dalam poligon Kota Bogor + saran nama jalan Nominatim + cek duplikat 30m |
| **F1: Upload Foto** | `POST /v1/uploads/cloudinary-sign` | Warga / Instansi | Buat *signature* unggah ke Cloudinary & daftarkan metadata EXIF foto |
| **F1: Draf Fakta Asal** | `POST /v1/reports` | Warga | Inisialisasi laporan baru (`DRAFT`) dan simpan Fakta Asal versi 1 beserta `facts_hash` |
| **F1: Update Fakta Asal** | `PUT /v1/reports/:id/facts` | Warga | Perbarui Fakta Asal -> membuat versi baru, mengubah `facts_hash`, & otomatis **mereset persetujuan draf** (FR-3.4) |
| **F2: AI Draft Assistant** | `POST /v1/reports/:id/drafts/generate` | Warga | Panggil **OpenAI `gpt-6-luna` (`reasoning: medium`)** + Validator -> kembalikan Draf AI atau Fallback Draf Template |
| **F3: Panel Sanding & Kirim** | `POST /v1/reports/:id/approve` | Warga | Simpan centang persetujuan warga beserta `approved_facts_hash` |
| **F3: Kirim Final** | `POST /v1/reports/:id/submit` | Warga | Validasi `approved_facts_hash === current_facts_hash` (tolak `409` jika beda), kunci snapshot, terbitkan `ticket_code` |
| **F4: Peta Publik** | `GET /v1/public/reports` | Publik | Ambil daftar & GeoJSON laporan **Terverifikasi ke atas** (dengan filter kategori, status, kecamatan, waktu) |
| **F4: Detail Publik** | `GET /v1/public/reports/:id` | Publik | Ambil detail lengkap laporan publik (foto tanpa EXIF, label draf, jumlah +1, tag, timeline) |
| **F4: Upvote (+1)** | `POST /v1/public/reports/:id/upvote` | Warga | Toggle "+1 Saya Lewat Sini" (unik per perangkat) |
| **F4 & F10: OTP Email** | `POST /v1/auth/otp/request`<br>`POST /v1/auth/otp/verify` | Warga | Kirim & verifikasi OTP 6 digit via Resend untuk aksi Tag Kondisi (F4) dan Community Sign-Off (F10) |
| **F4: Tag Kondisi** | `POST /v1/public/reports/:id/tags` | Warga (OTP) | Tambah tag kondisi lapangan (misal: `[Tertutup Genangan Air]`) |
| **F8: Lacak Tiket** | `GET /v1/track/:ticketCodeOrToken` | Warga | Lacak status, alasan tolak/duplikat, instansi tujuan, dan foto bukti sebelum-sesudah tanpa login |
| **F10: Community Sign-Off** | `POST /v1/public/reports/:id/signoff` | Warga (Eligible + OTP) | Beri penilaian `"MULUS"` atau `"CEPAT_RUSAK"` selama masa pemantauan 30 hari |
| **Auth Internal/Mitra** | `POST /v1/auth/login` | Admin / Instansi | Login akun Admin, Auditor, Super Admin, atau Petugas Instansi (Simulasi) |
| **F5: Moderasi Admin** | `GET /v1/admin/reports`<br>`GET /v1/admin/reports/:id` | Admin | Antrean moderasi lengkap dengan metadata EXIF asli, jarak deviasi GPS, dan kandidat duplikat |
| **F5: Keputusan Moderasi** | `POST /v1/admin/reports/:id/verify`<br>`POST /v1/admin/reports/:id/reject`<br>`POST /v1/admin/reports/:id/merge` | Admin | Verifikasi laporan, tolak dengan alasan baku, atau gabungkan duplikat ke laporan induk |
| **F5: Blur Foto** | `POST /v1/admin/photos/:photoId/blur` | Admin | Terapkan transformasi blur wajah/pelat nomor pada varian publik di Cloudinary (FR-5.12) |
| **F5: Routing & Eskalasi** | `PUT /v1/admin/reports/:id/route`<br>`GET /v1/admin/reports/:id/escalation-preview`<br>`POST /v1/admin/reports/:id/escalate` | Admin | Pilih instansi tujuan, lihat pratinjau email, dan kirim email resmi via Resend (hanya lewat konfirmasi manusia) |
| **F6: Portal Instansi** | `GET /v1/agency/reports`<br>`POST /v1/agency/reports/:id/status`<br>`POST /v1/agency/reports/:id/return` | Petugas Instansi | Lihat laporan milik instansinya, ubah status (`DIPROSES` / `SELESAI` + foto sesudah), atau kembalikan ke Admin |
| **F9: Weather-Road Intel** | `GET /v1/analytics/weather-road`<br>`POST /v1/analytics/weather-road/sync`<br>`GET /v1/analytics/weather-road/export` | Instansi / Admin | Ambil layer curah hujan (BMKG/Open-Meteo) vs kepadatan kerusakan, zona merah, titik kambuhan, & rekomendasi indikatif |
| **F7: Audit Trail & Config** | `GET /v1/audit/reports/:id`<br>`GET /v1/admin/config`<br>`PUT /v1/admin/config` | Admin / Auditor / Super Admin | Telusuri *append-only audit log* & diff versi fakta, serta ubah parameter `system_config` |

---

## **3. Detail Payload Endpoint Kritis**

### **3.1 Validasi Titik Geotagging & Deteksi Duplikat Awal (`POST /v1/geo/validate-point`)**
Digunakan saat warga mengunci titik GPS atau menggeser pin manual di peta (FR-1.3, FR-1.4, FR-1.7, FR-1.9).

**Request Body:**
```json
{
  "lat": -6.5971469,
  "lng": 106.8060388,
  "gpsAccuracyMeters": 12.5,
  "category": "LUBANG"
}
```

**Response `200 OK`:**
```json
{
  "success": true,
  "data": {
    "isInsideBogorCity": true,
    "districtName": "Bogor Tengah",
    "subdistrictName": "Paledang",
    "suggestedStreetName": "Jl. Ir. H. Juanda, Bogor Tengah, Kota Bogor",
    "nearbyActiveReports": [
      {
        "id": "8f14e45f-ceea-467f-a8f1-4e45fceea67f",
        "ticketCode": "SJG-2026-1042",
        "title": "Lubang Jalan Sedalam 10 cm di Dekat Stasiun Bogor",
        "category": "LUBANG",
        "distanceMeters": 18.4,
        "upvoteCount": 14,
        "status": "TERVERIFIKASI",
        "thumbnailUrl": "https://res.cloudinary.com/sijago/image/upload/c_fill,w_300/v1/reports/sample1.jpg"
      }
    ]
  }
}
```
*(Catatan: Jika `isInsideBogorCity: false`, UI menolak langkah berikutnya dengan pesan jelas sesuai US-01).*

---

### **3.2 Simpan / Perbarui Fakta Asal (`POST /v1/reports` & `PUT /v1/reports/:id/facts`)**
Menyimpan sumber kebenaran (*Source of Truth*) dari pelapor. Jika dipanggil lewat `PUT` setelah draf pernah disetujui, server **otomatis membatalkan persetujuan** dan mencatat audit log `SOURCE_FACTS_MODIFIED` serta `APPROVAL_RESET`.

**Request Body:**
```json
{
  "lat": -6.5971469,
  "lng": 106.8060388,
  "gpsAccuracyMeters": 12.5,
  "locationSource": "GPS",
  "category": "LUBANG",
  "sizeClass": "SEDANG",
  "sizeLengthCm": 80,
  "sizeWidthCm": 60,
  "sizeDepthCm": 12,
  "landmark": "Depan gerbang masuk Kebun Raya Bogor Pintu 1, lajur kiri arah BTM",
  "observedAt": "2026-10-25T09:15:00.000Z",
  "impactTags": ["MEMBAHAYAKAN_RODA_DUA", "MENIMBULKAN_MACET"],
  "freeText": "Lubang cukup dalam dan sering tertutup genangan air saat hujan sore, kemarin ada pemotor hampir jatuh.",
  "photoIds": ["c39a11e0-72b1-4810-91a3-99b8c7d6e5f4"],
  "contactEmail": "warga.bogor@example.com",
  "consentNotification": true,
  "shareIdentityWithAgency": false
}
```

**Response `200 OK`:**
```json
{
  "success": true,
  "data": {
    "reportId": "550e8400-e29b-41d4-a716-446655440000",
    "versionNo": 2,
    "factsHash": "a94a8fe5ccb19ba61c4c0873d391e987982fbbd3e29b41d4a716446655440000",
    "approvalReset": true,
    "approvalResetBanner": "Data laporan Anda telah diubah. Demi akurasi, silakan buat ulang dan tinjau kembali draf sebelum mengirim."
  }
}
```

---

### **3.3 Buat Draf Laporan via OpenAI `gpt-6-luna` (`POST /v1/reports/:id/drafts/generate`)**
Memanggil model **OpenAI `gpt-6-luna`** dengan `reasoning_effort: "medium"` (FR-2.1). Jika model melewati timeout, error, atau gagal lolos **Validator Pascagenerasi (FR-2.4)**, endpoint ini **tidak pernah gagal (500)**, melainkan langsung mengembalikan `sourceType: "TEMPLATE"` dengan status `200 OK`.

**Request Body:**
```json
{
  "factsHash": "a94a8fe5ccb19ba61c4c0873d391e987982fbbd3e29b41d4a716446655440000"
}
```

**Response `200 OK` (Sukses AI):**
```json
{
  "success": true,
  "data": {
    "draftId": "d1122334-e29b-41d4-a716-446655449999",
    "sourceType": "AI",
    "permanentLabel": "✨ [Draf Dibantu AI]",
    "title": "Lubang Jalan Berukuran 80x60x12 cm di Depan Pintu 1 Kebun Raya Bogor",
    "body": "Dilaporkan kerusakan jalan kategori lubang berukuran panjang 80 cm, lebar 60 cm, dan kedalaman 12 cm (kelas sedang) dengan patokan di depan gerbang masuk Kebun Raya Bogor Pintu 1, lajur kiri arah BTM. Kerusakan ditemukan pada 25 Oktober 2026 pukul 16.15 WIB. Kondisi lubang tertutup genangan air saat hujan serta berdampak membahayakan pengendara roda dua dan menimbulkan kemacetan.",
    "modelInfo": {
      "model": "gpt-6-luna",
      "reasoningEffort": "medium",
      "latencyMs": 2140
    },
    "fallbackNotice": null,
    "factsHash": "a94a8fe5ccb19ba61c4c0873d391e987982fbbd3e29b41d4a716446655440000"
  }
}
```

**Response `200 OK` (Saat Fallback ke Draf Template):**
```json
{
  "success": true,
  "data": {
    "draftId": "d9988776-e29b-41d4-a716-446655441111",
    "sourceType": "TEMPLATE",
    "permanentLabel": "📄 [Draf Template]",
    "title": "[LUBANG - SEDANG] Laporan Kerusakan Jalan di Depan Pintu 1 Kebun Raya Bogor",
    "body": "Laporan kerusakan jalan kategori LUBANG (kelas ukuran SEDANG: panjang 80 cm, lebar 60 cm, kedalaman 12 cm). Patokan lokasi: Depan gerbang masuk Kebun Raya Bogor Pintu 1, lajur kiri arah BTM. Waktu ditemukan: 25 Oktober 2026 16:15 WIB. Dampak: Membahayakan pengendara roda dua, Menimbulkan macet. Catatan pelapor: Lubang cukup dalam dan sering tertutup genangan air saat hujan sore.",
    "modelInfo": null,
    "fallbackReason": "TIMEOUT",
    "fallbackNotice": "Layanan perapi teks AI sedang sibuk. Sistem otomatis menyusun draf menggunakan format baku agar Anda dapat langsung melanjutkan laporan.",
    "factsHash": "a94a8fe5ccb19ba61c4c0873d391e987982fbbd3e29b41d4a716446655440000"
  }
}
```

---

### **3.4 Persetujuan Sanding & Pengiriman Final (`POST /v1/reports/:id/approve` & `POST /v1/reports/:id/submit`)**

#### **A. Mencatat Centang Persetujuan (`POST /v1/reports/:id/approve`)**
```json
{
  "draftId": "d1122334-e29b-41d4-a716-446655449999",
  "factsHash": "a94a8fe5ccb19ba61c4c0873d391e987982fbbd3e29b41d4a716446655440000",
  "agreed": true
}
```

#### **B. Mengirim Laporan Final (`POST /v1/reports/:id/submit`)**
Wajib menyertakan header `Idempotency-Key: <uuid>`.
```json
{
  "draftId": "d1122334-e29b-41d4-a716-446655449999",
  "approvedFactsHash": "a94a8fe5ccb19ba61c4c0873d391e987982fbbd3e29b41d4a716446655440000",
  "turnstileToken": "0.xxxx"
}
```
* **Aturan Penegakan Server (FR-3.5):** Jika `approvedFactsHash` pada body tidak identik dengan `reports.current_facts_hash` di database, atau draf berstatus `is_stale = true`, server wajib mengembalikan **`409 Conflict` (`APPROVAL_HASH_MISMATCH`)**.
* **Response `201 Created`:**
```json
{
  "success": true,
  "data": {
    "reportId": "550e8400-e29b-41d4-a716-446655440000",
    "ticketCode": "SJG-2026-4819",
    "trackingToken": "trk_9f8e7d6c5b4a3f2e1d0c9b8a",
    "trackingUrl": "/lacak/SJG-2026-4819?t=trk_9f8e7d6c5b4a3f2e1d0c9b8a",
    "status": "MENUNGGU_VERIFIKASI",
    "submittedAt": "2026-10-25T09:18:30.000Z"
  }
}
```

---

### **3.5 Konfirmasi Eskalasi Email Resmi oleh Admin (`POST /v1/admin/reports/:id/escalate`)**
Sesuai prinsip **Aksi Eksternal Hanya Lewat Konfirmasi Manusia (BRL-06, FR-5.7)**, endpoint ini hanya bisa dipanggil oleh Admin setelah meninjau pratinjau email.

**Request Body:**
```json
{
  "agencyId": "ag_pupr_kota_bogor",
  "confirmHumanAction": true,
  "adminAdditionalNote": "Titik berada di jalur padat komuter Stasiun Bogor - BTM, mohon atensi."
}
```

**Response `200 OK`:**
```json
{
  "success": true,
  "data": {
    "escalationId": "esc_77182910",
    "reportStatus": "DITERUSKAN",
    "deliveryStatus": "SENT",
    "resendMessageId": "re_881920a1-b2c3-4d5e",
    "sentTo": ["pengaduan.pupr@kotabogor.simulasi.id"],
    "auditLogId": 1084
  }
}
```

---

### **3.6 Dashboard Weather-Road Intelligence (`GET /v1/analytics/weather-road`)**
Menyediakan data korelasi curah hujan (BMKG / Open-Meteo) dengan frekuensi kerusakan jalan dan titik kambuhan di Kota Bogor (Fitur F9).

**Query Parameters:**
`?periodDays=30&district=ALL&category=ALL`

**Response `200 OK`:**
```json
{
  "success": true,
  "data": {
    "weatherMeta": {
      "primarySource": "OPEN_METEO_AND_BMKG",
      "lastUpdatedAt": "2026-10-25T06:00:00.000Z",
      "isStale": false,
      "earlyWarningBanner": {
        "active": true,
        "source": "BMKG",
        "message": "Peringatan Dini BMKG: Potensi hujan sedang-lebat disertai angin kencang di wilayah Bogor Tengah, Bogor Utara, dan Tanah Sareal pada sore hari."
      }
    },
    "summaryCards": {
      "totalVerifiedReports": 48,
      "avgCityRainfallMm": 68.4,
      "redZoneCount": 3,
      "recurrentHotspotCount": 5
    },
    "districtComparison": [
      {
        "districtName": "Bogor Tengah",
        "rainfallMm": 82.5,
        "verifiedReportsCount": 14,
        "recurrentCount": 2,
        "isRedZone": true
      },
      {
        "districtName": "Tanah Sareal",
        "rainfallMm": 76.0,
        "verifiedReportsCount": 11,
        "recurrentCount": 2,
        "isRedZone": true
      }
    ],
    "hotspotZones": [
      {
        "id": "hz_01",
        "districtName": "Bogor Tengah",
        "center": { "lat": -6.5950, "lng": 106.7990 },
        "radiusMeters": 250,
        "verifiedReportCount": 6,
        "avgRainfallMm": 85.2,
        "recurrenceCount": 2,
        "isRedZone": true,
        "triggerRuleText": "≥ 5 laporan terverifikasi dalam radius 250m dengan curah hujan > 50 mm & ≥ 1 kasus kambuhan dalam 90 hari.",
        "indicativeRecommendation": "[Rekomendasi Indikatif — Bukan Pengganti Survei Teknis] Evaluasi kapasitas saluran drainase tepi jalan dan pertimbangkan perkerasan kaku (rigid pavement / beton) untuk mencegah pengelupasan aspal berulang akibat genangan."
      }
    ]
  }
}
```
