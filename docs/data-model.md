# Data Model / Schema — Agap Alerto

> Traces back to: `system-design.md`. Traces forward to: `technical-design.md`,
> `security-compliance.md`.

## Entities & relationships (ERD)
```
Barangay 1---* Site 1---* SiteStatusUpdate
Barangay 1---* ResidentContact
Barangay 1---* FloodDeclaration 1---* AlertDelivery *---1 ResidentContact
Site 1---* LookupQuery (resolved_site, nullable)
Site 1---1 StaffCode
```

### `Barangay`
| Field | Type | Null? | Default | Description |
|---|---|---|---|---|
| id | uuid | no | gen | primary key |
| name | text | no | — | barangay name, unique within Manila City |
| district | text | yes | null | Manila city district (e.g. Tondo, Sta. Ana) |

### `Site`
| Field | Type | Null? | Default | Description |
|---|---|---|---|---|
| id | uuid | no | gen | primary key |
| name | text | no | — | e.g. "Barangay 105 Health Center" |
| kind | enum(health_center, hospital) | no | — | one of the 44 centers or 7 hospitals (`idea.md` EV-004) |
| barangay_id | uuid FK → Barangay | no | — | which barangay it primarily serves |
| address | text | no | — | for the F-001 result's Address line |
| hours_text | text | no | — | free-text hours (e.g. "8am–5pm weekdays") — `[assumption]`: real-time hours feed not confirmed available; static text for MVP |
| staff_code_id | uuid FK → StaffCode | no | — | the credential staff use for F-003 |

### `StaffCode`
| Field | Type | Null? | Default | Description |
|---|---|---|---|---|
| id | uuid | no | gen | primary key |
| code_hash | text | no | — | hashed per-site access code (never store plaintext — `security-compliance.md`) |
| site_id | uuid FK → Site | no | — | 1:1 with Site |
| issued_at | timestamptz | no | now() | for rotation tracking |

### `SiteStatusUpdate`
Append-only log; the *current* status for F-001 is the newest row per `site_id` whose
`confirmed_at` is inside the freshness window (config, see `technical-design.md`).
| Field | Type | Null? | Default | Description |
|---|---|---|---|---|
| id | uuid | no | gen | primary key |
| site_id | uuid FK → Site | no | — | which site |
| stock_state | enum(in_stock, low, out) | no | — | F-003 input |
| open_state | enum(open, closed) | no | — | F-003 input |
| confirmed_at | timestamptz | no | now() | server-assigned on write; never client-supplied (INV-002 integrity) |
| confirmed_by_staff_code_id | uuid FK → StaffCode | no | — | which credential wrote it |

### `FloodDeclaration`
| Field | Type | Null? | Default | Description |
|---|---|---|---|---|
| id | uuid | no | gen | primary key |
| barangay_id | uuid FK → Barangay | no | — | which barangay is declared |
| declared_at | timestamptz | no | now() | start of the exposure window |
| window_ends_at | timestamptz | no | — | end of the clinically relevant window (F-002 trigger scope; `[assumption]` default duration — see open questions) |
| declared_by | text | no | — | operator identity (v1: manual staff entry; F-102: system-derived) |

### `ResidentContact`
| Field | Type | Null? | Default | Description |
|---|---|---|---|---|
| id | uuid | no | gen | primary key |
| phone_hash | text | no | — | one-way hash of phone number, never the raw number (see Retention & privacy) |
| barangay_id | uuid FK → Barangay | yes | null | best-known barangay, if any |
| opted_in | boolean | no | true | consent flag for receiving F-002 alerts |
| source | enum(self_reported, barangay_roster) | no | self_reported | how the number was collected |

### `AlertDelivery`
| Field | Type | Null? | Default | Description |
|---|---|---|---|---|
| id | uuid | no | gen | primary key |
| flood_declaration_id | uuid FK → FloodDeclaration | no | — | which declaration triggered it |
| resident_contact_id | uuid FK → ResidentContact | no | — | who it went to |
| sent_at | timestamptz | no | now() | |
| delivery_status | enum(queued, sent, delivered, failed) | no | queued | from SMS gateway webhook |

### `LookupQuery`
For §8 metrics (activation, comparative baseline) — not resident-identifying beyond the same
`phone_hash` already used for delivery.
| Field | Type | Null? | Default | Description |
|---|---|---|---|---|
| id | uuid | no | gen | primary key |
| phone_hash | text | yes | null | null for a web-lite, non-SMS lookup |
| submitted_text | text | no | — | raw barangay/location string submitted |
| resolved_site_id | uuid FK → Site | yes | null | null if no site had a fresh status (BR-001 stale case) |
| responded_at | timestamptz | no | now() | |
| via_alert_delivery_id | uuid FK → AlertDelivery | yes | null | set when the lookup followed an F-002 alert reply (feeds the §8 comparative-baseline metric) |

## Constraints & indexes
- `Site.barangay_id`, `Barangay` indexed for the F-001 nearest-site query (v1: barangay-match, not
  geodistance — see Open questions in `prd.md`).
- `SiteStatusUpdate(site_id, confirmed_at DESC)` index — F-001's hot-path read is "latest row per
  site within window."
- `ResidentContact.phone_hash` unique — one contact record per resident.
- `AlertDelivery` unique on `(flood_declaration_id, resident_contact_id)` — enforces BR-003 (at most
  one alert per declared event per phone number) at the database level, not just application logic.
- `StaffCode.code_hash` unique.

## Retention & privacy classification
| Field | Classification | Retention | Deletion |
|---|---|---|---|
| `ResidentContact.phone_hash` | PII (hashed) | duration of the active epidemic + 90 days, then purge | scheduled job; also honors a resident's opt-out via reply keyword (final scope) |
| `StaffCode.code_hash` | secret-adjacent (credential) | rotated per epidemic declaration | rotation invalidates prior hash |
| `SiteStatusUpdate`, `Site`, `Barangay` | internal/public (site operating status is meant to be shown to residents) | indefinite (operational history) | — |
| `LookupQuery.submitted_text` | internal, low-sensitivity | 90 days, then purge or aggregate-only | raw text never includes more than a barangay/location string per BR-004 |

No field stores a resident's health status, diagnosis, or medication history — the system's data
model is deliberately limited to "where/when someone looked for or was alerted about a site,"
consistent with INV-001 (never a dosing/medical-advice surface).

## Migration notes
Greenfield schema — no prior version to migrate from. `SiteStatusUpdate` is append-only by design
so a future analytics need (e.g., stock-out frequency per site) never requires a backfill; only new
read queries.
