# Weighting Governance Log

Four records. The structural weight register, which holds the agreed encoding of how this business works. The guardrail ranges, which bound contextual weighting. The contextual run log, which captures every analysis run under non-default weighting. And the approval record, which documents what leadership actually agreed to.

All four exist for the same reason. A causal model that nobody can audit is a model that produces whatever the person running it wanted.

---

## Part 1: Structural Weight Register

The standing configuration. One row per structural causal factor per cohort.

### The split

The structural and contextual split is a structural decision and lives at the top of this register.

| Field | Value |
|---|---|
| **Current split** | 70% structural / 30% contextual |
| **Reasoning** | Context object has 3 quarters of history, entries skew Confirmed, CRM data is well maintained |
| **Approved by** | Named |
| **Approved date** | |
| **Last reviewed** | |

### Structural factor weights

| Column | What to capture |
|---|---|
| **Factor** | The structural causal factor. Champion engagement change, deal size mix shift, stage velocity, rep tenure, activity levels, discount depth, product mix, source and channel. |
| **Cohort** | Where this weight applies. Company ARR range, product, motion, or All. |
| **Weight** | The value, on whatever scale your model uses. Be consistent. |
| **Basis** | Evidence-based or starting assumption. Say which. |
| **Evidence** | What supports this weight. A loss analysis, a cohort comparison, a win/loss interview pattern. If the basis is assumption, write the reasoning instead. |
| **Approved by** | Named people, not roles. The revenue leaders whose narratives run on this. |
| **Approved date** | |
| **Last reviewed** | |

### Example rows

| Factor | Cohort | Weight | Basis | Evidence | Approved by | Approved date | Last reviewed |
|---|---|---|---|---|---|---|---|
| Champion engagement change | $250K+ ARR | High | Evidence-based | 2025 loss analysis: 14 of 22 losses above $250K had a stakeholder engagement drop in the final 90 days. Deals with champion continuity closed at 41% vs 19% without. | J. Okafor (CRO), M. Reid (VP CS) | 2026-04-12 | 2026-07-15 |
| Champion engagement change | Under $25K ARR | Low | Evidence-based | Same analysis. Cycle averages 31 days, engagement drop occurred in 3 of 61 losses. | J. Okafor (CRO) | 2026-04-12 | 2026-07-15 |
| Stage velocity | All | Medium | Starting assumption | No isolated evidence yet. Velocity correlates with outcome but has not been separated from deal size effects. Flagged for evidence at next review. | J. Okafor (CRO) | 2026-04-12 | 2026-07-15 |
| Discount depth | $100K-$250K ARR | Medium | Evidence-based | Deals discounted above 20% closed at 34% vs 29% at lower discounts, but with 11% lower ACV. Net effect roughly neutral. | J. Okafor (CRO), D. Lin (CFO) | 2026-06-30 | 2026-07-15 |

**On the basis column:** a register where every weight says evidence-based is a register nobody was honest about. Most models start mostly assumption. Marking that plainly is what makes the review meaningful, because starting assumptions are the queue of things to go find evidence for.

### Structural change log

Every change to a structural weight or to the split. No exceptions.

| Date | What changed | From | To | Reason | Approval-by-example run? | Approved by |
|---|---|---|---|---|---|---|
| 2026-06-30 | Discount depth, $100K-$250K | Low | Medium | H1 analysis found a real but offsetting effect on close rate and ACV | Yes, re-ran Q1 and Q2 | J. Okafor, D. Lin |
| 2026-04-12 | Initial configuration | n/a | 70/30 split, all structural weights, all guardrails | Initial model | Yes, ran against Q3 2025 and Q4 2025 | J. Okafor, M. Reid, D. Lin |

**Read this log for patterns, not just entries.** A weight that has moved in the same direction three reviews running is either tracking a real shift in the business, which is worth naming explicitly and logging in the context object, or being fitted to recent results. The log is the only place that distinction is visible.

---

## Part 2: Guardrail Ranges

Bounds on how much any single contextual causal factor can contribute within the contextual portion of the model. Set by the administrator, approved by leadership, varied by nobody.

