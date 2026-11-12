---
name: feedback_never_test_on_real_customers
description: Never re-trigger a live bot with a real customer's number to test a fix — verify on a fake identity, hand the customer to a human
metadata:
  type: feedback
---

When a live bot has left a real customer stuck, do NOT re-trigger the webhook with that customer's
number to check whether a fix worked. Every attempt sends a real message. In one incident the same
"one moment, checking…" filler reached a waiting customer three times before the bug was understood.

**Why:** a recovery attempt looks exactly like the bot malfunctioning again. The customer sees
repetition and neglect, and nothing is proven that a fake identity couldn't have proven.

**How to apply:**
- Diagnose and verify on a fake identity. Plumbing, tool calls and reply text are all observable there.
- If a real customer is stuck, the first action is to ask a human agent to reply. That also mutes
  the bot, so the loop closes for the customer.
- State this rule explicitly before starting any recovery. Time pressure is when it gets broken.

Related: [[feedback_definition_of_done]], [[skill_live_e2e]]
