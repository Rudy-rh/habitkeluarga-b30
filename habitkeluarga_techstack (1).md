# HABITKELUARGA B30 — Tech Stack & Arsitektur App
### Panduan untuk Developer Pemula | Versi 1.0

---

## Selamat datang, developer baru! 👋

Dokumen ini dirancang khusus untuk kamu yang baru belajar coding. Setiap bagian dijelaskan dengan bahasa sederhana, disertai alasan kenapa teknologi itu dipilih, dan estimasi waktu belajarnya.

**Jangan panik** — kamu tidak perlu menguasai semua ini sekaligus. Ada urutan belajar yang disarankan di bagian akhir dokumen.

---

## 1. Gambaran Besar Arsitektur

```
┌─────────────────────────────────────────┐
│           APLIKASI ANDROID              │
│         (yang user pegang)              │
│                                         │
│  ┌──────────┐    ┌──────────────────┐   │
│  │  Layar   │    │  Logika Aplikasi │   │
│  │  (UI)    │◄──►│  (JavaScript)    │   │
│  └──────────┘    └────────┬─────────┘   │
└───────────────────────────┼─────────────┘
                            │ Internet
                            ▼
┌─────────────────────────────────────────┐
│              BACKEND / SERVER           │
│         (otak di balik layar)           │
│                                         │
│  ┌──────────┐    ┌──────────────────┐   │
│  │ Database │    │   Claude AI API  │   │
│  │(Supabase)│    │  (untuk Insight) │   │
│  └──────────┘    └──────────────────┘   │
└─────────────────────────────────────────┘
```

**Analoginya sederhana:**
- **Aplikasi** = toko yang pelanggan masuk dan lihat produk
- **Backend** = gudang dan kasir di belakang layar
- **Database** = rak penyimpanan semua data
- **Claude AI API** = konsultan pintar yang dipanggil saat dibutuhkan

---

## 2. Tech Stack yang Dipilih

### 2.1 Frontend (Tampilan Aplikasi)

| Teknologi | Fungsi | Kenapa Dipilih |
|-----------|--------|----------------|
| **React Native** | Membuat tampilan aplikasi | 1 kode = bisa Android & iOS |
| **Expo** | Tools bantu React Native | Lebih mudah untuk pemula, tidak perlu setup rumit |
| **JavaScript** | Bahasa pemrograman utama | Bahasa paling ramah pemula, komunitas besar |
| **NativeWind** | Styling / pewarnaan tampilan | Seperti CSS tapi untuk mobile, cepat dipelajari |

> **Untuk pemula:** Mulai dari JavaScript dulu (2–3 bulan), baru masuk React Native.

---

### 2.2 Backend & Database

| Teknologi | Fungsi | Kenapa Dipilih |
|-----------|--------|----------------|
| **Supabase** | Database + autentikasi | Gratis untuk mulai, ada dashboard visual, dokumentasi bagus |
| **PostgreSQL** | Tipe database | Sudah termasuk di Supabase, tidak perlu setup sendiri |
| **Supabase Auth** | Login / daftar akun | Tinggal pakai, tidak perlu buat dari nol |
| **Supabase Storage** | Simpan foto profil, file | Gratis sampai 1GB |

> **Untuk pemula:** Supabase punya dashboard seperti Excel — kamu bisa lihat data langsung tanpa perlu hafal perintah database.

---

### 2.3 AI & Notifikasi

| Teknologi | Fungsi | Kenapa Dipilih |
|-----------|--------|----------------|
| **Claude API (Anthropic)** | Generate AI Insights laporan | Sudah terbukti bagus untuk bahasa Indonesia |
| **Expo Push Notifications** | Kirim notif ke HP anggota | Sudah terintegrasi dengan Expo, mudah dipakai |
| **Resend** | Kirim laporan via email | Gratis 3000 email/bulan, mudah diintegrasikan |

---

### 2.4 Tools Pengembangan

| Tool | Fungsi | Biaya |
|------|--------|-------|
| **VS Code** | Tempat menulis kode | Gratis |
| **GitHub** | Simpan dan backup kode | Gratis |
| **Expo Go** | Test aplikasi di HP langsung | Gratis |
| **Figma** | Desain UI (sudah ada dari prototype ini) | Gratis |
| **Postman** | Test koneksi ke server | Gratis |

---

