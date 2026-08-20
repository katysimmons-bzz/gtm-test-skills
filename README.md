# GTM Test Skills — Meeting Output Into Opportunity Workflow

Two skills that make up the "Meeting Output Into Opportunity Workflow"
automation: a fix for the gap between a customer call happening and its output
reliably becoming opportunity workflow, rather than nicely formatted evidence
that a call happened.

## The two skills

- **[post-call-opportunity-actions](skills/post-call-opportunity-actions/)** —
  the operational half. Runs on every call, no exceptions. Produces
  commitments, follow-up actions, open questions, a proposed CRM update, and a
  follow-up email draft. Drafts only, never an unattended write.
- **[deal-signal-coaching](skills/deal-signal-coaching/)** — the evaluative
  half. Runs on every call but only surfaces output when there's an actual
  risk, gap, or coaching signal worth a rep's attention. Rep-only by default.

## Why two skills and not eight

The eight structured outputs split into two trust tiers: five that are
mechanical and low-stakes (the rep wants these automatically, every time), and
three that are evaluative (risk, discovery gaps, coaching — getting these
wrong, or making them unavoidable instead of chosen, is how you recreate the
exact adoption problem this project exists to fix — the coaching skill that
already existed and never got used because it required a manual trigger).
Splitting by trust tier, not by output type, is the load-bearing decision here.

## Why the trigger matters more than the skill boundary

Neither skill fixes anything if it still requires someone to remember to ask
for it. Both are designed to be wired to fire on a Gong or Granola call-end
event, not a nightly batch and not a manual command. That wiring lives in the
n8n workflow, not in these skill files — these skills assume they're being
invoked with call context already in hand.

## Playbook as a shared resource

Both skills carry their own copy of `references/net-new-sales-playbook.md`, a
static snapshot of Honeycomb's Net New Sales Playbook (motions, the
structure-gate bucketing logic, and the Identifiers/Qualifiers/Disqualifiers
table per motion). Static and duplicated on purpose:

- **Static, not a live Notion fetch** — keeps Notion out of the critical path
  of an automation that needs to be reliable enough to trust without checking.
  Re-pull this file when the playbook changes materially; don't wire live
  fetches into the hot path.
- **Duplicated, not shared via a pointer** — each skill stays standalone and
  dependency-free, consistent with how the rest of this skill set is built.

Source of truth: [Net New Sales Playbook](https://app.notion.com/p/398ef7d73d39819fa1f3c544a669b12c)
(Sales Playbooks > Revenue Enablement), including the Motion Bucketing —
Decision Logic sub-page and the Land & Expand motion sub-pages.

## Opportunity Post-Call Actions (the actual delivery destination)

Both skills write into one shared Notion database, one row per call:
https://app.notion.com/p/a7bed23291f64c888cf7f4a435666ba5

- `post-call-opportunity-actions` creates the row: commitments, follow-up
  actions, open questions, proposed CRM update, email draft, motion assessed.
- `deal-signal-coaching` updates the same row rather than creating a second
  one, keyed on `Call ID`: deal risk/blockers, discovery gaps, coaching
  feedback, and a `Coaching Gate Status` field so a gated-silent run is
  visibly different from one that hasn't run yet.
- `CRM Update Applied` is a manual checkbox, never ticked by either skill —
  it's how a human closes the loop after actually pushing the change.

**Known tradeoff, not fully resolved:** Notion permissions are database-level,
not field-level. Putting both skills' output in one database means anyone
with access to it can see the coaching fields too, which is in tension with
`deal-signal-coaching`'s rep-only default. Until there's a real answer, keep
this database's sharing scoped narrowly rather than opened to the whole GTM
team — that's a workspace-sharing decision, not something either skill file
can enforce.

## Skill Usage Log (the run log, separate from the above)

Both skills also write one row per run to Honeycomb's shared Skill Usage Log
(Notion), following the same pattern as `daily-schedule` and the rest of the
department standup skills: run timestamp, trigger type, who it ran for,
success/error, and a one-line note. This tracks that a skill executed at all;
it is not where the call's actual content lives, that's Opportunity Post-Call
Actions above. `deal-signal-coaching`'s gated-silent runs still get logged
here too, since a gate that fires correctly is a real outcome, not a
non-event. Both skill names are registered as `Skill` select options on this
database rather than falling into the `other` catch-all.

## Open items (not resolved by these skill files)

1. **Visibility on deal-signal-coaching.** Shipped rep-only by default. Manager
   visibility is a governance call for Katy/Arno, not a technical one, and
   probably shouldn't be bundled into the same release as v1.
2. **Composition vs. duplication.** deal-signal-coaching should call the
   existing MEDDPICC Scorecard and AE Discovery Coaching skills rather than
   reimplement their scoring logic. Confirm those skills' interfaces before
   wiring the handoff.
3. **Trigger implementation.** These are skill definitions, not the n8n
   workflow. The webhook-on-call-end wiring is the next build step, and it's
   the one that actually determines whether this gets used.
