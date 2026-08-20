---
name: deal-signal-coaching
description: Reads a completed customer call against Honeycomb's Net New Sales Playbook and surfaces deal risks or blockers, discovery gaps, and call coaching / quality feedback — but only when there's actually a signal worth surfacing, not as a running commentary on every call. Use this whenever someone asks "what's the risk on this deal," "did I miss anything in discovery," "coach me on this call," "score this against MEDDPICC," or whenever this skill is wired to run automatically after post-call-opportunity-actions on the same Gong/Granola/Zoom call. Do not use this to generate a report for every single call by default — check the Delivery Gate below first. This is the evaluative half of the meeting-to-opportunity pipeline; pair it with post-call-opportunity-actions but never let it block that skill's output. This skill updates the same row post-call-opportunity-actions created in the shared "Opportunity Post-Call Actions" Notion database rather than creating a second row per call, and also writes a run record to the shared "Skill Usage Log," including gated-silent runs.
---

# Deal Signal & Coaching

## Why this skill is separate from post-call-opportunity-actions

The other five outputs (commitments, follow-up actions, open questions, CRM
updates, email draft) are mechanical and low-stakes. Getting one wrong costs a
rep a few minutes. These three outputs — deal risk, discovery gaps, coaching
quality — are evaluative. Getting one wrong, or making it feel like unavoidable
surveillance instead of a tool the rep chose to use, is how you recreate the
exact problem that made the existing coaching skill go unused: it already
exists, it's just never triggered, because nobody wants to manually ask a
grading system to grade them.

The fix here isn't "run it automatically and force everyone to see it." It's
"run it automatically, but only surface it when it's actually worth a rep's
attention, and default to rep-only visibility." See Delivery Gate and
Visibility below — both are deliberate, not defaults left unconsidered.

## What this skill produces (three outputs, gated)

1. **Deal risk / blockers** — a disqualifier from the playbook's
   Identifiers/Qualifiers/Disqualifiers table (see
   `references/net-new-sales-playbook.md`) that showed up on this call, or a
   qualifier that should be present for this motion and isn't.
2. **Discovery gaps** — questions the playbook indicates should have been asked
   for this motion and weren't. Motion-specific: a Finding What's New deal
   missing a ship-date question is a real gap; a Blueprint deal missing the
   same question isn't, because it's not diagnostic there.
3. **Call coaching / quality feedback** — aligns with the existing AE
   Discovery Coaching skill's private-artifact framing ("coaching, not
   surveillance"). If that skill's category structure is available in this
   workspace, defer to it rather than inventing a parallel rubric. If it isn't
   available, use the Identifiers/Qualifiers table as the coaching backbone:
   did the rep get the identifier signals needed to bucket the deal correctly,
   and the qualifier signals needed to know if it can win.

## Delivery Gate — do not fire on every call

Before producing a rep-facing output, check:

- Is there at least one disqualifier present, or at least one qualifier
  missing that the motion requires past a reasonable point in the deal
  (roughly: past the second call)?
- Is there a genuine discovery gap — a question the playbook flags as
  diagnostic for this motion that never got asked?

If neither is true, this skill should say so briefly and stop, not manufacture
a coaching note to justify having run. A quiet, correctly-timed flag is worth
more than a ceremonial one on every call. This is the single biggest design
choice standing between this skill getting used and this skill getting muted
the way the original coaching skill did.

## Visibility — assumption, not a settled decision

**Default for this build: rep-only.** Coaching and risk output goes to the rep
who ran the call, not automatically to a manager or into a shared channel.
Trust has to be earned before this becomes anything closer to a performance
signal. This is an assumption made to ship a working v1, not a permanent
design — if Katy or Emily want manager visibility, that's a governance
decision that should be made explicitly and probably rolled out separately
from this skill's first release, not bundled into it by default.

## Process

1. Run after (or alongside) `post-call-opportunity-actions` on the same call.
   Never delay that skill's rep-facing output waiting on this one.
2. Identify or confirm the deal's motion using the Structure Gate in
   `references/net-new-sales-playbook.md`. If the motion is ambiguous from
   available context, say so rather than guessing — an ambiguous motion is
   itself worth flagging, not silently resolving.
3. Walk the Identifiers/Qualifiers/Disqualifiers table for that motion against
   what actually happened on the call.
4. Apply the Delivery Gate. If nothing clears it, stop here.
5. If something clears it, write the three outputs with the specific evidence
   from the transcript that supports each one — a coaching note or risk flag
   with no transcript evidence attached is not something a rep can act on or
   trust.

## Guardrails

