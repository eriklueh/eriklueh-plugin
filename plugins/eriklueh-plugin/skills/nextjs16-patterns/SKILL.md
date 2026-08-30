---
name: nextjs16-patterns
description: Patrones y cambios clave de Next.js 16 App Router (params async, proxy.ts, rutas paralelas, caching con cacheComponents, server actions). Usar al crear paginas o rutas, migrar codigo de Next 14/15, o trabajar con caching e invalidacion.
---

# Next.js 16 — patrones del proyecto

Skill indice; el detalle vive en `references/`. Delega: auth -> `clerk`/`project-conventions`, DB -> `drizzle-sqlite`, estado -> `zustand-patterns`, UI -> `shadcn-tailwind-theming`.

## Reglas de oro
- **APIs de request son async**: `params`, `searchParams`, `cookies()`, `headers()`, `draftMode()` se usan con `await`. Tipar `params` como `Promise<...>`.
- **`middleware.ts` -> `proxy.ts`**: renombrado en Next 16 (archivo y funcion exportada). Sigue funcionando `middleware.ts` pero esta deprecado. `proxy` corre SOLO en runtime nodejs (sin Edge). Si necesitas Edge, quedate en `middleware`.
- **Rutas paralelas**: cada slot requiere un `default.tsx` explicito o el build FALLA.
- **Caching es opt-in**: nada se cachea por defecto. `'use cache'` requiere `cacheComponents: true` en `next.config`. Ver `references/caching.md`.
- **`ssr: false`** en `next/dynamic` NO se permite en Server Components (solo en `'use client'`).
- Base: Node >= 20.9, TS >= 5.1, React 19.2, Turbopack por defecto.

## Cuando cargar cada reference
| Necesitas... | Lee |
|---|---|
| Migrar de Next 14/15, tabla de breaking changes, firma async de Page | `references/breaking-changes.md` |
| `use cache`, `revalidateTag`, `updateTag`, invalidacion desde server actions/webhooks | `references/caching.md` |
| Convertir la app en PWA (manifest, service worker, web push, offline) | `references/pwa.md` |
| Diagnosticar errores comunes de build/runtime | `references/errors.md` |
