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

**Threshold example:** Flag when score-to-outcome correlation drops more than 15% from the calibration baseline, or when override rate exceeds 15% in a rolling month.

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

**Caution:** Anomaly detection produces false positives during legitimate business changes. A new product launch, a territory reassignment, or a pricing change will trip these. Every flag needs a human to distinguish a real error from a real change. See Threshold Governance and False Positive Reduction below.

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

## Threshold Governance and False Positive Reduction

Every detection method has a threshold set by a human. Thresholds decay. And a check that fires constantly gets ignored, which makes it functionally worse than no check at all.

The goal is fewer flags without fewer catches. Those pull against each other, and managing that tension is the whole job of this section.

### Disposition Every Flag

No flag closes without a disposition. This is the input that everything else in this section depends on.

| Disposition | Definition | What it triggers |
|---|---|---|
| **True positive** | A real failure. The agent produced a wrong output or took a wrong action. | Move to escalation. If it reached a decision, `agentic-incident-playbook`. |
| **Known business change** | The deviation is real but expected. A launch, a reorg, a territory shift, a pricing change. | Log the change. Consider a suppression window (below). Feed the label back to the baseline. |
| **Threshold artifact** | Nothing meaningfully changed. The threshold is too tight for normal variance. | Retune the threshold. Do not suppress. |
| **Unexplained** | Cause could not be determined within the review window. | Keep open. Review at the next scheduled cycle. Never close as false positive by default. |

**Rule:** unexplained is not the same as false positive. Closing unknowns as noise is how a monitoring layer trains itself into blindness. Track them separately and revisit them.

### The Feedback Loop

This applies to anomaly detection specifically. Deterministic checks do not learn, they get retuned by a human.

Anomaly detection produces most of the false positive volume, because legitimate business change and real error look structurally similar to a baseline model. The fix is supervised: dispositions become labels, labels update the baseline.

**How it works:**
- Every flag carries its disposition back to the model
- Patterns labeled "known business change" stop registering as anomalous when they recur in a similar shape
- Patterns labeled "true positive" increase sensitivity to that shape
- The baseline recalculates on a defined cadence rather than continuously, so a burst of unusual activity does not silently become the new normal before anyone reviews it

**Expect a tuning period.** The first 60 to 90 days after deploying anomaly detection will produce a high false positive rate. This is not a malfunction. The model has no labeled history yet. Budget human review time accordingly and resist the urge to loosen thresholds during this window, because premature loosening bakes in blind spots before the model has learned anything.

**What the loop cannot fix:** if dispositions are applied carelessly, the model learns carelessly. Garbage labels produce a model that confidently ignores real failures. Whoever dispositions flags needs enough context to distinguish a real error from a real change, which usually means RevOps rather than a rotating queue.

### Suppression Windows

The cheapest false positive reduction available, and it requires no model at all.

When a business change is planned, tell the system to expect deviation in that window instead of letting it flag daily for two weeks.

**When to open a suppression window:**
- Product launches and sunsets
- Pricing or packaging changes
- Territory or segment reassignment
- Comp plan changes
- Large data migrations or CRM configuration changes
- Seasonal patterns with known historical shape

**Rules for suppression:**
- Every window has a start date, an end date, and a named owner. Open-ended suppression is how monitoring quietly turns off.
- Suppression narrows scope, it does not disable the check. Suppress the specific metric or record population affected, not the whole detection method.
- Critical severity never suppresses. An out-of-scope agent write escalates during a product launch the same as any other day.
- Suppressed flags are still logged, just not escalated. When the window closes, review what accumulated. If something in there was a real failure, that is a lesson about how the window was scoped.

**Best practice:** build suppression into the change management process. When a launch date is set, the suppression window gets opened at the same time, by the same person, with the same end date.

### Check Performance Register

Track for every check:

