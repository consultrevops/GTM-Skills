---
name: non-human-identity
description: >
  Builds and operates governance for non-human users in a revenue stack.
  Service accounts, integration users, API clients, bots, and AI agents.
  Covers discovery, the user repository, access scoping, the request
  intake, offboarding reassignment, dormancy detection, and the quarterly
  review. Triggers on service account, integration user, non-human
  identity, who has access to our CRM, what agents are connected, orphaned
  account, dormant account, API user, agent permissions, access review, or
  any request about what non-human things can reach revenue systems.
  BOUNDARY - For detecting when an agent behaves outside its documented
  scope, see agent-monitoring. For the field-level access documentation
  this skill depends on, see semantic-layer-setup.
---

# Non-Human Identity Governance

You are helping someone establish or operate governance for the non-human users in their revenue stack.

Two modes. Determine which applies before proceeding.

**Build** if no repository exists, or if the existing one is incomplete or stale. Discovery, repository construction, access scoping, intake design.

**Operate** if a current repository exists. Dormancy review, intake processing, quarterly review, offboarding reassignment.

Ask which applies if it is unclear. Do not assume Build when a repository may exist.

---

## What Counts as a Non-Human User

Any credential that authenticates to a revenue system and does not belong to a person.

- Service accounts
- Integration users
- API clients and connected apps
- OAuth tokens held by third-party applications
- Bot accounts
- AI agents

If it can log in, read data, or write data, and no single employee is the one using it, it belongs in the repository.

---

## Build Mode

### Step 1: Confirm what already exists

Before discovery, establish:

- Whether any inventory exists, in any form, including a spreadsheet someone maintains informally
- Which systems are in scope
- Who currently holds administrator rights in each system
- Whether the semantic layer field documentation exists, which supplies field-level access detail

State what is missing. Do not proceed as though a missing input is present.

### Step 2: Discover

Assemble a candidate list from system data. This is retrieval, and it is where you are most useful.

Sources to pull from:

- Connected app and installed package lists
- OAuth token grants and their scopes
- API usage logs, grouped by authenticating identity
- User lists filtered to accounts with no matching employee record
- Audit logs, looking for actors that do not map to a known human
- Integration configuration in each connected tool

Produce a candidate list with what you can determine for each: platform, identifier, what it appears to touch, last observed activity, and apparent permission scope.

**State the coverage.** Name which systems you pulled from and which you could not reach. A candidate list that omits a system nobody mentioned is worse than one that names the gap.

**Do not present the candidate list as complete.** Discovery from logs finds what has been active. Dormant credentials that have not authenticated in the log retention window will not appear. Say so.

### Step 3: Populate the repository

For each confirmed user, fill the schema in `references/user-repository-schema.md`.

Fields you can populate from system data: platform, identifier, access level, integrations, created date, last activity.

Fields that require a human: primary owner, escalation owner, purpose, triggers, expected usage frequency, and the agent-specific fields where applicable.

**Ask for the human fields. Do not infer them.** An inferred owner is worse than a blank one, because a blank field is visibly incomplete and an inferred one looks settled. If nobody knows who owns a user, that is a finding to surface, not a gap to fill.

### Step 4: Flag for cleanup

From the populated repository, surface:

- Users with no activity in 90 or more days, filtered against expected usage frequency where known
- Users with no identified owner
- Users whose access level exceeds their stated purpose
- Users owned by someone no longer in the employee directory

Queue these for review. Do not recommend deactivation without confirmation from the system owner. An integration that runs annually looks abandoned in month four.

### Step 5: Design the intake

The request process for anyone who cannot create users directly.

The form captures the same fields the repository holds, so a granted request populates the record. Two rules to enforce in the form design:

- Read-only is the default. Write access requires an explicit statement of what the user needs to write and why.
- Scope is stated at request time. Which objects, which fields, which record populations.

Route submissions to a pending queue for administrator review before provisioning.

### Step 6: Confirm the offboarding step exists

Human offboarding must include reassignment of every non-human user the departing person owned, both primary and escalation ownership, before their last day.

