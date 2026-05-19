# ADR Format

ADRs live in `docs/adr/` and use sequential numbering: `0001-slug.md`, `0002-slug.md`, etc.

Create the `docs/adr/` directory lazily — only when the first ADR is needed.

## Template

```md
# {Short title of the decision}

{1-3 sentences: what's the context, what did we decide, and why.}
```

That's it. An ADR can be a single paragraph. The value is in recording *that* a decision was made and *why* — not in filling out sections.

## Optional sections

Only include these when they add genuine value. Most ADRs won't need them.

- **Status** frontmatter (`proposed | accepted | deprecated | superseded by ADR-NNNN`) — useful when decisions are revisited
- **Considered Options** — only when the rejected alternatives are worth remembering
- **Consequences** — only when non-obvious downstream effects need to be called out

## Numbering

Scan `docs/adr/` for the highest existing number and increment by one.

## When to offer an ADR

All three of these must be true:

1. **Hard to reverse** — the cost of changing your mind later is meaningful
2. **Surprising without context** — a future reader will look at the code and wonder "why on earth did they do it this way?"
3. **The result of a real trade-off** — there were genuine alternatives and you picked one for specific reasons

If a decision is easy to reverse, skip it — you'll just reverse it. If it's not surprising, nobody will wonder why. If there was no real alternative, there's nothing to record beyond "we did the obvious thing."

### What qualifies (for an agentic workflow spec)

ADRs in this skill record **design** decisions about how the agent behaves — not implementation choices. The team that builds the agent will pick the language, framework, model, SDK, and deploy target on their own; those are not ADR material here.

- **HITL placement.** Where in the workflow a human must approve before the agent proceeds. "The drafting step is autonomous; the send step is human-only." This is the single highest-leverage design call in most agent workflows.
- **Autonomy promotions.** When an action defaults to Review-then-act and the spec promotes it to Autonomous, record why. The reason should reference reversibility, frequency, and the failure-mode analysis behind the choice.
- **Scope boundaries that came from a real trade-off.** Out-of-scope items where a reasonable reader would assume the agent *should* be doing them. "The agent flags duplicates but never closes — even obvious spam." Stops the next person from "fixing" something deliberate.
- **Escalation policy.** Who gets paged, under what conditions, through which channel, with what SLA. Especially when the choice was non-obvious (e.g., paging an individual rather than a rotation).
- **Data-handling stances.** Categories of data the agent must never read or write, even when it has the credentials. "Never read `private-` or `exec-` Slack channels." "Never name individual employees in drafts."
- **Failure-mode response patterns.** When the agent encounters missing or stale input: flag-and-stop vs. retry vs. fall back vs. improvise. The default should be flag-and-stop; deviations need an ADR.
- **Constraints not visible in the spec itself.** Regulatory, contractual, or political constraints that shape the design but won't show up in any single artifact. "Customer-data actions require legal sign-off per the DPA with EnterpriseCo."
- **Rejected design alternatives when the rejection is non-obvious.** If you considered making an action autonomous and chose Review-then-act for subtle reasons, record it — otherwise someone will re-propose the autonomous version in six months.

### What does NOT qualify

Out of scope for ADRs in this skill — these are implementation choices owned by whoever builds the agent:

- Model selection, prompting strategy, context window management
- Framework, SDK, or library choice
- Sync vs async execution, queueing, retry mechanics
- Database, message bus, vector store, deployment target
- API endpoint or call structure

If you find yourself writing one of these, you've drifted from spec into implementation. Stop and move it out of the spec entirely.
