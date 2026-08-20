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

## Two Kinds of Causal Factors

The distinction is the foundation of this skill. Conflating them is how causal models become manipulable.

### Structural causal factors

**What they are:** factors observable in your systems. The CRM, the BI layer, and the call platform already know about these. They're already logged.

- Champion or stakeholder engagement change
- Deal size and segment mix shift
- Stage velocity and where deals stall
- Rep tenure and ramp status
- Activity levels and multi-threading depth
- Discount depth and approval path
- Product mix
- Source and channel

**Who weights them:** RevOps, or whoever holds system administration for the causal model.

**Who approves the weights:** leadership, by example rather than by number. See Approval by Example below.

**How often they change:** rarely. Fine-tuning after a review, or a genuine shift in how the business operates. A structural weight that changes quarterly was not a structural weight.

### Contextual causal factors

**What they are:** factors only observable because a human logged them. These live in the context object and do not exist anywhere in your systems otherwise.

- Competitor repositions
- Pricing or packaging changes
- Enablement changes
- Team capacity and morale
- Market shifts
- Product changes and outages
- Process changes
- Strategic decisions

**Who weights them:** RevOps or a system admin recommends and sets the bounds in the systems after leadership approves the bounds. See Approval by Example below. A leader running analysis varies within the bounds when running analysis.

**How often they change:** deliberately and often, during analysis. That is the point.

**What they are for:** testing how a conclusion moves under different reasonable assumptions. Two experienced people disagreeing about the cause is common and usually neither is wrong. They are weighting the same facts differently. Contextual weighting makes that disagreement testable instead of arguable.

### The dividing line

Structural is what your systems know. Contextual is what someone had to write down.

Champion engagement dropping is structural, because activity data shows it. A competitor releasing a compliance module in April is contextual, because nothing in your CRM records that. Deal velocity slowing is structural. The pricing change that may have slowed it is contextual.

This matters because the two have different failure modes. Structural factors are reliable but incomplete, since your systems only capture what they were built to capture. Contextual factors fill that gap but depend entirely on someone having logged them accurately, which is why the context object has a confidence field.

### Why the split matters

If everything is adjustable, the model produces whatever the person running it wants. If nothing is adjustable, genuine disagreement gets resolved by whoever has the most authority in the room rather than by evidence.

Structural weights are the shared, agreed, slow-moving encoding of how the business works. Contextual weights are the fast, exploratory layer on top, bounded so exploration cannot become fabrication.

---

## The Structural and Contextual Split

Before weighting individual factors, set the overall split between the two categories.

**Suggested baseline: 70% structural, 30% contextual.**

Seventy percent of a causal explanation comes from patterns observable in your data. Thirty percent comes from environmental context a human recorded. That ratio reflects a simple reality: your systems capture a lot, but they do not capture why a competitor's move mattered or that the enterprise AEs are overworked compared to the SMB AEs.

**When there are no context records, the split is 100/0.** The model reasons from structural factors alone, and the output must state that no contextual records existed for the period analyzed. That absence is itself a finding. A quarter with no logged context is a quarter where nobody wrote down what was happening around the numbers.

**Customize the split for your business.** The baseline is a starting point, not a standard.

**Weight contextual higher when:**
- The context object has been maintained consistently for a year or more
- Entries are mostly Confirmed rather than Speculative
- Your market moves fast enough that environmental factors genuinely drive outcomes
- Your CRM data is thin, so structural factors have less to work with

**Weight structural higher when:**
- The context object is new or sparsely populated
- Entries skew Speculative
- Your CRM and call data are rich and well-maintained
- Your business is stable enough that most variance is internally explainable

**Review the split quarterly** alongside the structural weight register. A context object that has matured over four quarters justifies a different split than the one you set when it was empty.

**The split is a structural decision, not a contextual one.** It is set by RevOps, approved by leadership through the same approval-by-example process, and logged in the structural weight register. A leader running analysis varies weights within the contextual 30%. They do not change the 30% itself.

---

## The Context Object

Contextual weighting requires something to weight. That is the context object, a structured record of non-deal-specific factors that affect revenue outcomes.

Full schema in `references/context-object-schema.md`. In brief, each entry captures what changed, when it took effect, what category it falls into, who logged it, where the information came from, and how confident the logger is.

