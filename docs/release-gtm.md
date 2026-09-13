# Release / Go-to-Market & Roadmap — Agap Alerto

> Traces back to: `prd.md`, `seed/market.md`. Selected because `release_planning: true`.

## Release plan & phases
- **Alpha (1–2 barangays):** F-001/F-002/F-003 live in 1–2 flood-prone barangays (e.g., one Tondo
  barangay, one Baseco/Port Area barangay per `seed/research-brief.md` segment candidates), staffed
  by 2–4 health-center volunteers who already agreed to try F-003. Entry: gate exit 0 +
  `validation-report.md` PASS. Exit: ≥1 real flood/declaration event observed end-to-end, staff
  actually updating status without prompting.
- **Beta (citywide, all 51 sites):** rolled out across all 44 health centers + 7 hospitals once
  alpha's A-002 (staff-update willingness) is confirmed, not assumed. Exit: a full epidemic wave
  handled with F-002 dispatch at city scale.
- **GA (standing service):** F-103's post-epidemic mode active — the same infrastructure runs at a
  lower cadence for routine rainy-season advisories between epidemic declarations.

## Rollout / rollback strategy
- Feature-flagged by barangay: a barangay is only in scope for F-002 dispatch once explicitly
  enabled, so alpha can run without accidentally alerting the whole city.
- Rollback is a config change (disable the flag for a barangay/citywide), not a data rollback —
  `SiteStatusUpdate`/`AlertDelivery` history is append-only and never needs reverting (`ops.md`).

## Launch checklist
- [ ] `security-compliance.md` Pre-milestone hard-gate checklist passed, including the Data Privacy
      Act determination.
- [ ] At least one staff member per alpha site has completed a live F-003 update (not a demo).
- [ ] `qa-test-plan.md` full gate green on the default branch.
- [ ] The Manila Health Department (or equivalent LGU sponsor) has confirmed the initial site list
      and barangay coverage mapping — `[assumption]` in `prd.md` Dependencies, must be resolved
      before beta.

## GTM channels
From `seed/market.md` reachability — these are the channels used to *reach the segment*, distinct
from the product's own SMS delivery channel:
1. **The 44 health centers + 7 district hospitals themselves** — the fastest path to both staff
   buy-in (F-003) and resident awareness (posted signage pointing residents to text the shortcode).
2. **The existing barangay SMS rapid-reporting network** — a natural amplifier for awareness once
   F-102 integration exists; before that, barangay health workers can manually cross-promote.

## Success metrics & instrumentation
From `idea.md` §8, instrumented per `ops.md` Observability:
- **Activation:** % of F-002 alert recipients who reply and receive a fresh F-001 result within the
  exposure window, **measured against the comparison baseline** (staggered barangay rollout or a
  documented general-campaign baseline estimate) — `idea.md` §8, added per `validation.md` cycle-1
  recommendation.
- **Retention (proxy):** % of alpha-site staff still submitting F-003 updates daily after week 2.
- **Value (proxy):** reduction in "status unknown" responses (BR-001) over time as staff-update
  habits form, relative to the alpha baseline week.

## Post-launch learning loop
Weekly review during any active epidemic declaration: activation rate, staff-update frequency per
site, and any `[assumption]` in `seed/market.md` or `prd.md` Open questions that real data has now
resolved (feeds `decision-ledger.md` §2).

## Roadmap beyond MVP
Sequenced per `idea.md` §7 final scope:
1. **F-101** — multilingual/plain-language content pass, once alpha surfaces real reading-level
   friction (not preemptively).
2. **F-102** — integration with the existing barangay SMS rapid-reporting system, once a concrete
   integration point is identified (currently an open dependency, `prd.md`).
3. **F-103** — post-epidemic routine mode, once beta demonstrates the alert/lookup loop holds up at
   citywide scale during an actual epidemic.
