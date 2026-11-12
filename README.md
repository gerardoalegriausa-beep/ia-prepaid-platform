# IA Prepaid Platform

Plataforma B2B2C: comercios locales en EE. UU. compran tarjetas prepago con QR cargadas de créditos de IA y las regalan a sus clientes. Los usuarios canjean la tarjeta en un wallet unificado (créditos promo primero, luego pagados) y usan una biblioteca de herramientas IA guiadas. **1,000 créditos = $1 USD.**

Especificación completa: [`docs/blueprint.md`](docs/blueprint.md).

## Stack

Next.js 14 (App Router) · TypeScript · Tailwind CSS · shadcn/ui · Vercel · Supabase (Postgres + RLS + RPC) · Stripe · Resend · Anthropic / OpenAI / Google / Fal / ElevenLabs

## Estructura

```
.
├── docs/                    # Blueprint y decisiones de arquitectura
├── public/                  # Assets estáticos (logos de comercios, imágenes)
├── scripts/                 # Utilidades de desarrollo/operación
├── supabase/
│   ├── migrations/          # Esquema SQL, RLS y RPC del ledger
│   └── functions/           # Edge Functions
├── src/
│   ├── app/                 # Rutas (App Router)
│   │   ├── (marketing)/     # /
│   │   ├── redeem/          # /redeem?code=XXXX
│   │   ├── (app)/           # Rutas con sesión
│   │   │   ├── dashboard/   # /dashboard
│   │   │   ├── tools/[slug]/# /tools/[slug]
│   │   │   └── billing/     # /billing
│   │   ├── admin/           # /admin (comercios, lotes, stats)
│   │   ├── auth/            # login y callback
│   │   └── api/             # cards/redeem, tools/execute, stripe/checkout,
│   │                        # webhooks/stripe, admin, cron
│   ├── components/          # ui (shadcn), wallet, tools, merchant, admin, layout
│   ├── config/              # Precios, paquetes y constantes de producto
│   ├── hooks/
│   ├── lib/
│   │   ├── ai/              # providers/{anthropic,openai,google,fal,elevenlabs}, tools, guardrails
│   │   ├── credits/         # Wrappers del ledger atómico
│   │   ├── cards/           # Códigos de canje, QR, export CSV
│   │   ├── supabase/        # Clientes y tipos
│   │   ├── stripe/
│   │   ├── email/           # Resend + plantillas
│   │   ├── validation/
│   │   └── utils/
│   └── types/
└── tests/                   # unit, integration, e2e
```

Cada carpeta principal tiene un `README.md` con su propósito.

## Roadmap

1. **Fase 1** — Infraestructura y ledger (Next.js, Supabase, migraciones, RPC, auth)
2. **Fase 2** — Generación de lotes en `/admin` y flujo de canje QR
3. **Fase 3** — Las 5 herramientas IA y guardrails
4. **Fase 4** — Stripe y reportes ROI para comercios (Resend)