| Field | What to capture |
|---|---|
| Check name | What it monitors |
| Failure type | Which of the five it detects |
| Method | Deterministic, anomaly detection, or model-based |
| Threshold | Current trigger condition |
| Owner | One named person |
| Last reviewed | Date |
| Last retuned | Date, with what changed and why |
| Times fired | Count, trailing quarter |
| True positives | Count |
| Known business change | Count |
| Threshold artifacts | Count |
| Unexplained | Count, currently open |
| Precision | True positives divided by total fired |
| Seeded failures caught | From the quarterly exercise, out of how many attempted |

**Precision is the false positive metric.** A check firing 40 times with 8 true positives has 20% precision. Track it per check, not in aggregate, because one noisy check will drag an otherwise healthy suite below any threshold you set.

### Retuning Rules

- **Precision below 30% for two consecutive quarters:** retune the threshold or narrow the check's scope. Document what changed.
- **Precision below 15%:** the check is actively harmful. Retire it or rebuild it. People have already stopped reading its alerts.
- **Precision above 90% with low fire volume:** the threshold may be too tight in the other direction. Verify the check still catches seeded failures before assuming it is well-tuned.
- **A check that has never fired:** unknown state, not a healthy one. It gets tested in the next quarterly exercise before anyone concludes it works.

### The Sensitivity Guardrail

Every threshold loosened to reduce noise is a real failure that might now slip through. A monitoring layer optimized only for fewer alerts will eventually optimize into catching nothing, and its precision metric will look excellent the entire time.

**Three rules prevent this:**

1. **Never retune a threshold without running the seeded-failure test at the new setting.** If the check no longer catches the test failure, the new threshold is wrong regardless of what it did to precision.

2. **Track seeded failures caught alongside precision.** Precision measures noise. Seeded failure catch rate measures whether the check still works. A check with 95% precision that misses seeded failures is a check that stopped detecting anything.

3. **Retuning is directional and reviewed.** Every threshold change is logged with what changed, why, and who approved it. A threshold that has been loosened three quarters in a row is a pattern worth questioning, and that pattern is only visible if the changes are logged.

**The failure mode this prevents:** a quarter where flags dropped 60%, the team celebrates less noise, and nobody notices that two real drift events passed through unflagged. The seeded-failure test is the only thing that surfaces this before an incident does.

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

**Step 3: Map each agent to its likely failure types.** A scoring model risks drift. A sequencing agent risks unauthorized actions. An analysis agent risks confident wrong narrative and partial data. Some agents carry two or three.

**Step 4: Build the deterministic checks first.** They cover the majority of failure modes, cost the least to build, and produce the fewest false positives. Drift correlation, permission auditing, coverage ratios.

**Step 5: Add anomaly detection where rules cannot reach.** Workflow deviation, enrichment quality, unexpected record clusters. Expect a tuning period.

**Step 6: Add model-based screening only where text must be read.** Narrative quality checks on causal explanations. Nothing else needs a model.

**Step 7: Set thresholds, assign owners, schedule the review.** Every check gets a person and a date.

**Step 8: Run the quarterly exercise.** Until detection has been tested against a seeded failure, you have monitoring on paper.

---

## Common Failure Modes

**The green dashboard.** Every check shows healthy and nobody has verified any of them can catch anything. This is the same problem as an untested incident plan. It provides the feeling of coverage and removes the urgency to look manually. Only the quarterly exercise resolves it.

**Correlated blindness.** The monitor and the monitored system read from the same source. Bad enrichment data feeds both the scoring model and the check watching it, and both see the same wrong value. Mitigate by checking against realized outcomes rather than against upstream data.

**Alert fatigue.** Too many flags, and people stop reading them. Track precision per check and retire anything below the retuning threshold.

**Threshold rot.** A check calibrated eighteen months ago against a business that no longer exists. Quarterly review with named owners is the only fix.

**Monitoring without escalation.** The check fires into a channel nobody watches, or flags a problem with no named owner. Detection without a defined escalation path is logging, not monitoring.

---

## Voice Rules

- Label every detection method as deterministic, anomaly detection, or model-based. Never describe a scheduled query as AI.
- State precision and seeded failure catch rate when reporting on a check's performance. Never report one without the other.
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
