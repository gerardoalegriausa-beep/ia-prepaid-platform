# AI Prepaid Cards & Tools Library Platform — Technical Specification & Blueprint

## 1. Project Overview
This project is a **B2B2C SaaS platform** that enables local businesses (coffee shops, coworking spaces, academies, gyms) to distribute **physical QR-coded prepaid cards or digital passes** loaded with AI credits as customer loyalty rewards. End users redeem their cards inside the web platform to access a curated library of guided AI tools (chat, PDF analysis, HD image generation, text-to-speech). Once promotional credits are depleted, users can seamlessly purchase pay-as-you-go top-up packs or subscribe to a monthly plan via Stripe.

---

## 2. Recommended Tech Stack
- **Frontend & Backend API:** Next.js 14+ (App Router), TypeScript, Tailwind CSS, shadcn/ui.
- **Hosting & Serverless Functions:** Vercel (Edge/Node.js runtime for streaming AI responses).
- **Database & Atomic Ledger:** Supabase (PostgreSQL with Row Level Security and RPC stored procedures for atomic credit transactions).
- **Authentication:** Clerk or Supabase Auth (Email magic link, Google OAuth, Apple Sign-In).
- **File Storage (PDFs & Generated Media):** Cloudflare R2 or Supabase Storage.
- **Payments & Subscriptions:** Stripe Checkout, Stripe Billing (subscriptions), and Stripe Webhooks.
- **AI Providers (Server-side SDKs):**
  - Text & Reasoning: OpenAI SDK (`gpt-4o-mini`, `gpt-4o`), Anthropic SDK (`claude-3-5-sonnet`, `claude-3-5-haiku`).
  - Image Generation: Fal.ai / Replicate (`flux-schnell`, `flux-dev`) or OpenAI (`dall-e-3`).
  - Text-to-Speech: ElevenLabs API or OpenAI TTS.
- **Transactional Emails:** Resend.

---

## 3. Core Architecture & Security Rules
1. **Closed-Loop Credit System:** Credits are strictly internal software usage units (`1,000 credits = $1.00 USD` retail value). API keys are never exposed to the client.
2. **Unified Wallet Priority:** Each user has a single wallet tracking two balances:
   - `promo_credits`: Loaded via merchant prepaid cards (consumed **first**).
   - `paid_credits`: Loaded via Stripe top-ups or subscriptions (consumed **second**).
3. **Atomic Deduction Flow:**
   - Before invoking any external AI API, verify `total_credits >= tool_cost`.
   - Deduct credits inside an atomic PostgreSQL transaction (`BEGIN ... COMMIT`) to prevent race conditions or double-spending.
   - If the external AI API fails or times out, execute an automatic credit refund transaction.
4. **Co-Branding Context:** When a user redeems a card issued by a specific merchant (`merchant_id`), store the attribution so the UI displays a subtle co-branded banner (*"Powered by [Merchant Name]"*) and logs redemption analytics for the merchant's ROI report.

---

## 4. Database Schema (PostgreSQL / Supabase)

