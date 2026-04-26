# HABITKELUARGA B30 — Backend Plan
## Panduan Backend untuk Developer | Versi 2.0

---

## Pendekatan: MVP dulu, fitur lanjutan bertahap

Berdasarkan diskusi dengan owner, pengembangan backend dibagi menjadi 2 fase:

- **Fase 1 (MVP)** — fitur inti yang cukup untuk dipakai keluarga nyata
- **Fase 2** — fitur lanjutan ditambah setelah MVP launch dan ada feedback

---

## FASE 1 — MVP Backend (Prioritas Utama)

### Fitur yang harus ada di MVP:

| Fitur | Teknologi | Keterangan |
|-------|-----------|------------|
| Login & daftar akun | Supabase Auth | Email + Google OAuth |
| Database keluarga & anggota | Supabase PostgreSQL | Simpan data owner, anggota, keluarga |
| Habit tracker | Supabase DB | Log habit harian/mingguan/bulanan |
| Sistem poin & reward dasar | Supabase DB | Hitung poin, request redeem |
| QR code undang anggota | Supabase + UUID | Generate & validasi QR |
| Dashboard progress | Supabase DB | Query kepatuhan & poin |
| Notifikasi push dasar | Expo Push Notifications | Reminder & pencapaian |
| Pembayaran premium via QRIS | Midtrans/Xendit | Deteksi otomatis + webhook |
| Email aktivasi | Resend.com | Invoice + kode aktivasi |

---

### Arsitektur MVP

```
┌─────────────────────────────────────────────────────┐
│                  MOBILE APP (React Native)           │
│                  Frontend — Expo                     │
└──────────────────────┬──────────────────────────────┘
                       │ HTTPS API calls
                       ▼
┌─────────────────────────────────────────────────────┐
│                   SUPABASE                          │
│                                                     │
│  ┌─────────────┐  ┌──────────┐  ┌───────────────┐  │
│  │  PostgreSQL │  │   Auth   │  │  Edge          │  │
│  │  Database   │  │ (login)  │  │  Functions     │  │
│  └─────────────┘  └──────────┘  └───────┬───────┘  │
└──────────────────────────────────────────┼──────────┘
                                           │
                    ┌──────────────────────┼────────┐
                    │                      │        │
                    ▼                      ▼        ▼
             ┌──────────┐         ┌──────────┐  ┌────────┐
             │ Midtrans │         │  Resend  │  │  Expo  │
             │  (QRIS)  │         │  (email) │  │ Push   │
             └──────────┘         └──────────┘  └────────┘
```

---

### Tabel Database MVP

#### 1. `families` — Data keluarga
```sql
id              UUID PRIMARY KEY
owner_id        UUID REFERENCES users(id)
family_name     TEXT
plan            TEXT DEFAULT 'free' -- 'free' atau 'premium'
plan_expires_at TIMESTAMP
qr_code         TEXT UNIQUE
created_at      TIMESTAMP DEFAULT now()
```

#### 2. `users` — Data pengguna
```sql
id              UUID PRIMARY KEY (dari Supabase Auth)
family_id       UUID REFERENCES families(id)
name            TEXT
role            TEXT -- 'owner' atau 'member'
age_category    TEXT -- 'dasar', 'menengah', 'atas'
avatar_url      TEXT
push_token      TEXT -- untuk notifikasi
created_at      TIMESTAMP DEFAULT now()
```

#### 3. `habits` — Daftar habit
```sql
id              UUID PRIMARY KEY
family_id       UUID REFERENCES families(id)
name            TEXT
description     TEXT -- keterangan singkat
frequency       TEXT -- 'daily', 'weekly', 'monthly'
points          INTEGER DEFAULT 10
is_active       BOOLEAN DEFAULT true
sort_order      INTEGER DEFAULT 0 -- untuk custom ordering
created_at      TIMESTAMP DEFAULT now()
```

#### 4. `habit_logs` — Catatan habit selesai
```sql
id              UUID PRIMARY KEY
habit_id        UUID REFERENCES habits(id)
user_id         UUID REFERENCES users(id)
completed_at    TIMESTAMP DEFAULT now()
log_date        DATE -- tanggal (untuk cek duplikat harian)
```

