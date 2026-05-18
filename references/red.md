# Red Kata Issues

Red issues are implementation work. Their output is shipped, verified change.

## Issue Body

Use title prefix `[Red]` and label `red`.

Required fields:

- `## Objective` - the known goal
- `## Context` - selected context needed to execute
- `## Scope In` / `## Scope Out` - boundaries for the work
- `## Acceptance Criteria` - observable completion criteria
- `## Pause Triggers` - conditions that should stop work and open Blue
- `## Deliverables` - every path that must be reviewed before close

Small Red issues may omit `Scope In/Out` only when acceptance criteria are
unambiguous.

Pause trigger examples: loss does not converge after repeated runs, data schema differs from Blue assumptions, or the implementation would require changing the settled product decision.

## Workflow

1. Read the issue body and linked decision docs.
2. Check `Pause Triggers` before broadening scope.
3. Implement the smallest change that satisfies acceptance criteria.
4. Run focused checks while developing.
5. Run the issue's required verification before close.
6. If unexpected uncertainty changes the decision, stop and open or update a Blue issue.

## Performance Work

For any Red issue that claims a performance improvement, optimization, fewer syncs, fewer allocations, faster runtime, lower memory, or benchmark evidence:

- Run or create the baseline benchmark against current committed behavior before changing code.
- Record the baseline command, dataset or shape, seed, metric, and result.
- After implementation, run the same benchmark against the changed code.
- Do not call the change an improvement unless same-harness before/after results show it.
- If the benchmark is slower or inconclusive, let the user know and ask them to decide whether to revert the attempted change and record the result before closing or handing back.

## Close-Out

Red closure must include:

- brief `--message` naming what changed
- `--test "<cmd>"` for each verification command that passed
- skipped or unavailable checks documented in reviewed docs/artifacts when they
  affect future work
- `--commit <sha>` or `--pr <url>` when available
- `--reviewed <path>` for every deliverable path
- residual risk or deferred follow-up documented in reviewed docs/artifacts

If the issue has no doc/artifact deliverable, record skipped checks, residual
risk, or deferred follow-up in a short kata comment before closing.

Example:

```bash
kata close abc4 --done \
  --message "Added ANN recall gates." \
  --commit <sha> \
  --test "cargo test" \
  --test "uv run pytest" \
  --reviewed tests/test_ann_recall.py \
  --reviewed examples/ann_benchmark.rs
```

Do not close if verification did not run and no accepted rationale is recorded.
