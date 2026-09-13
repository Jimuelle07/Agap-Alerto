# Documentation Index — Agap Alerto

**Maintained by:** UNASSIGNED (no named human lead yet — `implementation-plan.md` §1) ·
**Last updated:** 2026-09-13 · **Vault version:** 1.5.0

## 0. Source-of-truth map (one fact, one home)

| Concern | Canonical owner | Note |
|---|---|---|
| Vision · problem · segment · riskiest assumption `A-001`/`A-002` | [idea.md](../seed/idea.md) | the seed this suite was generated from |
| Whether the design still serves the concept | [Validation Report](validation-report.md) | phase-5.2 verdict — **PASS**, cycle 2 of 2; written by a model (Fable 5.1) that authored none of the docs it judged, PARTIAL independence disclosed |
| What we build (`F-###`, `UJ-###`, `BR-###`, `INV-###`) | [PRD](prd.md) | origin of the spine downstream of the seed; EARS acceptance criteria |
| How it's built (components, trade-offs) | [System Design](system-design.md) | FastAPI + PostgreSQL + a Philippine SMS gateway (`[assumption]`-flagged vendor) |
| Behavior detail per feature | [FRD](frd.md) | inputs/outputs/edge cases per `F-###` |
| Low-level module/algorithm design | [Technical Design](technical-design.md) | signatures, sequence diagrams |
| Data schema · entities | [Data Model](data-model.md) | — |
| API contracts | — | *not selected — `exposes_api: false`* |
| UI tokens · components · routes · banned copy | [Design System](design-system.md) | grounds in `seed/brand.md`; reuses `seed/usability.md`'s cleared flow verbatim |
| Tests · traceability (`TC-###`; every `F-###` ≥1, every `INV-###` ≥1 negative) | [QA Test Plan](qa-test-plan.md) | the executor's fence; TC-N01 flagged as lexically brittle, not yet strengthened |
| Security · auth/authz · `INV-###` threat mitigations | [Security & Compliance](security-compliance.md) | Data Privacy Act applicability is an open pre-milestone gate |
| Operations · deploy · runbook | [Ops](ops.md) | selected — `outlives_demo: true` |
| Release · GTM | [Release / GTM](release-gtm.md) | alpha → beta → GA rollout plan |
| Onboarding | [Onboarding](onboarding.md) | — |
| Live execution state (`PH-##`, `TASK-###`, status, red/green evidence) | [Implementation Plan](implementation-plan.md) | the only execution-state file; PH-01 open, TASK-001 ready |
| One task's bounded brief | [handoff/TASK-001.md](handoff/TASK-001.md) | checked APPROVE by `check-handoff.py`; more packets are written at claim time |
| Build-time agent roster | [Crew](crew.md) | 4 agents; materialises to `.claude/agents/*.md` |
| Guides and sensors (what steers, what catches) | [Harness](harness.md) | includes S13, flagging TC-N01's known weakness |
| Decisions · pivots · rejected choices · immutable IDs · INV audits | [Decision Ledger](decision-ledger.md) | 2 pivots recorded so far (both gap-closures, no reversal) |
| Changes to Locked docs | [Change Record](change-record.md) | no CRs yet — nothing is Locked |
| Market · positioning | [Market](../seed/market.md) | the Key's seed sibling, not regenerated here |

> Rows for docs this run did not select (`methods`, `api-spec`, `pitch-kit`, `brd`, `mrd`, `srs`) are
> omitted per `select-docs.py`'s output — see `docs/select-docs-output.txt`-equivalent in this run's
> terminal history; none were generated.

## 1. Document suite

| Document | File | Status | Last updated |
|---|---|---|---|
| PRD | [prd.md](prd.md) | Draft | 2026-09-13 |
| System Design | [system-design.md](system-design.md) | Draft | 2026-09-13 |
| Data Model | [data-model.md](data-model.md) | Draft | 2026-09-13 |
| QA Test Plan | [qa-test-plan.md](qa-test-plan.md) | Draft | 2026-09-13 |
| Validation Report | [validation-report.md](validation-report.md) | PASS (cycle 2) | 2026-09-13 |
| Security & Compliance | [security-compliance.md](security-compliance.md) | Draft | 2026-09-13 |
| Design System | [design-system.md](design-system.md) | Draft | 2026-09-13 |
| FRD | [frd.md](frd.md) | Draft | 2026-09-13 |
| Technical Design | [technical-design.md](technical-design.md) | Draft | 2026-09-13 |
| Ops | [ops.md](ops.md) | Draft | 2026-09-13 |
| Release / GTM | [release-gtm.md](release-gtm.md) | Draft | 2026-09-13 |
| Onboarding | [onboarding.md](onboarding.md) | Draft | 2026-09-13 |
| Decision Ledger | [decision-ledger.md](decision-ledger.md) | living | 2026-09-13 |
| Change Record | [change-record.md](change-record.md) | living (empty) | 2026-09-13 |
| Implementation Plan | [implementation-plan.md](implementation-plan.md) | living — PH-01 open | 2026-09-13 |
| Crew | [crew.md](crew.md) | Draft | 2026-09-13 |
| Harness | [harness.md](harness.md) | Draft | 2026-09-13 |
| Handoff packets | [handoff/](handoff/) | 1 of 6 written (TASK-001) | 2026-09-13 |

## 2. Health check (before calling the suite "done")

- [x] Every `F-###` in the PRD has ≥1 `TC-###` in the QA plan (T1: 3/3) — `trace-ids.py`.
- [x] Every `INV-###` has ≥1 `TC-N##` (T5: 2/2) — `trace-ids.py`.
- [x] No doc restates a fact owned by another (§0 respected).
- [ ] Every exposed surface declares auth/authz — **partial**: the staff surface's per-site-code
      mechanism is named (`system-design.md`) but `security-compliance.md`'s Authn/authz model
      section states it only at the same level of detail; a concrete implementation (hashing scheme,
      rotation trigger) is still a `TASK-###`-level decision, not yet made. The resident surface
      correctly needs no auth (public read).
- [x] Methods doc: N/A — `computes_numbers: false`, not selected.
- [x] `check-implementation-plan.py docs/implementation-plan.md` → **APPROVE**; every feature task
      names a Verify command and a deferred `TC-###` (or an explicit, checker-accepted waiver);
      exactly one phase (`PH-01`) is `open`.
- [x] Crew exists (`crew.md`, 4 agents, ≤6) — materialised to `.claude/agents/*.md` (see below).
- [x] Validation report verdict is `PASS` (cycle 2 of 2) — see §0.
- [ ] **Not yet true, flagged honestly rather than hidden:** a human lead has confirmed the doc set,
      the delivery facts (`implementation-plan.md` §1 `UNASSIGNED`), and the Data Privacy Act
      determination (`security-compliance.md`). This run had no human present at phase-5.0/5.3 to
      answer those questions — they are the first things a real lead should resolve before any
      `TASK-###` is actually claimed and built.
