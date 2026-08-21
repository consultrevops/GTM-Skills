# Quarterly Exercise Guide

The run sheet for the combined detection and response exercise. This exercise covers both `agent-monitoring` and `agentic-incident-playbook`, because for a monitor to provide protection, you need to know how to run it. Suggested cadence is a quarterly run.

---

## The Structure

Two halves, tested at different depths.

**Detection: all five failure types, every quarter.** Seed a failure of each type and check whether the corresponding check catches it. This is fast, it costs little once you are already set up, and it is the half that decays quietly. A check that broke should be promptly surfaced.

**Response: one failure type, rotating.** Walk the full seven-step protocol from `agentic-incident-playbook` on one type per quarter. The protocol is largely the same across types.

**If a seeded failure is not caught during the detection round, that type becomes the response walkthrough for the quarter.** The miss tells you where the deeper time is worth spending.

---

## Two Exercise Types

The agent determines how the response half is run.

**Type A: Analysis agents.** Agents that produce outputs a human reads before acting. Scoring models, forecast tools, causal analysis. Covers failure types 1, 3, 4, and 5. The question is capability and correctness. Timing does not matter, because these failures run for days or weeks before anyone notices, and speed of response changes nothing.

**Type B: Action-taking agents.** Agents that do things a customer or prospect can see. Sending email, enrolling sequences, updating records that trigger outbound. Covers failure type 2. The question is still capability, but timing genuinely matters, because every minute the agent runs is another wrong message sent.

Run a response for Type B at least once a year, and any time you deploy a new agent with send permissions.

---

## Transparency Rules

**Everyone involved in testing is aware of the procedures.** What is being tested, how, when it triggers, and what the results were. 

**People affected by the agents are informed of testing.** Anyone whose work depends on an agent being tested should know a test is happening and what the results were. This typically a few messages. Who receives the notifications is at the discretion of whoever runs the test.

**Production testing requires more notice.** If a tool has no sandbox, or the capability you need to test cannot be tested in one, the exercise runs in production. Announce it further in advance and to a wider group.

**Bad results get communicated regardless of scope.** If a test reveals that an agent has been producing wrong output for weeks, everyone affected by that output needs to know, along with how it is being fixed and by when. This happens after the exercise.

---

## Before the Exercise

### One week out

- [ ] Confirm which five checks will be tested, one per failure type
- [ ] Pick the failure type for the response walkthrough. Rotate so all five are covered annually.
- [ ] Determine whether the response walkthrough is Type A or Type B
- [ ] Pick or write the scenario for the response half. Use `agentic-incident-playbook/references/tabletop-scenarios.md` or adapt a real near-miss.
- [ ] Confirm which seeded failures can run in a sandbox. For any that cannot, note that they run in production and widen the notice.
- [ ] Identify who holds each permission the response depends on. Confirm they can attend.
- [ ] Book 60 minutes with the people running the test and planning what is being tested. Nobody else needs to attend.
- [ ] Send the pre-test notification
- [ ] Confirm the facilitator, who should not own the checks being tested
- [ ] Pull the current detection checks register

### Day of

- [ ] Seed all five failures
- [ ] Confirm each is present and would be visible to its check if the check works
- [ ] If the response walkthrough is Type B, note the exact seeding time for that failure. This is the start point for timing.

---

## Notifications

Two messages. Short.

### Pre-test

> **Subject: AI agent test, [date]**
>
> We're running our quarterly test of the agents in the revenue stack on [date]. This checks whether our monitoring catches five categories of failure, and walks the response protocol for [failure type].
>
> [Sandbox / Production. If production: what will be visible, and what will not actually reach anyone outside the company.]
>
> If you see anything unusual from [agent names] on [date], check with [name] before acting on it.
>
> Results will follow within [timeframe].

### Post-test

