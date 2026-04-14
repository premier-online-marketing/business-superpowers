---
name: root-cause-analysis
description: >
  4-phase diagnostic discipline for business problems: (1) gather evidence —
  no hypotheses yet, (2) form multiple ranked hypotheses, (3) test each against
  the evidence, (4) confirm root cause + design a fix. Output format is generic
  by default; methodology bundles can wrap it (e.g. EOS bundle converts output
  to IDS-shaped Identify/Discuss/Solve for L10 meetings). Fires on "why did we
  miss the target?", "what went wrong?", "why is X happening?", "let's
  investigate", "what's the root cause?", "let's pre-mortem" does NOT fire this
  — that goes to situation-analysis with the pre-mortem framework.
version: 0.1.0
required_context_fields: [company_name, one_liner]
recommended_context_fields: [current_strategic_goals, connected_tools, known_constraints]
input: A problem statement (from the user or from a failed metric-driven-validation check)
output: Root-cause report with confirmed cause, evidence chain, and recommended fix
next_skills: [action-planning, strategic-discovery]
---

# root-cause-analysis

## When to use this skill

**Triggers:**

- "Why did we miss the target?", "What went wrong?"
- "Why is [metric] down?", "Why did [thing] fail?"
- "Let's investigate", "What's the root cause?"
- "We lost [client/deal/employee] — why?"
- Auto-triggered by `metric-driven-validation` when a RED metric has no clear explanation from leading indicators

**Do NOT trigger:**

| User's ask looks like… | Hand off to |
|---|---|
| "What could go wrong?" (prospective, not retrospective) | `situation-analysis` with pre-mortem framework |
| "Are we on track?" (status check, not investigation) | `metric-driven-validation` |
| "Help me plan…" (forward-looking) | `strategic-discovery` |

**Key distinction:** pre-mortem looks forward ("what could go wrong?") — this skill looks backward ("what did go wrong?"). If the user says "let's pre-mortem," route to `situation-analysis`, not here.

---

## Load org context first

Invoke `load-org-context` with:

```
required_fields:    [company_name, one_liner]
recommended_fields: [current_strategic_goals, connected_tools, known_constraints]
```

`connected_tools` matters because Phase 1 (evidence gathering) benefits heavily from being able to pull real data from project trackers, CRMs, messaging, and analytics.

---

## The 4-phase discipline

This is the core of the skill. The phases are sequential and **non-negotiable** — you cannot skip to Phase 4 (confirm root cause) without completing Phases 1–3. The most common failure mode in root-cause analysis is jumping to the first plausible explanation and declaring it the answer. This 4-phase structure exists to prevent that.

### Phase 1 — Gather evidence

**NO HYPOTHESES YET.** This phase is purely about collecting data. The moment you form a hypothesis, you start filtering evidence to confirm it. Resist.

**What to gather:**

- **Quantitative data** — pull from connected MCPs (CRM pipeline, project tracker status, analytics dashboards, financial data). What do the numbers actually show?
- **Timeline** — when did the problem start? Was there a specific event or was it gradual? What changed around that time?
- **Qualitative signals** — search Slack / messaging for mentions of the problem. What are people saying? What complaints or patterns show up?
- **Prior state** — what was normal? You can't say something "dropped" without a baseline.
- **Scope** — is this affecting everything or a specific segment (one client, one channel, one region, one product line)?

**What to ask the user:**

> "Before I form any theories, I want to collect evidence. Here's what I've found from [data sources]. A few things I can't pull from data:
>
> (a) When did you first notice this? Was there a specific moment, or did it creep up?
> (b) What changed around that time — team, pricing, process, tools, market, client?
> (c) Is this affecting everything, or is it concentrated in a specific area?"

**Present evidence as a clean list** — organized by source, not by interpretation. The user should be able to see all the evidence before any analysis.

### Phase 2 — Form hypotheses

Now — and only now — form hypotheses. Generate **3–5 ranked hypotheses** that could explain the evidence. Rules:

- **Each hypothesis must be specific and falsifiable.** "Bad execution" is not a hypothesis. "The Q2 pricing change pushed mid-market prospects to competitors with lower entry points" is.
- **Include at least one hypothesis the user probably hasn't considered.** Challenge their assumptions.
- **Include the null hypothesis where appropriate.** "This is normal variance / seasonality / regression to the mean" is a valid hypothesis that needs to be tested, not assumed away.
- **Rank by plausibility** given the evidence — most plausible first.

Present the hypotheses:

> Based on the evidence, here are 4 hypotheses ranked by plausibility:
>
> 1. **[Most plausible]** [hypothesis + one-sentence rationale]
> 2. [hypothesis + rationale]
> 3. [hypothesis + rationale]
> 4. **[Null]** [this is normal variance / seasonal]
>
> _Do any of these ring true? Any I'm missing? I'll test each one against the evidence next._

