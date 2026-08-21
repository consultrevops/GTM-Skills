# Discovery Queries

Patterns for finding non-human users that were never documented.

These describe what to look for and where. They are not runnable as written. Object names, field names, API endpoints, permission models, and log retention windows differ by platform, edition, and version, and they change. Translate each pattern against current documentation for the specific instance you are working in.

---

## How Discovery Works

Non-human users are found four ways, in rough order of reliability.

**Direct enumeration.** The platform has a list of connected apps, integration users, or API clients. Most complete, when it exists.

**Employee directory subtraction.** Pull all users, remove everyone who matches an employee record. What remains is either non-human or a stale human account, and both are worth knowing about.

**Log-based inference.** Group authentication or API activity by actor and look for actors that do not map to a person. Finds active users the other methods missed.

**Configuration inspection.** Read the connection settings inside each connected tool to find what credential it authenticates with. Catches users that exist in a system you did not think to search.

No single method is complete. Run all four where possible and reconcile.

---

## Pattern 1: Connected Apps and Integrations

**What you are looking for:** every third-party application with an active connection.

**Where it lives:** most platforms expose this in a setup or admin area as connected apps, installed packages, integrations, or authorized applications.

**What to capture for each:**

- Application name and publisher
- The credential or user it authenticates as
- Granted scopes, at the finest granularity available
- Date authorized and who authorized it
- Last used, where the platform records it
- Whether the scopes include write

**Why this comes first.** Both the Salesloft Drift and Gainsight compromises ran through connected applications holding legitimate OAuth grants. A connected app with broad scopes is the highest-risk category in the inventory, and this is the only place it appears.

**Common gap.** Applications authorized by someone who has since left. The authorization survives the person. Cross-reference the authorizing user against the employee directory.

---

## Pattern 2: OAuth and Refresh Token Grants

**What you are looking for:** active tokens, their scopes, and their age.

**Where it lives:** usually adjacent to connected apps, sometimes in a separate token or session administration area. Some platforms expose refresh tokens separately from access tokens.

**What to capture:**

- Which application or user the token was issued to
- Scopes granted
- Issue date
- Expiration, or a note that it does not expire
- Last used

**What matters most here:** refresh tokens with no expiration. An access token expiring in an hour is bounded. A refresh token that lives indefinitely and can mint new access tokens is functionally a permanent credential, and it is the specific mechanism both 2025 breaches exploited.

**Flag for review:** any refresh token older than your credential rotation policy, and any token whose scopes exceed what the application's documented purpose requires.

---

## Pattern 3: User List Minus Employee Directory

**What you are looking for:** accounts in the system with no matching person.

**How it works:** pull all active users from the platform. Match against your HRIS or directory on email, employee ID, or name. Everything unmatched goes on the candidate list.

**What to capture:**

- Username and identifier
- Profile, role, or permission set
- Created date and creating user
- Last login
- Whether the account has API-only access or full interactive access

**Interpreting the result.** Unmatched accounts fall into three groups. Genuine non-human users, which belong in the repository. Stale human accounts belonging to people who left, which belong in an offboarding conversation. And accounts belonging to contractors or partners who are not in the HRIS, which need a decision about whether they belong in either process.

**Useful signal:** accounts flagged API-only, or with a profile that cannot log into the interface, are almost always non-human. Start there.

---

## Pattern 4: Audit and Event Logs by Actor

**What you are looking for:** things that acted, grouped by who acted.

**Where it lives:** field history, setup audit trail, event monitoring, API usage logs. Availability varies significantly by edition, and some of the most useful logs are add-ons rather than standard.

**What to capture:**

- Distinct actors appearing in the log
- Action volume per actor
- Which objects and fields each actor touched
- Time-of-day distribution
- First and last observed activity

**What the patterns tell you:**

- An actor writing at a consistent overnight hour is a scheduled integration
- An actor writing to a narrow field set at high volume is an enrichment or sync process
- An actor with a wide field footprint and irregular timing is either a human or an agent, and the field list usually distinguishes them
- An actor writing to fields outside any documented purpose is the finding worth chasing

**The retention limit.** Log-based discovery finds what has been active within the retention window. A credential that has not authenticated in longer than that will not appear, and dormant credentials are exactly the category that matters most. State the retention window alongside any log-derived list so the gap is visible.

---

## Pattern 5: Configuration Inspection in Connected Tools

**What you are looking for:** the credential each connected tool authenticates with, read from the tool rather than from the platform.

**Why this catches what the others miss.** A middleware platform, a data warehouse connector, or an ETL job may authenticate through a credential that appears in the platform as an unremarkable API user. Reading the configuration from the connecting side tells you what it is actually for.

**Where to look:** integration or connection settings in every tool in the revenue stack. Marketing automation, sales engagement, CS platform, billing, data warehouse, iPaaS, BI tool, enrichment providers.

**What to capture:**

- Which credential the tool authenticates with
- What the tool is configured to read and write
- Sync frequency and direction
- Who configured it

**Practical note.** This is the most manual of the five patterns and the one most likely to require asking the tool owner rather than reading a screen. Budget accordingly.

---

## Reconciling the Results

The five patterns produce overlapping candidate lists. Reconcile before populating the repository.

**Deduplicate on the credential, not the name.** One service account can appear as a connected app, an unmatched user, and a log actor. These are one row.

**Classify each candidate:**

| Classification | Meaning | Next step |
|---|---|---|
| Confirmed non-human | Identified, purpose known, owner identified | Populate the repository |
| Candidate | Found, purpose unclear or unowned | Investigate with the system owner |
| Stale human | Belongs to a former employee | Route to offboarding, not to this repository |
| Unresolved | Cannot determine what it is | Keep on the list. Do not close as resolved. |

**Do not close unresolved candidates as noise.** An account nobody can identify is the most interesting thing discovery produces, and it is the one most likely to get dropped because it is hard.

---

## Reporting Coverage

Every discovery output states its own limits. Include:

- Which systems were searched and which were not
- Which patterns were run in each system
- Log retention window for anything log-derived
- Count by classification, including unresolved
- Known gaps, named specifically

A candidate list without a coverage statement reads as complete. It never is.
