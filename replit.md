# Edusolve — School Student Grievance Portal

A full-stack school grievance management system where students submit complaints and the principal manages, tracks, and resolves them.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080, proxied at `/api`)
- `pnpm --filter @workspace/grievance-portal run dev` — run the frontend (port from `$PORT`, proxied at `/`)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string, `SESSION_SECRET` — JWT signing secret

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite + Tailwind CSS v4 + shadcn/ui
- API: Express 5 + Pino logger
- DB: PostgreSQL + Drizzle ORM
- Auth: JWT (stored in localStorage as `grievance_token`)
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec → React Query hooks)
- Charts: Recharts
- Build: esbuild (CJS bundle for server)

## Where things live

- `artifacts/api-server/src/routes/` — Express route handlers (auth, complaints, notifications, analytics, students)
- `artifacts/api-server/src/middlewares/auth.ts` — JWT middleware (requireAuth, requirePrincipal)
- `artifacts/api-server/src/db/schema.ts` — Drizzle schema (users, complaints, timeline_events, notifications)
- `artifacts/api-server/uploads/` — File upload storage (multer)
- `artifacts/grievance-portal/src/pages/student/` — Student-facing pages
- `artifacts/grievance-portal/src/pages/principal/` — Principal-facing pages
- `artifacts/grievance-portal/src/components/auth-provider.tsx` — JWT auth context
- `artifacts/grievance-portal/src/components/layout.tsx` — Sidebar navigation
- `lib/api-spec/openapi.yaml` — Source of truth for API contract
- `lib/api-client-react/src/generated/` — Generated React Query hooks (do not edit manually)

## Architecture decisions

- Contract-first API: OpenAPI spec → Orval codegen → typed React Query hooks; never write fetch calls by hand
- JWT in localStorage (not cookies) because this is a demo school intranet app without CSRF concerns
- Anonymous complaints: principal sees `studentName`, `studentId`, `studentClass` as `null` when `isAnonymous=true`
- File uploads handled by multer server-side; files served at `/api/complaints/files/:filename`
- Timeline events auto-created on every complaint status change via DB trigger in route handler
- Notifications auto-sent to student on status change or principal reply

## Product

**Students** can: register, log in, submit grievances (with category, attachments, anonymous mode, draft saving), track complaint status with timeline, receive notifications on updates, view/reply in detail, and manage their profile.

**Principal** can: view all complaints in a filterable/searchable table, update status + assign + send replies, see analytics (by category, status, monthly trends, resolution rate), manage the student list, export complaints as CSV.

## Demo Credentials

- Principal: `principal@school.edu` / `principal123`
- Student 1: `aryan@student.edu` / `student123`
- Student 2: `priya@student.edu` / `student123`

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Run `pnpm --filter @workspace/api-spec run codegen` after any OpenAPI spec change before editing frontend code
- The principal account must have `role = 'principal'` set directly in DB (registration always creates students)
- Generated hooks use `{ query: { ... } as any }` for `enabled`/`refetchInterval` options due to Orval strict queryKey requirement
- Do NOT run `pnpm dev` at workspace root — use workflows or `pnpm --filter` scoped commands

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
