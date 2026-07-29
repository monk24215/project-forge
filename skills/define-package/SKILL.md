---
name: define-package
description: >
  Add or specify a single package inside an existing pnpm/TypeScript workspace. Use when the user says
  "add a package", "create a new workspace package", "I need a shared/db/api/worker package", "add an
  SDK package", or asks how to introduce a new deployable or library into the monorepo. Produces the
  package skeleton, wires it into the workspace, and states its dependency boundary — one package, done
  right, without disturbing the others.
---

# Define Package

Introduce one new package into an existing workspace correctly: right role, right boundary, wired into
the build, without touching packages outside scope. Use this rather than **scaffold-project** when the
workspace already exists and only needs one more unit.

## First, justify it

A new package earns its place only if it has a distinct role and consumer. Before creating one, confirm:

- **What it is** — deployable (api, web, worker) or library (shared types, db, sdk, config)?
- **Who consumes it** — which existing packages will import it, or does it deploy standalone?
- **Why not fold it in** — if an existing package could hold this without straining its boundary, do that
  instead. Don't split for tidiness.

If it's a shared type or schema that a second package now needs, the answer is usually "put it in the
existing `shared` package," not "make a new package."

## Determine the boundary

Place the new package in the one-directional dependency graph and state it explicitly:

- **Libraries** sit low: `shared` (imports nothing internal) < `db`/`config` (may import `shared`).
- **Deployables** sit high: `api`, `web`, `worker` import libraries; nothing imports them.
- A new package must not create a cycle. If it needs to import something that imports it, the boundary
  is wrong — extract the shared piece into a lower library.

## Create the skeleton

1. `packages/<name>/package.json` — scoped name `@PROJECT/<name>`, `private: true` for libraries,
   `type: module`, standard `build`/`dev`/`test` scripts. (Skeleton in scaffold-project's
   `references/package-layout.md`.)
2. `packages/<name>/tsconfig.json` — extends `../../tsconfig.base.json`, sets `outDir`/`rootDir`.
3. `packages/<name>/src/index.ts` — the public entry; export only what other packages should import.
4. Add internal dependencies with the workspace protocol: `"@PROJECT/shared": "workspace:*"`.
5. If deployable, add a root `dev:<name>` script and a `Dockerfile.<name>` (see scaffold-project's
   `references/docker.md`).

## Wire it in

- Run the install so pnpm links the workspace dependency (`pnpm install`).
- Confirm `pnpm typecheck` still passes across the whole repo — a new package must not break the build.
- If the package is tested, ensure `pnpm test` picks it up.
- Update **CLAUDE.md** and the README package table to list the new package and its boundary (hand off
  to write-project-docs, or do it inline if small).

## Package-type quick reference

- **shared library** — types + Zod schemas, no runtime deps, imported by everyone. There should be exactly one.
- **db package** — schema + migrations only; the single source of schema truth.
- **api/worker deployable** — server or scheduled process; owns its Dockerfile.
- **web deployable** — framework app; owns its build output and Dockerfile.
- **sdk/snippet artifact** — client-facing bundle with a declared size budget; imports types from `shared`
  but bundles them out.

## Guardrails

- Touch only the new package and the minimal wiring (root scripts, workspace file, docs). Do not refactor
  sibling packages while adding one — that's a separate task.
- Don't duplicate a type into the new package that already exists in `shared`; import it.
- Keep the new package's public surface (`index.ts` exports) small and intentional.