> **Subject: AI agent test results**
>
> **Detection:** [X of 5 seeded failures caught. Name any that were missed.]
>
> **Response walkthrough:** [failure type tested, and what it surfaced]
>
> **What we found:** [gaps, in plain language]
>
> **What's changing:** [actions, owners, dates]
>
> **Does this affect anything you rely on:** [yes/no. If yes, what, for how long, and how it's being corrected.]

**If the test surfaces a real problem,** meaning an agent has actually been producing wrong output in production, the post-test message goes to everyone affected, not just the people who ran the exercise. Include how long it has been happening, what decisions may have been made on bad output, and the correction timeline.

---

## Roles

**Facilitator.** Runs the detection round, reads the scenario for the response half, introduces complications, keeps the group walking the protocol rather than debating it, and captures gaps. Should not own the checks being tested.

**Participants.** The people running the test and the people who planned it, plus whoever holds a permission the response depends on. For a Type B response walkthrough, this usually includes someone outside operations, since the team whose customers are affected would make the real correction.

---

## Part 1: Detection, All Five Types

Run this before the scenario walkthrough. This is the fast half.

For each seeded failure:

| Failure type | Check ID | Fired? | Within stated window? | Correct severity? | Notes |
|---|---|---|---|---|---|
| 1. Model drift | | | | | |
| 2. Unauthorized action | | | | | |
| 3. Cascading error | | | | | |
| 4. Confident wrong narrative | | | | | |
| 5. Stale or partial data | | | | | |

**Also record:** did any check fire that should not have.

**For every miss, diagnose before moving on:**

- Was the threshold too loose?
- Was the seeded failure realistic enough to be catchable?
- Did the check run at all, or is it silently broken?
- Has this check ever fired on a real failure?

Enter all five results in the detection checks register under "seeded failures caught," including misses.

**If one or more failures were missed,** the response walkthrough shifts to one of the missed types. A check that does not fire and a response nobody has practiced is the combination that turns a small problem into a long one.

---

## Part 2: Escalation

Trace where the flags went. Run this for every check that fired.

| Check ID | Channel routed to | Named owner saw it? | Route matches register? |
|---|---|---|---|
| | | | |

**Common finding:** a check fires correctly into a channel nobody reads. Neither a detection failure nor a response failure, which is why it persists.

---

## Part 3: Response, One Failure Type

Read the scenario aloud. Walk the seven steps from `agentic-incident-playbook`.

**One ground rule:** open the systems rather than describing what you would do. The value is discovering what is not actually possible, and that only surfaces when someone tries.

### Type A: Capability check

| Step | What to establish |
|---|---|
| 1. Contain | Can this agent be paused? By whom? Is their access current? |
| 2. Assess impact | Can the team list affected records? Can they see downstream automated actions, or only the direct effect? |
| 3. Notify | Who gets told, at what severity? Draft the actual message. |
| 4. Root cause | Does the group reach a systemic cause, or stop at the instance? "The AI was wrong" is not a root cause. |
| 5. Remediate | Are all three levels addressed: data, decisions, system? "Correct the decisions" is the most commonly skipped. |
| 6. Test the fix | Can the group describe a specific test, or does it stop at "we'd verify it works"? |
| 7. Document | Does everyone know where the incident log lives? |

### Type B: Capability check plus timing

Same seven steps, with two numbers that matter.

**Number 1: Time from wrong action to someone knowing.** Measured from when the agent took the action to when a human was aware of it. A detection problem. If it is long, the fix is monitoring, not response.

**Number 2: Time from knowing to stopping.** Measured from awareness to the agent being fully stopped, including in-flight actions. A response problem. If it is long, the fix is permissions and access.

Different problems with different owners. Record them separately.

**Additional Type B questions:**

- Can in-flight sends be stopped, or only future ones? Different capabilities, both need testing.
- If the tool has no sandbox, can stopping be tested at all in production without actually sending?
- Who has the permission to stop it, and are they reachable outside working hours?
- If a customer already received something wrong, who makes the correction? Is that person in the room?

---

## Part 4: Debrief

**1. What did we discover we can't do?**

The primary output. Every "we'd need to check whether that's possible" from the walkthrough.

**2. Where was ownership unclear?**

Any moment where people looked at each other before someone volunteered.

**3. Which steps depend on one specific person?**

The exercise runs with everyone present. Real incidents don't.

**4. Did this surface anything real?**

Separate from the seeded failures, did the exercise reveal that an agent has actually been producing wrong output in production? If so, that becomes its own incident and follows `agentic-incident-playbook`, not this guide.

---

## Recording Results

### Exercise record

| Field | Detail |
|---|---|
| Date | |
| Checks tested | All five, list by check ID |
| Response walkthrough type | Failure type, and A or B |
| Scenario used | |
| Environment | Sandbox / Production, note which checks ran where |
| Facilitator | |
| Participants | |
| Pre-test notification sent to | |
| Post-test notification sent to | |

### Detection results

Use the Part 1 table. Enter every result in the detection checks register under "seeded failures caught."

### Capability results

| Step | Possible? | With what tool or permission | Who can do it | Notes |
|---|---|---|---|---|
| 1. Contain | | | | |
| 2. Assess impact | | | | |
| 3. Notify | | | | |
| 4. Root cause | | | | |
| 5. Remediate | | | | |
| 6. Test the fix | | | | |
| 7. Document | | | | |

### Type B timing

| Measure | Time | What it points to |
|---|---|---|
| Wrong action to awareness | | Detection |
| Awareness to fully stopped | | Permissions and access |
| In-flight actions stoppable? | Yes / No / Untestable | Tool capability |

### Gaps and actions

| Gap | Type (system / ownership / process) | Owner | Due date |
|---|---|---|---|

**Rule:** the exercise is complete when every gap has an owner and a date, tracked somewhere that gets reviewed. Otherwise next quarter surfaces the same gaps.

---

## Response Rotation

Detection covers all five types every quarter. Response rotates.

| Quarter | Response walkthrough | Type |
|---|---|---|
| Q1 | Type 2, unauthorized action | B |
| Q2 | Type 1, model drift | A |
| Q3 | Type 4, confident wrong narrative | A |
| Q4 | Type 3 or Type 5, alternate annually | A |

**The rotation yields when detection finds something.** A missed seeded failure takes priority over the scheduled type.

**Off-schedule triggers:**

- After any real incident, run a response walkthrough on that failure type within 30 days
- After any threshold retune, run the seeded failure test at the new setting before considering the retune complete
- After deploying a new agent, add it to the next quarterly detection round. If it has send permissions, make it the Type B response walkthrough within one quarter.
- After the owner of a check changes, include that check in the next detection round

---

## Common Failure Modes in the Exercise Itself

**Running it as a discussion.** If nobody opens a system, it is a meeting about incident response rather than a test of it.

**The check owner facilitating.** They know what was seeded and where the checks are weak.

**Skipping the notifications.** The pre-test and post-test messages are part of the exercise, not admin around it. Skipping them is how a seeded failure gets treated as real, or how a real finding stays inside the team that found it.

**Testing detection on fewer than five types.** The detection round is the cheap half. Cutting it to save time removes the coverage that matters most, since a broken check produces no signal at all.

**Timing a Type A response walkthrough.** If the team running the exercise is the team doing the containment, the stopwatch is self-administered and the number means nothing.

**Debriefing on performance.** Assess what was missing, not how well the team did.

**Not logging the actions.** A useful exercise surfaces several gaps. If they live in a doc nobody reopens, next quarter surfaces the same ones.
