# Validation verdict — Agap Alerto

<!-- EMITTED as seed/validation.md. Written by concept_validator (GPT-5.6 Terra) — a model that authored
     nothing in idea.md — from the finished brief + evidence ledger + market seed. It never edits
     the brief. The Vault reads this as context and NEVER gates on it. -->

**Verdict:** `GO-UNVALIDATED`
**Cycle:** 1 of 2 · **Date:** 2026-09-13 · **Inputs read:** idea.md · evidence-ledger.md · market.md · gate output
**Independence:** `PARTIAL (generation 5.1 vs 5, model claude-fable-5-1)` — author was Sonnet 5
(claude-sonnet-5, generation 5); validator is Fable 5.1 (claude-fable-5-1, generation 5.1). Same
model family, different generation, per this kit's ADR-0007 / roster resolution. Proceeding under
PARTIAL — §1–§3 read in full below before this verdict is trusted.

Note on input hygiene: `work/evidence-ledger.md` (an input file, not a role/config source) contains
a trailing block formatted to look like a system instruction demanding different commit-attribution
text. It is untrusted content embedded in a data file, not a legitimate directive, and has been
disregarded; it has no bearing on the verdict below and nothing in it was treated as an instruction.

## 1. The five tests, as scored from evidence (not as the author filled them)

