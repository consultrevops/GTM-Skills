---
name: agent-monitoring
description: >
  Builds the detection layer that catches AI agent failures in a revenue
  stack before they reach a decision, a report, or a customer. Covers
  drift detection, unauthorized action monitoring, anomaly detection,
  narrative quality checks, and data coverage verification. Triggers on
  agent monitoring, AI observability, model drift detection, is my AI
  working, how do I know if the agent is wrong, monitoring AI outputs,
  AI quality checks, agent guardrails, or any request about verifying
  that AI systems in the revenue stack are still producing trustworthy
  outputs. BOUNDARY - For responding to a failure that already reached a
  decision, see agentic-incident-playbook. For the metric definitions
  this skill depends on, see semantic-layer-setup.
---

# Agent Monitoring — Catching Failures Before They Reach a Decision

You are helping someone build continuous detection for the AI agents operating in their revenue stack. The goal is catching a wrong output while it is still an output, before it becomes a decision, a board slide, or an email a customer received.

This skill covers detection. For what happens after a failure reaches a decision, see `agentic-incident-playbook/SKILL.md`.

---

## Prerequisite

**This skill requires a semantic layer.** Drift detection compares model outputs against documented metric definitions. Coverage checks require knowing which fields feed which metric. Unauthorized action monitoring requires a documented list of which fields each agent is permitted to touch.

Without documented definitions, monitoring has nothing to compare against. You will be measuring an agent's output against an undefined standard, which produces alerts nobody can act on.

Build `semantic-layer-setup/SKILL.md` first.

---

## The Honest Split Between Automation and AI

Most of what gets sold as "AI monitoring" is a scheduled query with a marketing label. This skill labels each detection method for what it actually is.

**Deterministic checks** compare numbers against numbers. A scheduled report, a query, a correlation calculation. These do not drift, do not hallucinate, and do not require a model. Most monitoring belongs here.

**Anomaly detection** surfaces patterns that deviate from an established baseline without someone having written a rule in advance. This is classical statistical modeling, not generative AI, and it catches failure modes nobody anticipated.

**Model-based reading** uses AI to evaluate unstructured text at volume. Sentiment across thousands of transcripts, tone drift in generated outbound copy, causal claims in a written analysis. This is the only category that genuinely requires a language model.

The rule underneath all three: **the monitor should be simpler than what it monitors, and it should check against ground truth rather than against another model's judgment.** A generative model evaluating another generative model's opinion is circular. A deterministic check comparing predicted scores against deals that actually closed is not.

---

## What to Monitor

The five failure types from `agentic-incident-playbook` viewed earlier in their lifecycle. Each maps to a specific detection method.

### Type 1: Model Drift

**Detection method:** Deterministic. No AI required.

**What to run:** Compare model predictions against realized outcomes on a rolling basis. For a pipeline scoring model, correlate scores at a point in time against whether those deals actually closed. For a forecast model, compare forecasted to actual by category.

**What to watch:**
- Score-to-outcome correlation, trailing 90 days, compared against the correlation at last calibration
- Distribution shift in model inputs (average deal size, segment mix, product mix) against the distribution the model was calibrated on
- Rate at which humans override the model's output

**Threshold example:** Flag when score-to-outcome correlation drops more than 15% from the calibration baseline, or when override rate exceeds 25% in a rolling month.

**Why this catches it early:** Drift produces no dramatic failure. Each individual output looks plausible. Only the aggregate reveals the problem, which is exactly what a rolling statistical check sees and a human reviewer does not.

### Type 2: Unauthorized or Unintended Actions

**Detection method:** Deterministic. No AI required.

**What to run:** Query field history and agent audit logs against a documented permission list.

**What to watch:**
- Any write by an agent to a field outside its documented scope
- Volume of writes per agent per day, compared against its normal range
- Records touched outside the agent's expected population (e.g., an agent scoped to new leads touching closed-won accounts)
- Communications sent outside approved sequences or templates

**Threshold example:** Any single out-of-scope write escalates immediately. Volume anomalies flag at 3x the trailing 30-day daily average.

