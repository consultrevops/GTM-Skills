# Field Documentation Template

This file maps every CRM and BI field that the metric glossary depends on. The glossary says how a metric is calculated. This file says what the underlying fields actually contain, who writes to them, and whether AI is allowed to read or write them.

One entry per field. Copy the blank template below. Three worked examples follow.

---

## How to Use This File

**What to document:** Every field referenced in `metric-glossary-template.md`, plus every field an AI agent can read or write, plus every field that feeds a report anyone outside your team looks at.

**What not to document:** Fields nobody uses and no metric depends on. This file is a working map, not a full schema dump. A complete export of every field in your CRM is a different artifact and nobody reads it.

**Where it lives:** Alongside the metric glossary. Same file, separate tab, or a linked document. They are read together.

**Who owns it:** The system administrator for each source system owns the entries for that system. The semantic layer owner owns the file overall.

**Review cadence:** Quarterly, alongside the metric glossary review. Also review any time a new AI tool gains access to a system.

---

## Blank Template

### [Field API Name]

| Attribute | Value |
|---|---|
| **Field name (API)** | |
| **Display name** | |
| **System** | |
| **Object** | |
| **Data type** | |
| **What it stores** | |
| **Who updates it** | |
| **Update frequency** | |
| **Validation rules** | |
| **Picklist values** | |
| **Feeds which metrics** | |
| **Known issues** | |
| **AI read access** | |
| **AI write access** | |
| **Restriction reason** | |
| **Owner** | |
| **Last reviewed** | |

---

## Attribute Definitions

**Field name (API).** The API name as it appears in the source system, not the label users see. `Opportunity.Loss_Reason__c`, not "Loss Reason." AI tools and integrations reference the API name.

**Display name.** The label users see in the interface. Worth capturing because people describe fields by their label, and the two often diverge over time.

**System.** Salesforce, HubSpot, Snowflake, the billing platform, wherever this field lives.

**Object.** Which object or table the field sits on. Opportunity, Account, Contact, Lead, or a custom object.

**Data type.** Text, long text, number, currency, percent, date, datetime, checkbox, picklist, multi-select picklist, lookup, formula, roll-up summary.

**What it stores.** Plain language. What is this field for, and what should a correct value look like. Include an example value.

**Who updates it.** One of four: a human (specify which role), an automation (name the workflow, flow, or process), an integration (name the tool), or an AI agent (name the agent). If more than one thing writes to this field, list all of them. Multiple writers is a common source of data quality problems and worth flagging.

**Update frequency.** Real-time, on record save, daily sync, weekly batch, manual, event-triggered. If it is a sync, note the direction and what wins on conflict.

**Validation rules.** Anything constraining what can be entered. Required at a stage, picklist-only, dependent on another field, format enforcement, uniqueness. If there are no validation rules, say so explicitly rather than leaving it blank, because "no validation" is meaningful information.

**Picklist values.** For picklist fields, list every active value. Note any deprecated values still present in historical records, since AI reasoning over history will encounter them.

**Feeds which metrics.** Which entries in the metric glossary depend on this field. This is the link between the two files, and it is what tells you what breaks when a field changes.

**Known issues.** Data quality problems someone should know about before trusting this field. Fill rate, legacy value formats, periods where the field was used differently, known duplicates. Be blunt here. "Blank on 40% of records created before Q2 2025" is more useful than silence.

**AI read access.** Yes, No, or Conditional. If conditional, state the condition.

**AI write access.** Yes, No, or With approval. Default to No and grant deliberately.

**Restriction reason.** If read or write access is restricted, why. PII, GDPR, compensation data, contractual confidentiality, or a business reason. Documenting the reason prevents someone quietly reversing the restriction later because nobody remembered why it was there.

**Owner.** The person responsible for this field's configuration and data quality.

**Last reviewed.** Date last confirmed accurate.

---

## Worked Example 1: A Standard Metric Field

### Opportunity.ARR__c

| Attribute | Value |
|---|---|
| **Field name (API)** | `Opportunity.ARR__c` |
| **Display name** | Annual Recurring Revenue |
| **System** | Salesforce |
| **Object** | Opportunity |
| **Data type** | Currency |
| **What it stores** | The annualized recurring contract value for this opportunity, excluding one-time fees and services. Example value: 48000 |
| **Who updates it** | Human (Account Executive) at Stage 3, then locked by validation rule at Closed Won |
| **Update frequency** | On record save, editable until close |
| **Validation rules** | Required to advance past Stage 3. Cannot be edited after Closed Won without Sales Ops approval. Must be greater than zero. |
| **Picklist values** | N/A |
| **Feeds which metrics** | ARR movement, NRR, GRR, average deal size, CLTV, pipeline coverage |
| **Known issues** | Opportunities created before March 2025 sometimes include services revenue in this field. Records prior to that date should be treated as unreliable for ARR reporting. |
| **AI read access** | Yes |
| **AI write access** | No |
| **Restriction reason** | Write restricted because this field feeds six metrics and is locked at close for audit purposes. AI-suggested values go to a staging field for human review. |
| **Owner** | Director of Sales Operations |
| **Last reviewed** | 2026-07-15 |

