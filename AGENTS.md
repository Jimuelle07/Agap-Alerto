# Agap Alerto — Agent Guide

## Project overview
Agap Alerto is a same-day SMS/web-lite lookup and alert that tells Manila residents exposed to
floodwater which of the city's 51 free-doxycycline sites is open, stocked, and near them right now.
It serves informal workers and market vendors in flood-prone Manila barangays who don't know which
site is open before their prophylaxis window closes.

## 30-second orientation (new here?)
You need `/docs` and this file — not the framework internals.
- **`F-###` = a feature; `INV-###` = a hard must-never guardrail** (holds across every change).
  Tie each change to an `F-###`; never breach an `INV-###`. PRD + QA plan own the details.
- **Acceptance criteria and tests are EARS sentences** — `WHEN <trigger>, the system SHALL
  <result>`; guardrails are `the system SHALL NEVER <X>`. Write yours the same way.
- **`docs/index.md §0` says which doc owns which fact.** Link to the owner; never restate.
- **Spec first, build second, tests deferred.** Before coding, write the exact signature/data
  shape/edge cases the task needs (SDD). Build against it, verify, and the task's `TC-###` lands
  in its own commit at phase end, covering that spec — never invented from scratch there. If
  asked to make a test pass by editing the test, the answer is no — return `[blocked: test]`.

## Architecture
FastAPI core service + PostgreSQL (Site/Status Store) + a Philippine SMS gateway (`[assumption]`,
vendor TBD) for resident SMS in/out; a thin staff web-lite UI for stock/status updates.
See [System Design](./docs/system-design.md).

## Build & run
```
# Greenfield — no code exists yet. TASK-001 (docs/handoff/TASK-001.md) creates the skeleton and the
# real setup commands belong here once it lands. Do not treat the commands below as already true.
pip install -r requirements.txt
uvicorn app.main:app --reload
```

## Test
```
pytest -m fast        # per task, < ~60s
pytest                # phase exit, on the default branch
```
Nothing is done until the named test is green on the current base.

## Living plan · handoff · SDD → TDD protocol
Read [`docs/implementation-plan.md`](./docs/implementation-plan.md) before coding. Phases
(`PH-##`) are demoable stopping points; tasks (`TASK-###`) live inside them. It is the only
execution-state file. Roles and their models: [`box/vault/roster.json`](./box/vault/roster.json)
(this environment's resolved binding is Claude-only — see `.vault-work/roster.resolved.json`).

- **Claim work:** pick a `ready` task in the `open` phase (currently `PH-01`; `TASK-001` is
  `ready`). Branch `task/TASK-###-slug` from the default branch. One task, one owner, one branch.
- **Get your packet:** `docs/handoff/TASK-###.md` — the planner writes it; it must pass
  `python3 box/vault/tools/check-handoff.py docs/handoff/TASK-###.md --plan docs/implementation-plan.md --qa docs/qa-test-plan.md`.
  The packet + your write scope + the Verify command (must fail on the base commit) are your
  whole context.
- **Spec, then build:** state the SDD spec — signature(s), data shape(s), edge cases — from the
  packet's E/A/S sections before touching code (Operations step 1). Then build the smallest
  change inside your write scope, against that spec. This commit contains no test files.
- **Verify:** run the exact Verify command; it must exit 0. **Three failed verifies → stop** and
  hand the outputs to the architect, who fixes the packet, splits the task, or adds a sensor.
  Nobody lowers a gate.
- **Evidence, not claims:** return the SDD spec · `verify_sha` · command · `result: PASS` · files
  touched. The keeper re-runs it on the current base and is the **only** writer of `Status`.
- **Do not edit the plan ledger, a test, or a file outside your scope.** Ask the planner for a
  scope change; it is a plan edit + checkpoint, not a shortcut.
- **Deferred test:** the task's `TC-###` lands in its own commit at phase end, covering the
  spec's edge cases — never invented from scratch there. Never mix source and test files in one
  commit.
- **Phase gate:** a phase passes when all its tasks are done/cut, deferred test debt is closed or
  waived, the full gate is green on the default branch, and the validator's one-pass re-read finds
  no `INV-###` breach. Then the next phase opens. If time ends, the last passed phase is the
  honest demo.
- **PRs (if used):** small, early, draft. Body carries `Task: TASK-###`, the Verify command, and
  docs impact (`none` is valid). Reviewers report evidence; they don't set status.
- **Checkpoints:** `python3 box/vault/tools/check-implementation-plan.py docs/implementation-plan.md`
  before review and at every trigger in `box/vault/process/checkpoint.md`. A REJECT is fixed first.

## Code style & conventions
- Language / runtime: Python 3.12+ (FastAPI, SQLAlchemy) — `[assumption]`, not yet built.
- Formatting: `[to be set at TASK-001]`.
- Naming & patterns: modules under `app/core/` own business rules (`BR-###`); `app/gateway/`,
  `app/web/`, `app/db/` are boundary layers only.
- Avoid: any dosing-language literal in code, fixtures, or comments (INV-001) — even in test data.

## Stack currency (verify before coding — overrides your training memory)
Fast-moving frameworks drift. **Do not emit framework code from memory.** Confirm against the
pinned version's docs; if you can't, say so.

| ❌ Stale / from memory | ✅ Current (this project) | Why |
|---|---|---|
| — | — | no stale API caught yet — this project has no code yet |

Pinned versions to confirm at setup: FastAPI, SQLAlchemy, `pytest`, Playwright (staff UI smoke
test) — exact versions to be pinned at `TASK-001`.

## Sensors (what will catch you — run them yourself first)
See [`docs/harness.md`](./docs/harness.md). Minimum: the plan checker, the handoff checker, your
task's TC, the fast gate, and the `pre-commit` hook. Note **S13**: `TC-N01` (INV-001's negative
test) is a known-brittle lexical check — a manual spot-check of any F-001 response copy change is
recommended until it's strengthened.

