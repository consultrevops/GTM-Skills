# False Confidence Markers

This document lists specific language patterns and structural tells that signal an AI-generated causal answer is stating an unverified claim with the same confidence it would use for a verified one. Use this during Step 3 of the Why Audit.

The core problem: AI doesn't lower its confidence when it's less certain. It generates fluent, well-structured text regardless of whether the underlying claim is supported by data, inferred from correlation, or fabricated to fill a gap. These markers help you catch the difference.

---

## How to Use This File

Read the AI-generated answer slowly. For each claim that explains why something happened (not what happened — that's retrieval), check it against the markers below. If you find two or more markers in a single answer, treat the entire explanation as a hypothesis requiring manual verification before it reaches a board deck, forecast narrative, or coaching conversation.

---

## Category 1: Single-Cause Certainty

The AI names one cause as if it fully explains the outcome, when the underlying data likely supports several equally plausible ones.

**What it sounds like:**
- "The primary driver of the decline was..."
- "This was caused by..."
- "The main reason for the drop is..."
- "The root cause is..."

**Why it's a marker:** Revenue outcomes are almost never single-cause. A dropped win rate is usually some mix of deal-mix shift, rep performance variance, competitive pressure, and pricing dynamics all at once. When the AI picks one and presents it as the answer, it's making a weighting decision it was never equipped to make — unless a causal model explicitly told it how to rank these factors.

**What to do:** Ask: does a causal model exist that told the AI to weight this factor above the others? If not, this is the AI's best guess at a narrative, not a validated root cause.

---

## Category 2: Mechanism Without Evidence

The AI describes a specific causal mechanism ("the champion went dark," "reps lost confidence in the pricing") without citing a data point that confirms it happened.

**What it sounds like:**
- "As the champion became less engaged, the deal lost momentum..."
- "Reps appear to have struggled with the new pricing structure..."
- "Buyer sentiment shifted after the competitive announcement..."

**Why it's a marker:** These read as insider knowledge. They sound like someone who was close to the deals and watched it happen. But if the AI is generating this from CRM stage data and activity logs rather than from a transcript where the champion literally said "I'm stepping back" or a call where a rep said "I don't know how to position this price," then the mechanism is inferred, not observed. The AI is filling a narrative gap with a plausible story.

**What to do:** For each stated mechanism, ask: is there a specific data point (a transcript moment, an email, a logged note) that confirms this happened, or did the AI construct it from correlated signals? If you can't point to the source, the mechanism is a hypothesis.

---

## Category 3: Confident Tone on Thin Data

The AI answers with full confidence even when it could only have accessed a small fraction of the relevant data — often due to context window limits, missing integrations, or incomplete CRM fields.

**What it sounds like:**
- The answer reads identically whether the AI processed 5 transcripts or 500
- No disclaimer, caveat, or confidence qualifier anywhere in the response
- The structure and tone match what you'd expect from a human analyst who reviewed everything personally

**Why it's a marker:** An AI agent that hit a context window limit (e.g., keyword-sampled 15 of 5,000 transcripts to fit a 1M token budget) produces output with the same fluency and confidence as one that read everything. There is no built-in signal that says "I answered this based on 0.2% of your data." The absence of uncertainty language is itself the tell — a human analyst working from partial data would caveat their answer. An AI almost never does.

**What to do:** Ask: how much of the relevant data could the AI have actually processed? If the answer spans a time period or deal volume that exceeds what fits in a single context window, assume the AI sampled rather than read comprehensively, and treat the output accordingly.

---

## Category 4: Recency Bias Framing

The AI disproportionately weights recent events over structural factors, because recent data is more available in the context window or more prominently logged in the CRM.

**What it sounds like:**
- "The Q3 decline was driven by the pricing change announced in August..."
- "Following the departure of [rep name], pipeline velocity dropped..."
- "Since the competitor launched their free tier in July..."

**Why it's a marker:** The most recent, most salient event gets treated as the cause even when a slower-moving structural factor (gradual ICP drift, accumulating tech debt in the sales process, a multi-quarter decline in rep ramp quality) is equally or more plausible. The AI doesn't naturally weight "slow structural decay" against "dramatic recent event" — it pattern-matches on whatever is most prominently represented in the data it accessed.

**What to do:** Ask: is there a slower-moving factor that started before this recent event and could explain part or all of the same outcome? If yes, the recent event might be a trigger on top of an existing structural problem, not the root cause itself.

---

## Category 5: Borrowed Authority

The AI references a metric, a benchmark, or a framework to make its claim sound externally validated, when the reference is either generic (not specific to your org) or unverifiable.

**What it sounds like:**
- "Industry benchmarks suggest that a win rate below X% indicates..."
- "Best practices recommend..."
- "According to standard SaaS metrics..."
- "This pattern is consistent with what high-performing orgs typically see..."

**Why it's a marker:** The AI is using external-sounding authority to bolster a claim that should be evaluated against your own data, not against a generic benchmark. "Industry benchmarks" in an AI-generated answer usually means "something plausible-sounding from the training data," not a specific, dated, sourced study. And even a real benchmark doesn't explain why your specific pipeline moved — it just tells you where you sit relative to an average that may not apply to your segment, stage, or motion.

**What to do:** If the AI cites a benchmark or external reference, ask: is this a specific, named, dated source I can verify? If not, ignore it entirely and evaluate the causal claim on its own merits against your own data.

---

## Category 6: Narrative Coherence as a Substitute for Evidence

The AI tells a clean, well-structured story where each point flows logically into the next — but the logical flow is doing the work that evidence should be doing.

**What it sounds like:**
- "First, the champion departed. This led to reduced internal advocacy, which in turn caused the deal to stall at the procurement stage, ultimately resulting in the loss."
- A clean chain of cause → effect → consequence where each step sounds reasonable but none are independently confirmed

**Why it's a marker:** This is the most dangerous marker because it's the hardest to catch. A well-told story feels true. The human brain treats narrative coherence as evidence — if the story makes sense, it must be what happened. AI exploits this instinct perfectly, because generating coherent narrative structure is exactly what language models are optimized to do. The story isn't evidence. It's a construction that happens to fit.

**What to do:** Break the chain. Take each link separately and ask: is there a data point confirming this specific step happened, independent of the step before it? If the answer is "it makes sense that this would follow" rather than "here's the evidence that it did," the narrative is doing the work that data should be doing.

---

## Quick Reference: Marker Checklist

| # | Marker | Core question |
|---|---|---|
| 1 | Single-cause certainty | Did a causal model tell the AI to weight this factor first? |
| 2 | Mechanism without evidence | Is there a specific data point confirming this mechanism? |
| 3 | Confident tone on thin data | How much of the relevant data could the AI have actually seen? |
| 4 | Recency bias | Is there a slower structural factor underneath the recent event? |
| 5 | Borrowed authority | Is the cited benchmark a real, named, verifiable source? |
| 6 | Narrative coherence as evidence | Does each link in the chain have independent confirmation? |

**Two or more markers in a single answer:** treat the entire explanation as unverified. Do not use it in board materials, forecast narratives, or rep coaching without manual reconstruction.
