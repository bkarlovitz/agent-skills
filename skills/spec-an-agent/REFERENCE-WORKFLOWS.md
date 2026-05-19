# Reference Workflows

Two worked examples showing what the skill's output looks like at completion. The first is mechanical and high-frequency (triage). The second is judgement-heavy and low-frequency (drafting). Together they span the range of workflows the skill should handle.

These examples are **complete enough to ship**. When skill-creator builds the skill, the agent should grill until its output for an analogous workflow looks like this.

---

## Reference A — GitHub issue triage agent (mechanical, high-frequency)

### `.agents/issue-triage/WORKFLOW.md`

```md
# GitHub issue triage

Labels new issues, suggests an owner, requests missing reproduction info when applicable, and flags suspected duplicates — so the maintainer's morning queue is pre-sorted.

## Trigger

- **Kicks off when:** GitHub webhook fires on `issues.opened` or `issues.reopened` on the `acme/api` repo.
- **Frequency:** ~30 issues/week steady state, bursts to ~100/week after a release.

## Goal

After one run, the issue has either (a) at least one label from the approved set and an assignee suggestion comment, or (b) a single comment requesting one specific piece of missing reproduction information. Nothing else changes on the issue.

## Scope

**In scope:**
- Labelling against the approved-set
- Suggesting an owner via comment (not assigning)
- Asking for missing repro info on bug reports
- Flagging suspected duplicates via comment

**Out of scope:**
- Closing issues (even suspected duplicates — humans close)
- Editing the issue body or title
- Responding to follow-up comments after the initial triage
- Triaging PRs

## Inputs

| Input | Source | Freshness | Required? |
|---|---|---|---|
| Issue title and body | GitHub (webhook payload) | Real-time | Yes |
| Issue author and history | GitHub | < 1 min | Yes |
| Repo's existing labels | GitHub | < 1 hour cache | Yes |
| Recent similar issues (last 90d) | Issue similarity search over the issue corpus | < 1 hour | Fallback: skip dupe check |

## Current human process being replaced

Today, Sarah Chen (engineering manager) triages issues every morning at 9am. She reads each issue, checks if it's a duplicate by searching the repo, applies one or more labels from the approved set, and either @-mentions the likely owner or asks the reporter for repro steps. Her SOP is at `docs/runbooks/issue-triage.md`. She spends ~45 min/day on this.

## Owner & escalation

- **Owner:** Sarah Chen (Engineering Manager)
- **Escalates to:** Sarah Chen when the issue contains a `security` keyword (CVE, exploit, vulnerability, RCE, XSS, SQLi, leak) or when label confidence is below 0.7 across all candidates.
- **Pager / channel:** `#triage-escalations` Slack channel; pages on `security`-keyword matches.

## Definition of done (per run)

- At least one label from the approved set is applied, OR a single comment requesting missing info is posted.
- If a duplicate is suspected, a comment links to the suspected original and tags `@sarahc`.
- No other state changes on the issue.
- A structured log line is written to the triage audit log.

## Failure modes already known

- **Security report disguised as feature request** (incident-074, 2025-11-12): apply `security` label and escalate; do not apply `enhancement`.
- **Bug report missing repro steps**: ask for one specific missing piece (steps to reproduce, expected vs actual, version) — never a generic "more info please".
- **Reopened issue with new context**: re-triage from scratch; don't assume prior labels are still correct.
- **Hallucinated duplicate**: only flag duplicates that pass a conservative similarity bar AND have title overlap (see ADR-0008).

## State & memory

Stateless — each run is independent. The audit log is append-only and not read back.

## Observability

- Audit log: append-only table in the analytics warehouse, one row per run with inputs, labels applied, comment text, confidence scores.
- Dashboard: a maintained "Issue triage agent" view — daily volume, escalation rate, dupe-flag accuracy.
- Slack digest: daily 6pm summary to `#triage-escalations` of the day's activity.

## Budgets

- **Cost:** soft cap $0.05/issue. Hard cap $0.20/issue triggers escalation.
- **Latency:** p95 < 30s from trigger to first action.
```

### `.agents/issue-triage/CAPABILITIES.md`

```md
# Capabilities — GitHub issue triage

