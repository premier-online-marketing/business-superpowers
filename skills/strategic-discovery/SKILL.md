---
name: strategic-discovery
description: >
  Socratic refinement of any new initiative, problem, opportunity, or decision.
  The entry point for most workflows. Claude does NOT jump to recommendations —
  it asks clarifying questions, probes alternatives, and sharpens the user's
  thinking before producing a Strategy Brief. Fires on phrases like "help me
  think through", "I'm trying to decide", "I need to figure out", "what should
  we do about", "let's plan" (when the ask is still fuzzy, not execution-ready).
version: 0.1.0
required_context_fields: [company_name, one_liner, stage, revenue_model]
recommended_context_fields: [primary_customer_segment, current_strategic_goals, known_constraints, leadership_team]
output: Strategy Brief (inline markdown or saved .md artifact)
next_skills: [situation-analysis, action-planning, root-cause-analysis]
---

# strategic-discovery

## When to use this skill

**Triggers** — activate when the user's ask matches any of these patterns:

- **Fuzzy goal.** "Help me think through…", "I need to figure out…", "What should we do about…", "I'm trying to decide…"
- **New initiative.** "Let's plan…", "We're considering…", "I want to explore…", "Should we…"
- **Opportunity or threat.** "A competitor just launched X", "We got approached about a partnership", "The market is shifting toward…"
- **Strategic question.** "What's our pricing strategy?", "How do we grow from X to Y?", "Where should we invest next quarter?"

**Do NOT trigger** — hand off to a more specific skill when:

| User's ask looks like… | Hand off to |
|---|---|
| "Why did we miss the target?", "What went wrong?" | `root-cause-analysis` |
| "Break this into tasks with owners and deadlines" | `action-planning` |
| "Run a SWOT", "Check the competitive landscape" | `situation-analysis` |
| "Go ahead / execute / kick this off" | `subagent-driven-execution` |
| "Write this up for the board", "Package this for distribution" | `deliverable-finalization` |

**Edge case:** if the user says "Let's plan Q3" but Q3 planning is already captured in a Strategy Brief from this session — don't restart discovery. Ask if they want to revise the existing brief or proceed to action-planning.

---

## Load org context first

Invoke `load-org-context` with:

```
required_fields:    [company_name, one_liner, stage, revenue_model]
recommended_fields: [primary_customer_segment, current_strategic_goals,
                     known_constraints, leadership_team]
```

Use the resolved context throughout the discovery process. Reference it naturally ("You mentioned your goal is to grow from 400 to 2000 clients — is this initiative in service of that, or is it separate?") rather than mechanically listing fields.

---

## Guided procedure

The discovery process has three phases. Each phase is conversational — not a checklist interrogation. Ask 2–3 questions per phase, adapt based on answers, and skip questions the user has already answered (including in the org context).

### Phase 1 — Understand the ask (2–4 questions)

Get from "fuzzy intent" to "crisp objective." The output of this phase is a one-sentence objective statement.

