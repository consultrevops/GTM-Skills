---
name: agent-monitoring
description: >
  Builds and operates the detection layer that catches AI agent failures
  in a revenue stack before they reach a decision, a report, or a
  customer. Covers drift detection, unauthorized action monitoring,
  anomaly detection, narrative quality screening, and coverage
  verification. Triggers on agent monitoring, AI observability, model
  drift, is my AI working, how do I know if the agent is wrong, monitoring
  AI outputs, agent guardrails, detection register, seeded failure test,
  or any request about verifying that AI in the revenue stack is still
  producing trustworthy output. BOUNDARY - For responding to a failure
  that already reached a decision, see agentic-incident-playbook. For the
  agent permission baseline this skill compares against, see
  non-human-identity. For metric definitions, see semantic-layer-setup.
---

# Agent Monitoring

You are helping someone build or operate detection for the AI agents in their revenue stack.

Two modes. Determine which applies before proceeding.

**Build** if checks do not exist, or if a new agent has been deployed without coverage. Inventory review, failure type mapping, check design, threshold setting, register population.

**Operate** if checks are running. Flag context assembly, disposition support, clustering, quarterly register analysis, seeded failure generation.

Ask which applies if it is unclear.

---

## The Five Failure Types

| Type | What it is | Detection method |
|---|---|---|
| 1. Model drift | Scoring, weighting, or classification has shifted from its calibration. The model did not change, the data did. | Deterministic |
| 2. Unauthorized action | The agent acted outside its defined scope. Wrote a field it should not touch, sent a communication without approval, acted on records outside its population. | Deterministic |
| 3. Cascading error | One wrong output fed the next step, which fed the next, amplifying the original error. | Anomaly detection |
| 4. Confident wrong narrative | The agent produced a causal explanation that was wrong but sounded right, used before anyone validated it. | Model-based |
| 5. Stale or partial data | The agent answered a question spanning a large dataset but processed a fraction of it, with no coverage disclosure. | Deterministic |

## Three Detection Methods

**Deterministic.** Numbers compared against numbers. A query, a report, a correlation. No model involved. Four of five failure types belong here.

**Anomaly detection.** Baseline deviation surfaced without a rule written in advance. Statistical modeling, not generative AI.

**Model-based.** A language model evaluating unstructured text at volume. Only Type 4 requires this.

**The rule:** the monitor is simpler than what it monitors, and it checks against ground truth rather than another model's judgment. Never label a scheduled query as AI.

---

## Build Mode

### Step 1: Check prerequisites

Establish and report on each before designing anything:

| Prerequisite | What it supplies | Source |
|---|---|---|
| User repository, agent fields populated | `permitted_write_fields`, `scoped_population`, `external_actions` | `non-human-identity` |
| Field documentation | Field-level AI read and write access | `semantic-layer-setup` |
| Metric definitions | What drift and coverage checks compare against | `semantic-layer-setup` |
| System access | Field history, audit logs, score-to-outcome history | The platforms themselves |

**State which are missing.** Design checks where the inputs exist. Mark the rest blocked and name the specific missing input.

Without `permitted_write_fields` and `scoped_population`, Type 2 checks cannot be built at all. If those are blank, route to `non-human-identity` before proceeding.

### Step 2: Map agents to failure types

Read `likely_failure_types` from the repository for each agent. Where it is blank, propose a mapping based on what the agent does:

- Scores, classifies, or forecasts → Type 1
- Has any `permitted_write_fields` or `external_actions` → Type 2
- Feeds another automated step → Type 3
- Produces causal explanations → Type 4
- Analyzes datasets larger than a context window → Type 5

Most agents carry two or three. Propose, do not write to the repository.

### Step 3: Design checks

One check per failure shape, not one per agent. For each agent and failure type, ask what specifically could go wrong and design a check for each answer.

**Type 1, drift.** Compare predictions against realized outcomes on a rolling window. Score-to-outcome correlation against the correlation at last calibration. Distribution shift in model inputs against the calibration distribution. Human override rate.

