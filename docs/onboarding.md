# Onboarding — Agap Alerto

> Get productive in the first few minutes. Points to canonical docs; does not restate them.

## What this is
A same-day SMS/web-lite lookup and alert that tells Manila residents exposed to floodwater which of
the city's 51 free-doxycycline sites is open, stocked, and near them right now — closing the gap
between the city's general epidemic information campaign and an individual's own exposure window.
See `seed/idea.md` for the full problem and `seed/validation.md` for the independent verdict
(`GO-UNVALIDATED`).

## Read first
- `docs/index.md` §0 — which doc owns which fact (read this first)
- `docs/prd.md` — what we're building (F-001/F-002/F-003, MVP)
- `docs/system-design.md` — how it's built (SMS gateway, FastAPI core, Postgres store)
- `docs/implementation-plan.md` — `PH-##` phases and live `TASK-###` work
- `docs/harness.md` — the sensors that will catch you and the guides that steer you

## Run it locally
<!-- Greenfield project — no code exists yet as of this doc suite's generation. This is the intended
     setup once PH-01/TASK-001 (skeleton) lands; update this section with the real commands at that
     point rather than treating this as already-true. -->
1. Prereqs: Python 3.12+, PostgreSQL 15+ (or a local container), an SMS gateway sandbox credential
   (`system-design.md` Integration points).
2. Install: `pip install -r requirements.txt` (once `TASK-001` creates it).
3. Run: `uvicorn app.main:app --reload` (once the skeleton exists).
4. Verify: `pytest -m fast` should show the seeded test suite; `curl localhost:8000/health` should
   return 200 once `TASK-001` lands.

## Where things live
- `seed/` — the Key's problem brief, context, market, brand, validation, usability (input, not
  edited by the Vault).
- `docs/` — this generated suite (PRD, system design, data model, QA plan, security, design system,
  ops, gtm, frd, technical design, decision ledger, implementation plan, crew, harness, handoff
  packets, index).
- `docs/crew.md` — the build-time agent roster for this product.
- (Application code paths are named as they're created — `TASK-001`'s write scope in
  `implementation-plan.md` is the first source of truth for where `app/`, `tests/`, etc. land.)

## Conventions
If `docs/implementation-plan.md` has an open phase: claim one `ready` `TASK-###`, use branch
`task/TASK-###-short-slug`, get your handoff packet from `docs/handoff/TASK-###.md`, confirm its
Verify command is red on the base commit, stay inside its write scope, never edit a test, return
red/green evidence, and let the keeper write Status. No direct shared-branch work. Full protocol in
`AGENTS.md` (emitted alongside this suite).
