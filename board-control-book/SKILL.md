---
name: board-control-book
description: "Builds and maintains a standing board-ready reporting package for revenue teams — same format every cycle, AI-touch disclosed, named owners, so board conversations are about what the numbers mean rather than what the numbers are. Triggers on 'board deck,' 'board prep,' 'board reporting,' 'board meeting,' 'quarterly review,' 'investor update,' 'forecast review,' 'board-ready,' 'QBR deck,' or any request to prepare revenue metrics or narrative for board-level consumption. BOUNDARY: For validating whether AI-generated causal explanations in the report are trustworthy, see why-audit. For incident response when an AI-generated number reaches the board unchecked, see agentic-incident-playbook."
---

# Board Control Book — Revenue Reporting That Survives Scrutiny

You are helping someone build a standing revenue reporting package that eliminates the quarterly scramble, makes AI-touched numbers explicitly visible, and ensures the board conversation is about meaning, not reconciliation.

The goal is not a better slide deck. It's a reporting architecture that works the same way every cycle so the people in the room can focus on what changed, what it means, and what happens next.

---

## The Problem This Solves

Most revenue teams rebuild their board reporting from scratch every quarter. Someone spends a week pulling numbers from five systems, reconciling discrepancies between what sales sees and what finance sees, and formatting slides that will be outdated by the time the meeting starts.

Three things go wrong consistently:

**1. The number changes depending on who pulls it.** Sales has one pipeline view, finance has another, and nobody agrees which goes in the deck. This usually traces back to undefined or inconsistent metric definitions upstream — not a presentation problem, a semantic layer problem.

**2. The meeting becomes about the numbers, not the meaning.** Directors spend the first 30 minutes asking "how did you calculate this" or "why does this differ from last quarter's format" instead of asking "what does this mean for next quarter." Every minute spent reconciling is a minute not spent on strategy. Boards themselves report spending roughly a third of their time — in meetings and reviewing materials — on financial performance. If that third is wasted on reconciliation, the strategic conversation never happens.

**3. When a quarter goes badly, the explanation defaults to vague language.** "Macro headwinds." "Timing dynamics." "Pipeline softness." A board that's sat through enough decks can feel the difference between an honest explanation and a managed one. The vague phrase does more damage than the miss itself.

---

## The Control Book Concept

A control book is a standing revenue report delivered to board and management in the same format every cycle. Same structure. Same metrics. Same order. Same owner.

When the format is consistent, directors stop asking what the numbers are. They already know where to find them. The conversation moves immediately to what changed and why — which is the only part of a board meeting that actually produces decisions.

**The format is the discipline.** It's not a template for making slides look better. It's an operating commitment that says: these are the numbers we track, this is how we calculate them, this is who owns them, and this is how we explain variance.

---

## What Goes in the Control Book

### Section 1: Financial Summary

| Component | What it covers | Owner |
|---|---|---|
| P&L snapshot | Revenue, gross margin, operating expenses, EBIT — actuals vs. plan vs. prior period | CFO |
| Cash position | Cash on hand, burn rate, runway — actuals vs. forecast | CFO |
| ARR movement | Starting ARR, new, expansion, contraction, churn, ending ARR — with waterfall | CRO |

**Rules:**
- Same format every cycle — no restructuring the waterfall categories quarter to quarter
- Variance explanations are required for any line item that moved more than 10% from plan (threshold adjustable per org)
- If any number in this section was generated or influenced by an AI tool (e.g., an AI-generated forecast that fed the plan), flag it explicitly with the label `[AI-assisted]` and name the tool

### Section 2: Pipeline and Forecast

| Component | What it covers | Owner |
|---|---|---|
| Pipeline coverage | Current pipeline vs. target, by segment and stage, with coverage ratio | CRO / RevOps |
| Forecast summary | Committed, best case, and upside — with movement from prior period | CRO / RevOps |
| Stage conversion rates | Stage-to-stage conversion rates vs. historical average | CRO / RevOps |
| Deal velocity | Average days in stage, by segment — current vs. trailing 3 quarters | CRO / RevOps |
| Lost rates by stage | Loss rates and categorized loss reasons at each funnel and opportunity stage, broken down by source/channel, region, industry, product, and rep | CRO / RevOps |

**Rules:**
- Pipeline is reported after hygiene filters are applied — excluding deals past their close date that haven't been updated, zombie deals with no activity in 45+ days, and test/duplicate records. The board should see the pipeline the CRO is actually working, not the raw unfiltered number that includes dead weight.
- Forecast categories (commit, best case, upside) have written definitions that don't change between cycles — if a CRO and a rep would categorize the same deal differently, the definitions need tightening
- If pipeline scoring or forecast weighting uses an AI model, disclose it: name the model, when it was last calibrated, and what its historical accuracy has been
- Compare stage conversion rates against your own historical baseline first. External benchmarks can add context but shouldn't substitute for understanding your own motion's patterns.
- Lost rates are tracked at every stage, not just closed-lost. Where in the pipeline deals die is as important as the final win rate. Loss reasons use a defined taxonomy and should be reported quarterly to each department, especially product.

