# Usability gate — Agap Alerto

<!-- EMITTED as seed/usability.md. Written by concept_validator (GPT-5.6 Terra) in the phase-4
     dispatch, from the finished idea.md §7 + §9 A-001 only. Text-only semantic usability test —
     see playbooks/usability-gate.md. The brief is never edited here. The Vault reads the approved
     flow as a phase-5.1 design input and NEVER gates on it. -->

**Verdict:** `USABILITY CLEARED`
**Cycle:** 2 of 2 · **Date:** 2026-09-13 · **has_ui:** true
**A-001:** Informal workers/vendors in flood-exposed Manila barangays will act on a same-day,
site-specific alert within their exposure window, at a meaningfully higher rate than they act on
the city's existing general information campaign alone. (idea.md §9)

**Independence (ADR-0007):** This re-walk is run by `concept_validator` as Fable 5.1
(`claude-fable-5-1`, Anthropic, generation 5.1). idea.md's revision (the F-001 change being
re-gated) was authored by Sonnet 5 (`claude-sonnet-5`, Anthropic, generation 5). Same model family,
different generation → **Independence: PARTIAL**. This is also the same *role* (`concept_validator`)
that produced the cycle-1 finding — the re-walk is that role checking its own prior finding against
the author's fix, which is what a PIVOT/re-gate cycle is for, not a fresh independent read.

## 1. UX flow (text only)

There are two user types on the resident side (walked below) plus a separate staff side (F-003,
noted for feature coverage only, not walked). Only the "Nearest site result" screen's `Shows:`
content has changed from cycle 1, per the author's single named revision to idea.md §7 F-001.

```
Screen: Alert received (SMS/push) [F-002]
Shows:   "Leptospirosis alert: You are in a flood-declared area during the active exposure window.
         Free doxycycline is available. Reply SITE for the nearest open, stocked site now."
Actions: [Reply SITE]

Screen: Nearest site result (SMS/web-lite) [F-001]
Shows:   "Nearest open site: [Barangay Health Center name]. Status: Open, stocked (confirmed 14 min
         ago). Hours: open until 5:00pm. Address: [address], ~10 min walk. Ask staff for free
         doxycycline prophylaxis (supervised)."
Actions: [Reply MAP for directions] [Reply DONE]
```

Alternate entry — a resident who did **not** receive a push alert (first-open, nothing configured,
per playbook Step 1's empty-state rule) reaches the same result screen via F-001's standalone
lookup:

```
Screen: Start — check a site (no alert received) [F-001]
Shows:   Empty state: "Enter your barangay or location to find the nearest open, stocked
         free-doxycycline site."
Actions: [Text barangay/location to shortcode] / [Enter barangay or location + Submit (web-lite)]
```
— this leads to the same, now-revised "Nearest site result" screen above.

**Feature coverage:** F-002 → Alert received · F-001 → Nearest site result (revised), Start — check
a site · F-003 → Staff status update (below, not walked as part of the resident flow):

```
Screen: Staff status update [F-003]
Shows:   This site's current status: In stock / Low / Out, and Open / Closed.
Actions: [Mark: In stock] [Mark: Low] [Mark: Out] [Mark: Open] [Mark: Closed]
```

## 2. Impatient-user walkthrough

Persona: impatient, non-technical, will not read help, abandons on the second confusion. Goal: the
`A-001` outcome — receive same-day, site-specific, actionable information and be positioned to act
on it within the exposure window. Walked from the primary A-001 trigger, F-002's push alert (the
alternate F-001 standalone entry above is one click longer by the same logic and is not
separately re-walked — unchanged from cycle 1).

| Click | Screen | Action taken | Why it is the obvious choice | Leads to |
|---:|---|---|---|---|
| 1 | Alert received | `[Reply SITE]` | It is the only action offered, and the alert text names it directly — no other path exists on this screen | Nearest site result |

**Outcome reached at click 1:** a specific, named, currently-open, currently-stocked site, now with
a stated last-confirmed freshness timestamp, plus hours and distance, is on screen — the
site-specific, same-day, actionable content A-001 requires.

## 3. Kill criteria

Re-scored from scratch, per the playbook. Only the result screen's shown content changed between
cycles; nothing about the number or structure of screens, or the actions offered, changed.

| # | Criterion | Tripped? | Evidence (step) |
|---|---|---|---|
| K1 | User must guess the next step | no | Unchanged from cycle 1: both screens in the walked path still offer exactly one obvious action each (`[Reply SITE]`; the alert names the keyword directly). The F-001 revision only added shown content to the result screen — it did not add, remove, or ambiguate any `Actions:` entry. |
| K2 | Goal is more than 3 clicks from the start screen | no | Unchanged from cycle 1: click count = 1. Adding a timestamp to the result screen's `Shows:` text is a content change, not a new screen or click. |
| K3 | User missing context a decision needs | **no** | idea.md §7 F-001 now specifies the result includes "hours **and a last-confirmed freshness timestamp** (e.g., 'confirmed 14 min ago')." This is exactly the piece of context cycle 1 found missing: at click 1, the resident deciding whether the ~10-minute walk is worth it can now see both the claimed status ("Open, stocked") and how recently that claim was confirmed, with no added click or screen. idea.md §9 INV-002 requires the freshness window to be "stated" to the person deciding — a visible last-confirmed timestamp on the one screen the impatient persona actually reaches satisfies that as specified. |

**On the raw-number question (considered, not applied as a new requirement):** one could ask
whether "confirmed 14 min ago" is legible to an impatient, non-technical user without an explicit
stated threshold for what counts as "too old" (e.g., "fresh" vs. "stale" labeling, or a cutoff
after which the system stops showing "open, stocked" at all). idea.md §9 INV-002 requires a
**stated** freshness window — it does not require the UI to additionally label or threshold that
number for the user, and playbooks/usability-gate.md's scoring rules explicitly instruct: "Do not
invent polish requirements (copy tone, empty states, error design). The gate is reachability of the
core outcome, nothing more." A bare elapsed-time stamp ("14 min ago") is common, ordinarily-legible
information (the same convention as a chat-message timestamp) and is the specific missing fact
cycle 1 named — not a new, un-evidenced usability need. Whether the system additionally enforces a
hard cutoff behavior once data ages past some threshold is a backend/data-freshness question for
F-001's build, not a screen-reachability question this text-only walkthrough can test; it is out of
scope for this gate per the rule above and is not raised as a K3 finding.

## 4. Decision

- **`USABILITY CLEARED`** — no criterion tripped. The flow in §1 (with the revised "Nearest site
  result" screen) is approved as-is; it is the Vault's screen-flow input for the PRD user journeys
  and design-system. The single named revision to idea.md §7 F-001 (adding a last-confirmed
  freshness timestamp to the result, with no new click or screen) resolves the cycle-1 K3 finding
  as specified, without requiring any further change to the brief.

  This is cycle 2 of 2 under the qaLoop rule (playbooks/usability-gate.md). Since the flow clears
  on this cycle, the loop ends here with a clean pass — there is no open item to hand to the human
  and no third revision to propose.

## 5. Disagreement record

None to record. Cycle 1 and cycle 2 were both run by the same role (`concept_validator`); cycle 2
is that role re-checking its own cycle-1 finding against the author's fix, not an independent
second opinion, so there is no second position to disagree with. Cycle 2 agrees with cycle 1 that
the K3 gap was real and needed a fix, and finds that the author's single named revision to F-001
resolves it as specified. (Independence for this cycle is PARTIAL per ADR-0007 — see header above
— which is a limit on how much weight this clearance should carry, not a disagreement.)