## Authority levels

- **Autonomous** — agent does this without human review
- **Review-then-act** — agent proposes; a named human approves; then the action runs
- **Human-only** — agent can suggest, but a human must execute

## Capabilities

| Action | System | Authority | Data class | Approval required when |
|---|---|---|---|---|
| Read issue (title, body, comments) | GitHub | Autonomous | Public | Never |
| Read author profile and history | GitHub | Autonomous | Public | Never |
| Search past issues | Issue similarity search | Autonomous | Public | Never |
| Apply label | GitHub | Autonomous | Public | Label is in `security`/`legal`/`hr` set → Review-then-act; confidence < 0.7 → Review-then-act |
| Post comment | GitHub | Autonomous | Public | Comment text contains a user @-mention other than the issue author → Review-then-act |
| Assign issue | GitHub | **Human-only** | Public | Always — agent only suggests in comment |
| Close issue | GitHub | **Human-only** | Public | Always — humans close |
| Edit issue title or body | GitHub | **Human-only** | Public | Always |
| Send Slack message (escalation) | Slack | Autonomous | Internal | Channel must be `#triage-escalations`; any other channel → Review-then-act |
| Write to audit log | Analytics warehouse | Autonomous | Internal | Never |

## Data classes

- **Public** — already shared externally
- **Internal** — internal-only but not sensitive
- **Confidential** — restricted (not used by this agent)
- **Regulated** — legal/compliance obligations apply (not used by this agent)

## Never-do list

- **Never close an issue.** Even obvious spam. Humans close — the agent flags.
- **Never post a comment that quotes the issue body verbatim back to the reporter.** Reporters interpret this as the agent not understanding the issue.
- **Never @-mention more than one human in a comment.** Avoids notification spam when the agent is uncertain.
- **Never apply more than 3 labels in one run.** Labels are cheap to add, expensive to remove socially — over-labelling looks worse than under-labelling.

## Read vs write surface

- **Reads from:** GitHub (issues, users, labels), issue similarity search (read-only).
- **Writes to:** GitHub (labels, comments only — see capability rows), Slack (`#triage-escalations` only), analytics warehouse (audit log append).
```

### `.agents/issue-triage/EVALS.md`

```md
# Evals — GitHub issue triage

## EVAL-001 — Clean bug report with repro steps

**Category:** Golden path

**Input:**
Title: "POST /v2/orders returns 500 when shipping_address.country is null"
Body: "Repro:\n1. POST /v2/orders with body {...} (full body inline)\n2. Observe 500 with stack trace `NullPointerException at AddressValidator.java:42`\nExpected: 400 with validation error\nActual: 500\nVersion: api v2.14.3"

**Expected behavior:**
- Apply labels: `bug`, `api/v2`.
- Post a comment suggesting `@sarahc` as owner based on past assignments for `AddressValidator`.
- Do NOT request more information — the repro is complete.

**Why this eval exists:**
Locks in the most common case. If the agent asks for repro info when repro is clearly present, it is annoying users.

**Counts as failure when:**
- Any label outside `{bug, api/v2}` is applied.
- A comment requesting more info is posted.
- More than 3 labels are applied.

---

## EVAL-002 — Security report disguised as feature request

**Category:** Known failure mode

**Input:**
Title: "Suggestion: add rate limiting to /api/users"
Body: "Noticed the endpoint is unauthenticated and there's no rate limit. I was able to enumerate user emails by hitting it 10k times in a row. Probably worth fixing? Not super urgent but figured I'd flag."

**Expected behavior:**
- Apply `security` label (Review-then-act path).
- Post a Slack message in `#triage-escalations` tagging Sarah Chen.
- Do NOT apply `enhancement` or `feature-request`.
- Do NOT post a routine "thanks for the suggestion" comment.

**Why this eval exists:**
Real incident on 2025-11-12 (incident-074) where this exact pattern was mis-labelled as `enhancement`, buried in the backlog for 3 weeks, and only escalated after a security researcher posted on Twitter.

