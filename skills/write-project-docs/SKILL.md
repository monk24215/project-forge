---
name: write-project-docs
description: >
  Generate the core documentation triad for a project — CLAUDE.md (agent working context), a README,
  and a CONTRIBUTING guide. Use when the user says "write the README", "create a CLAUDE.md", "add a
  contributing guide", "document this project", or "set up the docs" for a repo. Produces documentation
  matched to the actual repo structure, package list, and scripts — not generic boilerplate.
---

# Write Project Docs

Produce the three documents every serious repo needs, each with a distinct job:

- **README.md** — the front door for a human evaluating or using the project.
- **CLAUDE.md** — persistent working context for an AI agent operating in the repo.
- **docs/CONTRIBUTING.md** — how to set up, develop, and get a change merged.

Do not generate generic filler. Read the actual repo (or the scaffold just created) and describe *this*
project: its real packages, real scripts, real stack. If a fact isn't known, ask or omit — never invent
setup steps that don't match the code.

## README.md

Keep it scannable and honest. Sections, in order:

1. **Title + one-line description** — what it is, in a sentence a stranger understands.
2. **Production URL / status** — where it runs, if deployed.
3. **Packages table** — for a monorepo: each package and its one-line role. Skip for single-package.
4. **Stack** — the actual technologies, as a terse list.
5. **Quickstart** — the literal commands to run it locally, copy-pasteable:
   ```
   pnpm install
   cp .env.example .env
   pnpm dev:api
   pnpm dev:web
   ```
6. **CI checks** — the gate commands, so contributors know what must pass.
7. **License.**

Rules: no marketing language, no aspirational features that don't exist, every command must actually
work against the current code. Link to CONTRIBUTING for depth rather than bloating the README.

## CLAUDE.md

This is instructions and context *for an agent*, not prose for a human. It's the highest-leverage doc
because it shapes every future AI session in the repo. Include:

1. **What the project is** — two sentences of orientation.
2. **Architecture map** — the packages and how they depend on each other; the one-directional boundary
   rules (what may import what).
3. **The quality gate** — the exact `typecheck` / `lint` / `test` commands; the rule that all three
   pass before any change is considered done.
4. **Conventions that matter** — strict TypeScript, shared types live in `shared`, migrations only in
   `db`, size budgets on client artifacts, whatever this repo actually enforces.
5. **What to run for common tasks** — dev servers, adding a migration, building an image.
6. **Landmines** — anything non-obvious that has bitten people; things not to touch without care.

Write it as directives ("Run `pnpm typecheck` after any change to `shared` — it's imported everywhere"),
not description. See `references/claude-md-template.md`.

## docs/CONTRIBUTING.md

The operational guide. Sections:

1. **Prerequisites** — Node version (from `.nvmrc`), pnpm, any services (Postgres, Redis) and how to
   start them (docker compose snippet if relevant).
2. **Setup** — clone, install, env file, first run.
3. **Development workflow** — branch naming, how to run a single package, how to add a package.
4. **Before you push** — run the gate; run integration tests if touching data/API.
5. **Commit / PR conventions** — message format, what CI enforces, review expectations.
6. **Adding a migration / a package** — the exact steps for the two most common structural changes.

## Cross-document consistency

The three docs must agree. The gate commands, package names, and setup steps appear in more than one
place — they must be identical. When you change one, check the others. A README that says `npm test`
and a CLAUDE.md that says `pnpm test` is a bug.

Generate all three in one pass so they stay consistent, then present them. Offer to place them at their
canonical paths (`README.md` and `CLAUDE.md` at root, `CONTRIBUTING.md` under `docs/`).
