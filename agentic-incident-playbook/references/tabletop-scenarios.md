# Tabletop Scenarios

Pre-written incident scenarios for walking the response protocol without a real incident. One per failure type, plus two combined scenarios for teams that want a harder test.

These exist to remove the blank-page problem. "Run a tabletop exercise" gets skipped when step one is inventing a realistic scenario yourself.

**If you have a detection layer built,** these scenarios also work as seeded failures for the combined quarterly exercise in `agent-monitoring/SKILL.md`. Run detection first, then response.

**If you do not have a detection layer yet,** use these to test response on its own. Incomplete, but better than testing neither.

---

## How to Run One

**Who attends:** Everyone who would be involved in a real incident of this type. RevOps, the affected department head, the person who owns the AI tool, and whoever would make the notification calls. If someone can't attend, that absence is itself a finding.

**Time:** 60 to 90 minutes. Longer than that and people stop engaging.

**Format:** Read the scenario aloud. Walk the seven response steps in order. At each step, the person who would own that step in reality says what they would actually do, using the systems in front of them where possible.

**What to time:** How long each step takes. Not the discussion, the actual work. "Pull the field history for every record this agent touched" should be timed by someone actually pulling it.

**What to record:**
- Where people hesitated or disagreed about ownership
- Where the protocol assumed something exists that doesn't (an audit log nobody configured, a permission list nobody wrote)
- How long each step took
- What information was missing that would have been needed

**Do not:**
- Let people describe what they would do in the abstract. Have them do it.
- Skip the notification step because it feels awkward. Drafting the actual message is the point.
- Run it in production. Seeded data in a sandbox only.

**After:** Every gap the exercise surfaced gets an owner and a date. An exercise that produces a list nobody actions is theater.

---

## Scenario 1: Model Drift

**Failure type:** Type 1

**The setup, read aloud:**

> It's the second week of the quarter. Your VP of Sales forwards you a Slack thread from two of her AEs. Both are saying the pipeline scoring model "feels off." One says a deal she considers her strongest opportunity this quarter is scoring 34 out of 100. The other says he's been ignoring the scores for about six weeks.
>
> You pull the numbers. Over the trailing 90 days, deals scored above 70 closed at 31%. Deals scored below 40 closed at 28%. The model is producing scores that no longer separate good deals from bad ones.
>
> The model was last calibrated 14 months ago. Since then, you launched a second product line with a materially lower average deal size, and roughly 40% of new opportunities now come from that product.
>
> The scores feed three things: the forecast weighting in your BI tool, an automated deal-review flag that pulls low-scoring deals into a weekly manager review, and a routing rule that assigns high-scoring inbound to senior reps.

**Complications to introduce mid-exercise:**
- Forty minutes in, someone points out the Q1 board deck used forecast numbers weighted by this model.
- The person who originally built the model left the company eight months ago.

**What this scenario tests:**
- Whether anyone can determine when the model was last calibrated without the person who built it
- Whether the team can identify every downstream system consuming the scores
- How the group handles a wrong number that already reached a board
- Whether "revert to manual scoring" is actually operationally possible

**Expected sticking points:**
- Containment is harder than it sounds, because pausing the model breaks the routing rule
- The board question surfaces late and changes the severity from High to Critical
- Nobody has documentation on the original calibration baseline, so "how far has it drifted" can't be answered precisely

---

## Scenario 2: Unauthorized Action

**Failure type:** Type 2

**The setup, read aloud:**

> At 9:40am a CSM messages you. A customer who churned four months ago, on bad terms after a failed implementation, just received an email offering them 30% off an annual renewal. The customer forwarded it to their former CSM with a one-line reply: "Is this a joke?"
>
> Your outbound sequencing agent was reconfigured last Thursday to enroll leads from a new inbound form. The configuration change widened the enrollment criteria. Instead of pulling only new leads, it is now pulling any contact record with a lifecycle stage that isn't currently "Customer."
>
> Churned customers have lifecycle stage "Former Customer."

**Complications to introduce mid-exercise:**
- Thirty minutes in, reveal that the sequence has been running for four days and includes a three-email cadence.
- Reveal that 61 former customers were enrolled, and 18 of them churned specifically over pricing disputes.

