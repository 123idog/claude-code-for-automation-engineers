# 3. A Memory System That Compounds

The biggest long-term multiplier isn't the model. It's **what the model remembers about your systems.** After a few hundred sessions, a structured memory turns into a searchable record of every incident, gotcha and decision.

## Structure: one fact per file, plus an index

```
memory/
├── MEMORY.md                         ← the index, loaded every session (one line per memory)
├── feedback_active_put_ignored.md    ← one fact
├── project_cs_bot_dedup.md           ← one fact
├── reference_crm_queues.md           ← one fact
└── index_cs_bot.md                   ← a sub-index for a large topic
```

Each file has frontmatter and one fact:

```markdown
---
name: feedback_active_put_ignored
description: Workflow API silently ignores node changes while the workflow is ACTIVE
metadata:
  type: feedback
---

PUT /workflows/{id} returns 200 but discards node changes when the workflow is active.

**Why:** cost a full debugging cycle. Every "fix" appeared to revert.
**How to apply:** always deactivate → PUT → activate → GET-verify.

Related: [[feedback_settings_whitelist]]
```

## Four memory types

| Type | Holds | Example |
|---|---|---|
| `user` | Who the user is, role, preferences | "Automation engineer; prefers short answers in their own language" |
| `feedback` | Rules from corrections **and** confirmed approaches | "Never retry a fix on a real customer's number" |
| `project` | Ongoing work, constraints and decisions that aren't in the code | "Bot v6 live since the 26th; open issue: premature unqualify" |
| `reference` | Pointers to external resources | Dashboards, API quirks, which system is the source of truth |

## Why "Why" and "How to apply" matter

A bare rule ("deploy in quiet hours") gets ignored or misapplied. A rule with its **reason** ("each deploy wipes in-memory chat sessions, so active conversations lose context") lets the agent judge edge cases: is this bot one that depends on in-memory sessions? The **How to apply** line turns the lesson into an action.

## The index is the product

`MEMORY.md` loads every session, so it has to be **scannable**. Each line is a link plus a hook that carries the key fact:

```markdown
## Deploying
- [**Active PUT ignored**](feedback_active_put_ignored.md) deactivate → PUT → activate → verify
- [UI save clobbers API edits](feedback_ui_save_clobbers.md) last writer wins, whole document
- [**Prove write access BEFORE deactivating**](feedback_prove_write_path.md)
```

- **Bold** marks the entries that have cost the most when forgotten.
- Group by system (platform, CRM, chatbots, content), not by date.
- When a topic outgrows a few lines, give it a **sub-index file** and keep one line for it in the main index.

## Hygiene rules

- **Update, don't duplicate.** Check for an existing memory before writing a new one.
- **Delete memories that turn out to be wrong.** A wrong memory is worse than none.
- **Convert relative dates to absolute** ("Thursday" → "2026-09-03"). Memories outlive the week they were written in.
- **Verify before recommending.** A memory that names a file, flag or ID reflects what was true when it was written. Check that it still exists.
- **Don't store what the repo or tool already records** (code structure, git history). Store what was *non-obvious*.
- **Link liberally** with `[[name]]`. Related incidents form clusters, and the clusters are where patterns show up.

## What it pays back

Examples from practice:
- A new session about a "silent bot" pulls in, unprompted, the memory that **provider webhook delays look exactly like bot silence**, along with the two queries that tell them apart.
- A deploy session recalls that **this platform's MCP partial-update drops connections** and goes straight to REST.
- An analysis session recalls that **a CRM date field is wrong in about a quarter of records** and uses the source-of-truth system instead.

Each of those cost hours the first time and seconds afterwards.

See [`examples/memory/`](../examples/memory/) for a sample index and files.
