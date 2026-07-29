# CI workflow templates

## .github/workflows/ci.yml (fast gate)

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4

      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: pnpm

      - name: Install
        run: pnpm install --frozen-lockfile

      - name: Typecheck
        run: pnpm typecheck

      - name: Lint
        run: pnpm lint

      - name: Test
        run: pnpm test

  secret-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITLEAKS_CONFIG: .gitleaks.toml
```

## Integration-test job (service containers)

Add as a third job when tests need a real database/cache. Keep it separate so the fast gate stays fast.

```yaml
  integration:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: pass
          POSTGRES_DB: test
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready --health-interval 10s
          --health-timeout 5s --health-retries 5
      redis:
        image: redis:7
        ports: ["6379:6379"]
    env:
      DATABASE_URL: postgres://postgres:pass@localhost:5432/test
      REDIS_URL: redis://localhost:6379
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm --filter @PROJECT_NAME/db migrate
      - run: pnpm test:integration
```

## Optional pre-commit hook

Fast local check on staged files. Use with a git hooks manager (e.g. `simple-git-hooks` or `lefthook`).
Run lint + typecheck only; leave the full test suite to CI.

```yaml
# lefthook.yml
pre-commit:
  parallel: true
  commands:
    lint:
      run: pnpm biome check --staged --no-errors-on-unmatched
    types:
      run: pnpm typecheck
```

## Notes

- `node-version-file: .nvmrc` keeps CI and local Node versions in lockstep.
- The `cache: pnpm` option on setup-node handles store caching keyed on the lockfile.
- Pin action versions to major tags (`@v4`) and bump deliberately.
