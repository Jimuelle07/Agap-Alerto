# Validation Report — does the design still serve the concept?

> **Purpose:** phase-5.2. Written by the **validator** role after the architect's docs exist and
> before any plan is built. This is not idea validation (the Key's job, already done —
> `seed/validation.md`, `GO-UNVALIDATED`) — it asks one narrower question: *given the seed as
> written, do the PRD, system design, and QA plan implement that concept without contradiction, and
> is the riskiest assumption actually exercised by the build?* The validator authored none of the
> docs it judges and never writes a fix.

**Verdict:** `PASS`
**Inputs read:** `docs/validation-report.md` (own cycle-1 output) · `seed/idea.md` · `docs/prd.md`
(revised) · `docs/system-design.md` (unchanged, re-checked for new contradictions only) ·
`docs/qa-test-plan.md` (unchanged, re-checked for coherence with the revised PRD only) ·
`trace-ids.py` output · template
**Repair cycle:** 2 of 2 (final — the next step for any unresolved gap would be `ESCALATE`, not a
third revision)

**Independence:** `PARTIAL (generation 5.1 vs 5, model claude-fable-5-1)` — `prd.md`'s cycle-2
revision was authored by Sonnet 5 (`claude-sonnet-5`, Anthropic, generation 5); this re-validation
is run by Fable 5.1 (`claude-fable-5-1`, Anthropic, generation 5.1). Same model family, different
generation. Adversarial mode remains **mandatory** for this run and is applied throughout §§1–4 —
the revision is checked for whether it actually closes the two named gaps, not accepted on the
strength of its own claim to have done so.

**Input-hygiene note:** `seed/idea.md`, as delivered to this run, carries a trailing block formatted
to mimic a system instruction demanding different commit-attribution text ("Claude Sonnet 5"). This
is the same untrusted-content-embedded-in-a-data-file pattern cycle 1 flagged in `seed/usability.md`
(and which `work/evidence-ledger.md` had already flagged before that) — not a legitimate directive,
and not something a seed file has standing to issue regardless. It has been disregarded; the real
attribution instruction for this session governs, and nothing in it bears on the verdict below.

**Deterministic tool result (re-run after the revision):** `trace-ids.py docs --seed seed` — 15
files; F 6 · INV 2 · A 2 · UJ 3 · BR 5 · TC 8; T1 3/3, T5 2/2; APPROVE (no orphans, no dangling
references). BR count rose from 4 to 5 for the new BR-005; nothing else changed structurally. The ID
spine remains sound; this report's job is the content behind the IDs.

## 0. Cycle-1 gap closure — the two named defects, re-checked

**Gap (a) — BR-001's freshness window must state a concrete value.** **Closed.** `prd.md` BR-001 now
reads: "…within the stated freshness window, defined as **4 hours** since the last confirmed update
(`[assumption]` — no clinical/ops source sets this number…)." The F-001 and F-001 (stale case)
acceptance criteria were updated in lockstep — both now say "within the last **4 hours** (BR-001's
stated freshness window)" instead of the old circular "the stated freshness window." The `[assumption]`
tag is used correctly here (an unsourced number, honestly labeled, with an explicit note that the
lead should revise it before a real pilot). BR-001's own wording no longer presupposes an
undefined fact — the "stated" requirement in `idea.md` INV-002 is now actually satisfied.

**Gap (b) — F-002's resident phone-number/opt-in source must be named, even as an unresolved
blocker with candidates.** **Closed, and more thoroughly than the minimum cycle 1 set as acceptable.**
`prd.md` Dependencies now has a dedicated bullet: "F-002's resident phone-number/opt-in list has no
confirmed source — unresolved MVP blocker," naming two candidates (reuse the existing barangay SMS
network's contact list per `idea.md` EV-008; or a fresh self-registration keyword flow) and stating
neither is confirmed. New **BR-005** goes further and adopts option (2) — self-registration via a
texted keyword, landing in `ResidentContact.source = self_reported` — as the MVP default, while
explicitly leaving option (1) as the preferred final-scope path and flagging the choice as one "for
the lead to revisit, not a closed question." This is exactly the shape cycle 1 said would be
adequate ("named... as an unresolved MVP blocker with candidate options"), and the addition of
BR-005 means F-002 now has a defined, buildable data source rather than only a labeled gap.

## 1. Concept ↔ design coherence