## 3. Struktur Folder Proyek

```
habitkeluarga-app/
│
├── app/                        ← Semua halaman aplikasi
│   ├── (auth)/
│   │   ├── login.jsx           ← Halaman login
│   │   └── register.jsx        ← Halaman daftar
│   ├── (owner)/
│   │   ├── dashboard.jsx       ← Dashboard Owner
│   │   ├── habits.jsx          ← Kelola habit
│   │   ├── rewards.jsx         ← Kelola reward
│   │   ├── members.jsx         ← Kelola anggota
│   │   └── report.jsx          ← Monthly report + AI Insights
│   ├── (member)/
│   │   ├── home.jsx            ← Home anggota
│   │   ├── leaderboard.jsx     ← Papan peringkat
│   │   └── notifications.jsx   ← Notifikasi
│   └── premium.jsx             ← Halaman upgrade premium
│
├── components/                 ← Komponen yang dipakai berulang
│   ├── HabitRow.jsx            ← Satu baris habit
│   ├── MemberCard.jsx          ← Kartu anggota
│   ├── MilestonePopup.jsx      ← Animasi milestone
│   ├── ProgressRing.jsx        ← Lingkaran progress
│   └── QRScanner.jsx           ← Scanner QR kode
│
├── lib/                        ← Logika & koneksi
│   ├── supabase.js             ← Koneksi ke database
│   ├── claudeAI.js             ← Koneksi ke Claude API
│   └── notifications.js        ← Setup notifikasi push
│
├── hooks/                      ← Fungsi-fungsi khusus React
│   ├── useHabits.js            ← Ambil data habit
│   ├── useFamily.js            ← Ambil data keluarga
│   └── usePoints.js            ← Hitung poin
│
├── constants/
│   └── colors.js               ← Warna-warna app (ungu, hijau, dll)
│
└── assets/
    └── images/                 ← Gambar, logo, ikon
```

---

## 4. Database — Tabel yang Dibutuhkan

Ini seperti "rak-rak" tempat menyimpan data. Ditulis sederhana agar mudah dipahami:

### Tabel `users` — Data pengguna
```
id              → nomor unik tiap pengguna
name            → nama (Ayah, Ibu, Afkar, Zahra)
role            → "owner" atau "member"
family_id       → kode keluarga (untuk sambungkan anggota ke owner)
avatar_url      → foto profil
created_at      → tanggal daftar
```

### Tabel `families` — Data keluarga
```
id              → nomor unik keluarga
owner_id        → siapa ownernya
family_name     → nama keluarga (misal: "Keluarga Bapak Ahmad")
plan            → "free" atau "premium"
qr_code         → kode QR untuk undang anggota
```

### Tabel `habits` — Daftar habit
```
id              → nomor unik habit
family_id       → milik keluarga mana
name            → nama habit (Sholat Subuh, dll)
frequency       → "daily", "weekly", atau "monthly"
points          → berapa poin jika selesai
is_active       → aktif atau tidak (toggle on/off)
time_reminder   → waktu pengingat (misal: "05:00")
```

### Tabel `habit_logs` — Catatan habit yang sudah dikerjakan
```
id              → nomor unik
habit_id        → habit yang dikerjakan
user_id         → siapa yang mengerjakan
completed_at    → kapan selesai
date            → tanggal (untuk tracking harian)
```

### Tabel `rewards` — Daftar hadiah
```
id              → nomor unik
family_id       → milik keluarga mana
name            → nama hadiah (Es krim, Gaming 1 jam, dll)
points_required → berapa poin untuk redeem
emoji           → ikon hadiah
type            → "points" atau "milestone"
```

### Tabel `redemptions` — Catatan redeem hadiah
```
id              → nomor unik
reward_id       → hadiah apa
user_id         → siapa yang redeem
status          → "pending", "approved", atau "rejected"
requested_at    → kapan minta
approved_at     → kapan disetujui owner
```

### Tabel `monthly_reports` — Laporan bulanan
```
id              → nomor unik
family_id       → keluarga mana
month           → bulan laporan (misal: "2025-04")
ai_insight_good → teks insight "yang berjalan baik"
ai_insight_improve → teks "perlu perhatian"
ai_insight_target  → teks "target bulan depan"
owner_note      → pesan tambahan dari owner
is_sent         → sudah dikirim ke anggota atau belum
```

