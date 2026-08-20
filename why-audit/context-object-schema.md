# Context Object Schema

This document defines the structure for a non-deal-specific context object — a structured place where subjective, environmental, and organizational factors that affect revenue outcomes are logged. It holds the contextual causal factors that causal-model-setup/SKILL.md weights, and satisfies Requirement 3 in the why-audit scorecard.

Most CRMs capture what happened inside a deal. This object captures what happened around it.

---

## Why This Exists

When AI answers a causal question ("why is churn up," "why did we lose these deals"), it reasons over whatever data it can access. Deal-level CRM data gives it stage, activity, and outcome. What it almost never has access to is the context that surrounds those deals: a competitor repositioned, the team lost two senior AEs, enablement changed the pitch deck mid-quarter, or pricing strategy shifted.

Without this layer, the AI fills the gap with plausible guesses built from correlated deal-level signals. With it, the AI can consider factors that actually drove outcomes but were never attached to a specific opportunity record.

This object is not a CRM field on a deal. It's a standalone, time-stamped log of organizational and market context that the AI references alongside deal data when generating causal answers.

---

## Schema

### Core Fields (every entry requires these)

| Field | Type | Description |
|---|---|---|
| `entry_id` | Auto-generated | Unique identifier for each entry |
| `date_logged` | Date | When this entry was created |
| `effective_date` | Date | When the change or event actually occurred (may differ from date_logged) |
| `category` | Picklist | Which type of context this is (see Category Definitions below) |
| `summary` | Short text (150 char max) | One-line description of what changed |
| `detail` | Long text | Full explanation, as specific as possible |
| `logged_by` | User | Who created this entry |
| `source` | Picklist | Where this information came from (see Source Definitions below) |
| `confidence` | Picklist | How certain the logger is that this is accurate: Confirmed / Probable / Speculative |
| `active` | Boolean | Whether this factor is still in effect or has been resolved |

### Optional Fields (use when applicable)

| Field | Type | Description |
|---|---|---|
| `date_resolved` | Date | When this factor stopped being active (leave blank if ongoing) |
| `affected_segment` | Multi-select | Which ICP segments, regions, or deal types this most affects |
| `affected_metric` | Multi-select | Which revenue metrics this is most likely to influence (churn, win rate, deal velocity, ACV, pipeline coverage) |
| `related_deals` | Lookup | Link to specific deals if the context is partially deal-specific (e.g., a competitor undercut pricing on three specific opportunities) |
| `supporting_evidence` | URL or attachment | Link to a source document, Slack thread, call recording, or article that supports this entry |

---

## Category Definitions

Each entry must be tagged with exactly one category. These are intentionally broad — the detail field carries the specifics.

| Category | What belongs here | Examples |
|---|---|---|
| **Competitor Repositions** | A competitor changed pricing, positioning, product, or GTM approach in a way that could affect your pipeline or retention | "Competitor X launched a free tier targeting our SMB segment"; "Competitor Y hired away our former VP Sales" |
| **Enablement Change** | An internal change to how reps sell — pitch, messaging, demo flow, objection handling, onboarding, training | "New pitch deck rolled out week of June 3"; "Objection-handling training completed for enterprise team" |
| **Pricing or Packaging Change** | Any change to how your product is priced, packaged, discounted, or bundled | "Removed annual discount for new customers starting July 1"; "Added usage-based pricing tier for mid-market" |
| **Team Change** | Hiring, departures, reorgs, territory reassignments, capacity shifts, or burnout signals that affect revenue execution | "Lost 2 senior AEs in enterprise segment"; "SDR team reassigned from outbound to inbound for Q3" |
| **Market Shift** | External market changes — regulatory, economic, industry trend, buyer behavior shift — not tied to a specific competitor | "New compliance requirement in healthcare vertical"; "Budget freeze signals across Series B SaaS" |
| **Product Change** | Changes to your own product that affect how it sells, retains, or expands — feature launches, deprecations, reliability issues | "Core integration with Salesforce broke for 3 days in week 2"; "New reporting feature launched, early signal of strong expansion interest" |
| **Process Change** | Changes to internal revenue processes — lead routing, handoff procedures, approval workflows, stage definitions | "Revised MQL definition effective August 1"; "Added VP approval gate for discounts over 20%" |
| **Strategic Decision** | Leadership decisions that affect revenue direction but don't fit neatly into other categories | "CEO decided to deprioritize SMB segment for Q4"; "Board requested pivot toward enterprise" |

