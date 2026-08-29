# Checklist de revision (Drizzle + SQLite)

Correr esta lista al revisar codigo de base de datos.

- [ ] **Nada de `await`** en queries/transacciones de better-sqlite3 (es sincrono). Un `await` sobre un builder "funciona" por ser thenable, pero suele indicar que se copio el driver equivocado (Postgres/libSQL/D1).
- [ ] La conexion aplica PRAGMAs: `journal_mode = WAL`, `foreign_keys = ON`, `busy_timeout`. Sin `foreign_keys = ON`, las FK no se aplican (bug silencioso).
- [ ] El cliente es **singleton** y solo se importa en codigo servidor (no `'use client'`, no edge).
- [ ] Transacciones que escriben usan `behavior: 'immediate'`.
- [ ] La funcion de transaccion es sincrona (no async, no devuelve promesa).
- [ ] Toda escritura multi-paso va dentro de una transaccion.
- [ ] Columnas usadas en WHERE/JOIN/ORDER BY tienen indice; las FK tienen indice.
- [ ] Listas paginadas: preferir cursor (`where(gt(t.id, lastId)).limit(n)`) sobre OFFSET en tablas grandes.
- [ ] `select()` en tablas grandes selecciona columnas explicitas, no todo.
- [ ] Nada de SQL construido por concatenacion de input del usuario; usar el query builder o `sql\`\`` con placeholders.
- [ ] N+1: no consultar hijos en un loop; usar `leftJoin` o relational queries `db.query.x.findMany({ with: {...} })`.
- [ ] Upsert con `onConflictDoUpdate` + `sql\`excluded.col\``, no `onDuplicateKeyUpdate` (MySQL).
- [ ] Tipos derivados del esquema (`$inferSelect`/`$inferInsert`), no interfaces duplicadas a mano.
