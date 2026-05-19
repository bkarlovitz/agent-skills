# WORKFLOW.md Format

`WORKFLOW.md` is the single source of truth for one agentic workflow. It answers: what does this agent do, for whom, when, and what does "done" look like. It does **not** answer: what tools the agent uses (CAPABILITIES.md) or what specific scenarios it must handle correctly (EVALS.md).

## Structure

```md
# {Workflow name}

{One sentence: what the agent does and the observable outcome of a successful run.}

## Trigger

- **Kicks off when:** {event / cron / user request / threshold breach — be concrete}
- **Frequency:** {expected volume per day/week, peak vs steady state}

## Goal

{One paragraph. What "done" looks like for a single run. Must reference an observable signal — a label that exists, a draft document that's saved, a row that's updated. If you can't point at a thing that changes in the world, the goal isn't observable enough yet.}

## Scope

**In scope:**
- {Concrete responsibilities the agent owns}

**Out of scope:**
- {Things the agent must not attempt, even if they seem adjacent — equally important}

## Inputs

| Input | Source | Freshness | Required? |
|---|---|---|---|
| {what} | {where it comes from} | {how old at start of run} | {yes / fallback if missing} |

## Current human process being replaced

{Step-by-step description of who does this today and how. Cite the SOP if one exists. If this is greenfield with no existing process, say so explicitly — do not leave this section blank.}

## Owner & escalation

- **Owner:** {Named person, role in parens. e.g., "Sarah Chen (Head of Ops)"}
- **Escalates to:** {Named person} when {specific trigger condition}
- **Pager / channel:** {Where the escalation actually lands}

## Definition of done (per run)

- {Bulleted, observable, measurable criteria. Each item should be checkable by looking at external state.}

## Failure modes already known

- **{Failure name}:** {What goes wrong} → {How the agent should respond}

## State & memory

{What the agent remembers across runs, which system owns that state (system of record — not storage technology), who else can read or write it. If nothing persists, say "stateless — each run is independent" explicitly.}

## Observability

{How a human can see what the agent did. Log destination, summary report, dashboard, audit trail. Be specific about where to look.}

## Budgets

- **Cost:** {hard or soft cap per run, or "not budgeted yet"}
- **Latency:** {p50 / p95 target, or "not budgeted yet"}
```

## Rules

- **Every field is required.** If a field can't be filled, the workflow isn't ready — keep grilling. Exception: a field may say "not budgeted yet" or "stateless" or "greenfield, no existing process", but it cannot be silently omitted.
- **Names, not roles, for owner and escalation.** "Sarah Chen (Head of Ops)" beats "Operations team". A real human owns the failure.
- **Goal references an observable signal.** "Triages the issue" is not observable. "Issue has a label from the approved set, or a comment requesting one specific missing piece of information" is.
- **No vague verbs.** "Handles", "manages", "processes", "deals with" are banned. Replace with what observably changes.
- **Out-of-scope is as important as in-scope.** The boundary is where future scope creep gets argued. Make the boundary explicit.
- **Don't put tool details here.** Tools, integrations, and authority belong in CAPABILITIES.md. WORKFLOW.md should still make sense if the tools change.
- **Don't put example inputs/outputs here.** Concrete scenarios belong in EVALS.md. WORKFLOW.md is the rules; EVALS.md is the test cases.
- **Current human process must be concrete or explicitly greenfield.** If you can't describe it step by step, you haven't asked enough. If it doesn't exist, say so — don't leave the section ambiguous.
- **Failure modes are paired with responses.** Listing a failure mode without specifying what the agent should do about it is incomplete.

## Single vs multi-agent repos

**Single agent:** one `.agents/<name>/WORKFLOW.md`. Skip the index.

**Multiple agents:** a `.agents/AGENTS.md` index lists each agent with a one-line description and a link to its WORKFLOW.md. The skill infers which structure applies and creates the index lazily when a second agent appears.
