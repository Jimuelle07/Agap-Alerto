# OPS — Operations & Observability Runbook — Agap Alerto

> Traces back to: `system-design.md`, `security-compliance.md`. Selected because `outlives_demo: true`.

## Deploy
- **Environments:** one staging environment (SMS gateway sandbox mode), one production environment
  (live gateway) — `[assumption]`: no formal LGU-hosted environment identified yet; pilot runs on a
  managed PaaS per `system-design.md`.
- **Deploy command/pipeline:** CI runs the full gate (`pytest`) on every merge to the default
  branch; a green full gate auto-deploys to staging; production deploy is a manual promotion step
  (deliberate — an epidemic-response tool should not auto-deploy to production unattended).
- **Rollback:** redeploy the immediately prior successful build artifact; `SiteStatusUpdate` and
  other data are never rolled back (append-only), only the application code.
- **Pinned versions:** language runtime, framework, and the SMS gateway SDK are all pinned in the
  dependency lockfile; no floating versions in production.

## Configuration & secrets
- **Required config:** database connection string, SMS gateway API key, `freshness_window` duration
  (`technical-design.md`), staff-code hashing pepper.
- **Where secrets live:** hosting platform's secret manager / environment variables — never in the
  repository, never logged (`security-compliance.md` Secrets handling).
- **Rotation:** staff codes rotate per declared epidemic (operational action, not a deploy); gateway
  and DB credentials rotate per the hosting platform's standard schedule `[assumption]`.

## Observability
- **Logged/measured:** alert dispatch counts and delivery status per declaration; lookup counts and
  "fresh" vs. "status unknown" outcome rate (the §8 activation signal); staff status-update
  frequency per site (the A-002 signal from `seed/idea.md`).
- **SLIs that matter:** F-001 response latency (must be fast enough to feel immediate over SMS);
  F-003 write-to-visible latency (must stay under the 5-second acceptance criterion); SMS gateway
  delivery success rate.

## Alerts & thresholds
- Page the ops owner if: SMS gateway delivery failure rate exceeds a forward-looking threshold
  `[assumption — no historical baseline yet to set a specific number]`; if F-003 writes stop
  arriving from a site that has an active declaration nearby (a silent-staff signal, not just a
  system error); if the database is unreachable.

## Runbook — common incidents
| Symptom | Diagnosis | Fix |
|---|---|---|
| Residents report no alert received during a declared flood | Check `AlertDelivery` rows for the declaration; check gateway delivery status | If `queued`/`failed`, check gateway quota/outage; re-trigger dispatch for undelivered rows only (BR-003's unique constraint prevents duplicate sends to already-delivered contacts) |
| A site shows "status unknown" despite staff insisting they updated it | Check `SiteStatusUpdate` for that site — likely the staff code was invalid/expired (frd.md F-003 edge case) or the update is just outside the freshness window | Reissue/rotate the staff code; confirm the update lands with a fresh timestamp |
| F-001 responses are slow | Check DB query plan on the freshness-window read (`data-model.md` index) | Confirm the `(site_id, confirmed_at DESC)` index exists and is used |

## Backup & recovery
- Database: automated daily backups via the hosting platform's managed Postgres offering
  `[assumption — specific provider/cadence not chosen yet]`; restore procedure to be tested before
  the first real pilot (not yet exercised — an open item for `implementation-plan.md`).
- No backup is needed for the application code itself (recovered by redeploying from the version
  control history).
