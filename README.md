# Claude Code for Automation Engineers

**How I run a production automation stack (100+ n8n workflows, WhatsApp AI agents, Salesforce, LLM pipelines) with Claude Code as my day-to-day engineering partner, and the guardrails that keep it from breaking things.**

Most Claude Code content is about writing application code in a repo. My work is different: the "codebase" is a live automation platform, every change hits real customers, and much of the work runs through APIs and MCP servers instead of files.

This repo covers the operating model, the configuration and the hard-won rules that make that safe.

---

## The operating model in one picture

```
                ┌──────────────────────────────┐
                │  CLAUDE.md  (the constitution) │  stable facts + non-negotiable rules
                └──────────────┬───────────────┘
                               │
   ┌───────────────┐   ┌───────▼────────┐   ┌──────────────────────┐
   │  Memory        │◄─►│  Claude Code   │◄─►│  Skills & commands    │
   │  (institutional│   │  (the engineer)│   │  (proven recipes)     │
   │   knowledge)   │   └───────┬────────┘   └──────────────────────┘
   └───────────────┘           │
                     ┌─────────▼──────────┐
                     │  MCP servers + REST │  the hands: n8n, CRM, messaging,
                     └────────────────────┘  observability, CMS, cloud
```

- **MCP servers are the hands.** They let Claude Code read and change real systems.
- **Skills are the decision layer.** A fixed, proven procedure over those hands, so the work is consistent instead of improvised each time.
- **Memory is institutional knowledge.** Every incident, gotcha and correction is written down once and recalled automatically.
- **CLAUDE.md is the constitution.** The facts and rules that apply to every session.

---

## Contents

| # | Chapter | What's inside |
|---|---|---|
| 1 | [The Operating Model](docs/01-operating-model.md) | MCP first, REST on first failure, what stays human |
| 2 | [Writing CLAUDE.md](docs/02-claude-md.md) | What belongs there and what doesn't, with a section-by-section template |
| 3 | [A Memory System That Compounds](docs/03-memory-system.md) | One fact per file, an index, sub-indexes, and Why / How-to-apply |
| 4 | [Skills: Turning Incidents into Procedures](docs/04-skills.md) | When to write a skill, its anatomy, the "traps first" pattern |
| 5 | [Guardrails for Production Work](docs/05-guardrails.md) | Definition of done, assumption ledgers, never test on real customers |

### Examples (generalised, ready to adapt)

| File | What it is |
|---|---|
| [`examples/CLAUDE.md.template`](examples/CLAUDE.md.template) | A CLAUDE.md skeleton for an automation team |
| [`examples/skills/safe-deploy/SKILL.md`](examples/skills/safe-deploy/SKILL.md) | Backup → deactivate → PUT → activate → verify for n8n |
| [`examples/skills/live-e2e/SKILL.md`](examples/skills/live-e2e/SKILL.md) | Live end-to-end testing of a chatbot with fake identities |
| [`examples/commands/audit-workflow.md`](examples/commands/audit-workflow.md) | A slash command that audits a workflow against known bug patterns |
| [`examples/memory/`](examples/memory/) | A sample memory index and memory files |

---

## Five principles

1. **A tool's success message isn't evidence.** Done means a fresh read proves the change landed and a live run proves it works.
2. **Every incident becomes a memory, and every repeated procedure becomes a skill.** You pay for a lesson once.
3. **Audit, then confirm, then build.** The agent proposes and the human says GO before anything changes in production.
4. **Follow the proven recipe exactly.** The task that "looks trivial" is where improvisation does the most damage.
5. **Real customers are never test subjects.** Fake identities, live systems, then clean up.

---

## About

Written by **Ido Guy**, AI & Automation Engineer. I design and run production AI agents and automation systems for sales and customer-service teams.

[LinkedIn](https://www.linkedin.com/in/ido-guy) · Companion repo: [n8n Production Playbook](https://github.com/123idog/n8n-production-playbook)

*All examples are generalised. No client data, credentials, prompts or identifiers are included.*
