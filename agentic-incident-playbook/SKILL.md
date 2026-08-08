---
name: agentic-incident-playbook
description: >
  A tested response protocol for when an AI agent in the revenue stack
  produces a wrong output that reaches a decision, a report, or a
  customer. Covers pipeline scoring errors, forecast model drift,
  unauthorized CRM writes, outbound sequence failures, and cascading
  data errors. Triggers on AI broke something, wrong forecast, bad lead
  score, agent sent the wrong email, AI updated the wrong field, model
  drift, forecast was off, the agent messed up, incident response, or
  any situation where an AI-touched output in the revenue stack produced
  a wrong result that was acted on. BOUNDARY - For validating AI outputs
  before they become incidents, see why-audit.
---

# Agentic Incident Playbook — What to Do When AI Gets It Wrong in Your Revenue Stack

You are helping someone respond to a specific incident where an AI agent in their revenue stack produced a wrong output that was acted on — a bad score that triggered the wrong sequence, a forecast number that reached a board deck unchecked, a CRM field updated with wrong data, or an outbound message sent to the wrong person with the wrong context.

This is not a governance planning document. This is what you do when something already went wrong.

---

## Why This Exists

74% of business leaders are giving agentic AI access to their data and processes. Only 1 in 5 have a tested response plan for when it fails. 48% have a plan documented but never tested. (Grant Thornton, 2026 AI Impact Survey, 950 business leaders.)

A plan you haven't tested is a hypothesis, not a plan.

Most incident response frameworks were designed for a world where humans make decisions with AI assistance. In that world, a human checkpoint exists at every decision point. Agentic AI removes that checkpoint. When an agent scores a deal, updates a field, triggers a sequence, or generates a forecast number without a human reviewing each step, the failure mode is different: the error can propagate before anyone knows it happened.

This playbook is built for that failure mode.

---

## The Five Failure Types

Every agentic AI incident in a revenue stack falls into one of these categories. Identifying the type first determines the response path.

### Type 1: Model Drift

**What it is:** The AI's scoring, weighting, or classification has silently shifted from its original calibration. The model wasn't updated, but the data it's reasoning over has changed enough that its outputs no longer match reality.

**Example:** A pipeline scoring model was calibrated on Q1 data when average deal size was $45K. By Q3, a new product launched that shifted average deal size to $28K. The model is still scoring deals as if $45K is normal, flagging smaller deals as low-quality when they're actually on-target for the new product.

**How you'd catch it:** Forecast accuracy declines gradually. Scores stop correlating with outcomes. Reps start ignoring the scores because they "feel wrong" but can't explain why.

**Severity:** Medium-high. Doesn't cause a single dramatic failure — causes a slow, invisible degradation that's harder to catch because each individual output looks plausible.

### Type 2: Unauthorized or Unintended Actions

**What it is:** The agent performed an action it shouldn't have — updated a field it didn't have permission to touch, sent a communication without approval, or escalated/de-escalated something outside its defined scope.

**Example:** An outbound sequencing agent was configured to enroll new leads into a nurture sequence. A configuration error caused it to also re-enroll churned customers into a win-back sequence that hadn't been approved, sending pricing offers to accounts that left on bad terms.

**How you'd catch it:** A customer or rep reports something unexpected. For field updates, check the field history in your CRM — it logs what changed, when, and what user or integration made the change. An agent writing to a field it shouldn't touch will show up there before anyone notices the downstream effect. For actions outside the CRM (emails sent, sequences triggered), check the agent's own audit log for actions outside its defined scope.

**Severity:** High. Direct customer or prospect impact. Potential brand and relationship damage.

### Type 3: Cascading Data Errors

**What it is:** One wrong AI output fed the next step in a workflow, which fed the next, creating a chain of errors that amplified the original mistake.

**Example:** An AI enrichment tool pulled the wrong company data for a contact, associating them with a competitor instead of their actual employer. That wrong association fed the lead scoring model, which scored them as disqualified. The disqualification triggered an automated removal from an active sequence. A real prospect was silently dropped from the pipeline because of one wrong data point three steps upstream.

**How you'd catch it:** A rep notices a prospect disappeared. A pipeline review reveals deals that were removed without explanation. An audit of the enrichment source shows a mismatch.

**Severity:** High. The further downstream the error travels before detection, the harder it is to reconstruct what happened and the more decisions it has contaminated.

### Type 4: Confident Wrong Narrative

