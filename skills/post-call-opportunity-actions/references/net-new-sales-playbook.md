# Net New Sales Playbook — Reference Snapshot

Static snapshot of Honeycomb's Net New Sales Playbook (Notion), captured for use by
this skill without a live Notion dependency in the automation path. Source of truth
lives at: https://app.notion.com/p/398ef7d73d39819fa1f3c544a669b12c

**Re-sync note:** this file is a snapshot, not a live mirror. Re-pull it when the
playbook changes materially (new motion, new signal set, new CRM field). Do not
wire this skill to fetch Notion live on every call — that puts a third-party
dependency in the critical path of an automation that needs to be reliable enough
that reps trust it without checking.

## The three (five) motions

Every Honeycomb deal is bucketed into one of these, by **structure**, not by
**trigger** (an incident, a new exec, a cost crisis are the "why now" — they do not
by themselves determine the bucket):

1. **Full Platform Takeout (FPT)** — wall-to-wall replacement of the incumbent
   across the estate, regardless of what triggered it. Most common entry point,
   most concerning: highest loss rate unless specific win-conditions are present.
2. **Land and Expand (L&E)** — the golden path. Start narrow, expand with proof.
   Three sub-motions:
   - **Finding What's New (FWN)** — land on a workload with no incumbent to
     displace (new BU, new AI workload, new leadership mandate). Honeycomb's #1
     wedge into incumbent-entrenched accounts.
   - **Incident Management (IM)** — a dated, specific incident is the wedge the
     land is scoped to fix.
   - **Blueprint Deployment** — land one critical existing service to prove value
     when a full takeout isn't viable today.
3. **Pro to Enterprise (P2E)** — the land already happened; an existing paying Pro
   customer is hitting a ceiling (event volume, trigger/SLO limits, or a
   consolidation initiative).

## Structure gate (how to bucket a deal — first match wins)

1. Existing paying Pro/self-serve user hitting a ceiling → **P2E**
2. Initial land scoped to one team/service/workload with a phased expand path →
   **L&E** (then resolve which sub-motion, below)
3. Wall-to-wall replacement of the incumbent across the estate → **FPT**,
   regardless of trigger

Within L&E (first match wins):
1. New workload/BU/AI workload/new-leadership greenfield, nothing to displace →
   **Finding What's New**
2. A specific, dated incident is the wedge → **Incident Management**
3. Land one existing critical service to prove value → **Blueprint Deployment**

**Trap to avoid:** a named competitor (Datadog, Splunk, Dynatrace, New Relic,
Grafana) shows up in every motion. It does not, by itself, mean FPT. FPT is
defined by scope (whole estate), not by the presence of a competitor.

## Three CRM fields that make this deterministic instead of reconstructed later

Capture these on the opportunity, separate from Motion, and mark them
**conversation-confirmed** (not tool-detectable, so no automation should infer
them from a single field like the SFDC competitor field alone):

| Field | Picklist | Role |
|---|---|---|
| Land Scope | One service/team · Multi-team · Whole estate | The FPT-vs-L&E discriminator |
| Incumbent Disposition | Replacing wall-to-wall · Keeping + landing beside · Greenfield/none | Confirms structure |
| Compelling Event | Incident · New workload · New leadership · Cost · Renewal · Other | The "why now," visible but not bucket-driving |

This is the CRM update this skill should propose when the call establishes or
changes one of these three fields — not a freeform "update Salesforce" note.

## Identifiers / Qualifiers / Disqualifiers by motion

Use this table to judge whether a next step, a commitment, a proposed CRM
update, a risk flag, or a discovery gap actually fits the motion the deal is in.

| Motion | Identifiers (bucket it) | Qualifiers (win-likelihood) | Disqualifiers |
|---|---|---|---|
| **Full Platform Takeout** | Initial scope = whole-estate replacement | Regulatory deadline; leadership mandate; incumbent renewal date; real OTel readiness; EB directly engaged; MVC agreed pre-POC; technical win > commercial ask | Feature-parity bake-off; values sameness; broad/generic pain; no champion ≠ EB; committee RFP |
| **Finding What's New** | New CTO/VP with modernization mandate; greenfield AI/agentic platform; new BU with own P&L; high-volume AI/ML inference | `gen_ai.*` attributes flowing; building-vs-running (the evals line); a real ship date | New leadership kills the initiative; familiarity-gap trap ("we need dashboards, not queries"); $350K+ with no VP sponsor |
| **Incident Management** | Public/high-profile incident; named dated incident with quantified impact; recurring cost/uptime bleed | In-house SLO stack retained as scope boundary | Single champion, no multithreading; prospect wants AI auto-resolution, not observability |
| **Blueprint Deployment** | One critical, visible service with an on-call/observability gap | Incumbent spend self-funds next phase; prior closed-lost blocker resolved | Prospect frames the ask as a full platform switch (re-route to FPT) |
| **Pro to Enterprise** | Event volume > 18B/yr Pro cap; trigger/SLO limits; sampling degrading fidelity; logs/tooling consolidation initiative | AI/MCP curiosity as value-add, not primary driver | Single-feature ask with no volume growth; no exec touch in 1–2 calls; champion unavailable |

## The evals line (only relevant to AI-workload / Finding What's New deals)

The single most important qualifying question: is the prospect talking about
**building** the agent, or **running** it? Honeycomb wins the "running" (production)
conversation. If the prospect is only discussing evals, prompt tuning, or offline
testing, that is a "too early" signal, not a disqualifier. It belongs in nurture
until a ship date exists — not in every-call risk escalation.
