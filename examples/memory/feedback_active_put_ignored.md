---
name: feedback_active_put_ignored
description: n8n public API silently ignores node changes while a workflow is ACTIVE — deactivate before PUT
metadata:
  type: feedback
---

`PUT /workflows/{id}` returns **200 but discards node and parameter changes when the workflow is
active**. The write appears to succeed, and a later GET shows the old nodes.

**Why:** cost a whole debugging cycle. Several "fixes" in a row all seemed to revert, because the
workflow was active during every PUT.

**How to apply:** always `deactivate → PUT → activate → GET-verify`. Prove the write path before
deactivating ([[feedback_prove_write_path]]). Send only `{name, nodes, connections, settings}` with
settings whitelisted ([[feedback_settings_whitelist]]).
