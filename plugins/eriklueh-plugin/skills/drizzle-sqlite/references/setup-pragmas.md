# Conexion, PRAGMAs y singleton

## Cliente singleton (db/index.ts)
Drizzle no toca los PRAGMAs: se configuran en el handle nativo ANTES de envolver con `drizzle()`.

```ts
import Database from 'better-sqlite3'
import { drizzle } from 'drizzle-orm/better-sqlite3'
import * as schema from './schema'

const sqlite = new Database(process.env.DATABASE_URL ?? 'app.db')
sqlite.pragma('journal_mode = WAL')     // persiste en el header del archivo (una vez)
sqlite.pragma('foreign_keys = ON')      // POR CONEXION: SQLite lo trae OFF por defecto
sqlite.pragma('busy_timeout = 5000')    // POR CONEXION: espera en vez de fallar al instante
sqlite.pragma('synchronous = NORMAL')   // buen equilibrio durabilidad/rendimiento con WAL
sqlite.pragma('cache_size = -64000')    // ~64 MB de cache de paginas

export const db = drizzle(sqlite, { schema }) // `schema` es necesario para db.query (relational)
```

Gotcha clave: `journal_mode = WAL` queda grabado en el archivo, pero `foreign_keys` y `busy_timeout` son **por conexion** y hay que re-aplicarlos en cada nueva conexion.

## Singleton a prueba de hot-reload (Next dev)
El hot-reload de Next puede reabrir la conexion en cada recarga. Cachear en `globalThis`:

```ts
const g = globalThis as unknown as { __db?: ReturnType<typeof drizzle> }
export const db = g.__db ?? (g.__db = drizzle(sqlite, { schema }))
```

## next.config.ts
better-sqlite3 es un modulo nativo; hay que externalizarlo del bundle del servidor:

```ts
const nextConfig = {
  serverExternalPackages: ['better-sqlite3'],
}
export default nextConfig
```

## Entorno
`.env.local`: `DATABASE_URL=./app.db` (ruta al archivo SQLite). Anadir `*.db`, `*.db-wal`, `*.db-shm` al `.gitignore`. En backups incluir los tres archivos, o usar `VACUUM INTO`.