## Do not touch
- `seed/` — the Key's input brief; the Vault reads it, never edits it.
- Any file outside your claimed task's write scope.
- A test, ever, as an executor.

## Decision ledger & reconcile discipline
Decisions live in [`docs/decision-ledger.md`](./docs/decision-ledger.md), not in chat.
- §1 immutable IDs are never renamed (a rebrand is a logged pivot that skips them). §2 assumptions
  marked UNVALIDATED are never stated as fact — this currently includes `A-001` and `A-002`.
- **ADR ⇒ ledger line, same commit** — the `pre-commit` hook enforces it. Any change touching an
  `INV-###` surface gets a §5 audit line before merge.
- The brief is never frozen: a pivot is logged in §3 and every affected canonical doc is
  reconciled in the same checkpoint. Run the reconcile pass before any pitch/demo/handoff.
- Install the guard once: [`hooks/README.md`](./hooks/README.md).

## Definition of done
- Build passes; the Verify command and the fast gate are green on the current base
  (keeper-verified). The task's deferred `TC-###` lands at phase end.
- The `TASK-###` row carries `base:`/`verify:` shas and the plan checker approves.
- Traceability preserved: change ↔ `F-###` ↔ `TC-###`; every touched `INV-###` still holds.
- Decisions in the ledger; ADRs paired with a ledger line.
- Framework APIs verified against pinned docs, not memory.
- No secrets; exposed surfaces have auth/authz; docs updated only when behaviour changed.

## References
- [Docs index](./docs/index.md) — §0 ownership map · [PRD](./docs/prd.md) · [System Design](./docs/system-design.md) · [QA Test Plan](./docs/qa-test-plan.md)
- [Validation Report](./docs/validation-report.md) · [Implementation Plan](./docs/implementation-plan.md) · [Harness](./docs/harness.md) · [Crew](./docs/crew.md)
- [Decision Ledger](./docs/decision-ledger.md) · [Security & Compliance](./docs/security-compliance.md) · [Ops](./docs/ops.md)
- Vault process: [`box/vault/process/build-loop.md`](./box/vault/process/build-loop.md) · [`box/vault/process/checkpoint.md`](./box/vault/process/checkpoint.md)

Scoped, path-specific rules: `./.cursor/rules/*.mdc`.