#### 5. `rewards` — Daftar hadiah
```sql
id              UUID PRIMARY KEY
family_id       UUID REFERENCES families(id)
type            TEXT -- 'point' atau 'habit'
subtype         TEXT -- 'individual' atau 'kolektif'
name            TEXT
emoji           TEXT
age_category    TEXT -- 'dasar', 'menengah', 'atas', 'all'
points_required INTEGER -- untuk reward point
compliance_pct  INTEGER DEFAULT 100 -- untuk reward habit
duration_months INTEGER DEFAULT 1 -- durasi kepatuhan
is_active       BOOLEAN DEFAULT true
created_at      TIMESTAMP DEFAULT now()
```

#### 6. `redemptions` — Catatan redeem
```sql
id              UUID PRIMARY KEY
reward_id       UUID REFERENCES rewards(id)
user_id         UUID REFERENCES users(id)
status          TEXT DEFAULT 'pending' -- 'pending','approved','rejected'
requested_at    TIMESTAMP DEFAULT now()
approved_at     TIMESTAMP
notes           TEXT
```

#### 7. `subscriptions` — Data pembayaran premium
```sql
id              UUID PRIMARY KEY
family_id       UUID REFERENCES families(id)
plan_type       TEXT -- 'monthly','6months','yearly'
amount          INTEGER -- dalam rupiah
payment_id      TEXT -- ID dari Midtrans/Xendit
status          TEXT DEFAULT 'pending'
activation_code TEXT UNIQUE
activated_at    TIMESTAMP
expires_at      TIMESTAMP
created_at      TIMESTAMP DEFAULT now()
```

---

### Alur Pembayaran Premium (Fully Otomatis)

```
1. User pilih paket & isi data daftar
        ↓
2. App request ke Supabase Edge Function
        ↓
3. Edge Function buat transaksi di Midtrans/Xendit
        ↓
4. QRIS ditampilkan ke user
        ↓
5. User bayar via bank/e-wallet
        ↓
6. Midtrans/Xendit kirim webhook ke Edge Function
        ↓
7. Edge Function:
   - Update status pembayaran → 'paid'
   - Generate kode aktivasi unik (misal: HKB-7429)
   - Update plan keluarga → 'premium'
   - Kirim email via Resend (invoice + kode aktivasi)
        ↓
8. User terima email, masukkan kode aktivasi di app
        ↓
9. App aktif sebagai Premium!
```

---

### Contoh Edge Function — Webhook Pembayaran

```javascript
// supabase/functions/payment-webhook/index.ts

import { serve } from 'https://deno.land/std/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js'

serve(async (req) => {
  const payload = await req.json()
  
  // Verifikasi signature dari Midtrans
  const isValid = verifyMidtransSignature(payload)
  if (!isValid) return new Response('Unauthorized', { status: 401 })

  if (payload.transaction_status === 'settlement') {
    const supabase = createClient(SUPABASE_URL, SUPABASE_SERVICE_KEY)
    
    // 1. Update status pembayaran
    await supabase
      .from('subscriptions')
      .update({ status: 'paid', activated_at: new Date() })
      .eq('payment_id', payload.order_id)

    // 2. Ambil data subscription
    const { data: sub } = await supabase
      .from('subscriptions')
      .select('*, families(*)')
      .eq('payment_id', payload.order_id)
      .single()

    // 3. Update plan keluarga jadi premium
    await supabase
      .from('families')
      .update({ 
        plan: 'premium',
        plan_expires_at: calculateExpiry(sub.plan_type)
      })
      .eq('id', sub.family_id)

    // 4. Kirim email via Resend
    await sendActivationEmail({
      to: sub.families.owner_email,
      activationCode: sub.activation_code,
      planType: sub.plan_type,
      amount: sub.amount
    })
  }

  return new Response('OK', { status: 200 })
})
```

---

### Contoh Kirim Email via Resend

```javascript
// lib/email.js

async function sendActivationEmail({ to, activationCode, planType, amount }) {
  const response = await fetch('https://api.resend.com/emails', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${RESEND_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      from: 'HABITKELUARGA B30 <noreply@habitkeluarga.app>',
      to: [to],
      subject: `Aktivasi Premium HABITKELUARGA B30 — Kode: ${activationCode}`,
      html: generateEmailTemplate({ activationCode, planType, amount })
    })
  })
  return response.json()
}
```

