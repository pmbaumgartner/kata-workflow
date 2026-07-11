---
name: kata-workflow
description: Use when an agent needs to inspect, claim, assign, create, update, triage, coordinate, choose next work from, close kata issues, or translate markdown plans into linked kata issue sets; follow Blue decision workflows and Red implementation workflows, preserve evidence, avoid false-closing, and use kata as the shared issue ledger across projects.
---

# Kata Workflow

Kata is the shared issue ledger. Use it as durable external memory for task scope, decisions, evidence, and close-out.

## First Moves

- Run `kata quickstart` when entering an unfamiliar kata workspace or after a kata upgrade.
- Use `kata ready --unowned --agent` when asked what new work to take next, then `kata claim <ref> --agent` after the user chooses it. Use an owner filter instead when resuming assigned work.
- Use `kata list --status all --agent`, `kata show <ref> --agent`, and `kata search "<term>" --agent` to orient.
- Do not `delete` or `purge` unless the user explicitly asks for that exact destructive action and ref.
- Keep local scratch state such as `.scratch/` untracked and out of code indexes unless the project explicitly says otherwise.

## Hard Rules

- Do not false-close. Closing asserts the work is verified.
- If work is incomplete, leave it open, add `needs-review` if useful, and comment with what was attempted and what remains.
- Close verified Red work promptly.
- Close Blue work only after explicit signoff from the named decision owner. Default to the current user when the issue or project names no decision owner.
- Use `blocks` / `blocked_by` for real sequencing. Use `related` only for context.
- Give every issue exactly one canonical workflow label: `blue` or `red`. Titles make the type visible but do not replace the label.
- Search before creating. Always pass `--idempotency-key <slug>-<YYYY-MM-DD>` on `kata create`.
- Treat a failed `kata claim` as a coordination signal. Do not use `--force` unless the user explicitly asks to take ownership from the current owner.
- Every artifact-producing issue needs a `## Deliverables` block.
- Every deliverable path must be reviewed before close.
- Keep close `--message` brief but substantive. Put durable detail in reviewed docs/artifacts, typed evidence, or a short kata comment when no artifact exists.

## Refs And Invocation

- Refs are short IDs derived from ULIDs, such as `abc4`. Cross-project refs look like `kata#abc4`. Full ULIDs also resolve. Legacy numeric refs do not.
- This workflow expects kata v0.9.0 or newer. Check `kata version` if `--agent`, `claim`, or the documented filters are missing; report a stale CLI instead of silently substituting a different workflow.
- Commands run against the current workspace unless `--workspace` or `--project` overrides it.
- Author resolves as `$KATA_AUTHOR` > `$USER` > `git user.name`.
- Use `--agent` for concise agent-readable output. Use `--json` only when a script or `jq` projection needs the full structured shape.
- Use `kata whoami --agent` when actor identity matters. If kata is uninitialized, report that `kata init` is needed.

Common commands:

```bash
kata search "login race" --agent
kata create "Fix login race" --body "Observed double-submit in Safari callback." \
  --label red --idempotency-key "login-race-2026-05-02" --agent
kata show abc4 --agent
kata comment abc4 --body "Found another reproduction path." --agent
kata label add abc4 safari --agent
kata edit abc4 --blocks d4ex --agent
```

If lexical search misses a likely match and `[search.embeddings]` is configured, retry with `kata search "<term>" --hybrid --agent`. Without embeddings, hybrid and semantic search fail validation; keep lexical search and vary the query instead.

## Relationships

Relationship flags are framed from the operating issue's point of view:

- `parent` - this issue is a sub-task of a larger issue
- `blocks` - this issue must resolve before the target can proceed
- `blocked_by` - the target must resolve before this issue can proceed
- `related` - useful context, not ordering

Agents use `kata ready --agent`, so weak ordering links make the ready queue less trustworthy.

If Red work depends on a Blue decision, make the Red issue `--blocked-by` the Blue issue. Labels and title verbs do not affect readiness; a missing relationship makes dependent Red work appear ready too early.

