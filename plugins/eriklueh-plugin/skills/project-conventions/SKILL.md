---
name: project-conventions
description: Convenciones generales y arranque del stack (Next.js 16 App Router + Clerk + Zustand + Drizzle/SQLite + shadcn/ui + Tailwind v4). Usar al crear features, decidir donde va el codigo, configurar el proyecto, o integrar Clerk.
---

# Convenciones del proyecto (indice)

Skill paraguas: enlaza a las skills especificas y fija las convenciones transversales.

## Skills del stack
- Next.js 16 App Router -> skill `nextjs16-patterns`
- Estado de cliente -> skill `zustand-patterns`
- Datos (Drizzle + SQLite) -> skill `drizzle-sqlite`
- UI (shadcn/ui + Tailwind v4) -> skill `shadcn-tailwind-theming`
- Auth (Clerk) -> resumen abajo + skill/MCP oficial de Clerk

## Estructura de carpetas
- `app/` rutas (App Router). `components/ui/` shadcn; `components/` propios.
- `db/` esquema, cliente y migraciones Drizzle. `stores/` stores Zustand. `lib/` utilidades.

## Fronteras servidor / cliente
- Todo lo que toque `db/` o secretos es server-only: Server Components, Route Handlers, Server Actions. Nunca importar `db/` en `'use client'` ni en runtime edge (better-sqlite3 es nativo de Node).
- Zustand y los hooks de cliente de Clerk (`useUser`, `useAuth`) van en `'use client'`.

## Auth con Clerk (Core 3, Next 16) — verificado
- **Middleware**: en Next 16 el archivo es `proxy.ts` (en Next <=15 seria `middleware.ts`); mismo contenido, solo cambia el nombre.
  ```ts
  // proxy.ts
  import { clerkMiddleware } from '@clerk/nextjs/server'
  export default clerkMiddleware()
  export const config = { matcher: ['/((?!_next|[^?]*\\.[^?]*).*)', '/(api|trpc)(.*)', '/__clerk/(.*)'] }
  ```
  Este archivo es OBLIGATORIO aunque no protejas rutas ahi (si falta, `auth()` lanza error). En Next 16 el matcher DEBE incluir `'/__clerk/(.*)'`.
- **NO usar `createRouteMatcher`** (deprecado) ni proteger rutas dentro del middleware. Proteger cerca del recurso.
- **Servidor**: `import { auth, currentUser } from '@clerk/nextjs/server'`; ambas ASYNC -> `const { userId } = await auth()`, `const user = await currentUser()`, o `await auth.protect()` en Server Components / Route Handlers / Server Actions.
- **Cliente**: `import { useAuth, useUser } from '@clerk/nextjs'` en `'use client'`; comprobar `isLoaded` antes de leer.
- **Provider**: `ClerkProvider` (de `@clerk/nextjs`) va DENTRO de `<body>` en `app/layout.tsx`, NO envolviendo `<html>` (patron Core 3).
- **Enlace con la DB**: guardar el `userId` de Clerk como `clerkId` en la tabla `users` (ver `drizzle-sqlite`); sincronizar con webhooks de Clerk si hace falta.
- Requisitos: Clerk Core 3 pide Next >= 15.2.3 (16 OK) y Node >= 20.9.

## Entorno y config
- `.env.local`: `DATABASE_URL` (ruta del .db) y claves de Clerk (`NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`).
- `next.config.ts`: `serverExternalPackages: ['better-sqlite3']` (y `cacheComponents: true` si se usa `'use cache'`).
- Gestor de paquetes: pnpm.
