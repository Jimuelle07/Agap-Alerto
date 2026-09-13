---
# Vault context block — schema 2.1.0
team_size: 1               # [assumption] no team/staffing info was given this run; revise once a real build team is assigned
mode: solo
build_type: production     # treated as a real civic-tech deployment target, not a one-off demo — matches the user's "large scale/full project" framing; revise to `hackathon`/`graded` if this is actually a competition or coursework submission
time_budget: 2w            # [assumption] order-of-magnitude for a first real MVP build, not a hackathon sprint
judged: false               # no competition/rubric context was supplied
computes_numbers: false    # stock status is categorical (open/stocked/closed), not a computed score
exposed_surface: true      # resident-facing lookup/alert + staff-facing status update are both network-exposed
exposes_api: false         # MVP has no external contract yet; F-102 (barangay SMS system integration) is final-scope, not MVP
has_ui: true                # F-001 lookup + F-003 staff status update both need a (lightweight) interface
outlives_demo: true         # must keep running through the current epidemic and future flood seasons (F-103) — this is the point of the project
build_crew: true            # "full project" — generate the phased plan, crew, harness, handoff packets
tests: deferred
release_planning: true      # real deployment/adoption to Manila LGU health infrastructure is the goal, not just a pitch
handoff_expected: true       # a Manila LGU health-IT team or civic-tech successor would need to take this over
pivots_expected: true        # A-001 and the Significant test are explicitly unvalidated by any human conversation (see evidence-ledger.md Decisions) — first real interviews are likely to revise this brief
rigor: standard
selection_mode: auto
speed: standard              # full path was chosen; not the sprint lane

# competition: N/A — judged: false, no rubric/theme supplied this run.
---

# Context intake

## Notes on this run
This context block was filled by the Key's author role without a live human present to confirm
every field (autonomous run from a public news report + open-web research). Fields marked
`[assumption]` above are the orchestrator's best judgment, not a stakeholder's confirmed choice —
flag them for a human to revisit before `build_crew` actually starts generating the phased plan.
The one field NOT an assumption: `pivots_expected: true` and `release_planning: true` follow directly
from the evidence ledger's own Decisions section (zero primary interviews conducted) and the "large
scale/full project" framing of the request, respectively.
