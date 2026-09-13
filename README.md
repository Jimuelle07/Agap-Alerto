# Agap Alerto

**The 99-second pitch** (read this out loud — it clocks in right around 99 seconds):

---

> Last week, Manila's leptospirosis attack rate jumped from 1.27% to 3.48% in seven days. Ospital ng
> Maynila, Sta. Ana Hospital, and Justice Jose Abad Santos General Hospital are already near capacity.
> Nationally, the case fatality rate is 6.05% — and DOH itself says a second wave is likely within
> two weeks, because that's the incubation period.
>
> Here's the thing: the city already did the hard part. Mayor Isko Moreno ordered free doxycycline —
> the prophylactic that prevents this disease — distributed across 44 health centers and 7 district
> hospitals. The medicine exists. It's free. It's already sitting on shelves across the city.
>
> The problem isn't the medicine. It's that nobody who just waded through floodwater knows *which*
> of those 51 sites is open, right now, with stock, close enough to reach before their window
> closes. The mayor's info campaign tells everyone, eventually, in general. It doesn't tell any one
> person, today, "go here, it's ten minutes away, it's open, it's stocked." So people wait, or guess,
> or self-medicate without supervision — which is exactly what DOH is separately warning against —
> and by the time symptoms show up, they're the ones filling the hospitals that are already full.
>
> Agap Alerto closes that one gap. Text your barangay, get back the nearest open, stocked site —
> with a timestamp so you know it's not stale. If a barangay just flooded, residents there get that
> alert automatically, same day, within the window that actually matters. No app required — it works
> over plain SMS, because that's what this segment actually has.
>
> We didn't skip the hard questions. The riskiest assumption — that people actually act on a
> targeted alert more than they act on a general campaign — is unproven, and we say so, in writing.
> An independent reviewer put it through a usability walk and a design audit before we wrote a line
> of code, and caught real gaps, which we fixed. The plan, the tests, and the first build task are
> ready to go.
>
> We're not guessing. We're not done either. That's exactly why we need you.

---

## What this actually is

A same-day SMS/web-lite lookup + alert that tells flood-exposed Manila residents which of the
city's 44 health centers + 7 hospitals has free doxycycline in stock, open, near them — right now.
Built in response to the September 2026 Manila leptospirosis epidemic
([source](https://www.pna.gov.ph/articles/1283462)).

## Status
- **Concept:** validated on secondary evidence, `GO-UNVALIDATED` — real, urgent, and reachable; the
  one thing nobody's tested yet is whether people act on the alert. That's what the pilot is for.
- **Usability:** cleared — one tap, one click to the answer.
- **Design docs:** PRD → system design → data model → QA plan → security review, all done,
  independently reviewed, `PASS`.
- **Build:** phased plan ready, first task fully specced. No code written yet.

## Where to look
| Want to know... | Read... |
|---|---|
| **Everything — which doc owns which fact, start here** | [`docs/index.md`](docs/index.md) |
| The problem, evidence, and the one risky bet we're making | [`seed/idea.md`](seed/idea.md) |
| Whether an independent reviewer thinks this is worth building | [`seed/validation.md`](seed/validation.md) |
| What we're building and why, in detail | [`docs/prd.md`](docs/prd.md) |
| How the build is sequenced | [`docs/implementation-plan.md`](docs/implementation-plan.md) |
| How to work on this | [`AGENTS.md`](AGENTS.md) |
