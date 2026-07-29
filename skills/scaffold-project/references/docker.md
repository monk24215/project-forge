# Per-service Dockerfiles

One `Dockerfile.<service>` per deployable package, at repo root. Multi-stage: install with the full
workspace (so pnpm can resolve internal deps), prune to the target package, run as non-root.

## Dockerfile.api (Node service)

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-slim AS base
ENV PNPM_HOME="/pnpm" PATH="/pnpm:$PATH"
RUN corepack enable
WORKDIR /app

FROM base AS build
COPY . .
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile
RUN pnpm --filter @PROJECT_NAME/api... build
RUN pnpm --filter @PROJECT_NAME/api deploy --prod /prod/api

FROM base AS runtime
ENV NODE_ENV=production
COPY --from=build /prod/api /app
USER node
EXPOSE 3001
CMD ["node", "dist/index.js"]
```

## Dockerfile.web (Next.js)

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-slim AS base
ENV PNPM_HOME="/pnpm" PATH="/pnpm:$PATH"
RUN corepack enable
WORKDIR /app

FROM base AS build
COPY . .
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile
RUN pnpm --filter @PROJECT_NAME/web... build

FROM base AS runtime
ENV NODE_ENV=production
COPY --from=build /app/packages/web/.next/standalone /app
COPY --from=build /app/packages/web/.next/static /app/packages/web/.next/static
COPY --from=build /app/packages/web/public /app/packages/web/public
USER node
EXPOSE 3000
CMD ["node", "packages/web/server.js"]
```

## .dockerignore

```
node_modules
**/node_modules
**/dist
**/.next
.git
.env
.env.local
coverage
*.md
```

## Notes

- The `...` filter suffix (`@PROJECT_NAME/api...`) builds the package and its internal dependencies.
- `pnpm deploy --prod` produces a self-contained folder with only production deps — small runtime image.
- For a scheduled worker, add `Dockerfile.cron` with the same build stage and a CMD that runs the
  scheduler entrypoint.
