# QA — Test Plan & Test Cases — Agap Alerto

> Traces back to `prd.md`, `system-design.md`. Owns the `TC-###` traceability sink and the
> executor's fence.

## Test strategy
Cheapest layer first: **unit** for BR-001/BR-002/BR-004 logic and the freshness-window check
(pure functions, no I/O); **integration/contract** for the Site/Status Store reads/writes and the
SMS gateway adapter (against a local Postgres + a stubbed gateway); **e2e** reserved for UJ-001 only
(the one flow that would break the demo).

## Test profile (context-scaled)
- **Browser UI present:** yes — the staff web-lite UI (F-003) only; the resident-facing surface is
  SMS-first (`has_ui: true`, `seed/market.md`).
- **Core smoke journey:** UJ-001 → TC-005 (end-to-end alert → reply → fresh result).
- **Fast gate command (per task, <60s):** `pytest -m fast`
- **Full gate command (phase exit, default branch):** `pytest`
- **CI budget / constraints:** no live SMS-carrier or live gateway calls in CI — the gateway is
  stubbed (`system-design.md` Integration points); secrets referenced by name only
  (`security-compliance.md`).
- **Environments:** one primary (staging, connected to the SMS gateway's sandbox mode) — no
  multi-environment need identified from the seed.

## Scope
### In scope
F-001, F-002, F-003 (MVP) and their INV-001/INV-002 enforcement.
### Out of scope
F-101/F-102/F-103 (final scope — no TC yet; scheduled once claimed, per `implementation-plan.md`).

## Environments
Local/staging Postgres, seeded with a fixed fixture set (3 barangays, 5 sites, 2 staff codes);
SMS gateway adapter runs in stub mode (records "sends" without hitting a real carrier); no
production data or real phone numbers in any test environment.

## Traceability matrix

| F-ID | Feature | Test case ID(s) | Lowest proving level | Automation | Red as of |
|---|---|---|---|---|---|
| F-001 | Nearest-site lookup with freshness timestamp | TC-001, TC-002, TC-006 | unit | planned | not yet |
| F-002 | Exposure-window push alert | TC-003, TC-004 | integration | planned | not yet |
| F-003 | Staff one-tap status update | TC-005 | integration | planned | not yet |
| F-101 | Multilingual content pass | — | — | final scope | not scheduled |
| F-102 | Barangay SMS system integration | — | — | final scope | not scheduled |
| F-103 | Post-epidemic routine mode | — | — | final scope | not scheduled |

**Infra (no F-### — the skeleton itself):** TC-007 covers `TASK-001`'s health endpoint; it is not a
feature test and is not counted in the T1 MVP-feature-coverage check, but it is still a real,
runnable case (`implementation-plan.md` TASK-001).

## Automation contract

| Test ID | Level / tool | Test path | Command | Trigger | Artifact / evidence |
|---|---|---|---|---|---|
| TC-001 | unit + pytest | `tests/unit/test_lookup.py::test_fresh_status_returned` | `pytest tests/unit/test_lookup.py::test_fresh_status_returned` | task | stdout |
| TC-002 | unit + pytest | `tests/unit/test_lookup.py::test_stale_status_hidden` | `pytest tests/unit/test_lookup.py::test_stale_status_hidden` | task | stdout |
| TC-003 | integration + pytest | `tests/integration/test_alerts.py::test_one_alert_per_contact` | `pytest tests/integration/test_alerts.py::test_one_alert_per_contact` | task | stdout |
| TC-004 | integration + pytest | `tests/integration/test_alerts.py::test_no_duplicate_alert_same_declaration` | `pytest tests/integration/test_alerts.py::test_no_duplicate_alert_same_declaration` | task | stdout |
| TC-005 | integration + pytest | `tests/integration/test_status.py::test_status_visible_within_5s` | `pytest tests/integration/test_status.py::test_status_visible_within_5s` | task | stdout |
| TC-006 | unit + pytest | `tests/unit/test_lookup.py::test_barangay_fallback_match` | `pytest tests/unit/test_lookup.py::test_barangay_fallback_match` | task | stdout |
| TC-N01 | negative + pytest | `tests/negative/test_invariants.py::test_no_dosing_language` | `pytest tests/negative/test_invariants.py::test_no_dosing_language` | task | stdout |
| TC-N02 | negative + pytest | `tests/negative/test_invariants.py::test_no_stale_open_claim` | `pytest tests/negative/test_invariants.py::test_no_stale_open_claim` | task | stdout |
| TC-007 | unit + pytest | `tests/unit/test_skeleton.py::test_health_endpoint` | `pytest tests/unit/test_skeleton.py::test_health_endpoint` | task | stdout |

## Test cases

### TC-001 — fresh status is returned with a freshness timestamp
- **Covers:** F-001
- **Level:** unit
- **Preconditions / controlled data:** one `Site` with a `SiteStatusUpdate` confirmed 10 minutes ago (in_stock, open)
- **Steps:** call the lookup function for that site's barangay
- **Expected (EARS):** WHEN a resident submits that barangay, the system SHALL return the site's
  name, open/stocked state, hours, distance, and "confirmed 10 min ago" (or equivalent elapsed-time
  phrasing).
- **Automation:** `tests/unit/test_lookup.py` · `pytest tests/unit/test_lookup.py::test_fresh_status_returned` · red on current codebase (no implementation yet)

### TC-002 — stale status is never shown as open/stocked (BR-001)
- **Covers:** F-001
- **Level:** unit
- **Preconditions / controlled data:** one `Site` whose newest `SiteStatusUpdate.confirmed_at` is
  older than the configured freshness window
- **Steps:** call the lookup function for that site's barangay
- **Expected (EARS):** WHEN no site near the submitted location has a status confirmed inside the
  freshness window, the system SHALL return "status unknown for sites near you" and SHALL NOT
  return "open" or "stocked" for that site.
- **Automation:** `tests/unit/test_lookup.py` · `pytest tests/unit/test_lookup.py::test_stale_status_hidden` · red on current codebase

### TC-003 — each opted-in resident in a declared barangay receives exactly one alert
- **Covers:** F-002
- **Level:** integration
- **Preconditions / controlled data:** 3 `ResidentContact` rows in one barangay, one new `FloodDeclaration`
- **Steps:** trigger the declaration; run the dispatcher
- **Expected (EARS):** WHEN a barangay is marked flood-declared, the system SHALL send one alert to
  each opted-in resident contact associated with that barangay.
- **Automation:** `tests/integration/test_alerts.py` · `pytest tests/integration/test_alerts.py::test_one_alert_per_contact` · red on current codebase

### TC-004 — no duplicate alert to the same contact for the same declaration (BR-003)
- **Covers:** F-002
- **Level:** integration
- **Preconditions / controlled data:** one contact, one declaration, dispatcher run twice
- **Steps:** trigger dispatch twice for the same declaration id
- **Expected (EARS):** IF a dispatch is attempted twice for the same `(flood_declaration_id,
  resident_contact_id)`, THEN the system SHALL send only one alert (unique constraint / idempotent
  dispatch).
- **Automation:** `tests/integration/test_alerts.py` · `pytest tests/integration/test_alerts.py::test_no_duplicate_alert_same_declaration` · red on current codebase

### TC-005 — staff status update visible to lookups within 5 seconds
- **Covers:** F-003
- **Level:** integration
- **Preconditions / controlled data:** one site, valid staff code
- **Steps:** submit a status update via the staff endpoint; immediately query F-001 for that site's barangay
- **Expected (EARS):** WHEN staff submit a status update, the system SHALL make it available to
  F-001 lookups within 5 seconds.
- **Automation:** `tests/integration/test_status.py` · `pytest tests/integration/test_status.py::test_status_visible_within_5s` · red on current codebase

### TC-006 — barangay-name fallback resolves to the correct nearest site
- **Covers:** F-001
- **Level:** unit
- **Preconditions / controlled data:** 2 barangays, each with one fresh site
- **Steps:** submit each barangay name
- **Expected (EARS):** WHERE precise geolocation is unavailable, the system SHALL resolve the
  nearest site by declared barangay match (BR-004).
- **Automation:** `tests/unit/test_lookup.py` · `pytest tests/unit/test_lookup.py::test_barangay_fallback_match` · red on current codebase

### TC-007 — skeleton boots and the health endpoint responds (infra, not a feature test)
- **Covers:** infra (`TASK-001`, no `F-###` — the runnable skeleton itself)
- **Level:** unit
- **Preconditions / controlled data:** none — a fresh app instance
- **Steps:** import the application module; issue a request to the health route
- **Expected (EARS):** WHEN a request hits the health endpoint, the system SHALL return HTTP 200
  with `{"status": "ok"}`.
- **Automation:** `tests/unit/test_skeleton.py` · `pytest tests/unit/test_skeleton.py::test_health_endpoint` · red on current codebase (no `app/main.py` exists yet)

## Invariant (negative) tests — the `INV-###` guardrails

| INV-ID | Invariant (must never…) | Negative test ID(s) | Automation |
|---|---|---|---|
| INV-001 | never imply a dosing regimen without directing to a supervising health worker | TC-N01 | `pytest tests/negative/test_invariants.py::test_no_dosing_language` |
| INV-002 | never show open/stocked past a stated freshness window | TC-N02 | `pytest tests/negative/test_invariants.py::test_no_stale_open_claim` |

### TC-N01 — asserts INV-001 is never violated
- **Covers:** INV-001
- **Assertion (EARS unwanted):** the system SHALL NEVER include a dose amount, frequency, or
  duration (e.g., "200mg", "once weekly", "twice daily") in any F-001 response text.
- **Probe:** a lookup response scanned against a banned-term list drawn from the known doxycycline
  prophylaxis protocol (`seed/research-brief.md` R-011: "200mg", "100mg", "BID", "weekly").
- **Expected:** none of the banned terms appear in any generated response.
- **Automation:** `tests/negative/test_invariants.py` · `pytest tests/negative/test_invariants.py::test_no_dosing_language`

### TC-N02 — asserts INV-002 is never violated
- **Covers:** INV-002
- **Assertion (EARS unwanted):** the system SHALL NEVER return "open" or "stocked" for a site whose
  latest `SiteStatusUpdate.confirmed_at` is older than the configured freshness window.
- **Probe:** a site with a status update timestamped just past the window boundary.
- **Expected:** the response shows "status unknown," never "open, stocked."
- **Automation:** `tests/negative/test_invariants.py` · `pytest tests/negative/test_invariants.py::test_no_stale_open_claim`

## Browser E2E with Playwright
Only the staff web-lite UI (F-003) has a browser surface. One smoke test: staff logs in with a site
code, taps a status button, sees the confirmation — `tests/e2e/test_staff_status_smoke.py`
(Playwright, Chromium only, `workers: 1`, `forbidOnly`, `trace: on-first-retry`). The resident-facing
UJ-001 core journey is exercised via the integration layer (TC-003/TC-005 chained), not Playwright,
since it is an SMS conversation, not a browser flow.

## Acceptance criteria
See `prd.md` Acceptance criteria (EARS) — each is one of TC-001…TC-006 above; no restatement here.

## Regression plan
Per task: `pytest -m fast` + the task's own TC. Per phase: `pytest` (full gate) on the default
branch. Before any pilot/demo milestone: full gate + TC-005 (core smoke).

## Exit criteria
All of TC-001…TC-006 and TC-N01/TC-N02 green on the default branch; no open defect class above
"cosmetic" (a defect that changes INV-001/INV-002 behavior or F-001/F-002/F-003 correctness blocks
exit).
