# Package layout & boundary rules

## Canonical web-service monorepo

```
PROJECT_NAME/
├── .github/workflows/        # CI (see wire-ci skill)
├── docs/                     # CONTRIBUTING.md, architecture notes
├── packages/
│   ├── api/                  # server (Fastify/Express/Hono) — deployable
│   ├── web/                  # Next.js dashboard + public pages — deployable
│   ├── snippet/              # optional browser artifact, size-budgeted
│   ├── db/                   # ORM schema + migrations
│   └── shared/               # TS types + Zod schemas — imported by all
├── scripts/                  # repo automation
├── Dockerfile.api
├── Dockerfile.web
├── CLAUDE.md                 # agent working context (see write-project-docs)
├── README.md
├── package.json
├── pnpm-workspace.yaml
├── tsconfig.base.json
└── biome.json
```

## Package naming

- Scope every package: `@PROJECT_NAME/api`, `@PROJECT_NAME/shared`, etc.
- Directory name = unscoped package name, kebab-case.
- `shared` and `db` are libraries (`private: true`, no build output shipped as an image).
- `api`, `web`, and any client artifact are deployables.

## Per-package skeleton

`packages/<name>/package.json`:

```json
{
  "name": "@PROJECT_NAME/<name>",
  "version": "0.0.0",
  "private": true,
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "scripts": {
    "build": "tsc -b",
    "dev": "tsc -b -w",
    "test": "vitest run"
  }
}
```

`packages/<name>/tsconfig.json`:

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src"]
}
```

## Boundary rules (enforce these)

1. **Dependencies flow one direction.** `shared` depends on nothing internal. `db` may depend on
   `shared`. `api` and `web` depend on `shared` and `db`. Nothing depends on `api` or `web`.
2. **No duplicated shapes.** Any type or validation schema used by more than one package lives in
   `shared`. Duplicating a shape across packages is a defect, not a shortcut.
3. **The client artifact has a size budget.** A browser snippet/SDK declares a gzipped ceiling
   (e.g. <15kb) and CI or a build step checks it. It imports types from `shared` but bundles them out.
4. **Migrations live only in `db`.** No other package writes schema. `api` imports the query layer.

## When NOT to split

A single-package project is correct when there's one deployable and no shared-type boundary. Don't
create `shared` just to have it — introduce it the moment a second consumer of a type appears.
