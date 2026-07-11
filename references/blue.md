# Blue Kata Issues

Blue issues are decision and discovery work. Their output is "we know enough to
act," not shipped code.

## Contents

- Issue Body
- Decision Frame
- Workflow
- Final Artifact
- Close-Out

## Issue Body

Use label `blue`. Start the title with one of the Blue verbs listed in SKILL.md; do not add a `[Blue]` prefix.

Examples: `Decide whether to retain ANN dispatch`, `Investigate Safari callback duplication`, `Compare vector-index options`.

Required fields:

- `## Decision Mode` - `Human`, or `Agent` with the inherited authority source; omitted means Human
- `## Question` - the decision or unknown to resolve
- `## Context` - selected context, not a full history
- `## Predictability` - one of:
  - `Choosing between known options`
  - `Probing the unknown`
- `## Approach` - how to investigate
- `## Decision Artifact` - path such as `docs/<slug>.md`
- `## Exit Criteria` - what makes the decision settled
- `## Deliverables` - include the decision artifact path

In Human mode, use the decision owner named by the issue or project and default to the current user. In Agent mode, the primary agent is the decision owner; other agents may advise or review but do not sign off.
Prefer `docs/<slug>.md` for decision artifacts unless the project has another
convention.

## Decision Frame

The first draft is an outline with selected context, not the answer. Use it to check whether the original issue still points at the right higher-level goal.

Initial doc shape:

```md
---
kata: <ref>
created: YYYY-MM-DD
---

# <Title>

**Decision to make:**
**Why now / higher-level goal:**
**Goal check:**
**Evaluation criteria:**
**Hypotheses / options:**
**Evidence needed:**
**Known constraints:**
**Open questions:**
```

Do not rush to a final recommendation in the first draft.

## Workflow

1. Read the issue body, parent, links, and existing docs.
2. Draft the decision frame at the artifact path.
3. Frame-check the higher-level goal, criteria, and hypotheses.
   - In Human mode, pause and iterate with the decision owner until they confirm proceeding.
   - In Agent mode, audit the frame against the delegated goal and continue without pausing.
4. Diverge: explore plausible options, including likely losers.
5. Converge: gather evidence and rank options against the criteria.
6. Keep `## Open questions` ordered by what blocks the decision.
   - In Human mode, minibatch 1-3 questions from the top and iterate with the relevant participants.
   - In Agent mode, resolve questions through evidence or documented assumptions; do not send them to the human for approval.
7. Before signoff, ask what would make the recommendation wrong.
8. Record explicit decision-owner signoff. Human mode requires the human owner's response; Agent mode uses the signoff defined in agent-mode.md.
9. Create necessary follow up issues.
10. Finalize the artifact, then close and mark it as reviewed.

Fast path: for small "Choosing between known options" issues, one frame-check and one evidence pass may be enough.

For "Probing the unknown" issues in Human mode, do at least one human checkpoint before the doc contains a final recommendation. In Agent mode, do an adversarial review instead.

Do not substitute another participant's agreement for the named decision owner's signoff. In Agent mode, advice or review from another agent does not replace primary-agent signoff.

## Final Artifact

After evidence and mode-appropriate calibration, convert the decision frame into the final artifact:

```md
**Finding:** 1-3 sentences.
**Recommendation:** What next? Link follow-up Red/Blue issues.
**Confidence:** High / Medium / Low - what would raise it?
**Evidence:** Commands, benchmarks, docs, upstream behavior, links.
**Rejected alternatives:** What was considered and why it lost.
**Dead ends:** What did not work, so nobody repeats it.
```

Remove `## Open questions` before close, unless the final decision is explicitly to defer and the remaining questions are part of that outcome.

If a question needs new investigation, open a child Blue issue and mark the current issue `--blocked-by` it rather than stalling.

## Close-Out

Blue closure must include:

- brief `--message` naming the settled decision
- explicit decision-owner signoff in the artifact or a short kata comment
- `--reviewed <path>` for the decision artifact
- `--commit <sha>` when there is a relevant commit, but do not require one just to close Blue work
- follow-up issues listed in the artifact, if opened
- rejected or deferred options recorded in the artifact

Example:

```bash
kata close abc4 --done \
  --message "Decided to keep match-based ANN dispatch." \
  --reviewed docs/ann_abstraction_decision.md \
  --agent
```

In Human mode, agreement to start work is not signoff.
