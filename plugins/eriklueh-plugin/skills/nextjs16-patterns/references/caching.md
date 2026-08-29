# Caching e invalidacion en Next.js 16

Modelo: **dinamico por defecto**. Cachear es explicito y opt-in.

## Activar Cache Components
`'use cache'` NO funciona suelto: requiere el flag top-level (ya no experimental):
```ts
// next.config.ts
const nextConfig = { cacheComponents: true }
export default nextConfig
```
Reemplaza a `experimental.dynamicIO` / `experimental.useCache` / `experimental.ppr` (eliminados). Requiere runtime Node.js. Al activarlo, el modelo pasa a "shell estatico + datos dinamicos dentro de `<Suspense>`"; puede romper el build con errores de "uncached data outside <Suspense>" hasta adaptar.

## 'use cache' + cacheLife/cacheTag (estables, sin unstable_)
```ts
import { cacheLife, cacheTag } from 'next/cache'

async function getPosts() {
  'use cache'
  cacheLife('hours')
  cacheTag('posts')
  return db.select().from(posts).all()
}
```
GOTCHA: NO leer `cookies()`/`headers()`/`searchParams` (ni helpers que los lean) dentro de un scope `'use cache'` — lanza `next-request-in-use-cache` o cuelga el build. Leerlos fuera y pasarlos como argumentos.

## Invalidacion
- `revalidateTag(tag, 'max')` — SIEMPRE 2 argumentos. `'max'` = stale-while-revalidate largo (~1 año). La forma de 1 arg esta deprecada, da error de TS y se comporta como `{ expire: 0 }`. Solo en Server Functions / Route Handlers.
- `revalidateTag(tag, { expire: 0 })` — expiracion inmediata (miss bloqueante). Usar desde webhooks / Route Handlers externos.
- `updateTag(tag)` — SOLO en Server Actions. Semantica read-your-writes: expira y refresca los datos en la MISMA request, para que el usuario vea su cambio al instante (forms, settings).
- `refresh()` (de `next/cache`) — NO es invalidacion de datos: refresca el router del CLIENTE desde un Server Action. No confundir con revalidateTag/updateTag.

## Regla practica
- Mutacion en un form/Server Action y el usuario debe ver el cambio ya -> `updateTag`.
- Cambio que puede tardar en propagarse (cron, webhook) -> `revalidateTag(tag, 'max')` o `{ expire: 0 }` si debe ser inmediato.
