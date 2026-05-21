# MODEL.md Format

The conceptual domain model: the important things in the domain, the meaningful facts each one carries, the states it moves through over its life, and the rules that must always hold. It is written in plain domain language for engineers to build from. It is **not** a database schema or a specification — no tables, columns, data types, formats, APIs, or storage. It describes the *shape of the domain*, which is knowledge the domain expert owns; engineers decide how to represent it.

Terms in **bold** refer to entries in `GLOSSARY.md` — reference them, don't redefine them here.

## What goes where

- **GLOSSARY.md** — what the words *mean* (definitions, preferred terms, resolved ambiguities) and simple cardinality.
- **MODEL.md** — how the concepts are *structured*: the facts each carries, the states it moves through, ownership, and the invariants.
- **BRIEF.md** — the problem and the solution narrative, and higher-level business rules.

The litmus test for this file: **"a Job has a due date" belongs here; "due_date is a nullable timestamp" does not.** The first is domain knowledge; the second is an engineering decision. The moment you're naming a type, a format, or a column, you've crossed the line — pull back to the plain fact.

## Structure

```md
# Domain Model — {Name of the app, feature, or workflow}

## Concepts

**Job**
- Carries: a **Customer**, a description of the work, a due date, an assigned **Reviewer** (once claimed)
- States: open → claimed → submitted → closed (see Lifecycles)

**Submission**
- Carries: which **Job** it belongs to, who submitted it, when, the work itself
- States: draft → submitted → approved / rejected

**Reviewer**
- Carries: a name, the **Jobs** they're currently assigned
- States: (none — a Reviewer doesn't move through a lifecycle)

## Lifecycles

For each concept that changes over time, the states and what moves it between them:

**Job**
- open → claimed — a **Reviewer** picks it up
- claimed → submitted — the **Reviewer** completes and submits the work
- submitted → closed — the work is accepted and delivered
- claimed → open — the **Reviewer** releases it before submitting

## Relationships (structural)

Beyond the simple cardinality in the glossary, note ownership and identity where it matters:

- A **Submission** cannot exist without a **Job** — it is owned by the Job
- A **Reviewer** is referenced by a **Job** but exists independently

## Invariants — things that must always be true

- A **Job** has at most one assigned **Reviewer** at a time
- A **Submission**'s **Job** never changes once it's created
- A **Job** cannot be closed while it has an un-reviewed **Submission**
```

## Rules

- **Facts, not fields.** "Carries: a due date" is right. "due_date: Date" is not. If you're naming a type or format, stop.
- **Draw states from the question "what can happen to this thing over its life?"** Many concepts have a lifecycle the expert can describe easily once asked. Some (like a person or a place) have none — that's fine, omit the states line.
- **Invariants are the rules engineers must enforce.** Capture the structural ones here (about states, ownership, counts). Broader policy rules can stay in `BRIEF.md` — cross-reference rather than duplicate, and never let the two contradict.
- **Reference the glossary; don't redefine.** Bold a term to point at its glossary entry.
- **Keep it lean.** A concept with three or four meaningful facts and a clear lifecycle is far more useful than an exhaustive list of every detail.