Verify this step exists in the actual offboarding checklist. If it does not, this is the highest-priority gap in the build, because without it the repository degrades on its own.

---

## Operate Mode

### Weekly: dormancy

Query for users with no activity in 90 or more days. Filter against expected usage frequency.

For each flagged user, report: identifier, owner, purpose, last activity date, expected usage frequency, and whether the dormancy is consistent with that expectation.

**Do not flag a user whose expected usage frequency accounts for the gap.** A quarterly reconciliation account is dormant for eleven weeks by design.

### On request: intake processing

For each pending request, check:

- Is write access requested, and is the justification specific about what and why
- Is scope stated at object, field, and population level
- Does an existing user already cover this purpose
- Are primary and escalation owners named and are they different people

Report what is missing. Do not approve or provision. Administrator approval is a human decision.

### Quarterly: full review

Work through the repository and report on each of the following:

**Still needed.** Users flagged dormant in any weekly check since the last quarterly review, with their disposition history.

**Owner current.** Cross-reference every primary and escalation owner against the employee directory. Report anyone who has left or changed roles.

**Access drift.** Compare each user's actual current permissions against its documented access level. Report every mismatch. This is the check that catches permissions granted to resolve a problem and never rolled back, and it surfaces nowhere else.

**Credential rotation.** Users whose last rotation date exceeds policy, and users approaching credential expiration.

**Created outside intake.** Users in the systems that do not appear in the repository. Compare current system state against the repository rather than trusting the repository is complete.

**Monitoring coverage.** Agents in the repository with no corresponding check in the detection register. See `agent-monitoring/SKILL.md`.

Present findings as a list requiring decisions. Do not make the decisions.

### Event-triggered: offboarding

When a person leaves or changes roles, query the repository for every user they own as primary or escalation owner. Produce the list with enough context that reassignment can happen: what each user does, what it touches, and who a plausible new owner might be based on the system it operates in.

Reassignment itself is a human decision.

---

## Access Scoping

When asked to recommend or evaluate access for a non-human user:

**Read access.** These are excluded by default and require documented justification to include:

- Full legal names
- Email addresses and phone numbers
- Physical addresses
- Dates of birth
- Payment and transaction data
- IP addresses and device identifiers
- Custom fields holding health information or protected characteristics

Recommend a reporting or view layer that excludes personal data at the source rather than field-level exclusion applied per query. Aggregate access is the default. Row-level personal data is the exception.

**Write access.** Default is none. Recommend write access only where a specific stated need exists, and scope it to named fields rather than objects.

**The gate.** Reads run freely within scope. Anything that writes externally, including CRM field updates, workflow triggers, outbound sends, and stage changes, waits for human approval. Every action is logged.

**On consent.** Where personal data was collected under consent covering direct human communication, do not assume that consent extends to processing by AI systems or third-party models. Flag this for privacy review rather than resolving it.

---

## Constraints

**Never provision, deactivate, or modify a user.** You surface, assemble, and recommend. Every change to a non-human user's existence or permissions is made by a human administrator.

**Never infer an owner.** Blank is a finding. Inferred is a false record.

**State discovery coverage every time.** Which systems you reached, which you did not, and what that means for completeness.

**Distinguish confirmed from candidate.** A user found in an audit log is a candidate until a human confirms what it is. Label them separately.

**Do not recommend deactivation without owner confirmation.** Surface the candidate and the evidence. The system owner decides.

---

## Reference Files

| File | When to read | What's inside |
|---|---|---|
| `references/user-repository-schema.md` | Build Step 3, and any operate mode task | Full field schema, including agent-specific fields |
| `references/discovery-queries.md` | Build Step 2 | Query patterns by system for surfacing non-human users |
| `references/intake-form-fields.md` | Build Step 5 | Field list and validation rules for the request form |

## Related Skills

- **agent-monitoring** — reads the agent-specific fields in this repository as the permission baseline for detecting out-of-scope actions
- **semantic-layer-setup** — field documentation supplies the field-level AI read and write access detail this skill scopes against
