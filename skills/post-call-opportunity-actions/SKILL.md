---
name: post-call-opportunity-actions
description: Turns a completed customer call (Gong, Granola, or Zoom) into everything an AE needs before their next call — commitments, follow-up actions, open questions, a proposed CRM update, and a follow-up email draft, every time — plus deal risk/blockers, discovery gaps, and coaching feedback, but only when there's a real signal worth surfacing. Use whenever a sales call, discovery call, demo, or customer meeting has just ended and needs to move into opportunity workflow; whenever someone asks to "process this call," "turn this call into next steps," "draft the follow-up," "what do I owe this account," "what's the risk on this deal," "did I miss anything in discovery," "coach me on this call," or "score this against MEDDPICC"; or whenever wired to fire on a Gong/Granola/Zoom call-end event. Full meeting-to-opportunity pipeline in one skill — the five operational outputs never wait on the three evaluative ones, which never fire without clearing the Delivery Gate below.
---

# Post-Call Opportunity Actions

## Why this skill exists

The failure point isn't that meeting output doesn't exist. Gong has something,
Granola has something, the AE has the relationship context in their head. The
failure point is that none of it reliably becomes opportunity workflow, because
turning it into workflow has always required someone to remember to do it by
hand, right when they're already late to the next call.

This skill removes that step entirely. It runs the moment a call ends, not
when a rep gets around to asking for it. It carries two tiers of output,
handled differently on purpose:

- **Operational (always fires):** commitments, follow-up actions, open
  questions, proposed CRM update, follow-up email draft. Mechanical, low-stakes.
  Getting one wrong costs a rep a few minutes. Every rep gets these five on
  every call, no exceptions — that's what earns the trust that makes people
  actually open the next draft.
- **Evaluative (gated):** deal risk/blockers, discovery gaps, coaching
  feedback. Getting one wrong, or making it feel like unavoidable surveillance
  instead of a tool the rep chose to use, is how you recreate the exact
  problem that made the prior manually-triggered coaching skill go unused —
  it existed, it just never got invoked, because nobody wants to manually ask
  a grading system to grade them. The fix isn't "run it and force everyone to
  see it." It's "run it every time, but only surface it when it clears the
  Delivery Gate, and default to rep-only visibility." See below.

Read `references/net-new-sales-playbook.md` before producing any of the eight
outputs. The motion the deal is in changes what a good next step, CRM update,
risk flag, or discovery gap looks like.

## What this skill produces

**Always (five operational outputs):**

1. **Commitments** — anything the rep or the customer explicitly promised,
   quoted or closely paraphrased, with who owns it and any date mentioned.
2. **Follow-up actions** — concrete next steps, whether or not they were framed
   as a commitment (e.g. "I'll loop in our security team" said by the customer
   is a follow-up action even if no one called it a commitment).
3. **Open questions** — anything asked and left unanswered, or anything the
   rep should have asked and didn't get to. Distinguish the two.
4. **Proposed CRM update** — see CRM Update Rules. A specific field-level
   proposal, not a generic "update Salesforce" note.
5. **Follow-up email draft** — see Email Draft Rules.

**Only if the Delivery Gate clears (three evaluative outputs):**

6. **Deal risk / blockers** — a disqualifier from the playbook's
   Identifiers/Qualifiers/Disqualifiers table that showed up on this call, or
   a qualifier that should be present for this motion and isn't.
7. **Discovery gaps** — questions the playbook flags as diagnostic for this
   motion that never got asked.
8. **Call coaching / quality feedback** — private, coaching-not-surveillance
   framing. If the existing AE Discovery Coaching skill's category structure
   is available in this workspace, defer to it rather than inventing a
   parallel rubric. If not, use the Identifiers/Qualifiers table as the
   backbone: did the rep get the identifier signals needed to bucket the deal
   correctly, and the qualifier signals needed to know if it can win.

## Delivery Gate — outputs 6–8 only, do not fire on every call

Before producing any evaluative output, check:

- Is there at least one disqualifier present, or at least one qualifier
  missing that the motion requires past a reasonable point in the deal
  (roughly: past the second call)?
- Is there a genuine discovery gap — a question the playbook flags as
  diagnostic for this motion that never got asked?

