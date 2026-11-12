# 1. The Operating Model

## Claude Code as the engineer, not the autocomplete

In an automation role, most changes aren't edits to files in a repo. They're changes to **live systems**: a workflow on an n8n server, a field in a CRM, a message template, a webhook subscription. Claude Code works well here because, with MCP servers attached, it can read and change those systems directly. The engineering discipline then has to live in configuration, memory and skills, not only in code review.

A typical session:

1. **I describe the outcome**, in my own language and often loosely ("the bot answered the same customer twice this morning").
2. **Claude Code investigates first:** it reads executions, logs, workflow JSON and CRM records, and states what it found with evidence.
3. **It proposes a fix and waits for GO.** Nothing changes in production on the agent's own initiative.
4. **It applies the fix through a skill** (a fixed procedure), verifies it with a fresh read, and runs a live end-to-end test.
5. **It writes down what was learned** as a memory file, so the next session starts smarter.

---

## MCP first, REST on the first failure

MCP servers are the primary interface: n8n, CRM, messaging, observability, CMS, cloud hosting. They're fast and give the agent structured, validated operations.

They also have failure modes: partial-update tools that drop connections, schema validators that block saves for unrelated reasons, payloads the underlying API rejects. The rule:

> **On the first MCP failure, switch to the direct REST API. Don't retry the same tool.**

Retrying a tool that failed in an unexpected way wastes time at best. At worst it applies half a change. Direct REST, with a script that builds the exact body, is predictable.

---

## Build through the agent, not the UI

New workflows are built by Claude Code through MCP, validated offline, and **created inactive**. In practice:
- A 14-node workflow is built and validated in minutes, and the validator catches structural errors before deploy.
- Every change is reproducible, because the JSON and scripts are files.
- The human reviews in the UI and switches it on.

**What stays human:**
- **Credentials.** They're created in the platform UI, never pasted into chat. The agent only references credential IDs.
- **Activation of new workflows**, after a human review.
- **Anything customer-facing or irreversible:** sending campaigns, deleting data, publishing.
- **Tiny edits** where the UI is genuinely faster.

---

## The cost of an editor that can see everything

An agent with access to production has to be calibrated for **blast radius**:

| Action | Default |
|---|---|
| Read anything (executions, logs, records) | ✅ freely |
| Create new resources | ✅, but **inactive** and clearly named |
| Change an existing live resource | ⚠️ only after GO, through the deploy skill, verified |
| Delete, send to customers, publish | 🛑 explicit confirmation every time, no carry-over approval |

"You approved this last week" isn't approval for today.

---

## Multiple sessions, one production

Running several Claude Code sessions in parallel is powerful. It also means **two agents can edit the same live workflow.** A local copy goes stale without notice, and the platform has no merge: the last writer wins.

Rule: **before overwriting any live field, fetch it and diff it against your own last deployed version.** If it changed, someone else is working there, so apply a minimal patch instead of a replacement.