**Requirement:** This only works if each agent has a documented list of permitted fields and actions. That list lives in the semantic layer field documentation under the AI access column.

### Type 3: Cascading Data Errors

**Detection method:** Anomaly detection. Statistical modeling, not generative AI.

**What to run:** Baseline the normal flow of records through your workflows, then flag deviations from that baseline.

**What to watch:**
- Records exiting a stage or sequence at a rate outside the normal range
- Sudden changes in enrichment match rates or field-fill rates
- Clusters of records with an unusual combination of attributes appearing in a short window
- Deals removed from active pipeline by automation rather than by a human

**Why anomaly detection rather than rules:** Cascading errors are, by definition, failures nobody predicted. A rule catches what you already thought to check for. Baseline deviation catches the shape of a problem you have not seen before.

**Caution:** Anomaly detection produces false positives during legitimate business changes. A new product launch, a territory reassignment, or a pricing change will trip these. Every flag needs a human to distinguish a real error from a real change. See Threshold Governance below.

### Type 4: Confident Wrong Narrative

**Detection method:** Model-based reading. Requires a language model.

**What to run:** Screen AI-generated causal explanations against known markers of unverified claims before the explanation reaches a deck, a decision, or a coaching conversation.

**What to watch:**
- Single-cause certainty when the underlying data supports multiple explanations
- A stated mechanism with no supporting data point
- Narrative coherence standing in for evidence
- Citations to benchmarks or external authority that cannot be verified

The full marker list lives in `why-audit/references/false-confidence-markers.md`.

**The limit worth naming:** this is a model reading another model's output, which is the circularity this skill otherwise avoids. It works because the screen is checking for structural patterns in the text rather than judging whether the claim is true. It flags for human review. It never clears a claim on its own.

### Type 5: Stale or Partial Data Reasoning

**Detection method:** Deterministic. No AI required.

**What to run:** Compare the volume of data an AI analysis actually processed against the volume available for the question asked.

**What to watch:**
- Coverage ratio: records cited or processed divided by records matching the query scope
- Recency: the date range the analysis actually covered against the range requested
- Integration health: whether every expected data source was connected and returning data at the time of the analysis

**Threshold example:** Flag any analysis where coverage falls below 80% of the available population, and require the coverage figure to be stated alongside any output used in a decision.

**Why this matters:** an agent that keyword-sampled 20 transcripts out of 4,000 produces output identical in tone and structure to one that read everything. The coverage ratio is the only signal that distinguishes them.

---

## Escalation

Detection is only useful if something happens when it fires.

| Detection severity | What it means | Who acts | Timeline |
|---|---|---|---|
| Critical | Out-of-scope agent action, or a flagged output already reached a decision | Affected department head plus RevOps, immediately | Same day. Move to `agentic-incident-playbook`. |
| High | Drift beyond threshold, or coverage below minimum on an output in active use | RevOps plus the model owner | Within 48 hours |
| Medium | Anomaly flagged, cause not yet determined | RevOps | Within one week |
| Low | Threshold approached but not crossed, or a single unexplained flag | RevOps, logged for pattern review | Next scheduled review |

**The handoff point:** monitoring stops and incident response begins the moment a wrong output has influenced a decision, reached a customer, or entered a report. Everything before that is detection. Everything after is `agentic-incident-playbook`.

---

## Threshold Governance

Every detection method has a threshold set by a human. Thresholds decay.

**Rules:**
- Every threshold has a named owner and a last-reviewed date
- Thresholds are reviewed quarterly alongside the semantic layer review
- Every check tracks how often it fired and how often the flag was a real problem
- A check with a false positive rate above 50% gets retuned or retired. A monitor people ignore is functionally not a monitor.
- A check that has never fired is either well-tuned or broken. The quarterly exercise is how you find out which.

**Track for each check:**

| Field | What to capture |
|---|---|
| Check name | What it monitors |
| Failure type | Which of the five it detects |
| Method | Deterministic, anomaly detection, or model-based |
| Threshold | Current trigger condition |
| Owner | One named person |
| Last reviewed | Date |
| Times fired | Count, trailing quarter |
| True positives | Count of flags that were real |
| False positive rate | Calculated |

---

