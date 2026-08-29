# Errores comunes de Next.js 16 (error -> causa -> fix)

1. **"params/searchParams should be awaited" / propiedades undefined**
   Causa: acceso sincrono (habito de Next 14/15). Fix: `const { id } = await params`. Tipar como `Promise<...>`. Codemod: `npx @next/codemod@canary next-async-request-api .`.

2. **"auth() was called but Clerk can't detect usage of clerkMiddleware()"**
   Causa: falta `proxy.ts` (Next 16) con `clerkMiddleware()`, o la ruta quedo fuera del `matcher`. Fix: crear `proxy.ts` e incluir `'/__clerk/(.*)'` y `'/(api|trpc)(.*)'` en el matcher. Ver skill `project-conventions`.

3. **Build falla en rutas paralelas ("@slot matched but no default")**
   Causa: falta `default.tsx` en un slot. Fix: crear `app/@slot/default.tsx` que retorne `null` o llame a `notFound()`.

4. **"ssr: false is not allowed with next/dynamic in Server Components"**
   Causa: `dynamic(..., { ssr:false })` en un Server Component. Fix: moverlo a un archivo `'use client'` (wrapper).

5. **"Route used cookies()/headers() inside 'use cache'" o build colgado**
   Causa: leer APIs de request dentro de un scope cacheado. Fix: leerlas fuera y pasarlas como argumentos a la funcion `'use cache'`.

6. **`revalidateTag` da error de TypeScript**
   Causa: llamada con 1 solo argumento (deprecada en 16). Fix: `revalidateTag(tag, 'max')` o `revalidateTag(tag, { expire: 0 })`.

7. **El build usa webpack y una config custom lo rompe**
   Causa: Turbopack es el default en Next 16. Fix: migrar la config a Turbopack, o `next build --webpack` como puente temporal.

8. **Errores de Node/tipos al actualizar**
   Causa: Node 18 / React types viejos. Fix: Node >= 20.9, TS >= 5.1, actualizar `@types/react` y `@types/react-dom` a React 19.2.
