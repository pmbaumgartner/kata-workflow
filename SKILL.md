---
name: kata-workflow
description: Use when an agent needs to inspect, create, claim, assign, prioritize, coordinate, update, link, or close Kata issues; choose next work; translate plans into issue graphs; or run delegated reversible work. Enforce Blue decision and Red implementation workflows, durable evidence, ownership, and verified close-out across projects.
---

# Kata Workflow

Use Kata as the shared issue ledger and durable memory for scope, decisions, evidence, and close-out.

## Start And Scope

- Run `kata quickstart` in an unfamiliar workspace or after an upgrade. Treat it as normative for operating rules and `kata <command> --help` as normative for syntax.
- Treat examples here as checked against Kata v0.16.0 (`kata_api_version: 1`, `agent_format: 1`). If the installed CLI drifts, follow its contract and report the mismatch before mutating.
- Resolve project-scoped commands from the workspace, `--workspace`, or `--project`. Without a `.kata.toml` binding, use `--project <name>` or qualified refs; run `kata init` only when binding is intended.
- Prefer `--agent` for reads and mutations. Use `--json` only for scripts that need the full structured shape.
- Orient with `kata list --status all --agent`, then inspect with `kata show <ref> --agent` and `kata search "<term>" --agent`.
- Use `list --all`, `ready --all`, or `next --all` only for intentional cross-project discovery. Keep mutations project-scoped and use qualified refs.
- Resolve identity with `kata whoami --agent` when attribution matters. Give every mutating agent in a fan-out a distinct `--as` or `$KATA_AUTHOR`.
- Never run `delete` or `purge` unless the user explicitly requests that exact destructive action and ref.

For JSON shapes, low-frequency command behavior, optional interfaces, hooks, and daemon operations, read [references/cli-gotchas.md](references/cli-gotchas.md).

## Non-Negotiable Rules

- Never false-close. Closing asserts verified completion.
- Leave incomplete work open. Add `needs-review` when useful and comment with what was attempted, evidence, and the next action.
- Close verified Red work promptly.
- Close Blue work only after recording the mode-appropriate decision.
- Use `blocks` / `blocked_by` for sequencing and `related` only for context.
- Rank ready work with priority, never as a substitute for dependencies. Priority `0` is highest.
- Search before creating and always pass `--idempotency-key <slug>-<YYYY-MM-DD>`.
- Treat a failed claim as a coordination signal. Never `--force` ownership transfer without explicit user direction.
- Give every artifact-producing issue a `## Deliverables` block and review every listed path before close.
- Keep close messages brief but substantive. Put durable detail in reviewed artifacts, typed evidence, or a short issue comment.

## Decision Modes

Use exactly one mode for a goal or root issue:

- **Human mode** is the default. State intent and decide nonconsequential choices within the authorized goal. For consequential choices, present viable options, tradeoffs, and a recommendation, then pause once for the user.
- **Agent mode** requires explicit user delegation for reversible, non-production work. The primary agent decides, records signoff, closes Blue issues, and reports decisions at the end without human checkpoints.

Treat a decision as consequential when a wrong choice could cause material harm, establish a durable dependency or precedent, commit significant downstream work, or require more than a local correction to reverse. Uncertainty alone only increases the evidence required.

Record Agent mode and its authority source in the root issue or a reviewed policy; child issues inherit it. A Goal does not select Agent mode or expand permissions. For the full workflow and signoff format, read [references/agent-mode.md](references/agent-mode.md).

## Blue And Red

Give every issue exactly one canonical workflow label:

| Type | Completion claim | Opening verbs |
| --- | --- | --- |
| Blue | Settled knowledge: decision, finding, or recommendation | `Decide`, `Investigate`, `Evaluate`, `Compare`, `Determine`, `Define`, `Audit` |
| Red | Verified shipped change | `Add`, `Fix`, `Implement`, `Migrate`, `Remove`, `Refactor`, `Optimize`, `Upgrade`, `Document`, `Configure` |

Do not add `[Blue]` or `[Red]` prefixes. Classify by the completion claim and split mixed work only when decision and implementation outputs are independently durable. Keep ordinary local implementation judgment inside Red work.

- For Blue work, read [references/blue.md](references/blue.md).
- For Red work, read [references/red.md](references/red.md).

