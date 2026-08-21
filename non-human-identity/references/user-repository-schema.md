# User Repository Schema

The record of every non-human user in the revenue stack. One row per user.

This is the source of truth for two things: who owns what access, and what an agent is permitted to do. The second is what `agent-monitoring` compares against when detecting out-of-scope actions, which means a blank permission field there is a monitoring gap, not just a documentation gap.

---

## How to Use This File

**What goes in it.** Every credential that authenticates to a revenue system and does not belong to a person. Service accounts, integration users, API clients, connected apps, OAuth grants held by third-party tools, bots, and AI agents.

**Where it lives.** A CMDB or your identity provider's service account management if you have one. A spreadsheet works below roughly 75 users. Past that, a spreadsheet becomes hard to keep accurate and the tooling is worth the cost.

**Who owns it.** One named person owns the repository. Each user in it has its own primary owner, who is accountable for that user specifically.

**Review cadence.** Weekly automated dormancy check. Quarterly full human review. Immediate update on any human offboarding or role change.

---

## Core Fields

Every user gets these, regardless of type.

| Field | Type | What to capture |
|---|---|---|
| `user_id` | Auto | Unique identifier for this record |
| `platform` | Picklist | Which system this user exists in |
| `username` | Text | The identifier as it appears in the platform |
| `user_type` | Picklist | Service account, integration user, API client, connected app, OAuth grant, bot, or AI agent |
| `primary_owner` | User | One named person accountable for this user |
| `escalation_owner` | User | Who gets contacted if the primary is unavailable, or if this user causes a problem. A different person from the primary. |
| `purpose` | Text | What it does, in a sentence someone outside the team would understand |
| `access_level` | Text | What permissions it holds, at the scope level |
| `write_access` | Boolean | Whether it can write anything at all |
| `integrations` | Multi-select | What it connects to |
| `triggers` | Text | What causes it to run |
| `expected_usage_frequency` | Picklist | Continuous, daily, weekly, monthly, quarterly, annual, or event-driven |
| `last_activity` | Date | Last observed authentication or action |
| `last_credential_rotation` | Date | When credentials were last changed |
| `credential_expiration` | Date | When they expire, if applicable |
| `created_date` | Date | |
| `created_via` | Picklist | Intake request, direct provisioning, or unknown (discovered) |
| `status` | Picklist | Active, Pending review, Queued for deactivation, Deactivated |
| `deactivated_date` | Date | Populated at deactivation |
| `deactivated_reason` | Text | Populated at deactivation |
| `last_reviewed` | Date | |

---

## Agent Fields

These apply only where `user_type` is AI agent. Leave blank for everything else.

They exist so `agent-monitoring` has a permission baseline to compare behavior against. Without them, monitoring can detect drift and coverage problems but cannot detect an agent acting outside its scope, which is the failure type with the shortest path to a customer.

| Field | Type | What to capture |
|---|---|---|
| `permitted_write_fields` | Multi-select | Every field this agent may write to, by API name. Empty means read-only. |
| `permitted_read_fields` | Multi-select | Fields this agent may read, or a reference to the view layer it reads through |
| `restricted_fields` | Multi-select | Fields explicitly excluded from this agent's access, with the reason in `restriction_reason` |
| `restriction_reason` | Text | Why the restricted fields are restricted. GDPR, compensation data, contractual confidentiality, or a business reason. |
| `scoped_population` | Text | Which records this agent may act on. "Leads with lifecycle stage New or Marketing Qualified, created within 90 days." |
| `external_actions` | Multi-select | Actions visible outside the company. Email send, sequence enrollment, meeting booking, form submission. Empty means none. |
| `approval_gate` | Picklist | None, All writes, External actions only, or Specific fields. Which actions require human approval before executing. |
| `likely_failure_types` | Multi-select | Which of the five failure types this agent realistically risks |
| `covering_checks` | Multi-select | Check IDs from the detection register that monitor this agent |
| `model_or_tool` | Text | Which model or product this agent runs on |
| `last_calibrated` | Date | For scoring or classification agents, when the model was last calibrated |

---

## Field Notes

**`escalation_owner` must differ from `primary_owner`.** A single point of contact means a problem waits on one person's availability. Enforce this as a validation rule if the system allows it.

**`expected_usage_frequency` prevents the most common false positive.** A user running a quarterly reconciliation is dormant eleven weeks out of thirteen. Without a documented expectation, the weekly dormancy check flags it every cycle and people stop reading the flags.

**`created_via` marks discovery gaps.** A user marked unknown was found through discovery rather than created through intake. That is not a problem in itself, but a rising count of unknowns means the intake is being bypassed.

**`permitted_write_fields` empty is meaningful.** Empty means read-only, which is the default and the desired state for most agents. It is different from the field never having been filled in, which is a documentation gap. Use a validation rule requiring an explicit entry rather than allowing null.

