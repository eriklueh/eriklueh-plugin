---
name: drizzle-sqlite
description: Convenciones de Drizzle ORM sobre SQLite (better-sqlite3) en esta app Next.js 16. Usar al definir el esquema, generar o aplicar migraciones, o escribir queries a la base de datos.
---

# Drizzle ORM + SQLite — convenciones del proyecto

## Stack
- `drizzle-orm` + `better-sqlite3` (archivo local, driver sincrono).
- Migraciones con `drizzle-kit`.
- Variante remota (si algun dia pasan a Turso): `@libsql/client` en lugar de better-sqlite3.

## Estructura
- Esquema en `db/schema.ts`:
  ```ts
  import { sqliteTable, integer, text } from 'drizzle-orm/sqlite-core'

  export const users = sqliteTable('users', {
    id: integer('id').primaryKey({ autoIncrement: true }),
    clerkId: text('clerk_id').notNull().unique(),
    email: text('email').notNull(),
    createdAt: integer('created_at', { mode: 'timestamp' }).notNull(),
  })
  ```
- Cliente singleton en `db/index.ts`:
  ```ts
  import Database from 'better-sqlite3'
  import { drizzle } from 'drizzle-orm/better-sqlite3'
  import * as schema from './schema'

  const sqlite = new Database(process.env.DATABASE_URL ?? 'app.db')
  sqlite.pragma('journal_mode = WAL')
  sqlite.pragma('foreign_keys = ON')

  export const db = drizzle(sqlite, { schema })
  ```
- `drizzle.config.ts`:
  ```ts
  import { defineConfig } from 'drizzle-kit'

  export default defineConfig({
    dialect: 'sqlite',
    schema: './db/schema.ts',
    out: './db/migrations',
    dbCredentials: { url: process.env.DATABASE_URL ?? 'app.db' },
  })
  ```

## Migraciones (nunca editar el SQL a mano)
- Editar `db/schema.ts` -> `npx drizzle-kit generate` (crea SQL en `db/migrations/`).
- Aplicar: `npx drizzle-kit migrate`, o `migrate()` de `drizzle-orm/better-sqlite3/migrator` al arrancar.
- `npx drizzle-kit studio` para inspeccionar la base de datos.

## Reglas
- Queries tipadas (`db.select().from(...)`, `db.insert(...).values(...)`). SQL crudo solo en casos puntuales con el helper `sql` de drizzle-orm.
- Escrituras multiples atomicas -> `db.transaction((tx) => { ... })`.
- Server-only: better-sqlite3 es un modulo nativo; nunca importar `db/` desde componentes 'use client'. Usar solo en Server Components / Route Handlers / Server Actions.
- `next.config.ts` (Next 16): `serverExternalPackages: ['better-sqlite3']`.
- Timestamps como `integer({ mode: 'timestamp' })`. Ids con `autoIncrement`, o `text` (uuid) segun la convencion del proyecto.
- El enlace con Clerk se hace guardando `clerkId` (el user id de Clerk) como clave, no duplicando datos de auth.