**What this scenario tests:**
- How fast the team can revoke the agent's send permissions
- Whether anyone can produce a list of every contact enrolled since the config change
- Who makes the call on customer notification, and whether it's a mass correction or individual outreach
- Whether the four-day detection lag prompts a detection-layer conversation

**Expected sticking points:**
- Revoking permissions and stopping in-flight sends are two different actions and both need doing
- The notification decision is genuinely hard, and teams often want to skip it
- Nobody documented what the agent was permitted to touch, so "was this out of scope" is a judgment call rather than a lookup

---

## Scenario 3: Cascading Data Error

**Failure type:** Type 3

**The setup, read aloud:**

> A senior AE escalates that a named account she's been working for three months has vanished from her pipeline. No closed-lost record, no notes, just gone from her views.
>
> You trace it. The account's enrichment record was updated 11 days ago and the employer field now shows one of your competitors instead of the actual company. The enrichment provider matched on a similar domain.
>
> The wrong employer value fed your lead scoring model, which applies a hard disqualification rule for competitor domains. The disqualification triggered an automation that removes disqualified contacts from active sequences and sets the associated opportunity to a hidden stage.
>
> You check whether this is isolated. In the same enrichment batch, 340 records were updated. You do not yet know how many were mismatched.

**Complications to introduce mid-exercise:**
- Twenty minutes in, reveal that 23 of the 340 were mismatched to competitor domains.
- Reveal that seven of those had open opportunities, totaling $410K in pipeline.
- Reveal that two of the seven had been in the Q3 commit forecast.

**What this scenario tests:**
- Whether the team can trace a chain backward through three systems
- How they scope the blast radius when the full extent is initially unknown
- Whether the forecast implication gets caught, or only the record-level fix
- Whether anyone asks why a hard disqualification rule runs without a human check

**Expected sticking points:**
- Quarantining records is easy to say and slow to actually do
- Restoring an opportunity to its prior stage may not be possible if stage history wasn't retained
- The forecast correction is the piece most often missed, because attention goes to the data fix

---

## Scenario 4: Confident Wrong Narrative

**Failure type:** Type 4

**The setup, read aloud:**

> Six weeks ago, during Q2 board prep, someone asked your analytics tool why enterprise win rate had dropped. It produced a clear answer: increased competitive pressure, specifically from one named competitor, citing two deals where that competitor appeared in call notes.
>
> That explanation went into the board deck. The board responded by approving $200K in incremental competitive enablement spend, which has since been committed to a battlecard project and a competitive intelligence subscription.
>
> Yesterday, a manager doing her own loss review found that the two cited deals were the only two of eleven enterprise losses where a competitor was mentioned at all. Of the other nine, seven show the same pattern: the deal stalled after the pricing proposal, in every case following the April pricing change that raised enterprise entry pricing by 22%.
>
> The competitive narrative was wrong. The cause was internal pricing.

**Complications to introduce mid-exercise:**
- Twenty-five minutes in, reveal that the next board meeting is in nine days.
- Reveal that the competitive intelligence subscription is a 12-month contract already signed.

**What this scenario tests:**
- Whether the team retracts to everyone who received the original narrative, including the board
- Who owns telling the board that an approved spend was based on a wrong analysis
- Whether the root cause conversation reaches "no why-audit was run" or stops at "the AI was wrong"
- Whether Q3 forecast assumptions built on the competitive theory get revisited

**Expected sticking points:**
- This is the scenario where teams most want to soften the notification. Watch for it.
- Blame lands on the tool rather than the process, which is comfortable and wrong
- The forward correction is bigger than the retraction, because six weeks of decisions rest on the wrong cause

---

## Scenario 5: Stale or Partial Data Reasoning

**Failure type:** Type 5

**The setup, read aloud:**

> Last quarter your CPO asked the analytics agent to identify the top customer pain points across Q2 call transcripts, to inform the H2 roadmap. The output was clean and confident: three themes, ranked, with representative quotes.
>
> That analysis shaped the roadmap. Two engineering teams have been working against it for seven weeks.
>
> This week, a product manager tried to reproduce the analysis with different phrasing and got substantially different themes. Digging in, you find the agent processed 22 transcripts. Your call recording platform holds 3,140 transcripts for that period.
>
> The agent keyword-searched for terms it associated with "pain point," pulled matching fragments until it filled its context window, and analyzed those. Nothing in the output indicated it had processed less than 1% of the available data.

