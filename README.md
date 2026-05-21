# Agent Skills

A small collection of agent skills built on the
[open agent-skills standard](https://agentskills.io). They are **agent-agnostic** — a
skill is just a `SKILL.md` plus reference docs, with nothing tied to a specific tool. They work the same in Claude Code, Cursor, opencode, Codex, and the other agents the
ecosystem supports. Browse the source below, or install any skill with one command.

## Install

The [`skills`](https://github.com/vercel-labs/skills) CLI auto-detects which supported
agents you have installed and sets the skill up for each of them. No global install of the
CLI is required — `npx` fetches it on demand.

```bash
# Install a single skill from this repo
npx skills add bkarlovitz/agent-skills --skill spec-an-agent

# Install every skill in this repo
npx skills add bkarlovitz/agent-skills

# Install to your user directory (available in every project) instead of the current project
npx skills add bkarlovitz/agent-skills --skill spec-an-agent -g

# Target specific agents only (otherwise all detected agents are set up)
npx skills add bkarlovitz/agent-skills --skill spec-an-agent -a claude-code -a cursor
```

### Manual install (no CLI)

Each skill is a self-contained folder under [`skills/`](./skills). To install by hand,
copy the skill folder into your agent's skills directory — the location depends on the
agent:

- **Claude Code:** `<project>/.claude/skills/<skill-name>/` (project) or `~/.claude/skills/<skill-name>/` (all projects)
- **Other agents:** the agent's own skills directory, e.g. `.agents/skills/<skill-name>/` or the path documented by that agent

That's it — drop the folder in and the skill is available.

## Skills

| Skill | What it does |
|---|---|
| <a href="./skills/shape-work"><code>shape&#8209;work</code></a> | A guided interview that helps a domain expert or other non-engineer articulate a problem and solution clearly enough to hand to an engineering team. Asks one question at a time to build a shared glossary, a problem-and-solution brief, and a conceptual domain model (concepts, facts, states, and rules), writing them inline as `GLOSSARY.md`, `BRIEF.md`, and `MODEL.md`. Deliberately makes *no* technical decisions — it captures *what* the problem is and *the shape of the domain*, leaving *how* to build it to the engineers. |
| <a href="./skills/spec-an-agent"><code>spec&#8209;an&#8209;agent</code></a> | A grilling session that stress-tests the *design* of an agentic workflow before it gets built. Interrogates trigger, scope, systems, authority, escalation, failure modes, and evals, then writes the spec inline as `WORKFLOW.md`, `CAPABILITIES.md`, `EVALS.md`, and ADRs. Implementation-agnostic by design — it pins down *what* the agent does and *under what authority*, never *how* it's built. |

## How these skills are structured

Every skill follows the open standard: a folder containing a `SKILL.md` with YAML
frontmatter (`name`, `description`) plus any supporting reference docs the skill loads on
demand. Nothing else is required — no build step, no manifest.

## License

MIT — see [LICENSE](./LICENSE).
