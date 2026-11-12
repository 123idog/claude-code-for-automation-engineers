---
name: safe-deploy
description: Safely apply and verify a change to an existing n8n workflow — backup, deactivate, PUT via REST, reactivate, and GET-verify that the change actually landed. Use whenever editing an existing workflow's nodes, parameters, prompts or connections, or when the user says "deploy" / "update the workflow". Do NOT use for creating a brand-new workflow.
---

# Safe deploy — the verified patch procedure

Applying a change to a live n8n workflow has several failure modes that **report success while doing
nothing**. This procedure defeats them.

**Never** report a change as done because a tool returned 200. Done means a fresh GET proves it.

## The traps

1. **PUT to an ACTIVE workflow is discarded.** It returns 200, and a later GET shows the OLD nodes.
2. **Unknown `settings` keys break the public API** (`settings must NOT have additional properties`).
3. **Some MCP partial-update operations silently drop connections**, or refuse to save because of
   unrelated orphan nodes. → On the first MCP failure, switch to REST.
4. **Some shells corrupt non-ASCII (RTL) text** on round-trip. → Build request bodies with Node.
5. **Another session may have edited the live workflow** since your local copy was made.

## Inputs

Workflow ID and a description of the change. If either is unclear, ask and stop.

## Step 1 — Fetch, back up, diff

```bash
curl -s -H "X-N8N-API-KEY: $N8N_API_KEY" "$N8N_API_URL/workflows/<ID>" -o "$SCRATCH/wf_<ID>_before.json"
```

Record `name`, `active` (**restore this exact state at the end**) and the node count.
If you deployed to this workflow before, diff the fields you're about to change against your last
deployed version. If they differ, someone else changed them: **patch, don't replace.**

## Step 2 — Prove the write path

Before deactivating anything, confirm writes are permitted (for example, PUT the unchanged body).
If that's refused, stop. Never take a live workflow down without a proven way to bring it back.

## Step 3 — Edit locally

Apply the change with a Node script to `wf_<ID>_after.json`. Touch the minimum number of nodes. Never
rewrite `connections` unless the change requires it. Wire any new node in the same script.

## Step 4 — Build the PUT body (whitelist)

```js
const fs = require('fs');
const wf = JSON.parse(fs.readFileSync(process.env.SCRATCH + '/wf_after.json', 'utf8'));
const ALLOWED = ['executionOrder','timezone','saveDataErrorExecution','saveDataSuccessExecution',
                 'saveManualExecutions','saveExecutionProgress','executionTimeout','errorWorkflow'];
const settings = Object.fromEntries(Object.entries(wf.settings || {}).filter(([k]) => ALLOWED.includes(k)));
const body = { name: wf.name, nodes: wf.nodes, connections: wf.connections, settings };
fs.writeFileSync(process.env.SCRATCH + '/put_body.json', Buffer.from(JSON.stringify(body), 'utf8'));
```

## Step 5 — Deploy

```bash
curl -s -X POST -H "X-N8N-API-KEY: $N8N_API_KEY" "$N8N_API_URL/workflows/<ID>/deactivate"   # only if active
curl -s -X PUT  -H "X-N8N-API-KEY: $N8N_API_KEY" -H "Content-Type: application/json" \
     --data-binary "@$SCRATCH/put_body.json" "$N8N_API_URL/workflows/<ID>"
curl -s -X POST -H "X-N8N-API-KEY: $N8N_API_KEY" "$N8N_API_URL/workflows/<ID>/activate"     # only if it WAS active
```

If any write fails after deactivation: **re-activate first, report second.**

⚠️ Every deploy wipes in-memory chat sessions. For customer-facing bots, deploy in quiet hours.

## Step 6 — Verify (non-negotiable)

Choose a concrete assertion **before** deploying (a changed string, a new node name, a parameter
value). Then re-fetch and check:
- the assertion is present
- `active` matches the original state
- the node count is as expected
- every touched node's connections are intact
- for RTL text, the count of `�` is 0

If the assertion is missing, the PUT was discarded. Confirm deactivation and retry **once**. If it
fails again, stop and report.

## Step 7 — Report

- Workflow name + ID, active state before → after
- What changed (node names + fields)
- **The verification evidence**
- Backup file path
- For chatbots: a live E2E (`/live-e2e`) is still required before the fix is done
