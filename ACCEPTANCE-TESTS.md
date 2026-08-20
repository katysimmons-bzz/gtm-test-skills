# GTM Test Skills Acceptance Tests

These tests exist to keep `post-call-opportunity-actions` inside its intended
boundaries. The point is not "did the output sound smart." The point is "did
the system behave predictably, with the right split between operational
output and evaluative output, and without inventing work."

This skill was originally two separate skills (`post-call-opportunity-actions`
and `deal-signal-coaching`) and has since been merged into one, with the old
skill boundary now enforced as an internal tiering (operational vs.
evaluative, gated by the Delivery Gate) instead. These tests are written
against the merged skill.

## How to use this file

For each test case, run the skill against the call artifact and score pass or
fail. A pass means the behavior holds consistently, not once by luck. If a
test fails, fix the boundary or rule that caused the failure before adding new
prompt prose elsewhere.

## Test 1. Open questions stay literal

**Scenario:** The rep asks two questions on the call and one remains unanswered.
There are also several obvious discovery questions the rep should have asked
but never did.

**Expected behavior:**
- The operational output includes only the actually asked but still
  unresolved item in `Open Questions`.
- The evaluative output captures the missing discovery as a discovery gap if
  the motion makes that question diagnostic and the Delivery Gate clears.
- The operational output does not smuggle "questions the rep should have
  asked" into `Open Questions`.

## Test 2. No ceremonial coaching

**Scenario:** A clean routine call with no clear disqualifier, no motion-specific
discovery miss, and no strong coaching signal.

**Expected behavior:**
- The skill sets `Coaching Gate Status` to `Gate Not Cleared`.
- It does not invent a blocker, soft warning, or generic improvement note just
  to justify running the evaluative tier.
- The five operational outputs still ship regardless.

## Test 3. Missing positive evidence is not automatically a risk

**Scenario:** A first or second call where some desirable qualifier has not yet
been confirmed, but there is no transcript evidence of a contradiction,
disqualifier, or decision-blocking omission.

**Expected behavior:**
- The evaluative output does not flag the absence alone as deal risk.
- It may note ambiguity only when the playbook timing or motion logic makes it
  materially relevant.
- Risk language requires transcript-grounded evidence, not missing optimism.

## Test 4. Motion-specific discovery only

**Scenario:** The same call artifact is evaluated once as a valid
`Finding What's New` motion and once as a valid `Blueprint Deployment` motion.

**Expected behavior:**
- The discovery-gap output changes based on motion.
- Questions that are diagnostic for one motion are not treated as universal.
- The operational output still drafts the CRM update and email in a way that
  fits the assessed motion.

## Test 5. No duplicate call rows

**Scenario:** The skill runs on a call, and is then re-run on the same call ID
(e.g. a retry after a partial failure, or a manual re-invocation).

**Expected behavior:**
- Only one row exists in the Opportunity Post-Call Actions database for that
  call ID.
- The skill queries for an existing row by `Call ID` and updates it rather
  than inserting a duplicate.

## Test 6. Manual invocation does not create side effects it can't back up

**Scenario:** A user runs the skill manually in analysis mode outside the
automated call-end pipeline.

**Expected behavior:**
- The skill can still analyze and draft its output.
- It does not pretend the write happened if the automation context or database
  write step is not available.
- Side effects are reserved for the actual automated workflow or clearly
  stated write-capable runs.

## Test 7. CRM updates stay evidence-bound

**Scenario:** The call supports some field updates but not others. For example,
the motion is clear and a compelling event is mentioned, but there is no
transcript support for a stage change.

**Expected behavior:**
- The operational output proposes only the fields the call actually supports.
- It does not suggest a stage change, owner change, amount change, or close
  action without explicit evidence.
- Each proposed CRM update includes field-level evidence and confidence.

## Test 8. Commitments and follow-up actions do not blur together

**Scenario:** The rep explicitly promises one thing, the customer says they
will do another thing, and there is one next step implied but never stated as a
promise.

**Expected behavior:**
- `Commitments` captures explicit promises with owner and any date mentioned.
- `Follow-up Actions` captures concrete next steps whether or not they were
  framed as commitments.
- The skill does not infer a commitment that was never actually stated.

## Test 9. Source disagreement is surfaced, not flattened

**Scenario:** Gong and Granola disagree on a material point, such as whether a
customer accepted a next step or whether a specific commitment was made.

**Expected behavior:**
- The operational output flags the disagreement in `Source Notes`.
- It does not silently choose whichever version produces the cleaner summary.
- Any evaluative output that depends on the disputed point is qualified
  accordingly.

## Test 10. Operational output never waits on the evaluative gate

**Scenario:** The call is ambiguous enough that the Delivery Gate can't be
resolved quickly (e.g. the motion is unclear).

**Expected behavior:**
- The five operational outputs still ship on their normal timeline.
- The evaluative tier reports "gate not cleared" or flags the motion as
  ambiguous rather than blocking or delaying the operational write.
- A merged skill only works if this guarantee survives the merge — this test
  exists specifically to catch a regression where one write path starts
  blocking on the other.
