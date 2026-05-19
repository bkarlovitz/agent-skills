# EVALS.md Format

`EVALS.md` is the test suite for an agentic workflow — a set of frozen scenarios with expected behaviour. It is the file that lets you ship the agent and notice when its behaviour regresses. Unlike a unit test, an eval names the *failure mode it guards against* so anyone reading the file knows why the scenario is there.

A workflow spec is not complete until EVALS.md contains at least three evals covering: a **golden path**, a **known failure mode**, and an **edge case**.

## Structure

```md
# Evals — {Workflow name}

## EVAL-001 — {Short descriptive name}

**Category:** Golden path | Known failure mode | Edge case

**Input:**
{The actual input data, not a paraphrase. The real issue body, the real CRM snapshot, the real Slack thread. Trim only obvious noise.}

**Expected behavior:**
- {Observable action the agent should take}
- {What it should NOT do, when the negative is non-obvious}

**Why this eval exists:**
{One or two sentences. What failure mode does this scenario guard against, or what behaviour does it lock in? If you can't answer this, the eval is not pulling its weight.}

**Counts as failure when:**
- {Concrete failure signal — wrong label, fabricated number, sent message that shouldn't have been sent}

---

## EVAL-002 — {...}

{...}
```

## Rules

- **Inputs are real.** "An issue about a bug" is not an eval — the actual issue body is. Real inputs catch real failures; synthesised inputs catch synthesised failures.
- **Expected behaviour is observable.** "Labels the issue as a bug" beats "understands the issue is about a bug". Pin the externally-visible outcome, not the internal reasoning.
- **Every eval names its failure mode.** The "Why this eval exists" line is non-negotiable. If you can't say what the eval guards against, delete it — it'll rot.
- **Three categories must be present** before the spec is complete:
  - **Golden path:** the most common, easiest case. Locks in baseline behaviour.
  - **Known failure mode:** a case the human version of this workflow gets wrong, or a case the agent has gotten wrong before.
  - **Edge case:** rare, weird, or ambiguous input — the kind that makes a junior reviewer pause.
- **Real incidents become evals immediately.** When the agent (or its human predecessor) gets something wrong in production, the input goes in EVALS.md the same day, with the correct behaviour pinned.
- **Negative expectations are explicit when they matter.** "Does not auto-close the issue" or "Does not send a message" — list these when the wrong action is plausible.
- **Don't paraphrase failures.** "Counts as failure when: the wrong label" is too loose. "Counts as failure when: the issue receives a label outside the approved-set, or a 'bug' label without including a reproduction-steps comment" is a rule.
- **Order doesn't matter, but stability does.** Use sequential IDs (EVAL-001, EVAL-002) so they can be referenced from incident reports and ADRs. Don't renumber when adding new evals.

## Worked example (one eval)

```md
## EVAL-003 — Security report mis-categorised as feature request

**Category:** Known failure mode

**Input:**
Title: "Suggestion: add rate limiting to /api/users"
Body: "Hey team — noticed the endpoint is unauthenticated and there's no rate limit. I was able to enumerate user emails by hitting it 10k times. Probably worth fixing? Not super urgent but figured I'd flag."

**Expected behavior:**
- Apply the `security` label, not `enhancement` or `feature-request`.
- Escalate to the named security owner (Maya Patel) via the security-incidents Slack channel.
- Do NOT respond to the reporter with a generic "thanks for the feature request" comment.

**Why this eval exists:**
Security reports often arrive disguised as feature suggestions because reporters don't always know the right framing. Mis-labelling as `enhancement` buries the issue. This scenario was a real incident on 2025-11-12 (see incident-074).

**Counts as failure when:**
- A label other than `security` is applied as the primary label.
- The security owner is not notified within the same run.
- A reply comment treats the issue as a routine feature request.
```

## Optional: input fixtures

For larger inputs (e.g., a full CRM export, a multi-thread Slack channel dump), keep the input in a sibling file under `.agents/<agent-name>/eval-fixtures/EVAL-NNN-input.json` and reference it from the eval entry. Don't bloat EVALS.md itself — keep it readable.
