# Metric Glossary Template

This is the canonical record of how every revenue metric in your organization is calculated. One entry per metric. It is not recommended to put metrics in a board deck, dashboard, commission calculation, or AI prompt without an entry here.

Copy the blank template below for each metric. Two worked examples follow.

---

## How to Use This File

**Where it lives:** Anywhere your team can maintain it and your AI tools can reference it. A shared doc, a wiki page, a Notion database, or a file in your repo. The format matters less than the maintenance.

**Who owns it:** One person owns the file overall. Each metric has its own owner who approves changes to that definition.

**When to add an entry:** Any metric that appears in a board deck, an executive dashboard, a CRM report, a commission calculation, a team scorecard, a health score, or an AI prompt.

**When to update an entry:** Any time the calculation changes, a source field changes, or the inclusion/exclusion rules change. Log it in the changelog at the bottom.

**Review cadence:** A quarterly cadence is suggested for organizations actively changing product, pricing, or reporting. Twice a year is recommended for organization with less consistent changes.

---

## Blank Template

### [Metric Name]

| Field | Value |
|---|---|
| **Metric name** | |
| **Aliases** | |
| **Definition** | |
| **Formula** | |
| **Numerator** | |
| **Denominator** | |
| **Inclusions** | |
| **Exclusions** | |
| **Time logic** | |
| **Source system** | |
| **Source fields** | |
| **Owner** | |
| **Last reviewed** | |
| **Known conflicts** | |

---

## Field Definitions

**Metric name.** The official name as used in board reporting and internal discussion. Keep the naming conventions consistent everywhere.

**Aliases.** Every other name people use for this same metric. "Logo churn," "customer churn," and "churn rate" might all mean the same thing in your org, or they might mean three different things. List every variant you have heard so AI tools can map them to the canonical definition, and so you catch cases where the same word means different things to different teams.

**Definition.** Plain language, one or two sentences. What does this metric measure and why does the business track it. Written so a new hire understands it without needing the formula.

**Formula.** The actual calculation. Not "revenue minus costs." Write the real arithmetic with the real components. If it involves filters or conditional logic, include them.

**Numerator.** What sits on top of the fraction, with the specific fields that produce it. Leave blank for metrics that are not ratios.

**Denominator.** What sits on the bottom, with the specific fields. Leave blank for metrics that are not ratios.

**Inclusions.** What is explicitly counted. Be specific about record types, statuses, and populations. "All closed-won opportunities with a close date inside the reporting period."

**Exclusions.** What is explicitly not counted, and ideally why. This is the field that prevents the most disagreements, because exclusions are where teams silently diverge. "Excludes partner-sourced deals. Excludes deals under $5K ACV. Excludes internal test accounts."

**Time logic.** How the period works. Trailing 12 months, calendar quarter, fiscal quarter, cohort-based, point-in-time snapshot. Include when it recalculates.

**Source system.** Where the underlying data lives. Name every system if the metric pulls from more than one.

**Source fields.** The exact field names, using API names rather than display labels. If a field is filtered, state the filter.

**Owner.** One named person. Not a team, not a role, a person. They approve any change to this definition.

**Last reviewed.** Date this definition was last confirmed accurate by the owner.

**Known conflicts.** If teams currently calculate this differently, document both versions here rather than erasing the disagreement. This field should be empty once the conflict is resolved, but while it exists, it is the most important field in the entry.

---

## Worked Example 1: Net Revenue Retention

### Net Revenue Retention (NRR)

| Field | Value |
|---|---|
| **Metric name** | Net Revenue Retention (NRR) |
| **Aliases** | Net dollar retention, NDR, net retention |
| **Definition** | The percentage of recurring revenue retained from existing customers over a period, including the effect of expansion and contraction. Measures whether the existing customer base grows or shrinks in revenue terms without counting new logos. |
| **Formula** | (Starting ARR - Contraction ARR - Churn ARR + Expansion ARR) / Starting ARR |
| **Numerator** | Starting ARR minus contraction minus churn plus expansion, all measured only on customers who were active at the start of the period |
| **Denominator** | Starting ARR from the same cohort of customers active at period start |
| **Inclusions** | All customers with at least one paid subscription active on the first day of the measurement period |
| **Exclusions** | Free tier accounts. Pilot accounts not yet converted to paid. One-time services and implementation revenue. Customers who signed during the measurement period (they belong in new ARR, not retention). |
| **Time logic** | Trailing 12 months, recalculated monthly on the first of the month |
| **Source system** | Salesforce (ARR fields, opportunity types) and Chargebee (churn and contraction events) |
| **Source fields** | `Account.ARR__c`, `Opportunity.Type` filtered to Renewal, Expansion, Contraction, `Account.Churn_Date__c`, `Account.Customer_Status__c` filtered to Active |
| **Owner** | VP RevOps |
| **Last reviewed** | 2026-07-15 |
| **Known conflicts** | None. Resolved 2026-04-02: finance previously included services revenue, now excluded by agreement. |

