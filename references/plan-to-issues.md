# Plan Markdown To Issues

Use this workflow when translating a large markdown plan, or several smaller plan documents, into kata issues. Preserve source fidelity, but improve the issue set when codebase evidence exposes missing scope, stale assumptions, or better increments.

## Workflow

1. Coordinator draft:
   - Read every source plan first.
   - Identify outcomes, phases, dependencies, deliverables, unresolved decisions, acceptance evidence, and likely repo areas.
   - Decide the provisional issue graph before creating anything. Classify each issue by its completion claim. Split thinking and doing only when they have distinct durable decision and implementation outputs; keep ordinary local implementation judgment inside Red work. Give each issue exactly one `blue` or `red` label and start its title with the corresponding verb family from SKILL.md.
   - Build a source coverage map from plan headings, bullets, checklist items, or requirement IDs to planned issue titles.
   - Search existing kata issues before creation and reuse/update obvious matches instead of duplicating them.

2. Pre-mortem and modularity review:
   - Before creating issues, assume the issue set failed to deliver the source plan. Name concrete causes.
   - Look for missing issues, oversized issues, hidden dependencies, vague acceptance evidence, unverified code assumptions, missing migrations, missing rollback/observability/tests, and material scope, acceptance, policy, or settled-decision changes buried inside Red work.
   - Convert each material risk into an issue split, issue edit, relationship candidate, new Blue issue, or decision. Route decisions through the Blue workflow's consequential/nonconsequential protocol; do not add another checkpoint.
   - Apply the Lego rule: prefer small, repeatable, independently verified ship units.
   - Favor vertical slices over layer-only tasks. For each Red issue ask: smallest useful ship unit, independent verification, stable interface, and whether later work can repeat the pattern.

3. Unlinked creation:
   - Create issues in parallel only when independent agents are available, scopes do not overlap, and coordination overhead is justified. Otherwise, have the coordinator create them sequentially.
   - When using subagents, tell each one it is not alone in the kata ledger and must not create relationships yet.
   - Each creator handles only its assigned issue(s), using `--idempotency-key <plan-slug>-<issue-slug>-<YYYY-MM-DD>`.
   - Each issue body must preserve provenance with a short `## Source Plan` block naming the file(s) and section(s) covered.
   - Each artifact-producing issue still needs a `## Deliverables` block.
   - Creators may add necessary detail from repo/code context, but must report additions separately from source-plan content.
   - Whether work is parallel or sequential, record for each issue: created/reused kata ref, title, Blue/Red label, source sections covered, likely `blocks` / `blocked_by` / `related` candidates, additions, and open questions.

4. Central linking:
   - Inspect the creation record and `kata show <ref> --json` for each created/reused issue.
   - Add all relationships centrally after creation. Do not link during parallel creation.
   - Use `blocks` / `blocked_by` only for true ordering constraints. Use `related` for shared context.
   - Add mutation comments that explain non-obvious sequencing decisions.
   - If creation exposed an unresolved decision, create or update a Blue issue and block affected Red work on it.

5. Drift reconciliation:
   - Before starting or handing off an issue, compare its assumptions against current code, docs, tests, and existing kata comments.
   - If the issue is stale, contradicted, or underspecified, comment or edit before implementation. Do not silently work around stale scope.
   - If implementation discovers a durable constraint, record it in acceptance evidence, a deliverable doc, or a Blue issue.
   - If a bug or test failure reveals a missing invariant, use a backprop-style loop: trace root cause, decide whether a reusable invariant or acceptance check would catch recurrence, then fix. Follow project testing policy; add a regression test only when the behavior remains part of the intended contract, and use an explicit evidence expectation when a test would preserve removed behavior or merely test a language or third-party guarantee.
   - Distinguish source-plan drift, codebase drift, and useful scope additions. Surface unresolved calls to the user in Human mode; resolve them with documented assumptions in Agent mode.

6. Fidelity review:
   - Reconcile the source coverage map against the final issue set and relationship graph.
   - Verify every issue has exactly one `blue` or `red` label and a matching opening verb.
   - Verify every substantive source-plan requirement is represented by a kata issue, explicitly folded into another issue, or intentionally left out with a noted reason.
   - Verify source-plan dependencies became `blocks` / `blocked_by` links when they affect readiness.
   - Verify deliverables, acceptance checks, and evidence expectations were not lost.
   - Verify pre-mortem risks and drift findings were handled. Carry unresolved items as explicit questions in Human mode and as documented assumptions or deferred issues in Agent mode.
   - Review additions from issue creation. Keep useful repo-grounded additions. In Human mode, surface speculative additions or unresolved scope questions to the user; in Agent mode, decide or document an assumption.
   - If fidelity is incomplete, edit issues or create missing ones before reporting completion.

7. Human decision handoff:
   - In Human mode, inspect the finalized graph for open Blue issues that are unblocked, require an explicit human choice under the Blue workflow, and block one or more open Red issues.
   - If any exist, list each Blue ref with a brief decision summary and the Red refs it blocks. Ask whether the user would like to proceed with those Blue decisions first so the dependent Red work can become ready.
   - Do not start or claim those Blue issues until the user agrees. If none exist, omit the decision-first prompt.

Final response: list the created/reused refs, major links added, coverage/fidelity result, handled pre-mortem/drift findings, and additions. In Human mode, include user decisions still needed and the decision-first offer from step 7 when applicable. In Agent mode, include decisions made, material assumptions, confidence, and reversal paths.