These are expressed as a share of the contextual allocation. At a 70/30 split, a factor capped at 40% of contextual can contribute at most 12% of the total explanation.

| Contextual factor | Min share | Max share | Notes |
|---|---|---|---|
| Competitor Repositions | 0% | 100% | Full range. Competitive impact varies enormously by situation. |
| Pricing or Packaging Change | 10% | 100% | Cannot be zeroed. A confirmed pricing change is a real event and cannot be weighted out of existence. |
| Enablement Change | 0% | 40% | Capped. Real, but rarely the dominant cause of a cohort-wide movement. |
| Team Change | 0% | 40% | Capped for the same reason. |
| Market Shift | 0% | 30% | Capped deliberately. This is the category most often used to externalize an internal cause. |
| Product Change | 0% | 100% | Full range. |
| Process Change | 0% | 40% | |
| Strategic Decision | 10% | 100% | Cannot be zeroed. A deliberate leadership decision that affected revenue is not optional context. |

**Two design principles behind the caps:**

Categories that let a team attribute an outcome to something outside their control get capped. Market shift is the clearest case. It is the "macro headwinds" of causal modeling, and left uncapped it absorbs every uncomfortable finding.

Categories representing confirmed internal decisions cannot be zeroed. If pricing changed and revenue moved, the pricing change is in the analysis. How heavily is a judgment. Whether is not.

**Confidence caps apply on top of category caps.** An entry logged as Speculative cannot exceed half its category's maximum, regardless of who is running the analysis. A Speculative competitor repositioning caps at 50% of contextual rather than 100%. See `context-object-schema.md`.

**Shares are relative, so they survive a split change.** If you move from 70/30 to 60/40, every contextual factor's absolute contribution rises proportionally without anyone rewriting this table.

**Categories must match the context object.** These rows correspond one to one with the categories in `context-object-schema.md`. If you add or rename a category there, add or rename it here in the same change. A category with no guardrail range has no cap.

---

## Part 3: Contextual Run Log

Every analysis run under non-default weighting.

| Column | What to capture |
|---|---|
| **Run ID** | |
| **Date** | |
| **Run by** | Named person |
| **Question asked** | The causal question. "Why did win rate drop for $250K+ ARR accounts in Q2." |
| **Purpose** | Board prep, QBR, internal investigation, exploration. |
| **Weighting used** | Default, or the specific deviations from default |
| **Context entries weighted** | Which entries from the context object informed this run, with their confidence levels |
| **Output summary** | The conclusion the run produced, in one or two sentences |
| **Differs from default?** | Yes / No. If yes, how materially. |
| **Published?** | Did this run inform a deck, a decision, or a communication |

### Example rows

| Run ID | Date | Run by | Question | Purpose | Weighting | Output summary | Differs? | Published? |
|---|---|---|---|---|---|---|---|---|
| CW-041 | 2026-07-08 | M. Reid | Why did win rate drop for $250K+ accounts in Q2 | Board prep | Default | Champion engagement drop in 3 of 5 losses is the dominant factor. Competitive pricing secondary. | No | Yes, Q2 board deck |
| CW-042 | 2026-07-08 | M. Reid | Same | Board prep, alternative | Competitor Repositions raised from 40% to 80% of contextual | Competitive displacement becomes co-dominant with champion engagement. Forecast implication shifts by roughly 4 points. | Yes, materially | No |
| CW-043 | 2026-07-09 | J. Okafor | Same | Board prep, alternative | Pricing Change raised from 30% to 70% of contextual | Pricing becomes the leading factor. Champion engagement drop reads as a symptom of longer cycles rather than an independent cause. | Yes, materially | No |

**What this example shows:** three runs, one published, two explored. The board deck went out on the default weighting, and the two alternatives are logged and available if a director asks whether other explanations were considered. That is a much stronger position than having run only the one that supported the preferred narrative.

### What to review in this log

