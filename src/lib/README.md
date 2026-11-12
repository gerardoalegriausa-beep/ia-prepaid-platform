# src/lib — lógica de servidor y utilidades

- `supabase/` — clientes (browser, server, service-role) y tipos generados
- `credits/` — wrappers de las RPC del ledger (`deduct_user_credits`, `refund`), precios y reglas de prioridad promo → pagado
- `ai/` — capa de proveedores IA y registro de herramientas
- `stripe/` — cliente, catálogo de precios, helpers de webhooks
- `email/` — Resend + plantillas (reportes ROI para comercios)
- `cards/` — generación de códigos de alta entropía, URLs de QR y export CSV
- `validation/` — esquemas (zod) de entrada de API
- `utils/` — utilidades generales (`cn`, formato de créditos, etc.)
