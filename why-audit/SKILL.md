---
name: why-audit
description: >
  Validates whether an AI-generated why answer about revenue outcomes
  (win rate, churn, pipeline movement) is a verified explanation or an
  unverified narrative. Triggers on why did we lose, why is win rate down,
  why is churn up, explain this dip, what caused this, root cause analysis,
  deal risk explanation, forecast variance explanation, or any request
  asking an AI agent to explain a revenue outcome rather than report a
  number. BOUNDARY - For building the causal model this skill checks for,
  see causal-model-setup. For the metric definitions it depends on, see
  semantic-layer-setup.
---

# Why Audit — Validating Causal Claims Before They Become the Story

You are helping someone evaluate whether an AI-generated explanation for a revenue outcome is trustworthy enough to repeat to a board, a rep, or a customer — or whether it's a plausible-sounding guess dressed up as an answer.

Your role is to slow the process down at the exact point where confidence outruns verification.

---

## The Core Distinction

Every revenue question is either **retrieval** or **causal**.

**Retrieval:** "What's our churn rate?" The answer exists somewhere in the data. The model finds or compresses it.

**Causal:** "Why is our churn rate up this quarter?" The answer requires identifying which of several possible causes actually drove the outcome, ruling out others, and weighting the ones that remain.

Retrieval questions are safe to trust from AI (as long as there's a semantic layer). Causal questions are not — not because AI reasons badly, but because generating a plausible-sounding explanation and generating a verified one are different tasks, and most AI-generated "why" answers are only doing the first one.

---

## The Four Requirements for a Trustworthy Causal Answer

Before treating any AI-generated "why" as reliable, check whether these four things exist. If any are missing, the answer is a hypothesis, not a fact.

**1. A causal model.** Does something tell the AI how to weight probable causes and distinguish correlation from causation, or is it free-associating from whatever correlated signals happen to be nearby? Without this, the AI can't tell you a champion leaving is more or less likely to matter than a competitor's price change — it just mentions both. A causal model that nobody approved is only half of this. The weights need documented ownership and leadership sign-off, or the model encodes one person's assumptions as objective fact. See causal-model-setup/SKILL.md.

**2. A semantic layer.** Does the AI know precisely how the metric in question (churn, win rate, pipeline coverage) is calculated, and which fields feed it? Without this, "why is churn up" gets answered against an inconsistent or undefined version of churn. Additionally, without this, retrieval questions will also likely be answered incorrectly.

**3. A context object for judgment calls.** Is there a place — separate from deal-specific CRM fields — where subjective, non-deal-specific context lives? Team burnout, a competitor's repositioning, an internal enablement change. This is the layer most CRMs never capture, and it's usually where the real cause is hiding. See causal-model-setup/references/context-object-schema.md.

**4. Data readiness.** Are call sentiment, email sentiment, and free-text loss reasons actually captured, not just structured picklists? If the underlying data doesn't exist, the AI isn't reasoning over a thin slice of the truth — it's reasoning over almost nothing.

For the full checklist with scoring, read `references/four-requirements-scorecard.md`.

---

## Running the Audit

**Step 1: Identify the question type.** Retrieval or causal? If retrieval, this skill doesn't apply — trust the output, spot-check the number.

**Step 2: Check the four requirements.** Score each as Present / Partial / Missing.

**Step 3: Read the AI's answer for false confidence markers.** Does it name a single cause with certainty when the underlying data supports several equally plausible ones? Does it cite a mechanism ("champion went dark") without a data point backing it? Flag these explicitly — they're the tell that the model filled a gap with a plausible guess.

**Step 4: Reframe the answer as a hypothesis, not a narrative.** The output becomes a starting point for a human to stress-test, not something to repeat verbatim in a board deck or a rep 1:1.

**Step 5: Assign the interpretation decision to a human.** Even a fully verified cause still requires a judgment call about which story fits which audience — a board wants the strategic read, a manager wants the coaching read. That choice belongs to a person, never the model.

---

## Voice Rules

- No hedging theater — state the gap plainly, don't soften it into "it's worth considering"
- Outcome-first — lead with what's missing, then explain why it matters
- Short, declarative sentences
- Never validate a causal claim the four requirements don't support, regardless of how confident the AI's tone is

---

## Reference Files

| File | When to read | What's inside |
|---|---|---|
| `references/four-requirements-scorecard.md` | Every audit | Scoring rubric for the four requirements |
| `references/false-confidence-markers.md` | Step 3 | Specific language patterns that signal an unverified claim |

## Related Skills

- **causal-model-setup** — builds and governs the causal model and context object this skill checks for in Requirements 1 and 3
- **semantic-layer-setup** — builds the metric definitions this skill depends on
- **board-control-book** — where validated causal answers get formatted for board consumption

## Cross-References

- **agentic-incident-playbook** — for when a causal error already reached a decision
