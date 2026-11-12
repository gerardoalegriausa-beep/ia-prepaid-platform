# src/app — Next.js 14 App Router

Rutas de la plataforma. Cada carpeta corresponde a un segmento de URL.

- `(marketing)/` — landing pública (`/`)
- `redeem/` — canje de tarjeta por QR (`/redeem?code=XXXX`)
- `(app)/` — rutas autenticadas: `/dashboard`, `/tools/[slug]`, `/billing`
- `admin/` — panel de administración (`/admin`)
- `auth/` — login, callback de OAuth / magic link
- `api/` — route handlers del backend