### Section 3: Marketing Performance

| Component | What it covers | Owner |
|---|---|---|
| Marketing-sourced and marketing-influenced pipeline | Pipeline generated and influenced from marketing channels — by source/channel, with conversion rates from lead to opportunity | CMO |
| Lead volume and quality | Total leads, MQLs, and SQL conversion rates — by source/channel, with trend vs. prior period | CMO |
| Marketing contribution to revenue | Percentage of closed-won revenue attributable to marketing-sourced pipeline | CMO / CRO |
| Channel performance | Spend, pipeline generated, and cost-per-opportunity by channel | CMO |

**Rules:**
- Attribution methodology must be documented and consistent across cycles. If the org uses multi-touch attribution, disclose the model. If first-touch or last-touch, state it. The board should never have to ask "how are you counting this."
- Marketing-sourced and marketing-influenced need to be explicitly defined.
- Marketing contribution to revenue is co-owned with the CRO because the number depends on how pipeline handoff is defined — if marketing and sales disagree on what counts as marketing-sourced, fix the definition before reporting it.
- Channel performance is reported with enough history (trailing 3-4 quarters minimum) to show trend, not just current-period spend. A channel that looks expensive this quarter may have a longer payback cycle that only shows up over time.
- If any lead scoring, attribution modeling, or content performance analysis uses AI, disclose it in Section 7 (AI Disclosure Log).

### Section 4: Retention and Expansion

| Component | What it covers | Owner |
|---|---|---|
| Gross and net retention | GRR and NRR — actuals vs. plan, trailing 4 quarters | CRO / CS |
| Churn detail | Lost customers with categorized reasons (voluntary/involuntary, by segment) | CRO / CS |
| Customer health summary | Global health score with component breakdown — product usage, support, account growth, feedback, advocate scores | CRO / CS |
| Expansion pipeline | Expansion opportunities by stage and expected close | CRO / CS |
| Product white space | Matrix of customers by product — showing what each customer has, what they don't, and where open expansion opportunities exist | CRO / CS |

**Rules:**
- Churn reasons use a defined taxonomy, not free text only — see `why-audit/references/context-object-schema.md` for the category structure
- If churn analysis or health scoring uses AI, disclose it and flag any score that triggered an action (e.g., "AI flagged this account as high-risk, CS intervened")
- Health scores are reported as both the current snapshot and the trend (improving, stable, declining) over the trailing 2-3 quarters. A single-point score without trend context invites misinterpretation.
- Product white space is reviewed quarterly to identify systematic expansion gaps — if 80% of customers with Product A don't have Product B, that's a strategic finding, not just a rep-level upsell list

### Section 5: Unit Economics

| Component | What it covers | Owner |
|---|---|---|
| CAC | Customer acquisition cost — total and marketing percentage of CAC, broken down by source/channel, segment, and product where possible | CFO / CRO |
| CLTV | Customer lifetime value — current calculation and trailing trend | CFO / CRO |
| CLTV:CAC ratio | Ratio of lifetime value to acquisition cost — actuals vs. target | CFO / CRO |
| CAC payback period | Months to recover acquisition cost, by segment | CFO / CRO |

**Rules:**
- CAC should include sales and marketing costs at minimum. More mature models add customer support cost for renewing customers — disclose which costs are included so the board knows what's in the number
- CLTV:CAC ratio is compared against the org's own historical trend, not a generic "3:1 is good" benchmark, unless the board specifically requests a benchmark comparison
- If CAC or CLTV calculations use AI-assisted attribution models, disclose it in Section 7 (AI Disclosure Log)

### Section 6: Variance Narrative

This is the section most teams skip or fill with vague language. It's the most important section in the control book.

**Structure for every variance explanation:**

1. **What happened.** The actual change, stated plainly. "Win rate dropped 8 points in enterprise segment between Q1 and Q2."

2. **Why it happened.** The mechanism — not "macro headwinds," but the specific factors. "Three of the five losses were deals where the champion changed roles mid-cycle. The other two were competitive losses to [competitor] on pricing." If this explanation was informed by an AI-generated analysis, it must pass the why-audit before inclusion. See `why-audit/SKILL.md`.

3. **What's changing.** What's already been adjusted and what's planned. "Revised champion-tracking process deployed in week 3. Pricing response playbook in progress, expected live by [date]."

