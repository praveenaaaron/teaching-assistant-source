# Teaching Assistant

A personal teaching-assistant web app for a solo faculty member: manage a weekly timetable, lesson plans, a daily dashboard, a calendar, and monthly reports — no login required (single user).

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server
- `pnpm --filter @workspace/teaching-assistant run dev` — run the frontend
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec (`lib/api-spec/openapi.yaml`)
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite (`artifacts/teaching-assistant`)
- API: Express 5 (`artifacts/api-server`), mounted at `/api`
- DB: PostgreSQL + Drizzle ORM (`lib/db`)
- Validation: Zod, `drizzle-zod`
- API codegen: Orval (React Query hooks in `lib/api-client-react`, Zod schemas in `lib/api-zod`) generated from `lib/api-spec/openapi.yaml`
- Report export: `pdfkit` (PDF), `exceljs` (Excel)

## Where things live

- `lib/api-spec/openapi.yaml` — source of truth for API contracts (timetable, lesson-plans, dashboard, calendar, reports, notifications)
- `lib/db/src/schema/` — Drizzle schema: `timetable.ts` (timetableEntries), `lesson-plans.ts` (lessonPlans)
- `artifacts/api-server/src/routes/` — one file per resource, registered in `routes/index.ts`
- `artifacts/api-server/src/lib/dates.ts` — day-of-week / date helpers shared across routes
- `artifacts/teaching-assistant/src/` — frontend pages/components (Dashboard, Timetable, Lesson Plans, Calendar, Reports, Settings)

## Architecture decisions

- Original spec asked for Python Flask + SQLite; this project is a pnpm/TypeScript monorepo, so it was built with the platform-native stack instead (React+Vite, Express, Postgres/Drizzle) — same feature set, different framework. User was informed of this substitution.
- Lesson plan `date` fields are kept as raw `"YYYY-MM-DD"` strings end-to-end (request → DB) rather than round-tripped through `Date` objects, to avoid timezone-shift bugs from the generated Zod schemas' `zod.coerce.date()`.
- Dashboard "current/upcoming/past" class state is computed server-side from the day's timetable entries compared against server time.

## Product

- **Dashboard**: today's classes with current/upcoming highlighting, pending lesson plans, completed count, tomorrow's reminders and missing-lesson-plan alerts.
- **Timetable**: weekly (Mon–Sat) view with custom time slots; full CRUD.
- **Lesson Plans**: CRUD with search/filter by subject, class, date, unit, semester, department, status; optional link to a timetable entry.
- **Calendar**: month view with per-day lesson-plan density; day drill-down.
- **Reports**: monthly summary (classes handled, lesson plans completed, per-subject breakdown) with PDF/Excel export.

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- `pdfkit` needs its `data/` (font `.afm` files) directory copied next to the esbuild output — handled by a post-build step in `artifacts/api-server/build.mjs`. If PDF export starts failing with `ENOENT ... Helvetica.afm`, check that step still runs.
- `pdfkit`'s dependency `fontkit` requires `@swc/helpers` at the exact `^0.3.x` API (`applyDecoratedDescriptor`); newer `@swc/helpers` versions break it at runtime even though esbuild bundles fine.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