```sql
-- 1. Users (extends auth.users)
CREATE TABLE public.profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT NOT NULL,
  full_name TEXT,
  promo_credits INTEGER NOT NULL DEFAULT 0 CHECK (promo_credits >= 0),
  paid_credits INTEGER NOT NULL DEFAULT 0 CHECK (paid_credits >= 0),
  referred_by_merchant_id UUID,
  stripe_customer_id TEXT UNIQUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 2. B2B Merchants (Local Businesses)
CREATE TABLE public.merchants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  business_name TEXT NOT NULL,
  contact_name TEXT,
  contact_email TEXT NOT NULL,
  logo_url TEXT,
  brand_color TEXT DEFAULT '#2563eb',
  status TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'inactive')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 3. Prepaid Card Batches (B2B Orders)
CREATE TABLE public.card_batches (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  merchant_id UUID NOT NULL REFERENCES public.merchants(id) ON DELETE RESTRICT,
  batch_name TEXT NOT NULL,
  card_count INTEGER NOT NULL,
  credits_per_card INTEGER NOT NULL DEFAULT 500,
  price_paid_usd NUMERIC(10, 2) NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 4. Individual Prepaid Cards (QR / PIN Codes)
CREATE TABLE public.prepaid_cards (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  batch_id UUID NOT NULL REFERENCES public.card_batches(id) ON DELETE CASCADE,
  merchant_id UUID NOT NULL REFERENCES public.merchants(id),
  redeem_code TEXT UNIQUE NOT NULL, -- e.g., high-entropy 10-char alphanumeric code
  credits_amount INTEGER NOT NULL,
  status TEXT NOT NULL DEFAULT 'unredeemed' CHECK (status IN ('unredeemed', 'redeemed', 'void')),
  redeemed_by_user_id UUID REFERENCES public.profiles(id),
  redeemed_at TIMESTAMPTZ,
  expires_at TIMESTAMPTZ NOT NULL DEFAULT (NOW() + INTERVAL '5 years'), -- US Federal CARD Act compliance
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 5. AI Tools Catalog
CREATE TABLE public.ai_tools (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  slug TEXT UNIQUE NOT NULL, -- e.g., 'fast-chat', 'pro-chat', 'pdf-summarizer', 'hd-image', 'voice-narrator'
  name TEXT NOT NULL,
  description TEXT NOT NULL,
  category TEXT NOT NULL, -- 'productivity', 'education', 'design', 'audio'
  provider TEXT NOT NULL, -- 'openai', 'anthropic', 'fal', 'elevenlabs'
  model_id TEXT NOT NULL,
  credit_cost INTEGER NOT NULL,
  system_prompt TEXT,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 6. Immutable Credit Ledger (Audit Log)
CREATE TABLE public.credit_transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  transaction_type TEXT NOT NULL CHECK (transaction_type IN ('card_redemption', 'stripe_topup', 'subscription_grant', 'tool_usage', 'api_error_refund')),
  promo_delta INTEGER NOT NULL DEFAULT 0,
  paid_delta INTEGER NOT NULL DEFAULT 0,
  tool_id UUID REFERENCES public.ai_tools(id),
  card_id UUID REFERENCES public.prepaid_cards(id),
  stripe_payment_intent_id TEXT,
  metadata JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 5. Credit Pricing & Tool Seed Data
Target gross margin is **66% to 75%** per API execution (`1 Credit = $0.001 USD`):

| Tool Slug | Display Name | Model | Avg API Cost | Credit Cost | Retail Equivalent |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `fast-chat` | Fast Assistant | `gpt-4o-mini` / `claude-3-5-haiku` | $0.0005 | **2 credits** | $0.002 |
| `pro-chat` | Deep Reasoning Chat | `gpt-4o` / `claude-3-5-sonnet` | $0.0150 | **45 credits** | $0.045 |
| `pdf-analyzer` | PDF & Contract Summary | `claude-3-5-sonnet` (10 pages) | $0.0250 | **75 credits** | $0.075 |
| `voice-narrator` | Voice & Audio Studio | `eleven_multilingual_v2` (1 min) | $0.0300 | **90 credits** | $0.090 |
| `hd-image` | HD Image Generator | `flux-dev` / `dall-e-3` | $0.0400 | **120 credits** | $0.120 |

---

## 6. Key API Endpoints & Routes

### Frontend Routes
- `/` — Landing page (B2C tools showcase + B2B merchant inquiry link).
- `/redeem?code=XXXX` — Direct QR landing page; prompts sign-up/login if unauthenticated, then automatically redeems the code.
- `/dashboard` — Unified wallet balance, co-branded merchant banner, and categorized AI Tools Library.
- `/tools/[slug]` — Interactive tool workspace (guided form inputs + streaming output + visible credit cost before execution).
- `/billing` — Top-up packs ($5 = 1,000 credits, $15 = 3,500 credits) and Pro Subscription ($12/mo = 4,000 credits).
- `/admin` — Admin panel to create merchants, generate batch QR codes (exportable as CSV for print vendors), and view redemption stats.

### Backend API Routes (`/api/...`)
- `POST /api/cards/redeem` — Validates `redeem_code`, checks `status = 'unredeemed'`, marks card redeemed, increments `profiles.promo_credits`, sets `referred_by_merchant_id`, and writes a `card_redemption` ledger entry.
- `POST /api/tools/execute` — Verifies user session, checks `promo_credits + paid_credits >= credit_cost`, atomically deducts credits (`promo_credits` first), calls the AI provider with input/output token limits, streams response, and refunds credits on API error.
- `POST /api/stripe/checkout` — Creates a Stripe Checkout Session for one-time credit bundles or recurring monthly subscriptions.
- `POST /api/webhooks/stripe` — Listens for `checkout.session.completed` and `invoice.paid` events to credit `profiles.paid_credits` idempotently.

---

## 7. MVP Development Roadmap

### Phase 1: Core Infrastructure & Ledger (Week 1)
- [ ] Initialize Next.js + TypeScript + Tailwind project and configure Vercel deployment.
- [ ] Set up Supabase project, run database schema migrations, and write PostgreSQL RPC functions (`redeem_prepaid_card`, `deduct_user_credits`).
- [ ] Integrate authentication (Google + Email login) and user profile initialization trigger.

### Phase 2: Card Generation & QR Redemption Flow (Week 2)
- [ ] Build `/admin` batch generator to create unique high-entropy codes (`https://yourdomain.com/redeem?code=ABC123XYZ`) and export CSVs for QR printing.
- [ ] Build `/redeem` seamless onboarding flow so scanning a card takes < 15 seconds from QR scan to active wallet balance.

### Phase 3: AI Tools Library & Guardrails (Week 3)
- [ ] Implement the 5 core guided tools (`fast-chat`, `pro-chat`, `pdf-analyzer`, `voice-narrator`, `hd-image`).
- [ ] Enforce max input/output token caps and dynamic file-size credit calculation for PDFs.
- [ ] Display real-time credit cost badges on every "Generate" button and low-balance alerts (< 50 credits).

### Phase 4: Stripe Monetization & Merchant Analytics (Week 4)
- [ ] Integrate Stripe Checkout for top-up packs ($5 / $15) and Pro Subscription ($12/mo).
- [ ] Implement automated merchant ROI email reports via Resend (cards activated & tools used per batch).
- [ ] Conduct end-to-end testing with test cards and launch local pilot with first 3 merchants.
