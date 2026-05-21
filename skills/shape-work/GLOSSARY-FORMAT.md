# GLOSSARY.md Format

The glossary captures the shared language of the problem domain: what the important words mean, and how the things they name relate to each other. It is written in plain English for both the domain expert and the engineers who will read it. It contains **no problem narrative, no solution description, and no technical detail** — those live in `BRIEF.md`. It also does **not** hold a concept's facts/attributes or its lifecycle states — those belong in `MODEL.md`. The glossary answers "what does this word mean?"; the model answers "what is this thing made of and how does it change?"

## Structure

```md
# Glossary — {Project or area name}

{One or two sentences: what this set of terms is about.}

## Terms

**Job**:
A unit of work a customer asks us to do, from request through completion.
_Avoid_: task, ticket, order

**Reviewer**:
A staff member who checks a submission before it goes out.
_Avoid_: approver, checker

**Submission**:
The completed work a reviewer evaluates.
_Avoid_: entry, upload

## Relationships

- A **Job** is carried out by one **Reviewer** at a time
- A **Reviewer** evaluates many **Submissions**
- A **Submission** belongs to exactly one **Job**

## Resolved ambiguities

- "account" was used to mean both the **Customer** (the business) and the **User** (the person logging in) — resolved: these are distinct.
```

## Rules

- **Be opinionated.** When several words mean the same thing, pick the best one and list the rest under `_Avoid_`.
- **Define what it IS, not what it does.** One sentence. A noun, not a paragraph.
- **Only include terms specific to this domain.** Everyday words and general software concepts don't belong — only words that carry special meaning here.
- **Show relationships** with the bold term names, and express "one" vs "many" where it's obvious.
- **Flag conflicts explicitly.** When a word was used two ways, record it under "Resolved ambiguities" with the resolution, so the confusion doesn't come back.
- **No implementation.** If a definition starts describing how something is stored or built, it's gone too far — pull it back to meaning.
