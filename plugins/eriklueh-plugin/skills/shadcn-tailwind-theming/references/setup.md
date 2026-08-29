# Setup de Tailwind v4 + shadcn/ui en Next.js 16

## 1. Tailwind v4 (PostCSS)
```bash
pnpm add tailwindcss @tailwindcss/postcss postcss
```
`postcss.config.mjs`:
```js
const config = { plugins: { '@tailwindcss/postcss': {} } }
export default config
```
NO usar `@tailwindcss/vite` ni un plugin en `next.config`. En v4 no se crea `tailwind.config.js`: la deteccion de contenido es automatica y el tema va en CSS.

## 2. globals.css (orden para shadcn v4)
```css
@import "tailwindcss";
@import "tw-animate-css";
@custom-variant dark (&:is(.dark *));

:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  /* ...resto de tokens en OKLCH... */
}
.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
}

@layer base {
  * { @apply border-border outline-ring/50; }
  body { @apply bg-background text-foreground; }
}
```

## 3. shadcn/ui
```bash
pnpm dlx shadcn@latest init
```
En `components.json` (v4): `"tailwind": { "config": "", "css": "app/globals.css" }`. Dejar `config` vacio.

Deps que instala/usa: `class-variance-authority`, `clsx`, `tailwind-merge`, `lucide-react`, `tw-animate-css`.

`lib/utils.ts` (convencion cn, dejar tal cual):
```ts
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"
export function cn(...inputs: ClassValue[]) { return twMerge(clsx(inputs)) }
```

Para anadir componentes: usar el MCP de shadcn (p.ej. "add button, dialog, card") o `pnpm dlx shadcn@latest add <componente>`. El estilo por defecto (new-york, etc.) puede cambiar entre versiones del CLI: leer el `components.json` generado en vez de asumirlo.
