---
name: project-conventions
description: Convenciones generales del stack de esta app (Next.js 16 App Router + Clerk + Zustand + Drizzle/SQLite + shadcn/ui + Tailwind). Usar al crear features, decidir donde va el codigo, o configurar el proyecto.
---

# Convenciones del proyecto

## Stack
- Next.js 16 (App Router, Server Components por defecto).
- Auth: Clerk (para patrones al dia, apoyarse en el skill/MCP oficial de Clerk).
- Estado de cliente: Zustand (ver skill `zustand-patterns`).
- Datos: Drizzle ORM sobre SQLite (ver skill `drizzle-sqlite`).
- UI: shadcn/ui + Tailwind (instalar componentes con el MCP de shadcn).

## Estructura de carpetas
- `app/` rutas (App Router).
- `components/ui/` componentes de shadcn; `components/` componentes propios.
- `lib/` utilidades; `db/` esquema, cliente y migraciones de Drizzle; `stores/` stores de Zustand.

## Fronteras servidor / cliente
- Todo lo que toque `db/` o secretos es server-only: Server Components, Route Handlers, Server Actions. Nunca importar `db/` en componentes `'use client'`.
- Zustand y los hooks de cliente de Clerk (`useUser`, `useAuth`) van en componentes `'use client'`.

## Auth con Clerk (resumen)
- `clerkMiddleware` en `middleware.ts` (verificar `proxy.ts` en Next 16 segun version del SDK).
- Proteger Server Actions / Route Handlers con `auth()`; en cliente usar `useAuth()` / `useUser()`.
- Enlazar el usuario de Clerk con la DB guardando `clerkId` en la tabla `users` (ver `drizzle-sqlite`). Mantener sincronia con webhooks de Clerk si hace falta.
- Los patrones concretos y actualizados: apoyarse en el skill/MCP oficial de Clerk.

## Entorno y config
- `.env.local`: `DATABASE_URL` (ruta del archivo SQLite) y las claves de Clerk.
- `next.config.ts`: `serverExternalPackages: ['better-sqlite3']`.
- Gestor de paquetes: pnpm.