`--remove-parent <ref>` is strict: the ref must equal the current parent. Other `--remove-*` flags are idempotent.

## Closing Evidence

Use typed sugar where possible:

- `commit:<sha>` / `--commit <sha>`
- `pr:<url>` / `--pr <url>`
- `test:<cmd>` / `--test "<cmd>"`
- `reviewed-paths:<path>` / `--reviewed <path>`
- `no-change-audit:<text>` with `--audit-no-change`
- `duplicate-of:<ref>` via `--duplicate-of <ref>`
- `superseded-by:<ref>` via `--superseded-by <ref>`

Do not invent evidence prefixes such as `sanity-check:` or `smoke-test:`. Put manual checks, skipped checks, benchmarks, residual risk, and rationale in the reviewed artifact or a short kata comment.

Every path under `## Deliverables` becomes a `--reviewed <path>` flag. If a deliverable does not get reviewed, either the work is not done or the issue body should be edited before close.

Verify that commit evidence contains the completed work before closing. Do not use the current `HEAD` merely because it is convenient.

Example:

```bash
kata close abc4 --done --message "Fixed Safari callback double-submit." \
  --commit <sha-containing-completed-work> --test "cargo test" --dry-run --agent

kata close abc4 --done \
  --message "Fixed Safari callback double-submit." \
  --commit <sha-containing-completed-work> \
  --test "cargo test" \
  --test "uv run pytest" \
  --reviewed docs/corpus_profile.md \
  --agent
```

Use `--dry-run` before a complicated or high-impact close. It validates without mutating; rerun without `--dry-run` after the preview is correct.

Other close forms:

```bash
kata close abc4 --duplicate-of d4ex --message "Same Safari race condition." --agent
kata close abc4 --superseded-by d4ex --message "Replaced by broader scope." --agent
kata close abc4 --wontfix --message "Out of scope after contract review." --agent
kata close abc4 --audit-no-change --message "Reviewed schema; no change needed." \
  --evidence "no-change-audit:schema unchanged after review" \
  --reviewed internal/db/schema.sql --agent
```

Parent close is refused while open children remain.

## Coordination And Mutations

Use ownership to prevent duplicate work:

```bash
kata ready --unowned --agent
kata claim abc4 --comment "Starting the agreed implementation." --agent
kata unassign abc4 --comment "Releasing; blocked on a missing fixture." --agent
```

`edit`, `close`, `reopen`, `claim`, `assign`, `unassign`, and `label add/rm` accept `--comment TEXT`. Use it when wiring relationships, changing ownership, or flipping labels so the reason lands with the mutation.

```bash
kata edit abc4 --blocked-by skwh \
  --comment "Need skwh's decision artifact before we can scope this." --agent
```

The mutation lands first. If the follow-up comment fails, retry with `kata comment <ref> --body ...`.

Use `kata label add <ref> needs-review`; `kata edit --label` is not a valid command even if older help text suggests it.

For a long-running or multi-agent session, poll `kata events --after <cursor> --agent` and resume from the returned cursor. Use `kata digest --since 24h --agent` for a human-scale handoff summary. If an issue belongs to the wrong project, preview `kata move <ref> <project> --dry-run --agent`; moving preserves history and links but assigns a new short ref in the target project.

## Blue Or Red

Every issue is either:

- Blue: thinking work. Outputs decisions, findings, recommendations.
- Red: doing work. Outputs code, models, features, pipelines.

Blue variability is useful; Red variability is a cost to minimize.

Use the label as the canonical type and start the title with a type-specific verb so the type remains visible in the TUI and `ready --agent` output:

| Type | Opening verbs |
| --- | --- |
| Blue | `Decide`, `Investigate`, `Evaluate`, `Compare`, `Determine`, `Define`, `Audit` |
| Red | `Add`, `Fix`, `Implement`, `Migrate`, `Remove`, `Refactor`, `Optimize`, `Upgrade`, `Document`, `Configure` |