If neither is true, log that the gate didn't clear and stop there for outputs
6–8 — don't manufacture a coaching note to justify running. The five
operational outputs still ship regardless; the gate only governs the
evaluative three. A quiet, correctly-timed flag is worth more than a
ceremonial one on every call.

## Visibility — assumption, not a settled decision

**Default for this build: rep-only.** Outputs 6–8 go to the rep who ran the
call, not automatically to a manager or into a shared channel. Trust has to be
earned before this becomes anything closer to a performance signal. This is an
assumption made to ship a working v1, not a permanent design — if Katy or
Arno want manager visibility, that's a governance decision that should be
made explicitly, not bundled into this skill by default.

## Process

1. **Pull the call.** Transcript or notes from Gong, Granola, or Zoom. If more
   than one source exists for the same call, prefer the one with full
   transcript text over a summary-only source, and note if sources disagree
   on a material point rather than silently picking one.
2. **Pull account/opportunity context.** Current stage, current Motion field
   if set, current Land Scope / Incumbent Disposition / Compelling Event if
   set. If none of these three fields are populated yet and the call contains
   enough to populate them, that is itself a CRM update to propose.
3. **Identify or confirm the deal's motion** using the Structure Gate in
   `references/net-new-sales-playbook.md`. If ambiguous from available
   context, say so rather than guessing.
4. **Extract the five operational outputs.** Ground every commitment and
   follow-up action in something actually said — don't infer a commitment
   from a rep's intention if it wasn't stated. When in doubt, put it in Open
   Questions instead of Commitments.
5. **Walk the Identifiers/Qualifiers/Disqualifiers table** for the deal's
   motion against what actually happened on the call, and apply the Delivery
   Gate. If it clears, produce outputs 6–8 with the specific transcript
   evidence supporting each one — a flag with no transcript evidence attached
   is not something a rep can act on or trust. If it doesn't clear, stop
   there for those three.
6. **Check all outputs against the motion** before finalizing. A proposed next
   step, CRM update, or risk flag that ignores the motion (e.g. proposing a
   full-suite POC plan for a deal scoped as Blueprint Deployment) is a bad
   output even if every fact in it is correct.
7. **Draft, never write.** This skill produces drafts for the rep to review
   and approve with one action. It never writes directly to Salesforce, never
   sends the email, never posts to Slack on its own — deliberate constraint,
   not a missing feature, see Guardrails.

## CRM Update Rules

- Only propose updates to fields the call actually supports. "Stage" changes
  need an explicit signal (a next meeting booked, a POC agreed, a contract
  requested), not vibes.
- If the call surfaces information relevant to Land Scope, Incumbent
  Disposition, or Compelling Event and those fields are unset or stale,
  propose setting them.
- Never propose closing an opportunity, changing owner, or changing amount
  from this skill. Those are Deal Deep Dive / manager-level actions, out of
  scope here.

## Email Draft Rules

- Match the rep's own voice and Honeycomb's brand voice for customer-facing
  drafts, not a generic AI tone. No "I hope this email finds you well," no
  unearned enthusiasm.
- Every commitment mentioned as coming from the rep on the call must appear in
  the email. If the rep committed to something and the draft omits it,
  that's a failure, not a stylistic choice.
- Keep it to what was actually discussed. Don't pitch things that didn't come
  up on the call just because they're relevant to the motion.

## Guardrails

- **No unattended writes.** Every output here is a draft. A human taps to
  approve the CRM update; a human sends the email. Process before automation,
  human-in-the-loop by design.
- **Silence is a valid outcome for individual fields, not for the whole
  skill.** If a call genuinely produced no open questions, say so, don't
  invent one. If the Delivery Gate doesn't clear, say so and leave outputs
  6–8 blank — don't pad them.
- **Source disagreement gets surfaced, not resolved silently.** If Gong and
  Granola capture the same call differently on a material point, flag it in
  the draft rather than picking a version.
- **Every evaluative flag needs a transcript-grounded reason.** "This deal
  feels risky" is not an output; "no VP+ sponsor engaged after two calls on a
  $350K Finding What's New deal, which the playbook flags as a disqualifier"
  is.
- **This skill never scores, files, or sends on its own initiative** beyond
  what's described above — read-only against source systems, advisory,
  rep-facing by default.

## Update the call record — one row per call

