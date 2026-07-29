---
name: wire-ci
description: >
  Add continuous integration and a local quality gate to a TypeScript project. Use when the user says
  "set up CI", "add a GitHub Actions workflow", "wire up the typecheck/lint/test gate", "add pre-commit
  checks", "add secret scanning", or asks how to keep the build green automatically. Produces the CI
  workflow, the canonical quality-gate scripts, and secret-scanning config — matched to the project's
  existing package manager and scripts.
---

# Wire CI

Install the automated quality gate: the same three checks locally and in CI, plus secret scanning, so
nothing merges that breaks types, lint, or tests. This skill assumes the root scripts contract from
**scaffold-project** (`typecheck`, `lint`, `test`). If those scripts don't exist yet, create them first.

## The canonical gate

Exactly three commands, same names everywhere:

```
pnpm typecheck   # strict tsc across all packages, no emit
pnpm lint        # biome check (or eslint) — no writes in CI
pnpm test        # vitest run
```

Everything below runs these three. Do not invent per-environment variations — the value of the gate is
that green locally means green in CI.

## What to create

1. **CI workflow** — `.github/workflows/ci.yml`. Runs on push and PR. Installs with a frozen lockfile,
   restores the pnpm store cache, runs the three gate commands. Template: `references/ci-workflow.md`.
2. **Secret scanning** — a gitleaks step in CI plus `.gitleaks.toml` at repo root (from scaffold-project).
   Fail the build on a leak. Template: `references/ci-workflow.md`.
3. **Optional pre-commit** — a lightweight local hook that runs `lint` and `typecheck` on staged files
   so problems are caught before push. Keep it fast; put the full test run in CI, not the hook.

## Principles

- **Frozen lockfile in CI.** `pnpm install --frozen-lockfile` — CI must fail if the lockfile is stale,
  never silently resolve new versions.
- **Cache the store, not node_modules.** Cache pnpm's content-addressable store keyed on the lockfile hash.
- **One job, sequential gate, fail fast.** Typecheck → lint → test in one job. A separate matrix is
  overkill for most projects; add it only when you truly target multiple Node versions.
- **Secret scanning is not optional.** A leaked key in git history is expensive to remediate. Scan every
  push, allowlist only documented placeholders in `.gitleaks.toml`.
- **Don't gate on formatting separately.** Biome check covers lint + format drift in one pass.

## Sequencing with other checks

If the project has integration tests that need a database, run them in a second job that spins up a
Postgres/Redis service container — keep them out of the fast unit-test gate so PR feedback stays quick.
See `references/ci-workflow.md` for the service-container variant.

After wiring CI, confirm the gate passes locally once (`pnpm typecheck && pnpm lint && pnpm test`) before
telling the user it's ready — a workflow that's never been run green is unverified.
