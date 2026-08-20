---
name: semantic-layer-setup
description: >
  Builds and maintains the metric definitions, field documentation, and
  governance structure that AI tools need to reason accurately about
  revenue data. Triggers on semantic layer, metric definitions, how do
  we calculate, data dictionary, field documentation, everyone defines
  it differently, the numbers dont match, which version of churn, define
  ARR, what fields feed this report, or any situation where AI is
  answering questions about revenue metrics without a documented
  definition underneath. BOUNDARY - For validating AI-generated causal
  answers that depend on this layer, see why-audit. For the causal model
  that reasons on top of it, see causal-model-setup.
---

# Semantic Layer Setup — The Definitions AI Needs Before It Can Reason About Your Revenue

You are helping someone build the foundational data layer that every AI tool in their revenue stack depends on — documented metric definitions, field-level mappings, standing system prompts, and named ownership. Without this, every AI answer about revenue is built on whatever the model assumes a metric means, which may not match how the business actually calculates it.

This is not a data warehouse project. It's a documentation and governance exercise that sits between your raw data and any AI tool that reasons about it.

---

## Why This Exists

When you ask AI "why is churn up this quarter," the first thing it needs to know is how you calculate churn. Not the textbook definition. Your definition. Which customers count. What time period. Whether voluntary and involuntary are separated or blended. Whether a downgrade counts as churn or contraction.

The definition lives in a finance team's spreadsheet formula, a Salesforce report filter, or a senior leader's memory. Different teams often calculate the same metric differently without realizing it.

AI doesn't resolve this ambiguity. It picks one version or infers its own. The output looks confident either way. The person reading it has no way to know which definition the AI used, or whether it matches the one the board expects to see.

A semantic layer fixes this by giving AI (and humans) one documented, authoritative source for what each metric means, how it's calculated, and which fields feed it.

---

## What a Semantic Layer Contains

### Component 1: Metric Glossary

A single document listing every revenue metric the organization tracks, with a precise, unambiguous definition for each.

**For each metric, document:**

| Field | What to capture |
|---|---|
| Metric name | The official name as used in board reporting and internal discussions |
| Aliases | Other names teams use for the same metric (e.g., "logo churn" vs. "customer churn" vs. "churn rate") — list all known variants so AI can map them to the canonical definition |
| Definition | Plain-language explanation of what this metric measures |
| Formula | Exact calculation, written out. Not "revenue minus costs" but the specific fields, filters, and time logic |
| Numerator | What's in the top of the fraction (if applicable), with field names |
| Denominator | What's in the bottom of the fraction (if applicable), with field names |
| Inclusions | What's explicitly counted — e.g., "includes all closed-won opportunities with a close date in the reporting period" |
| Exclusions | What's explicitly not counted — e.g., "excludes partner-sourced deals," "excludes deals under $5K ACV" |
| Time logic | How the time period works — trailing 12 months, calendar quarter, cohort-based, etc. |
| Source system | Where the underlying data lives (Salesforce, HubSpot, billing system, spreadsheet) |
| Source fields | The exact field names in the source system that feed this metric |
| Owner | One named person responsible for this definition. Not a team. |
| Last reviewed | Date this definition was last confirmed as accurate |

**Example entry:**

| Field | Value |
|---|---|
| Metric name | Net Revenue Retention (NRR) |
| Aliases | Net dollar retention, NDR |
| Definition | The percentage of recurring revenue retained from existing customers over a period, including expansion and contraction |
| Formula | (Starting ARR - Contraction ARR - Churn ARR + Expansion ARR) / Starting ARR |
| Numerator | Starting ARR minus contraction minus churn plus expansion, all from customers who were active at the start of the period |
| Denominator | Starting ARR from the same cohort |
| Inclusions | All customers active at the start of the measurement period with at least one paid subscription |
| Exclusions | Free tier accounts, pilot accounts not yet converted to paid, one-time services revenue |
| Time logic | Trailing 12 months, recalculated monthly |
| Source system | Salesforce (ARR fields) + billing system (churn/contraction events) |
| Source fields | `Account.ARR__c`, `Opportunity.Type` (filtered to Renewal, Expansion, Contraction), `Account.Churn_Date__c` |
| Owner | VP RevOps |
| Last reviewed | 2026-07-15 |

### Component 2: Field-Level Documentation

A separate document (or tab/section within the glossary) that maps every CRM and BI field the semantic layer depends on.

**For each field, document:**