**Confidence and weighting interact.** An entry logged as Speculative can be weighted, but it cannot carry a causal claim alone. See the schema for the full rules on how AI uses the confidence field.

**AI never writes to the context object.** Entries are human-authored or human-approved. An agent that can log its own context and then reason over it has closed a loop nobody is watching.

---

## Approval by Example

Leadership cannot meaningfully approve a weighting without an output. Nobody can evaluate whether champion disengagement should be weighted 0.1 or 0.2 in the abstract. Show them how a real past quarter would have been explained under the proposed configuration, and they can tell you whether that explanation matches what they believe actually happened.

**What gets approved:** three things, not one.

- The structural and contextual split
- The structural weights
- The contextual guardrail ranges

Leadership approves all three, because all three shape the answer. A defensible set of structural weights paired with an uncapped market shift category still produces a model that can externalize any uncomfortable finding.

**The process:**

1. **Draft the configuration.** RevOps proposes the split, the structural weights, and the guardrail ranges. Structural weights come from historical patterns and domain knowledge. Guardrails come from a judgment about which categories could be used to avoid an internal cause.

2. **Capture leadership's understanding first.** Before showing any model output, ask the revenue leaders what they believe caused the outcome in two or three past quarters. Write it down. This has to happen before they see the model, or you are testing whether they will agree with a plausible explanation rather than whether the model is right.

3. **Run the model against those quarters.** Generate the causal answer the configuration would have produced, using the context records that existed at the time. If no context records exist for the period, run it structural-only and say so.

4. **Compare.** If the model's explanation of a known quarter doesn't match the shared understanding of that quarter, something is wrong. Either the weights are wrong, or the shared understanding was. Both are worth finding out.

5. **Show the alternatives.** Present the same quarter under two or three different configurations so leadership sees what they are choosing between rather than approving in isolation. Vary one thing at a time.

   - A different split, such as 60/40 against 70/30, to show how much environmental context is allowed to explain
   - A different structural weighting, such as champion disengagement High against Medium for enterprise

6. **Get explicit agreement on all three.** Named sign-off from the revenue leaders whose narratives will run on this model. Record what they approved, not just that they approved.

7. **Re-run at review.** Same process against a recent quarter. As the context object matures, expect the split to be the thing most likely to change.

**Why step 4 is the important one:** a causal model that explains the past incorrectly will explain the present incorrectly. Testing against quarters where the answer is already known is the only validation available before the model is used on a quarter where it isn't.

**Why step 5 matters more than it looks:** approving a configuration in isolation is approving whatever was put in front of you. Approving it against alternatives is a decision. The difference shows up later, when a director asks whether other explanations were considered and there is a real answer.

---

## Guardrails on Contextual Weighting

Contextual weighting exists to test assumptions. It must not become a way to produce a preferred conclusion.

**Bounded ranges.** The administrator sets minimum and maximum weights for each context category. A leader can vary within the range. They cannot zero out a Confirmed pricing change or weight a Speculative rumor as the dominant cause.

**Every run is logged.** Who ran it, what weights they set, when, and what the output was. If a weighting changes the day before a board meeting and it differs materially from the standing configuration, then it is visible (to system admins and leadership) rather than silent.

**Weights are stated in the output.** Any causal answer produced under non-default weighting says so. "Under a weighting that emphasizes competitive displacement over pricing" is part of the answer, not a footnote.

**Default weighting exists and is the baseline.** There is always a standing configuration. Alternatives are explicitly labeled as alternatives.

**Nobody sets weights on their own performance.** A leader exploring why their own function underperformed should not be the only person who touched the weights. Not because of bad faith, but because the incentive is obvious and the appearance matters.

---

## Narrative Consistency Across Teams

Contextual weighting creates a real risk: sales runs one weighting, marketing runs another, and two incompatible stories arrive at the same board meeting.

**Rules:**

- **Structural weights are shared and singular.** There is one model. Teams do not maintain their own.
- **One weighting goes in the board deck.** Exploration is fine and encouraged. Publication is singular.
- **Disagreement is surfaced, not resolved by volume.** If two teams reach different conclusions under different weightings, that is a finding worth reporting, not a conflict to settle offline. "Under weighting A the cause reads as competitive, under weighting B it reads as pricing, and we have not yet determined which is right" is an honest board statement.
- **The variance narrative names its weighting.** If a control book variance explanation was informed by a causal model run, state the weighting used. See `board-control-book/SKILL.md` Section 6.

