# Intake Form Fields

The request form for provisioning a non-human user. Every field maps to the repository schema, so a granted request populates a record without anyone retyping it.

Two things this form exists to enforce:

**Read-only is the default.** Write access requires the requester to state what needs writing and why.

**Scope is stated at request time.** Which objects, which fields, which records. A request for "Salesforce access" gets provisioned broadly, because nobody specified otherwise.

---

## How to Use This File

**Where the form lives.** Anywhere the request reaches an administrator and produces a record. A form tool, a ticketing queue, or a request template in your existing service desk. The mechanism matters less than the routing.

**Where submissions go.** A pending queue an administrator reviews. Nothing provisions on submission.

**Who reviews.** A named administrator for the platform being requested. Approval is a human decision.

**What happens on approval.** The record is created in the repository with `created_via` set to Intake request, and the administrator fills in the fields only they can know, like the actual username assigned.

---

## Section 1: Requester and Purpose

| Field | Type | Required | Maps to | Notes |
|---|---|---|---|---|
| Requester | Auto | Yes | — | Captured automatically from the submitter |
| Request date | Auto | Yes | — | |
| Platform | Picklist | Yes | `platform` | Which system. Populate from your actual system list rather than free text. |
| User type | Picklist | Yes | `user_type` | Service account, integration user, API client, connected app, bot, or AI agent |
| Purpose | Long text | Yes | `purpose` | What this user will do, in a sentence someone outside the requesting team would understand |
| Business justification | Long text | Yes | — | Why this needs its own credential rather than running under an existing one. Reviewed, not stored. |
| Requested live date | Date | Yes | — | |

**On business justification.** This field is not stored in the repository, but it is the field that prevents the most sprawl. A request that cannot explain why an existing user will not work is often a request for a duplicate.

---

## Section 2: Ownership

| Field | Type | Required | Maps to | Notes |
|---|---|---|---|---|
| Primary owner | User | Yes | `primary_owner` | The person accountable for this user. May be the requester or someone else. |
| Escalation owner | User | Yes | `escalation_owner` | Contacted if the primary is unavailable or this user causes a problem |

**Validation:** primary and escalation owner must be different people. Enforce in the form rather than catching it at review.

**On naming an owner at request time.** Requiring an owner before provisioning is what prevents the ownerless accounts discovery keeps finding. If the requester cannot name someone willing to own it, that is a signal worth surfacing rather than a field to work around.

---

## Section 3: Access Scope

| Field | Type | Required | Maps to | Notes |
|---|---|---|---|---|
| Objects needed | Multi-select | Yes | `access_level` | Which objects or tables this user needs to reach |
| Read fields | Multi-select or long text | Yes | `permitted_read_fields` | Which fields, or the view layer it will read through |
| Write access needed | Boolean | Yes | `write_access` | Defaults to No |
| Write fields | Multi-select | Conditional | `permitted_write_fields` | Required when write access is Yes. Named fields, not objects. |
| Write justification | Long text | Conditional | — | Required when write access is Yes. What it writes and why a human cannot. |
| Record population | Long text | Yes | `scoped_population` | Which records this user may act on. "Leads with lifecycle stage New, created within 90 days." |
| Sensitive data needed | Boolean | Yes | — | Whether this user needs access to personal data, payment data, or compensation data |
| Sensitive data justification | Long text | Conditional | `restriction_reason` | Required when sensitive data is Yes. Routes to privacy review. |

**Validation:**

- Write fields cannot be empty when write access is Yes
- Write justification cannot be empty when write access is Yes
- Sensitive data justification cannot be empty when sensitive data is Yes

**On write fields being named individually.** Requesting write access to an object provisions write access to every field on it. Requesting two named fields provisions two fields. The form should make the narrower request the easier one to complete.

**On record population as free text.** This describes a filter in terms a person can read and an administrator can translate into a permission set or sharing rule. A vague entry produces a broad grant, so review it as carefully as the field list.

