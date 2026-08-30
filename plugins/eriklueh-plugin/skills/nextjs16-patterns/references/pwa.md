# PWA en Next.js 16 (enfoque oficial)

Next.js tiene guia oficial de PWA y es CASI todo nativo (sin plugin). No usar `next-pwa` (obsoleto).

## 1. Manifest (nativo)
`app/manifest.ts` con `MetadataRoute.Manifest`:
```ts
import type { MetadataRoute } from "next";
export default function manifest(): MetadataRoute.Manifest {
  return {
    name: "App", short_name: "App", start_url: "/", display: "standalone",
    background_color: "#0a0a0a", theme_color: "#0a0a0a",
    icons: [{ src: "/icon-192x192.png", sizes: "192x192", type: "image/png" },
            { src: "/icon-512x512.png", sizes: "512x512", type: "image/png" }],
  };
}
```
Instalar en pantalla de inicio = manifest valido + HTTPS. Los navegadores muestran el prompt solos; en iOS se instruye "Compartir -> Anadir a pantalla de inicio".

## 2. Service worker (manual)
Crear `lib/service-worker.js` (listeners `push` y `notificationclick`) y registrarlo desde un Client Component:
```ts
await navigator.serviceWorker.register(
  new URL("../lib/service-worker.js", import.meta.url),
  { scope: "/", updateViaCache: "none" },
);
```
Esta forma (`new URL(..., import.meta.url)`) es la que documenta Next 16 y funciona con Turbopack.

## 3. Web Push (server actions + web-push)
- `pnpm add web-push` (+ `-D @types/web-push`). Es server-only.
- Claves VAPID: `npx web-push generate-vapid-keys` -> `NEXT_PUBLIC_VAPID_PUBLIC_KEY` + `VAPID_PRIVATE_KEY`.
- Server Actions `subscribeUser`/`unsubscribeUser`/`sendNotification`. `webpush.setVapidDetails(...)` antes de enviar.
- **Guardar la suscripcion en la DB** (con Drizzle: tabla `push_subscriptions` con endpoint unico, p256dh, auth, userId de Clerk). Al enviar, limpiar suscripciones con statusCode 404/410.
- Gatear con Clerk: `const { userId } = await auth()` dentro de la action.

## 4. Testing local
Push requiere HTTPS incluso en local: `next dev --experimental-https`.

## 5. Offline
No hay caché offline nativa. Opciones:
- Hook experimental `useOffline` (Next 16) + `experimental.useOffline` para UI consciente de conexion y reintentos.
- Para caché real via service worker: **Serwist** (`@serwist/next`), sucesor de next-pwa; tiene ejemplos oficiales para Turbopack y webpack.

## Gotchas
- No usar `next-pwa` (abandonado) en proyectos nuevos.
- El icono del manifest: usar PNG 192/512 (+ purpose maskable) para produccion; un SVG sirve para prototipar pero no todas las plataformas lo aceptan para install/notificaciones.
- El service worker registrado via `new URL(...)` NO queda en `/sw.js`; los headers de `next.config` para `/sw.js` no aplican a esa ruta.
