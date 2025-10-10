# AGENTS.md

1. Monorepo: Turborepo + Bun workspaces (apps/_, packages/_); Node >= 18.
2. Install deps: run `bun install` at repo root (packageManager is bun).
3. Dev: web `bun -C apps/web run dev` (Vite: http://localhost:3000); api `bun -C apps/server run dev` (Wrangler).
4. Build: root `bun run build` (turbo); per-app: `bun -C apps/web run build`.
5. Deploy (server): `bun -C apps/server run deploy`; typegen: `bun -C apps/server run cf-typegen`.
6. Lint: root `bun run lint` (turbo -> per package). Example: `bun -C packages/ui run lint`.
7. Types: root `bun run check-types` (turbo cascades); UI package uses `tsc --noEmit`.
8. Tests (web): `bun -C apps/web run test`; single file: `bun -C apps/web vitest run path/to.test.ts`.
9. Single test name: `bun -C apps/web vitest run -t "name substring"`; UI: no tests yet.
10. Format: root `bun run format`; web quick fix `bun -C apps/web run check` (prettier+eslint --fix).
11. API: Cloudflare Workers using Hono (`apps/server/src/index.ts`); config `apps/server/wrangler.jsonc`.
12. Web: Vite + React 19 + TanStack Router/Query/Form/Table; Tailwind v4 plugin; env helper `apps/web/src/env.ts`.
13. TypeScript: strict true; bundler resolution; web alias `@/*` -> `src/*`; server JSX via `hono/jsx`.
14. Imports: prefer absolute `@/*` (web); avoid deep `../../..`; order: node -> third-party -> internal -> relative.
15. Formatting: Prettier (semi: false, singleQuote: true, trailingComma: all); 2-space indent.
16. Naming: React components PascalCase; files kebab-case (web); constants SCREAMING_SNAKE_CASE.
17. Types: avoid `any`; prefer `unknown` then narrow; use `type` for shapes, `interface` for extension.
18. Errors: throw `Error`/custom classes; never raw strings; surface safe API messages only.
19. Cursor rules: see `apps/web/.cursorrules` (shadcn usage via `pnpx shadcn@latest add <component>`).
20. Commits: Conventional (`feat:`, `fix:`, etc.); no secrets; keep changes minimal and atomic.