These categories are the contextual causal factors the causal model weights. If you add or rename a category, update the guardrail ranges in weighting-governance-log.md in the same change. A category with no guardrail range has no cap.

---

## Source Definitions

| Source | When to use |
|---|---|
| **Direct observation** | The logger personally witnessed or participated in the event |
| **Internal communication** | Learned from a Slack thread, email, all-hands, or internal meeting |
| **Rep feedback** | A rep or frontline team member reported this (flag confidence accordingly) |
| **External intelligence** | Sourced from a competitor's website, a news article, an industry report, or a prospect mentioning it on a call |
| **Data signal** | Identified from a pattern in CRM/BI data rather than a human report (e.g., "win rate dropped 12 points in segment X, investigated and found...") |
| **AI-generated** | An AI agent surfaced this signal from transcripts, emails, or other data — requires human validation before confidence is set above Speculative |

---

## Maintenance Rules

1. **Cadence:** Review and update at least once per month. A quarterly-only cadence is too slow — context decays faster than that, and stale entries actively mislead AI reasoning.

2. **Ownership:** One named person owns this object. Not a team, not a channel, one person. They don't need to write every entry, but they're responsible for making sure it's current, accurate, and actively referenced.

3. **Resolve, don't delete:** When a factor is no longer active (the competitor's free tier failed, the rep who left has been replaced), set `active` to false and fill in `date_resolved`. Don't delete the entry — historical context matters for understanding past outcomes.

4. **Confidence hygiene:** Anything sourced from `AI-generated` or `Rep feedback` starts at Speculative or Probable, never Confirmed, until independently verified. This prevents the same problem the context object is meant to solve — a plausible-sounding claim getting treated as fact.

5. **AI access:** The AI agent answering causal questions should be able to read this object filtered by `effective_date` range and `active` status. It should never be able to write to it directly — entries are human-authored or human-approved only.

---

## How AI Uses the Confidence Field

The confidence field changes how an entry can be used in a causal answer. It is not decoration.

**Disclose it.** When an AI-generated explanation references a context entry, it states the entry's confidence level and source alongside the claim. "This coincides with a competitor repositioning logged October 3 (Probable, sourced from rep feedback)" rather than presenting it as established fact.

**Weight it.** Confirmed entries carry more weight than Probable, which carry more than Speculative. A Speculative entry cannot exceed half its category's maximum share of the contextual allocation, regardless of who is running the analysis. See `weighting-governance-log.md`.

**Never let Speculative carry a causal claim alone.** If the only support for an explanation is a Speculative entry, the correct output is that the cause is unconfirmed. Naming the speculative factor as the cause is exactly the failure mode this object was built to prevent.

**Flag heavy reliance.** When an analysis depends materially on an entry below Confirmed, say so explicitly. That flag is a prompt for a human to verify the entry, upgrade its confidence, or retire it.

**AI does not upgrade confidence.** An agent can propose that an entry looks confirmable and cite what would confirm it. Only a human changes the field.

---

## How This Connects to Other Skills

**causal-model-setup:** the categories in this object are the contextual causal factors the model weights. The confidence field feeds the weighting cap. The `effective_date` determines which entries are in scope for a given period's analysis.

**why-audit:** Requirement 3 is scored based on this object.

- **Present:** exists, actively maintained (updated within the last 30 days), has entries across at least 4 of the 8 categories, and the AI can read it when generating causal answers.
- **Partial:** some context is captured but not in a structured, searchable form. Living in Slack, meeting notes, or a leader's memory. Or the object exists but hasn't been updated in 60+ days.
- **Missing:** no structured capture of non-deal context. The AI reasons over deal-level CRM data only.
