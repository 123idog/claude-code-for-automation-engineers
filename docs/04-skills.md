# 4. Skills: Turning Incidents into Procedures

Memory stores **facts**. Skills store **procedures**. A skill is a markdown file the agent loads when a task matches its description, and from then on it follows that procedure instead of improvising.

## When to write a skill

- **The same multi-step task has been done about 3 times**, with the same rules each time.
- **The agent keeps re-deriving the same constraints** from memory at the start of a task.
- **A mistake in the task is expensive**: production deploys, customer-facing tests, publishing.

I asked my agent to **flag these candidates proactively**: when it notices a repeating pattern, it names the candidate skill and the rules it would lock in, and asks whether to build it.

## Anatomy of a good skill

```markdown
---
name: safe-deploy
description: Safely apply and verify a change to a live workflow — backup, deactivate,
  PUT, reactivate, GET-verify. Use whenever editing an existing workflow's nodes,
  prompts or connections, or when the user says "deploy".
---

# Title — one line on what it guarantees

## The traps (all production-confirmed)      ← why this skill exists
## Inputs                                     ← what to ask for, when to stop
## Step 1 … Step N                            ← exact commands, exact order
## Verify (non-negotiable)                    ← the evidence required
## Report                                     ← what the final message must contain
```

### 1. The description is the trigger

The agent decides whether to load a skill from its `description`. Include **when** to use it, the **phrases** users actually say (in every language they use), and **when not** to use it.

### 2. Traps first

Open with the failure modes the skill defeats, each one production-confirmed. It explains *why* every step is there, so the agent doesn't "optimise" a step away.

### 3. Exact commands, not intentions

"Back up the workflow" leaves room for improvisation. A specific `curl … -o backup.json` with a named path doesn't.

### 4. Decide the verification before acting

Make the skill require a **concrete, checkable assertion chosen before the change**: a string that must appear, a node count, a field value. Then verify against a fresh read.

### 5. A required report

Spell out what the final message has to include: IDs, before → after state, the evidence, backup location, and anything that couldn't be tested. This turns "done ✅" into an auditable report.

## Skills I run daily (generalised versions in `examples/`)

| Skill | Guarantees |
|---|---|
| **safe-deploy** | A workflow change actually landed, the active state was restored, nothing else was dropped |
| **live-e2e** | A chatbot change works on the live bot, with fake identities, verified against the CRM record, then cleaned up |
| **audit-workflow** (slash command) | Every workflow is checked against the same list of known bug patterns, audit only, no fixes without GO |
| **language-edit** | Copy reads like a native speaker wrote it, without touching tuned prompts |

## Skills chain into a definition of done

`safe-deploy` ends with: *"for chatbots, done is defined by `/live-e2e`"*. Skills referencing each other enforce the full chain (change → verify → live test → report) without relying on anyone remembering it.

## Follow the recipe exactly

The most expensive incident in this setup didn't come from a missing skill. It came from **deviating from a proven one** on a task that looked trivial: an unrequested "hygiene" step was added, a safety guard was dropped, and a website went down.

Rule recorded afterwards: *the smaller the task feels, the more strictly the recipe gets followed.* No extra steps and no "while I'm here" changes, because every extra step adds risk.