**What it is:** The AI generated a causal explanation or analysis that was wrong but sounded right, and it was used in a decision, a board deck, or a coaching conversation before anyone validated it.

**Example:** A forecast variance report asked AI "why did enterprise win rate drop." The AI attributed the decline to "increased competitive pressure" based on two deals where a competitor was mentioned in call notes. In reality, the primary driver was an internal pricing change that made enterprise deals harder to close. The "competitive pressure" narrative reached the board, and the company invested in competitive positioning when the fix was actually a pricing adjustment.

**How you'd catch it:** Someone close to the deals questions the narrative. A why-audit (see `why-audit/SKILL.md`) would have flagged single-cause certainty and mechanism-without-evidence markers. Often caught too late — after the narrative has already shaped a decision.

**Severity:** Critical. Directly causes wrong strategic decisions. The most dangerous failure type because the output looks and feels like expert analysis.

### Type 5: Stale or Partial Data Reasoning

**What it is:** The AI answered a question spanning a large dataset but only processed a fraction of it due to context window limits, missing integrations, or stale data — and gave no indication that its answer was based on incomplete information.

**Example:** A revenue leader asked the AI to analyze customer pain points across Q2 call transcripts. The system had 4,000 transcripts but could only process roughly 15-20 before quality degraded. It keyword-sampled for "pain" and "challenge," processed fragments of matching transcripts, and generated a confident analysis. The response read identically to one that had processed every transcript. The leader used it to set Q3 product priorities.

**How you'd catch it:** Compare the AI's cited evidence against the total available data. If the answer references 5 calls but 4,000 exist, the gap is the tell. Often not caught because the output gives no signal of how much data it actually saw.

**Severity:** Medium-high. Decisions made on partial data feel fully informed. The absence of a confidence disclaimer is itself the problem.

---

## Response Protocol

When an incident is identified, follow this sequence. Do not skip steps.

### Step 1: Contain

**Goal:** Stop the error from spreading further.

**Who acts:** Every containment action is performed by a human. The agent that caused the incident should not be trusted to assess or contain its own failure.

| Failure type | Containment action |
|---|---|
| Model drift | Pause the model's outputs from feeding downstream systems. Revert to manual scoring/classification until recalibrated. |
| Unauthorized action | Revoke the agent's write access immediately. For CRM field changes, pull the field history to identify every record the agent touched. For actions outside the CRM (emails sent, sequences triggered), pull the agent's own audit log to identify every action taken outside its defined scope. |
| Cascading error | Trace the chain back to the originating data point. Quarantine every record the error touched. |
| Confident wrong narrative | Retract or flag the narrative in every place it was shared — board deck, Slack, email, CRM notes. Notify everyone who received it. |
| Stale/partial data | Flag the output as unverified. Re-run the analysis with explicit data-coverage checks or do it manually. |

**Rule:** Containment happens before root-cause analysis. The instinct to understand why it happened before stopping it is natural and wrong. Stop it first.

### Step 2: Assess Impact

**Questions to answer:**
- What decisions were made based on this output?
- Who saw it — reps, managers, executives, board members, customers?
- How long was the error active before detection?
- Did the error trigger any downstream automated actions (sequence enrollments, field updates, score changes, alerts)?
- Is the error still producing wrong outputs, or has containment stopped it?

**Document everything.** The incident log (see `references/incident-log-template.md`) captures this in a structured format.

### Step 3: Notify

**Who needs to know depends on severity:**

**Critical and High:** Notify the head of every affected department (sales, marketing, CS, finance — whoever owns a metric or process the error touched), anyone who made a decision based on the output, and leadership if the output reached a board deck, investor communication, or customer. If a customer or prospect received a wrong communication, the relationship owner (rep or CSM) handles the correction directly. These notifications happen the same day and cannot wait for root cause.

**Medium:** Notify the affected department head and document the incident. Broader notification is a judgment call based on whether the error, even though it was caught, reveals a pattern that others should know about.

**Low:** Document it. Notify the affected department head if recalibration is needed. No broader notification unless investigation reveals a higher severity than initially assessed.

**Rule:** For Critical and High incidents, notification cannot be delayed until root cause is complete. "We found an error, here's what we know so far, here's what we're doing" is a complete notification. For Medium and Low, use judgment on timing and scope.

### Step 4: Root Cause

**For each failure type, the root-cause question is different:**

