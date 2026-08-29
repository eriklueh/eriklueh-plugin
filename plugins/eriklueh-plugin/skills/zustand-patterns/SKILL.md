---
name: zustand-patterns
description: Patrones de estado con Zustand (v5) en esta app Next.js 16 App Router. Usar al crear o modificar un store, elegir entre estado de servidor y de cliente, o resolver problemas de hidratacion o de re-render.
---

# Zustand (v5) — patrones del proyecto

Skill indice; el detalle vive en `references/`.

## Reglas de oro
- Zustand SOLO para estado de UI de cliente (modales, filtros, wizard, carrito, preferencias). Los datos del servidor viven en la DB (Drizzle) y llegan por props desde Server Components o server actions. Zustand **no** es cache de datos remotos.
- `'use client'` obligatorio en todo componente que consuma el store.
- Selectores atomicos. **Nunca** devolver un objeto/array nuevo en un selector (causa re-render infinito en v5). Para varios valores: `useShallow` de `zustand/react/shallow`.
- Exportar el hook, nunca la instancia del store. Las mutaciones van en acciones dentro del store.
- Tras cualquier `await` dentro de una accion, releer con `get()` (no confiar en valores capturados por closure).
- App Router: para estado inicializado por request, usar store-per-request + Context Provider (evita fuga de estado entre usuarios en SSR). Ver `references/app-router.md`.

## Cuando cargar cada reference
| Necesitas... | Lee |
|---|---|
| Store con datos por request, SSR/RSC, Provider, hidratacion de `persist` | `references/app-router.md` |
| Slices, selectores, middleware, acciones async, testing, gotchas | `references/patterns-and-gotchas.md` |
