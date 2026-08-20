# Four Requirements Scorecard

Use this scorecard to evaluate whether an AI-generated causal answer (any "why" question about a revenue outcome) has the infrastructure underneath it to be trustworthy. Score each requirement as Present, Partial, or Missing.

---

## How to Score

| Score | Definition |
|---|---|
| **Present** | Fully built, documented, and actively maintained. A new team member could find it and understand it without asking someone. |
| **Partial** | Exists in some form but incomplete, outdated, or only in one person's head. Would not survive that person leaving. |
| **Missing** | Does not exist. The AI is reasoning without this layer entirely. |

---

## Requirement 1: Causal Model

**What it is:** A defined set of instructions that tells AI how to weight probable causes, distinguish correlation from causation, and rank competing explanations for a revenue outcome.

**What "Present" looks like:**
- Written rules specifying which variables the AI should evaluate when answering a causal question (e.g., champion status change, pricing change, competitor activity, rep behavior change, enablement change)
- Explicit weighting logic that can be adjusted (e.g., champion departure is weighted higher than a minor pricing concession for enterprise deals, but not for SMB)
- Clear separation between "these factors correlate with the outcome" and "this factor is the most likely cause" — the model doesn't treat correlation as causation by default
- Structural and contextual factors are distinguished, with a documented split between them
- Weights have a named owner, documented reasoning, and leadership sign-off through approval by example
- Guardrail ranges bound how far contextual weights can be varied, and every weighting run is logged

**What "Partial" looks like:**
- Some variables are defined but weighting is implicit or hardcoded — nobody can easily adjust which factors matter more
- The AI considers whatever fields exist in the CRM without guidance on which ones actually drive outcomes
- A senior leader informally knows how to weight causes but hasn't documented it
- Weights exist and are adjustable but were never approved by the leaders whose narratives depend on them
- No guardrails on contextual weighting, so any conclusion can be produced by adjusting inputs
- Weighting runs are not logged, so nobody can tell whether an output used the standing configuration

**What "Missing" looks like:**
- The AI generates a "why" answer by pattern-matching across whatever data it can access, with no instruction on how to rank or weight causes
- No one has defined what "caused" means in this context — the AI treats any correlated signal as a plausible explanation

**Questions to ask:**
- If I changed the weight of "champion departure" from high to low, would the AI's answer change? If you can't do this, the model is missing.
- Can someone who didn't build the system explain how it decides which cause to surface first?
- Who approved these weights, and did they see example outputs before approving?
- Can someone produce a conclusion they wanted by adjusting weights, and would anyone know?

For building and governing this, see causal-model-setup/SKILL.md.

---

## Requirement 2: Semantic Layer

**What it is:** A documented, single-source definition of how every metric the AI references is calculated, including which fields feed it and which are excluded.

**What "Present" looks like:**
- A glossary or data dictionary specifying the exact calculation for each metric (churn, win rate, pipeline coverage, ARR, etc.)
- Field-level documentation: which CRM/BI fields feed each metric, which are excluded, and why
- Versioned and owned — someone is named as responsible for updating it, and changes are logged

**What "Partial" looks like:**
- Definitions exist but are spread across multiple docs, Slack threads, or tribal knowledge
- Different teams use slightly different definitions of the same metric (e.g., finance calculates churn one way, CS calculates it another)
- The AI has access to the fields but no instruction on which calculation to use

**What "Missing" looks like:**
- No documented metric definitions — the AI infers what "churn" means from field names or context
- Two people asking "why is churn up" could get answers based on different underlying calculations without either of them knowing

**Questions to ask:**
- If I asked the AI "how did you calculate churn for this answer," would it give me a specific, documented formula, or a best guess?
- Do sales, finance, and CS agree on the definition of every metric the AI is answering questions about?

---

## Requirement 3: Context Object

**What it is:** A structured place — separate from deal-specific CRM fields — where non-deal-specific context that affects revenue outcomes is logged. This is the subjective, interpretive layer most systems never capture.

**What "Present" looks like:**
- A defined object (custom object, structured doc, or dedicated system) where the team logs: enablement changes, market shifts, competitor moves, team capacity/burnout signals, pricing strategy changes, internal reorgs
- Updated within the last 30 days, with entries across at least four of the eight categories
- Readable by the AI when generating causal answers, filtered by effective date and active status

For the full schema, see causal-model-setup/references/context-object-schema.md.

**What "Partial" looks like:**
- Some of this context exists in Slack threads, meeting notes, or a leader's memory, but it's not structured or searchable, or the object exists but hasn't been updated in 60+ days
- The AI doesn't have access to it even if it exists somewhere in the org
- One or two categories are tracked (e.g., competitor moves) but others aren't (e.g., enablement changes, team burnout)

**What "Missing" looks like:**
- No structured capture of non-deal context at all
- When someone asks "why did win rate drop," the only data available is deal-level CRM fields — everything else requires a human to reconstruct from memory
- The AI fills this gap by generating plausible explanations from whatever deal data correlates with the outcome

**Questions to ask:**
- If a new RevOps hire started tomorrow, could they find a record of what changed in the business last quarter beyond what's in the CRM?
- When the AI answers "why did we lose these deals," does it have access to anything that happened outside of those specific deals?

---

## Requirement 4: Data Readiness

**What it is:** Whether the raw data the AI needs to reason about deals actually exists, is current, and is trustworthy.

**What "Present" looks like:**
- Call recordings are transcribed with sentiment and key-moment tagging
- Email threads are captured with sentiment analysis
- Won/loss reasons include both a structured picklist AND a free-text field written by the rep
- Data hygiene is actively maintained — duplicates are merged, stale records are flagged, fields have validation rules
- The data covers enough history to identify patterns (not just last month)

**What "Partial" looks like:**
- Some data sources are connected but others aren't (e.g., calls are transcribed but emails aren't captured)
- Won/loss reasons exist but are picklist-only with no free text, or reps select "Other" 40% of the time
- Data exists but integrity is poor — duplicates, missing fields, inconsistent formatting
- Transcripts are available but not processed for sentiment or key moments

**What "Missing" looks like:**
- No call transcription or email capture
- Won/loss reasons are either not required or not enforced
- CRM data integrity is poor enough that even retrieval questions ("what's our win rate") return unreliable answers
- If the data can't support retrieval questions, it definitely can't support causal ones

**Questions to ask:**
- What percentage of closed-lost deals have a substantive loss reason beyond a picklist selection?
- If I pulled all calls from last quarter, would they be transcribed and searchable, or sitting as unprocessed audio files?
- When was the last time someone audited CRM data quality? Who owns that process?

---

## Scoring Summary

| Requirement | Present | Partial | Missing |
|---|---|---|---|
| 1. Causal Model | | | |
| 2. Semantic Layer | | | |
| 3. Context Object | | | |
| 4. Data Readiness | | | |

**Interpretation:**

- **4 Present:** The AI's causal answers are worth stress-testing. They're still hypotheses, not facts, but they're built on real infrastructure. Human validation is a quality check, not a safety net.
- **3 Present, 1 Partial:** Usable with caution. Identify which requirement is Partial and flag it explicitly when reading any AI-generated explanation — the gap will show up in that specific dimension.
- **2 or fewer Present:** The AI is generating plausible stories, not validated explanations. Treat every causal answer as a starting point for manual investigation, not an input to a decision.
- **Any requirement Missing:** Do not use AI-generated causal answers in board materials, forecast narratives, or rep coaching without full manual reconstruction of the "why." The infrastructure isn't there yet.
