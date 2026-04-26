# Panduan Simpan Proyek ke GitHub
## HABITKELUARGA B30 — Step by Step untuk Pemula

---

## Apa itu GitHub?

GitHub = Google Drive khusus untuk kode.
- **Gratis** untuk proyek personal
- **Aman** — tersimpan di cloud, tidak akan hilang
- **Riwayat** — setiap perubahan tersimpan, bisa kembali ke versi lama
- **Bisa dibagikan** ke developer yang kamu hire nanti

---

## LANGKAH 1 — Daftar GitHub (5 menit)

1. Buka **github.com**
2. Klik **"Sign up"**
3. Isi email, username, password
4. Verifikasi email
5. Pilih paket **Free** (sudah lebih dari cukup)

---

## LANGKAH 2 — Buat Repository Baru (3 menit)

> Repository = folder proyek di GitHub

1. Setelah login, klik tombol **"+"** di pojok kanan atas
2. Pilih **"New repository"**
3. Isi form:
   - **Repository name:** `habitkeluarga-b30`
   - **Description:** `Aplikasi mobile habit tracker keluarga - Jalan Ninja Keluargaku`
   - Pilih **Public** (gratis) atau **Private** (juga gratis)
   - Centang ✅ **"Add a README file"**
4. Klik **"Create repository"**

---

## LANGKAH 3 — Upload File Prototype (5 menit)

Sekarang upload file-file yang sudah kamu download:

1. Di halaman repository yang baru dibuat, klik **"Add file"**
2. Pilih **"Upload files"**
3. Drag & drop file-file berikut:
   - `habitkeluarga_complete.html` ← prototype lengkap
   - `habitkeluarga_techstack.md` ← dokumen teknis
   - `PANDUAN_GITHUB.md` ← panduan ini
4. Di bagian bawah, isi **"Commit changes"**:
   - Tulis: `Add initial prototype and documentation`
5. Klik **"Commit changes"**

✅ File sudah tersimpan di GitHub!

---

## LANGKAH 4 — Aktifkan GitHub Pages (Preview Online)

Ini agar prototype HTML bisa dibuka siapapun via link — berguna untuk show ke developer atau investor!

1. Di repository, klik **"Settings"** (tab paling kanan)
2. Scroll ke bawah, cari **"Pages"** di menu kiri
3. Di bagian **"Source"**, pilih:
   - Branch: **main**
   - Folder: **/ (root)**
4. Klik **"Save"**
5. Tunggu 1–2 menit
6. Refresh halaman — akan muncul link seperti:
   `https://USERNAME.github.io/habitkeluarga-b30/`

Buka link itu → prototype langsung bisa dilihat online! 🎉

---

## LANGKAH 5 — Struktur Folder yang Disarankan

Setelah prototype tersimpan, buat struktur ini untuk proyek ke depannya:

```
habitkeluarga-b30/              ← repository utama
│
├── README.md                   ← deskripsi proyek (otomatis ada)
├── habitkeluarga_complete.html ← prototype lengkap
├── habitkeluarga_techstack.md  ← dokumen teknis
│
├── docs/                       ← semua dokumentasi
│   ├── PRD.md                  ← Product Requirements Document
│   ├── UI_GUIDE.md             ← panduan warna & desain
│   └── API_DESIGN.md           ← rancangan API
│
├── prototype/                  ← semua file prototype
│   ├── 01_onboarding.html
│   ├── 02_owner_dashboard.html
│   ├── 03_member_view.html
│   └── ... dst
│
└── app/                        ← kode React Native (nanti)
    └── ... (diisi saat mulai coding)
```

---

## LANGKAH 6 — Share ke Developer

Saat kamu hire developer, cukup kirim link repository:

```
https://github.com/USERNAME/habitkeluarga-b30
```

Developer akan langsung bisa:
- Lihat semua prototype
- Baca dokumen teknis
- Paham visi & desain aplikasi
- Clone (copy) kode ke komputer mereka

**Ini menghemat biaya konsultasi awal yang biasanya mahal!**

---

## Tips Penting GitHub untuk Pemula

### Commit secara rutin
Setiap kali ada perubahan, "commit" = simpan versi baru.
Anggap seperti save game — lakukan sesering mungkin.

### Tulis pesan commit yang jelas
- ✅ `Add AI insights feature`
- ✅ `Fix habit toggle bug`
- ❌ `update` (terlalu umum)
- ❌ `aaaaaa` (tidak bermakna)

### Jangan upload info sensitif
Jangan pernah upload:
- Password
- API Key (kode rahasia Claude API, Supabase, dll)
- Data pribadi pengguna

Untuk API key, gunakan file `.env` dan tambahkan ke `.gitignore`

### Buat README yang bagus
README.md adalah "wajah" proyek kamu. Isi dengan:
- Apa itu aplikasinya
- Screenshot / link prototype
- Fitur-fitur utama
- Cara menjalankan (nanti saat sudah ada kode)

---

## Contoh README.md yang Bagus

```markdown
# 🏠 HABITKELUARGA B30 — Jalan Ninja Keluargaku

Aplikasi mobile Android untuk membangun kebiasaan terbaik
bersama seluruh anggota keluarga. Bergaya game reward dengan
estetika hangat dan mekanik engagement ala Duolingo.

## Demo Prototype
👉 [Lihat Prototype Interaktif](https://USERNAME.github.io/habitkeluarga-b30/)

## Fitur Utama
- ✅ Habit tracker harian/mingguan/bulanan
- ✅ Sistem poin & reward yang bisa di-redeem
- ✅ Leaderboard keluarga
- ✅ Milestone celebration animations
- ✅ AI-powered monthly insights (Claude API)
- ✅ Pembayaran via QRIS

## Tech Stack
React Native · Expo · Supabase · Claude AI API

## Status
🚧 Dalam pengembangan — prototype selesai, coding dimulai
```

---

## Selamat! Proyekmu sekarang aman di GitHub 🎉

Link yang perlu kamu catat:
- **Repository:** `https://github.com/USERNAME/habitkeluarga-b30`
- **Prototype online:** `https://USERNAME.github.io/habitkeluarga-b30/`

Ganti `USERNAME` dengan username GitHub kamu.

---

*Dokumen ini bagian dari HABITKELUARGA B30 Project · April 2025*
