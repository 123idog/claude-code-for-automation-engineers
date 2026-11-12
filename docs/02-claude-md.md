# 2. Writing CLAUDE.md

`CLAUDE.md` loads into every session. Treat it as a **constitution**: stable facts and non-negotiable rules. It's not a project diary.

## What belongs in it

| Section | Why it earns its place |
|---|---|
| **Who I am and what the business is** | Gives the agent context for judgment calls: who the customers are, which markets, what "premium" means |
| **The stack, one line per layer** | So the agent picks the right tool without asking |
| **Naming conventions** | Workflow names, tags, versioning (`v2`, `v3`, never overwrite) |
| **Time zone and locale rules** | Schedule triggers silently default to the wrong zone |
| **Confirmed resource IDs** | Credential and resource *IDs*, never secrets, so the agent doesn't guess or invent them |
| **Working preferences** | MCP vs. REST, create inactive, preserve prompts verbatim, audit before building |
| **Known platform bugs to check for** | Turns every audit into a checklist |
| **Definition of done** | The single most valuable section (see below) |
| **Debugging rules** | How to reach a root cause without guessing |
| **Data query rules** | Explicit limits and pagination, and "retrieved N of M" before any analysis |
| **Language** | Which language to answer in. Technical content stays English |

## What does NOT belong in it

- **In-flight projects and recent activity.** They go stale in days. Put them in memory.
- **Secrets.** Never.
- **Long incident write-ups.** Keep the rule they produced in CLAUDE.md and the story in memory.
- **Evolving preferences.** Wait until a convention has been used about 3 times before promoting it.

## The definition of done

The section that changed the quality of the work most:

```markdown
## Definition of done

A fix is NOT done until it's proven live:
1. Change saved, verified by a fresh GET, not by the tool's success message
2. Resource reactivated if it was active before
3. Live end-to-end run with a fake identity
4. Execution ID + the resulting record pasted in the report
5. Test data deleted

If any step fails, report the failure. Never report success on a replay or a dry run alone.
```

Without it, an agent reports "fixed ✅" because a tool returned 200. With it, the report carries evidence.

## The debugging rule

```markdown
## Debugging

Before declaring a root cause, list every assumption it depends on and mark each
VERIFIED (with the log line / API response that proves it) or UNVERIFIED.
Ask about the UNVERIFIED ones instead of guessing.
```

It exists because three confident wrong diagnoses in a row each came from one unchecked assumption. See [chapter 5](05-guardrails.md#the-assumption-ledger).

## The "never trust defaults" rule

```markdown
## Data queries

Always set an explicit limit + pagination on every query. Default caps of 50/100
truncate silently. Before analysing anything, state "retrieved N of M available".
If they differ, stop and report.
```

APIs that quietly return the first 50 rows have produced wrong conclusions more than once. This rule makes truncation visible.

## Keep it correct

CLAUDE.md can be wrong, and when it is, every session inherits the error. When a session finds that a "fact" in CLAUDE.md is wrong (a picklist value, an ID, even the user's name), **record the correction in memory immediately and flag it**, then fix CLAUDE.md when the user approves.

See [`examples/CLAUDE.md.template`](../examples/CLAUDE.md.template) for a full skeleton.
