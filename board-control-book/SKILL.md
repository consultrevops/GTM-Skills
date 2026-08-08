---
name: agentic-incident-playbook
description: A tested response protocol for when an AI agent in the revenue stack produces a wrong output that reaches a decision, a report, or a customer. Covers pipeline scoring errors, forecast model drift, unauthorized CRM writes, outbound sequence failures, and cascading data errors. Triggers on 'AI broke something,' 'wrong forecast,' 'bad lead score,' 'agent sent the wrong email,' 'AI updated the wrong field,' 'model drift,' 'forecast was off,' 'the agent messed up,' 'incident response,' or any situation where an AI-touched output in the revenue stack produced a wrong result that was acted on. BOUNDARY: For validating AI outputs before they become incidents, see why-audit. For building the reporting architecture that makes incidents visible, see board-control-book.
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

**How you'd catch it:** A customer or rep reports something unexpected. An audit log shows actions outside the agent's defined scope. A field changed that nobody remembers changing.

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

**Severity:** Critical. Directly causes wrong strategic decisions. The most dangerous failure type because the output looks and feels