**`scoped_population` is prose, not a picklist.** It describes a filter in terms a person can read and an engineer can translate into a query. Vague entries produce checks that cannot be built.

**`covering_checks` is the coverage link.** An agent with an empty value and a populated `likely_failure_types` is an agent nobody is watching. The quarterly review surfaces these.

**`last_calibrated` only applies to some agents.** A scoring or forecasting model has a calibration date. A summarization agent does not. Leave blank rather than inventing one.

---

## Worked Example: An AI Agent

| Field | Value |
|---|---|
| `platform` | HubSpot |
| `username` | `svc_sequencing_agent` |
| `user_type` | AI agent |
| `primary_owner` | Priya Nair, Marketing Ops Manager |
| `escalation_owner` | Dan Whitfield, Director of RevOps |
| `purpose` | Enrolls new inbound leads into the appropriate nurture sequence based on lead score, activity data, and intent |
| `access_level` | Marketing user, restricted profile |
| `write_access` | Yes |
| `integrations` | HubSpot, Salesforce (read only), inbound form handler |
| `triggers` | New lead created from an inbound form |
| `expected_usage_frequency` | Continuous |
| `last_activity` | 2026-08-19 |
| `last_credential_rotation` | 2026-06-01 |
| `credential_expiration` | 2026-12-01 |
| `created_date` | 2026-02-14 |
| `created_via` | Intake request |
| `status` | Active |
| `permitted_write_fields` | `Contact.Sequence_Enrolled__c`, `Contact.Sequence_Status__c` |
| `permitted_read_fields` | Reads through `vw_lead_routing`, which excludes personal contact data |
| `restricted_fields` | `Contact.Personal_Email__c`, `Contact.Phone`, all compensation fields |
| `restriction_reason` | Personal contact data under GDPR, consent does not cover AI processing. Compensation fields are out of scope for any marketing user. |
| `scoped_population` | Contacts with lifecycle stage New or Marketing Qualified, created within the last 90 days |
| `external_actions` | Sequence enrollment, which triggers email sends |
| `approval_gate` | External actions only |
| `likely_failure_types` | Type 2 unauthorized action, Type 3 cascading error |
| `covering_checks` | PERM-01, PERM-02, VOL-01 |
| `model_or_tool` | HubSpot Breeze, sequence routing workflow |
| `last_calibrated` | N/A |

**What this record makes possible.** Monitoring can now query field history for any write outside those two fields, and any write to a record outside that population, because both are documented. Without `permitted_write_fields` and `scoped_population`, neither check can be built.

---

## Worked Example: A Service Account

| Field | Value |
|---|---|
| `platform` | Salesforce |
| `username` | `svc_billing_sync` |
| `user_type` | Integration user |
| `primary_owner` | Marcus Reid, Revenue Accounting |
| `escalation_owner` | Sam Ortiz, Salesforce Administrator |
| `purpose` | Nightly sync of invoice and payment status from the billing platform into Salesforce account records |
| `access_level` | API-only integration profile, write access to three account fields |
| `write_access` | Yes |
| `integrations` | Salesforce, Chargebee |
| `triggers` | Scheduled, nightly at 02:00 UTC |
| `expected_usage_frequency` | Daily |
| `last_activity` | 2026-08-20 |
| `last_credential_rotation` | 2026-01-15 |
| `credential_expiration` | None |
| `created_date` | 2024-09-03 |
| `created_via` | Unknown (discovered) |
| `status` | Active |
| `last_reviewed` | 2026-07-15 |

Agent fields left blank throughout. This user is not an agent, and populating those columns for it would add noise without adding information.

**Two things this record surfaces.** Credentials last rotated seven months ago with no expiration date set, which the quarterly review should flag against policy. And `created_via` marked unknown, meaning this predates the intake process and nobody has confirmed the original approval.

---

## Validation Rules

Enforce where the system supports it, check manually where it does not.

- `primary_owner` and `escalation_owner` are both required and must differ
- `purpose` is required and cannot be blank
- `expected_usage_frequency` is required, because the dormancy check depends on it
- Where `user_type` is AI agent, `permitted_write_fields`, `scoped_population`, and `approval_gate` are required
- Where `write_access` is true, `permitted_write_fields` cannot be empty
- Where `restricted_fields` is populated, `restriction_reason` is required
- `deactivated_date` and `deactivated_reason` are required when `status` is Deactivated

---

## Maintenance

**Resolve, do not delete.** When a user is decommissioned, set status to Deactivated and populate the date and reason. Keep the row. Historical records matter for reconstructing what had access when.

**Update on change, not only on review.** A permission change, an ownership change, or a credential rotation updates the record when it happens. The quarterly review verifies the record, it is not where the record gets written.

**Compare against system state, not just against itself.** The quarterly review pulls current users from each platform and compares to the repository. A repository that only reviews its own contents cannot find what was created outside it.
