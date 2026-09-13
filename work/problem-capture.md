# Problem capture — Manila Lepto Prophylaxis Access

**Path:** full · **Date:** 2026-09-13 · **Captured with:** Key author, from `work/research-brief.md`
(phase-0) and the user's original problem statement (PNA article 1283462). No interviews conducted
this session — see §3 tag discipline and phase-2 decision.

## 1. The pain (one paragraph, no product words)
During Manila's declared leptospirosis epidemic, people who wade through contaminated floodwater —
mostly informal workers and market vendors in low-income, flood-prone barangays — have a narrow
window after exposure in which a free, government-supplied preventive medicine could stop them from
getting sick. The city has already put that medicine in 44 health centers and 7 district hospitals
and is running a citywide information campaign. But a citywide campaign does not tell any one person,
today, which of those 51 sites is open right now, still has the medicine in stock, and is close
enough to reach before their exposure window closes. People who don't already have a habit of using a
particular clinic are left to guess, ask around, or wait — and by the time symptoms appear and they
go to a hospital instead, the hospitals are the ones already at capacity (Ospital ng Maynila, Sta. Ana
Hospital, Justice Jose Abad Santos General Hospital). The cost of guessing wrong isn't abstract: DOH's
own numbers tie delayed treatment to a higher death rate, and the health department itself has said a
second wave of cases is likely within two weeks because of the incubation period — this is not a
one-time decision, it recurs every flood.

## 2. Who has it (the segment)
- **Role:** Informal-sector worker or market vendor living or working in a flood-prone Manila
  barangay (e.g., Tondo, Baseco/Port Area, Sta. Ana, Pandacan) who was in floodwater during the
  current epidemic window.
- **Context (situation / tools they already use):** No fixed employer or regular family physician;
  paid daily or per transaction, so time away from work has a direct cost; relies on barangay health
  centers rather than a private clinic; already has access to word-of-mouth and barangay
  announcements, but not to real-time information; likely has a basic phone capable of receiving SMS,
  with data/smartphone access unconfirmed `[assumption]`.
- **Frequency (how often the pain recurs):** Every major flooding event during rainy season — several
  times a year in Manila — and, within any single event, the pain is acute for the ~72-hour clinical
  window after exposure.
- **Who is NOT in this segment (and why):** Salaried workers who commute through but don't wade in
  floodwater (lower exposure, and more likely to already have a regular doctor); residents of Manila's
  non-flood-prone districts; people already symptomatic and hospitalized (their decision point — get
  prophylaxis in time — has already passed, they need clinical treatment, not access information);
  barangay health-center staff themselves (a related but distinct segment — see below, they are the
  supply side of the same transaction, not the pain-holder this capture centers on).

## 3. What they do today (workaround) and what it costs
- **Workaround:** `[said]` Wait until symptomatic and then go to a hospital, OR self-medicate with
  over-the-counter doxycycline without medical supervision, OR rely on word-of-mouth/barangay
  announcements to eventually learn where the free dose is available, OR simply wait for the city's
  general information campaign to reach them.
- **Cost — time:** `[said]` Unbounded — depends on when the campaign or word-of-mouth reaches them,
  not on their own exposure timing (research brief R-007). **money:** `[said]` None if they never
  seek care, out-of-pocket if they buy doxycycline over the counter themselves instead of taking the
  free supervised dose. **risk:** `[said]` Delayed antimicrobial treatment is a documented mortality
  risk factor (research brief R-006, national CFR 6.05%); unsupervised self-dosing risks improper use,
  which is exactly what DOH's own public warning targets (research brief R-010).

## 4. First guess at the riskiest assumption
- **A-001:** Informal workers/vendors in flood-exposed Manila barangays will act on a same-day,
  site-specific alert ("Barangay Health Center X is open and stocked, 10 minutes from you") within
  their exposure window, at a meaningfully higher rate than they act on the city's existing general
  information campaign alone.
- **A-002 (optional):** Barangay health-center and hospital staff will keep a stock/open status
  current with near-zero extra effort (e.g., one tap, once per shift) — if updating it is not nearly
  free for them, the resident-facing information becomes stale and the tool actively misleads people
  during an active epidemic.

## 5. Competition fit (only if judged)
- N/A — no competition/theme context was supplied for this run; `context.md` sets `judged: false`.

## Parking lot (solution words that came up — park them here)
- SMS/low-bandwidth alert-and-lookup channel that repurposes the existing barangay rapid-reporting
  rails (candidate MVP direction; not yet a committed feature).
- A staff-facing one-tap stock/status update mechanism (candidate MVP direction).
- Rat population control / drainage infrastructure rehabilitation — real root-cause interventions,
  explicitly out of scope for a software/information MVP on this timescale (research brief §10).
- A general leptospirosis-prevention education app — would duplicate the mayor's existing information
  campaign rather than closing the "which site, right now" gap; parked, not pursued.