- Database: `Opportunity Post-Call Actions` — https://app.notion.com/p/a7bed23291f64c888cf7f4a435666ba5, data source `39bfce03-75ad-40a5-aa69-a6c1d4e3325b`

1. Query the data source for a row where `Call ID` matches this call's Gong or
   Granola call ID.
2. **If a row exists** (e.g. from an earlier partial run), update it — never
   insert a duplicate for a call that already has one.
3. **If no row exists**, create one and populate everything in a single write:
   `Call`, `Call ID`, `Account / Opportunity`, `Call Date`, `Call Source`,
   `Motion Assessed`, the five operational fields, and the three evaluative
   fields (or the gate status if they didn't clear).

Properties:
- `Commitments`, `Follow-Up Actions`, `Open Questions` (rich_text)
- `Proposed CRM Update` (rich_text): field-level, not freeform
- `Follow-Up Email Draft` (rich_text)
- `Deal Risk / Blockers` (rich_text): one line per flag — `<risk/blocker> —
  evidence: "<quote>"`. Only populate if the Delivery Gate cleared.
- `Discovery Gaps` (rich_text): one line per gap — `<missing question or
  signal> — why it's diagnostic for this motion`. Only populate if the gate
  cleared.
- `Coaching Feedback` (rich_text): grounded in transcript evidence. Only
  populate if the gate cleared.
- `Coaching Gate Status` (select): `"Flagged"` if the gate cleared and the
  three evaluative fields are populated, `"Gate Not Cleared"` if this skill
  ran and found nothing worth surfacing — leave those three fields blank in
  that case.
- `Motion Assessed` (select): the motion this skill identified via the
  Structure Gate.

## Log the run

Every run writes one row to the shared Skill Usage Log — this tracks that the
skill executed, separate from what it found, including runs where the
Delivery Gate stops outputs 6–8 entirely.

Get the run timestamp: `TZ=America/Chicago date -Iseconds`.

- Database: `Skill Usage Log` — https://app.notion.com/p/eae2c789d88040029296dfa1bdb5d481, data source `f43c363f-edc7-461e-8fd3-6c835239f247`
- Row properties:
  - `Run` (title): `"post-call-opportunity-actions - <account/opportunity name> - <YYYY-MM-DD>"`
  - `Skill` (select): `"post-call-opportunity-actions"`
  - `date:Run Timestamp:start`: the timestamp above; `date:Run Timestamp:is_datetime`: `1`
  - `Trigger` (select): `"Scheduled"` if fired by the call-end webhook, `"Manual"` if invoked directly
  - `Run By` (people): attribute to the rep who owns the opportunity, not
    whoever wired the automation. Never hardcode a specific person.
  - `Status` (select): `"Success"` whether or not the gate cleared; `"Error"`
    only if the call itself couldn't be read or the motion couldn't be
    identified at all
  - `Notes` (rich_text): one line distinguishing outcome, e.g. `"5 operational
    outputs shipped, gate not cleared, no evaluative flag surfaced"` or `"5
    operational outputs shipped, 1 disqualifier flagged (no VP+ sponsor, FWN
    motion), 1 discovery gap"`

Do both writes before ending the turn regardless of outcome. If either Notion
write fails, say so plainly — the likely first-use cause is that the Claude
connection hasn't been added to the relevant database yet (Notion: open the
database, ••• menu, Connections, add Claude).

## What this skill deliberately does not do

It does not write to Salesforce, does not send email, does not post to Slack,
and does not treat a gated-silent evaluative run as license to invent a flag.
The tiering inside one skill exists so the operational half keeps shipping
unconditionally even on a call where nothing evaluative clears — a merged
skill only works if that guarantee survives the merge.

## Open items before this goes live

- Confirm with Katy/Arno whether the evaluative tier ships rep-only or with
  manager visibility, and whether that's a per-rep opt-in or a blanket
  policy — merging into one skill doesn't resolve this, it just means the
  answer now applies to fields 6–8 within a single Notion row instead of a
  cross-skill visibility split.
- Confirm whether coaching feedback should call the existing AE Discovery
  Coaching and MEDDPICC Scorecard skills directly (composition) or use the
  Identifiers/Qualifiers table as a fallback rubric (current default).
  Composition is preferred — reimplementing scoring logic that already
  exists is exactly the kind of drift this project is trying to eliminate.
