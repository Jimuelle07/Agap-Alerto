# hooks/ — IDE-agnostic commit-time sensors

These hooks are the mechanical backstop for the parts of `AGENTS.md` that a shell script can
check, regardless of which IDE or agent you use.

## What `pre-commit` does

| # | Check | Effect |
|---|---|---|
| 1 | `docs/adr/` changed without `docs/decision-ledger.md` | **blocks** (ADR ⇒ ledger line, same commit) |
| 2 | `docs/implementation-plan.md` staged → `box/vault/tools/check-implementation-plan.py` | **blocks** on REJECT |
| 3 | `docs/handoff/TASK-###.md` staged → `box/vault/tools/check-handoff.py --plan --qa` | **blocks** on REJECT |
| 4 | tests and source edited in the same commit | reminder (R3: an executor never edits a test) |
| 5 | a hard-rule surface (data / copy / rules) touched | reminder to add an `INV` audit line |

Checks 2–3 need `python3` (or `python`) on PATH and the Vault at `./box/vault` (or `$VAULT_DIR`);
otherwise they print a warning and skip. Nothing here freezes the spec. Genuine exception:
`git commit --no-verify`.

## Install (pick one)

```sh
# per clone
cp hooks/pre-commit .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit
# or shared via the repo (one-time per clone)
git config core.hooksPath hooks && chmod +x hooks/pre-commit
```

## What is deliberately NOT here
- Status transitions, branch/PR policy, phase gates — the keeper + the plan own those.
- Semantic conflict resolution and pre-milestone reconciliation — judgment calls in `AGENTS.md`.
- Running the product's test suite — that is the fast/full gate the executor and keeper run.
