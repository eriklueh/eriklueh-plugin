# Theming (v4) y gotchas

## Contrato de color CSS-first
1. Tokens crudos como valores OKLCH en `:root` y `.dark` (alpha: `oklch(1 0 0 / 10%)`).
2. Mapeo en `@theme inline` con `var()` directo, SIN funcion envolvente:
   ```css
   @theme inline { --color-primary: var(--primary); }
   ```
3. Dark mode automatico via `@custom-variant dark (&:is(.dark *))` + los tokens de `.dark`. Los utilities semanticos (`bg-primary`, `text-foreground`) cambian solos: **prohibido** `dark:` variants y pares hardcodeados como `bg-blue-600 dark:bg-blue-400`.

## Gotchas (verificados contra docs oficiales)
- **NO doble-wrap**: en v4 el token ya contiene el color (OKLCH); referenciar `var(--background)`, nunca `hsl(var(--background))`. El doble-wrap del patron v3 rompe los colores.
- **`@apply` NO esta deprecado**: funciona en `globals.css`. En un CSS separado/scoped (CSS Module, o un .css que no importe Tailwind) hay que anadir arriba `@reference "../app/globals.css";` (o `@reference "tailwindcss";`) o la utilidad no resuelve. Por defecto, preferir clases de utilidad en el JSX sobre `@apply`.
- **Animaciones**: `@import "tw-animate-css";` + instalar `tw-animate-css`. NO `@plugin 'tailwindcss-animate';` ni instalar `tailwindcss-animate` (deprecado) en proyectos v4.
- **Nada de v3**: no emitir `@tailwind base; @tailwind components; @tailwind utilities;` (en v4 es `@import "tailwindcss";`), ni arrays `content: [...]`, ni tema en JS.
- **`:root`/`.dark` fuera de `@layer base`**: van a nivel raiz; v4 puede descartar CSS fuera de `@theme`/`@layer`. No anidar `@theme` dentro de `.dark`.
- **`data-slot`**: cada primitivo de shadcn v4 expone `data-slot` (p.ej. `[data-slot=accordion-item]`); usarlo para overrides en vez de selectores anidados fragiles.

## Sintoma -> causa
| Sintoma | Causa probable |
|---|---|
| `bg-primary` "no existe" | falta la entrada en `@theme inline` que mapea `--color-primary` |
| Colores rotos/invalidos | doble-wrap `hsl(var(--x))` (patron v3) |
| `@apply` no aplica en un CSS Module | falta `@reference` al inicio del archivo |
| Animaciones de shadcn no funcionan | falta `@import "tw-animate-css";` o se instalo el paquete deprecado |
