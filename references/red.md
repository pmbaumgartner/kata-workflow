# Red Kata Issues

Red issues are implementation work. Their output is shipped, verified change.

## Issue Body

Use label `red`. Start the title with one of the Red verbs listed in SKILL.md; do not add a `[Red]` prefix.

Examples: `Fix Safari callback duplication`, `Add ANN recall gates`, `Migrate legacy issue references`.

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

Red work still permits local implementation judgment. Do not open Blue merely to choose among equivalent local techniques that leave scope, acceptance criteria, settled decisions, and reusable policy unchanged.

## Workflow

1. Read the issue body and linked decision docs.
2. Check `Pause Triggers` before broadening scope.
3. Implement the smallest change that satisfies acceptance criteria.
4. Run focused checks while developing.
5. Run the issue's required verification before close.
6. Commit the completed work before close, unless the user explicitly declares that no commit is required.
7. If uncertainty would change scope, acceptance criteria, a settled decision, or a reusable policy, stop and open or update a Blue issue.
8. Before close, consider whether execution produced reusable learning, exposed a faulty assumption, or suggests changing future decisions or process. If so, record it in a linked Blue issue; otherwise close without retrospective ceremony.

The learning review does not block Red closure unless it invalidates acceptance or verification. A follow-up Blue issue is sufficient when the Red change remains complete.

## Performance Work

For any Red issue that claims a performance improvement, optimization, fewer syncs, fewer allocations, faster runtime, lower memory, or benchmark evidence:

- Run or create the baseline benchmark against current committed behavior before changing code.
- Record the baseline command, dataset or shape, seed, metric, and result.
- After implementation, run the same benchmark against the changed code.
- Do not call the change an improvement unless same-harness before/after results show it.
- If the benchmark is slower or inconclusive, record the result. In Human mode, ask the user whether to revert before closing or handing back. In Agent mode, choose the smallest reversible outcome that best serves the goal, record the choice, and continue without waiting.

## Close-Out

Red closure must include:

- brief `--message` naming what changed
- committed work, unless the user explicitly declared otherwise
- `--commit <sha>` for the commit containing the completed work, unless the user explicitly declared otherwise
- `--test "<cmd>"` for each verification command that passed
- skipped or unavailable checks documented in reviewed docs/artifacts when they
  affect future work
- `--reviewed <path>` for every deliverable path
- residual risk or deferred follow-up documented in reviewed docs/artifacts

If the issue has no doc/artifact deliverable, record skipped checks, residual
risk, or deferred follow-up in a short kata comment before closing.

If the user declares that no commit is required, record that exception in a
short kata comment or the close message. Do not treat a PR URL as a substitute
for `--commit` unless the user explicitly accepts that exception.

Example:

```bash
kata close abc4 --done \
  --message "Added ANN recall gates." \
  --commit <sha> \
  --test "cargo test" \
  --test "uv run pytest" \
  --reviewed tests/test_ann_recall.py \
  --reviewed examples/ann_benchmark.rs \
  --agent
```

Do not close if verification did not run and no accepted rationale is recorded.
