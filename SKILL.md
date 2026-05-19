---
name: kata-workflow
description: Use when an agent needs to inspect, create, update, triage, choose next work from, or close kata issues; follow Blue decision workflows and Red implementation workflows, preserve evidence, avoid false-closing, and use kata as the shared issue ledger across projects.
---

# Kata Workflow

Kata is the shared issue ledger. Use it as durable external memory for task scope, decisions, evidence, and close-out.

## First Moves

- Use `kata ready --json` when asked what to work on next.
- Use `kata list --status all --json`, `kata show <ref> --json`, and `kata search "<term>" --json` to orient.
- Do not `delete` or `purge` unless the user explicitly asks for that exact destructive action and ref.
- Keep local scratch state such as `.scratch/` untracked and out of code indexes unless the project explicitly says otherwise.

## Hard Rules

- Do not false-close. Closing asserts the work is verified.
- If work is incomplete, leave it open, add `needs-review` if useful, and comment with what was attempted and what remains.
- Close verified Red work promptly.
- Close Blue work only after explicit user decision-owner signoff.
- Use `blocks` / `blocked_by` for real sequencing. Use `related` only for context.
- Search before creating. Always pass `--idempotency-key <slug>-<YYYY-MM-DD>` on `kata create`.
- Every artifact-producing issue needs a `## Deliverables` block.
- Every deliverable path must be reviewed before close.
- Keep close `--message` brief. Put durable detail in reviewed docs/artifacts, typed evidence, or a short kata comment when no artifact exists.

## Refs And Invocation

- Refs are short IDs derived from ULIDs, such as `abc4`. Cross-project refs look like `kata#abc4`. Full ULIDs also resolve. Legacy numeric refs do not.
- Commands run against the current workspace unless `--workspace` or `--project` overrides it.
- Author resolves as `$KATA_AUTHOR` > `$USER` > `git user.name`.
- Use `--json` for parsing. If kata is uninitialized, report that `kata init` is needed.

Common commands:

```bash
kata search "login race" --json
kata create "fix login race" --body "Observed double-submit in Safari callback." \
  --idempotency-key "login-race-2026-05-02" --json
kata show abc4 --json
kata comment abc4 --body "Found another reproduction path." --json
kata label add abc4 safari --json
kata edit abc4 --blocks d4ex --json
```

## Relationships

Relationship flags are framed from the operating issue's point of view:

- `parent` - this issue is a sub-task of a larger issue
- `blocks` - this issue must resolve before the target can proceed
- `blocked_by` - the target must resolve before this issue can proceed
- `related` - useful context, not ordering

Agents use `kata ready --json`, so weak ordering links make the ready queue less trustworthy.

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

Example:

```bash
kata close abc4 --done \
  --message "Fixed Safari callback double-submit." \
  --commit "$(git rev-parse HEAD)" \
  --test "cargo test" \
  --test "uv run pytest" \
  --reviewed docs/corpus_profile.md
```

Other close forms:

```bash
kata close abc4 --duplicate-of d4ex --message "Same Safari race condition."
kata close abc4 --superseded-by d4ex --message "Replaced by broader scope."
kata close abc4 --wontfix --message "Out of scope after contract review."
kata close abc4 --audit-no-change --message "Reviewed schema; no change needed." \
  --evidence "no-change-audit:schema unchanged after review" \
  --reviewed internal/db/schema.sql
```

Parent close is refused while open children remain.

## Mutations

Every mutation verb (`edit`, `close`, `label`, `assign`, `reopen`) accepts `--comment TEXT`. Use it when wiring relationships or flipping labels so the reason lands with the mutation.

```bash
kata edit abc4 --blocked-by skwh \
  --comment "Need skwh's decision artifact before we can scope this."
```

The mutation lands first. If the follow-up comment fails, retry with `kata comment <ref> --body ...`.

## Blue Or Red

Every issue is either:

- Blue: thinking work. Outputs decisions, findings, recommendations.
- Red: doing work. Outputs code, models, features, pipelines.

Blue variability is useful; Red variability is a cost to minimize.

No purple work: split mixed work into separate Blue and Red issues.

For Blue work, read [references/blue.md](references/blue.md).
For Red work, read [references/red.md](references/red.md).

When Red work hits unexpected uncertainty, stop and open or update a Blue issue rather than pushing through.

## What's Next

When the user asks what to work on next:

1. Run `kata ready --json`. It returns open issues with no open `blocks` predecessor. It does not include labels; classify by `[Blue]` / `[Red]`.
2. Inspect unfamiliar structure with `kata show <root-ref> --json`.
3. Prefer upstream feeders and work without a stakeholder loop.
4. Prefer Blue before Red within a phase.
5. Recommend one issue and the tradeoff in 1-3 sentences. Do not start until the user agrees.


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
kata create "<title>" --body-file .kata/drafts/<slug>.md --idempotency-key <slug>-<date>
kata comment <ref> --body-file .kata/drafts/<ref>-<topic>.md
kata edit <ref> --body "$(cat .kata/drafts/<ref>-<i>.md)"
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
