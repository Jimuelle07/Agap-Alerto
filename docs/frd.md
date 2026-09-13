# FRD — Functional Requirements Document — Agap Alerto

> Traces back to: `prd.md` feature list. Traces forward to: `qa-test-plan.md` test cases.

### F-001 — Nearest-site lookup with freshness timestamp

**Description**
Given a barangay name or location string, returns the single nearest site whose status was
confirmed within the freshness window, including a human-readable elapsed-time-since-confirmed
line. If no site nearby has a fresh status, returns an explicit "status unknown" result instead of
a stale claim (BR-001).

**Inputs**
- `submitted_text` (string, from SMS body or web-lite form) — a barangay name or free-text location; validated against the known `Barangay` table with fuzzy match (typo-tolerant) before falling back to "not recognized."
- `phone_hash` (string, nullable) — present for SMS-originated lookups, null for anonymous web-lite lookups.

**Outputs**
- SMS/web-lite text response: site name, open/closed state, stock state (or "unknown"), hours,
  distance/address, freshness line ("confirmed N min/hr ago"), and reply-keyword actions (MAP, DONE).
- A `LookupQuery` row (data model) for §8 metrics.

**Business rules**
- **BR-001** — status only shown if inside the freshness window; else "status unknown."
- **BR-002** — response never includes a dosing instruction.
- **BR-004** — barangay-name fallback when precise geolocation is unavailable.

**State transitions**
N/A — F-001 is a stateless read per request; no multi-step state machine.

**Edge cases**
- Barangay string not recognized → prompt a retry with an example format (does not silently guess).
- Multiple sites tied for "nearest" within the same barangay → return the one with the most recent
  fresh status (breaks the tie by freshness, not by an arbitrary ID order).
- No site in the barangay has ever posted a status (new deployment, zero data) → "status unknown,"
  same as the stale case — the resident-facing behavior is identical whether data is stale or has
  never existed.

**Error handling**
- SMS gateway send failure → retried per `system-design.md` Integration points; if all retries fail,
  logged (not resident-facing, since the resident already sent the request and cannot be re-prompted
  without a new inbound message).

### F-002 — Exposure-window push alert

**Description**
When an operator marks a barangay flood-declared, sends one SMS alert to every opted-in resident
contact on file for that barangay, once per declaration (BR-003).

**Inputs**
- `barangay_id`, `declared_at`, `window_ends_at` (from the operator-facing declaration action).

**Outputs**
- One `AlertDelivery` row + one outbound SMS per eligible `ResidentContact`.

**Business rules**
- **BR-003** — at most one alert per `(flood_declaration_id, resident_contact_id)`, enforced at the
  database uniqueness level, not just in application code.

**State transitions**
`AlertDelivery.delivery_status`: `queued` → `sent` → `delivered` | `failed` (from SMS gateway
webhook callbacks).

**Edge cases**
- Zero contacts on file for a newly declared barangay → dispatch is a no-op; this is expected for a
  barangay with no prior resident opt-ins and is not an error condition.
- Declaration re-triggered for the same barangay while a prior declaration's window is still active
  → treated as a distinct `FloodDeclaration` row (a new event, e.g. a second flood), so residents can
  receive a new alert — BR-003 scopes uniqueness per declaration, not per barangay.

**Error handling**
- Gateway throttling during a large fanout → paced dispatch (queue, not a single burst),
  `system-design.md` Scaling strategy.

### F-003 — Staff one-tap status update

**Description**
Authenticated staff (per-site code) submit a stock state and open/closed state in one action; the
system records it with a server-assigned timestamp and makes it visible to F-001 within 5 seconds.

**Inputs**
- `site_id` (implied by the authenticated staff code), `stock_state`, `open_state`.

**Outputs**
- A new `SiteStatusUpdate` row; the staff panel's own display updates to confirm the write.

**Business rules**
- Timestamp is always server-assigned (`confirmed_at = now()`), never accepted from the client —
  this is load-bearing for INV-002's integrity, not just a convenience default.

**State transitions**
N/A — each submission is a new, independent log row (append-only, `data-model.md`).

**Edge cases**
- Staff code expired/rotated mid-shift → write rejected with a clear "code expired, contact your
  site administrator" message, not a silent failure.
- Rapid repeated taps (e.g., staff double-taps "In stock") → idempotent from the resident's
  perspective (both rows say the same thing); not deduplicated at the database level since each is
  a legitimate, independently timestamped confirmation.

**Error handling**
- Write failure (DB unavailable) → staff panel shows an explicit error and does not show a false
  "saved" confirmation.
