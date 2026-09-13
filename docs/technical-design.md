# Technical Design Document (LLD) — Agap Alerto

> Traces back to: `system-design.md`.

## Module breakdown
- **`gateway/`** — SMS Gateway Adapter. Public interface: `send_sms(to_hash, body) -> DeliveryResult`,
  `parse_inbound(webhook_payload) -> InboundMessage`. Collaborates with `core/dispatch.py` and
  `core/lookup.py`.
- **`core/lookup.py`** — F-001. Public interface: `resolve_nearest_site(submitted_text, freshness_window) -> LookupResult`.
- **`core/dispatch.py`** — F-002. Public interface: `dispatch_alert(flood_declaration_id) -> list[AlertDelivery]`.
- **`core/status.py`** — F-003. Public interface: `record_status(staff_code, stock_state, open_state) -> SiteStatusUpdate`.
- **`core/copy_guard.py`** — shared by `lookup.py` and any outbound template: `assert_no_banned_terms(text) -> None` (raises if INV-001 banned dosing terms are present; used both by tests TC-N01 and defensively at send time).
- **`web/staff_panel.py`** — thin FastAPI routes for F-003's browser surface; delegates to `core/status.py`.
- **`store/`** — Postgres access layer (SQLAlchemy models matching `data-model.md` exactly; no ORM logic duplicating business rules — those stay in `core/`).

## Class / function-level design

```
# core/lookup.py
def resolve_nearest_site(submitted_text: str, freshness_window: timedelta) -> LookupResult:
    barangay = match_barangay(submitted_text)          # BR-004 fallback logic
    if barangay is None:
        return LookupResult(kind="not_recognized")
    site, status = store.latest_fresh_status(barangay.id, freshness_window)
    if status is None:
        return LookupResult(kind="status_unknown", site=None)
    text = render_result(site, status)                  # includes freshness line, BR-002 safe
    copy_guard.assert_no_banned_terms(text)              # INV-001 defensive check before send
    return LookupResult(kind="fresh", site=site, status=status, text=text)
```

```
# core/dispatch.py
def dispatch_alert(flood_declaration_id: UUID) -> list[AlertDelivery]:
    declaration = store.get_declaration(flood_declaration_id)
    contacts = store.opted_in_contacts(declaration.barangay_id)
    deliveries = []
    for contact in paced(contacts):                      # rate-paced, system-design.md Scaling
        delivery = store.create_alert_delivery(declaration.id, contact.id)  # unique constraint = BR-003
        if delivery is not None:                           # None if the unique constraint rejected a dup
            gateway.send_sms(contact.phone_hash, ALERT_TEMPLATE)
            deliveries.append(delivery)
    return deliveries
```

```
# core/status.py
def record_status(staff_code: str, stock_state: StockState, open_state: OpenState) -> SiteStatusUpdate:
    site = store.site_for_staff_code(staff_code)          # raises AuthError if invalid/expired
    return store.insert_status_update(site.id, stock_state, open_state, confirmed_at=utcnow())
    # confirmed_at is ALWAYS server-assigned — never accept a client-supplied timestamp (INV-002 integrity, frd.md)
```

## Algorithms
- **Freshness check:** `now() - status.confirmed_at <= freshness_window` — a single comparison, no
  approximation; `freshness_window` is a configuration value (`ops.md` Configuration), not hardcoded,
  so it can be tuned per epidemic without a redeploy.
- **Barangay fuzzy match (BR-004):** exact match first, then a simple edit-distance fallback (e.g.,
  Levenshtein ≤ 2) against the known `Barangay.name` set — deliberately simple; no geocoding
  dependency for MVP (`system-design.md` trade-offs).
- **Alert pacing:** a bounded-concurrency queue (e.g., N sends/second, N from the SMS gateway's
  documented rate limit `[assumption — exact limit not sourced this session]`) rather than firing
  every contact's send concurrently.

## Sequence diagrams

```
Resident            SMS Gateway         Agap Alerto Core        Site/Status Store
   |  "text SITE"  ---->|                     |                        |
   |                     |--inbound webhook -->|                        |
   |                     |                     |--latest_fresh_status-->|
   |                     |                     |<--site+status----------|
   |                     |<--reply text--------|                        |
   |<--SMS reply---------|                     |                        |

Staff                Agap Alerto Core        Site/Status Store
   |--POST status update-->|                        |
   |                       |--insert (confirmed_at=now())-->|
   |                       |<--ok----------------------------|
   |<--confirmation--------|                        |
```

## Error-handling strategy
- Gateway errors (send/receive) are caught at the adapter boundary and logged with a delivery
  status, never surfaced to the resident as a raw exception (there is no retry-prompt UX for a
  resident who already sent one SMS).
- `AuthError` (invalid/expired staff code) surfaces a specific, actionable message on the staff
  panel; never a generic 500.
- `copy_guard.assert_no_banned_terms` raising is treated as a **hard stop** on that send — better to
  fail the request than risk an INV-001 breach reaching a resident.

## Key decisions
- Server-assigned `confirmed_at` (never client-supplied) — promoted from `frd.md`; significant
  enough for an ADR given it is the single mechanism enforcing INV-002's integrity end-to-end.
- Simple edit-distance barangay match over a geocoding service for MVP — a deliberate scope cut
  (`system-design.md` trade-offs), revisit if `[assumption]` on segment device/location access
  resolves toward "residents can share precise location."
