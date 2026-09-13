# Security & Compliance / Threat Model — Agap Alerto

> Traces back to: `system-design.md`, `data-model.md`. `exposed_surface: true` — both the SMS
> lookup/alert surface and the staff web-lite UI are network-exposed.

## Data classification
- **Public/internal:** `Site`, `Barangay`, `SiteStatusUpdate` (operating status is meant to be shown
  to residents; see `data-model.md`).
- **PII (hashed at rest):** `ResidentContact.phone_hash`, `LookupQuery.phone_hash`.
- **Secret:** `StaffCode.code_hash`, SMS gateway API credentials, database credentials.
- No field is health/diagnosis data (`data-model.md` Retention & privacy) — the system never stores
  whether a resident is symptomatic or what they were prescribed.

## Authn / authz model
- **Residents:** no account. Identity is the phone number (hashed on receipt) for SMS interactions;
  the web-lite standalone lookup (UJ-002) requires no identity at all — it is a public, read-only
  query by barangay/location string.
- **Staff:** a per-site `StaffCode` (hashed, rotated per epidemic declaration) authorizes F-003
  writes to exactly that site's status. No cross-site write access; no admin UI is in MVP scope
  (barangay declarations are entered by an operator role with its own credential, distinct from
  site staff codes).

## Threat model

| T-ID | Threat (STRIDE) | Vector | Impact | Mitigation | Enforces |
|------|-----------------|--------|--------|------------|----------|
| T-001 | Spoofing | A shared/stolen `StaffCode` used to post fake "in stock, open" status | Resident sent on a wasted or unsafe trip during an active epidemic | Hashed, per-site, rotated-per-declaration codes; rate-limit writes per code; audit log (`SiteStatusUpdate.confirmed_by_staff_code_id`) | INV-002 |
| T-002 | Tampering | A crafted SMS/web payload injects dosing-like text that the system echoes back | Response appears to give medical dosing advice | Server-side response templates only (no free-text echo into F-001 replies); `TC-N01` scans for banned dosing terms before any send | INV-001 |
| T-003 | Repudiation | Staff denies having posted a status change | No accountability for a bad update during an incident review | Append-only `SiteStatusUpdate` log with staff-code attribution and timestamp; never overwritten | BR-001 |
| T-004 | Information disclosure | Database compromise exposes resident phone numbers | Re-identification of who was in a flood zone at a given time | Phone numbers hashed, never stored in plaintext; DB access restricted to the API service's own credential (no direct human DB access to production) | — (data-model.md privacy) |
| T-005 | Denial of service | Mass fake `FloodDeclaration` entries trigger alert-dispatch storms, exhausting SMS gateway quota/budget | Real alerts delayed or budget exhausted during an actual epidemic | Only an authenticated operator role can create a declaration; dispatch is rate-paced (system-design.md Scaling strategy) | BR-003 |
| T-006 | Elevation of privilege | A resident-facing SMS command attempts to reach the staff status-write endpoint | Unauthorized status write via the wrong channel | F-003 is only reachable via the staff web-lite UI with a valid site code; the SMS channel has no route to it | INV-002 |

## Abuse & safety-specific risks (ethical)

| Risk | Who is harmed | Trigger | Guard (INV-### / mitigation) |
|------|---------------|---------|------------------------------|
| Resident travels to a site shown as open/stocked that is actually closed or out | The resident, during an active epidemic when hospitals are already near capacity | A stale or manipulated status | INV-002 (freshness window, server-enforced, TC-N02) |
| Resident interprets the alert/result as medical authorization to self-dose | The resident (worsens the exact self-medication risk DOH already warns against, `idea.md` EV-010) | Ambiguous or dosing-adjacent copy in the response | INV-001 (BR-002, TC-N01, `design-system.md` banned copy) |
| A resident's floodwater-exposure pattern (barangay + timing) is inferred from stored data and used for a purpose beyond this service | Resident, as a privacy/surveillance harm | Data broader than "barangay + timestamp" retained, or shared outside this service | `data-model.md` retention limits (90-day purge, hashed phone, no health data); no secondary-use clause exists — flagged as an open question below |
| Staff-code compromise is used to spam "closed"/"out" status, suppressing legitimate access | Residents who would otherwise have gone to a genuinely open site | Leaked or guessed staff code | T-001 mitigations (rotation, rate-limit, audit log) |

## Compliance obligations
- **Philippine Data Privacy Act (RA 10173)** applies to `ResidentContact`/`LookupQuery` phone data
  even hashed, given the health-adjacent context of the service. Whether this deployment requires
  NPC (National Privacy Commission) registration or a Data Privacy Impact Assessment has **not**
  been determined this session — `[assumption]`, and is the single largest open compliance question
  before any real pilot (see Pre-milestone checklist).
- No other sector-specific regulation (e.g., a formal health-data/HIPAA-equivalent regime) was
  identified in this session's research as clearly applicable, since the system stores no
  diagnosis/prescription data — flagged, not confirmed by legal review.

## Secrets handling
- SMS gateway API key, database credentials, and the staff-code hashing pepper are held in the
  hosting platform's secret manager / environment variables, never committed to the repository and
  never logged. Rotation: staff codes rotate per declared epidemic (operational); gateway/DB
  credentials rotate on the hosting platform's standard schedule (`[assumption]` — no specific
  cadence sourced this session).

## Audit & logging
- **Logged:** `SiteStatusUpdate` writes (who, what, when — already the data model's audit trail),
  `AlertDelivery` dispatch events, `LookupQuery` (barangay/location string + resolved site, no raw
  phone in plaintext).
- **Never logged:** raw (unhashed) phone numbers; any response text containing dosing-adjacent
  terms (the same banned-term scan as TC-N01 applies to logs, not just outbound replies); staff-code
  plaintext.

## Incident response basics
- **Detection:** hosting platform's error/uptime monitoring (`ops.md` Observability).
- **Escalation:** to the project's on-call/ops owner — no named Manila Health Department contact
  exists yet in this seed; `[assumption]`, an open item for the pilot's delivery facts
  (`implementation-plan.md` §1).
- **Rollback:** redeploy the prior known-good build (`ops.md` Deploy); a bad `SiteStatusUpdate` is
  correctable by a new status write (append-only log, never edited/deleted).
- **Notify:** the site(s) affected by any incident that could have shown a false status, plus the
  ops owner.

## Pre-milestone hard-gate checklist
- [ ] Every network-exposed surface declares auth/authz (no open write paths). — {pilot launch date}
- [ ] No secret is committed; all secrets referenced, not inlined. — {pilot launch date}
- [ ] Every `INV-###` invariant still holds — INV-001 (TC-N01) and INV-002 (TC-N02) green on the
      default branch. — {pilot launch date}
- [ ] The decision ledger and `docs/index.md §0` agree across live branches. — {pilot launch date}
- [ ] Data Privacy Act applicability (NPC registration / DPIA need) resolved with an actual legal
      reviewer, not this document's `[assumption]`. — {before any real resident phone number is
      collected}
