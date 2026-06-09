## Project Intelligence

> `project_intelligence.md` — the gold standard for AI-assisted development.

Stop wasting tokens re-explaining your codebase to AI agents on every new context window.

---

## The Problem

Every time you start a new AI coding session, the agent reads dozens of files just to understand the project — entry points, models, functions, configs, gotchas. That's **10,000–20,000 tokens of exploration before a single line of code is written**. Multiply that across every session, every developer, every feature — and you're burning a significant chunk of your token budget on context reconstruction that could have been avoided.

## The Solution

`project_intelligence.md` is a single, structured, agent-optimised file that lives at the root of your repo. It contains everything a coding agent needs to navigate and modify your codebase:

- Every file's purpose
- Every function's signature and behaviour
- Every data model's fields and constraints
- All prompt templates and their injected variables
- The full call graph
- All runtime output files and their schemas
- Config constants and what breaks if they're wrong
- Gotchas that would otherwise take an agent hours to discover

**First session**: generate it once using `generate_intelligence_prompt.md` — the agent scans your codebase and writes the file. Still cheaper than reading the whole codebase from scratch.

**Every session after**: the agent reads one file instead of the entire codebase.

**After every feature**: the agent appends a structured entry to the Feature Log. The file never goes stale.

---

## Token Savings

| Without `project_intelligence.md` | With `project_intelligence.md` |
|-----------------------------------|-------------------------------|
| Agent reads 30–50 source files | Agent reads 1 file |
| ~15,000 tokens per new session | ~1,500 tokens per new session |
| Context rots as codebase grows | File stays current via Feature Log |
| Every developer pays full cost | One-time generation, reused forever |

---

## Files in this repo

| File | Purpose |
|------|---------|
| `generate_intelligence_prompt.md` | Paste into your IDE AI to generate `project_intelligence.md` for any project |
| `CONTRIBUTING.md` | How to improve the standard |

---

## How to use inside Skill folder for IDE Agent

**Step 1 — Generate**

Copy `generate_intelligence_prompt.md` into the skill folder of your project. Open your IDE (Cursor, Copilot, Windsurf, or any agent) and add the following:

```
## project_intelligence.md — required protocol

- On first run in any project: check if `project_intelligence.md` exists at the repo root.
  If it does not exist, follow `generate_intelligence_prompt.md` and generate it before doing anything else.
- On every new context window or session start: read `project_intelligence.md` fully before
  touching any source file.
- After every feature addition, function change, or architectural decision: append an entry
  to the Feature Log (§12) in `project_intelligence.md` and update any in-place sections
  per the META table at the top of the file.
- Never ask the user to update `project_intelligence.md` — do it autonomously as part of
  completing any coding task.

```

The agent will scan your codebase across 11 steps and write a fully populated file.

**Step 2 — Use**

At the start of every new AI session:

```
Read project_intelligence.md before touching any file.
```

That's it. The agent has full project context in one read.

**Step 3 — Maintain**

After every feature or architectural change, tell your agent:

```
Append an entry to the Feature Log in project_intelligence.md for what we just built.
```

The META table at the top of every `project_intelligence.md` tells the agent exactly which sections to update for each type of change.

---

## What gets generated

`project_intelligence.md` is structured in 12 sections:

```
§1  Project Identity       — name, type, language, entry points, env vars
§2  File Tree              — every file with a one-line description
§3  Data Models            — every field, type, and constraint
§4  Function Map           — every constructor, method, and function
§5  Output Files           — runtime-generated files and their schemas
§6  Prompt Map             — injected variables, structure, expected output
§7  Call Graph             — full execution tree from every entry point
§8  Config & Environment   — every constant, default, and failure mode
§9  Gotchas                — non-obvious things that cause bugs
§10 Dependencies           — every package, where used, exactly what for
§11 Architecture Decisions — what was chosen, what was rejected, why
§12 Feature Log            — append-only change history
```

---

## Works with any project type

The generator detects your project type and includes only the relevant domain module:

`web-app` · `api-server` · `cli-tool` · `mobile-app` · `agentic-ai` · `ml-pipeline` · `data-pipeline` · `library-sdk` · `desktop-app`

---

## Works with any IDE AI

- Cursor
- GitHub Copilot
- Windsurf
- Any agent that accepts a system prompt or context file

---

## Contributing

See `CONTRIBUTING.md`. The standard improves when people use it on real projects and report what was missing.

---

## License

MIT