| Field | What to capture |
|---|---|
| Field name | API name as it appears in the source system (e.g., `Opportunity.Stage__c`) |
| Display name | The label users see in the UI |
| System | Where this field lives (Salesforce, HubSpot, Snowflake, etc.) |
| Data type | Text, number, picklist, date, boolean, lookup, formula, etc. |
| What it stores | Plain-language description of what this field is supposed to contain |
| Who updates it | Is this field updated by a human (rep, manager, admin), an automation (workflow, flow, integration), or an AI tool? |
| Update frequency | Real-time, daily sync, manual, event-triggered |
| Validation rules | Any constraints on what can be entered (required, picklist-only, dependent on another field) |
| Known issues | Any known data quality problems — e.g., "40% of records have this blank," "legacy values from before 2024 use a different picklist" |
| AI access | Can AI tools read this field? Can they write to it? Should they be restricted from either? |

**The AI access column is critical.** Not every field should be visible to AI. PII, sensitive compensation data, contact data protected by privacy laws (GDPR), and certain custom fields should be explicitly excluded from AI read access. Document which fields are AI-readable and which are restricted, so the boundary is a policy decision rather than an accident of integration.

### Component 3: Standing System Prompts

If AI tools in your revenue stack use system prompts (instructions that shape how the model behaves before the user asks a question), those prompts should reference the semantic layer directly.

**What a standing prompt should include:**
- A reference to the metric glossary — "When answering questions about [metric], use the definition in [location]"
- Explicit instructions on which fields to use and which to ignore for each metric
- A fallback instruction — "If a metric is referenced that does not appear in the glossary, state that no documented definition exists rather than inferring one"
- A disclosure instruction — "If your answer depends on a metric calculation, state which definition you used"

**Example standing prompt fragment:**

> When calculating Net Revenue Retention (NRR), use the following definition:
>
> Formula: (Starting ARR - Contraction ARR - Churn ARR + Expansion ARR) / Starting ARR
> Time period: Trailing 12 months, recalculated monthly
> Include: All customers active at period start with at least one paid subscription
> Exclude: Free tier accounts, pilot accounts, one-time services revenue
> Source fields: Account.ARR__c, Opportunity.Type (Renewal, Expansion, Contraction), Account.Churn_Date__c
>
> If asked about a metric not defined in this glossary, respond: "This metric does not have a documented definition in the semantic layer. I can attempt an answer based on available data, but the calculation has not been validated. Do you want me to proceed with that caveat?"

### Component 4: Ownership and Versioning

A semantic layer without governance decays into the same problem it was built to solve — competing definitions with no authority.

**Rules:**
- Every metric has one named owner. That person approves any change to the definition. Changes without owner approval are reverted.
- Every change is versioned with a date, a description of what changed, and who approved it. A simple changelog at the bottom of the glossary doc is sufficient — this doesn't require a database.
- The full glossary is reviewed on a fixed cadence. Quarterly is the minimum for orgs actively changing their product, pricing, or reporting. Twice a year is acceptable for stable orgs.
- When a metric definition changes, every report, dashboard, and AI prompt that references it must be updated in the same cycle. A changed definition with unchanged downstream references is worse than no definition at all, because it creates a false sense of alignment.

---

## Building the Semantic Layer for the First Time

### Step 1: Inventory your metrics

List every metric that appears in any of the following places:
- Board decks or investor updates
- Executive dashboards
- CRM reports and dashboards
- Commission or compensation calculations
- Sales team KPIs or scorecards
- CS health scores or retention reports
- Marketing attribution reports

Don't define them yet. Just list them. The goal is a complete inventory before you start documenting definitions.

### Step 2: Identify conflicts

For each metric on the list, ask every team that uses it: "How do you calculate this?" Do this separately — don't put sales and finance in the same room for the first pass, because they'll negotiate toward a compromise before you've documented the actual disagreement.

Flag every metric where two or more teams gave different answers. These are your highest-priority definitions to resolve, because they're the ones most likely to produce conflicting AI outputs.

### Step 3: Define the canonical version

For each conflicting metric, get the relevant stakeholders in a room and agree on one definition. This is often harder than it sounds — "how do we calculate win rate" can be a 45-minute conversation when you discover that sales excludes partner deals and finance includes them.

Document the agreed definition using the metric glossary format above. The owner signs off.

### Step 4: Map the fields

For every metric you've defined, trace it back to the actual fields in the source system. Document each field using the field-level documentation format above. This is where you'll discover problems — a metric defined as "trailing 12 months" pulling from a field that only stores "current quarter," or a formula that references a field that's been deprecated.

### Step 5: Set AI access boundaries

