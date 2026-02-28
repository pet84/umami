# Agents

## Cursor Cloud specific instructions

### Project overview

Umami is a privacy-focused web analytics platform built as a single Next.js 15 application with React 19, TypeScript, and Prisma ORM. It runs on port 3000.

### Required services

| Service | How to start | Notes |
|---|---|---|
| PostgreSQL 15 | `sudo docker start umami-postgres` (if container exists) or `sudo docker run -d --name umami-postgres -e POSTGRES_DB=umami -e POSTGRES_USER=umami -e POSTGRES_PASSWORD=umami -p 5432:5432 postgres:15-alpine` | Must be running before the app starts |
| Next.js dev server | `pnpm dev` | Runs on http://localhost:3000, default login: `admin` / `umami` |

### Environment variables

A `.env` file at the repo root is required with at minimum:
```
DATABASE_URL=postgresql://umami:umami@localhost:5432/umami
APP_SECRET=dev-secret-change-me-in-production
```

### Database setup (one-time after fresh install)

After `pnpm install`, the database layer must be initialized before running the dev server:
```
pnpm run copy-db-files    # copies PostgreSQL prisma schema
pnpm run build-db-client  # generates Prisma client
pnpm run check-db         # connects to DB and runs migrations
```

### Build steps needed before dev server

The tracker script and GeoIP database must be built before the dev server works correctly:
```
pnpm run build-tracker   # builds public/script.js
pnpm run build-geo       # downloads GeoLite2-City.mmdb to geo/
```

### Key commands

- **Lint**: `pnpm run lint` — runs `next lint --quiet`. Note: there are 3 pre-existing lint errors (not blockers).
- **Test**: `pnpm run test` — runs Jest (3 test suites, 18 tests).
- **Dev server**: `pnpm dev` — starts Next.js dev server on port 3000.
- **Full build**: `pnpm run build` — runs check-env, build-db, check-db, build-tracker, build-geo, and next build.

### Gotchas

- The `prisma/` directory is generated (not committed) — it's copied from `db/postgresql/` or `db/mysql/` by `copy-db-files`. If it's missing, run `pnpm run copy-db-files` before `pnpm run build-db-client`.
- Docker must be running with fuse-overlayfs storage driver and iptables-legacy in this cloud environment.
- The `pnpm-workspace.yaml` has `onlyBuiltDependencies` configured for Prisma packages, so `pnpm install` is non-interactive.
- The pre-commit hook runs `lint-staged` (prettier + eslint on JS/TS, stylelint + prettier on CSS, prettier on JSON).
