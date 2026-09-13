# System Design Document (HLD) — Agap Alerto

> Traces back to: `prd.md`. Traces forward to: `technical-design.md`, `data-model.md`,
> `security-compliance.md`.

## Context diagram
```
                     +-------------------------+
  Flood/epidemic  -->|  Barangay Declaration    |
  declaration data   |  Loader (F-002 trigger)  |
  (city/DOH, manual  +------------+-------------+
   entry v1)                      |
                                   v
+------------------+      +-------------------+      +----------------------+
| SMS Gateway       |<---->|  Agap Alerto Core |<---->| PostgreSQL           |
| (resident SMS in/ |      |  (FastAPI service)|      | (sites, status,      |
|  out, F-001/F-002)|      |                   |      |  barangays, alerts)  |
+------------------+      +---------+---------+      +----------------------+
                                     |
                                     v
                          +----------------------+
                          | Staff web-lite UI     |
                          | (F-003, desk device)  |
                          +----------------------+
```
External actors: resident (via SMS or a low-bandwidth web page), barangay/hospital staff (via a
simple web page on a shared desk device), and a barangay-declaration data source (initially manual
entry by an operator during an active epidemic, per `prd.md` Dependencies).

## Components & responsibilities
- **SMS Gateway Adapter** — sends/receives SMS via a Philippine SMS API provider; owns retry/backoff
  for carrier delivery; translates provider webhooks into internal events.
- **Agap Alerto Core (API service)** — owns F-001 (lookup), F-002 (alert dispatch), F-003 (status
  write), BR-001–BR-004, INV-001/INV-002 enforcement (freshness check, no-dosing-language check on
  any templated response).
- **Site/Status Store** — the single source of truth for site metadata, barangay coverage, and
  live stock/open status with timestamps.
- **Barangay Declaration Loader** — v1: an authenticated staff action marking a barangay
  flood-declared (start/end of the exposure window); triggers F-002 for phone numbers on file for
  that barangay. Final-scope (F-102): ingests the existing barangay SMS rapid-reporting system
  instead of a manual entry (`idea.md` EV-008).
- **Staff web-lite UI** — the F-003 status-update surface; no login beyond a per-site staff code
  (see `security-compliance.md`).

## Data flow
1. An operator (or, at F-102, the existing barangay reporting feed) marks a barangay
   flood-declared → Barangay Declaration Loader writes an `Alert` record and hands phone numbers on
   file to the SMS Gateway Adapter → each resident receives one F-002 SMS.
2. Resident replies (or a new resident texts in cold) → SMS Gateway Adapter → Agap Alerto Core →
   F-001 queries Site/Status Store for the nearest site with a status inside the freshness window →
   composes the reply (BR-001, BR-002) → SMS Gateway Adapter sends it back.
3. Staff taps a status button on the web-lite UI → Agap Alerto Core validates the per-site staff
   code → writes a new status + timestamp to the Site/Status Store → subsequent F-001 lookups see it
   immediately (F-003 acceptance criterion: available within 5 seconds).

## Key technology choices + rationale

| Choice | Why | Trade-off | Alternative rejected |
|---|---|---|---|
| Philippine SMS aggregator (e.g. Semaphore-class provider) for the SMS gateway `[assumption — vendor not confirmed this session]` | Direct relationships with PH carriers give better deliverability/cost for a segment assumed to be SMS-first (`seed/market.md`) | Vendor lock-in; a second provider would need a re-integration | Twilio/international gateway — works technically but typically costlier per-SMS in PH and adds an extra carrier hop |
| FastAPI (Python) backend | Small, explicit, easy for a lean crew to reason about; matches the kit's own stdlib-first bias | Less batteries-included than a full framework; more manual wiring for auth | Node/Express — equally viable; Python chosen for one less language in a small crew's stack |
| PostgreSQL for Site/Status Store | Strong consistency matters for INV-002 (a stale read must not show "open, stocked"); relational shape fits sites/barangays/status/alerts cleanly | Requires a managed instance / ops burden vs. a spreadsheet | A shared spreadsheet (the segment's literal status quo per `idea.md` EV-008 root cause) — rejected because it cannot enforce a freshness window or serve SMS replies in real time |
| Managed PaaS hosting for the pilot (e.g. a small always-on web service + managed Postgres) | Stands up fast enough to matter during an active, time-boxed epidemic (`ops.md`); `outlives_demo: true` requires it to actually run, not just demo | Not under direct LGU infrastructure control at pilot stage; a handoff/migration plan is needed for a real deployment | Waiting for LGU-provisioned infrastructure before piloting — rejected because the DOH's own stated 2-week resurgence window (EV-007) does not allow for a slow procurement cycle before the first pilot |
| TTL-based freshness check (server-side, not client-side) | INV-002 must hold even if a client caches a stale response; server always re-checks the timestamp against "now" before answering | Requires the server clock/TTL config to be correct and consistently applied | Client-side-only freshness display — rejected: a cached SMS reply could show a stale "open, stocked" with no server re-check |

## Integration points
- **SMS gateway API** (send/receive, delivery receipts) — failure mode: provider outage or carrier
  throttling; mitigation: queue-and-retry with a capped backoff, and a staff-visible dashboard
  showing undelivered-alert counts (`ops.md`).
- **Barangay boundary/site data** — initially a static, staff-maintained table (site → serves which
  barangays); failure mode: an outdated mapping sends a resident to a site outside their real
  barangay; mitigation: the freshness/BR-004 fallback plus a manual data-correction runbook
  (`ops.md`).

## Deployment topology
- Single-region deployment (Philippines-adjacent region for latency to PH carriers/SMS gateway):
  one API service instance (stateless, horizontally scalable if load requires it — unlikely at
  tens-of-thousands scale per `seed/market.md`), one managed Postgres instance, the SMS gateway as an
  external managed service.
- Staff web-lite UI is served by the same API service (no separate frontend deploy needed for MVP).

## Scaling strategy
At the sized load (tens of thousands of residents per epidemic wave, `seed/market.md`), a single
small API instance plus managed Postgres is expected to be sufficient; the risk is not compute
scale but **SMS throughput and carrier rate limits** during a burst alert dispatch (F-002 fanout to
an entire barangay at once) — the Alert Dispatcher must queue and pace sends rather than fire all at
once. This is a non-functional need the seed does not explicitly state; flagged here as an
`[assumption]` requirement, not an invented target.

## Trade-offs considered
- Building F-102 (integration with the existing barangay SMS system) as part of the MVP instead of
  a standalone F-002/F-001 loop was considered and rejected for the MVP: it would require access
  to a system this project does not yet have a confirmed integration point for, and would delay a
  pilot past the DOH's stated 2-week resurgence window (EV-007). Kept as final-scope F-102.