- Every flag needs a transcript-grounded reason. "This deal feels risky" is
  not an output; "no VP+ sponsor engaged after two calls on a $350K Finding
  What's New deal, which the playbook flags as a disqualifier" is.
- This skill never changes CRM data and never sends anything. It's read-only,
  advisory, and — per the visibility default above — rep-facing by default.
- If this skill and `post-call-opportunity-actions` disagree about which
  motion the deal is in, that disagreement is worth surfacing on its own; it
  usually means the CRM's Motion field is stale.

## Step 6. Update the call record — do not create a second row

This skill does **not** own row creation in the **Opportunity Post-Call
Actions** database. `post-call-opportunity-actions` creates the row for a
given call and sets `Call ID`; this skill finds that row and updates it.

- Database: https://app.notion.com/p/a7bed23291f64c888cf7f4a435666ba5, data source `39bfce03-75ad-40a5-aa69-a6c1d4e3325b`

1. Query the data source for the row where `Call ID` matches this call's Gong
   or Granola call ID.
2. **If a row exists**, update it — never insert a new row for a call that
   already has one.
3. **If no row exists** (this skill ran first, or ran standalone), create one
   with `Call`, `Call ID`, `Account / Opportunity`, `Call Date`, `Call
   Source`, and `Motion Assessed` populated at minimum, so
   `post-call-opportunity-actions` can find and complete it later instead of
   creating a duplicate.

Properties this skill is responsible for:
- `Deal Risk / Blockers` (rich_text): one line per flag — `<risk/blocker> — evidence: "<quote>"`. Only populate if the Delivery Gate cleared.
- `Discovery Gaps` (rich_text): one line per gap — `<missing question or signal> — why it's diagnostic for this motion`. Only populate if the Delivery Gate cleared.
- `Coaching Feedback` (rich_text): the coaching note, grounded in transcript evidence. Only populate if the Delivery Gate cleared.
- `Coaching Gate Status` (select): `"Flagged"` if the gate cleared and the three fields above are populated, `"Gate Not Cleared"` if this skill ran and found nothing worth surfacing — leave the three text fields blank in that case, don't pad them.
- `Motion Assessed` (select): only set this if the row didn't already have it set (i.e. this skill ran first). If `post-call-opportunity-actions` already set a different motion than what this skill's own Structure Gate check found, don't silently overwrite it — write the disagreement into `Source Notes` instead.

## Step 7. Log the run

Every run also writes one row to the shared Skill Usage Log — this tracks
that the skill executed, separate from what it found. Do this including runs
where the Delivery Gate stops output entirely; a gated-and-silent run is
still a run, and needs to show up here the same as one that surfaced a flag,
otherwise this log undercounts how often the skill actually executes.

Get the run timestamp: `TZ=America/Chicago date -Iseconds`.

- Database: https://app.notion.com/p/eae2c789d88040029296dfa1bdb5d481, data source `f43c363f-edc7-461e-8fd3-6c835239f247`
- Row properties:
  - `Run` (title): `"deal-signal-coaching - <account/opportunity name> - <YYYY-MM-DD>"`
  - `Skill` (select): `"deal-signal-coaching"`
  - `date:Run Timestamp:start`: the timestamp above; `date:Run Timestamp:is_datetime`: `1`
  - `Trigger` (select): `"Scheduled"` if fired by the call-end webhook, `"Manual"` if invoked directly
  - `Run By` (people): attribute to the rep who owns the opportunity, not whoever wired the automation. Never hardcode a specific person.
  - `Status` (select): `"Success"` for both a gated-silent run and a run that surfaced a flag, `"Error"` only if the call itself couldn't be read or the motion couldn't be identified at all
  - `Notes` (rich_text): one line, distinguishing the two real outcomes, e.g. `"Gate not cleared, no flag surfaced"` or `"1 disqualifier flagged (no VP+ sponsor, FWN motion), 1 discovery gap"`

Do both writes before ending the turn regardless of outcome. If either
Notion write fails, say so plainly rather than silently dropping it — the
likely first-use cause is that the Claude connection hasn't been added to
the relevant database yet (Notion: open the database, ••• menu, Connections,
add Claude).

## Open items before this goes live

- Confirm with Katy/Emily whether v1 ships rep-only or with manager
  visibility, and whether that's a per-rep opt-in or a blanket policy.
- Confirm whether this should call the existing AE Discovery Coaching and
  MEDDPICC Scorecard skills directly (composition) or reimplement their logic
  here (duplication). Composition is strongly preferred — reimplementing
  scoring logic that already exists is exactly the kind of drift this whole
  project is trying to eliminate.
