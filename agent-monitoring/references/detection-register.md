# Detection Register

The operating record for every monitoring check running against your AI agents. One row per check. This is where thresholds live, where dispositions accumulate, and where you find out whether a check is actually working or just quiet.

The register is reviewed quarterly alongside the semantic layer review, and updated after every quarterly exercise.

---

## How to Use This File

**What goes in it:** Every active check, regardless of method. Deterministic queries, anomaly detection models, and model-based screens all get an entry. A check that is not in the register is a check nobody owns.

**Where it lives:** A spreadsheet is fine and probably better than a document, because the disposition counts and precision calculations want to be columns. Keep it wherever the person who reviews it will actually open it.

**Who owns it:** RevOps owns the register. Each individual check has its own named owner, who is accountable for its threshold and its performance.

**Update cadence:** Dispositions are logged as flags are resolved, continuously. Precision and performance columns are calculated at quarter end. Threshold changes are logged whenever they happen.

---

## Register Structure

Each check gets a row with these columns.

### Identity

| Column | What to capture |
|---|---|
| **Check ID** | A short unique identifier. `DRIFT-01`, `PERM-03`, `COV-02`. Used to reference the check in incident logs and exercise notes. |
| **Check name** | Plain-language description of what it monitors |
| **Failure type** | Which of the five it detects: Model Drift, Unauthorized Action, Cascading Error, Confident Wrong Narrative, Stale/Partial Data |
| **Agent monitored** | Which agent or model this check watches. If it covers several, list them. |
| **Method** | Deterministic, Anomaly Detection, or Model-Based |
| **Owner** | One named person |

### Configuration

| Column | What to capture |
|---|---|
| **What it measures** | The specific quantity being checked. "Score-to-outcome correlation, trailing 90 days." |
| **Threshold** | The current trigger condition, stated precisely enough that someone else could rebuild it |
| **Run frequency** | Daily, weekly, monthly, on-trigger |
| **Data source** | Which system or query produces the input |
| **Escalation severity** | Critical, High, Medium, or Low. Determines who gets notified and how fast. |
| **Escalation route** | Where the flag goes and who receives it |

### Performance

| Column | What to capture |
|---|---|
| **Times fired** | Count for the trailing quarter |
| **True positives** | Count of flags that were real failures |
| **Known business change** | Count of flags caused by legitimate expected change |
| **Threshold artifacts** | Count of flags where nothing meaningful happened |
| **Unexplained (open)** | Count of flags still without a determined cause |
| **Precision** | True positives divided by times fired, as a percentage |
| **Seeded failures caught** | From the quarterly exercise. Format: caught / attempted. |
| **Median time to fire** | How long between the seeded failure entering the system and the check flagging it |

### Maintenance

| Column | What to capture |
|---|---|
| **Last reviewed** | Date |
| **Last retuned** | Date |
| **Retune direction** | Tightened, Loosened, Scope narrowed, Scope widened |
| **Retune reason** | Why the change was made |
| **Approved by** | Who signed off on the threshold change |
| **Active suppression** | Any open suppression window, with end date |
| **Status** | Active, Tuning, Suppressed, Retired |

---

## Worked Example: A Healthy Check

**DRIFT-01: Pipeline scoring model, score-to-outcome correlation**

| Attribute | Value |
|---|---|
| Check ID | DRIFT-01 |
| Check name | Pipeline scoring model, score-to-outcome correlation |
| Failure type | Model Drift |
| Agent monitored | Pipeline scoring model (production) |
| Method | Deterministic |
| Owner | Director of Sales Operations |
| What it measures | Correlation between score at Stage 2 and eventual closed-won outcome, trailing 90 days |
| Threshold | Flag when correlation drops more than 15% below the value recorded at last calibration |
| Run frequency | Weekly |
| Data source | Salesforce, scheduled report |
| Escalation severity | High |
| Escalation route | Slack alert to #revops-alerts, plus email to check owner |
| Times fired | 3 |
| True positives | 2 |
| Known business change | 1 |
| Threshold artifacts | 0 |
| Unexplained (open) | 0 |
| Precision | 67% |
| Seeded failures caught | 2 / 2 |
| Median time to fire | 6 days |
| Last reviewed | 2026-07-15 |
| Last retuned | 2026-04-10 |
| Retune direction | Tightened |
| Retune reason | Original 25% threshold missed a real drift event that ran for two months. Tightened to 15%. |
| Approved by | Director of Sales Ops, VP RevOps |
| Active suppression | None |
| Status | Active |

