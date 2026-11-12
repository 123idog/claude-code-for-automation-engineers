---
name: live-e2e
description: Run a real end-to-end test on a LIVE chatbot workflow using fake identities — seed the CRM record and state row, drive the webhook multi-turn, read the reply from executions, verify the downstream record, then clean up. Use after ANY change to a chatbot, or when the user says "run E2E" / "test the bot".
---

# Live bot E2E

**Rule:** after any change to a chatbot, run a real E2E **on the live bot** with fake identities.
Unit tests and single-node runs prove the pieces. Only a live run proves the wired flow: triggers,
memory, node references, routing and CRM writes.

Fake identities are safe because the messaging provider can't deliver to them, so no real customer is
ever messaged. **Never use a real customer's number to test a fix.**

## Step 0 — Identify the target

Fetch the workflow and read the real webhook path, state-table ID and agent node name from it.
**Never guess a webhook path.** Confirm the bot's current `active` state.

## Step 1 — Seed the preconditions

Most bots *look up* a CRM record and a state row and don't create them. Without both, the bot can
chat happily while every downstream node receives zero items.

1. **CRM record** with the fake phone, tagged unmistakably (e.g. `Company = 'E2E TEST - DELETE ME'`).
2. **State row** in the bot's table for the fake phone.

Use a reserved fake range (e.g. `+1 555 0100 XX`) and a **distinct fake name**, so you can tell which
data source the bot read.

⚠️ **ID reuse trap:** if memory is keyed by a row ID and deleted IDs get reused, a fresh test row can
inherit an old conversation. Insert new rows first and clean up at the very end. Check that turn 1's
reply is the expected opening before trusting anything after it.

## Step 2 — Drive the bot

POST provider-shaped payloads to the live webhook, one per turn, with the same conversation ID
throughout. Match the exact inbound shape (status strings are often case-sensitive and route
differently). Build non-ASCII payloads with Node, not the shell.

## Step 3 — Read the reply

Sends to fake numbers fail, so read the reply from the execution data:

```
GET /executions?workflowId=<ID>&includeData=true&limit=<N>
→ runData["<Agent node>"][0].data.main[0][0].json.output
```

Always set an explicit `limit`. Running executions may be hidden from the list, so probe IDs directly.

## Step 4 — Cover multiple scenarios

Not just the happy path. One scenario per branch: qualified, unqualified, below threshold, off-script
→ human handoff, **plus the branch your change actually touches.**

## Step 5 — Verify the record, not the chat

A nice reply isn't a pass. Query the CRM record, state row and created tasks **fresh**, and compare
them with the expected values.

## Step 6 — Clean up and report

Delete the test records, verified by a per-key query, not by scanning a page.

Report:
- Scenarios run, pass/fail each, with **execution IDs**
- The resulting CRM/state values that prove it
- Anything that **couldn't** be tested with a fake identity, stated plainly
- Confirmation that the test data was deleted

Never report success from a replay or a dry run.