**Type 2, unauthorized action.** Three distinct checks, because they catch different things:
- Field history for writes outside `permitted_write_fields`
- Records written to outside `scoped_population`
- Write volume against the trailing average

An agent writing permitted fields to the wrong records passes the first check and fails the second. Both are needed.

**Type 3, cascading error.** Baseline record flow through workflows, then flag deviation. Records exiting a stage or sequence outside normal range. Enrichment match rate changes. Unusual attribute clusters in a short window. Records removed from pipeline by automation rather than a human.

**Type 4, confident wrong narrative.** Screen AI-generated causal explanations against the markers in `why-audit/references/false-confidence-markers.md` before they reach a deck or a decision. Flag for human review. Never clear a claim.

**Type 5, stale or partial data.** Coverage ratio of records processed divided by records matching the query scope. Date range covered against range requested. Integration health at time of analysis.

### Step 4: Propose thresholds

Propose a starting threshold with the reasoning stated. Do not set one silently.

Reference points, not defaults:
- Drift: correlation drop beyond 15% from calibration baseline, or override rate above 15% in a rolling month
- Unauthorized action: any single out-of-scope write. Volume at 3x trailing 30-day average.
- Coverage: below 80% of available population

**Ask the user to confirm or adjust.** How much drift is tolerable before someone should act is a business judgment. State that when proposing.

### Step 5: Assign severity and routing

| Severity | Meaning | Routing |
|---|---|---|
| Critical | Out-of-scope agent action, or a flagged output that already reached a decision | Monitoring channel, plus DM and email to two named contacts |
| High | Drift beyond threshold, or coverage below minimum on an output in use | Monitoring channel, plus DM and email to two named contacts |
| Medium | Anomaly flagged, cause undetermined | Monitoring channel |
| Low | Threshold approached, not crossed | Monitoring channel |

Every flag notifies immediately. Two named contacts, not one.

### Step 6: Populate the register

Fill `references/detection-register.md` for each check. Ask for owner. Do not infer one.

Report back which agents in the repository still have no covering check.

---

## Operate Mode

### On a flag: assemble context

This is the highest-value task in this skill. Do it before a human opens the flag.

Assemble and present:

1. **What fired.** Check ID, severity, threshold crossed, by how much.
2. **What it touched.** The records involved, with enough detail to recognize them.
3. **Deviation size.** Against baseline, and against the last several periods.
4. **This check's history.** The last several times it fired and how each was dispositioned.
5. **Context object overlap.** Query the context object for entries whose `effective_date` and `affected_segment` overlap this flag. Report any match with its confidence level.
6. **What else moved.** Other checks that fired in the same window, other changes in the same population.
7. **Proposed disposition,** with the reasoning.

**Where a context object entry overlaps,** propose Known business change and cite the entry, its confidence level, and its effective date. Where the entry is logged Speculative, say so and do not present the match as settled.

**Where nothing explains it,** propose Unexplained. Do not construct a plausible cause to fill the gap. That is the failure this whole repository exists to prevent.

### On multiple flags: cluster

During a business change or a real incident, one cause produces many flags. Group by likely common cause and present clusters rather than a list.

For each cluster: the flags in it, the proposed common cause, and the evidence. A human dispositions the cluster.

State clustering confidence. Flags grouped on a shared timestamp and population are a stronger cluster than flags grouped on timing alone.

### Quarterly: register analysis

Produce ahead of the review, delivered with the seeded failure data.

Read the register as a dataset and report:

- **Precision per check,** with any check crossing a retuning or retirement threshold flagged and the specific rule named
- **Checks loosened in two or more consecutive quarters.** Invisible in any single quarter. This is the pattern the sensitivity guardrail exists to catch.
- **Checks with blank seeded-failure results.** The register claims coverage nobody verified.
- **Rising unexplained counts,** which point at either output nobody can interpret or nobody having time to investigate
- **High known-business-change counts,** which point at suppression not being used rather than a threshold being wrong
- **Owners who have left or changed roles,** cross-referenced against the directory
- **Agents in the repository with no covering check.** The coverage question the register cannot answer about itself.

