# CLAUDE.md template

Fill against the real repo. Delete sections that don't apply. Written as directives for an agent.

```markdown
# CLAUDE.md

## What this is
<Two sentences: what the project does and who it's for.>
Production: <url if any>

## Architecture
Monorepo, pnpm workspaces. Packages:

- `packages/shared` — TS types + Zod schemas. Depends on nothing internal. Imported everywhere.
- `packages/db` — ORM schema + migrations. May import `shared`. **All schema changes live here.**
- `packages/api` — <server framework>. Depends on `shared` + `db`.
- `packages/web` — <web framework>. Depends on `shared` + `db`.
- `packages/<client>` — <artifact>. Size budget: <Nkb gzipped>.

Dependency direction is one-way: `shared` → `db` → (`api`, `web`). Nothing imports `api` or `web`.
If two packages define the same shape, move it to `shared` — duplication is a defect.

## Quality gate (run before considering any change done)
    pnpm typecheck   # strict tsc, all packages
    pnpm lint        # biome
    pnpm test        # vitest
All three must pass. `shared` is imported everywhere — after changing it, run `typecheck` on the whole repo.

## Common tasks
- Run one service: `pnpm dev:api` / `pnpm dev:web`
- Add a migration: <exact command>, then commit the generated file under `packages/db`.
- Build an image: `docker build -f Dockerfile.api .`

## Conventions
- TypeScript strict + `noUncheckedIndexedAccess`. No `any` without a written reason.
- Validation at boundaries with Zod schemas from `shared`.
- Env vars: read from a validated config module; add new ones to `.env.example` in the same PR.
- Secrets never in git. `.gitleaks.toml` allowlists only documented placeholders.

## Landmines
- <non-obvious gotcha that has caused problems>
- <thing not to touch without understanding X>
```
