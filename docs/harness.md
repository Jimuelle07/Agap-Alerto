# Harness — guides and sensors for Agap Alerto

> Traces back to: `implementation-plan.md`, `qa-test-plan.md`.

## Principle
Spend CPU before GPU: deterministic checks catch what regex can see; a model judges only what it
cannot — INV-001's *semantic* dosing-implication risk (TC-N01's known brittleness, see
`validation-report.md` §3/§4) is the clearest case in this build where that boundary actually
matters.

## Guides (feedforward — shape the first attempt)

| Guide | Where | Loaded by | Purpose |
|---|---|---|---|
| Always-on rules | `AGENTS.md` (≤200 lines) | every tool, every turn | conventions, protocol, do-not-touch |
| Scoped rules | `.cursor/rules/*.mdc` | when a matching file is in context | path-specific depth |
| Handoff packet | `docs/handoff/TASK-###.md` | executor | the bounded brief (REASONS) |
| Source-of-truth map | `docs/index.md §0` | everyone | which doc owns which fact |
| Crew definitions | `.claude/agents/*.md` | orchestrating tool | role, tools, model, never-list |

## Sensors (feedback — catch and report), computational first

| # | Sensor | Command | Catches | Output style | Who runs it |
|---|---|---|---|---|---|
| S1 | Plan integrity | `python3 box/vault/tools/check-implementation-plan.py docs/implementation-plan.md` | phase/DAG/status/evidence errors; forward-phase deps; missing red/green | `REJECT` + line + fix hint | keeper, every checkpoint |
| S2 | Handoff integrity | `python3 box/vault/tools/check-handoff.py docs/handoff/TASK-###.md --plan docs/implementation-plan.md --qa docs/qa-test-plan.md` | missing REASONS section; TC not in QA plan; INV not in safeguards | `REJECT` + section | keeper, before dispatch |
| S3 | Task Verify (the fence) | `<Verify command from the task row>` | wrong behaviour / not-yet-built | test/import runner output | executor (red then green), keeper (verify) |
| S4 | Fast gate | `pytest -m fast` | regressions in the touched area | pytest output | executor, keeper |
| S5 | Full gate | `pytest` | phase-level regressions | pytest output | keeper, phase gate, default branch |
| S6 | Lint / type check | `<to be pinned once the language/tooling is set up at TASK-001>` | style/type errors with file:line | linter text | executor before returning |
| S7 | Ledger guard | `hooks/pre-commit` | ADR without ledger line; INV surface touched without audit | blocking message + fix | git, every commit |
| S8 | Seed preflight | `python3 box/vault/tools/check-seed.py seed/idea.md` | missing load-bearing sections | gap list | orchestrator, phase-5.0 and on pivot |
| S9 | ID spine | `python3 box/vault/tools/trace-ids.py docs --seed seed` | MVP feature with no test; invariant with no negative test; dangling refs | `REJECT`/`APPROVE` + owner doc named | orchestrator before phase-5.2; keeper on any ID/fact change |
| S10 | Run summary | `python3 box/vault/tools/summarize-run.py docs/run-evidence.jsonl` | a postmortem with vibes instead of numbers; an unclosed run | metrics, `unknown` where honest | orchestrator phase-5.6 |
| S11 | Consistency (inferential) | consistency-checker role | contradictions, term drift, semantic orphans | PASS/FAIL + list | orchestrator, after suites |
| S12 | Design validation (inferential) | validator role | design ↔ concept drift; INV breach in delivered work | verdict + contradiction | orchestrator phase-5.2; phase gates |
| S13 | INV-001 semantic probe (product-specific) | `pytest tests/negative/test_invariants.py::test_no_dosing_language` | lexical dosing terms only — **known gap:** paraphrased dosing language is not caught (`validation-report.md` §3/§4); a model-judgment spot-check of F-001 response copy is recommended before each phase gate until this sensor is strengthened | pytest output + a manual reviewer note | executor + keeper; **flagged for strengthening, not yet resolved** |

## Sensors added by depth-first debugging

| Date | Task that was stuck | Missing thing (packet / capability / sensor) | Sensor added | Result |
|---|---|---|---|---|
| — | none yet | — | — | this build has not started execution |

## Observability (optional, when the product runs somewhere)
- Once deployed: alert dispatch counts, lookup fresh/stale ratio, F-003 update frequency per site
  (`ops.md` Observability) — available to `design-validator` and `plan-keeper` for phase-gate
  re-reads once real data exists.

## Human-in-the-loop checkpoints
- Doc set confirmation (phase-5.0) — **not yet performed with an actual human in this run; flagged
  as an open item, see `implementation-plan.md` §1 `UNASSIGNED`.**
- Validation `ESCALATE` (phase-5.2) — did not occur this run (cycle 2 `PASS`).
- Cut decisions, any destructive shell command, anything that publishes, and — specific to this
  product — any change to INV-001/INV-002 enforcement logic or to the Data Privacy Act determination
  in `security-compliance.md`.
