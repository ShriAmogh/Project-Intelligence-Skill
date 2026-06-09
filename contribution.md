# Contributing to project-intelligence

The standard improves through real-world use. If you used `generate_intelligence_prompt.md` on a real project and found something missing, wrong, or improvable — this is how to contribute.

---

## What we want

- **Missing gotcha patterns** — patterns the generator prompt doesn't instruct the agent to scan for, that caused a real problem
- **Missing section types** — information that a coding agent needed but the file had no home for
- **Domain module gaps** — project types where the domain module section was incomplete or wrong
- **Generator prompt failures** — cases where the IDE AI produced a weak or incomplete file despite following the prompt
- **Self-check gaps** — items that passed Phase 4 self-check but still produced an incomplete file

## What we don't want

- Human-readability improvements — this file is for agents, not humans
- Prose additions — the format is intentionally terse; don't add explanatory paragraphs
- Project-specific additions — contributions must be generic enough to apply to any project of that type

---

## How to contribute

1. Fork the repo
2. Edit `generate_intelligence_prompt.md` or the template inside it
3. Test it — run it on a real project and verify the output passes all Phase 4 checks
4. Open a PR with:
   - What was missing or wrong
   - What project type and real-world scenario exposed it
   - What you changed and why it's generic enough to be in the standard

---

## Reporting issues

Open an issue with:
- Project type (web-app / agentic-ai / etc.)
- Which section was incomplete or missing
- What the agent needed but couldn't find
- Ideally: the fragment of `project_intelligence.md` that was wrong or missing