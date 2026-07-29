# Root config templates

Drop-in templates for the workspace root. Replace `PROJECT_NAME` and adjust package lists to match
the actual scaffold.

## package.json (workspace root)

```json
{
  "name": "PROJECT_NAME",
  "private": true,
  "type": "module",
  "packageManager": "pnpm@9",
  "engines": { "node": ">=20" },
  "scripts": {
    "dev:api": "pnpm --filter @PROJECT_NAME/api dev",
    "dev:web": "pnpm --filter @PROJECT_NAME/web dev",
    "build": "pnpm -r build",
    "typecheck": "tsc -b --pretty",
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "test": "vitest run",
    "test:watch": "vitest"
  },
  "devDependencies": {
    "@biomejs/biome": "^1.9.0",
    "typescript": "^5.6.0",
    "vitest": "^2.1.0"
  }
}
```

## pnpm-workspace.yaml

```yaml
packages:
  - "packages/*"
```

## tsconfig.base.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "lib": ["ES2022"],
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "verbatimModuleSyntax": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "declaration": true,
    "sourceMap": true,
    "composite": true,
    "incremental": true
  }
}
```

## biome.json

```json
{
  "$schema": "https://biomejs.dev/schemas/1.9.0/schema.json",
  "organizeImports": { "enabled": true },
  "linter": {
    "enabled": true,
    "rules": { "recommended": true }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "files": {
    "ignore": ["dist", "node_modules", ".next", "coverage", "**/*.gen.ts"]
  }
}
```

## .nvmrc

```
20
```

## .npmrc

```
engine-strict=true
auto-install-peers=true
```

## .gitignore (essentials)

```
node_modules/
dist/
.next/
coverage/
*.tsbuildinfo
.env
.env.local
.DS_Store
```

## .env.example

List every environment variable the project reads, with placeholder (never real) values and a short
comment. This file is the contract for local setup and for whatever secret store production uses.

```
# Database
DATABASE_URL=postgres://user:pass@localhost:5432/PROJECT_NAME

# Cache / queue
REDIS_URL=redis://localhost:6379

# Email (transactional)
RESEND_API_KEY=re_xxx
EMAIL_FROM=noreply@example.com

# App
NODE_ENV=development
PUBLIC_BASE_URL=http://localhost:3000
SESSION_SECRET=change-me-32-chars-min
```

## .gitleaks.toml

```toml
title = "PROJECT_NAME gitleaks config"

[extend]
useDefault = true

[allowlist]
description = "Allow example env placeholders"
paths = ['''\.env\.example''']
regexes = ['''change-me''', '''re_xxx''', '''user:pass''']
```

## LICENSE

Default to MIT unless the user specifies otherwise. Fill the year and copyright holder.
