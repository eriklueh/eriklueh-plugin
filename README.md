# eriklueh-plugin

Marketplace + plugin de Claude Code con **skills propias y refinadas** para una app
**Next.js 16 + Clerk + Zustand + Drizzle/SQLite + shadcn/ui + Tailwind**.

Filosofia: aqui solo van skills refinadas por nosotros. Los recursos **oficiales**
(Clerk, shadcn, Vercel) se instalan aparte (ver abajo). Los **comunitarios** estan en
revision antes de decidir si se adoptan o se les copian ideas.

## Que incluye el plugin `eriklueh-plugin`
- `drizzle-sqlite` — convenciones de Drizzle ORM sobre SQLite (esquema, migraciones, server-only).
- `zustand-patterns` — patrones de stores Zustand (cuando si/no, selectores, SSR).
- `project-conventions` — estructura del proyecto, fronteras servidor/cliente, entorno.
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
- Skills propias: v0.1 (borradores refinados; ajustar a las convenciones reales del proyecto).
- Comunitarios: en revision (ver el informe que acompana este kit).
