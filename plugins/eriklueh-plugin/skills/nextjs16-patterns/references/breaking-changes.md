# Next.js 15 -> 16: cambios que rompen

Fuente: docs oficiales de upgrade a v16 (nextjs.org).

| Tema | Antes (14/15) | Ahora (16) |
|---|---|---|
| params / searchParams | sincronos (a veces) | **async**: `await params` / `await searchParams` |
| cookies() / headers() / draftMode() | podian ser sincronos | **async**: `await cookies()` |
| Archivo middleware | `middleware.ts`, `export function middleware` | `proxy.ts`, `export function proxy` (deprecado middleware; proxy = solo nodejs) |
| Rutas paralelas | fallback implicito | `default.tsx` obligatorio en cada slot (si no, build falla) |
| fetch caching | variable | dinamico por defecto; cache es opt-in (`'use cache'`) |
| Flags caching | `experimental.dynamicIO/useCache/ppr` | `cacheComponents: true` (top-level; los experimental fueron eliminados) |
| revalidateTag | `revalidateTag(tag)` | `revalidateTag(tag, 'max' | { expire })` (1 arg deprecado) |
| dynamic ssr:false | permitido casi donde sea | prohibido en Server Components |
| Runtime | webpack default | Turbopack default (usar `--webpack` para volver) |
| Config runtime | serverRuntimeConfig / publicRuntimeConfig | eliminados (usar env vars) |
| Lint | `next lint` | eliminado (usar ESLint/Biome directo) |
| Node | 18+ | **20.9.0+** |

## Firma de Page/Layout async
```tsx
// Manual
export default async function Page(
  { params, searchParams }: {
    params: Promise<{ slug: string }>
    searchParams: Promise<Record<string, string | string[] | undefined>>
  },
) {
  const { slug } = await params
  const query = await searchParams
}

// Recomendado: helpers generados con `npx next typegen`
export default async function Page(props: PageProps<'/blog/[slug]'>) {
  const { slug } = await props.params
}
```
`cookies()`/`headers()` se importan de `next/headers` y se await-ean. Codemods utiles: `npx @next/codemod@canary next-async-request-api .` y `... middleware-to-proxy .`.

## ssr:false correcto
`dynamic(() => import('./C'), { ssr: false })` solo dentro de un archivo `'use client'`. Si el padre es Server Component, envolver el `dynamic` en un wrapper client e importar ese wrapper.
