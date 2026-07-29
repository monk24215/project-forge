# project-forge

A project-development plugin for Claude Code. It encodes a proven methodology for standing up and
shipping production TypeScript projects: monorepo scaffolding, phased planning, an automated quality
gate, per-service Docker, and the docs triad that keeps humans and agents oriented.

The patterns are abstracted from real shipped services (pnpm workspaces, strict TypeScript, Fastify +
Next.js + Drizzle, Biome, Vitest) but the skills are stack-shaped, not stack-locked — the *shape*
(strict types, a shared-schema boundary, one linter, one test runner, a three-command gate) transfers
to any TypeScript project.

## Skills

| Skill | Use it to |
|-------|-----------|
| `scaffold-project` | Create a new monorepo/workspace — layout, root config, package skeletons, Dockerfiles |
| `define-package` | Add one package to an existing workspace with the right dependency boundary |
| `plan-development` | Turn an idea or spec into a phased, demonstrable build plan (walking skeleton first) |
| `wire-ci` | Add the `typecheck`/`lint`/`test` gate, GitHub Actions CI, and secret scanning |
| `write-project-docs` | Generate the README + CLAUDE.md + CONTRIBUTING triad, matched to the real repo |

## Install

```
/plugin marketplace add monk24215/project-forge
/plugin install project-forge@project-forge
```

Or add it to an existing marketplace repo (see below).

## The methodology in one paragraph

Plan in phases where phase one is a walking skeleton that exercises every layer end-to-end; scaffold a
workspace with strict TypeScript and a single shared-schema package that nothing duplicates; keep one
canonical quality gate — `pnpm typecheck && pnpm lint && pnpm test` — green at the end of every phase and
enforced in CI with secret scanning; and maintain a docs triad where the README is the human front door,
CLAUDE.md is the agent's working context, and CONTRIBUTING is the operational guide, all three kept
mutually consistent.

## License

MIT
