---
name: scaffold-project
description: >
  Scaffold a new production-grade TypeScript project or monorepo from scratch. Use when the
  user says "start a new project", "scaffold a monorepo", "set up a pnpm workspace", "bootstrap
  a TypeScript service", "create a new repo structure", or asks how to lay out packages, tooling,
  and config for a fresh codebase. Produces the workspace layout, root config files, and package
  skeletons — not application logic.
---

# Scaffold Project

Stand up a new TypeScript project with a workspace layout, strict tooling, and the config files a
production repo needs on day one. This skill encodes a monorepo pattern proven on shipped services:
pnpm workspaces, strict TypeScript, Biome, Vitest, Docker per-service, and a clean package boundary
between API, web, shared types, and data layer.

## When to use vs. adjacent skills

- Use **this skill** to create the physical structure — directories, root configs, package skeletons.
- Use **define-package** to add or specify one package inside an existing workspace.
- Use **wire-ci** after scaffolding to add the CI workflow and pre-commit gate.
- Use **plan-development** before scaffolding when the user wants a phased build plan first.
- Use **write-project-docs** to generate the CLAUDE.md / CONTRIBUTING / README triad.

## Decide the shape first

Before creating files, settle three things. Ask only what the user hasn't already stated.

1. **Single package or monorepo?** Monorepo when there are ≥2 deployable/publishable units that
   share types (e.g. an API + a web app + a browser snippet). Single package otherwise. Default to
   monorepo only when a shared boundary genuinely exists — don't split prematurely.
2. **What are the packages?** Name each and its role. A typical web service monorepo:
   `api` (server), `web` (dashboard/public pages), `shared` (types + validation schemas), `db`
   (ORM schema + migrations), and optionally a client artifact like `snippet` or `sdk`.
3. **Runtime targets.** Node version (pin in `.nvmrc`), package manager (default pnpm), and which
   packages ship as Docker images vs. deploy from source.

## Reference stack

This is the default stack the pattern assumes. Substitute freely, but keep the *shape*: strict
types, a shared schema package, one linter/formatter, one test runner, per-service Dockerfiles.

Node 20 · TypeScript strict · pnpm workspaces · Biome (lint + format) · Vitest · Postgres + an ORM
(Drizzle) · Redis where a cache/queue is needed · a transactional email provider where auth flows exist.

## Build order

Create files in this order so each references the last correctly. Full file contents and templates
are in `references/`.

1. **Root config** — `package.json` (workspace root, `private: true`, shared scripts),
   `pnpm-workspace.yaml`, `tsconfig.base.json`, `biome.json`, `.nvmrc`, `.npmrc`,
   `.gitignore`, `.gitattributes`, `.dockerignore`, `.env.example`, `LICENSE`.
   Templates: `references/root-config.md`.
2. **Package skeletons** — for each package: `packages/<name>/package.json`,
   `tsconfig.json` (extends base), `src/index.ts`, and a `vitest` setup where the package is tested.
   Naming and boundary rules: `references/package-layout.md`.
3. **Per-service Docker** — one `Dockerfile.<service>` per deployable package at repo root
   (e.g. `Dockerfile.api`, `Dockerfile.web`, `Dockerfile.cron`). Template: `references/docker.md`.
4. **Secret hygiene** — `.gitleaks.toml` and a `.env.example` that lists every var with placeholder
   values. Never commit real secrets. See `references/root-config.md`.

## Root scripts contract

The workspace root `package.json` exposes these scripts, delegating into packages. Downstream skills
(wire-ci, plan-development) assume these names exist:

```
dev:api / dev:web    # run a single package in watch mode
build                # build all packages
typecheck            # strict tsc across all packages, no emit
lint                 # biome check
test                 # vitest across all packages
```

Keep the trio `typecheck` / `lint` / `test` as the canonical quality gate — every other skill and
the CI workflow reference exactly these three.

## Guardrails

- **Strict from the start.** `tsconfig.base.json` sets `strict: true` and `noUncheckedIndexedAccess`.
  Retrofitting strictness later is expensive; pay it upfront.
- **One source of shared truth.** Cross-package types and validation schemas live in the `shared`
  package and are imported, never duplicated. If two packages define the same shape, that's a defect.
- **Don't scaffold empty ceremony.** Only create a package when it has a real reason to exist. Five
  half-empty packages are worse than two full ones.
- **Pin versions.** `.nvmrc` for Node, a committed lockfile, exact-ish ranges for critical deps.

After scaffolding, hand off: suggest running **wire-ci** to add the quality gate and **write-project-docs**
to generate the docs triad, unless the user asked for structure only.