---

## Worked Example 2: Win Rate

### Win Rate

| Field | Value |
|---|---|
| **Metric name** | Win Rate |
| **Aliases** | Close rate, opportunity win rate |
| **Definition** | The percentage of qualified opportunities that result in closed-won revenue. Measures sales effectiveness on opportunities the team chose to pursue. |
| **Formula** | Closed-won opportunities / (Closed-won opportunities + Closed-lost opportunities) |
| **Numerator** | Count of opportunities with stage Closed Won and a close date inside the reporting period |
| **Denominator** | Count of all opportunities that reached a terminal stage (Closed Won or Closed Lost) with a close date inside the reporting period |
| **Inclusions** | Opportunities that reached Stage 2 (Qualified) or beyond. Both new business and expansion opportunities, reported separately and combined. |
| **Exclusions** | Opportunities disqualified before Stage 2. Duplicate opportunities. Opportunities marked Closed Lost with reason "Duplicate" or "Created in error." Renewal opportunities, which are measured in retention, not win rate. |
| **Time logic** | Calendar quarter, based on close date rather than create date. Reported alongside a trailing four-quarter trend. |
| **Source system** | Salesforce |
| **Source fields** | `Opportunity.StageName`, `Opportunity.CloseDate`, `Opportunity.Type`, `Opportunity.Loss_Reason__c`, `Opportunity.IsClosed` |
| **Owner** | Director of Sales Operations |
| **Last reviewed** | 2026-06-30 |
| **Known conflicts** | Marketing reports win rate from MQL forward, which produces a lower number than the sales definition above. Both are valid measurements of different things. Marketing's version is documented separately as "Lead-to-Close Rate" to prevent the two from being compared directly. |

---

## Changelog

Log every change to any definition. This is what prevents silent drift between what the glossary says and what reports actually calculate.

| Date | Metric | What changed | Why | Approved by |
|---|---|---|---|---|
| 2026-04-02 | NRR | Excluded services revenue from ARR calculation | Finance and RevOps aligned that services is non-recurring and should not sit in a recurring revenue metric | VP RevOps, CFO |
| 2026-05-20 | Win Rate | Added exclusion for renewal opportunities | Renewals were inflating win rate by roughly 6 points, since they close at a much higher rate than new business | Director of Sales Ops, CRO |

---

## When a Definition Changes

A changed definition is only half the work. Everything downstream that references it must be updated in the same cycle, or you have created a gap between what the glossary says and what your systems actually produce.

**Checklist for every definition change:**

- [ ] Owner has approved the change
- [ ] Changelog entry written
- [ ] Every dashboard using this metric updated
- [ ] Every report using this metric updated
- [ ] Every AI standing prompt referencing this metric updated (see `standing-prompt-examples.md`)
- [ ] Commission or comp calculations checked if this metric feeds them
- [ ] Historical comparisons flagged, since prior periods were calculated the old way
- [ ] Board or leadership notified if the change affects a reported number
- [ ] Causal model weights checked if this metric feeds a structural factor (see causal-model-setup/references/weighting-governance-log.md)

**On historical comparisons:** when a definition changes, prior periods calculated under the old definition are not directly comparable. Either restate history under the new definition or annotate the break in the series. Silently changing the definition and showing an unbroken trend line is how a board loses trust in a number. The same applies to the causal model. Structural weights derived from analysis under the old definition may not hold under the new one. Flag them for review at the next approval cycle rather than assuming they carry over.

---

## Common Mistakes

**Writing the definition without the fields.** "Churn rate is lost customers divided by total customers" is not a definition, it is a description. Without the field names and filters, AI will guess which records count as lost and which count as total.

**One entry for a metric that is actually two metrics.** If marketing and sales both track "pipeline" and mean different things, that is two metrics with two names, not one metric with a conflict. Split them and name them distinctly.

**Leaving exclusions blank.** An empty exclusions field usually means nobody checked, not that nothing is excluded. Every metric excludes something. Test accounts, at minimum.

**Assigning ownership to a team.** "Sales Ops owns this" means nobody owns it. One name.

**Letting last reviewed go stale.** A definition last reviewed 14 months ago is a definition nobody has verified against a business that has almost certainly changed.
