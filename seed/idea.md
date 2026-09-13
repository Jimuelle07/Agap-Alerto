---
status: unvalidated-by-choice
schema_version: 2.1.0
validation: GO-UNVALIDATED
riskiest_assumption: A-001
---

# Idea: Agap Alerto

## 1. Problem statement
During Manila's declared leptospirosis epidemic, people who wade through contaminated floodwater
have a narrow window after exposure in which a free, government-supplied preventive medicine could
stop them from getting sick. The city has already put that medicine in 44 health centers and 7
district hospitals and is running a citywide information campaign, but a citywide campaign does not
tell any one person, today, which of those 51 sites is open right now, still has the medicine in
stock, and is close enough to reach before the exposure window closes. People without a habit of
using a particular clinic are left to guess, ask around, or wait — and by the time symptoms appear
and they go to a hospital instead, the hospitals are the ones already at capacity. Delayed treatment
is tied to a higher death rate, and the pain recurs every flood, not once.

## 2. Target segment
Informal-sector workers and market vendors living or working in flood-prone Manila barangays (e.g.,
Tondo, Baseco/Port Area, Sta. Ana, Pandacan) who were in floodwater during the current epidemic
window. No fixed employer, no regular family physician, relies on barangay health centers, likely has
only a basic phone. **Not in this segment:** salaried commuters who don't wade in floodwater;
residents of non-flood-prone Manila districts; people already symptomatic and hospitalized (their
prophylaxis decision point has passed); barangay health-center staff (a related, secondary segment —
the supply side of the same transaction, tracked separately in F-003).

## 3. Evidence
- City is in a declared epidemic; attack rate jumped 1.27%→3.48% in one week.
  > [!evidence] EV-001: Type: said | Source: https://www.pna.gov.ph/articles/1283462 | Date: 2026-09-07
- Free doxycycline is a real, existing distribution network, not a proposal — 44 health centers + 7 district hospitals.
  > [!evidence] EV-004: Type: did | Source: https://www.pna.gov.ph/articles/1283462 | Date: 2026-09-07
- 72% of Metro Manila cases are flood-exposure-linked; 55% among unemployed/informal workers — confirms the segment.
  > [!evidence] EV-005: Type: said | Source: https://www.philstar.com/headlines/2026/09/04/2553874/leptospirosis-cases-reach-6253-doh-warns-post-flood-surge | Date: 2026-09-04
- Delayed treatment ties to mortality; national CFR 6.05% (378/6,253 cases) — the cost of missing the window.
  > [!evidence] EV-006: Type: said | Source: https://www.philstar.com/headlines/2026/09/04/2553874/leptospirosis-cases-reach-6253-doh-warns-post-flood-surge | Date: 2026-09-04
- DOH itself frames the next two weeks as a resurgence risk window (incubation period) — this recurs, it is not one-time.
  > [!evidence] EV-007: Type: said | Source: https://www.philstar.com/headlines/2026/09/04/2553874/leptospirosis-cases-reach-6253-doh-warns-post-flood-surge | Date: 2026-09-04
- Existing barangay SMS network runs case data upward to health workers, not guidance downward to residents — the gap this idea closes.
  > [!evidence] EV-008: Type: did | Source: https://pia.gov.ph/features/beyond-the-flood-how-ilocos-sur-fights-the-hidden-health-risks-of-the-rainy-season/ | Date: 2026-01-01
- No existing Philippine digital tool surfaces real-time free-prophylaxis stock/hours at government sites.
  > [!evidence] EV-012: Type: did | Source: https://www.pna.gov.ph/articles/1283462 | Date: 2026-09-07

No `INT-###` (primary interview) evidence exists yet — see `work/evidence-ledger.md` Decisions. The
`Significant` test and `A-001` are unvalidated by any human conversation; this is the honest state,
not a gap the author papered over.