**Complications to introduce mid-exercise:**
- Thirty minutes in, reveal that the 22 transcripts skewed heavily toward one segment, because that segment's reps use more explicit problem language on calls.
- Reveal that a roadmap item was deprioritized based on the original analysis.

**What this scenario tests:**
- Whether the team can determine coverage after the fact
- How they decide whether to redo the analysis or manually validate the existing conclusion
- Whether the fix is technical (coverage reporting) or procedural (require coverage disclosure before use)
- Who notifies product and engineering, and how the seven weeks of work gets addressed

**Expected sticking points:**
- The output looked identical to a complete analysis, so nobody can point to a moment where it should have been caught
- "Redo it properly" may not be technically possible with the same tool, which surfaces a real constraint
- The deprioritized roadmap item is the hardest part, because reversing it is expensive and the original decision felt well-informed

---

## Combined Scenario A: Drift Into Narrative

**Failure types:** 1 and 4

For teams that have run the individual scenarios and want a harder test.

**The setup, read aloud:**

> Your pipeline scoring model has drifted, though nobody has flagged it yet. Deals in the new product line score systematically low because the model was calibrated on the legacy product's deal profile.
>
> Because those deals score low, they get flagged into the weekly at-risk review, and reps deprioritize them. Those deals then close at lower rates, which the model reads as confirmation that its scores were correct.
>
> At quarter end, someone asks the analytics tool why the new product line is underperforming. The tool has access to the scores and to the outcomes. It reports that new-product deals are lower quality, citing the score distribution and the close rates as evidence.
>
> That conclusion goes to the executive team. A proposal is now on the table to reduce sales investment in the new product line.

**What this tests:** whether the team can identify that the evidence is circular, that the model created the outcome it then observed. This is the hardest pattern to catch in a real incident, and the most consequential.

---

## Combined Scenario B: The Quiet Quarter

**Failure types:** any, plus detection failure

For teams with a detection layer.

**The setup, read aloud:**

> Your monitoring register shows every check green for two consecutive quarters. Precision across the suite averages 88%. Alert volume is down 60% from the first quarter after deployment, which the team has treated as a sign the system is stable.
>
> During the quarterly exercise, you seed three failures. One is caught. Two pass through unflagged.
>
> Reviewing the register, you find that four checks have been loosened at least once, each time following a quarter with high alert volume. Each individual retune was reasonable. Nobody looked at the pattern.

**What this tests:** whether the team recognizes that the metric they were optimizing (fewer alerts) was in direct tension with the thing they needed (detection). This is the sensitivity guardrail failure from `agent-monitoring/SKILL.md`, played out over four quarters.

---

## Recording the Exercise

For each scenario run, capture:

| Field | Detail |
|---|---|
| Date | |
| Scenario | |
| Failure type | |
| Attendees | |
| Time to complete each step | Step 1 through Step 7 |
| Gaps surfaced | |
| Systems that did not exist as assumed | |
| Ownership ambiguities | |
| Actions committed, with owner and date | |

**Rule:** the exercise isn't finished when the walkthrough ends. It's finished when every gap it surfaced has an owner and a date. Log the actions somewhere they'll be reviewed, not in meeting notes nobody opens.

---

## Writing Your Own Scenarios

The best scenario is a near-miss your team actually had. If something almost went wrong last quarter, write it up and run it.

**A good scenario has:**
- A specific trigger, usually a person noticing something rather than a system alerting
- Enough detail that the first three steps are obvious and the rest isn't
- At least one complication revealed partway through, so the group can't plan the whole response upfront
- A consequence that already reached someone outside the team, because that's what forces the notification conversation
- Something the team will discover they can't do, since that's the actual output of the exercise

**A bad scenario:**
- Is so severe that everyone agrees on the response immediately
- Has a clean fix, which removes the interesting decisions
- Stops at the technical problem and never reaches the human ones
