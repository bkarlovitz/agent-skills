---
name: shape-work
description: A guided interview for domain experts and other non-engineers to articulate a problem and a solution clearly enough to hand to an engineering team — building a shared glossary, a problem statement, a plain-language description of the solution, and a conceptual domain model (the key concepts, their facts, states, and rules), while deliberately making NO technical decisions or specifications. Use when a non-engineer wants to shape an app, feature, or workflow; pin down the problem; sharpen terminology; sketch the domain model; or produce a hand-off brief for engineers.
---

<what-to-do>

Interview me one question at a time to draw out the problem I'm trying to solve and the solution I have in mind, until an engineer who has never spoken to me could read the result and understand both. Walk down each branch of the idea, resolving ambiguity before moving on. For each question, offer your recommended answer so I have something to react to rather than a blank page.

Ask one question, then wait for my answer before asking the next. Keep the tone supportive and curious, not adversarial — and when you ask about an edge case or a fine distinction, briefly say *why it matters*, because I may not see why the detail is worth pinning down.

I am a domain expert, not an engineer. Lean on what I know (how the work actually happens, who is involved, what good looks like) and don't ask me to make decisions that belong to engineers.

</what-to-do>

<supporting-info>

## The boundary — model the domain, don't design the build

This is the rule that makes `shape-work` different from a planning or spec session: **never make technical decisions and never write a specification.** Your job is to capture *what* the problem is, *what* the solution should do, and *the shape of the domain* — in the language of the domain, not in the language of the build.

The line is between **conceptual** and **implementation**, and it is *not* the line between "non-technical" and "technical":

- **Capture it — this is domain knowledge the expert owns:** the concepts (the important nouns), the meaningful facts each one carries ("a **Job** has a due date and an assigned **Reviewer**"), the states things move through ("open → claimed → submitted → closed"), how concepts relate, and the rules that must always hold. This *is* the domain model, and modelling it is not a technical decision.
- **Redirect it — this is the engineering team's call:** databases, tables, columns, data types and formats, APIs, frameworks, hosting, libraries, storage, algorithms, screen-by-screen UI layouts.

The litmus test: **"a Job has a due date" is domain (capture it); "due_date is a nullable timestamp" is implementation (redirect).** When you catch the conversation crossing into implementation, pull it back to the fact or requirement underneath:

> "That's a decision the engineering team should make. What I want to capture is the requirement behind it: it sounds like a **Submission** can never outlive its **Job** — is that right?"

The output should leave engineers free to choose *how*, while being completely clear on *what*, *why*, and the *shape of the things involved*.

## What to draw out (your agenda)

A domain expert won't always know what a complete articulation needs to contain, so *you* carry the checklist. Work through these, roughly in order, but follow the energy of the conversation:

1. **The problem.** What's wrong today? Be concrete. What does someone do now, and where does it break, cost time, or cause errors?
2. **Who is affected.** The people and roles involved — who feels the pain, who does the work, who else touches it.
3. **The cost of doing nothing.** Why is this worth solving now? What happens if it stays as-is?
4. **The desired outcome.** What does "good" look like once this exists? How would we know it worked?
5. **The solution, in plain terms.** What the thing does, told as a story or a walk-through — in domain language, not technical language.
6. **The key things involved.** The important nouns in the solution — the *concepts* — and the meaningful facts each one carries ("a **Job** has a customer, a due date, and an assigned **Reviewer**"). Plain facts, not data fields. This is the backbone of the domain model.
7. **How those things change.** For each key thing that has a life, the states it moves through and what moves it ("a **Job** goes open → claimed → submitted → closed"). Draw it out by asking "what can happen to a **Job** from start to finish?" Some concepts have no lifecycle — that's fine.
8. **Scope — in and out.** What this explicitly *will* and *will not* do. The out-of-scope list is often the most valuable thing you produce; capture it deliberately.
9. **Open questions and assumptions.** Anything you couldn't resolve, anything I'm assuming, anything an engineer will need to ask about.

## During the session

### Sharpen fuzzy language

When I use a vague or overloaded word, propose a precise canonical term. "You keep saying 'account' — sometimes you mean the *business* and sometimes the *person logging in*. Those are different things — can we name them separately?" Getting the language right is the single most valuable thing a domain expert can hand an engineer.

### Stress-test with concrete scenarios

When a rule or relationship comes up, invent a specific scenario that probes its edges and forces precision. "Walk me through what happens when two people try to claim the same job at the same time." "What if the customer cancels *after* the work has started but *before* it's finished?" Edge cases are where the real requirements hide.

### Check for internal contradictions

There's no codebase to check against, so check my answers against *each other*. If something I say now conflicts with something I said earlier, surface it: "Earlier you said a reviewer can never see their own submission, but this implies they can edit it later — which is it?"

### Capture the language inline

When a term gets resolved, write it to `GLOSSARY.md` right then — don't batch it. Use the format in [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md). The glossary is a glossary and nothing else: definitions and simple relationships, no problem narrative, no solution description, no implementation. A concept's *facts* and *states* don't go here — they go in `MODEL.md`.

### Build the model inline

As concepts, their facts, and their states surface, record them in `MODEL.md` using the format in [MODEL-FORMAT.md](./MODEL-FORMAT.md). This is the conceptual domain model the engineers will build from — concepts and the facts they carry, lifecycles, structural relationships, and invariants. Keep it conceptual: facts not fields, no schemas or types.

### Build the brief inline

As the problem and solution take shape, write them to `BRIEF.md` as you go, using the format in [BRIEF-FORMAT.md](./BRIEF-FORMAT.md). Update it as understanding sharpens rather than waiting until the end.

## File structure

Create files lazily — only once you have something real to write.

```
/
├── BRIEF.md       ← the problem, the solution, scope, open questions
├── GLOSSARY.md    ← the shared language (definitions + simple relationships)
└── MODEL.md       ← the conceptual domain model (concepts, facts, states, invariants)
```

If any of these already exist, read them first and extend rather than starting over.

## Handing off

When the brief, glossary, and model feel complete, say so, and remind me that these are meant to go to an engineering team — they describe the problem, the solution, and the shape of the domain, and leave the technical decisions to the engineers. If a `to-prd` skill is available, offer to turn the brief into a PRD as the next step.

</supporting-info>
