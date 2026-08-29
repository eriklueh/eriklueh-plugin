---
name: drizzle-sqlite
description: Convenciones de Drizzle ORM sobre SQLite (better-sqlite3) en esta app Next.js 16. Usar al definir el esquema, generar o aplicar migraciones, escribir queries o transacciones, o configurar la conexion a la base de datos.
---

# Drizzle ORM + SQLite (better-sqlite3)

Skill indice. El detalle vive en `references/`; cargalo solo cuando haga falta.

## Reglas de oro (memorizar)
- **better-sqlite3 es SINCRONO**: nada de `await` en queries ni transacciones. Terminadores: `.all()` (filas), `.get()` (una fila), `.run()` (insert/update/delete), `.returning()` (fila afectada).
- **Server-only**: better-sqlite3 es un addon nativo de Node. Instanciar el cliente UNA sola vez (singleton) y usarlo solo en Server Components / Route Handlers / Server Actions. Nunca en `'use client'` ni en runtime edge.
- **Drizzle NO configura PRAGMAs**: hay que activarlos a mano al abrir la conexion (WAL, foreign_keys, busy_timeout...). Ver `references/setup-pragmas.md`.
- **Escrituras**: envolver en transaccion; usar `behavior: 'immediate'` para escritores (evita `SQLITE_BUSY`). La funcion de transaccion DEBE ser sincrona (no devolver promesa).
- **Tipos**: exportar `typeof tabla.$inferSelect` / `$inferInsert` y reusarlos en server actions y stores de Zustand.

## Cuando cargar cada reference
| Necesitas... | Lee |
|---|---|
| Crear la conexion, WAL/PRAGMAs, singleton, next.config | `references/setup-pragmas.md` |
| Esquema, queries, upserts, transacciones, migraciones, drizzle-kit | `references/queries-transactions-migrations.md` |
| Revisar codigo de DB (checklist anti-bugs) | `references/red-flags.md` |
