---
name: causal-model-setup
description: >
  Builds and governs the causal model that tells AI how to weight
  probable causes when answering why questions about revenue outcomes.
  Covers structural weights owned by RevOps, contextual weights varied
  during analysis, the context object that holds non-deal-specific
  factors, approval by example, guardrail ranges, and narrative
  consistency across teams. Triggers on causal model, why did this
  happen, weight the causes, root cause analysis, competing
  explanations, we disagree on the cause, sensitivity analysis on
  causes, or any situation where AI needs to rank explanations rather
  than list them. BOUNDARY - For evaluating whether an existing causal
  answer is trustworthy, see why-audit. For the metric definitions this
  skill depends on, see semantic-layer-setup.
---

# Causal Model Setup — Teaching AI How to Weight Causes

You are helping someone build the layer that lets AI answer why questions with reasoning rather than free association.

Without a causal model, AI given a set of correlated signals produces the most plausible-sounding story it can construct. It cannot tell you whether a champion departure mattered more than a pricing change, so it names whichever one makes the cleanest narrative.

A causal model does two things. It encodes how this business actually works, which is stable. And it lets a human vary the weight on specific contextual claims, which is how genuine disagreement gets tested rather than argued.

---

## Prerequisite

**This skill requires a semantic layer.** A causal model reasons about why a metric moved, which requires knowing exactly what the metric is and which fields produce it. Build `semantic-layer-setup/SKILL.md` first.

---

## Two Kinds of Weights

The distinction is the foundation of this skill. Conflating them is how causal models become manipulable.

### Structural weights

**What they encode:** how much a given factor typically matters in this business, by segment, deal size, product, or motion. Champion departure weighted heavier than a minor pricing concession for enterprise, but not for SMB. Competitive displacement weighted heavier in one vertical than another.

**Who sets them:** RevOps, or whoever holds system administration for the causal model.

**Who approves them:** leadership, by example rather than by number. See Approval by Example below.

**How often they change:** rarely. Fine-tuning after a review, or a genuine shift in how the business operates. A structural weight that changes quarterly was not a structural weight.

**What they are not:** an opinion about a specific quarter. If a weight is being changed because of one bad quarter, that is a contextual judgment wearing structural clothing.

### Contextual weights

**What they encode:** how much a specific logged event should count toward explaining a specific outcome. A competitor released a feature in April. Two AEs left in May. Pricing changed in March. Each of those is an entry in the context object, and how heavily each one bears on this quarter's decline is a judgment.

**Who sets them:** a leader running analysis, within bounds the administrator defined.

**How often they change:** deliberately and often, during analysis. That is the point.

**What they are for:** testing how a conclusion moves under different reasonable assumptions. Two experienced people disagreeing about the cause is common and usually neither is wrong. They are weighting the same facts differently. Contextual weighting makes that disagreement testable instead of arguable.

### Why the split matters

If everything is adjustable, the model produces whatever the person running it wants. If nothing is adjustable, genuine disagreement gets resolved by whoever has the most authority in the room rather than by evidence.

Structural weights are the shared, agreed, slow-moving encoding of how the business works. Contextual weights are the fast, exploratory layer on top, bounded so exploration cannot become fabrication.

---

## The Context Object

Contextual weighting requires something to weight. That is the context object, a structured record of non-deal-specific factors that affect revenue outcomes.

Full schema in `references/context-object-schema.md`. In brief, each entry captures what changed, when it took effect, what category it falls into, who logged it, where the information came from, and how confident the logger is.

**Confidence and weighting interact.** An entry logged as Speculative can be weighted, but it cannot carry a causal claim alone. See the schema for the full rules on how AI uses the confidence field.

**AI never writes to the context object.** Entries are human-authored or human-approved. An agent that can log its own context and then reason over it has closed a loop nobody is watching.

---

## Approval by Example

Leadership cannot meaningfully approve a number. Nobody can evaluate whether champion departure should be weighted 0.4 or 0.6 in the abstract.

Leadership can meaningfully approve an output. Show them how a real past quarter would have been explained under the proposed weighting, and they can tell you whether that explanation matches what they believe actually happened.

**The process:**

1. **Draft the structural weights.** RevOps proposes, based on historical patterns and domain knowledge.

2. **Run them against known quarters.** Pick two or three past periods where leadership already agrees on what happened. Generate the causal answer the model would have produced.

3. **Compare to what leadership believes.** If the model's explanation of a known quarter doesn't match the shared understanding of that quarter, the weights are wrong. This is the actual test.

4. **Show the alternatives.** Present the same quarter under two or three different weightings, so leadership sees what they are choosing between rather than approving in isolation.

5. **Get explicit agreement.** Named sign-off from the revenue leaders whose narratives will be built on this model. Log who agreed and when.

6. **Re-run at review.** Structural weights are reviewed on a defined cadence. Same process, run against a recent quarter.

**Why step 3 is the important one:** a causal model that explains the past incorrectly will explain the present incorrectly. Testing against quarters where the answer is already known is the only validation available before the model is used on a quarter where it isn't.

---

## Guardrails on Contextual Weighting

Contextual weighting exists to test assumptions. It must not become a way to produce a preferred conclusion.

**Bounded ranges.** The administrator sets minimum and maximum weights for each context category. A leader can vary within the range. They cannot zero out a Confirmed pricing change or weight a Speculative rumor as the dominant cause.

