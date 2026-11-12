# Capa de proveedores IA

Los proveedores se exponen detrás de una interfaz común para que cada herramienta pueda cambiar de motor sin tocar la ruta `/api/tools/execute`.

- `providers/` — un adaptador por motor: `anthropic`, `openai`, `google`, `fal`, `elevenlabs`
- `tools/` — definición de cada herramienta (slug, proveedor/modelo, costo en créditos, prompt de sistema, límites de tokens/tamaño)
- `guardrails/` — límites de entrada/salida, cálculo dinámico de costo (p. ej. PDFs por páginas), moderación

> Los IDs de modelo del blueprint (gpt-4o, claude-3-5-*) están desactualizados; elegir modelos vigentes al implementar la Fase 3.
