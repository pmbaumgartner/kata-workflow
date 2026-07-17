---
name: kata-workflow
description: Use when an agent needs to inspect, claim, assign, create, update, triage, coordinate, choose the next kata issue, close kata issues, translate markdown plans into linked kata issue sets, or run autonomous reversible work with agent-owned decisions; follow Blue decision workflows and Red implementation workflows, preserve evidence, avoid false-closing, and use kata as the shared issue ledger across projects.
metadata:
  kata-developed-against: "0.10.0"
---

# Kata Workflow

Kata is the shared issue ledger. Use it as durable external memory for task scope, decisions, evidence, and close-out.

## First Moves

- Run `kata quickstart` when entering an unfamiliar kata workspace or after a kata upgrade.
- Treat the installed `kata quickstart` as normative for kata operating rules and `kata <command> --help` as normative for command syntax; the safety and Blue/Red rules here are additional. If an example differs from the installed CLI, follow the installed contract and report the drift before mutating.
- Use `kata next --unowned --agent` when asked to take one new issue; it selects the highest-priority ready issue but does not claim it. Use `kata ready --unowned --agent` to inspect or compare the queue. In Human mode, claim after the user chooses; in Agent mode, choose and claim without waiting. Use an owner filter when resuming assigned work.
- Use `kata list --status all --agent` for concise orientation; omit `--agent` when the rendered parent/child tree is useful. Use `kata show <ref> --agent` and `kata search "<term>" --agent` for detail.
- Do not `delete` or `purge` unless the user explicitly asks for that exact destructive action and ref.
- Keep local scratch state such as `.scratch/` untracked and out of code indexes unless the project explicitly says otherwise.

## Hard Rules

- Do not false-close. Closing asserts the work is verified.
- If work is incomplete, leave it open, add `needs-review` if useful, and comment with what was attempted and what remains.
- Close verified Red work promptly.
- Close Blue work only after explicit signoff from the mode's decision owner: the named human or current user in Human mode, and the primary agent in Agent mode.
- Use `blocks` / `blocked_by` for real sequencing. Use `related` only for context.
- Use priority to rank ready work, never as a substitute for a sequencing relationship. Priority `0` is highest.
- Give every issue exactly one canonical workflow label: `blue` or `red`. Titles make the type visible but do not replace the label.
- Search before creating. Always pass `--idempotency-key <slug>-<YYYY-MM-DD>` on `kata create`.
- Treat a failed `kata claim` as a coordination signal. Do not use `--force` unless the user explicitly asks to take ownership from the current owner.
- Every artifact-producing issue needs a `## Deliverables` block.
- Every deliverable path must be reviewed before close.
- Keep close `--message` brief but substantive. Put durable detail in reviewed docs/artifacts, typed evidence, or a short kata comment when no artifact exists.

## Decision Modes

Use exactly one decision mode for a goal or root issue:

- Human mode is the default. The current user owns frame checks, unresolved questions, and Blue signoff.
- Agent mode requires explicit user delegation for reversible, non-production work. The primary agent owns decisions, records signoff, closes Blue issues, and reports decisions at the end without waiting for human checkpoints.

Record Agent mode and its authority source in the root issue or a reviewed project policy. Child issues inherit that mode unless the user explicitly changes it. Activating a Goal alone does not select Agent mode or expand action permissions.

In Agent mode, resolve uncertainty with proportionate evidence and documented assumptions. Prefer the smallest, simplest, easiest-to-reverse choice when evidence is close. Defer nonessential branches to follow-up issues and continue other ready work. Only an environmental impossibility or unavailable required authority may stop the goal; ambiguity, competing designs, and low confidence do not.

For Agent mode, read [references/agent-mode.md](references/agent-mode.md).

## Refs And Invocation

- Refs are short IDs derived from ULIDs, such as `abc4`. Cross-project refs look like `kata#abc4`. Full ULIDs also resolve. Legacy numeric refs do not.
- The command examples were developed and checked against kata v0.10.0, recorded as `metadata.kata-developed-against`. Run `kata version`; if required commands or flags are absent, report a stale or drifted CLI instead of silently substituting a different workflow.
- Commands run against the current workspace unless `--workspace` or `--project` overrides it.
- Author resolves as `--as` > `$KATA_AUTHOR` > `$USER` > `git config user.name` > `anonymous`.
- Use `--agent` for concise agent-readable output. Use `--json` only when a script or `jq` projection needs the full structured shape.
- Use `kata whoami --agent` when actor identity matters. If kata is uninitialized, report that `kata init` is needed.
- Use `kata daemon status --agent` to diagnose the local daemon and `kata daemon restart --agent` when it needs a clean restart.

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

By default, `kata search` is lexical without embeddings and automatically hybrid when `[search.embeddings]` is configured. Use `--lexical`, `--hybrid`, or `--semantic` only to force a mode. Explicit hybrid and semantic search fail validation without embeddings; keep lexical search and vary the query instead.

## Relationships

Relationship flags are framed from the operating issue's point of view:

- `parent` - this issue is a sub-task of a larger issue
- `blocks` - this issue must resolve before the target can proceed
- `blocked_by` - the target must resolve before this issue can proceed
- `related` - useful context, not ordering

Agents use `kata ready --agent`, so weak ordering links make the ready queue less trustworthy.

If Red work depends on a Blue decision, make the Red issue `--blocked-by` the Blue issue. Labels and title verbs do not affect readiness; a missing relationship makes dependent Red work appear ready too early.

