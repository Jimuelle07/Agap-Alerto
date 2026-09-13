# PRD — Agap Alerto (Manila Lepto Prophylaxis Access)

> Traces back to `seed/idea.md` §1, §2, §6, §7, §9. Traces forward to `system-design.md`,
> `qa-test-plan.md`, `design-system.md`, `implementation-plan.md`.

## Overview & goals
Agap Alerto closes the gap between Manila's existing free-doxycycline distribution network (44
health centers + 7 district hospitals) and the individual resident who was just exposed to
floodwater during a declared leptospirosis epidemic: it must tell that person, same-day, which
specific nearby site is open and stocked *right now* — something the city's general information
campaign does not do (`idea.md` §1). The outcome it must achieve: a meaningfully higher rate of
residents reaching free, supervised prophylaxis within their exposure window than the general
campaign achieves alone (`idea.md` §9, `A-001`).

## Personas & use cases
- **Exposed resident** — informal worker/vendor in a flood-prone Manila barangay, basic phone,
  no regular clinic relationship (`idea.md` §2). Case: was just in floodwater; needs to know where
  to get free prophylaxis before the window closes.
- **Barangay health-center / hospital staff** — front-desk worker at one of the 51 distribution
  sites during a surge. Case: needs a near-zero-effort way to keep the site's stock/open status
  current so the resident-facing information stays trustworthy.

## User journeys
- **UJ-001** — *Alerted resident finds a site:* resident in a flood-declared barangay receives an
  SMS/push alert during the exposure window, replies with the keyword, and receives the nearest
  open, stocked site with hours, distance, and a last-confirmed freshness timestamp. *(core demo
  journey — this is the A-001 outcome.)*
- **UJ-002** — *Unalerted resident looks it up:* a resident who did not receive a push (e.g.,
  outside the declared barangay boundary but still exposed) texts or enters their barangay/location
  and reaches the same result.
- **UJ-003** — *Staff keeps status current:* health-center/hospital staff mark their site's stock
  (in stock/low/out) and open/closed state in one action, without leaving their desk workflow.

## Feature list (with priorities)

| F-ID | Feature | Priority | Solves (idea.md §) | Journey | Notes |
|---|---|---|---|---|---|
| F-001 | Nearest-site lookup with freshness timestamp | MVP | §1 | UJ-001, UJ-002 | Returns site name, status, hours, distance, last-confirmed time |
| F-002 | Exposure-window push alert | MVP | §1 | UJ-001 | Triggered per flood-declared barangay |
| F-003 | Staff one-tap status update | MVP | §1, INV-002 | UJ-003 | In stock/low/out + open/closed |
| F-101 | Multilingual (Filipino/English) plain-language content pass | Final | §1 | UJ-001, UJ-002 | Reading-level tuned to segment |
| F-102 | Integration with existing barangay SMS rapid-reporting system | Final | §4 root cause | UJ-001, UJ-003 | Replaces the standalone channel once integrated |
| F-103 | Post-epidemic routine rainy-season mode | Final | §4 root cause | UJ-001 | Same infra, non-epidemic advisory cadence |

## Business rules
- **BR-001** — A site's status is shown as "open, stocked" only if a staff update (F-003) exists
  within the stated freshness window, defined as **4 hours** since the last confirmed update
  (`[assumption]` — no clinical/ops source sets this number; chosen as roughly half a staff shift, so
  a site cannot go a full shift without a re-confirmation before its status goes stale; the lead
  should revise this default before the first real pilot if operational experience says otherwise —
  `validation-report.md` §3/§6 cycle-1 finding). Past 4 hours, the result shows the site as "status
  unknown" rather than a stale claim. *(Enforces INV-002.)*
- **BR-002** — F-001's result never includes a dosing instruction (dose, frequency, duration); it
  only directs the resident to a site and tells them to ask staff for the supervised prophylaxis.
  *(Enforces INV-001.)*
- **BR-003** — A resident is sent at most one F-002 push alert per declared flood event per phone
  number, to avoid alert fatigue that would degrade response to a future, real event.
- **BR-004** — F-001's lookup accepts a barangay name or a short location string and degrades to
  "nearest by declared barangay" when precise geolocation is unavailable (segment's device access is
  an open `[assumption]`, `seed/market.md`).