---

## Building the Causal Model

**Step 1: Confirm the semantic layer exists.** Metric definitions and field documentation. Without them, the model is reasoning about undefined quantities. See `semantic-layer-setup/SKILL.md`.

**Step 2: Inventory your structural causal factors.** What in your systems explains revenue outcomes? Start from your own closed-lost analysis, churn reasons, and win/loss interviews rather than a generic list. The list in Two Kinds of Causal Factors is a starting point. Add what your data actually shows and cut what it does not support.

**Step 3: Inventory your contextual causal factors.** What affects outcomes that your systems will never capture? These become the categories in your context object. See `references/context-object-schema.md`. If a factor you list here turns out to be observable in a system, move it to the structural list. Contextual should only hold what genuinely requires a human to record it.

**Step 4: Set the structural and contextual split.** Start at 70/30 and adjust for the maturity of your context object and the volatility of your market. Document the reasoning. If you have no context object yet, the split is 100/0 until you do.

**Step 5: Segment the structural factors.** The same factor matters differently by segment, deal size, product, and motion. Champion disengagement is close to fatal in a twelve-month enterprise cycle and often survivable in a thirty-day SMB cycle. Document where each factor's weight varies.

**Step 6: Draft structural weights.** Base them on evidence where you have it. Historical loss analysis, cohort comparisons, anything where you can observe the factor's association with outcomes. Where you have no evidence, say so and mark the weight as a starting assumption to be revisited.

**Step 7: Build the context object.** See `references/context-object-schema.md`. Backfill at least the last two quarters so the model has something to reason over. A context object with no history cannot inform an analysis of a period that predates it.

**Step 8: Run approval by example.** The seven-step process above. Do not skip the alternatives step.

**Step 9: Set contextual guardrail ranges.** Minimum and maximum share of the contextual allocation for each category. Document the reasoning behind every cap. See `references/weighting-governance-log.md`.

**Step 10: Configure logging.** Every weighting run captured with who, what weights, when, and what output it produced. A model nobody can audit is a model that produces whatever the person running it wanted.

**Step 11: Define the review cadence.** Structural weights and the split reviewed quarterly at minimum, alongside the semantic layer review. Contextual run log reviewed quarterly for patterns.

---

## What Good Looks Like

- The model explains two or three known past quarters in a way leadership recognizes as accurate
- Structural weights have a named owner, documented reasoning, and a last-approved date
- Every structural weight change is logged with who approved it
- A leader can run an alternative weighting without asking permission, and cannot exceed the guardrails
- Every causal output states which weighting produced it
- Sales, marketing, CS, and finance are reasoning from the same structural model
- When two teams disagree, the disagreement is documented

---

## Common Failure Modes

**Weights set by whoever built the model, approved by nobody.** The most common failure. Leadership inherits a set of assumptions they never examined and then treats the outputs as objective.

**Approval by number.** Showing leadership a table of weights and asking for sign-off produces sign-off without comprehension. Show outputs.

**Structural weights that change every quarter.** If the weight on pricing keeps moving, either the business is changing that fast, which is worth knowing on its own (logged in the context object), or someone is fitting the model to recent results.

**No default weighting.** Without a standing baseline, every run is an alternative and nothing is comparable across quarters.

**Contextual weighting used to reach a conclusion.** Flexibility in contextual weighting should be used as a tool to help discover the truth about the revenue narrative. It should not be used for self-benefiting reasons. 

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
| `references/context-object-schema.md` | Steps 3 and 7 | Field-level schema for the context object, categories, confidence rules, and how AI uses them |
| `references/weighting-governance-log.md` | Steps 9 and 10 | Templates for the structural weight register and the contextual run log |

## Related Skills

- **semantic-layer-setup** — required prerequisite. Provides the metric definitions this model reasons about.
- **why-audit** — evaluates whether a causal answer is trustworthy. Requirement 1 of the scorecard is satisfied by this skill.
- **board-control-book** — Section 6 variance narratives should state which weighting informed them
- **agent-monitoring** — Type 4 detection screens causal outputs for false confidence markers