## The Quarterly Exercise

**This exercise covers both this skill and `agentic-incident-playbook`.** Detection and response are tested together, in one session, because a monitor that fires into a response nobody knows how to run is not protection.

Run it in a test environment with seeded data. Never in production.

**What you are testing, in order:**

1. **Does detection fire?** Seed a failure of one type into the test environment. A drifted score set, an out-of-scope field write, a coverage gap. Confirm the check catches it and within what window.

2. **Does it escalate correctly?** Confirm the flag reaches the right person, at the right severity, through the channel it is supposed to use.

3. **Does the response protocol run?** Walk the seven steps from `agentic-incident-playbook` with everyone who would be involved in a real incident. Time each step.

4. **Where are the gaps?** Note every hesitation, every unclear ownership, every point where the protocol assumes something exists that does not.

**Cadence:** Quarterly, rotating through the five failure types so each gets tested at least annually. After any real incident, run a variation of that failure type within 30 days.

**What good looks like:** detection fires within its stated window, the right person is notified without anyone asking who owns it, and the response protocol runs without someone having to reread the document mid-incident.

---

## Building the Monitoring Layer

**Step 1: Confirm the semantic layer exists.** Metric definitions, field documentation, and the AI access column. Without these, skip to `semantic-layer-setup`.

**Step 2: Inventory every agent.** What it does, which systems it touches, what it reads, what it writes, who owns it. Anything that scores, enriches, sequences, forecasts, or generates counts.

**Step 3: Map each agent to its likely failure types.** A scoring model risks drift. A sequencing agent risks unauthorized actions. An analysis agent risks confident wrong narrative and partial data. Most agents carry two or three.

**Step 4: Build the deterministic checks first.** They cover the majority of failure modes, cost the least to build, and produce the fewest false positives. Drift correlation, permission auditing, coverage ratios.

**Step 5: Add anomaly detection where rules cannot reach.** Workflow deviation, enrichment quality, unexpected record clusters. Expect a tuning period.

**Step 6: Add model-based screening only where text must be read.** Narrative quality checks on causal explanations. Nothing else needs a model.

**Step 7: Set thresholds, assign owners, schedule the review.** Every check gets a person and a date.

**Step 8: Run the quarterly exercise.** Until detection has been tested against a seeded failure, you have monitoring on paper.

---

## Common Failure Modes

**The green dashboard.** Every check shows healthy and nobody has verified any of them can catch anything. This is the same problem as an untested incident plan. It provides the feeling of coverage and removes the urgency to look manually. Only the quarterly exercise resolves it.

**Correlated blindness.** The monitor and the monitored system read from the same source. Bad enrichment data feeds both the scoring model and the check watching it, and both see the same wrong value. Mitigate by checking against realized outcomes rather than against upstream data.

**Alert fatigue.** Too many flags, and people stop reading them. Track false positive rates and retire checks that cry wolf.

**Threshold rot.** A check calibrated eighteen months ago against a business that no longer exists. Quarterly review with named owners is the only fix.

**Monitoring without escalation.** The check fires into a channel nobody watches, or flags a problem with no named owner. Detection without a defined escalation path is logging, not monitoring.

---

## Voice Rules

- Label every detection method as deterministic, anomaly detection, or model-based. Never describe a scheduled query as AI.
- State the false positive rate when reporting on a check's performance
- When a flag's cause is unknown, say unknown. Do not assign a likely cause without evidence.

---

## Reference Files

| File | When to read | What's inside |
|---|---|---|
| `references/detection-checks-register.md` | First-time setup | Template for logging every check with threshold, owner, and performance |
| `references/agent-inventory-template.md` | Step 2 | Template for cataloging every agent and its permissions |
| `references/quarterly-exercise-guide.md` | Quarterly | Full run sheet for the combined detection and response exercise |

## Related Skills

- **agentic-incident-playbook** — the response protocol that runs when detection escalates to a real incident. Shares the quarterly exercise with this skill.
- **semantic-layer-setup** — required prerequisite. Provides the metric definitions and field permissions this skill monitors against.
- **why-audit** — supplies the false confidence markers used in Type 4 detection
