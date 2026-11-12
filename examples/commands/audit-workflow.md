---
description: Audit an n8n workflow against known production bug patterns (audit only, no fixes)
argument-hint: <workflow-id>
allowed-tools: mcp__n8n__n8n_get_workflow, mcp__n8n__n8n_validate_workflow, Read
---

You are auditing the n8n workflow with ID: **$ARGUMENTS**

If no ID was provided, ask for it and stop.

## Step 1 — Fetch
Get the workflow. Note its name, active state and node count.

## Step 2 — Offline validation
Run the validator and capture real errors. Ignore known false positives listed in CLAUDE.md.

## Step 3 — Check each pattern
For every pattern record ✅ PASS, ❌ FAIL (exact node + what's wrong) or ➖ N/A.

1. **Leading `=` in a literal `rightValue`** in Filter/If conditions. It's parsed as a broken
   expression and evaluates to empty, which silently disables the condition.
2. **A publish step reads the wrong upstream node.** For example, it reads the draft stage instead of
   the final edited stage.
3. **Guessing IDs** (e.g. "previous ID + 1" to build a URL). Use the ID returned by the create call.
4. **Retry branches wired to an output the node doesn't have.** That's dead code.
5. **Generating N samples and using only one.** That's N× the cost.
6. **No `retryOnFail` on external-call nodes** (HTTP, CRM, sheets, messaging).
7. **No error branch that releases a claimed item.** Items get stuck "in progress" forever.
8. **A read node downstream of a multi-item node** without `executeOnce` or a collapse step. This is
   item multiplication.
9. **Schedule trigger time zone** is unset or not the company zone.
10. **`$fromAI` parameters described as optional without a default** (4th argument).

## Step 4 — Report

| # | Check | Result | Node(s) | Detail / Fix |
|---|-------|--------|---------|--------------|

Then a 1–2 line verdict: how many real issues there are and which come first.

**Don't fix anything.** End by asking whether to apply fixes. Any fix goes through `/safe-deploy`.