Do not add `[Blue]` or `[Red]` prefixes. Avoid vague openings such as `Improve`, `Handle`, `Support`, `Review`, `Design`, or `Work on`; rewrite the title to expose whether the issue settles knowledge or ships a change. The verb is a human cue, not a substitute for checking the label.

No purple work: split mixed work into separate Blue and Red issues.

For Blue work, read [references/blue.md](references/blue.md).
For Red work, read [references/red.md](references/red.md).

When Red work hits unexpected uncertainty, stop and open or update a Blue issue rather than pushing through.

## Plan Markdown To Issues

When translating a large markdown plan, or several smaller plan documents, into kata issues, read [references/plan-to-issues.md](references/plan-to-issues.md).

Use the robust flow: coordinator draft, pre-mortem plus modularity review, unlinked creation, central linking, drift reconciliation, fidelity review. Create issues in parallel when independent agents are available and the coordination overhead is justified; otherwise use the same flow sequentially.

## What's Next

When the user asks what to work on next:

1. Run `kata ready --unowned --agent`. It returns unclaimed open issues with no open `blocks` predecessor.
2. Inspect unfamiliar structure with `kata show <root-ref> --agent`.
3. Prefer upstream feeders and work without a stakeholder loop.
4. Treat both Blue and Red results as eligible. If a Red issue still needs a Blue decision, fix the missing relationship instead of relying on type preference.
5. Recommend one issue and the tradeoff in 1-3 sentences. Do not claim or start it until the user agrees.


## Where Things Live

| Kind | Goes in |
| --- | --- |
| Scope, acceptance, deliverable paths | issue body |
| Findings, schemas, decision rationale | repo doc |
| Status, blockers, attempts | issue comment |
| Mutation rationale | mutation `--comment` |

If a future reader needs it on first read, put it in the body. If it happened along the way, put it in a comment. If it is a reusable finding, put it in a doc.

If a comment grows beyond a paragraph, move it to a doc and comment with the path.

Each doc should be the single source of truth for its topic. Edit in place; do not spawn `<topic>_v2.md`. New scope gets a new doc and its own kata ref.
Last-updated is recoverable from `git log`; the trail of what changed lives in kata comments.

Repo docs used as kata artifacts should include front matter:

```md
---
kata: <ref>
created: YYYY-MM-DD
---
```

## Drafts

Use `.kata/drafts/` for multi-line bodies and comments. Do not use `/tmp/`. Ensure `.kata/` is ignored by git.

| Draft kind | Name |
| --- | --- |
| New issue | `<slug>.md` |
| Body edit | `<ref>-<i>.md`, monotonic per issue |
| Comment | `<ref>-<topic>.md` |

```bash
kata create "<title>" --body-file .kata/drafts/<slug>.md --idempotency-key <slug>-<date> --agent
kata comment <ref> --body-file .kata/drafts/<ref>-<topic>.md --agent
kata edit <ref> --body "$(cat .kata/drafts/<ref>-<i>.md)" --agent
```

Inline `--body "..."` is fine for one-line edits.

## JSON Gotchas

Do not pipe full JSON to `head`; it can truncate mid-field. Project the fields you need.

In `kata show --json`, relationships live at the top level, not under `.issue`. `.issue` carries only scalar fields. Project from the real paths:

```bash
kata show <ref> --json | jq '{
  title:.issue.title, status:.issue.status, body:.issue.body,
  parent:.parent.short_id,
  links:[.links[]? | {type, from:.from.short_id, to:.to.short_id}]
}'
```

Use the right wrapper key:

```bash
kata ready  --json | jq -r '.issues[]?  | "\(.short_id) \(.title)"'
kata list   --json | jq -r '.issues[]?  | "\(.short_id) \(.title)"'
kata search "x" --json | jq -r '.results[]? | "\(.issue.short_id) \(.issue.title) (score=\(.score))"'
```

`kata search` returns `{issue, score, matched_in}`. If `?` hides expected hits, check the wrapper key before assuming the result set is empty.
