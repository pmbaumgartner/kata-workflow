# Agent Decision Mode

Use Agent mode only after the user explicitly delegates decisions for reversible, non-production work. Human mode remains the default. There is no Hybrid mode.

## Contents

- Activation
- Operating Rules
- Decision Workflow
- Agent Signoff
- Continuous Goal Loop
- End-Of-Run Report

## Activation

Record the mode and authority durably in the root issue or a reviewed project policy:

```md
## Decision Mode

Agent

## Decision Authority

The primary agent may make planning and technical decisions necessary to achieve this goal within its stated scope and existing permissions. Work is reversible and does not ship directly to production.
```

Child issues inherit the mode and cite the authority source:

```md
## Decision Mode

Agent, inherited from kata#abc4
```

A Goal supplies persistence and an objective, not decision authority. Do not infer Agent mode merely because a Goal is active.

## Operating Rules

- Do not request human frame checks, decisions, or signoff during the run.
- Gather evidence proportionate to the consequence and reversibility of the choice.
- When evidence is close, choose the smallest, simplest, least-coupled, easiest-to-reverse option.
- Treat open questions as research prompts. Resolve them with evidence or state an assumption and proceed.
- Prefer an experiment or narrow vertical slice when it can cheaply replace speculation.
- Use independent agents for investigation or adversarial review when useful, but keep final ownership with the primary agent.
- Defer nonessential uncertainty to a linked follow-up issue and continue other ready work.
- Preserve detailed decisions in the selected Blue issue record or artifact and give the human a compact report at the end.

Agent mode changes decision ownership, not action permissions. Do not treat it as authority for production deployment, destructive operations, external communication, spending, or other actions outside the user's granted scope. If a skipped unauthorized action is nonessential, record it and continue. If the goal is impossible without unavailable authority or an environmental dependency, report the genuine blocker with evidence.

## Decision Workflow

1. Draft the decision frame in the issue or artifact selected by the Blue workflow.
2. Audit the frame against the goal, parent issue, and authority source.
3. Explore plausible alternatives and gather evidence.
4. Rank options against the stated criteria.
5. Run an adversarial review: identify the strongest objection, missing option, and evidence that would reverse the recommendation.
6. Choose without waiting for human input.
7. Record signoff in the decision record.
8. Create necessary follow-up issues, close the Blue issue, and continue with ready work.

Low confidence does not block a decision. Record the uncertainty and choose the easiest-to-reverse path. Reopen or supersede the decision later if new evidence invalidates it.

## Agent Signoff

```md
## Decision Signoff

**Mode:** Agent
**Authority:** Inherited from <root-ref or policy path>
**Decision:** <choice>
**Confidence:** High | Medium | Low
**Material assumptions:** <unresolved facts accepted for this decision, or none>
**Reversal:** <how to undo or supersede the decision>
```

Keep goal alignment, evidence, rejected alternatives, and adversarial review in the decision record rather than duplicating them in signoff. Close the Blue issue after signoff and any required artifact review. Closure means the delegated decision was made, documented, and checked; it does not claim later human endorsement.

## Continuous Goal Loop

Repeat without human checkpoints:

1. Read the root authority and current issue graph.
2. Select and claim the best ready issue; set `work.attention=ok`.
3. Complete Blue decisions or Red implementation.
4. Verify, sign off, and close completed work.
5. Reconcile new issues and relationships.
6. Continue until the goal is achieved or a genuine environmental blocker prevents progress.

Create issues only for necessary work, newly exposed decisions, or durable follow-up. Do not generate backlog merely to keep the loop active.

## End-Of-Run Report

Report decisions retrospectively rather than requesting approval:

```md
## Autonomous Decision Report

### Decisions made
- <ref> - <decision and reason>. Confidence: <level>. Reversal: <path>.

### Material assumptions
- <assumption and affected refs>

### Deferred work
- <ref or explicitly none>

### Result
- <goal outcome, verification, and remaining risk>
```

Link to the detailed artifacts instead of repeating their full evidence. If the human later disagrees, reopen or supersede the affected Blue issue and create any required reversal work.