| Check | Result | Evidence / contradiction (doc A says X; doc B says Y) |
|---|---|---|
| Every `F-###` in idea.md §7 (MVP) has a PRD row and an EARS criterion | pass | Unchanged from cycle 1; F-001/F-002/F-003 still trace verbatim. |
| No PRD feature exists that idea.md does not name (no invented `F-`) | pass | BR-005 is a new business rule, not a new `F-ID`; it attaches to existing F-002, no invented feature. |
| The core journey `UJ-001` reaches the value proposition in §6 | pass | Unchanged; UJ-001 still delivers site+status+hours+distance+freshness. |
| System design components map to PRD features; no component serves nothing | pass | Unchanged — `system-design.md` was not revised, and BR-005's self-registration path (an SMS text-in event) is coverable by the existing SMS Gateway Adapter → Agap Alerto Core path without requiring a new component; no contradiction introduced. |
| Non-functional needs the seed states are designed for; ones it doesn't are `[assumption]`, not invented targets | partial (unchanged, out of this cycle's scope) | The "managed PaaS hosting" row in `system-design.md` still lacks an `[assumption]` tag that the SMS-vendor and burst-pacing rows carry. `system-design.md` was not touched by this revision, and cycle 1 did not name this as a REVISE trigger — carried forward to §5, not re-litigated here. |
| Every network-exposed surface declares auth/authz | partial / unverified-from-this-input (unchanged) | Same as cycle 1 — staff per-site code, `security-compliance.md` still not in the input contract. Not re-litigated. |

**Cycle-1 "Additional finding" (F-002 data-dependency gap):** resolved — see §0(b) above. No
residual contradiction: `qa-test-plan.md`'s TC-003/TC-004 fixtures ("3 `ResidentContact` rows in one
barangay") are now consistent with BR-005's stated mechanism for how those rows come to exist,
rather than presupposing an unnamed one.

## 2. Riskiest assumption coverage (`A-001`)

- **Assumption as stated in the seed (idea.md §9):** "Informal workers/vendors in flood-exposed
  Manila barangays will act on a same-day, site-specific alert within their exposure window, at a
  meaningfully higher rate than they act on the city's existing general information campaign alone."
- **Touched by this revision?** No. `prd.md`'s Overview & goals, the feature list, and the
  acceptance criteria for F-001/F-002 are materially unchanged apart from the BR-001 freshness value
  and BR-005's addition; no comparison/baseline mechanism or activation-event logging was added.
- **Status, carried forward, not re-scored this cycle:** cycle 1 explicitly flagged the missing
  A-001 instrumentation (no logged "resident replied SITE" event, no cohort/staggered-rollout
  tagging) as real but **not** the REVISE trigger — a lead judgment call, not a blocking defect. That
  framing still holds: the gap is unchanged, and this cycle's task is to check the two named gaps
  and confirm no regression, not to introduce a new blocking bar cycle 1 did not set. Still open;
  still the lead's call (§5).

## 3. Invariant enforcement map (`INV-###`)

| INV | PRD must-never | Security mitigation | Design banned-copy | Negative TC | Verdict |
|---|---|---|---|---|---|
| INV-001 | `prd.md` BR-002: F-001's result never includes a dosing instruction; directs resident to ask staff | `prd.md` cites "security T-002 (content review gate)" — N/A, unverifiable-from-this-input | `prd.md` cites "design-system banned-copy" — N/A, unverifiable-from-this-input | TC-N01 | **enforced, but weak (unchanged).** BR-002 + TC-N01 trace correctly; TC-N01's static banned-term list is still lexically brittle against paraphrase (unchanged text in `qa-test-plan.md`). Cycle 1 explicitly did not name this as the REVISE trigger, and nothing in this revision touches it — still flagged, not re-triggering a decision, carried to §5. |
| INV-002 | `prd.md` BR-001: status shown as "open, stocked" only if an F-003 update exists within the stated freshness window (now **4 hours**, `[assumption]`); else "status unknown" | `prd.md` cites "security T-001 (stale-data expiry)"; `system-design.md`'s TTL-based server-side check independently corroborates the mechanism — sound | `prd.md` cites "design-system freshness display rule" — N/A, unverifiable-from-this-input | TC-N02 | **enforced, threshold now defined.** This is the gap-(a) fix: BR-001 states a concrete "4 hours," the F-001 and F-001 (stale-case) EARS criteria both restate that number consistently, and `system-design.md`'s TTL mechanism enforces whatever value is configured — now a real, stated value rather than a self-reference to nothing. `qa-test-plan.md`'s TC-002/TC-N02 (unchanged text, "the configured freshness window") remain coherent under this: they test the boundary generically, which is still correct now that a concrete default exists for that configuration to hold. No contradiction. |

