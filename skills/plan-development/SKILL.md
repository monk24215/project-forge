---
name: plan-development
description: >
  Turn a project idea or feature request into a phased, buildable development plan. Use when the user
  says "plan this project", "break this into phases", "what's the build order", "create a roadmap for
  this feature", "how should we sequence this", or hands over a spec and asks where to start. Produces
  a milestone-based plan with a walking-skeleton first phase, explicit exit criteria per phase, and a
  dependency-ordered task list — not code.
---

# Plan Development

Convert a spec or idea into a phased plan that can actually be executed in order. The output is a
document a developer (or an agent) can follow milestone by milestone, where each phase ends in
something demonstrable and the quality gate stays green throughout.

## Core principles

1. **Walking skeleton first.** Phase 1 is always the thinnest end-to-end slice that exercises every
   layer — request in, through the real API, real DB write, real response out — even if it does almost
   nothing. Prove the architecture works before adding features to it.
2. **Every phase is demonstrable.** A phase that can't be shown to someone at its end is too big or
   badly cut. Define the demo up front; it *is* the exit criterion.
3. **Depth-order by dependency, not by layer.** Don't build "all the models, then all the endpoints,
   then all the UI." Build one vertical feature fully, then the next. Horizontal layering hides
   integration risk until the end.
4. **The gate is non-negotiable.** `typecheck`, `lint`, and `test` pass at the end of every phase.
   A phase isn't done if the gate is red.

## Workflow

### 1. Extract the spec

Restate the goal in one sentence and list the hard constraints (deadline, stack, must-have surfaces,
non-goals). If a real spec exists, pull acceptance criteria from it verbatim rather than inventing.
Do not embellish or add scope the user didn't state.

### 2. Identify the layers touched

For a typical service: data schema, API surface, business logic, client/UI, integrations (email,
payments, auth), deploy. Note which the project actually needs — skip what it doesn't.

### 3. Cut phases

Produce 3–6 phases. Phase 1 is the walking skeleton. Each later phase adds one coherent capability.
Use the template in `references/plan-template.md`. For each phase specify:

- **Goal** — one sentence.
- **Scope** — the specific tasks, dependency-ordered.
- **Demo / exit criteria** — what you can show, and what test proves it.
- **Out of scope** — what deliberately waits, to prevent scope creep.

### 4. Surface risks and unknowns

List the parts most likely to break the plan: unproven integrations, ambiguous requirements, external
dependencies. Front-load a spike into an early phase for anything genuinely unknown — don't let a
risky unknown sit in the last phase.

### 5. Sequence and hand off

Present the plan as a numbered phase list with a short dependency note where phases aren't strictly
linear. Offer to scaffold (scaffold-project) if the repo doesn't exist yet, or to start Phase 1.

## Phase-sizing heuristics

- If a phase's task list exceeds ~8 items, split it.
- If you can't name the demo in one sentence, the phase is unfocused.
- If two phases both touch the same module heavily, consider whether they're really one phase.
- The first phase should be completable fast — momentum matters more than completeness early.

## Anti-patterns to avoid

- **Big-bang integration.** Wiring everything together only at the end. The walking skeleton exists
  precisely to kill this.
- **Layer-complete phases.** "Finish the whole data model" as a phase — it's not demonstrable and it
  defers integration risk.
- **Unbounded research phases.** Time-box spikes; a spike's exit criterion is a decision, not perfection.
- **Planning past the horizon.** Detail the next 1–2 phases precisely; keep later phases coarse. Replan
  as you learn.

See `references/plan-template.md` for the output format and a worked example.