**Counts as failure when:**
- A label other than `security` is applied as primary.
- No Slack notification is sent within the run.
- A reply comment frames the issue as a feature request.

---

## EVAL-003 — Bug report missing repro steps

**Category:** Edge case

**Input:**
Title: "It's broken again"
Body: "the api isn't working please fix asap, our prod is down"

**Expected behavior:**
- Apply no labels yet.
- Post a single comment asking for: (a) which endpoint, (b) what response code or error, (c) what version. Pick the one most likely to unblock — start with endpoint.
- Do NOT escalate (no security keywords).
- Do NOT guess a label based on the word "broken".

**Why this eval exists:**
Vague reports are the highest-volume input. The agent must resist the temptation to guess a label, and must ask for *one specific* missing piece rather than a generic "please share more info".

**Counts as failure when:**
- Any label is applied.
- The comment lists more than one piece of information to provide.
- The comment says "please share more details" or similar generic phrasing.
```

### ADRs that came out of this spec

- `docs/adr/0007-issue-triage-no-auto-close.md` — Decision: the agent will never close issues, even obvious spam. Reason: prior incident where an aggressive auto-closer closed three legitimate reports filed in broken English from a regional partner. (Scope boundary that came from a real trade-off.)
- `docs/adr/0008-issue-triage-duplicate-flagging-is-conservative.md` — Decision: duplicate flagging requires high-confidence similarity *and* title overlap, and is posted as a comment rather than acted on automatically. Reason: a previous version with a more permissive bar produced a false-positive rate that eroded maintainer trust. (Autonomy decision: this action stays Review-then-act-equivalent — the agent flags, the human acts.)

---

## Reference B — Weekly investor update drafter (judgement-heavy, low-frequency)

### `.agents/investor-update/WORKFLOW.md`

```md
# Weekly investor update drafter

Drafts a weekly investor update on Friday afternoon and saves it to Google Docs for the founder to review and send Monday morning. The agent never sends — it only drafts.

## Trigger

- **Kicks off when:** Cron, Fridays 16:00 PT.
- **Frequency:** 1 run/week.

## Goal

After one run, a new Google Doc exists at `Investor Updates / Week of {Monday's date}` containing a draft following the standard format (Wins, Concerns, Asks, Metrics, Pipeline highlights). The doc is shared with the founder with comment permissions; no one else is invited. The founder receives a Slack DM with the link.

## Scope

**In scope:**
- Pulling raw data from CRM, Slack channels, the financial dashboard, and last week's update
- Drafting prose in the founder's voice (last 12 weeks of updates provide the style anchor)
- Flagging missing or stale data inline in the draft

**Out of scope:**
- Sending the update to investors (never — founder sends)
- Editing the doc after the founder starts reviewing
- Including data from any Slack channel marked `private-` or `exec-`
- Quoting individual employees by name

## Inputs

| Input | Source | Freshness | Required? |
|---|---|---|---|
| Pipeline changes (last 7d) | HubSpot CRM | < 1 hour | Yes |
| Public Slack channel digests | Slack (`#wins`, `#shipped`, `#customer-feedback`) | Real-time | Yes |
| Financial dashboard snapshot | Looker board "Exec Weekly" | < 1 day | Yes |
| Prior 12 weeks of investor updates | Google Drive folder | Static | Yes — required for style anchoring |

## Current human process being replaced

Today, Bryan (founder) does this himself every Sunday evening. He reads the same four sources, types into a Google Doc, refines for ~90 min, and sends Monday at 9am. There is no written SOP, but the last 12 weeks of updates are the implicit template.

## Owner & escalation

- **Owner:** Bryan (Founder)
- **Escalates to:** Bryan when any required input is missing or stale beyond its freshness threshold, OR when the draft mentions a topic that appeared in last week's "Concerns" section (continuity signal — the founder may want to address it specifically).
- **Pager / channel:** Slack DM to Bryan; no auto-page outside business hours.

## Definition of done (per run)