- **Runs immediately before high-stakes meetings that deviate materially from default.** Not automatically wrong. Automatically worth a look.
- **Whether the person running the analysis has a stake in the conclusion.** A leader exploring why their own function underperformed should not be the only person who touched the weights.
- **Published runs that used non-default weighting.** These should be rare and the deviation should be stated in the output itself.
- **Questions where every run reaches the same conclusion regardless of weighting.** A good finding. The conclusion is robust and the disagreement was less consequential than it felt.
- **Questions where the conclusion flips under reasonable weightings.** Also a good finding, and a more urgent one. The disagreement is the thing to resolve, not the analysis. Report it as such rather than picking a side.

---

## Part 4: Approval by Example Record

One record per approval cycle. Initial, and each review.

| Field | Detail |
|---|---|
| **Cycle** | Initial, or review date |
| **What was approved** | The split, the structural weights, the guardrail ranges. All three. |
| **Quarters tested** | Which known periods the model was run against |
| **Leadership's understanding, captured first** | What leadership believed caused each tested quarter's outcome, recorded before they saw any model output |
| **Model's explanation** | What the model said caused it |
| **Match?** | Did they align. Where they diverged, on what. |
| **Alternatives shown** | Which alternative configurations were presented, and what varied in each |
| **Changes made as a result** | To the split, to structural weights, to guardrails |
| **Approved by** | Named |
| **Date** | |

### Example record

| Field | Detail |
|---|---|
| Cycle | Initial |
| What was approved | 70/30 split. 14 structural weights across 4 ARR cohorts. 8 guardrail ranges. |
| Quarters tested | Q3 2025, Q4 2025 |
| Leadership's understanding, captured first | Q3: "We lost the quarter on pricing, the April increase was too aggressive for mid-tier." Q4: "Two AEs left in October and the territory never recovered." |
| Model's explanation | Q3: pricing dominant, matches. Q4: model attributed the decline primarily to deal mix shift, with rep departure secondary. |
| Match? | Q3 aligned. Q4 diverged. Model saw a mix shift leadership had not noticed. Investigation confirmed the mix shift was real and preceded the departures. |
| Alternatives shown | 60/40 split against 70/30. Champion engagement High against Medium for $250K+ cohort. |
| Changes made as a result | Rep tenure weight lowered from High to Medium after Q4 divergence. Split held at 70/30. |
| Approved by | J. Okafor (CRO), M. Reid (VP CS), D. Lin (CFO) |
| Date | 2026-04-12 |

**Capture leadership's understanding before showing them the model output.** Otherwise you are testing whether they will agree with a plausible explanation, which they usually will. The test only works if their answer exists independently.

**Divergence is the useful output.** The Q4 example above is what a good approval cycle looks like. The model and leadership disagreed, investigation resolved it, and a weight changed as a result. If the model had matched leadership on both quarters, the cycle would have confirmed the configuration without testing it.

---

## Review Cadence

| Record | Cadence | Owner |
|---|---|---|
| Structural weight register, including the split | Quarterly, alongside the semantic layer review | RevOps |
| Guardrail ranges | Quarterly | RevOps |
| Contextual run log | Reviewed quarterly for patterns. Individual entries reviewed when published. | RevOps |
| Approval by example | Quarterly at minimum, and on any material structural change | RevOps, with leadership |

---

## Common Mistakes

**A register with no assumption-based weights.** Nobody has evidence for every factor. A register claiming otherwise was not filled in honestly, and the starting assumptions that should be queued for validation are invisible.

**Guardrails set generously to avoid friction.** Uncapped ranges make the log a record of manipulation rather than a control against it. If a range has never constrained anyone, it is not doing anything.

**Logging runs but never reading the log.** The value is in the patterns. Nobody reading it quarterly means the log is documentation theater.

**Skipping the pre-disclosure step in approval by example.** Showing leadership the model's answer and asking whether they agree produces agreement. Capture their answer first.

**Approving weights without approving guardrails.** A defensible set of structural weights paired with an uncapped market shift category still produces a model that can externalize any uncomfortable finding. All three get approved together.

**Publishing under non-default weighting without saying so.** This is the failure the whole system exists to prevent. If the deck used an alternative weighting, the deck says so.