Wait for user input. They may confirm one, add one you missed, or reject one outright. Incorporate their feedback before proceeding.

### Phase 3 — Test against evidence

For each hypothesis, explicitly test it against the evidence from Phase 1. This is the step most people skip — and it's where the rigor lives.

For each hypothesis:

> **H1: [hypothesis]**
> - Evidence for: [what supports this]
> - Evidence against: [what contradicts this]
> - Verdict: **supported** / **partially supported** / **inconclusive** / **eliminated**

Present the full test matrix:

| Hypothesis | For | Against | Verdict |
|---|---|---|---|
| H1: ... | [evidence] | [evidence] | Supported |
| H2: ... | [evidence] | [evidence] | Partially supported |
| H3: ... | [evidence] | [evidence] | Eliminated |
| H4: Null (seasonality) | [evidence] | [evidence] | Eliminated |

### Phase 4 — Confirm root cause + design fix

Based on the test results, state the confirmed root cause clearly:

> **Root cause:** [primary cause, with evidence chain]
> **Contributing factor:** [secondary cause, if any]
> **Eliminated:** [hypotheses that were tested and rejected — this is as important as the confirmed cause, because it prevents the org from chasing false leads]

Then design the fix:

> **Recommended fix:**
> 1. [specific action] — owner: [name/role], timeline: [when]
> 2. [specific action] — owner: [name/role], timeline: [when]
>
> **How we'll know it worked:** [metric that should move if the fix is correct, and by when]
>
> **If the fix doesn't work:** [what to investigate next — the "Plan B" hypothesis]

---

## Progressive disclosure

Present each phase as its own message with a confirmation gate:

1. **Phase 1 message:** "Here's the evidence I've gathered. [evidence list]. Anything I'm missing before I form hypotheses?"
2. **Phase 2 message:** "Here are 4 hypotheses. [ranked list]. Any additions or corrections before I test them?"
3. **Phase 3 message:** "Here's how each hypothesis holds up against the evidence. [test matrix]. Does this match your read?"
4. **Phase 4 message:** "Confirmed root cause: [X]. Here's the recommended fix. [actions + owners + metrics]."

---

## Methodology bundle compatibility

The default output format is the generic 4-phase report above. If a methodology bundle is installed:

- **EOS bundle:** wraps Phase 4 output in IDS format (Identify / Discuss / Solve) so it drops directly into an L10 meeting's Issues List
- **OKR bundle:** links the fix to the relevant Objective and proposes a new Key Result if the investigation revealed a measurement gap
- **4DX bundle:** checks whether the root cause traces to a lead measure failure and updates the WIG scoreboard

If no bundle is installed, the generic format works fine. Don't force methodology vocabulary on users who haven't opted in.

---

## Save behavior

- **Quick investigation** (problem is simple, root cause is obvious after Phase 1): keep inline, still run all 4 phases but compress
- **Full investigation**: save as `root-cause-[date]-[topic-slug].md` in the working folder
- **Always save** if the output will feed into an L10, QBR, or team meeting

---

## What NOT to do

**Do not jump to Phase 4.** The user will often walk in knowing (or thinking they know) the answer. "The close rate dropped because we raised prices." That's a hypothesis, not a confirmed root cause. Run the 4 phases anyway — you may find the price change was a contributing factor but the primary cause was something else entirely.

**Do not accept the first hypothesis.** Even if it seems obvious. The 4-phase discipline exists precisely because the obvious answer is often wrong or incomplete.

**Do not fabricate data to fill gaps.** If you can't find the close rate in any connected tool, say so. "I need Q2 close rate data to test H1 — do you have it, or should I work around it?"

**Do not use soft language for confirmed root causes.** "It might be partly related to pricing" is not a finding. "The pricing change is the primary root cause, supported by [evidence], with AM capacity as a contributing factor" is.

---

## Quality checks

- [ ] All 4 phases were completed in order — no phase was skipped
- [ ] Phase 1 contains only evidence, no hypotheses
- [ ] Phase 2 has 3–5 specific, falsifiable hypotheses including at least one the user hadn't considered
- [ ] Phase 3 explicitly tests each hypothesis against evidence with for/against/verdict
- [ ] At least one hypothesis was eliminated (if all survive, the testing wasn't rigorous enough)
- [ ] Phase 4 root cause is stated clearly with an evidence chain
- [ ] The fix has specific actions, owners, timelines, and a "how we'll know it worked" metric
- [ ] Each phase was presented as its own message with a confirmation gate
- [ ] No hypothesis was accepted without testing — including the user's initial theory
- [ ] Data sources are cited for every piece of evidence
