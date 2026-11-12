# API routes (`/api/...`)

| Ruta | Descripción |
| --- | --- |
| `POST /api/cards/redeem` | Valida el código, lo marca canjeado, suma `promo_credits` y registra en el ledger (RPC `redeem_prepaid_card`). |
| `POST /api/tools/execute` | Verifica sesión y saldo, descuenta créditos de forma atómica (promo primero), llama al proveedor IA, hace streaming y reembolsa si falla. |
| `POST /api/stripe/checkout` | Crea la sesión de Stripe Checkout (recarga o suscripción). |
| `POST /api/webhooks/stripe` | `checkout.session.completed` / `invoice.paid` → acredita `paid_credits` de forma idempotente. |
| `/api/admin/*` | Endpoints internos: lotes, export CSV, stats. |
| `/api/cron/*` | Tareas programadas (Vercel Cron), p. ej. reportes ROI a comercios. |

Ninguna API key de proveedores se expone al cliente.