Present as items requiring a decision, each with what it points to and the rule or trend that surfaced it.

### Quarterly: seeded failure generation

Generate test data for one failure of each of the five types, sized and shaped to be realistic for this environment.

For each: what is being seeded, which check should catch it, the expected detection window, and how to remove it afterward.

**You generate. A human seeds.** Produce the test data, the insertion instructions, the expected detection window, and the removal steps. A human inserts it, confirms it landed where intended, and removes it afterward. Do not write test data to any system, sandbox included.

### On request: suppression windows

When a planned change is described or found in the context object, propose a suppression window: which checks it would trip, the affected population, a start date, and an end date.

**Scope narrowly.** Suppress the affected metric and population, not the whole check.

**Critical never suppresses.** An out-of-scope write escalates during a product launch the same as any other day.

**Every window gets an end date.** Estimate it and say it is an estimate. A window without an end outlasts the change it was opened for.

---

## The Quarterly Detection Test

Detection untested against a seeded failure is unverified.

**Cadence:** quarterly, all five failure types.

**Environment:** sandbox with seeded data. Where a system has no sandbox, note it and widen the notice.

**Sequence:**

1. Send the pre-test notification
2. A human seeds all five failures using the generated data and instructions
3. Wait for each check's normal run cycle
4. Record what fired, within what window, at what severity
5. Record any check that fired that should not have
6. Diagnose every miss before moving on
7. Enter all results in the register under seeded failures caught, including misses
8. Send the post-test notification

**For every miss, establish:** was the threshold too loose, was the seeded failure realistic enough to catch, did the check run at all, and has this check ever fired on a real failure.

**A missed seeded failure moves that type to the front of the incident protocol's response schedule.** See `agentic-incident-playbook`.

### Notifications

**Pre-test.** Sent to anyone whose work depends on an agent being tested. States what is being tested, when, whether sandbox or production, who to contact if something unusual appears, and when results will follow.

**Post-test.** States how many of five were caught and names any missed, what gaps surfaced, what is changing with owners and dates, and whether anything the recipient relies on is affected.

**If the test surfaces something real,** meaning an agent has actually been producing wrong output in production, that is an incident. Route to `agentic-incident-playbook` and notify everyone affected with how long it has been happening and the correction timeline.

**All Notification Drafts must be approved by monitoring owner first.**

---

## Constraints

**Propose dispositions. Never apply them.** Dispositions are the labels the anomaly detection feedback loop learns from. Inaccurate labels applied at volume teach the model to ignore the wrong things.

**Never set a threshold without stating it is a proposal.** How much deviation is tolerable is a business judgment.

**Never infer a check owner.** Ask.

**Label every detection method accurately.** Deterministic, anomaly detection, or model-based. A scheduled query is a scheduled query.

**Report precision and seeded failure catch rate together.** Either alone is misleading. A check with 95% precision that catches no seeded failures has stopped detecting anything.

**Say unknown when a cause is unknown.** Never construct a plausible cause to close a flag.

**Never write to the user repository.** Propose repository updates. A human applies them.

**Never insert seeded failure data into any system.** You generate the data and the instructions. A human performs the insertion and the removal. This holds for sandboxes as well as production.

---

## Reference Files

| File | When to read | What's inside |
|---|---|---|
| `references/detection-register.md` | Build Step 6, and all quarterly analysis | Register structure, worked examples, reading guide, retirement criteria |

## Related Skills

- **non-human-identity** — supplies the agent permission baseline. Type 2 checks cannot be built without `permitted_write_fields` and `scoped_population`.
- **agentic-incident-playbook** — response protocol when a flagged output reached a decision
- **semantic-layer-setup** — metric definitions and field-level access documentation
- **why-audit** — false confidence markers used in Type 4 screening
- **causal-model-setup** — the context object this skill cross-references when dispositioning flags
