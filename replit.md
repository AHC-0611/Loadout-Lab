# Loadout Lab

A community web app where players post their loadout builds and search others' builds for upgrade ideas.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/airsoft-armory run dev` — run the frontend (port 20706)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string
- Required env: `CLERK_SECRET_KEY`, `CLERK_PUBLISHABLE_KEY`, `VITE_CLERK_PUBLISHABLE_KEY` — Clerk auth (auto-provisioned)

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React 19 + Vite 7, Tailwind v4, shadcn/ui, wouter, TanStack Query
- API: Express 5
- Auth: Replit-managed Clerk (`@clerk/express` server-side, `@clerk/react` client-side)
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — OpenAPI contract (source of truth for all endpoints)
- `lib/db/src/schema/builds.ts` — builds table (gun name, category, description, totalCost)
- `lib/db/src/schema/upgrades.ts` — upgrades table (name, price, source, FK to builds)
- `artifacts/api-server/src/routes/builds.ts` — all build/upgrade API routes
- `artifacts/api-server/src/middlewares/clerkProxyMiddleware.ts` — Clerk proxy middleware
- `artifacts/airsoft-armory/src/App.tsx` — Clerk + wouter routing wiring
- `artifacts/airsoft-armory/src/pages/` — all page components

## Architecture decisions

- Clerk auth is cookie-based on web — no Bearer tokens, no `setAuthTokenGetter` in frontend code
- Orval generates `zod.int()` (Zod v4 syntax) but the workspace pins `zod@3` — the orval config has an `afterAllFilesWrite` hook that patches `from 'zod'` → `from 'zod/v4'` in the generated Zod schema file
- Routes `/builds/stats`, `/builds/categories`, and `/builds/my` are registered before `/builds/:id` to prevent the param route from swallowing them
- `totalCost` is stored as `numeric` in Postgres and serialized as a JS `number` (via `parseFloat`) in route handlers

## Product

- **Feed** — scrollable build cards with gun name, description, upgrade cost, category, poster
- **Search** — text search + category + price range filters across all builds
- **Post a Build** — form with dynamic upgrade entries (name, price, source) — requires sign-in
- **Build Detail** — full upgrade list, description, delete for owner
- **My Builds** — authenticated user's own builds

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Always run `pnpm run typecheck:libs` after changing `lib/*` packages; leaf artifact typechecks fail otherwise with "no exported member" errors
- After any OpenAPI spec change, re-run codegen before touching routes or frontend
- The orval `afterAllFilesWrite` hook targets only `lib/api-zod/src/generated/api.ts` — directing it at a directory causes a sed error (exit 4) but the hook still patches the file correctly

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
- See the `clerk-auth` skill for Clerk setup and customization