---

## Worked Example 2: A Field with Real Data Quality Problems

### Opportunity.Loss_Reason__c

| Attribute | Value |
|---|---|
| **Field name (API)** | `Opportunity.Loss_Reason__c` |
| **Display name** | Loss Reason |
| **System** | Salesforce |
| **Object** | Opportunity |
| **Data type** | Picklist |
| **What it stores** | The primary reason a deal was lost, selected by the rep at close. Paired with `Loss_Reason_Detail__c` for free text. Example value: Lost to Competitor |
| **Who updates it** | Human (Account Executive) at Closed Lost |
| **Update frequency** | On record save at close |
| **Validation rules** | Required when stage is set to Closed Lost. Free-text detail field required when value is Other. |
| **Picklist values** | Price, Lost to Competitor, No Budget, No Decision, Timing, Missing Feature, Champion Left, Duplicate, Created in Error, Other. Deprecated values still present in historical records: Not Interested (retired Q1 2025), Bad Fit (retired Q1 2025). |
| **Feeds which metrics** | Win rate, lost rates by stage, competitive loss rate, churn taxonomy reporting |
| **Known issues** | "Other" is selected on roughly 30% of closed-lost records, which limits how much causal analysis this field can support. Free-text detail is often a single word. Reps close deals in bulk at quarter end, which produces clustered timestamps and lower-quality selections. |
| **AI read access** | Yes |
| **AI write access** | No |
| **Restriction reason** | Write restricted because loss reason is a human judgment about why a deal ended. An AI-inferred loss reason would create the appearance of rep-reported data that is actually model output. |
| **Owner** | Director of Sales Operations |
| **Last reviewed** | 2026-07-15 |

---

## Worked Example 3: A Restricted Field

### Contact.Personal_Email__c

| Attribute | Value |
|---|---|
| **Field name (API)** | `Contact.Personal_Email__c` |
| **Display name** | Personal Email |
| **System** | Salesforce |
| **Object** | Contact |
| **Data type** | Email |
| **What it stores** | A contact's personal email address, captured only where the contact provided it directly and consented to its use |
| **Who updates it** | Human (Account Executive or CSM) |
| **Update frequency** | Manual, infrequent |
| **Validation rules** | Email format enforced. Consent checkbox `Personal_Email_Consent__c` required before the field can be populated. |
| **Picklist values** | N/A |
| **Feeds which metrics** | None |
| **Known issues** | Populated on fewer than 5% of contacts. Some legacy records predate the consent requirement and should be treated as unconsented. |
| **AI read access** | No |
| **AI write access** | No |
| **Restriction reason** | Personal data under GDPR. Consent covers direct human communication only, not processing by AI systems or third-party models. Excluded from every AI integration at the field-permission level, not just by prompt instruction. |
| **Owner** | RevOps, with legal review |
| **Last reviewed** | 2026-07-15 |

---

## Setting AI Access Boundaries

Access is a policy decision. Left undecided, it defaults to whatever the integration was configured to see, which is usually everything.

**Default to No on write access.** An agent that can write to a field that feeds a metric can change a reported number without anyone changing the definition. Grant write access deliberately, field by field, with a documented reason.

**Restrict read access on these categories at minimum:**

- Personal data covered by GDPR, CCPA, or similar regulation, especially where consent does not cover AI processing
- Compensation, quota, and commission fields
- Fields containing confidential contract terms
- Internal notes fields where people write candidly about accounts or colleagues
- Any field a customer would be surprised to learn was being processed by a model

**Restriction happens at the permission level, not in the prompt.** Telling an AI tool not to look at a field is a request. Removing the field from the integration's permission set is a control. Use the control.

**Fields with multiple writers deserve extra scrutiny.** If a human, an automation, and an agent all write to the same field, you have a conflict-resolution problem waiting to happen. Document who wins and under what conditions, or reduce the number of writers.

---

## Common Mistakes

**Documenting the label instead of the API name.** AI tools and integrations reference API names. A file full of display names is a file AI cannot use.

**Leaving known issues blank.** Every field of any age has issues. A blank known-issues field usually means nobody looked, and it is the field that most protects someone from trusting bad data.

**Treating AI access as a technical detail.** It is a policy decision with legal exposure attached. It belongs in this file with a documented reason and a named owner.

**Skipping the "feeds which metrics" link.** Without it, nobody knows what breaks when a field changes. That link is what turns two separate documents into an actual semantic layer.

**Documenting fields nobody uses.** A file with 400 entries is a file nobody maintains. Document what the glossary depends on and what AI can touch. Skip the rest.
