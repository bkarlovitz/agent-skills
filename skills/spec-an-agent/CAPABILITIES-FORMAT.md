# CAPABILITIES.md Format

`CAPABILITIES.md` is the authority surface for one agentic workflow. It lists every external side-effect the agent can take, the tool that makes it possible, and the authority level under which it operates. This file is what a security or safety reviewer should be able to read in two minutes to understand the blast radius.

If an action is not in this file, the agent **cannot** do it. No implicit autonomy.

## Structure

```md
# Capabilities — {Workflow name}

## Authority levels

- **Autonomous** — agent does this without human review
- **Review-then-act** — agent proposes; a named human approves; then the action runs
- **Human-only** — agent can suggest, but a human must execute

## Capabilities

| Action | System | Authority | Data class | Approval required when |
|---|---|---|---|---|
| {observable verb} | {External system the agent touches — e.g., GitHub, Slack, the CRM} | {Autonomous / Review-then-act / Human-only} | {Public / Internal / Confidential / Regulated} | {condition, or "Always" / "Never"} |

## Data classes

- **Public** — already shared externally; no harm if exposed
- **Internal** — internal-only but not sensitive
- **Confidential** — restricted; financial, customer, strategic
- **Regulated** — legal/compliance obligations apply (PII, PHI, payment data)

## Never-do list

- **{Forbidden action}:** {Why it's forbidden — the failure mode it would create}

## Read vs write surface

- **Reads from:** {list of systems the agent can read but not modify}
- **Writes to:** {list of systems the agent can modify, with link to relevant capability rows}
```

## Rules

- **Stay at the system level.** Capabilities describe what the agent can do *to which external system* — not which API, endpoint, SDK, or library. "Apply label to GitHub issue" with system "GitHub" is right; "GitHub REST API (issues.addLabels)" is the implementer's call, not the spec's. Two teams building this agent should be free to pick different SDKs and still satisfy the same CAPABILITIES.md.
- **Every side-effect is a row.** If the agent causes state to change in an external system, it goes in the table. Reading is also listed when the read itself is sensitive (e.g., reading customer PII).
- **Authority defaults to the strictest level.** Start every action at Review-then-act. Demote to Autonomous only with a stated reason captured either in this file or in an ADR. Promote to Human-only when the action is high-stakes and the agent shouldn't even execute on approval.
- **"Approval required when" is concrete.** "When risky" is not a rule. "When confidence < 0.7", "When the action touches an account flagged as VIP", "Always" — these are rules.
- **Data class describes the data the action touches**, not the data the agent stores. A `read_customer_record` action touches Confidential data even if the agent immediately summarises and discards.
- **Never-do list is for plausible-but-forbidden actions.** Don't pad with impossible actions. The list answers: "what's the worst thing the agent might try, and why must it refuse?"
- **Read vs write surface is a summary, not a duplicate.** Use it for quick scanning. If it disagrees with the capabilities table, the table wins.
- **Update this file when an action is added or its authority changes.** Authority drift is the most common way agents become unsafe in production.

## Worked example (one row)

| Action | System | Authority | Data class | Approval required when |
|---|---|---|---|---|
| Apply label to GitHub issue | GitHub | Autonomous | Public | Confidence < 0.7, or label is in the "sensitive" set (security, legal, hr) |

The reason this row is Autonomous: applying a label is cheap to reverse and high-frequency. The "approval required" column carves out the cases where autonomy *isn't* appropriate. Notice the System column says "GitHub", not a specific endpoint — that's a deliberate boundary. Which API call achieves "apply label" is the implementer's choice.

## When to split capabilities across multiple files

Usually a single file is enough. Split only when an agent has more than ~30 distinct actions or operates across clearly separate systems. In that case, organise by system (`CAPABILITIES-github.md`, `CAPABILITIES-slack.md`) and add a top-level summary in `CAPABILITIES.md` linking to them.
