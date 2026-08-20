---
name: post-call-opportunity-actions
description: Turns a completed customer call (Gong recording, Granola notes, or Zoom transcript) into the five things an AE actually needs before their next call — commitments made, follow-up actions, open questions, a proposed CRM update, and a follow-up email draft. Use this whenever a sales call, discovery call, demo, or customer meeting has just ended and its output needs to move into opportunity workflow, whenever someone asks to "process this call," "turn this call into next steps," "draft the follow-up," or "what do I owe this account," or whenever this skill is wired to fire automatically on a Gong/Granola call-end event. This is the operational half of the meeting-to-opportunity pipeline — always pair it with deal-signal-coaching's risk and discovery-gap read on the same call, but never block on it; this skill's output should ship to the rep even if the coaching skill has nothing to say. Every run also writes one row to the shared "Skill Usage Log" Notion database, success or failure, so skill usage across the whole system is tracked in one place.
---

# Post-Call Opportunity Actions

## Why this skill exists

The failure point isn't that meeting output doesn't exist. Gong has something,
Granola has something, the AE has the relationship context in their head. The
failure point is that none of it reliably becomes opportunity workflow, because
turning it into workflow has always required someone to remember to do it by
hand, right when they're already late to the next call.

This skill exists to remove that step entirely. It should run the moment a call
ends, not when a rep gets around to asking for it.

## What this skill produces (five outputs, every time)

1. **Commitments** — anything the rep or the customer explicitly promised,
   quoted or closely paraphrased, with who owns it and any date mentioned.
2. **Follow-up actions** — concrete next steps, whether or not they were framed
   as a commitment (e.g. "I'll loop in our security team" said by the customer
   is a follow-up action even if no one called it a commitment).
3. **Open questions** — anything asked and left unanswered, or anything the rep
   should have asked and didn't get to. Distinguish the two.
4. **Proposed CRM update** — see the CRM Update Rules below. This is not a
   generic "update Salesforce" note, it's a specific field-level proposal.
5. **Follow-up email draft** — see the Email Draft Rules below.

Read `references/net-new-sales-playbook.md` before producing outputs 4 and 5.
The motion the deal is in changes what a good next step and a good CRM update
look like.

## Process

1. **Pull the call.** Transcript or notes from Gong, Granola, or Zoom. If more
   than one source exists for the same call, prefer the one with full transcript
   text over a summary-only source, and note if sources disagree on a material
   point (who said what) rather than silently picking one.
2. **Pull account/opportunity context.** Current stage, current Motion field if
   set, current Land Scope / Incumbent Disposition / Compelling Event if set.
   If the opportunity has none of these three fields populated yet and the call
   contains enough to populate them, that is itself a CRM update to propose —
   don't wait for someone to ask.
3. **Extract the five outputs.** Ground every commitment and follow-up action in
   something actually said — do not infer a commitment from a rep's intention if
   it wasn't stated on the call. When in doubt, put it in Open Questions instead
   of Commitments.
4. **Check outputs against the motion.** Before finalizing the CRM update and
   email draft, check `references/net-new-sales-playbook.md`'s
   Identifiers/Qualifiers/Disqualifiers table for the deal's motion. A proposed
   next step that ignores the motion (e.g. proposing a full-suite POC plan for a
   deal that's actually scoped as Blueprint Deployment) is a bad output even if
   every fact in it is correct.
5. **Draft, never write.** This skill produces a draft for the rep to review and
   approve with one action, it never writes directly to Salesforce, never sends
   the email, never posts to Slack on its own. That's a deliberate constraint,
   not a missing feature — see Guardrails.

## CRM Update Rules

- Only propose updates to fields the call actually supports. "Stage" changes
  need an explicit signal (a next meeting booked, a POC agreed, a contract
  requested), not vibes.
- If the call surfaces information relevant to Land Scope, Incumbent
  Disposition, or Compelling Event and those fields are unset or stale,
  propose setting them — these are marked conversation-confirmed in the
  playbook precisely because a transcript is exactly the kind of evidence that
  should confirm them.
- Never propose closing an opportunity, changing owner, or changing amount from
  this skill. Those are Deal Deep Dive / manager-level actions, out of scope
  here.

## Email Draft Rules

- Match Katy's voice standard where this touches AI-ops-adjacent
  communication; match the rep's own voice and Honeycomb's brand voice for
  customer-facing drafts, not a generic AI tone. No "I hope this email finds
  you well," no unearned enthusiasm.
- Every commitment mentioned as coming from the rep on the call must appear in
  the email. If the rep committed to something and the draft omits it, that's
  a failure, not a stylistic choice.
- Keep it to what was actually discussed. Don't pitch things that didn't come
  up on the call just because they're relevant to the motion.

## Guardrails

- **No unattended writes.** Every output here is a draft. A human taps to
  approve the CRM update; a human sends the email. This mirrors how every
  other automation in this system is built — process before automation,
  human-in-the-loop by design, not as an afterthought.
- **Silence is a valid outcome for some fields, not for the whole skill.** If a
  call genuinely produced no open questions, say so, don't invent one. But the
  skill should still run and still produce the other four outputs.
- **Source disagreement gets surfaced, not resolved silently.** If Gong and
  Granola capture the same call differently on a material point, flag it in
  the draft rather than picking a version.

## Step 6. Log the run

Every run writes one row to the shared Skill Usage Log, whether the call
produced five clean outputs or nothing worth drafting, and whether the run
succeeded or failed partway through. This isn't optional and isn't
conditional on a good outcome.

Get the run timestamp: `TZ=America/Chicago date -Iseconds`.

Load Notion tools via tool search if deferred, then write one row:

- Database: https://app.notion.com/p/eae2c789d88040029296dfa1bdb5d481, data source `f43c363f-edc7-461e-8fd3-6c835239f247`
- Row properties:
  - `Run` (title): `"post-call-opportunity-actions - <account/opportunity name> - <YYYY-MM-DD>"`
  - `Skill` (select): `"post-call-opportunity-actions"`
  - `date:Run Timestamp:start`: the timestamp above; `date:Run Timestamp:is_datetime`: `1`
  - `Trigger` (select): `"Scheduled"` if fired by the call-end webhook, `"Manual"` if invoked directly
  - `Run By` (people): resolve the AE the call belongs to via Notion's `fetch` tool with id `"self"` if this is a self-serve invocation, otherwise attribute to the rep who owns the opportunity, not to whoever wired the automation. Never hardcode a specific person.
  - `Status` (select): `"Success"` if the five outputs were produced (even if some were empty, e.g. no open questions), `"Error"` if the call couldn't be pulled or parsed at all
  - `Notes` (rich_text): one line, e.g. `"3 commitments, 1 CRM field update proposed, email drafted"` or `"Call transcript unavailable: <short reason>"`

Do this before ending the turn, not conditionally on how the call went. If the
Notion write itself fails, say so plainly in the draft handoff rather than
silently dropping it — the likely first-use cause is that the Claude
connection hasn't been added to the Skill Usage Log database yet (Notion:
open the database, ••• menu, Connections, add Claude).

## What this skill deliberately does not do

It does not score the call, does not flag deal risk, and does not produce
coaching feedback. That's `deal-signal-coaching`. Keeping them separate is a
trust decision: every rep gets these five outputs on every call, no exceptions,
so this pipeline earns the trust that makes people actually open the next
draft. Coaching and risk are evaluative and are gated differently on purpose —
see that skill for why.