4. **What it means for next period.** The forward-looking implication. "Enterprise pipeline coverage is currently 2.8x against a 3.5x target. If champion attrition continues at the current rate, Q3 forecast should be discounted by approximately [X]%."

**Rules:**
- Every variance above the threshold requires all four parts. "We're looking into it" is not an explanation.
- If the explanation uses language from the false-confidence-markers list (`why-audit/references/false-confidence-markers.md`), it needs to be rewritten before it reaches the deck.
- The narrative is owned by a named person, not generated by a tool. AI can inform it. A human writes it and stands behind it.

### Section 7: AI Disclosure Log

This section is new relative to traditional control books and reflects the current reality that AI is touching revenue data in most organizations.

| Field | Description |
|---|---|
| Tool / model | Which AI system touched this reporting cycle's data |
| Where it touched | Which metrics, forecasts, or analyses were AI-assisted |
| Last calibrated | When the model was last validated against actual outcomes |
| Override rate | How often a human overrode the AI's output this cycle (if tracked) |
| Known limitations | Any known gaps, biases, or failure modes in this model |

**Rules:**
- If no AI tools touched any part of this cycle's reporting, state that explicitly. The absence of disclosure is itself a data point the board should see.
- This section is not optional. Boards are increasingly expected to oversee AI governance (Grant Thornton's 2026 AI Impact Survey found only 11% of boards meet a basic threshold for strong AI oversight). A CRO who proactively discloses AI involvement is ahead of 89% of organizations.

---

## Building the Control Book for the First Time

**Step 1: Define the metrics.** Every metric in the control book must have a documented definition — exact calculation, which fields feed it, which are excluded. If this doesn't exist, start with `why-audit` Requirement 2 (semantic layer). The control book is only as trustworthy as the definitions underneath it.

**Step 2: Assign ownership.** Every section has one named owner. Not a team, one person. That person is accountable for accuracy, timeliness, and variance explanations in their section.

**Step 3: Establish the format with the board.** Present the structure to the board before the first live cycle and get alignment on what's included. Expect requests for additions or changes, especially in the first two cycles. The goal is to reach a stable format as quickly as possible so the board can build pattern recognition across quarters. Once stable, resist cosmetic changes, but treat board requests for new components or different breakdowns as legitimate inputs, not threats to the format. The discipline is consistency where it matters (same metric definitions, same ownership, same variance structure), not rigidity about which slides are in the deck.

**Step 4: Set variance thresholds by section.** Different metrics need different thresholds. A 10% swing in ARR is a fundamentally different signal than a 10% swing in deal velocity. Work with section owners and the board to define what counts as a variance worth explaining for each component. These thresholds may change over time at the board's request as the company matures or priorities shift. Document the current thresholds and update them explicitly rather than letting them drift informally.

**Step 5: Run a dry run.** Before the first live board meeting using this format, run the full package internally with the executive team 3-5 days before. Every question that comes up in the dry run is a question that would have come up in the board meeting — catch it early.

**Step 6: Add the AI disclosure log.** Audit every tool in your revenue stack. Anything that uses AI (forecasting, scoring, enrichment, sequencing, health scoring) gets logged. If you're unsure whether something uses AI, assume it does and investigate.

---

## Maintenance

- **Cadence:** The control book is updated on the same schedule as board meetings — typically quarterly, but the underlying data (pipeline, forecast, retention) should be refreshed monthly so the quarterly version is a snapshot, not a reconstruction.
- **Format reviews:** Review the format itself annually. Not to redesign it, but to check whether any metrics have become irrelevant or whether new ones need adding.
- **Ownership reviews:** Every time someone leaves or changes roles, reassign their section immediately. An unowned section will decay within one cycle.

---

## Voice Rules

- Plain language in the variance narrative — if a board member needs a glossary to understand the explanation, rewrite it
- State the miss before the explanation — don't lead with the excuse
- Short, declarative sentences in variance explanations — no hedging, no "it's worth noting that"
- If you're uncertain about a cause, say so — "We believe X based on Y, but we're still investigating Z" is more trustworthy than a confident guess. NO GUESSING!

---

## Reference Files

| File | When to read | What's inside |
|---|---|---|
| `references/control-book-template.md` | First-time setup | A blank template with all sections, ready to fill |
| `references/variance-explanation-examples.md` | Writing Section 6 | Good and bad examples of variance explanations |

## Related Skills

- **why-audit** — validates any AI-generated causal claim before it enters the variance narrative
- **agentic-incident-playbook** — response protocol when an AI-generated error reaches a board or decision
- **semantic-layer-setup** — builds the metric definitions the control book depends on
