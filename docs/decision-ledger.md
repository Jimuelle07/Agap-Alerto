# Decision Ledger — Agap Alerto

> Append-only record of pivots, rejected approaches, naming decisions, and the assumptions behind
> them. Does not override PRD/system-design/QA plan/implementation plan.
> **Last reconciled:** 2026-09-13

## 1. Names & immutable identifiers (read first)

| Name / ID | Kind | Where it appears | Rule |
|-----------|------|------------------|------|
| Agap Alerto | public product name | UI, alerts, docs | Use everywhere a resident/staff sees it. |
| — | internal codename | — | none assigned yet; same as public name at this stage. |

No infrastructure-level immutable technical IDs (Firebase project id, DB name, etc.) have been
provisioned yet — this project is pre-code. This section will gain rows at `TASK-001` (skeleton).

## 2. Decision assumptions & evidence (confidence, not a product spec)

| SUPPORTED (decision context backed by events) | PROVISIONAL (accepted, not yet verified end-to-end) | UNVALIDATED (do not state as fact) |
|---|---|---|
| The 44 health centers + 7 hospitals distribution network exists and is real (`seed/idea.md` EV-004) | FastAPI + PostgreSQL is a workable stack for this load (`system-design.md`) — not yet built or load-tested | A-001: alerted residents act at a meaningfully higher rate than the general campaign alone |
| The city's declared attack-rate surge and hospital-capacity strain are real (`seed/idea.md` EV-001, EV-002) | A Philippine SMS aggregator gives adequate deliverability/cost (`system-design.md`) — vendor not yet selected or contracted | A-002: staff will keep F-003 status current with near-zero effort |
| — | — | Segment's smartphone/SMS-only device access (`seed/market.md`) |
| — | — | Data Privacy Act / NPC registration applicability (`security-compliance.md`) |

## 3. Pivots & decisions (newest first, append at top)

### 2026-09-13 — PRD revision: freshness window value + F-002 data-dependency named (Vault phase-5.2 cycle 1→2)
- **Type:** invariant-change (clarification) + need (data dependency named)
- **Change:** `docs/prd.md` BR-001 gained a concrete freshness-window default (4 hours,
  `[assumption]`) where it previously referred to "the stated freshness window" without ever stating
  it; F-001's acceptance criteria updated to match. A new BR-005 names a self-registration mechanism
  as the MVP-default source for F-002's resident contact list, and `Dependencies` now names this
  explicitly as an unresolved MVP blocker with two candidate resolutions.
- **Why:** `docs/validation-report.md` cycle 1 (validator, PARTIAL independence) found BR-001
  asserted a fact ("stated") that no document in the set actually supplied, and found F-002's data
  dependency had no owning feature anywhere — both named as the `REVISE(prd)` scope.
- **Invalidated:** none — both are gap-closures, not reversals of a prior claim.
- **Recorded as:** no separate ADR; `docs/validation-report.md` cycle 1 is the record; this is cycle
  1→2 of the phase-5.2 repair loop (max 2 per `manifest.json` qaLoop).

### 2026-09-13 — Usability fix: freshness timestamp added to F-001 (Key phase-4.5 cycle 1→2)
- **Type:** invariant-change (clarification, not a new invariant)
- **Change:** F-001's returned result (`seed/idea.md` §7) gained an explicit last-confirmed
  freshness timestamp, and `idea.md` §8's Activation metric gained a comparative-baseline
  requirement.
- **Why:** the independent `concept_validator` usability walkthrough (`seed/usability.md` cycle 1)
  found K3 (missing context) tripped — INV-002 requires a *stated* freshness window, and the
  original F-001 description did not surface it to the resident. `validation.md` §2 separately
  flagged that §8's original Activation metric could not distinguish "the alert worked" from "the
  general campaign would have worked anyway."
- **Invalidated:** none — this closes an open gap, it does not invalidate a prior claim.
- **Recorded as:** no separate ADR; the seed's own `usability.md` cycle 2 is the record.

## 4. Rejected approaches (what we tried and killed — and why)

| Approach considered | Rejected because | Would revisit if |
|---------------------|------------------|------------------|
| Building F-102 (barangay SMS system integration) as part of the MVP | Would require an integration point this project does not yet have access to, and would delay a pilot past DOH's stated 2-week resurgence window (`seed/idea.md` EV-007) | A confirmed integration point with the existing barangay reporting system becomes available |
| A shared spreadsheet for site status (the segment's literal current workaround) | Cannot enforce a freshness window server-side (INV-002) or serve real-time SMS replies | Never, for the resident-facing surface — a spreadsheet may still be an internal staff-facing fallback if F-003's UI is unavailable |
| Twilio/international SMS gateway | Assumed costlier per-SMS in the Philippines than a local aggregator with direct carrier relationships (`system-design.md`) | If no Philippine aggregator can be contracted in time for a pilot |

## 5. Invariant audit (the `INV-###` guardrails)

| Date | INV-### | Change that touched it | Audit verdict |
|------|---------|------------------------|---------------|
| 2026-09-13 | INV-002 | F-001 description amended to include a freshness timestamp (see §3 above) | kept — strengthens enforcement, does not weaken it |
| 2026-09-13 | INV-002 | `docs/prd.md` BR-001 given a concrete 4-hour default so "stated freshness window" is actually stated (Vault phase-5.2 REVISE cycle) | kept — closes the traceability gap `validation-report.md` cycle 1 found; value itself remains `[assumption]` pending ops review |
| 2026-09-13 | INV-001 | No change; confirmed as a PRD must-never (`prd.md` BR-002) and a QA negative test (`qa-test-plan.md` TC-N01) | kept |

## 6. Open items / risks (do not lose these)

- A-001 and A-002 remain UNVALIDATED (§2) — the alpha release (`release-gtm.md`) is designed to be
  the first real evidence for both.
- Data Privacy Act / NPC applicability unresolved (`security-compliance.md`) — blocks real resident
  phone-number collection until reviewed by counsel, not just this document's `[assumption]`.
- SMS gateway vendor not yet selected or contracted (`system-design.md`).
- Manila Health Department site-list/barangay-mapping confirmation not yet obtained
  (`prd.md` Dependencies).

## References
- `docs/index.md §0` — source-of-truth map
- `docs/adr/` — none recorded yet; the freshness-timestamp change above did not meet the ADR
  triple-gate (hard to reverse AND surprising AND a real trade-off) — it was a straightforward gap
  fix, not a significant decision.
