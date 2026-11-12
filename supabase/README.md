# Supabase

- `migrations/` — esquema SQL versionado (`supabase migration new <nombre>`): `profiles`, `merchants`, `card_batches`, `prepaid_cards`, `ai_tools`, `credit_transactions`, políticas RLS y trigger de creación de perfil
- `functions/` — Edge Functions (si se necesitan)
- `seed.sql` — datos iniciales (catálogo de las 5 herramientas)

Las operaciones de créditos van en funciones RPC de PostgreSQL atómicas (`redeem_prepaid_card`, `deduct_user_credits`) para evitar doble gasto.
