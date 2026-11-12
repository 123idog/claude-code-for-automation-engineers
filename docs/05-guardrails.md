# 5. Guardrails for Production Work

An agent with production access is only as safe as the rules it follows when things are urgent. Every rule here was written after an incident.

---

## Audit → confirm → build

When asked to "improve" or "duplicate and fix" something, the agent:
1. **Audits** and lists bugs and proposed improvements.
2. **Waits for an explicit GO.**
3. **Builds**, as a new version (`v2`), never overwriting the original.

This keeps the human in charge of *what* changes and lets the agent do the *how*.

---

## The definition of done

A change counts as done only when it's **proven live**: saved (verified by a fresh read), state restored, live end-to-end run, evidence in the report, test data removed. A tool's success message, a replay, or a dry run alone never counts.

---

## The assumption ledger

Before declaring a root cause, write down every assumption it depends on:

| Assumption | Status | Evidence |
|---|---|---|
| The workflow was deleted | ❌ UNVERIFIED | it was archived, not deleted |
| The conversation is muted permanently | ❌ UNVERIFIED | mutes auto-expire after a few hours |
| It's a platform bug | ❌ UNVERIFIED | the real cause was a file-size limit in a sync tool |

Each of those was a confident wrong diagnosis. Each one would have been caught by marking the assumption UNVERIFIED and asking.

**A zero or a perfect percentage is a bug signature.** Before trusting a measurement, make it flag a known-bad case.

---

## Never test on real customers

When a live bot has left a real customer stuck, the urge is to re-trigger it with their number "to see if the fix works". **Every attempt sends a real message.** One incident sent the same "one moment, checking…" filler to a waiting customer three times.

**Rule:**
- Verify fixes on **fake identities** against the live system.
- If a real customer is stuck, **a human answers them first.** That also mutes the bot.
- Say this rule out loud before starting recovery, because time pressure is exactly when it gets broken.

---

## Prove the write path before you take anything down

Deploying to a live workflow requires deactivating it first. In one incident the deactivate was permitted, and then the PUT and the re-activate were refused by a permission check. The bot stayed offline until the human ran the script.

**Rule:** prove the *whole* write path (for example, a no-op PUT) before deactivating. If a write fails mid-deploy, **restore service first and report second**, and say plainly that the outage was self-inflicted.

---

## Diff live before overwriting

Other sessions, and other people, edit the same live resources. Before replacing a prompt or code field, fetch the live version and compare it with your last deployed version. If they differ, patch. Don't replace.

---

## Encoding is a production risk

For RTL languages (Hebrew, Arabic), some shells corrupt multibyte characters on round-trip. A single broken letter in a live prompt ships to every customer. Upload with scripts that build bytes explicitly, and verify after every deploy that the replacement character `�` count is zero.

---

## Never hard-delete, never trust defaults

- **Archive instead of delete.** Some platforms keep in-memory state keyed by resource ID, and deleting a recently used resource can break unrelated features.
- **Explicit limits on every query**, and state "retrieved N of M" before any analysis.
- **Create new resources inactive**, and let the human switch them on.

---

## Approval doesn't carry over

Approval to send one campaign, delete one table or restart one server **doesn't extend** to the next one. Destructive and customer-facing actions get confirmed every time.

---

## Honest reporting

- If a test failed, say so and include the output.
- If a step was skipped or couldn't be tested (for example, a branch that can't be reached with a fake identity), **say that explicitly** instead of implying it passed.
- If the agent caused the problem, say so.

An agent whose reports you can trust saves more time than one that's never wrong.
