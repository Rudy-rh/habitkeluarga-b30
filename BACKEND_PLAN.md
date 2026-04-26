# HABITKELUARGA B30 — Backend Plan
## Panduan Backend untuk Developer | Versi 2.1

---

## Pendekatan: MVP dulu, fitur lanjutan bertahap

Pengembangan backend dibagi menjadi 2 fase:

- **Fase 1 (MVP)** — fitur inti yang cukup untuk dipakai keluarga nyata
- **Fase 2** — fitur lanjutan ditambah setelah MVP launch dan ada feedback pengguna

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
| Notifikasi push dasar | Expo Push Notifications | Reminder & pencapaian reward |
| Pembayaran premium via QRIS | Midtrans/Xendit | Deteksi otomatis + webhook |
| Email aktivasi | Resend.com | Invoice + kode aktivasi otomatis |

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
plan            TEXT DEFAULT 'free'  -- 'free' atau 'premium'
plan_expires_at TIMESTAMP
qr_code         TEXT UNIQUE
created_at      TIMESTAMP DEFAULT now()
```

#### 2. `users` — Data pengguna
```sql
id              UUID PRIMARY KEY  -- dari Supabase Auth
family_id       UUID REFERENCES families(id)
name            TEXT
role            TEXT              -- 'owner' atau 'member'
age_category    TEXT              -- 'dasar', 'menengah', 'atas'
avatar_url      TEXT
push_token      TEXT              -- untuk notifikasi
created_at      TIMESTAMP DEFAULT now()
```

#### 3. `habits` — Daftar habit
```sql
id              UUID PRIMARY KEY
family_id       UUID REFERENCES families(id)
name            TEXT
description     TEXT              -- keterangan singkat
frequency       TEXT              -- 'daily', 'weekly', 'monthly'
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
log_date        DATE              -- untuk cek duplikat harian
```

#### 5. `rewards` — Daftar hadiah
```sql
id              UUID PRIMARY KEY
family_id       UUID REFERENCES families(id)
type            TEXT              -- 'point' atau 'habit'
subtype         TEXT              -- 'individual' atau 'kolektif'
name            TEXT
emoji           TEXT
age_category    TEXT              -- 'dasar', 'menengah', 'atas', 'all'
points_required INTEGER           -- untuk reward point
compliance_pct  INTEGER DEFAULT 100
duration_months INTEGER DEFAULT 1 -- durasi kepatuhan (bulan)
is_active       BOOLEAN DEFAULT true
created_at      TIMESTAMP DEFAULT now()
```

#### 6. `redemptions` — Catatan redeem hadiah
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
plan_type       TEXT              -- 'monthly','6months','yearly'
amount          INTEGER           -- dalam rupiah
payment_id      TEXT              -- ID dari Midtrans/Xendit
status          TEXT DEFAULT 'pending'
activation_code TEXT UNIQUE
activated_at    TIMESTAMP
expires_at      TIMESTAMP
created_at      TIMESTAMP DEFAULT now()
```

---

### Alur Pembayaran Premium (Fully Otomatis)

```
1. User pilih paket & isi data pendaftaran
        ↓
2. App request ke Supabase Edge Function
        ↓
3. Edge Function buat transaksi di Midtrans/Xendit
        ↓
4. QRIS ditampilkan ke user
        ↓
5. User bayar via bank/e-wallet manapun
        ↓
6. Midtrans/Xendit kirim webhook ke Edge Function
        ↓
7. Edge Function otomatis:
   - Update status pembayaran → 'paid'
   - Generate kode aktivasi unik (contoh: HKB-7429)
   - Update plan keluarga → 'premium'
   - Kirim email via Resend (invoice + kode aktivasi)
        ↓
8. User terima email, masukkan kode aktivasi di app
        ↓
9. Akun Premium langsung aktif!
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

| Fitur | Teknologi |
|-------|-----------|
| AI Insights laporan bulanan | Claude API (Anthropic) |
| Reward kategori usia advanced | Supabase DB update |
| Monthly report PDF | Puppeteer / wkhtmltopdf |
| Milestone celebration system | Supabase DB + Push Notif |
| Reward kolektif & individual tracking | Supabase DB update |
| Share laporan via WA/Email | WhatsApp API + Resend |

### Contoh Integrasi Claude AI untuk Insights

```javascript
// lib/claudeAI.js

async function generateFamilyInsights(familyData) {
  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.CLAUDE_API_KEY, // simpan di .env, jangan hardcode!
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
        1. Yang berjalan baik (apresiasi spesifik, sebut nama anggota)
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

## Catatan Penting untuk Developer

1. **Gunakan Supabase Row Level Security (RLS)** — pastikan data satu keluarga tidak bisa diakses keluarga lain
2. **API key Claude jangan di frontend** — selalu panggil via Supabase Edge Function agar aman
3. **Test webhook Midtrans** di environment staging dulu sebelum production
4. **Backup database** secara rutin via Supabase dashboard
5. **Versi gratis Supabase** sudah mencukupi untuk ratusan keluarga di tahap awal
6. **Jangan upload file `.env`** ke GitHub — simpan API key di environment variables

---

## Referensi Dokumentasi

- Supabase: https://supabase.com/docs
- Expo Push Notifications: https://docs.expo.dev/push-notifications/overview
- Midtrans: https://docs.midtrans.com
- Resend: https://resend.com/docs
- Claude API: https://docs.anthropic.com

---

*Dokumen ini bagian dari HABITKELUARGA B30 Project — Jalan Ninja Keluargaku*
*Versi 2.1 | April 2025*
