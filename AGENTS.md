# AGENTS.md

1. Tooling (expected): Turborepo + Bun (API: Hono), Web: Vite + TanStack Router, DB: Drizzle (Postgres), Auth: Better Auth, Tests: Vitest.
2. Install: `bun install` (root). Generate Drizzle SQL: `bun drizzle:generate`; run migrations: `bun drizzle:migrate`.
3. Common scripts (add when package.json exists): `dev` (concurrent web+api), `build` (turbo run build), `lint` (biome or eslint), `test` (vitest run), `typecheck` (tsc --noEmit).
4. Single test file: `bun vitest run path/to/file.test.ts`; single test name: `bun vitest run -t "name substring"`.
5. Lint: prefer Biome (fast) or ESLint+@typescript-eslint; fix: `bun lint --fix`.
6. Formatting: Prettier/Biome defaults; 2-space indent; semicolons on; single quotes; import order: (node builtin) -> third-party -> internal alias (@core/*) -> relative.
7. TypeScript: strict true; no implicit any; use `unknown` over `any`; narrow early; prefer `type` aliases for shapes, `interface` only for object extension.
8. Naming: files kebab-case (web) or snake_case for SQL; React components PascalCase; constants SCREAMING_SNAKE_CASE; migrations timestamp-name.ts.
9. Imports: absolute via configured path aliases (e.g. @api/, @web/, @db/). No deep relative chains (../../..). Re-export barrel only when it reduces churn.
10. Errors: never throw raw strings; create domain Error subclasses (e.g. InviteLimitError) or use `new Error(message,{cause})`; log unexpected with Sentry capture.
11. API handlers: validate input (zod) at edge; return typed objects; never leak stack traces; map domain errors to 4xx codes; default 500 fallback JSON { error: "internal_error" }.
12. DB: drizzle schema central; no inline SQL in handlers; soft-delete respected in queries; always scope workspace resources by workspace_id + membership check.
13. Realtime: emit events after successful commit; never before transaction finishes.
14. Security: enforce auth + workspace membership; strip EXIF on image upload; size/type guard before upload; rate limit mutation endpoints (Redis key pattern user:{id}:{action}).
15. Testing: fast unit (pure functions), integration (API + ephemeral DB), e2e (playwright later). Mark slow tests with .slow. Keep unit tests deterministic (no real time.sleep).
16. Single test debugging: `bun vitest --ui` optionally; watch mode: `bun vitest`.
17. Commits: conventional style (feat:, fix:, chore:, refactor:, docs:, test:); focus on intent (why). No secrets (.env *.local) committed.
18. Env vars (placeholders): DATABASE_URL, REDIS_URL, SENTRY_DSN, SMTP_HOST/PORT/USER/PASS, AUTH_GOOGLE_CLIENT_ID/SECRET, AUTH_MICROSOFT_CLIENT_ID/SECRET, LICENSE_KEY_FLAG.
19. File layout (target): apps/api, apps/web, packages/db (drizzle), packages/auth, packages/ui, infra/ (deployment). Add a root turbo.json when scaffolding.
20. Agents: keep edits minimal & atomic; if script absent, add with placeholder and comment; prefer creating missing config over guessing hidden behavior.