**Every run is logged.** Who ran it, what weights they set, when, and what the output was. A weighting run the day before a board meeting that differs materially from the standing configuration is visible rather than silent.

**Weights are stated in the output.** Any causal answer produced under non-default weighting says so. "Under a weighting that emphasizes competitive displacement over pricing" is part of the answer, not a footnote.

**Default weighting exists and is the baseline.** There is always a standing configuration. Alternatives are explicitly labeled as alternatives.

**Nobody sets weights on their own performance.** A leader exploring why their own function underperformed should not be the only person who touched the weights. Not because of bad faith, but because the incentive is obvious and the appearance matters.

---

## Narrative Consistency Across Teams

Contextual weighting creates a real risk: sales runs one weighting, marketing runs another, and two incompatible stories arrive at the same board meeting.

**Rules:**

- **Structural weights are shared and singular.** There is one model. Teams do not maintain their own.
- **One weighting goes in the deck.** Exploration is fine and encouraged. Publication is singular.
- **Disagreement is surfaced, not resolved by volume.** If two teams reach different conclusions under different weightings, that is a finding worth reporting, not a conflict to settle offline. "Under weighting A the cause reads as competitive, under weighting B it reads as pricing, and we have not yet determined which is right" is an honest board statement.
- **The variance narrative names its weighting.** If a control book variance explanation was informed by a causal model run, state the weighting used. See `board-control-book/SKILL.md` Section 6.

---

## Building the Causal Model

**Step 1: Confirm the semantic layer exists.** Metric definitions and field documentation. Without them, the model is reasoning about undefined quantities.

**Step 2: Inventory your causal factors.** What actually explains revenue outcomes in this business? Start from your own closed-lost analysis, churn reasons, and win/loss interviews rather than a generic list. Typical categories: champion or stakeholder change, pricing and packaging, competitive activity, product gaps, rep performance and tenure, enablement changes, market and timing, process changes.

**Step 3: Segment the factors.** The same factor matters differently by segment, deal size, product, and motion. Champion departure is close to fatal in a twelve-month enterprise cycle and often survivable in a thirty-day SMB cycle. Document where each factor's weight varies.

**Step 4: Draft structural weights.** Base them on evidence where you have it. Historical loss analysis, cohort comparisons, anything where you can observe the factor's association with outcomes. Where you have no evidence, say so and mark the weight as a starting assumption to be revisited.

**Step 5: Build the context object.** See `references/context-object-schema.md`. Backfill at least the last two quarters so the model has something to reason over.

**Step 6: Run approval by example.** The six-step process above. Do not skip the alternatives step.

**Step 7: Set contextual guardrail ranges.** Minimum and maximum for each category. Document the reasoning.

**Step 8: Configure logging.** Every weighting run captured with who, what, when, and output.

**Step 9: Define the review cadence.** Structural weights quarterly at minimum, alongside the semantic layer review.

---

## What Good Looks Like

- The model explains two or three known past quarters in a way leadership recognizes as accurate
- Structural weights have a named owner, documented reasoning, and a last-approved date
- Every structural weight change is logged with who approved it
- A leader can run an alternative weighting without asking permission, and cannot exceed the guardrails
- Every causal output states which weighting produced it
- Sales, marketing, CS, and finance are reasoning from the same structural model
- When two teams disagree, the disagreement is documented rather than negotiated

---

## Common Failure Modes

**Weights set by whoever built the model, approved by nobody.** The most common failure. Leadership inherits a set of assumptions they never examined and then treats the outputs as objective.

**Approval by number.** Showing leadership a table of weights and asking for sign-off produces sign-off without comprehension. Show outputs.

**Structural weights that change every quarter.** If the weight on pricing keeps moving, either the business is changing that fast, which is worth knowing on its own, or someone is fitting the model to recent results.

**No default weighting.** Without a standing baseline, every run is an alternative and nothing is comparable across quarters.

**Contextual weighting used to reach a conclusion.** The tell is a weighting run immediately before a high-stakes meeting that differs materially from the default, by the person whose function is under scrutiny. This is exactly what the logging rule exists to make visible.

**AI writing to the context object.** An agent that logs its own context and then weights it has no external check at all.

**Treating model output as the answer.** Even a well-governed causal model produces a hypothesis. The output informs the narrative. It is not the narrative. See `why-audit/SKILL.md`.

---

## Voice Rules

- Always state the weighting that produced a causal output
- Distinguish structural from contextual weights explicitly in any documentation
- When a weight is a starting assumption rather than evidence-based, say so
- Never present a weighted output as more certain than the underlying context entries support

---

## Reference Files

| File | When to read | What's inside |
|---|---|---|
| `references/context-object-schema.md` | Step 5 | Field-level schema for the context object, categories, confidence rules, and how AI uses them |
| `references/weighting-governance-log.md` | Step 8 | Templates for the structural weight register and the contextual run log |

## Related Skills

- **semantic-layer-setup** — required prerequisite. Provides the metric definitions this model reasons about.
- **why-audit** — evaluates whether a causal answer is trustworthy. Requirement 1 of the scorecard is satisfied by this skill.
- **board-control-book** — Section 6 variance narratives should state which weighting informed them
- **agent-monitoring** — Type 4 detection screens causal outputs for false confidence markers