The four tests (scored from the ledger — see `work/evidence-ledger.md` for the validator's own scoring):

| Test | Pass / Fail | Why |
|---|---|---|
| Real | pass | EV-001, EV-002, EV-005 |
| Large | pass | EV-013 (Manila pop. 1,846,513), EV-004 (44+7 confirmed sites) |
| Significant | unknown | EV-006 is population-level; no individual `did`/`paid` evidence yet |
| Urgent | pass | EV-007 — DOH's own stated 2-week resurgence window |

## 4. Root cause (the WHY)
Manila (like the rest of the country) has SMS/barangay rapid-reporting infrastructure, but it is
built to push case counts *up* to health workers and DOH, not to push "go here, now, it's stocked"
guidance *down* to an exposed resident within the clinically relevant window. Five-whys: cases surge
→ hospitals fill → because prophylaxis wasn't taken in time → because residents didn't know
where/when to get it → because the existing alert infrastructure is a surveillance tool, not a
resident-facing access tool → because no one owns the "last mile" of a distributed government
benefit under time pressure (EV-008). A secondary, compounding root cause: DOH's own warning against
unsupervised self-medication with doxycycline (research brief R-010), while medically correct, may
suppress uptake among exactly this population unless matched by an equally clear "here is how to get
it FOR FREE, SUPERVISED, TODAY" message — this is inferred, not directly evidenced this session.

## 5. Market & alternatives
- **Size band:** tens of thousands per epidemic wave — source: PSA 2020 Census (Manila City pop.
  1,846,513, https://rssoncr.psa.gov.ph/content/highlights-2020-census-population-and-housing-city-manila,
  ~2021), narrowed by an `[assumption]` informal-settlement share.
- **Reachability:** (1) 44 health centers + 7 district hospitals — existing physical network
  (pna.gov.ph, 2026-09-07); (2) barangay SMS rapid-reporting network — existing, re-purposable
  (pia.gov.ph, 2026 undated).
- **Top 3 alternatives + their key failure:**
  1. City's general information campaign — fails at site-specific, same-day actionability.
  2. e-Konsulta / Bayanihan e-Konsulta teleconsultation — fails at being flood/lepto-specific and stock-aware.
  3. Do nothing / wait for symptoms — fails at cost: ties to a documented 6.05% national CFR.

## 6. Value proposition
For **informal workers and vendors in flood-exposed Manila barangays** who **don't know which of the
city's 51 free-doxycycline sites is open, stocked, and near them right now**, Agap Alerto is a
**same-day, SMS-first lookup and alert** that **names the specific nearest open, stocked site**,
unlike **the city's general information campaign**, because **it repurposes the barangay's existing
rapid-reporting rails to push actionable, site-specific guidance outward instead of only case data
inward**.

## 7. Feature set

### MVP — smallest path to core value + a learning signal
- **F-001** — SMS/web-lite lookup: text or check a barangay/location, get back the nearest open site currently marked in-stock, with hours **and a last-confirmed freshness timestamp** (e.g., "confirmed 14 min ago") → solves not knowing which of 51 sites to go to, right now, and satisfies INV-002's "stated freshness window" requirement so a resident can judge whether the status is stale enough to matter before making the trip (§1, EV-004, EV-008; usability.md cycle 1 K3 finding)
- **F-002** — Push alert to residents in a flood-declared barangay within the clinical prophylaxis window after a flood event → solves the "campaign reaches me eventually, not in time" gap (§1, EV-007)
- **F-003** — One-tap staff status update (in stock / low / out, plus open/closed) for health-center and hospital staff → solves stale information becoming actively misleading (§1, INV-002); this is A-002, tested alongside A-001

### Final product — full vision
- **F-101** — Multilingual (Filipino/English) plain-language content review pass, tuned to the segment's reading level
- **F-102** — Integration with the existing barangay SMS rapid-reporting system rather than a parallel channel
- **F-103** — Post-epidemic mode: same infrastructure repurposed for routine rainy-season advisories, not just declared epidemics

## 8. Success metrics
- **Activation:** % of alert recipients in a flood-declared barangay who look up or receive a
  same-day site-specific message within the prophylaxis window, **measured against a comparison
  baseline** — a staggered rollout across otherwise-similar flood-declared barangays (some receive
  F-002's alert, others see only the city's general campaign that week), or, failing that, a
  documented estimate of the general campaign's own same-day site-visit rate. A-001 claims a
  *meaningfully higher* rate than the general campaign alone; raw uptake cannot by itself prove or
  falsify that comparative claim (validation.md cycle 1, §2).
- **Retention:** % of health-center/hospital staff still updating stock status daily after week 2 of
  the epidemic (a proxy for whether F-003's effort bar is actually low enough).
- **Revenue / value:** N/A (free public-health service) — proxy value metric: reduction in "wasted
  trip" reports (resident went to a site marked open/stocked and found otherwise) relative to a
  baseline first week with no freshness enforcement.

## 9. Constraints, risks & kill criteria
**Riskiest assumption — A-001:** Informal workers/vendors in flood-exposed Manila barangays will act
on a same-day, site-specific alert within their exposure window, at a meaningfully higher rate than
they act on the city's existing general information campaign alone.

**A-002 (secondary):** Barangay health-center and hospital staff will keep stock/open status current
with near-zero extra effort; if updating it is not nearly free for them, F-001's information goes
stale and actively misleads people during an active epidemic.

**Kill criteria (explicit fail-states):**
- Regulatory: none identified — this is a government-run free-distribution program; a formal DOH/LGU
  data-sharing or health-data-privacy objection to publishing per-site stock status would be a kill
  signal (unconfirmed this session).
- Unit economics: N/A — the city already funds the drug and the physical network; this is an
  information/coordination layer, not a new paid good. A kill signal would be if no barangay/health
  center will supply even a once-per-shift status update (F-003 unusable → F-001 has nothing live to show).
- Technical: if the segment's SMS/basic-phone reach is materially lower than assumed (`[assumption]`
  in market.md), the channel itself fails and the idea needs a different distribution mechanism
  (e.g., barangay health worker relay instead of direct-to-resident SMS).

**Invariants (`INV-###`) — hard rules that must hold across every pivot:**
- **INV-001** — the system must never recommend or imply a specific dosing regimen to a resident
  without directing them to a supervising health worker (DOH explicitly warns against unsupervised
  self-medication with this exact drug — research brief R-010, R-011).
- **INV-002** — the system must never show a health center/hospital as open or stocked past a short,
  stated freshness window (a wasted trip during an active epidemic, toward hospitals already near
  capacity, actively harms the person it was meant to help).

## 10. Out of scope (for now)
- Vaccine development, rat-population/vector control, and drainage/floodway infrastructure — real
  root-cause interventions, but multi-year infrastructure/biomedical projects outside a software
  MVP's timescale (research brief §10 parking lot).
- A general leptospirosis-prevention education app — would duplicate the mayor's existing
  information campaign rather than closing the "which site, right now" gap this idea targets.
- Clinical treatment workflows for already-symptomatic/hospitalized patients — a different segment
  and decision point (§2 exclusions).
- Payment or insurance integration — the medicine and the information are both free; no billing surface exists in this MVP.
