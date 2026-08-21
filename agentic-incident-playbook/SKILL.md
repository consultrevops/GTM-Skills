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
  a wrong result that was acted on. BOUNDARY - For detecting failures
  before they reach a decision, see agent-monitoring. For validating AI
  outputs before they become incidents, see why-audit.
---

# Agentic Incident Playbook

You are helping someone respond to an incident where an AI agent in their revenue stack produced a wrong output that was acted on.

This skill covers response. For detection, see `agent-monitoring/SKILL.md`.

---

## The Five Failure Types

Classify the incident before responding. The type determines the containment action, the root cause question, and the response timeline.

### Type 1: Model Drift

**What it is:** The AI's scoring, weighting, or classification has shifted from its original calibration. The model was not updated, but the data it reasons over changed enough that its outputs no longer match reality.

**Example:** A pipeline scoring model was calibrated on Q1 data when average deal size was $45K. By Q3, a new product launched that shifted average deal size to $28K. The model is still scoring deals as if $45K is normal, flagging smaller deals as low-quality when they are on-target for the new product.

**Identifying signs:** Forecast accuracy declining gradually. Scores no longer correlating with outcomes. Reps ignoring scores without being able to explain why.

### Type 2: Unauthorized or Unintended Actions

**What it is:** The agent performed an action outside its defined scope. Wrote to a field it lacked permission for, sent a communication without approval, or escalated something it should not have.

**Example:** An outbound sequencing agent was configured to enroll new leads into a nurture sequence. A configuration error caused it to also re-enroll churned customers into an unapproved win-back sequence, sending pricing offers to accounts that left on bad terms.

**Identifying signs:** A customer or rep reports something unexpected. Field history shows writes by the agent's integration user to fields outside its permitted list. The agent's audit log shows actions outside its defined scope.

### Type 3: Cascading Data Errors

**What it is:** One wrong output fed the next step in a workflow, which fed the next, amplifying the original error.

**Example:** An AI enrichment tool matched a contact to a competitor's domain instead of their actual employer. That association fed the lead scoring model, which marked them disqualified. The disqualification triggered automated removal from an active sequence. A real prospect disappeared from the pipeline because of one wrong data point three steps upstream.

**Identifying signs:** A rep reports a prospect that disappeared. Pipeline review shows records removed without explanation. An audit of an enrichment source shows mismatches.

### Type 4: Confident Wrong Narrative

**What it is:** The AI generated a causal explanation that was wrong but sounded right, and it was used in a decision before anyone validated it.

**Example:** A forecast variance report asked AI why enterprise win rate dropped. The AI attributed the decline to competitive pressure based on two deals where a competitor appeared in call notes. The actual driver was an internal pricing change. The competitive narrative reached the board, and the company invested in competitive positioning when the fix was pricing.

**Identifying signs:** Someone close to the deals questions the narrative. A why-audit would have flagged single-cause certainty or mechanism without evidence. See `why-audit/SKILL.md`.

### Type 5: Stale or Partial Data Reasoning

**What it is:** The AI answered a question spanning a large dataset but processed only a fraction of it, with no indication that the answer was based on incomplete information.

**Example:** A revenue leader asked AI to analyze customer pain points across Q2 call transcripts. The system held 4,000 transcripts and processed roughly 20. It keyword-sampled, analyzed fragments, and produced a confident analysis that read identically to one built on the full set. The leader used it to set Q3 product priorities.

**Identifying signs:** The AI's cited evidence covers a small share of available data. The output carries no coverage disclosure.

---

## Response Protocol

Follow in sequence. Do not skip steps.

### Step 1: Contain

Stop the error from spreading before analyzing why it happened.

Every containment action is performed by a human. Do not direct the agent that caused the incident to assess or contain its own failure.

| Failure type | Containment action |
|---|---|
| Model drift | Pause the model's outputs from feeding downstream systems. Revert to manual scoring or classification until recalibrated. |
| Unauthorized action | Revoke the agent's write access. For CRM field changes, pull field history to identify every record touched. For actions outside the CRM, pull the agent's audit log to identify every out-of-scope action. |
| Cascading error | Trace the chain back to the originating data point. Quarantine every record the error touched. |
| Confident wrong narrative | Retract or flag the narrative everywhere it was shared. Board deck, Slack, email, CRM notes. Notify everyone who received it. |
| Stale or partial data | Flag the output as unverified. Re-run with explicit coverage checks or complete the analysis manually. |

### Step 2: Assess Impact

Establish:

- What decisions were made based on this output
- Who saw it: reps, managers, executives, board members, customers
- How long the error was active before detection
- What downstream automated actions it triggered
- Whether containment has stopped it

Record in `references/incident-log-template.md`.

### Step 3: Notify

Scope by severity.