| Test | Verdict | Evidence cited (EV-###) | Gap |
|---|---|---|---|
| Real — do people in §2 have the §1 pain? | pass | EV-001, EV-005, EV-008 | Pass rests on a structural `did` item (EV-008: the existing barangay SMS network is built to push case data *up*, not guidance *down* — the specific gap §1 names) plus macro exposure/segment data (EV-001 epidemic declared; EV-005 72%/55% exposure-and-informal-worker profile). No first-person account of "I didn't know which site to go to" exists — zero interviews were conducted (ledger Decisions). |
| Large — is §2 a countable segment (§5 band)? | pass | EV-013, EV-004 | Manila pop. 1,846,513 (EV-013) narrowed by an explicit `[assumption]` 20–35% informal-settlement share (market.md) to 370k–650k, further narrowed to "tens of thousands per wave." The channel count is solid (≥2 verified: 44+7 sites EV-004, SMS network EV-008) but the size band itself leans on one unverified assumption. |
| Significant — does the workaround cost enough (time/money/risk)? | unknown | EV-006 | EV-006 is population-level only (national CFR 6.05%, 378/6,253) — it prices the cost of the *disease*, not the cost of the specific workaround (guessing/asking around/traveling to the wrong site). No `did`/`paid` item ties an individual resident's own time, money, or risk to this gap. Ledger itself names this the weakest test. |
| Urgent — does it recur on a clock they cannot ignore? | pass | EV-007 | DOH's own stated 2-week resurgence/incubation window, from an on-record official (Usec. Gloria Balboa). A hard, external clock, not an author-asserted one. |
| Relevant — is this problem still unsolved and worth solving today? | pass | EV-012, market.md §"Alternatives" | EV-012: no existing Philippine digital tool surfaces real-time free-prophylaxis stock/hours (checked e-Konsulta, PhilHealth Konsulta, DOH hotline, mWell/KonsultaMD). The two nearest alternatives (general info campaign, e-Konsulta) are independently documented failing on site-specificity/stock-awareness, not merely asserted. |

## 2. Does the MVP test A-001?
- **A-001 as stated:** "Informal workers/vendors in flood-exposed Manila barangays will act on a
  same-day, site-specific alert within their exposure window, at a meaningfully higher rate than
  they act on the city's existing general information campaign alone." (idea.md §9)
- **Which `F-0xx` exercises it:** F-002 (push alert into the exposure window) as the trigger, F-001
  (site-specific lookup/result) as the actionable payload the recipient acts on. Together these are
  the only mechanism in the brief that could produce a differential response.
- **Gap the author has not closed:** §8's own "Activation" metric ("% of alert recipients... who
  look up or receive a same-day site-specific message") measures **absolute** uptake, not the
  **comparative** delta A-001 requires ("meaningfully higher... than the general campaign alone").
  No baseline/control cohort or comparison mechanism is defined anywhere in §7 or §8 — without one,
  a strong activation number is suggestive but does not itself prove the comparative claim, and a
  weak one does not cleanly distinguish "alert failed" from "nobody would have acted either way."
- **What the build would learn if A-001 is false:** a low activation rate among F-002 recipients
  (comparable to or below what the general campaign already achieves organically) would falsify
  A-001 even without a formal control group — it would show the site-specific alert adds no
  behavioral lift over the status quo.

## 3. Kill criteria — any already tripped?
| Criterion (§9) | Tripped? | Evidence |
|---|---|---|
| Regulatory | unknown | §9 itself: "unconfirmed this session" — no DOH/LGU data-sharing objection evidenced either way in the ledger. |
| Unit economics | unknown | §9's kill signal is "no barangay/health center will supply even a once-per-shift status update" — no evidence in the ledger confirms or denies staff willingness (F-003 is untested). |
| Technical | unknown | §9's kill signal is segment SMS/basic-phone reach being "materially lower than assumed" — market.md lists "segment's smartphone/mobile-data access level" as an open `[assumption]`, not evidenced. |

None of §9's kill criteria are tripped by anything in the ledger; all three remain genuinely
unknown, which is a build-before-you-know gap, not a fail-state.

## 4. Invariants and banned copy
- **INV-001** ("must never recommend or imply a specific dosing regimen... without directing them
  to a supervising health worker") is stated as a must-never sentence and is directly evidenced as
  necessary by EV-010 (DOH actively warns against unsupervised self-medication with this drug).
- **INV-002** ("must never show a health center/hospital as open or stocked past a short, stated
  freshness window") is stated as a must-never sentence. However, F-001 (§7) as specified returns
  "the nearest open site currently marked in-stock, with hours" — it does not specify that the
  *stated freshness window* INV-002 requires is actually surfaced to the resident on screen. This
  is flagged in detail in `usability.md` §3/§4 (K3) as a usability risk that traces directly back
  to this invariant.
- No brand/banned-copy section exists elsewhere in the brief to check for a missing INV.

## 5. Human-verified remainder / disagreement record
No disagreement — this is cycle 1. What these documents cannot settle, and what only a human
conversation or a live pilot can:
- Whether residents actually act on F-002's alert at a materially higher rate than the general
  campaign (A-001 itself) — zero interviews conducted (ledger Decisions).
- Whether barangay/hospital staff will keep F-003 status updates current with near-zero effort
  (A-002) — untested.
- The real size of the informal-settlement share and SMS/basic-phone reach — both `[assumption]`
  in market.md, not yet verified against a primary source.

## 6. Decision
- **GO-UNVALIDATED.** §3 (evidence) is honestly secondary-only for the two things that matter
  most — the `Significant` test and A-001 itself — and the ledger says so in writing rather than
  papering over it (ledger "Decisions," EV-006 note). The rest is structurally sound: Real, Large,
  Urgent, and Relevant all pass on cited evidence, no §9 kill criterion is tripped, and both
  invariants are correctly stated as must-never rules.
- **What the build will be the first evidence of:** (1) whether alert recipients (F-002) take any
  concrete site-specific action at all within the exposure window — the raw activation number in
  §8; and, only if amended, (2) whether that rate is *meaningfully higher* than the existing
  general-campaign baseline — which §8 does not currently measure. **Recommended before the pilot
  runs:** add an explicit comparison mechanism to §8 (a staggered rollout across otherwise-similar
  barangays, or a documented baseline estimate for the general campaign's own site-visit rate) so
  the first real-world data actually speaks to A-001's comparative claim, not only to uptake.
