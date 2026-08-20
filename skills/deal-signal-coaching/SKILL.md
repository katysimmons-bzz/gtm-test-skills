---
name: deal-signal-coaching
description: Reads a completed customer call against Honeycomb's Net New Sales Playbook and surfaces deal risks or blockers, discovery gaps, and call coaching / quality feedback — but only when there's actually a signal worth surfacing, not as a running commentary on every call. Use this whenever someone asks "what's the risk on this deal," "did I miss anything in discovery," "coach me on this call," "score this against MEDDPICC," or whenever this skill is wired to run automatically after post-call-opportunity-actions on the same Gong/Granola/Zoom call. Do not use this to generate a report for every single call by default — check the Delivery Gate below first. This is the evaluative half of the meeting-to-opportunity pipeline; pair it with post-call-opportunity-actions but never let it block that skill's output. Every run also writes one row to the shared "Skill Usage Log" Notion database, including gated-silent runs, so skill usage across the whole system is tracked in one place.
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

## Step 6. Log the run

Every run writes one row to the shared Skill Usage Log, including the runs
where the Delivery Gate stops output entirely. A gated-and-silent run is still
a run, and needs to show up in the log the same as one that surfaced a flag —
otherwise the log undercounts how often this actually executes.

Get the run timestamp: `TZ=America/Chicago date -Iseconds`.

Load Notion tools via tool search if deferred, then write one row:

- Database: https://app.notion.com/p/eae2c789d88040029296dfa1bdb5d481, data source `f43c363f-edc7-461e-8fd3-6c835239f247`
- Row properties:
  - `Run` (title): `"deal-signal-coaching - <account/opportunity name> - <YYYY-MM-DD>"`
  - `Skill` (select): `"deal-signal-coaching"`
  - `date:Run Timestamp:start`: the timestamp above; `date:Run Timestamp:is_datetime`: `1`
  - `Trigger` (select): `"Scheduled"` if fired by the call-end webhook, `"Manual"` if invoked directly
  - `Run By` (people): attribute to the rep who owns the opportunity the call belongs to, not to whoever wired the automation. Resolve via Notion's `fetch` tool with id `"self"` only for genuinely self-serve invocations. Never hardcode a specific person.
  - `Status` (select): `"Success"` for both a gated-silent run and a run that surfaced a flag, `"Error"` only if the call itself couldn't be read or the motion couldn't be identified at all
  - `Notes` (rich_text): one line, distinguishing the two real outcomes, e.g. `"Gate not cleared, no flag surfaced"` or `"1 disqualifier flagged (no VP+ sponsor, FWN motion), 1 discovery gap"`

Do this before ending the turn regardless of outcome. If the Notion write
itself fails, say so plainly rather than silently dropping it — the likely
first-use cause is that the Claude connection hasn't been added to the Skill
Usage Log database yet (Notion: open the database, ••• menu, Connections, add
Claude).

## Open items before this goes live

- Confirm with Katy/Emily whether v1 ships rep-only or with manager
  visibility, and whether that's a per-rep opt-in or a blanket policy.
- Confirm whether this should call the existing AE Discovery Coaching and
  MEDDPICC Scorecard skills directly (composition) or reimplement their logic
  here (duplication). Composition is strongly preferred — reimplementing
  scoring logic that already exists is exactly the kind of drift this whole
  project is trying to eliminate.