- **BR-005** — A resident is added to `ResidentContact` (and therefore becomes eligible for F-002
  alerts for their barangay) by texting a self-registration keyword with their barangay name at any
  time, independent of any active flood declaration. F-002 only ever alerts contacts already on file
  at the moment a declaration is made — it does not retroactively alert someone who registers after
  the alert fanout for that declaration has already run. This is the MVP-default answer to the
  F-002 data-dependency gap named in Dependencies below; it does not preclude the final-scope F-102
  path.

## Hard rules / must-never (invariants — `INV-###`)
- **INV-001** — the system SHALL NEVER recommend or imply a specific dosing regimen to a resident
  without directing them to a supervising health worker. *(Enforced by: security T-002 (content
  review gate), design-system banned-copy, TC-N01, packet safeguards.)*
- **INV-002** — the system SHALL NEVER show a health center/hospital as open or stocked past a
  short, stated freshness window. *(Enforced by: security T-001 (stale-data expiry), design-system
  freshness display rule, TC-N02, packet safeguards.)*

## User flows
1. **UJ-001:** Flood declared for barangay → F-002 sends SMS/push to residents registered/observed
   in that barangay → resident replies keyword → F-001 looks up nearest site with a fresh (F-003)
   status → result shown with freshness timestamp → resident travels to site.
2. **UJ-002:** Resident (no alert received) texts/enters barangay or location to the same F-001
   endpoint → same result screen.
3. **UJ-003:** Staff opens the status screen (web-lite, desk device) → taps one status button →
   F-001's data source updates immediately, freshness clock resets.

## Acceptance criteria (EARS)
- **F-001:** WHEN a resident submits a barangay or location string, the system SHALL return the
  single nearest site whose F-003 status was updated within the last **4 hours** (BR-001's stated
  freshness window), including its name, open/closed state, stock state, hours, distance, and the
  elapsed time since that status was last confirmed.
- **F-001 (stale case):** WHEN no site near the submitted location has an F-003 update within the
  last 4 hours, the system SHALL return "status unknown for sites near you" rather than an
  unconfirmed open/stocked claim.
- **F-002:** WHEN a barangay is marked flood-declared for the active epidemic, the system SHALL send
  one SMS/push alert to each phone number associated with that barangay, once per declared event.
- **F-003:** WHEN staff submit a status update for their site, the system SHALL record the new
  status and a new last-confirmed timestamp within 5 seconds, and SHALL make it immediately
  available to F-001 lookups.

## Non-goals
Mirrors `idea.md` §10: vaccine development, rat-population/vector control, drainage infrastructure;
a general leptospirosis-prevention education app; clinical treatment workflows for
already-symptomatic/hospitalized patients; payment or insurance integration.

## Dependencies
- An SMS gateway reachable by Philippine mobile carriers (F-002, F-001's SMS path) — vendor TBD,
  see `system-design.md` for the candidate and its trade-off. `[assumption]`.
- Manila Health Department cooperation for the initial list of 44 health centers + 7 hospitals and
  their barangay coverage areas (`idea.md` EV-004) — this is public-record-adjacent but the exact
  site→barangay mapping is not sourced this session. `[assumption]`.
- Staff willingness to perform F-003 updates (`idea.md` A-002) — untested; `qa-test-plan.md` and
  `implementation-plan.md` treat this as the first thing the pilot must observe.
- **F-002's resident phone-number/opt-in list has no confirmed source — unresolved MVP blocker**
  (`validation-report.md` cycle-1 finding). Two candidates, neither confirmed this session:
  (1) reuse the existing barangay SMS rapid-reporting network's contact list, per `idea.md` EV-008's
  root-cause framing (F-102 would formalize this integration; a lighter version may be reachable at
  MVP if barangay staff can export/share it); (2) a fresh self-registration flow (a resident texts an
  opt-in keyword, e.g. "REGISTER \<barangay\>," to `ResidentContact.source = self_reported` in
  `data-model.md`) — buildable without external dependency, but starts from zero contacts and so
  delays F-002 having anyone to alert. **BR-005** below adopts (2) as the MVP default so F-002 has a
  defined data source to build against; (1) remains the preferred final-scope path once a barangay
  contact-list export is confirmed available. This is a decision for the lead to revisit, not a
  closed question — see `decision-ledger.md`.

## Open questions
- Exact device/data access of the resident segment (smartphone vs. basic-phone-only) — `[assumption]`
  in `seed/market.md`; determines whether a companion web view is worth building for F-101/F-102.
- Legal/data-sharing basis for the city to share per-site stock data through a third-party-hosted
  system — flagged as an open kill-criterion signal in `idea.md` §9 (regulatory), not resolved here.
