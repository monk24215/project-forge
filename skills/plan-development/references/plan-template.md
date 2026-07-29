# Development plan template & worked example

## Output format

```markdown
# <Project> — Development Plan

**Goal:** <one sentence>
**Stack:** <as decided>
**Hard constraints:** <deadline, must-haves, non-goals>

## Phase 1 — Walking skeleton
**Goal:** <thinnest end-to-end slice>
**Scope (in order):**
1. ...
2. ...
**Demo / exit criteria:** <what you show> + <test that proves it>
**Out of scope:** ...

## Phase 2 — <capability>
... (same structure)

## Risks & unknowns
- <risk> → mitigated by <spike/phase>

## Open questions
- <anything the spec left ambiguous>
```

## Worked example — hosted magic-link auth service

**Goal:** Let a site owner paste a JS snippet so their visitors log in by email magic link.
**Stack:** Node 20 · Fastify · Next.js 15 · Drizzle/Postgres · Redis · Resend · pnpm monorepo.
**Hard constraints:** Snippet <15kb gzipped; no passwords; free tier.

### Phase 1 — Walking skeleton
**Goal:** One email → one magic link → one authenticated session, end to end, no UI polish.
**Scope (in order):**
1. `shared`: define `MagicLinkRequest` / `Session` Zod schemas + types.
2. `db`: `users`, `magic_tokens`, `sessions` tables + migration.
3. `api`: `POST /v1/request-link` (issues token, stores hash), `GET /v1/verify` (consumes token, sets session cookie).
4. Wire Resend to actually send the email in dev (or log the link if no key).
**Demo:** curl the request endpoint, click the link in the logged email, land on a page that reads the session.
**Exit test:** integration test covering request → verify → authenticated request.
**Out of scope:** dashboard, snippet embed, rate limiting, styling.

### Phase 2 — The embeddable snippet
**Goal:** A site owner drops one `<script>` and gets a working email form.
**Scope:**
1. `snippet`: form UI, calls `/v1/request-link`, handles states, size-budget check in build.
2. `api`: serve `/v1/snippet.js`; CORS + origin allowlist per site key.
3. `db`: `sites` table with public key + allowed origins.
**Demo:** paste the snippet on a throwaway HTML page, log in for real.
**Exit test:** snippet bundle under budget; origin-rejection test.
**Out of scope:** dashboard to manage sites (still hand-seeded).

### Phase 3 — Owner dashboard
**Goal:** Sign up, create a site, get your snippet + key, see basic usage.
**Scope:** `web` auth (dogfood the magic link), site CRUD, snippet copy view, usage counts.
**Demo:** new user self-serves a working embed with zero manual DB steps.
**Exit test:** e2e create-site → copy snippet → login flow.

### Phase 4 — Production hardening
**Goal:** Safe to run publicly.
**Scope:** rate limiting (Redis), token TTL + single-use enforcement, gitleaks in CI, Dockerfiles,
deploy config, structured logging.
**Demo:** deployed at the real domain, load-tested lightly, secrets scanned.

### Risks & unknowns
- Deliverability of magic emails → verify Resend domain + SPF/DKIM early (fold into Phase 1 spike).
- Snippet size budget vs. framework overhead → keep snippet dependency-free; check in Phase 2.

### Open questions
- Session length / refresh policy? (decide before Phase 1 schema.)
```
