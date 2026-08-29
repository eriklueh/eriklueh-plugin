# eriklueh-plugin

Marketplace + plugin de Claude Code con **skills propias y refinadas** para una app
**Next.js 16 + Clerk + Zustand + Drizzle/SQLite + shadcn/ui + Tailwind**.

Filosofia: aqui solo van skills refinadas por nosotros. Los recursos **oficiales**
(Clerk, shadcn, Vercel) se instalan aparte (ver abajo). Los **comunitarios** estan en
revision antes de decidir si se adoptan o se les copian ideas.

## Que incluye el plugin `eriklueh-plugin`
Cada skill es un `SKILL.md` indice + carpeta `references/` (progressive disclosure: el detalle se carga solo cuando hace falta).

- `drizzle-sqlite` — Drizzle ORM sobre SQLite/better-sqlite3: conexion+PRAGMAs, queries/transacciones/migraciones, checklist de revision.
- `zustand-patterns` — Zustand v5: store-per-request en App Router, slices/selectores/middleware, gotchas async.
- `nextjs16-patterns` — Next.js 16: breaking changes 15->16, caching (cacheComponents/revalidateTag/updateTag), errores comunes.
- `shadcn-tailwind-theming` — shadcn/ui + Tailwind v4 en Next (PostCSS, no Vite): setup, theming OKLCH/dark mode, gotchas.
- `project-conventions` — indice del stack + fronteras server/cliente + integracion Clerk (Core 3: proxy.ts, ClerkProvider en body, auth() async).
- `.mcp.json` (opcional) — conecta el **MCP de shadcn** (instalar componentes del registry) y el
  **MCP de Clerk** (snippets del SDK al dia). Borralo si prefieres configurar los MCP a nivel de proyecto.

## Instalacion (para quien reciba el kit)
```
/plugin marketplace add eriklueh/eriklueh-plugin
/plugin install eriklueh-plugin@eriklueh-kit
```
Ajusta `eriklueh/eriklueh-plugin` a tu usuario/repo real de GitHub. El sufijo
`@eriklueh-kit` es el `name` declarado en `.claude-plugin/marketplace.json`.

## Recursos oficiales (instalar por separado, NO viven aqui)
- Clerk: `npx skills add clerk/skills` (o `--skill clerk-nextjs-patterns`) + `clerk mcp install`.
- shadcn/ui: `pnpm dlx skills add shadcn/ui` (el MCP ya va en `.mcp.json`).
- Next.js (Vercel): `npx plugins add vercel/vercel-plugin`.

## Notas de desarrollo
- Validar antes de publicar: `claude plugin validate`.
- El schema exacto de `marketplace.json` / `plugin.json` puede cambiar; ver
  https://code.claude.com/docs/en/plugin-marketplaces
- Transporte del MCP de Clerk: verificar en https://clerk.com/docs. Si la config manual de
  `.mcp.json` da problemas, usar `clerk mcp install` (auto-configura el cliente).

## Estado
- Skills propias: v0.2 — contenido verificado contra docs oficiales (Next 16, Tailwind/shadcn v4, Drizzle, Clerk Core 3) el 2026-08-29. Ajustar a las convenciones reales del proyecto segun evolucione.
- Comunitarios: revisados; ninguno se instala tal cual (todos "adapt"), sus mejores ideas ya estan integradas y corregidas en estas skills.
