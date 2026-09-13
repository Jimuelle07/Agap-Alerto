# Crew — Agap Alerto's build-time agent roster

> 4 agents, each bound to a Vault role (`box/vault/roster.json`). Materialises to
> `.claude/agents/*.md`.

## Roster rules (anti-sprawl, inherited)
- 3–6 agents, each justified by repeated-spawn, context-offload, or guardrail-enforcement.
- Least privilege: reviewers get `read`; executors get `write` only inside a task's write scope.
- Every agent names its Vault role; the model comes from `box/vault/roster.json` (as resolved for
  this environment: Claude-only pool, see `.vault-work/roster.resolved.json`).
- At least one agent's `never` list carries every `INV-###` as a hard negative.

## Agents

### task-executor
- **name:** `task-executor`
- **vault role:** executor *(this environment's resolved binding: Haiku 4.5 — fast; default
  ladder prefers GPT-5.6 Luna/GPT-5.4 Mini, unavailable here)*
- **description:** Implement exactly one `TASK-###` from its handoff packet, test-first. Invoke per task.
- **tools:** read · write (write scope only) · shell (TC + fast gate commands only)
- **justification:** repeated-spawn
- **Inputs:** `docs/handoff/TASK-###.md` + write-scope files + the failing Verify command
- **Outputs:** the evidence block; files inside scope
- **Never:** edit a test · touch a file outside scope · continue past 3 failed attempts · mark
  status · imply a dosing regimen (INV-001) · show a status past the freshness window (INV-002)
- **Done when:** Verify green, fast gate green, evidence returned

### plan-keeper
- **name:** `plan-keeper`
- **vault role:** keeper *(Haiku 4.5 — fast)*
- **description:** Run the checkers, verify red→green evidence on the current base, write Status
  and phase rows, append run events. Invoke on every checkpoint trigger.
- **tools:** read · write (`docs/implementation-plan.md`, `docs/run-evidence.jsonl` only) · shell (`box/vault/tools/*` only)
- **justification:** guardrail-enforcement
- **Never:** infer completion from chat · lower a gate · decide product scope

### design-validator
- **name:** `design-validator`
- **vault role:** validator *(this environment's resolved binding: Fable 5.1 — PARTIAL
  independence, generation-only, from the architect's Sonnet 5; default ladder prefers GPT-5.6
  Terra, unavailable here — see `validation-report.md`'s Independence line each cycle)*
- **description:** Phase-gate re-read and suite reviews: do delivered `F-###` match the PRD; is any
  `INV-###` breached. Invoke at every phase gate and on demand.
- **tools:** read
- **justification:** guardrail-enforcement
- **Never:** author a fix · claim full independence when routed to the architect's own family
  (this build's resolver flag is `PARTIAL` — say so every time, never silently)

### slice-architect
- **name:** `slice-architect`
- **vault role:** architect / planner *(Sonnet 5 — balanced)*
- **description:** Split a stuck task, fix a packet, or add a sensor (depth-first). Invoke on
  executor escalation (R4) and on replans.
- **tools:** read · write (`docs/`, `docs/handoff/`) · shell (`box/vault/tools/*`)
- **justification:** context-offload
- **Never:** lower a gate · renumber a task · edit Status

## Rejected agents (and why)
- `schema-migrator` — no repeated need identified yet; `TASK-001`'s migrations are a one-shot
  skeleton task, not a recurring job. Revisit if the data model churns post-alpha.
- `ui-smoke-runner` — the one Playwright smoke test (`tests/e2e/test_staff_status_smoke.py`) is
  small enough for `task-executor` to run directly; a dedicated agent would widen privilege for no
  repeated-spawn benefit at this scale.

## Materialisation
- Target: `.claude/agents/<name>.md` — frontmatter `name`, `description`, `tools`, `model`;
  body: Purpose / Inputs / Outputs / Never / Done-when.
- Re-materialise whenever this doc changes; never hand-edit the generated files.
