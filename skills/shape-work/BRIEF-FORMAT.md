# BRIEF.md Format

The brief is the hand-off document. An engineer who has never spoken to the domain expert should be able to read it and understand the problem and what the solution needs to do. It is written in plain, domain language and contains **no technical decisions and no specification** — no databases, frameworks, schemas, or screen-by-screen designs. It says *what* and *why*, never *how*.

Terms in **bold** refer to entries in `GLOSSARY.md`.

## Structure

```md
# Brief — {Name of the app, feature, or workflow}

## Problem

{What's wrong today, concretely. What someone does now and where it breaks,
costs time, or causes errors. Two or three sentences, not a wall of text.}

## Who it affects

- **{Role}** — {how this problem touches them}
- **{Role}** — {how this problem touches them}

## Cost of doing nothing

{Why this is worth solving now. What keeps happening if it stays as-is.}

## Desired outcome

{What "good" looks like once this exists, and how we'd know it worked —
in terms of the work and the people, not metrics dashboards.}

## The solution

{A plain-language walk-through of what the thing does, told as the story of
someone using it. Domain language only. Describe behaviour and rules, not
implementation.}

### Rules

- {A business rule the solution must honour}
- {Another rule, ideally drawn out by a concrete scenario during the interview}

## Scope

**In:**
- {What this will do}

**Out (for now):**
- {What this explicitly will NOT do — often the most valuable section}

## Open questions for engineering

- {Something unresolved, or an assumption an engineer should confirm}
- {A place where the domain expert deferred a "how" decision to the team}
```

## Rules

- **Plain language, domain words.** Write the way the expert talks about the work. No jargon the engineers would have to be the source of.
- **Behaviour and rules, not build.** If a sentence describes storage, APIs, frameworks, or UI layout, it doesn't belong — restate it as the need behind it.
- **The "Out" list earns its keep.** Recording what's deliberately excluded prevents scope creep and saves the most argument later. Be generous here.
- **Capture rules from scenarios.** The strongest entries under "Rules" come from concrete edge-case scenarios worked through during the interview ("what happens when two people claim the same job?").
- **Open questions are a feature, not a failure.** A non-engineer will have gaps. Listing them honestly is more useful than papering over them.
- **Keep it short.** A reader should be able to absorb the whole brief in a few minutes. Tighten as it grows.