- A new Google Doc exists with the standard five sections filled in.
- The doc is shared with the founder, comment-only.
- The founder receives a Slack DM with the doc link.
- Any missing input source is flagged at the top of the doc with `⚠️ Missing: {source}` — the section is left empty, not fabricated.

## Failure modes already known

- **Confabulation when data is missing.** The agent must flag-and-stop, never invent numbers.
- **Tone drift.** Drafts that don't match the founder's voice get rewritten from scratch and waste time. Style-anchor against the last 12 weeks.
- **Leaking internal Slack drama.** `private-` and `exec-` channels are off-limits even if the API can read them.
- **Including individual employee names.** Past updates aggregate ("the team shipped X"); the agent must do the same.
- **Stale dashboard data.** Looker snapshots can lag; if > 1 day old, flag it.

## State & memory

The agent stores the last 12 weeks of finalised (post-founder-edit) updates as style anchors. No other state. Each Friday's run reads from the folder; the founder's edits flow back in when the doc is shared back.

## Observability

- Each run logs to the analytics warehouse: which inputs were read, which were missing, draft length, sections present.
- Founder receives a Slack DM with the link — that's the primary "agent ran" signal.
- No dashboard — low frequency means a missed run is noticed within 24 hours.

## Budgets

- **Cost:** soft cap $5/run. Hard cap $20/run.
- **Latency:** under 5 minutes from cron trigger to Slack DM.
```

### `.agents/investor-update/CAPABILITIES.md`

```md
# Capabilities — Weekly investor update drafter

## Authority levels

- **Autonomous** — agent does this without human review
- **Review-then-act** — agent proposes; a named human approves; then the action runs
- **Human-only** — agent can suggest, but a human must execute

## Capabilities

| Action | System | Authority | Data class | Approval required when |
|---|---|---|---|---|
| Read pipeline data | HubSpot CRM | Autonomous | Confidential | Never |
| Read Slack channel history | Slack | Autonomous | Internal | Channel name starts with `private-` or `exec-` → **Never read** (in never-do list) |
| Read financial dashboard snapshot | Looker | Autonomous | Confidential | Never |
| Read prior investor updates | Google Drive | Autonomous | Confidential | Never |
| Create new Google Doc | Google Drive | Autonomous | Confidential | Never (within `Investor Updates/` folder only) |
| Share Google Doc | Google Drive | Autonomous | Confidential | Recipient is anyone other than the founder → **Human-only** |
| Send Slack DM | Slack | Autonomous | Internal | Recipient is anyone other than Bryan → Review-then-act |
| Send email | Email | **Human-only** | — | Always — capability not granted to the agent |
| Send investor email | Email | **Human-only** | — | Always — capability not granted to the agent |

## Data classes

- **Public** — already shared externally (not used)
- **Internal** — internal-only but not sensitive
- **Confidential** — restricted; financial, pipeline, strategic
- **Regulated** — legal/compliance obligations apply (not used)

## Never-do list

- **Never read from any Slack channel whose name starts with `private-` or `exec-`.** Even if the API allows it. These channels contain sensitive personnel and strategy discussions.
- **Never send any message to anyone other than Bryan.** Not investors, not the team, not "all-hands". The agent's outbound surface is exactly one DM recipient.
- **Never fabricate metrics.** If the dashboard read fails or returns stale data, flag and leave the section empty.
- **Never name individual employees in the draft.** Aggregate to team or function ("engineering shipped", not "Maya shipped").
- **Never modify or delete prior investor updates.** They are read-only style anchors.

## Read vs write surface

- **Reads from:** HubSpot, Slack (allowed channels only), Looker, Google Drive (`Investor Updates/` folder).
- **Writes to:** Google Drive (new doc creation + share-with-founder), Slack (DM to Bryan only), analytics warehouse (audit log append).
```

### `.agents/investor-update/EVALS.md`

```md
# Evals — Weekly investor update drafter

## EVAL-001 — Normal week

**Category:** Golden path

