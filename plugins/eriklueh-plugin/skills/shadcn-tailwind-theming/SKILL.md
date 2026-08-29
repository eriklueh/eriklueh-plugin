---
name: shadcn-tailwind-theming
description: Setup y theming de shadcn/ui con Tailwind CSS v4 en Next.js 16 (PostCSS, no Vite). Usar al configurar Tailwind/shadcn, definir tokens de color y dark mode, o depurar por que un color o utilidad no aparece.
---

# shadcn/ui + Tailwind v4 en Next.js 16

Skill indice; el detalle vive en `references/`. Para INSTALAR componentes del registry usar el MCP de shadcn (`.mcp.json`); esta skill cubre el setup y el theming.

## Reglas de oro
- **Next usa PostCSS, no Vite**: plugin `@tailwindcss/postcss` en `postcss.config.mjs`. Nunca `@tailwindcss/vite` (eso es de tutoriales de Vite).
- **CSS-first**: en v4 no hay `tailwind.config.js`; el tema vive en `@theme inline` dentro de `globals.css`. Una sola linea `@import "tailwindcss";` (no el triple `@tailwind base/components/utilities` de v3).
- **Tokens en OKLCH** en `:root`/`.dark`; en `@theme inline` se referencian con `var()` **sin envolver** (nada de `hsl(var(--x))`).
- **`components.json`**: `tailwind.config: ""` (vacio) en v4.
- **Animaciones**: `tw-animate-css` (importado como CSS), NO `tailwindcss-animate` (deprecado).
- `@apply` **sigue vigente** en v4 (no esta deprecado). Solo requiere `@reference` en CSS separado/scoped.
- `cn()` = `clsx` + `tailwind-merge`.

## Cuando cargar cada reference
| Necesitas... | Lee |
|---|---|
| Instalar/configurar Tailwind v4 + shadcn en Next (postcss, globals.css, components.json, deps) | `references/setup.md` |
| Definir colores/dark mode y depurar theming, mas gotchas | `references/theming-and-gotchas.md` |
