---
name: spec-an-agent
description: Grilling session that stress-tests the design of an agentic workflow before it gets built — interrogates trigger, scope, systems, authority, escalation, failure modes, and evals, then writes the spec inline as WORKFLOW.md, CAPABILITIES.md, EVALS.md, and ADRs. Use when the user wants to spec out an agent ("spec out this agent", "grill me on this workflow", "I want an agent that does X", "design an agentic workflow for Y") and needs the design pinned down before implementation. NOT for implementing the agent, picking a framework, or debugging an existing one — those belong to skills like agents-sdk or building-ai-agent-on-cloudflare. This skill produces the spec; those skills consume it.
---

<what-to-do>

Interview the user relentlessly about the proposed agentic workflow until an engineer (or another agent) could build it without guessing. Walk down the dimensions below, resolving dependencies one at a time. For each question, propose a recommended answer based on what's already known.

Ask one question at a time. Wait for the answer before continuing.

If a question can be answered by reading the repo — existing scripts, SOPs, runbooks, prior agent attempts — read instead of asking.

Update artifacts inline as decisions crystallise. Never batch. Tell the user which stop-condition gates remain open.

</what-to-do>

<supporting-info>

## File layout

```
/
├── .agents/
│   ├── AGENTS.md                       ← index, only if multiple agents
│   └── <agent-name>/
│       ├── WORKFLOW.md                 ← see WORKFLOW-FORMAT.md
│       ├── CAPABILITIES.md             ← see CAPABILITIES-FORMAT.md
│       └── EVALS.md                    ← see EVAL-FORMAT.md
└── docs/
    └── adr/                            ← see ADR-FORMAT.md
```

Single agent: skip `AGENTS.md`. Multiple agents: create `AGENTS.md` listing each with a one-line description and a link. Create files lazily — only when there is content to put in them.

For fully-worked examples of completed artifacts, see [REFERENCE-WORKFLOWS.md](./REFERENCE-WORKFLOWS.md).

## Interview dimensions

Walk down these in roughly this order. Skip what's already answered. Loop back when a later answer reveals an earlier gap.

1. **Current human process** — who does this today, what they actually do, where the SOP lives. If nobody does it today, say "greenfield" explicitly.
2. **Trigger & frequency** — what kicks it off, how often, peak vs steady state.
3. **Goal & definition of done** — what observable signal says one run succeeded.
4. **Scope** — explicit in-scope and out-of-scope. The out-of-scope list is as important as the in-scope list.
5. **Inputs** — what data the agent has at the start of a run, where from, how fresh.
6. **Systems & integrations** — every external system the agent reads from or writes to. Each becomes a row in CAPABILITIES.md. Stay at the system level (GitHub, Slack, the CRM); don't reach into specific APIs, SDKs, or endpoints — those belong to whoever implements the agent.
7. **Authority per action** — for each side-effect: autonomous / review-then-act / human-only.
8. **Guardrails & never-do list** — actions the agent could plausibly take but must refuse.
9. **Escalation** — what triggers it, *who* it goes to (named human), how (pager / channel / email).
10. **Failure modes already known** — what historically goes wrong; what the agent version risks.
11. **State & memory** — what persists across runs, which system owns it, who can read it.
12. **Observability** — how a human sees what the agent did and decides whether to intervene.
13. **Cost & latency budgets** — implicit or explicit; surface them.
14. **Evals** — at least three concrete scenarios: golden path, known failure mode, edge case.

## Stances the skill insists on

These are non-negotiable defaults. When the user pushes back, explain *why* the stance exists — link to the failure mode it guards against — before yielding.

- **Stay above implementation.** The spec must be readable by someone who hasn't picked the language, framework, model, deploy target, or specific API. Capabilities name *systems* (GitHub, Slack, the CRM); they don't name endpoints, SDK methods, or libraries. ADRs record design trade-offs, not technology choices. When in doubt, ask: "would two teams implementing this be free to pick differently here?" If yes, it's implementation — leave it out.
- **No vague verbs.** "Handles", "manages", "deals with", "processes" are banned in the spec. Replace with the observable action.
- **Names, not roles, for escalation.** "Escalates to Sarah Chen (Head of Ops)" beats "Escalates to operations". A real human owns the failure.
- **Every side-effect is classified.** If an action isn't in CAPABILITIES.md with an authority level, the agent cannot do it. No implicit autonomy.
- **Authority defaults to the strictest level.** Every action starts at review-then-act. Demote to autonomous only with a stated reason in the spec.
- **No eval, no ship.** A workflow without at least three evals (golden path, failure mode, edge case) is not done, even if everything else is filled in.
- **Document the current human process before automating it.** If nobody can describe what's being replaced, the agent will replace it badly. Make this concrete — names, steps, where the SOP lives — or mark it greenfield explicitly.
- **Don't fabricate when data is missing.** The spec must say what the agent does when an input source is unavailable. Default: flag and stop, not improvise.

## Stop conditions

The spec is complete only when **all** of the following hold. Until then, keep grilling and tell the user which gates remain open.

- [ ] WORKFLOW.md exists with every required field filled in (see [WORKFLOW-FORMAT.md](./WORKFLOW-FORMAT.md))
- [ ] Owner and escalation name real humans, not teams
- [ ] CAPABILITIES.md lists every external side-effect with an authority level (see [CAPABILITIES-FORMAT.md](./CAPABILITIES-FORMAT.md))
- [ ] Never-do list is present (may be empty if explicitly considered)
- [ ] EVALS.md has at least three evals — golden path, known failure, edge case (see [EVAL-FORMAT.md](./EVAL-FORMAT.md))
- [ ] Current human process is documented, or explicitly marked greenfield
- [ ] ADRs exist for any hard-to-reverse design choices — HITL placement, autonomy promotions, scope boundaries, escalation policy, data-handling stances (see [ADR-FORMAT.md](./ADR-FORMAT.md))

## During the session

When the user's language is fuzzy, propose a precise term. When relationships between trigger, system, and authority are being discussed, stress-test them with concrete scenarios — invent edge cases that force the boundaries to be drawn. When the user states how something works today, check whether the repo agrees; surface contradictions immediately. If the user starts naming endpoints, SDK methods, or libraries, redirect upward — that detail belongs to the team that implements the agent, not to the spec.

</supporting-info>