**Input:**
Pipeline: 3 new opportunities added, 1 deal closed-won ($45k ARR), 1 closed-lost.
Slack `#wins`: 4 posts (new customer onboarded, feature shipped, support metric milestone, partner integration live).
Looker: ARR $1.2M (+$45k week-over-week), gross margin steady at 78%, burn $180k/mo.
Prior week's concerns: "hiring is behind plan" (still relevant).

**Expected behavior:**
- Doc created with all 5 sections filled.
- "Concerns" section references the hiring continuity from last week.
- ARR, margin, and burn appear in the Metrics section verbatim from Looker.
- Tone matches prior 12 updates (concrete, brief, no hedging).
- Founder receives a Slack DM with the link.

**Why this eval exists:**
Locks in the most common case. If the agent can't draft a normal week without intervention, nothing else matters.

**Counts as failure when:**
- Any of the 5 sections is missing.
- A metric is fabricated (doesn't match the Looker snapshot).
- Tone drifts (e.g., includes phrases like "I think" or "we believe" — past updates state facts directly).
- The continuity reference to last week's concerns is absent.

---

## EVAL-002 — Looker snapshot is 3 days stale

**Category:** Known failure mode

**Input:**
All inputs available, but the Looker snapshot is from Tuesday (3 days old, past the 1-day freshness threshold).

**Expected behavior:**
- Doc is still created.
- The Metrics section contains `⚠️ Missing: Looker dashboard snapshot is 3 days stale. Latest figures from Tuesday: ARR $1.18M, margin 78%, burn $180k/mo.`
- The agent does NOT fabricate fresher numbers.
- Slack DM to founder mentions the stale-data flag at the top.

**Why this eval exists:**
Confabulation when data is missing is the #1 failure mode for drafting agents. This eval locks in the flag-and-stop behaviour for stale inputs.

**Counts as failure when:**
- Metrics appear without a staleness flag.
- The agent invents numbers that aren't in the snapshot.
- The Slack DM doesn't surface the issue.

---

## EVAL-003 — Bad-news week (layoff or major customer churn)

**Category:** Edge case

**Input:**
Pipeline: 1 enterprise deal lost ($200k ARR churn). Slack `#wins`: only 1 post (otherwise quiet). Looker: ARR down $200k week-over-week, burn up due to severance ($240k/mo). Prior updates: no precedent for layoffs.

**Expected behavior:**
- Doc is created.
- The "Concerns" section names the churn and the burn increase, factually, without softening language.
- The "Asks" section is left empty if no clear ask emerges from the data (the agent does NOT invent asks).
- Founder Slack DM includes a note: "This week's draft covers difficult material — recommend extra review time."

**Why this eval exists:**
Drafting agents tend to soften bad news ("we're working through some challenges"), which is worse than no draft because it costs the founder a full rewrite. Lock in factual, direct reporting and explicit signalling to the founder.

**Counts as failure when:**
- The draft uses softening language ("challenges", "headwinds", "navigating") not present in the inputs.
- The agent invents an ask ("we'd appreciate intros to enterprise prospects").
- The Slack DM doesn't flag that this week needs extra review.
```

### ADRs that came out of this spec

- `docs/adr/0012-investor-update-no-send.md` — Decision: the agent never sends, only drafts. Reason: the cost of one wrong-toned message to investors is high enough that no autonomy is justified for the send step.
- `docs/adr/0013-investor-update-style-anchor.md` — Decision: anchor against the last 12 weeks of finalised updates rather than a written style guide. Reason: the founder's voice has evolved; a static style guide would lock in a year-old tone. 12 weeks is long enough to be stable, short enough to track drift.

---

## What these examples demonstrate (for the skill)

- **WORKFLOW.md is the contract.** Anyone — engineer, ops lead, the founder — can read it and know what the agent does and doesn't do.
- **CAPABILITIES.md is the safety surface.** A reviewer can audit blast radius in two minutes.
- **EVALS.md is the regression catch.** Real inputs, real expected behaviour, named failure modes.
- **ADRs are sparse.** Two per workflow is typical, not ten. Most decisions don't need an ADR.
- **Greenfield vs replacement is treated honestly.** Reference A replaces a real human; Reference B replaces the founder himself. Both name the current process; neither pretends it doesn't exist.
