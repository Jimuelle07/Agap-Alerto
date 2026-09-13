# Design System / UX Spec — Agap Alerto

> Traces back to: `prd.md` user flows, `seed/brand.md`, `seed/usability.md` (approved flow).

## Design principles
1. **Say the site, not the science.** Every surface names a place and a time, never a dose (INV-001).
2. **State the freshness, always.** Any status shown carries how recently it was confirmed (INV-002)
   — never hide the clock behind a tap.
3. **One obvious action per screen.** The `seed/usability.md` cleared flow (click count 1) is the
   contract; do not add a second competing action to the resident-facing screens.
4. **Degrade to SMS.** Every resident-facing capability must work over plain SMS; a web view is an
   enhancement, never a requirement (`seed/market.md` device-access assumption).
5. **Calm urgency.** Urgent enough to act on today, calm enough not to read as a scare alert
   (`seed/brand.md` tone).

## Component inventory
- **Alert message** (SMS) — F-002's outbound template.
- **Result message** (SMS/web-lite) — F-001's response: site name, status, hours, distance,
  freshness line, next-action reply keywords.
- **Empty-state lookup form** (web-lite) — F-001's standalone entry (UJ-002).
- **Staff status panel** (web page) — F-003's five status buttons + current-state display.

## Tokens
Per `seed/brand.md`: visual tokens (colour, type) are **undecided** at this stage — the brand seed
explicitly leaves them open. What is decided and binding now:
- **Contrast:** high-contrast required for outdoor/bright-light legibility (staff panel is often
  used at an outdoor barangay health center) — target WCAG AA minimum, not a specific palette yet.
- **Type scale:** large, high-legibility body text is a hard requirement (`seed/brand.md`); exact
  typeface undecided.
- **Accent use:** reserve any "act now" accent colour (candidate: amber/red per `seed/brand.md`) for
  the single primary action per screen — never decorative.

## Patterns & states
- **Result message states:** *fresh* (status shown + freshness line), *stale/unknown* (BR-001 —
  "status unknown for sites near you," no open/stocked claim), *no site found* (barangay not
  recognized — prompts a retry with an example format).
- **Staff panel states:** *idle* (shows current status + last-confirmed time), *updating*
  (optimistic one-tap, confirms within 5s per F-003's acceptance criterion), *stale-self-warning*
  (the panel itself flags to staff when their own last update is approaching the freshness window's
  expiry, so they know to refresh it — not a resident-facing state).

## UI voice & copy rules (enforced, not optional)

**Banned copy (enforced — tie each to its invariant):**
- Never use a dose amount, frequency, or duration (e.g., "200mg," "take twice daily," "once a
  week") anywhere in an F-001 response — breaches **INV-001**. Say "ask staff for the free,
  supervised dose" instead.
- Never state or imply a site is "open" / "in stock" / "stocked" without also stating the
  last-confirmed elapsed time on the same message — breaches **INV-002**.
- Never use alarmist language ("you WILL get sick," "act NOW or else") — undermines the calm-urgency
  tone `seed/brand.md` requires and risks the wasted-trip harm noted in `security-compliance.md` if
  it pressures a rushed decision.

**Tool/default overrides (this project intentionally deviates):**
- SMS replies are plain text only — no rich media, no links requiring a data connection, per the
  SMS-first accessibility constraint (`seed/brand.md`).

**Provenance:** tokens/components are net-new for this project; no prior source file to sync from.
Last synced: 2026-09-13 (this document's authoring date).

## Accessibility standards
- Target **WCAG AA** for the staff web-lite UI (the only browser surface, `qa-test-plan.md` Browser
  E2E scope).
- Filipino/Tagalog and English content, plain-language reading level, per `seed/brand.md`
  Accessibility / locale constraints.
- SMS content must fit within standard SMS length constraints without truncating the freshness
  timestamp or the no-dosing-language guarantee — a message that must be split across multiple SMS
  segments should repeat the critical fields (site name, status, freshness) in the first segment.

## Key screen specs / wireframes
Reuses `seed/usability.md`'s cleared flow verbatim as the canonical screen-flow input (per
`manifest.json`'s phase-4.5 → phase-5.1 handoff):
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

Screen: Start — check a site (no alert received) [F-001]
Shows:   Empty state: "Enter your barangay or location to find the nearest open, stocked
         free-doxycycline site."
Actions: [Text barangay/location to shortcode] / [Enter barangay or location + Submit (web-lite)]

Screen: Staff status update [F-003]
Shows:   This site's current status: In stock / Low / Out, and Open / Closed.
Actions: [Mark: In stock] [Mark: Low] [Mark: Out] [Mark: Open] [Mark: Closed]
```