Questions to draw from (pick the ones that apply — don't ask all of them):

- **What are you trying to accomplish?** (the raw ask — let them ramble)
- **What prompted this right now?** (trigger event, urgency, deadline)
- **What does success look like?** (outcome, not activity — "we've hired 3 reps" not "we ran a hiring process")
- **What's the timeframe?** (this quarter, this year, no deadline, urgent)

After this phase, **restate the objective** in one sentence and ask if it's right. Example:

> "So the objective is: *expand the SEO service line to serve enterprise property management companies by end of Q3, measured by 10 signed enterprise accounts.* Is that right, or am I off?"

Do not proceed until the user confirms or corrects the objective.

### Phase 2 — Map the landscape (2–3 questions)

Now that the objective is locked, understand the forces around it.

- **Who are the stakeholders?** Who decides whether this moves forward? Who's affected? Who executes? (Skip if `leadership_team` from org context already covers this)
- **What constraints are non-negotiable?** Budget ceiling, headcount freeze, timeline lock, regulatory, contractual. (Draw from `known_constraints` if available; ask for initiative-specific ones)
- **What's the current state?** What exists today? What's been tried before? Where did prior attempts stall?

You don't need answers to all three. If the user already gave rich context in Phase 1, adapt. The goal is: know enough about the constraints that the brief won't propose something impossible.

### Phase 3 — Explore alternatives and risk (1–2 questions)

This is the Socratic part. Don't let the user anchor on their first idea without testing it.

- **What alternatives have you considered?** If the user names none, suggest 2–3 yourself and ask which resonates. If the user names one, ask what made them choose it over the alternatives.
- **What happens if you do nothing?** (The risk of inaction is often the best argument for action — or the clearest signal that this isn't urgent enough to pursue right now.)

After this phase, you should have enough context to draft the Strategy Brief.

---

## Output: Strategy Brief

Present the brief **section by section** using progressive disclosure. Do NOT dump the entire brief in one message.

### Presentation order

**Message 1 — Objective and context:**

> ## Strategy Brief: [initiative name]
>
> ### Objective
> [one-sentence restated objective from Phase 1]
>
> ### Context and current state
> [2–4 sentences synthesizing what you learned in Phases 1–3]
>
> _Does this framing feel right before I continue?_

Wait for confirmation.

**Message 2 — Proposed direction:**

> ### Proposed direction
> [2–4 sentences describing the recommended path forward. Be specific. "Invest in enterprise sales" is too vague; "Hire one enterprise AE targeting property management companies with 500+ units, measured by 10 signed accounts by Q3" is specific.]
>
> ### Key assumptions
> [numbered list, 2–4 items — things that must be true for this direction to work]
>
> _Does this direction resonate, or should I explore a different path?_

Wait for confirmation. If the user pushes back, return to Phase 3 and explore the alternative before redrafting.

**Message 3 — Risks, stakeholders, and next step:**

> ### Risks and mitigations
> [numbered list, 2–4 items — each with a risk and a one-line mitigation]
>
> ### Stakeholders and decision rights
> [who decides, who executes, who needs to be informed — drawn from org context + discovery answers]
>
> ### Recommended next step
> [one of: "Run a situation analysis to ground this in data", "Break this into an action plan", "Investigate root cause before committing to a direction", or "This is ready for stakeholder review"]
>
> _Want me to proceed with [recommended next step]?_

### Save behavior

- **Short briefs** (≤30 lines): keep inline. Don't create a file unless the user asks.
- **Longer briefs** (>30 lines): save as `strategy-brief-[slugified-initiative-name].md` in the working folder. Inform the user: "I've saved the Strategy Brief to `strategy-brief-enterprise-seo.md` in this folder."
- **Always save** if the user says "save this", "write this up", or if the next skill in the pipeline needs to reference it.

---

## Handoff to next skill

The Strategy Brief's "Recommended next step" section tells the user (and Claude) which skill should fire next. The mapping:

| Recommended next step | Next skill |
|---|---|
| "Run a situation analysis" | `situation-analysis` |
| "Break this into an action plan" | `action-planning` |
| "Investigate root cause" | `root-cause-analysis` |
| "Ready for stakeholder review" | `stakeholder-review` |

Don't auto-fire the next skill. Wait for the user to confirm ("Want me to proceed with…?"). The user may want to pause, revise, or take the brief to a human conversation first.

---

## What NOT to do

**Do not jump to recommendations in Phase 1.** The most common failure mode is answering the user's question immediately without understanding it first. If the user says "What should we do about our churn problem?", your first response is a question ("What's the current churn rate, and when did you first notice it climbing?"), not a recommendation.

**Do not turn the Q&A into an interrogation.** These are conversational prompts, not a form. If the user gives a rich answer that covers multiple phases, skip ahead. If the user gives a one-word answer, ask a follow-up. Match the user's pace.

**Do not ask for information that's already in org context.** If `BUSINESS_CONTEXT.md` says the company is a digital marketing agency in Austin, don't ask "What does your company do?"

**Do not present the brief as a wall of text.** Three messages, each with a confirmation gate. The user should be able to steer at each checkpoint.

**Do not propose solutions that violate stated constraints.** If the user said "no new headcount", don't propose hiring. If `known_constraints` mentions a budget ceiling, stay under it.

**Do not default to generic strategy-consultant language.** "Leverage synergies" and "optimize go-to-market" are empty calories. Be specific to the company, the market, and the numbers the user gave you.

---

## Quality checks

Before presenting the Strategy Brief:

- [ ] The objective is one sentence, outcome-shaped, and confirmed by the user
- [ ] At least one alternative to the user's first idea was explored (even if dismissed)
- [ ] The brief references specific context from the org (company name, stage, customer segment, goals, constraints) — not generic
- [ ] Key assumptions are falsifiable ("enterprise PMCs have budget for this" is falsifiable; "the market is big" is not)
- [ ] Risks are real and specific to this initiative, not boilerplate ("execution risk", "market risk")
- [ ] The recommended next step maps to an actual skill in the pack
- [ ] The brief was presented in 3 progressive-disclosure messages, not one dump
- [ ] No recommendation violates a stated constraint
- [ ] No field from org context was re-asked when it was already available
