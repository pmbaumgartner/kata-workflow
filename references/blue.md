# Blue Kata Issues

Blue issues are decision and discovery work. Their output is "we know enough to
act," not shipped code.

## Contents

- Issue Body
- Decision Frame
- Workflow
- Final Record
- Close-Out

## Issue Body

Use label `blue`. Start the title with one of the Blue verbs listed in SKILL.md; do not add a `[Blue]` prefix.

Examples: `Decide whether to retain ANN dispatch`, `Investigate Safari callback duplication`, `Compare vector-index options`.

Required fields:

- `## Decision Mode` - `Human`, or `Agent` with the inherited authority source; omitted means Human
- `## Question` - the decision or unknown to resolve
- `## Context` - selected context, not a full history
- `## Decision Stakes` - `Consequential: Yes | No - <brief rationale>`, using the definition in SKILL.md
- `## Approach` - how to investigate
- `## Exit Criteria` - what makes the decision settled

Use one of two recording scales:

- Inline Blue - for a small, reversible decision local to one issue. Record the frame, evidence, decision, and who decided in the issue body or a short comment. A separate artifact and `## Deliverables` block are not required.
- Artifact Blue - for a decision that is expensive to reverse, affects multiple issues, establishes reusable policy, or needs durable evidence beyond the issue record. Add `## Decision Artifact` and `## Deliverables` fields and record the decision in a document. By default write it under the gitignored `.kata/` directory (see **Artifacts** in SKILL.md); write it to a tracked repo path such as `docs/<slug>.md` only when the user requests durable, version-controlled persistence.

In Human mode, the primary agent decides nonconsequential Blue work by intent; the decision owner named by the issue or project, defaulting to the current user, decides consequential Blue work. In Agent mode, the primary agent is the decision owner.
When the user asks for a tracked artifact, prefer `docs/<slug>.md` unless the project has another convention.

## Decision Frame

The first draft is an outline with selected context, not the answer. Use it to check whether the original issue still points at the right higher-level goal. For Inline Blue, the required issue fields may serve as this frame; do not create a document merely to reproduce them.

Artifact Blue doc shape:

```md
---
kata: <ref>
created: YYYY-MM-DD
---

# <Title>

**Decision to make:**
**Why now / higher-level goal:**
**Goal check:**
**Consequential:** Yes | No - <brief rationale>
**Evaluation criteria:**
**Hypotheses / options:**
**Evidence needed:**
**Known constraints:**
**Open questions:**
```

Do not rush to a final recommendation in the first draft.

## Workflow

1. Read the issue body, parent, links, and existing docs.
2. Draft the decision frame in the issue for Inline Blue or at the artifact path for Artifact Blue.
3. Classify the decision as consequential or nonconsequential. Pause early only when scope or authority is unclear.
4. Investigate and compare plausible alternatives in proportion to the stakes. Resolve researchable questions with evidence; in Agent mode, document any remaining assumptions.
5. Identify the strongest credible alternative and what evidence would change the recommendation.
6. Decide:
   - In Human mode, state intent and decide nonconsequential work without a checkpoint.
   - In Human mode, present consequential work in the format below and pause once for the human decision owner.
   - In Agent mode, choose without a human checkpoint and use the signoff in agent-mode.md.
7. Record the decision and who decided, then create necessary follow-up issues.
8. Finalize the decision record. For Artifact Blue, mark the artifact as reviewed, then close.

For a consequential Human-mode decision, present:

```md
## Decision Needed

- **Decision:** <the choice the human needs to make>
- **Why this matters now:** <higher-level goal, stakes, and downstream effect>
- **Decision artifact:** <path or link to the durable record>
- **Relevant context:** <current state, material evidence, constraints, and uncertainties>
- **Option A:** <tradeoff>
- **Option B:** <tradeoff>
- **Recommendation:** <choice and reason>
- **What would change the recommendation:** <evidence, or none>
```

Make the conversational presentation decision-ready without requiring the human to open the artifact. Reference the artifact as the durable record, not as a substitute for context. Include enough selected context to explain the choice and its consequences while leaving full history and supporting detail in the artifact.

Use 2-3 viable options when they exist; do not invent alternatives. High uncertainty scales investigation but does not by itself require a human checkpoint. For nonconsequential Inline Blue work, one proportionate evidence pass and recorded intent may be enough.

## Final Record

After evidence and mode-appropriate calibration, convert the frame into the final issue record or artifact:

```md
**Finding:** 1-3 sentences.
**Decision:** The chosen option and who decided.
**Recommendation:** What next? Link follow-up Red/Blue issues.
**Confidence:** High / Medium / Low - what would raise it?
**Evidence:** Commands, benchmarks, docs, upstream behavior, links.
**Rejected alternatives:** What was considered and why it lost.
**Dead ends:** What did not work, so nobody repeats it.
```

For Inline Blue, keep only the fields needed to make the decision legible; record rejected alternatives and dead ends only when material.

Remove `## Open questions` before close, unless the final decision is explicitly to defer and the remaining questions are part of that outcome.

If a question needs new investigation, open a child Blue issue and mark the current issue `--blocked-by` it rather than stalling.

## Close-Out

Blue closure must include:

- brief `--message` naming the settled decision
- mode-appropriate decision recorded in the issue, artifact, or a short kata comment: agent intent for nonconsequential Human-mode work, explicit human choice for consequential Human-mode work, or primary-agent signoff in Agent mode
- `--reviewed <path>` for Artifact Blue; omit it for Inline Blue unless another deliverable exists
- `--commit <sha>` when there is a relevant commit, but do not require one just to close Blue work
- follow-up issues listed in the decision record, if opened
- material rejected or deferred options recorded in the decision record

Example:

```bash
kata close abc4 --done \
  --message "Decided to keep match-based ANN dispatch." \
  --reviewed docs/ann_abstraction_decision.md \
  --agent
```

For consequential Human-mode work, agreement to investigate is not the final decision.