**On the sensitive data flag.** This routes the request to privacy review in addition to administrator review. Where personal data was collected under consent covering human communication, that consent generally does not extend to AI processing. That is a legal determination, not an administrator's call.

---

## Section 4: Operation

| Field | Type | Required | Maps to | Notes |
|---|---|---|---|---|
| Integrations | Multi-select | Yes | `integrations` | What this connects to on either side |
| Triggers | Long text | Yes | `triggers` | What causes it to run. A schedule, an event, a threshold, a human request. |
| Expected usage frequency | Picklist | Yes | `expected_usage_frequency` | Continuous, daily, weekly, monthly, quarterly, annual, event-driven |
| Expected duration | Picklist | Yes | — | Ongoing, or a fixed end date |
| End date | Date | Conditional | — | Required when duration is fixed. Populates a scheduled deactivation review. |

**On expected usage frequency.** This drives the weekly dormancy check. A quarterly reconciliation account flagged as dormant eleven weeks out of thirteen trains people to ignore the flags. Capturing the expectation at request time is the only reliable moment to get it.

**On expected duration.** Most requests will say ongoing, and that is usually honest. But a credential provisioned for a migration, a pilot, or a one-time backfill has a natural end, and capturing it at request time is far easier than discovering it two years later during a review.

---

## Section 5: Agent Fields

Shown only when user type is AI agent.

| Field | Type | Required | Maps to | Notes |
|---|---|---|---|---|
| Model or tool | Text | Yes | `model_or_tool` | Which model or product this agent runs on |
| External actions | Multi-select | Yes | `external_actions` | Actions visible outside the company. Email send, sequence enrollment, meeting booking, form submission. Select None if none. |
| Approval gate requested | Picklist | Yes | `approval_gate` | None, All writes, External actions only, Specific fields |
| Failure types this agent risks | Multi-select | No | `likely_failure_types` | Requester's assessment. Confirmed at review. |

**On external actions.** Any selection other than None should default the approval gate to at least External actions only. An agent that can send email without a gate is the configuration with the shortest path to customer impact, and the form should make that a deliberate choice rather than an oversight.

**On failure types.** Optional for the requester, since most will not know the framework. The reviewer confirms or corrects it, and it feeds the monitoring coverage check in the quarterly review.

---

## Reviewer Checklist

Not part of the form. What the administrator confirms before provisioning.

- [ ] An existing user does not already cover this purpose
- [ ] Business justification explains why a dedicated credential is needed
- [ ] Primary and escalation owners are named and are different people
- [ ] Both owners have confirmed they accept ownership
- [ ] Write fields are named individually, not requested at object level
- [ ] Write justification explains why a human cannot perform the write
- [ ] Record population is specific enough to translate into a permission set
- [ ] Sensitive data request has been through privacy review, where applicable
- [ ] Expected usage frequency is set, so dormancy detection works
- [ ] Where user type is AI agent, external actions and approval gate are both set
- [ ] Where the agent takes external actions, an approval gate is in place
- [ ] Credential rotation schedule is set at provisioning
- [ ] Repository record created with `created_via` set to Intake request
- [ ] Where the user is an agent, monitoring checks are queued in the detection register

**The last item is the one most often missed.** A new agent provisioned without corresponding checks is an agent nobody is watching, and it will not surface until the quarterly coverage review.

---

## Denial Reasons

Track why requests are denied. The pattern is more useful than any individual decision.

| Reason | What it usually indicates |
|---|---|
| Duplicate of existing user | The repository is not visible enough to requesters |
| Scope too broad | The form is not making the narrow request easy enough |
| No owner willing to be named | Worth investigating before provisioning anything |
| Write access without justification | Requester defaulted to write out of habit |
| Human alternative exists | The task did not need a credential |
| Pending privacy review | Not a denial, a hold |

A rising count in any single category points at a fixable problem in the form or the process rather than at the requesters.
