# GTM Test Skills — Meeting Output Into Opportunity Workflow

One skill that implements the "Meeting Output Into Opportunity Workflow"
automation: a fix for the gap between a customer call happening and its output
reliably becoming opportunity workflow, rather than nicely formatted evidence
that a call happened.

## The skill

- **[post-call-opportunity-actions](skills/post-call-opportunity-actions/)** —
  runs on every call, no exceptions. Produces eight structured outputs split
  into two trust tiers:
  - **Operational (always fires):** commitments, follow-up actions, open
    questions, a proposed CRM update, and a follow-up email draft. Drafts
    only, never an unattended write.
  - **Evaluative (gated):** deal risk/blockers, discovery gaps, and coaching
    feedback — only surfaced when there's an actual risk, gap, or coaching
    signal worth a rep's attention. Rep-only visibility by default.

This was originally shipped as two separate skills (`post-call-opportunity-actions`
and `deal-signal-coaching`) and has since been merged into one. The trust-tier
split that used to be the boundary between two skills is now the boundary
between two output groups inside a single skill — see "Why two tiers and not
eight flat outputs" below for why that boundary still matters even without a
skill-level split enforcing it.

## Why two tiers and not eight flat outputs

The eight structured outputs split into two trust tiers: five that are
mechanical and low-stakes (the rep wants these automatically, every time), and
three that are evaluative (risk, discovery gaps, coaching — getting these
wrong, or making them unavoidable instead of chosen, is how you recreate the
exact adoption problem this project exists to fix — the coaching skill that
already existed and never got used because it required a manual trigger).
Merging into one skill means this tiering is no longer enforced by a skill
boundary; it has to hold as an internal discipline instead — the Delivery Gate
inside `SKILL.md` is what does that job now.

## Why the trigger matters more than the tiering

Nothing here fixes anything if it still requires someone to remember to ask
for it. This skill is designed to be wired to fire on a Gong or Granola
call-end event, not a nightly batch and not a manual command. That wiring
lives in the n8n workflow, not in the skill file — the skill assumes it's
being invoked with call context already in hand.

## Playbook as a shared resource

The skill carries its own copy of `references/net-new-sales-playbook.md`, a
static snapshot of Honeycomb's Net New Sales Playbook (motions, the
structure-gate bucketing logic, and the Identifiers/Qualifiers/Disqualifiers
table per motion).

- **Static, not a live Notion fetch** — keeps Notion out of the critical path
  of an automation that needs to be reliable enough to trust without checking.
  Re-pull this file when the playbook changes materially; don't wire live
  fetches into the hot path.

Source of truth: [Net New Sales Playbook](https://app.notion.com/p/398ef7d73d39819fa1f3c544a669b12c)
(Sales Playbooks > Revenue Enablement), including the Motion Bucketing —
Decision Logic sub-page and the Land & Expand motion sub-pages.

## Opportunity Post-Call Actions (the actual delivery destination)

The skill writes into a shared Notion database, one row per call:
https://app.notion.com/p/a7bed23291f64c888cf7f4a435666ba5

- Creates or updates the row for a given `Call ID` in a single write:
  commitments, follow-up actions, open questions, proposed CRM update, email
  draft, motion assessed, and — only if the Delivery Gate clears — deal
  risk/blockers, discovery gaps, and coaching feedback, plus a
  `Coaching Gate Status` field so a gated-silent run is visibly different
  from one that hasn't run yet.
- `CRM Update Applied` is a manual checkbox, never ticked by the skill — it's
  how a human closes the loop after actually pushing the change.

**Known tradeoff, not fully resolved:** Notion permissions are database-level,
not field-level. Putting both output tiers in one database means anyone with
access to it can see the coaching fields too, which is in tension with the
evaluative tier's rep-only default. Until there's a real answer, keep this
database's sharing scoped narrowly rather than opened to the whole GTM team —
that's a workspace-sharing decision, not something the skill file can
enforce.

## Skill Usage Log (the run log, separate from the above)

The skill also writes one row per run to Honeycomb's shared Skill Usage Log
(Notion), following the same pattern as `daily-schedule` and the rest of the
department standup skills: run timestamp, trigger type, who it ran for,
success/error, and a one-line note. This tracks that the skill executed at
all; it is not where the call's actual content lives, that's Opportunity
Post-Call Actions above. Gated-silent runs still get logged here too, since a
gate that fires correctly is a real outcome, not a non-event.

## Open items (not resolved by this skill file)

1. **Visibility on the evaluative tier.** Shipped rep-only by default. Manager
   visibility is a governance call for Katy/Arno, not a technical one, and
   probably shouldn't be bundled into the same release as v1.
2. **Composition vs. duplication.** The coaching output should call the
   existing MEDDPICC Scorecard and AE Discovery Coaching skills rather than
   reimplement their scoring logic. Confirm those skills' interfaces before
   wiring the handoff.
3. **Trigger implementation.** This is a skill definition, not the n8n
   workflow. The webhook-on-call-end wiring is the next build step, and it's
   the one that actually determines whether this gets used.
