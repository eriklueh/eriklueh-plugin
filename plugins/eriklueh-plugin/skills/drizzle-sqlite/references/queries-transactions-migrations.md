# Esquema, queries, transacciones y migraciones

## Esquema (db/schema.ts)
El 3er parametro de `sqliteTable` (indices/constraints) es una **funcion que devuelve un ARRAY** (la forma de objeto quedo deprecada).

```ts
import { sqliteTable, integer, text, index, unique } from 'drizzle-orm/sqlite-core'

export const users = sqliteTable('users', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  clerkId: text('clerk_id').notNull().unique(),   // enlaza con Clerk (ver project-conventions)
  email: text('email').notNull(),
  prefs: text('prefs', { mode: 'json' }).$type<{ theme: 'light' | 'dark' }>(),
  createdAt: integer('created_at', { mode: 'timestamp' }).notNull(), // Date <-> epoch s
}, (t) => [
  index('users_email_idx').on(t.email),
])

// Tipos derivados: usarlos en server actions y stores
export type SelectUser = typeof users.$inferSelect
export type InsertUser = typeof users.$inferInsert
```

Modos de columna: `integer({ mode: 'timestamp' })` (Date en s) o `'timestamp_ms'`; `integer({ mode: 'boolean' })`; `text({ mode: 'json' }).$type<T>()`.

Nota: **STRICT tables NO existen** en el builder de Drizzle. Si las quieres, edita el SQL de la migracion generada (`CREATE TABLE ... STRICT`).

## Queries (SIN await)
```ts
const all = db.select().from(users).all()
const one = db.select().from(users).where(eq(users.id, 1)).get()
db.insert(users).values(v).run()
const created = db.insert(users).values(v).returning().get()
db.update(users).set({ email }).where(eq(users.id, 1)).run()
db.delete(users).where(eq(users.id, 1)).run()
```

## Upsert (SQLite)
Los valores entrantes se referencian con `sql\`excluded.col\``. (`onDuplicateKeyUpdate` es de MySQL, NO SQLite.)

```ts
db.insert(users)
  .values({ clerkId, email })
  .onConflictDoUpdate({ target: users.clerkId, set: { email: sql\`excluded.email\` } })
  .run()
```

## Transacciones (sincronas)
```ts
const result = db.transaction((tx) => {
  tx.insert(users).values(v).run()
  return tx.select().from(users).all()
}, { behavior: 'immediate' })   // 'deferred' (default) | 'immediate' | 'exclusive'
```
Usa `behavior: 'immediate'` para cualquier transaccion que ESCRIBE: adquiere el lock desde el inicio y evita `SQLITE_BUSY` al "subir" de lectura a escritura. La funcion NO puede ser async ni devolver una promesa. Batchear muchas escrituras en una sola transaccion es ~100x mas rapido que insertarlas sueltas.

## Migraciones (2 tiempos)
`drizzle.config.ts`:
```ts
import { defineConfig } from 'drizzle-kit'
export default defineConfig({
  dialect: 'sqlite',                    // NO `driver` (forma vieja)
  schema: './db/schema.ts',
  out: './drizzle',
  dbCredentials: { url: process.env.DATABASE_URL ?? './app.db' }, // `url`, no connectionString
})
```
- `npx drizzle-kit generate` -> escribe SQL en `./drizzle`. Revisar el SQL antes de aplicar.
- `npx drizzle-kit migrate` (CLI) o programatico:
  ```ts
  import { migrate } from 'drizzle-orm/better-sqlite3/migrator'
  migrate(db, { migrationsFolder: './drizzle' })   // sincrono
  ```
- `npx drizzle-kit push` para prototipar sin archivos de migracion; `npx drizzle-kit studio` para inspeccionar.