Use qualified refs for cross-project links. Federated kata v0.10 projects synchronize those links, so do not mirror dependency state with duplicate issues or comments in both projects.

`--remove-parent <ref>` is strict: the ref must equal the current parent. Other `--remove-*` flags are idempotent.

## Closing Evidence

Use typed sugar where possible:

- `commit:<sha>` / `--commit <sha>`
- `pr:<url>` / `--pr <url>`
- `test:<cmd>` via repeatable `--evidence "test:<cmd>"`; use `--test "<cmd>"` only when recording one verification command
- `reviewed-paths:<path>` / `--reviewed <path>`
- `no-change-audit:<text>` with `--audit-no-change`
- `duplicate-of:<ref>` via `--duplicate-of <ref>`
- `superseded-by:<ref>` via `--superseded-by <ref>`

Do not invent evidence prefixes such as `sanity-check:` or `smoke-test:`. Put manual checks, skipped checks, benchmarks, residual risk, and rationale in the reviewed artifact or a short kata comment.

Every path under `## Deliverables` becomes a `--reviewed <path>` flag. If a deliverable does not get reviewed, either the work is not done or the issue body should be edited before close.

For multiple verification commands, pass one `--evidence "test:<cmd>"` per command. In kata v0.10.0, repeated `--test` flags do not accumulate.

Verify that commit evidence contains the completed work before closing. Do not use the current `HEAD` merely because it is convenient.

Example:

```bash
kata close abc4 --done --message "Fixed Safari callback double-submit." \
  --commit <sha-containing-completed-work> --test "cargo test" --dry-run --agent

kata close abc4 --done \
  --message "Fixed Safari callback double-submit." \
  --commit <sha-containing-completed-work> \
  --evidence "test:cargo test" \
  --evidence "test:uv run pytest" \
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
kata next --unowned --agent
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

For scripted fan-out/fan-in, use `kata wait <refs...> --all --timeout <duration> --agent` instead of hand-written polling. Use `--any` to resume after the first issue completes and `--until attention` to resume when a worker records non-OK `work.attention`; always include units such as `30s` or `15m` in durations.

Keep narrative scope and status in bodies, comments, labels, and artifacts. Use `kata meta get/set/unset` only for machine-readable coordination state; preserve the `work.*` namespace for kata's work conventions.

For a long-running or multi-agent session, poll `kata events --after <cursor> --agent` and resume from the returned cursor, or use `kata events --tail --agent` for a live stream. Use `kata digest --since 24h --agent` for a human-scale handoff summary. If an issue belongs to the wrong project, preview `kata move <ref> <project> --dry-run --agent`; moving preserves history and links but assigns a new short ref in the target project.

## Blue Or Red

Every issue is either:

- Blue: decision and discovery work. Outputs settled knowledge: decisions, findings, recommendations.
- Red: implementation work. Outputs verified changes: code, models, features, pipelines.

Blue increases option and perspective variability. Red minimizes goal and scope variability while preserving local implementation judgment and anomaly reporting.

Use the label as the canonical type and start the title with a type-specific verb so the type remains visible in the TUI and `ready --agent` output:

| Type | Opening verbs |
| --- | --- |
| Blue | `Decide`, `Investigate`, `Evaluate`, `Compare`, `Determine`, `Define`, `Audit` |
| Red | `Add`, `Fix`, `Implement`, `Migrate`, `Remove`, `Refactor`, `Optimize`, `Upgrade`, `Document`, `Configure` |

Do not add `[Blue]` or `[Red]` prefixes. Avoid vague openings such as `Improve`, `Handle`, `Support`, `Review`, `Design`, or `Work on`; rewrite the title to expose whether the issue settles knowledge or ships a change. The verb is a human cue, not a substitute for checking the label.

No purple work: classify each issue by its completion claim. Split mixed work when it has distinct durable decision and implementation outputs; do not create a separate Blue issue for ordinary local implementation judgment inside Red work.

For Blue work, read [references/blue.md](references/blue.md).
For Red work, read [references/red.md](references/red.md).

When Red work exposes uncertainty that would change scope, acceptance criteria, a settled decision, or a reusable policy, stop and open or update a Blue issue rather than silently improvising. After Red work, open or update Blue only when execution produced reusable learning, exposed a faulty assumption, or suggests changing future decisions or process.

## Plan Markdown To Issues

When translating a large markdown plan, or several smaller plan documents, into kata issues, read [references/plan-to-issues.md](references/plan-to-issues.md).

Use the robust flow: coordinator draft, pre-mortem plus modularity review, unlinked creation, central linking, drift reconciliation, fidelity review. Create issues in parallel when independent agents are available and the coordination overhead is justified; otherwise use the same flow sequentially.

## What's Next

When the user asks what to work on next:

1. Run `kata next --unowned --agent`. It returns the highest-priority unclaimed open issue with no open `blocks` predecessor; it does not claim it.
2. If selection needs comparison, run `kata ready --unowned --agent`. Inspect unfamiliar hierarchy with the rendered `kata list --status all` tree and `kata show <root-ref> --agent`.
3. Prefer upstream feeders and work without a stakeholder loop when priority is otherwise close.
4. Treat both Blue and Red results as eligible. If a Red issue still needs a Blue decision, fix the missing relationship instead of relying on type preference or priority.
5. Recommend or select one issue and state the tradeoff in 1-3 sentences.
   - In Human mode, do not claim or start until the user agrees.
   - In Agent mode, record the choice and tradeoff, then claim and start without waiting.


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