---

## 5. Alur Utama Aplikasi (User Flow)

### Alur A — Owner pertama kali daftar
```
1. Download app → 2. Pilih "Owner" → 3. Daftar akun
→ 4. Isi nama keluarga → 5. Pilih habit dari daftar (free/premium)
→ 6. Generate QR code → 7. Share QR ke anggota
→ 8. Mulai tracking!
```

### Alur B — Anggota bergabung
```
1. Download app → 2. Pilih "Anggota" → 3. Scan QR dari owner
→ 4. Isi nama & foto → 5. Langsung terhubung ke akun keluarga
→ 6. Mulai log habit!
```

### Alur C — Log habit harian
```
1. Buka app → 2. Lihat daftar habit hari ini
→ 3. Tap centang saat selesai → 4. Poin otomatis bertambah
→ 5. Jika capai milestone → popup celebration muncul
→ 6. Notif terkirim ke owner & anggota lain
```

### Alur D — Generate AI Report (akhir bulan)
```
1. Owner buka Monthly Report → 2. Tap "Generate AI Insights"
→ 3. Data keluarga dikirim ke Claude API
→ 4. AI tulis insight dalam bahasa Indonesia
→ 5. Owner baca, edit jika perlu → 6. Tambah pesan pribadi
→ 7. Kirim ke semua anggota via notifikasi + email
```

---

## 6. Koneksi ke Claude AI — Cara Kerjanya

Ini adalah "otak" di balik fitur AI Insights. Cara kerjanya:

```javascript
// Contoh kode sederhana (lib/claudeAI.js)

async function generateFamilyInsights(familyData) {

  // 1. Siapkan data keluarga
  const prompt = `
    Kamu adalah konsultan habit keluarga yang hangat dan supportif.
    Tulis insight dalam Bahasa Indonesia yang campur sedikit Inggris.
    
    Data keluarga bulan ${familyData.month}:
    - Kepatuhan keluarga: ${familyData.familyPct}%
    - Ibu: ${familyData.ibuPct}% (${familyData.ibuPts} poin)
    - Ayah: ${familyData.ayahPct}%
    - Afkar: ${familyData.afkarPct}%
    - Zahra: ${familyData.zahraPct}%
    - Habit terlemah: ${familyData.worstHabit}
    - Habit terkuat: ${familyData.bestHabit}
    - Perubahan vs bulan lalu: ${familyData.improvement}%
    
    Tulis 3 paragraf singkat:
    1. Yang berjalan baik (apresiasi yang spesifik, sebut nama anggota)
    2. Yang perlu perhatian (konstruktif, tidak menyalahkan)
    3. Target bulan depan (konkret dan bisa dicapai)
  `;

  // 2. Kirim ke Claude API
  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': CLAUDE_API_KEY,        // simpan di .env, jangan hardcode!
      'anthropic-version': '2023-06-01'
    },
    body: JSON.stringify({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 800,
      messages: [{ role: 'user', content: prompt }]
    })
  });

  // 3. Ambil hasilnya
  const data = await response.json();
  return data.content[0].text;
}
```

> **Penting:** API key Claude jangan pernah ditulis langsung di kode aplikasi. Simpan di server (Supabase Edge Function) agar tidak bisa dicuri.

---

## 7. Fitur Gratis vs Premium — Cara Implementasinya

Di database, setiap keluarga punya kolom `plan` berisi "free" atau "premium". Cara ceknya di kode:

```javascript
// Contoh pengecekan plan
function canAddMember(family) {
  if (family.plan === 'premium') {
    return true;                    // premium: boleh tambah unlimited
  }
  return family.memberCount < 2;   // free: max 1 anggota (owner + 1)
}

function canAddCustomHabit(family) {
  return family.plan === 'premium'; // custom habit hanya premium
}
```

Setelah owner bayar via QRIS dan konfirmasi ke kamu (admin), kamu update kolom `plan` jadi "premium" di dashboard Supabase. Sesederhana itu untuk versi awal!

---

## 8. Estimasi Biaya Operasional per Bulan