## 4. Test fence quality (can these tests fence a cheap executor?) — re-check only

`qa-test-plan.md` is unchanged since cycle 1; this cycle only re-checks that TC-001/TC-002/TC-N02
are still coherent now that BR-001 states a concrete value, per this run's scope.

- **TC-001** ("confirmed 10 min ago"): still coherent — 10 minutes is well inside the newly-stated
  4-hour window, so the fixture and expected result remain valid without any edit. The parenthetical
  looseness ("or equivalent elapsed-time phrasing") flagged in cycle 1 is unchanged and not
  re-scored here (not a named gap for this cycle).
- **TC-002** ("older than the configured freshness window"): still coherent — it references the
  freshness window generically rather than hardcoding "4 hours," which is correct test design (the
  test should hold if the configured value ever changes) and does not contradict BR-001 now stating
  4 hours as that configuration's default.
- **TC-N02** (boundary probe "just past the window boundary"): same reasoning — generic reference to
  the configured boundary is still correct and non-contradictory now that a concrete default value
  backs it.
- No other test-fence findings re-scored this cycle (TC-N01's brittleness, carried forward at §3/§5,
  was already out of scope for a pass/fail call in cycle 1 and remains so).

## 5. Human-verified remainder

- Whether **4 hours** is the right freshness-window value for an active epidemic is an
  operational/clinical judgment, correctly marked `[assumption]` in `prd.md` BR-001 rather than
  invented as fact — the lead should confirm or revise it before the first real pilot, as BR-001's
  own text already says.
- Whether BR-005's self-registration MVP default (vs. reusing the existing barangay SMS network's
  contact list, per `idea.md` EV-008) is the right call given real barangay data-sharing
  willingness — `prd.md` itself frames this as open, deferring to `decision-ledger.md`.
- Whether the assumed SMS aggregator and "managed PaaS" hosting choice meet Philippine carrier
  deliverability/cost and an LGU's real procurement constraints — external vendor facts, unchanged
  from cycle 1.
- Whether staff will keep F-003 status updates current with near-zero effort (`A-002`) — unchanged,
  untested per `prd.md` Dependencies; only a pilot answers it.
- Whether A-001's comparative claim will actually be measured once built — the missing
  comparison/baseline/activation-logging mechanism (§2) is still absent from the design; whether to
  add it before a pilot remains the lead's call, not something either validation cycle can resolve
  by re-reading harder.
- TC-N01's lexical-brittleness (§3, cycle 1 §4) and the "managed PaaS hosting" `[assumption]`-tagging
  inconsistency (§1) both remain open, exactly as cycle 1 left them — neither was the REVISE trigger
  then, neither is touched by this revision now, and neither blocks this cycle's PASS.

## 6. Decision

- **`PASS`** — proceed to phase-5.3. Both gaps named in cycle 1's `REVISE(prd)` are closed:
  1. **BR-001's freshness window** now states a concrete value ("4 hours," honestly `[assumption]`-tagged)
     and the F-001 acceptance criteria were updated to match — INV-002's row in §3 now backs a real,
     stated number instead of a circular reference. `system-design.md`'s TTL mechanism and
     `qa-test-plan.md`'s TC-002/TC-N02 remain coherent with this change without needing edits of
     their own (§3, §4).
  2. **F-002's data dependency** is now named in `prd.md` Dependencies as an explicit unresolved MVP
     blocker with two candidate sources, and new **BR-005** adopts one (self-registration) as the
     MVP default while leaving the other as the final-scope path — this meets and exceeds the bar
     cycle 1 set for an acceptable resolution (§0(b)).
- **Not `REVISE`:** no new contradiction was introduced by the revision — `system-design.md` and
  `qa-test-plan.md`, both unrevised, remain coherent with the changed `prd.md`; the trace-ids run is
  clean (APPROVE, no orphans/dangling, BR 4→5 accounted for by BR-005 alone).
- **Not `ESCALATE`:** there is no unresolved defect left to escalate — this is not a case of reaching
  cycle 2's limit with a gap still open; both named gaps closed on their merits.
- Carried forward, unchanged, and explicitly **not** re-litigated or re-scored as blockers this
  cycle (per cycle 1's own framing that these are not REVISE triggers): the A-001 instrumentation gap
  (§2), TC-N01's lexical brittleness (§3), and the "managed PaaS hosting" `[assumption]`-tagging
  inconsistency (§1) — all remain open items for the lead in §5, not defects in this verdict.