Review every field in the documentation and explicitly decide: can AI read this? Can AI write to it? Fields containing PII (GDPR), compensation data, or sensitive internal notes should be restricted by policy, not left to whatever the integration defaults to. This column also serves as the permission baseline for agent monitoring. A field with no documented AI access decision cannot be monitored for unauthorized writes, because there is nothing to compare against.

### Step 6: Build standing prompts

For every AI tool in the revenue stack that answers questions about metrics (BI copilots, CRM assistants, forecast tools, reporting agents), create a standing system prompt that references the semantic layer. Use the format in Component 3.

### Step 7: Establish the review cadence

Set a recurring calendar event for the glossary review. Assign the overall semantic layer owner (usually the head of RevOps or the most senior analytics person). Document the cadence and the process for requesting a definition change between scheduled reviews.

---

## Common Failure Modes

**The glossary exists but nobody references it.** It was built once, shared in a Slack channel, and forgotten. AI tools were never configured to use it. Fix: the standing prompts in Component 3 are the mechanism that connects the glossary to actual AI behavior. Without them, the glossary is a document, not a layer.

**Definitions are documented but fields aren't mapped.** You know "churn rate = lost customers / total customers" but you haven't specified which CRM field counts as "lost" or what filters define "total." AI will guess. Fix: Step 4 (field mapping) is not optional, it's where the definition becomes actionable.

**The glossary is out of date.** A metric definition changed six months ago but the glossary still shows the old version. AI is using the old definition because that's what the standing prompt says. Fix: the versioning and review cadence in Component 4 exist specifically to prevent this. An unreviewed glossary is actively misleading.

**Two metrics use the same name but mean different things.** "Pipeline" means something different to marketing (all leads in progress) than to sales (qualified opportunities with a close date). Both are valid. Neither is wrong. But if AI doesn't know which one you're asking about, it picks one and doesn't tell you. Fix: the aliases field in the glossary forces you to surface these collisions and resolve them with distinct canonical names.

**AI writes to a field it shouldn't.** An enrichment tool or agent updates a field that feeds a metric calculation, changing the number without anyone changing the definition. Fix: the AI access column in Component 2 is the preventive measure. Restrict write access by policy before it becomes an incident.

---

## How This Connects to Other Skills

- **why-audit** Requirement 2 (Semantic Layer) is scored based on whether this layer exists. A "Present" score means the glossary, field mapping, standing prompts, and ownership/versioning are all in place and current. "Partial" means some exist but are incomplete, outdated, or not connected to AI tools. "Missing" means AI is inferring metric definitions on its own.
- **board-control-book** Step 1 says to start here if metric definitions don't exist. Every section of the control book depends on consistent metric definitions — the semantic layer is the foundation the entire reporting architecture is built on.
- **agentic-incident-playbook** lists this skill as a preventive measure against Type 1 (model drift) and Type 3 (cascading errors). Documented, versioned definitions make drift detectable and reduce the surface area for cascading data problems.
- **causal-model-setup** requires this layer as a prerequisite. A causal model reasons about why a metric moved, which requires knowing exactly what the metric is and which fields produce it.
- **agent-monitoring** requires this layer as a prerequisite. Drift detection compares outputs against documented definitions, and unauthorized action monitoring compares agent writes against the AI access column in the field documentation.

---

## Voice Rules

- Use the metric's canonical name from the glossary, not aliases, in all documentation and prompts
- When a metric has known conflicts between teams, state both versions before resolving — don't erase the disagreement, document it
- Field documentation should be precise enough that a new RevOps hire could reconstruct any metric from the glossary alone without asking someone

---

## Reference Files

| File | When to read | What's inside |
|---|---|---|
| `references/metric-glossary-template.md` | First-time setup | Blank glossary template with all fields, ready to fill |
| `references/field-documentation-template.md` | Step 4 (field mapping) | Blank field documentation template |
| `references/standing-prompt-examples.md` | Step 6 (building prompts) | Example standing prompts for common AI tools in the revenue stack |

## Related Skills

- **causal-model-setup** — required prerequisite relationship. Builds the weighted causal model that reasons on top of these definitions.
- **agent-monitoring** — required prerequisite relationship. Uses these definitions as the baseline for drift detection and the AI access column for permission monitoring.
- **why-audit** — depends on this layer for Requirement 2 (Semantic Layer) scoring
- **board-control-book** — depends on this layer for consistent metric definitions across all sections
- **agentic-incident-playbook** — references this layer as a preventive measure against model drift and cascading errors