**Critical and High:** the head of every affected department, anyone who made a decision based on the output, and leadership if the output reached a board deck, investor communication, or customer. Where a customer or prospect received a wrong communication, the relationship owner handles the correction. Notify the same day. Do not wait for root cause.

**Medium:** the affected department head. Broader notification is a judgment call based on whether the error reveals a pattern others should know about.

**Low:** document it. Notify the affected department head if recalibration is needed.

A complete notification states what was found, what is known so far, and what is being done. It does not require a completed root cause.

### Step 4: Root Cause

| Failure type | Root cause question |
|---|---|
| Model drift | When was the model last calibrated? What changed in the underlying data since then? |
| Unauthorized action | Was the agent's scope properly defined? Was this an access-control gap, or permissions that were too broad? |
| Cascading error | Where did the first wrong output enter the chain? Was there a validation step that should have caught it? |
| Confident wrong narrative | Was a why-audit run before the narrative was used? If not, why not. If yes, what did it miss. |
| Stale or partial data | Was there any signal to the user that the data was incomplete? Can that signal be added? |

Root cause must identify a systemic fix, not only an individual correction. "We fixed the data" addresses the instance. "We added a validation check for this class of error" addresses the category.

### Step 5: Remediate

Three levels, all required.

1. **Correct the data.** Fix every record the error touched.
2. **Correct the decisions.** Revisit every decision informed by the wrong output. Some may still hold. Others need reversing.
3. **Correct the system.** Implement the systemic fix from Step 4. Record it in the incident log.

### Step 6: Test the Fix

Test before restoring the agent to production.

- Run the fix against the data that caused the original failure. Confirm correct output.
- Run it against adjacent data that could trigger a similar failure.
- If the fix is a validation check, feed it an error and confirm it catches it.

### Step 7: Document and Close

Log in `references/incident-log-template.md`:

- Date detected, date contained, date resolved
- Failure type
- What happened, in plain language
- Impact assessment
- Root cause
- Systemic fix implemented
- Test results
- Who was notified

Where an AI disclosure log or board reporting package exists, this log feeds that cycle.

---

## Response Timelines

Type 2 is the exception. An agent taking unauthorized actions is still taking them, so containment begins as soon as someone is aware, regardless of severity. The other four types have already produced their output and are measured in hours or days.

These targets are starting points. Set your own based on team coverage and what the agents touch.

| Failure type | Severity | Containment | Notification |
|---|---|---|---|
| Type 2 | Any | Immediately on awareness | Same day |
| Types 1, 3, 4, 5 | Critical | Same day | Same day |
| Types 1, 3, 4, 5 | High | Within 1-2 business days | Within 2 business days |
| Types 1, 3, 4, 5 | Medium | Within the week | Affected department head, within the week |
| Types 1, 3, 4, 5 | Low | Next scheduled review | Affected department head if recalibration is needed |

### Severity definitions

| Severity | Definition |
|---|---|
| Critical | Wrong output reached a board, a customer, or triggered a strategic decision |
| High | Wrong output reached reps or managers and may have influenced deal-level decisions |
| Medium | Wrong output was generated but caught before it influenced a decision |
| Low | A potential drift or quality degradation was flagged, no wrong output yet |

### Escalation

| Severity | Escalation |
|---|---|
| Critical | RevOps and affected department head |
| High | RevOps and affected department head |
| Medium | RevOps |
| Low | RevOps |

Document the targets you set. Review them after the first real incident.

---

## Testing This Playbook

Walk the full seven steps at three points.

**At rollout,** before the playbook is in force, with the people who would run it.

**Once a year per failure type.** Running several types in one session is fine.

**After any real incident,** for that failure type, within 15 days.

Detection is tested quarterly and is documented in `agent-monitoring/references/quarterly-exercise-guide.md`. A seeded failure that goes uncaught during a detection round moves that type to the front of the response schedule.

Without a detection layer, run the response walkthrough on its own using `references/tabletop-scenarios.md`.

---

## Writing Rules for Incident Communications

Apply these when drafting a notification or an incident log entry.

- State what happened before explaining why
- Do not minimize. A wrong forecast that reached the board is not a minor data discrepancy.
- Short, direct sentences. The recipient should understand the impact in under 30 seconds.
- Attribute the error to the organization, not the tool. "Our AI scoring model drifted," not "the AI got it wrong."

---

## Reference Files

| File | When to read | What's inside |
|---|---|---|
| `references/incident-log-template.md` | Every incident | Structured log for the full incident lifecycle |
| `references/tabletop-scenarios.md` | Response testing | Pre-written scenarios for each failure type |

## Related Skills

- **agent-monitoring** — detection layer that catches these failures before they reach a decision
- **why-audit** — prevents Type 4 by validating causal claims before they reach decisions
- **semantic-layer-setup** — prevents Types 1 and 3 through documented, versioned metric definitions