| Layanan | Paket Gratis | Catatan |
|---------|-------------|---------|
| **Supabase** | Gratis (500MB database) | Cukup untuk ratusan keluarga |
| **Expo** | Gratis | Untuk development & testing |
| **Claude API** | Bayar per pemakaian | ~Rp 150/generate insight |
| **Resend (email)** | 3000 email/bulan gratis | Cukup untuk awal |
| **Google Play Store** | $25 sekali bayar | Untuk publish app |
| **Total awal** | ~$25 + biaya Claude API | Sangat terjangkau! |

---

## 9. Urutan Belajar yang Disarankan

Jangan belajar semua sekaligus. Ikuti urutan ini:

### Bulan 1–2: Fondasi
- [ ] **HTML & CSS dasar** — belajar di freeCodeCamp.org (gratis)
- [ ] **JavaScript dasar** — variabel, fungsi, kondisi, loop
- [ ] Buat halaman web sederhana

### Bulan 3–4: React & Mobile
- [ ] **React dasar** — komponen, props, state
- [ ] **React Native + Expo** — install, buat app pertama
- [ ] Buat halaman login sederhana

### Bulan 5–6: Database & Backend
- [ ] **Supabase** — buat akun, buat tabel, sambungkan ke app
- [ ] Buat fitur daftar & login yang nyata
- [ ] Simpan data habit ke database

### Bulan 7–8: Fitur Lengkap
- [ ] Semua fitur utama: habit tracker, poin, reward
- [ ] QR code untuk undang anggota
- [ ] Push notifications

### Bulan 9–10: AI & Polish
- [ ] Integrasi Claude API untuk AI Insights
- [ ] Animasi milestone
- [ ] Monthly report

### Bulan 11–12: Launch!
- [ ] Testing di berbagai HP Android
- [ ] Upload ke Google Play Store
- [ ] Launching HABITKELUARGA B30! 🚀

---

## 10. Sumber Belajar Gratis Terbaik

| Sumber | Untuk Belajar | Bahasa |
|--------|--------------|--------|
| **freeCodeCamp.org** | JavaScript, React | Inggris (ada subtitle) |
| **YouTube: Web Programming UNPAS** | HTML, CSS, JS | Indonesia |
| **YouTube: Kelas Terbuka** | React Native | Indonesia |
| **docs.expo.dev** | Expo & React Native | Inggris |
| **supabase.com/docs** | Supabase | Inggris |
| **Claude.ai** | Tanya apa saja saat stuck! | Indonesia |

---

## 11. Tips Penting untuk Developer Pemula

1. **Jangan takut error** — error adalah guru terbaik. Copy pesan error, tanya ke Claude atau Google.

2. **Buat versi paling sederhana dulu** — mulai dari 1 habit, 1 anggota, tanpa AI. Tambahkan fitur satu per satu.

3. **Commit ke GitHub setiap hari** — seperti save game. Kalau ada yang salah, bisa kembali ke versi sebelumnya.

4. **Jangan beli kursus mahal dulu** — sumber gratis sudah sangat cukup untuk tahap awal.

5. **Bergabung komunitas** — Discord "React Native Indonesia" atau grup Telegram developer lokal.

6. **Prototype ini adalah aset berharga** — tunjukkan ke developer lain sebagai referensi desain. Ini menghemat berminggu-minggu waktu!

---

## 12. Estimasi Timeline Realistis

| Kondisi | Waktu Sampai Bisa Publish |
|---------|--------------------------|
| Belajar 1 jam/hari | 18–24 bulan |
| Belajar 3 jam/hari | 10–14 bulan |
| Hire developer (sambil kamu belajar) | 3–6 bulan |
| Hire developer full, kamu jadi product owner | 2–3 bulan |

> **Rekomendasi:** Belajar JavaScript & React dulu sambil hire freelancer untuk versi pertama. Kamu bisa review kodenya dan pelan-pelan ambil alih.

---

## Penutup

Aplikasi **HABITKELUARGA B30 — Jalan Ninja Keluargaku** ini punya potensi yang luar biasa. Tidak hanya untuk keluarga sendiri, tapi bisa menjadi produk yang membantu ribuan keluarga Muslim Indonesia membangun budaya dan kebiasaan mulia bersama.

Semangat belajar! Setiap baris kode yang kamu tulis adalah investasi untuk masa depan keluarga dan masyarakat. 💪

*Dokumen ini dibuat bersama Claude AI — HABITKELUARGA B30 Project, April 2025*

---
*Versi 1.0 | Dibuat dengan ❤️ untuk keluarga Indonesia*