| Failure type | Root-cause question |
|---|---|
| Model drift | When was the model last calibrated? What changed in the underlying data since then? |
| Unauthorized action | Was the agent's scope properly defined? Was there an access-control gap, or did the agent operate within its permissions but those permissions were too broad? |
| Cascading error | Where in the chain did the first wrong output enter? Was there a validation step that should have caught it? |
| Confident wrong narrative | Was a why-audit run before the narrative was used? If not, why not? If yes, what did it miss? |
| Stale/partial data | Was there any signal to the user that the data was incomplete? Is there a way to add that signal? |

**Rule:** Root cause must identify a systemic fix, not just an individual correction. "We fixed the data" solves this instance. "We added a validation check so this class of error gets caught" solves the category.

### Step 5: Remediate

**Three levels of remediation:**

1. **Correct the data.** Fix every record the error touched. This is often the most time-consuming step because cascading errors can touch hundreds of records.
2. **Correct the decisions.** Revisit every decision that was informed by the wrong output. Some may still be correct despite the bad input. Others need to be reversed.
3. **Correct the system.** Implement the systemic fix identified in root cause. Log the change in `references/post-incident-changes-log.md`.

### Step 6: Test the Fix

**Do not skip this step.** The systemic fix must be tested before the agent is restored to production.

- Run the fix against the data that caused the original failure. Does it now produce the correct output?
- Run it against adjacent data that could trigger a similar failure. Does it hold?
- If the fix is a new validation check, deliberately feed it an error and confirm it catches it.

**Rule:** An untested fix is the same as an untested plan. It gives you the comfort of having acted without the evidence that it works.

### Step 7: Document and Close

Log the full incident in `references/incident-log-template.md`:
- Date detected, date contained, date resolved
- Failure type
- What happened (plain language)
- Impact assessment
- Root cause
- Systemic fix implemented
- Test results
- Who was notified

If your organization maintains an AI disclosure log or board reporting package, this incident log feeds directly into that reporting cycle. If not, the incident log is the permanent record of what happened and what changed.

---

## Severity Matrix

| Severity | Definition | Response time | Escalation |
|---|---|---|---|
| Critical | Wrong output reached a board, a customer, or triggered a strategic decision | Contain within 1 hour. Notify stakeholders same day. | CRO + CEO immediately |
| High | Wrong output reached reps or managers and may have influenced deal-level decisions | Contain within 4 hours. Notify affected teams within 24 hours. | CRO + affected department head |
| Medium | Wrong output was generated but caught before it influenced a decision | Document and fix within 1 week. | Affected department head |
| Low | A potential drift or quality degradation was flagged proactively, no wrong output yet | Investigate within 2 weeks. Recalibrate if confirmed. | RevOps |

---

## Running a Tabletop Exercise

A playbook you haven't tested is a hypothesis. Run a tabletop exercise at least once before you need it for real.

**How to run it:**

1. Pick one failure type from the five above.
2. Write a realistic scenario (use the examples in this document or create your own from a near-miss your team has actually experienced).
3. Walk through the response protocol with every person who would be involved in a real incident — RevOps, the affected department head, the CRO, whoever manages the AI tool.
4. Time each step. Note where people hesitate, where ownership is unclear, and where the protocol assumes something that isn't actually set up (e.g., "check the audit log" when no audit log exists).
5. Fix every gap the exercise surfaces before it becomes a gap in a real incident.

**Cadence:** Run a tabletop exercise quarterly, rotating through different failure types. After any real incident, run an exercise on a variation of that incident type within 30 days.

---

## Voice Rules

- State what happened before explaining why
- No minimizing language — "a minor data discrepancy" when a wrong forecast reached the board is not minor
- Short, direct sentences in notifications — the recipient needs to understand the impact in under 30 seconds
- Own the error organizationally, don't blame the tool — "our AI scoring model drifted" not "the AI got it wrong"

---

## Reference Files

| File | When to read | What's inside |
|---|---|---|
| `references/incident-log-template.md` | Every incident | Structured log for documenting the full incident lifecycle |
| `references/post-incident-changes-log.md` | After remediation | Running log of systemic changes made after incidents |
| `references/tabletop-scenarios.md` | Quarterly exercises | Pre-written scenarios for each failure type, ready to run |

## Related Skills

- **why-audit** — prevents Type 4 (confident wrong narrative) incidents by validating causal claims before they reach decisions
- **semantic-layer-setup** — prevents Type 1 (model drift) and Type 3 (cascading errors) by ensuring metric definitions are documented and versioned
