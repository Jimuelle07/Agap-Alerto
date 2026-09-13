# Implementation Plan — Agap Alerto

> Living, phased, test-first execution state. Validated by
> `python3 box/vault/tools/check-implementation-plan.py docs/implementation-plan.md`.

**Plan steward:** planner (Sonnet 5) — pending a named human lead for this build (`[assumption]`:
no delivery contact confirmed this run, per `manifest.json` phase-5.0's "ask ONE delivery question,"
which had no human present to answer; see Open items below)
**Status writer:** keeper *(the only role that edits Status cells and phase rows)*
**Last checkpoint:** 2026-09-13T00:00:00Z · plan authored (kickoff — no task claimed yet)
**Deadline / demo cutoff:** no external submission deadline (`context.md judged: false`); target
demo-ready: an alpha pilot in 1–2 flood-declared barangays within the `2w` `time_budget`
(`context.md`) — `[assumption]`, not a confirmed external date.
**Honest stopping point:** none yet — greenfield, no phase has opened.

## 1. Planning inputs (delivery facts, asked once at phase-5.0)
- **Current code state:** greenfield — no application code exists yet; this doc suite is the first
  artifact.
- **Team capacity:** `UNASSIGNED` — no human delivery contact was available to answer phase-5.0's
  delivery question in this run; `context.md team_size: 1, mode: solo`. The crew (`crew.md`) is
  agent roles, not a substitute for a named human owner.
- **Core demo journey:** `UJ-001` — alerted resident reaches a fresh, site-specific result.
- **Highest implementation risks:** the SMS gateway vendor is unselected (`system-design.md`
  `[assumption]`); INV-002's freshness-window correctness under real staff usage patterns (A-002,
  untested); resident phone-number acquisition (`prd.md` BR-005 / Dependencies, unresolved MVP
  blocker until a real registration channel exists).
- **Fast gate:** `pytest -m fast` · **Full gate:** `pytest`
- **Browser E2E:** `tests/e2e/test_staff_status_smoke.py` (Playwright, Chromium) — smoke only, does
  not gate any task's Verify.
- **Test mode:** `deferred` (`context.md`) — every feature task below carries a `TC-###` obligation
  that must land (in its own commit, at phase end) or be waived before its phase gate passes.
- **Hard constraints / rubric:** none (`context.md judged: false`).

## 2. Phases

| Phase | Goal (demoable outcome) | Exit command | Status |
|---|---|---|---|
| PH-01 | Runnable skeleton + F-001 (lookup) + F-003 (staff status write) working against seeded data; `UJ-002` (standalone lookup) walkable end to end via direct API calls — no live SMS yet | `pytest` | open |
| PH-02 | F-002 (alert dispatch) + SMS gateway integration; `UJ-001` (the core demo journey) walkable end to end via the gateway's sandbox mode | `pytest` | pending |

## 3. Task ledger

| ID | Phase | Outcome / trace | Depends on | Owner | Write scope | Verify | Tests | Work ref | Status | Gate / evidence |
|---|---|---|---|---|---|---|---|---|---|---|
| TASK-001 | PH-01 | infra: runnable skeleton — FastAPI app, DB models/migrations for every `data-model.md` entity, health endpoint | — | executor | `app/config/*`, `app/db/*`, `migrations/*`, `pyproject.toml` | `python -c "import app.main"` | TC-007 pending | — | ready | `python -c "import app.main"` |
| TASK-002 | PH-01 | F-001 core lookup logic (fresh/stale/fallback) + `copy_guard` (INV-001 defensive check); primary test TC-001, also exercises TC-002/TC-006/TC-N01 (same write scope) | TASK-001 | executor | `app/core/lookup.py`, `app/core/copy_guard.py` | `python -c "from app.core.lookup import resolve_nearest_site"` | TC-001 pending | — | blocked | `python -c "from app.core.lookup import resolve_nearest_site"` |
| TASK-003 | PH-01 | F-003 staff status write (`record_status`, server-assigned `confirmed_at`) | TASK-001 | executor | `app/core/status.py` | `python -c "from app.core.status import record_status"` | TC-005 pending | — | blocked | `python -c "from app.core.status import record_status"` |
| TASK-004 | PH-01 | F-003 staff web-lite UI (browser surface, one screen) | TASK-003 | executor | `app/web/staff_panel.py` | `python -m py_compile app/web/staff_panel.py` | waived: covered by TC-005 (TASK-003) + the Playwright smoke test, no dedicated TC-### | — | blocked | `python -m py_compile app/web/staff_panel.py` |
| TASK-005 | PH-02 | infra: SMS Gateway Adapter (send/receive against sandbox mode) | TASK-002 | executor | `app/gateway/adapter.py` | `python -c "from app.gateway.adapter import send_sms, parse_inbound"` | waived: adapter plumbing only, covered indirectly by TC-003/TC-004 (TASK-006) | — | blocked | `python -c "from app.gateway.adapter import send_sms, parse_inbound"` |
| TASK-006 | PH-02 | F-002 alert dispatch (BR-003 uniqueness, BR-005 self-registration intake); primary test TC-003, also exercises TC-004 (same write scope) | TASK-002, TASK-005 | executor | `app/core/dispatch.py`, `app/core/registration.py` | `python -c "from app.core.dispatch import dispatch_alert"` | TC-003 pending | — | blocked | `python -c "from app.core.dispatch import dispatch_alert"` |

## 4. Handoff and checkpoint protocol
At claim time the planner writes `docs/handoff/TASK-###.md` (template `handoff-packet.md`);
`check-handoff.py` must APPROVE; the keeper runs the Verify command on the base commit, records
`base: <sha> exit <n>` (must fail) and `in_progress`; the executor runs the build→verify loop
(`process/build-loop.md`) and returns evidence; the keeper re-runs the command at the returned sha,
then again on the current base after integration, appends the run event, and writes `done` once the
task's `TC-###` obligation is closed or waived. `docs/handoff/TASK-001.md` has already been written
and checked (below) as this plan's first packet.

## 5. Execution view (derived — paste the checker's output, do not hand-maintain)
```
APPROVE: docs/implementation-plan.md — 2 phase(s), 6 task(s); phase sequence, DAG, gating, and Build-First verify evidence are coherent (tests=deferred)
Honest stopping point: none passed yet · open: PH-01
PH-01 [open] — 4 task(s)
  wave 0: TASK-001(ready)
  wave 1: TASK-002(blocked), TASK-003(blocked)
  wave 2: TASK-004(blocked)
PH-02 [pending] — 2 task(s)
  wave 2: TASK-005(blocked)
  wave 3: TASK-006(blocked)
Ready now: TASK-001
Parallel-safety: no same-phase same-wave write-scope overlaps
```
Ready now: TASK-001 · Blocked on: TASK-002/003 (on TASK-001), TASK-004 (on TASK-003), TASK-005/006
(on TASK-002, +TASK-005 for TASK-006) · Parallel-safe this wave: none yet (only TASK-001 is ready) ·
Cut line if time ends: PH-01 alone (F-001 + F-003, no live alerting) is still a demoable, honest
partial product — a staff-only status board with a manual lookup, no SMS.

## 6. Plan change log

| Timestamp / event | Phases / tasks changed | Why / evidence | Canonical docs reconciled |
|---|---|---|---|
| 2026-09-13T00:00:00Z · kickoff | PH-01, PH-02; TASK-001…TASK-006 created | initial dependency/risk cut from `prd.md` + `system-design.md` + `qa-test-plan.md`, post `validation-report.md` PASS (cycle 2) | PRD · system design · QA plan · validation report |