When Red work exposes uncertainty that would change scope, acceptance, settled decisions, or reusable policy, stop and open or update a Blue issue. Do not create retrospective Blue work unless execution produced reusable learning or exposed a faulty assumption.

## Relationships

Frame relationship flags from the operating issue:

- `parent`: this issue is a subtask of the target.
- `blocks`: this issue must finish before the target can proceed.
- `blocked_by`: the target must finish before this issue can proceed.
- `related`: useful context without ordering.

If Red work depends on a Blue decision, make the Red issue `--blocked-by` the Blue issue. Labels and titles do not affect readiness.

Use qualified refs for cross-project links. Federated projects synchronize them, so do not mirror dependency state with duplicate issues or comments.

## Creation, Ownership, And Coordination

Create with an idempotency key after searching:

```bash
kata search "login race" --agent
kata create "Fix login race" --body "Observed double-submit in Safari callback." \
  --label red --idempotency-key "login-race-2026-05-02" --agent
```

Claim work before starting and release it when pausing:

```bash
kata claim abc4 --comment "Starting the agreed implementation." --agent
kata unassign abc4 --comment "Releasing; blocked on a missing fixture." --agent
```

Use `work.attention` only as transient state owned by the working-agent side:

```bash
kata meta set abc4 work.attention ok --agent
kata meta set abc4 work.attention needs-human --agent
kata meta set abc4 work.attention_msg "Blocked on a missing prod fixture." --agent
```

The launcher owns `work.branch`. Coordinators may read `work.*` but must not overwrite the working agent's signal. For fan-out/fan-in, use `kata wait <refs...> --all --timeout <duration> --agent`; use `--any` or `--until attention` when appropriate, and always include duration units.

Before pausing or ending a session:

- Make ownership and attention truthful.
- Comment incomplete work with evidence and the next action.
- Close verified work with typed evidence.
- Turn sequencing discovered during work into relationships.
- Report relevant refs; leave no next step only in chat.

## Choosing Next Work

1. Run `kata next --unowned --agent`.
2. If comparison is needed, run `kata ready --unowned --agent`; inspect unfamiliar hierarchy with `kata list --status all` and `kata show <root-ref> --agent`.
3. Prefer upstream feeders and work without a stakeholder loop when priority is otherwise close.
4. If Red work still needs a Blue decision, add the missing dependency rather than relying on priority or type.
5. In Human mode, recommend one issue and wait before claiming. In Agent mode, record the choice and tradeoff, then claim and start.

## Closing Evidence

Use typed evidence:

- `--commit <sha>` for the commit containing the completed work.
- `--pr <url>` for a pull request.
- One repeatable `--evidence "test:<cmd>"` per verification command; use `--test "<cmd>"` only for one command.
- One repeatable `--reviewed <path>` per deliverable.
- `--audit-no-change`, `--duplicate-of <ref>`, or `--superseded-by <ref>` for their corresponding dispositions.

Do not invent evidence prefixes. Record manual checks, skipped checks, benchmarks, residual risk, and rationale in a reviewed artifact or a short comment.

Verify that commit evidence contains the completed work; never cite the current `HEAD` only because it is convenient. Preview complicated or high-impact closes with `--dry-run`.

```bash
kata close abc4 --done \
  --message "Fixed Safari callback double-submit." \
  --commit <sha-containing-completed-work> \
  --evidence "test:cargo test" \
  --reviewed src/callback.rs \
  --agent
```

Parent close is refused while children remain open. Follow the Blue or Red reference for mode-specific close-out requirements.

## Records And Artifacts

| Information | Location |
| --- | --- |
| Scope, acceptance, deliverables | Issue body |
| Status, blockers, attempts | Issue comment |
| Reusable findings and decision rationale | One repo document |
| Mutation rationale | Mutation `--comment` |

Write drafts, scratch notes, logs, and untracked evidence under the gitignored `.kata/` directory. Use `.kata/drafts/<slug>.md` with `--body-file` for multiline issue bodies or comments. Persist artifacts in tracked repo paths only when the user requests durable version-controlled output or a deliverable requires it.

Edit durable documents in place; do not create version-suffixed copies. Let Git history and Kata comments carry the change trail.

## Plan Markdown To Issues

When translating one or more plans into issues, read [references/plan-to-issues.md](references/plan-to-issues.md). Use its coordinator draft, pre-mortem and modularity review, unlinked creation, central linking, drift reconciliation, and fidelity review flow.
