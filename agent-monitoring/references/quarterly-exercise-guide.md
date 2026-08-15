# Quarterly Exercise Guide

The run sheet for the combined detection and response exercise. This exercise covers both `agent-monitoring` and `agentic-incident-playbook`, because a monitor that fires into a response nobody can run is not protection.

Run it quarterly. Run it in a test environment. Never in production.

---

## What This Exercise Proves

Four things, in order. Each one is a separate failure point.

1. **Detection fires.** The check catches a seeded failure, and within a defined window.
2. **Escalation routes.** The flag reaches the right person at the right severity through a channel someone actually watches.
3. **Response runs.** The seven-step protocol executes without anyone rereading the document mid-incident.
4. **Gaps surface.** Every hesitation, ownership ambiguity, and missing system gets logged with an owner.

A monitoring layer that has never been tested against a seeded failure is monitoring on paper. Precision looks fine right up until the quarter you needed it.

---

## Before the Exercise

### Two weeks out

- [ ] Pick the failure type to test. Rotate so all five are covered annually.
- [ ] Pick or write the scenario. Use `agentic-incident-playbook/references/tabletop-scenarios.md` or adapt a real near-miss.
- [ ] Confirm the test environment has data recent enough that seeded failures will look realistic
- [ ] Book 90 minutes with everyone who would be involved in a real incident of this type

### One week out

- [ ] Send the invite with the failure type named but not the scenario details. Participants should not prepare a response in advance.
- [ ] Confirm the facilitator, who should not be the person who owns the check being tested
- [ ] Confirm the observer, whose only job is timing and note-taking
- [ ] Pull the current state of the detection checks register, so results can be entered directly

### Day of, before the session

- [ ] Seed the failure into the test environment
- [ ] Note the exact time of seeding, since detection window is measured from this moment
- [ ] Confirm the seeded failure is actually present and would be visible to the check if the check works
- [ ] Do not tell participants what was seeded or when

---

## Roles

**Facilitator.** Reads the scenario, introduces complications on schedule, keeps the group walking the protocol rather than debating it. Should not own the check being tested, since that person needs to participate honestly rather than defend their configuration.

**Observer.** Times each step. Records gaps, hesitations, and ownership ambiguities. Does not participate in the response. This role is easy to skip and the exercise loses most of its value without it.

**Participants.** Everyone who would be involved in a real incident. RevOps, the affected department head, whoever owns the AI tool, and whoever would make notification calls. If someone can't attend, note it. A response protocol that depends on one unavailable person is a finding.

---

## Part 1: Detection (20 minutes)

Run this before anyone sees the scenario. Participants should not know what was seeded.

**What happens:** the group watches whether the check fires on its own.

**Steps:**

1. Note the seeding time from the pre-session checklist
2. Wait for the check's normal run cycle. If the check runs weekly, seed it the week before and evaluate at the session.
3. Record whether it fired, and when

**Record for each check tested:**

| Question | Answer |
|---|---|
| Did the check fire? | Yes / No |
| Time from seeding to fire | |
| Was it within the check's stated detection window? | |
| Did it fire at the correct severity? | |
| Did any check that should not have fired, fire anyway? | |

**If detection failed:** stop and diagnose before moving on. A missed seeded failure is the single most important output of the exercise, and it should not get buried under a good response walkthrough. Ask:

- Was the threshold too loose?
- Was the seeded failure realistic enough to be caught?
- Did the check run at all, or is it silently broken?
- Has this check ever fired on a real failure?

Enter the result in the register under "seeded failures caught," including the misses.

---

## Part 2: Escalation (10 minutes)

**What happens:** the group traces where the flag went and who saw it.

**Questions to answer:**

- Which channel did the alert land in?
- Who is subscribed to that channel?
- Did the named owner of the check see it?
- How long between the check firing and a human noticing?
- If nobody noticed, how long would it have sat there?

**Common finding:** the alert fired correctly into a channel nobody reads. This is not a detection failure and it is not a response failure, which is why it survives so long. It only surfaces in an exercise that traces the path end to end.

**Record:**

| Question | Answer |
|---|---|
| Alert routed to | |
| Named owner saw it? | Yes / No |
| Time from fire to human acknowledgment | |
| Route matches what the register says | Yes / No |

---

## Part 3: Response (45 minutes)

Now read the scenario aloud. Walk the seven steps from `agentic-incident-playbook`.

**Ground rules:**

- Participants do the work, they don't describe it. "I'd pull the field history" becomes "pull the field history now, we'll time it."
- The facilitator introduces complications on schedule, not all at once
- Nobody skips the notification step because it feels awkward. Draft the actual message.

### Step 1: Contain

**Observer times:** how long from scenario start to containment action taken.

**Watch for:**
- Whether anyone tries to diagnose before containing
- Whether containment is technically possible (can you actually pause this agent?)
- Who has the permissions to take the containment action, and whether they're in the room