**Why this check is healthy:** it fires rarely, most flags are real, it catches seeded failures, and the one non-true-positive was a correctly identified business change rather than noise. The tightening in April was driven by a documented miss, which is the right reason to change a threshold.

---

## Worked Example: A Check That Needs Attention

**ANOM-02: Sequence exit rate deviation**

| Attribute | Value |
|---|---|
| Check ID | ANOM-02 |
| Check name | Sequence exit rate deviation from baseline |
| Failure type | Cascading Error |
| Agent monitored | Outbound sequencing agent, enrichment integration |
| Method | Anomaly Detection |
| Owner | Marketing Operations Manager |
| What it measures | Rate at which records exit an active sequence, compared against a rolling 90-day baseline by segment |
| Threshold | Flag when daily exit rate exceeds 2 standard deviations from baseline |
| Run frequency | Daily |
| Data source | HubSpot, anomaly detection model |
| Escalation severity | Medium |
| Escalation route | Slack alert to #marketing-ops |
| Times fired | 34 |
| True positives | 4 |
| Known business change | 9 |
| Threshold artifacts | 18 |
| Unexplained (open) | 3 |
| Precision | 12% |
| Seeded failures caught | 1 / 2 |
| Median time to fire | 2 days |
| Last reviewed | 2026-07-15 |
| Last retuned | Never |
| Retune direction | N/A |
| Retune reason | N/A |
| Approved by | N/A |
| Active suppression | None |
| Status | Tuning |

**Why this check needs attention:** precision at 12% is below the 15% retirement threshold, and 18 threshold artifacts means the sensitivity is set too tight for normal variance. It also missed one of two seeded failures, so loosening the threshold is risky without understanding why it missed. Nine known business changes suggests suppression windows were never used during launches or territory changes. Three open unexplained flags need resolution before any retuning decision, because they might be real failures that would justify the current sensitivity.

**The action:** resolve the open unexplained flags first. Implement suppression windows for planned changes. Then retune, and confirm it still catches both seeded failures at the new setting.

---

## Reading the Register

**Precision alone is not health.** A check with 95% precision that catches zero seeded failures has stopped detecting anything. Always read precision alongside seeded failures caught. Both columns exist because either one alone is misleading.

**High threshold artifact counts mean the threshold is wrong.** Not the concept, the number. Retune rather than retire.

**High known-business-change counts mean suppression is not being used.** The check is working correctly and detecting real deviation. The problem is process, not configuration. Build suppression windows into change management.

**Open unexplained flags are the most important column.** They are the ones nobody could resolve, and they get closed as noise by default when a register is not reviewed carefully. A rising unexplained count is a signal that either the check is producing incomprehensible output or nobody has time to investigate. Both need addressing.

**A check that has never fired is unknown, not healthy.** Test it before assuming it works.

**Never retuned plus never tested equals unverified.** A check with blank retune history and no seeded failure results is a check nobody has confirmed does anything.

---

## Quarterly Review Checklist

Run at quarter end, alongside the semantic layer review.

- [ ] Precision calculated for every check
- [ ] Every open unexplained flag has been revisited and either resolved or escalated
- [ ] Any check below 30% precision for two consecutive quarters is scheduled for retuning
- [ ] Any check below 15% precision is retired or rebuilt
- [ ] Seeded failure results from the quarterly exercise entered for every check tested
- [ ] Every check with a blank seeded-failure result is queued for the next exercise
- [ ] Expired suppression windows closed, and accumulated suppressed flags reviewed
- [ ] Every threshold change since last quarter has a logged reason and approver
- [ ] Any check loosened in two or more consecutive quarters flagged for review
- [ ] Every check has a current owner who still works here and still owns it
- [ ] New agents deployed this quarter have corresponding checks in the register

---

## Retirement Criteria

Retire a check when any of these is true:

- Precision has stayed below 15% after at least one retuning attempt
- The agent it monitors has been decommissioned
- Another check covers the same failure mode more reliably
- It has never fired and has failed seeded-failure testing twice

**Retire rather than ignore.** A check that fires into a channel nobody reads is worse than no check, because it appears in the register as coverage that does not exist. Set status to Retired, keep the row for history, and remove the alert routing.