---

## FASE 2 — Fitur Lanjutan (Setelah MVP Launch)

Ditambahkan berdasarkan feedback pengguna nyata:

| Fitur | Teknologi | Estimasi |
|-------|-----------|----------|
| AI Insights laporan bulanan | Claude API (Anthropic) | +2 minggu |
| Reward kategori usia advanced | Supabase DB update | +1 minggu |
| Monthly report PDF | Puppeteer / wkhtmltopdf | +2 minggu |
| Milestone celebration system | Supabase DB + Push Notif | +1 minggu |
| Reward kolektif & individual tracking | Supabase DB update | +2 minggu |
| Share laporan via WA/Email | WhatsApp API + Resend | +1 minggu |

### Contoh Integrasi Claude AI untuk Insights

```javascript
// lib/claudeAI.js

async function generateFamilyInsights(familyData) {
  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.CLAUDE_API_KEY,
      'anthropic-version': '2023-06-01'
    },
    body: JSON.stringify({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 800,
      messages: [{
        role: 'user',
        content: `Kamu adalah konsultan habit keluarga yang hangat dan supportif.
        Tulis insight dalam Bahasa Indonesia campur sedikit Inggris.
        
        Data keluarga bulan ${familyData.month}:
        - Kepatuhan keluarga: ${familyData.familyPct}%
        - Anggota terbaik: ${familyData.topMember} (${familyData.topPct}%)
        - Habit terkuat: ${familyData.bestHabit}
        - Habit terlemah: ${familyData.worstHabit}
        - Perubahan vs bulan lalu: ${familyData.improvement}%
        
        Tulis 3 paragraf singkat:
        1. Yang berjalan baik (apresiasi spesifik, sebut nama)
        2. Yang perlu perhatian (konstruktif, tidak menyalahkan)
        3. Target bulan depan (konkret dan achievable)`
      }]
    })
  })
  
  const data = await response.json()
  return data.content[0].text
}
```

---

## Estimasi Biaya & Waktu

### Fase 1 — MVP

| Item | Estimasi Biaya |
|------|---------------|
| Developer (2–3 bulan) | Rp 12.000.000 – 18.000.000 |
| Supabase (gratis di awal) | Rp 0 |
| Expo (gratis) | Rp 0 |
| Midtrans/Xendit setup | Rp 0 (bayar per transaksi 0.7%) |
| Resend email (3000/bln gratis) | Rp 0 |
| Google Play Store (sekali) | ~Rp 400.000 |
| **Total estimasi MVP** | **Rp 12–18 juta** |

### Fase 2 — Fitur Lanjutan

| Item | Estimasi Biaya |
|------|---------------|
| Developer (1–2 bulan tambahan) | Rp 6.000.000 – 12.000.000 |
| Claude API (per 1000 insight ~Rp 2.000) | Tergantung users |
| Supabase Pro (jika >500MB) | ~Rp 375.000/bln |
| **Total estimasi Fase 2** | **Rp 6–12 juta** |

---

## Timeline Pengembangan

```
Bulan 1–2   : Build MVP (backend + sambungkan ke prototype)
Bulan 3     : Testing internal dengan keluarga sendiri
Bulan 4     : Launch ke Google Play Store (soft launch)
Bulan 5–6   : Tambah fitur Fase 2 berdasarkan feedback
Bulan 7+    : Scale up, marketing, tambah pengguna
```

---

## Catatan Penting untuk Developer

1. **Gunakan Supabase Row Level Security (RLS)** — pastikan data satu keluarga tidak bisa diakses keluarga lain
2. **API key Claude jangan di frontend** — selalu panggil via Supabase Edge Function
3. **Test webhook Midtrans** di staging dulu sebelum production
4. **Backup database** minimal seminggu sekali via Supabase dashboard
5. **Versi gratis Supabase** cukup untuk ratusan keluarga di awal

---

*Dokumen ini bagian dari HABITKELUARGA B30 Project — Jalan Ninja Keluargaku*
*Versi 2.0 | April 2025*