### Step 2: Assess impact

**Observer times:** how long to produce a list of affected records.

**Watch for:**
- Whether the team can determine blast radius at all
- Whether they check for downstream automated actions or stop at the direct effect
- What information they need and can't get

### Step 3: Notify

**Observer records:** who the group decides to notify, and how long the decision took.

**Watch for:**
- Whether severity is assessed before notification scope
- Hesitation about telling someone uncomfortable, especially a board or a customer
- Whether anyone drafts the actual message or the group agrees to "let leadership know" in the abstract

**Do not let this step be hypothetical.** Have someone write the notification. Read it aloud. It's the step that reveals whether the team can be direct under pressure.

### Step 4: Root cause

**Watch for:**
- Whether the group reaches a systemic cause or stops at the instance
- Whether "the AI was wrong" is treated as a root cause, which it isn't
- Whether the process failure gets named alongside the technical one

### Step 5: Remediate

**Watch for:**
- Whether all three levels get addressed: data, decisions, system
- Whether "correct the decisions" gets skipped, which is the most commonly missed level
- Whether the fix is scoped to the category or just the instance

### Step 6: Test the fix

**Watch for:**
- Whether anyone proposes restoring the agent without testing
- Whether the group can describe a specific test rather than "we'd verify it works"

### Step 7: Document

**Watch for:**
- Whether the incident log gets filled in during the exercise or deferred
- Whether the group knows where the log lives

---

## Part 4: Debrief (15 minutes)

**Three questions, in this order:**

**1. What did we discover we can't do?**

This is the primary output. Every "we'd need to check whether we can actually do that" during the walkthrough belongs on this list.

**2. Where was ownership unclear?**

Any moment where people looked at each other before someone volunteered. Ambiguity that resolves in 15 seconds during an exercise takes an hour during a real incident.

**3. What would have been different at 2am on a Friday?**

The exercise runs with everyone present and attentive. Real incidents don't. Which steps depend on a specific person being available, and what happens when they aren't?

**Do not debrief on whether the group performed well.** Performance is not the output. Gaps are.

---

## Recording Results

### Exercise record

| Field | Detail |
|---|---|
| Date | |
| Failure type tested | |
| Scenario used | |
| Facilitator | |
| Observer | |
| Participants | |
| Absent, and their role | |

### Detection results

| Check ID | Fired? | Time to fire | Correct severity? | Notes |
|---|---|---|---|---|

Enter these in the detection checks register under "seeded failures caught."

### Response timings

| Step | Time taken | Notes |
|---|---|---|
| 1. Contain | | |
| 2. Assess impact | | |
| 3. Notify | | |
| 4. Root cause | | |
| 5. Remediate | | |
| 6. Test the fix | | |
| 7. Document | | |

### Gaps and actions

| Gap | Type (system / ownership / process) | Owner | Due date |
|---|---|---|---|

**Rule:** the exercise is not complete when the session ends. It is complete when every gap has an owner and a date, and those actions are tracked somewhere that gets reviewed. An exercise that produces a list nobody actions is theater with a calendar invite.

---

## Rotation Schedule

Cover all five failure types annually. Suggested rotation:

| Quarter | Failure type | Why this order |
|---|---|---|
| Q1 | Type 2, unauthorized action | Fastest-moving failure type, tests permission controls and containment speed |
| Q2 | Type 1, model drift | Slowest-moving, tests whether detection catches gradual degradation |
| Q3 | Type 4, confident wrong narrative | Tests the hardest notification conversation |
| Q4 | Type 3 or Type 5 | Alternate annually |

**Off-schedule triggers:**

- After any real incident, run a variation of that failure type within 30 days
- After any threshold retune, run the seeded failure test at the new setting before considering the retune complete
- After deploying a new agent, run an exercise against it within one quarter
- After the person who owns a check changes, run an exercise on that check within one quarter

---

## Common Failure Modes in the Exercise Itself

**Running it as a discussion.** If nobody opens a system, it's a meeting about incident response rather than a test of it. The value is in what people discover they can't actually do.

**The check owner facilitating.** They know what was seeded and where the check is weak. Someone else facilitates.

**Skipping the observer role.** Without timing and structured notes, the debrief becomes impressions. "That felt slow" is not a finding. "Step 2 took 34 minutes because field history export requires an admin who wasn't in the room" is.

**Only testing the response half.** Detection is the part that determines whether you find out at all. If you skip Part 1 because you don't have monitoring built, note that as a gap rather than treating the exercise as complete.

**Debriefing on performance.** The instinct is to assess how well the team did. Nobody learns anything from that. Assess what was missing.

**Not logging the actions.** The most common failure. A good exercise surfaces eight to twelve gaps. If they live in a doc nobody reopens, next quarter's exercise surfaces the same eight to twelve.
