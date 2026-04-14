---
name: stakeholder-review
description: >
  Decision gates between major phases. Summarizes progress against the Action
  Plan, flags blockers and risks by severity (Critical / Major / Minor),
  presents decision points that require human judgment, and blocks forward
  progress on critical issues. Fires on "any blockers?", "ready to ship this?",
  "should we move forward?", "what needs approval before we proceed?", "review
  the status for me". Also auto-activated between execution waves.
version: 0.1.0
required_context_fields: [company_name]
recommended_context_fields: [leadership_team, known_constraints, current_strategic_goals]
input: Current pipeline state (Strategy Brief, Action Plan, execution outputs, validation reports)
output: Review summary with decision points and severity flags
next_skills: [subagent-driven-execution, deliverable-finalization, action-planning]
---

# stakeholder-review

## When to use this skill

**Triggers:**

- "Any blockers before we move forward?", "What needs approval?"
- "Ready to ship this?", "Should we proceed?"
- "Summarize where we are for [stakeholder]"
- "Review the status for me", "What's the state of the plan?"
- Auto-triggered between execution waves and before `deliverable-finalization`

**Do NOT trigger:**

| User's ask looks like… | Hand off to |
|---|---|
| "Are we on track?" (metric check) | `metric-driven-validation` |
| "Why did this fail?" | `root-cause-analysis` |
| "Package this for distribution" | `deliverable-finalization` |

---

## Load org context first

Invoke `load-org-context` with:

```
required_fields:    [company_name]
recommended_fields: [leadership_team, known_constraints, current_strategic_goals]
```

`leadership_team` matters here because the review needs to route decisions to the right person. If missing, use role descriptions ("whoever owns delivery capacity", "whoever approves budget").

---

## Guided procedure

### Step 1 — Gather pipeline state

Collect everything produced so far in this session or saved in the working folder:

- Strategy Brief (`strategy-brief-*.md`)
- Situation Assessment (`situation-assessment-*.md`)
- Action Plan (`action-plan-*.md`)
- Execution outputs (`initiative-*.md`, `execution-summary-*.md`)
- Validation reports (`validation-report-*.md`)

If nothing exists: "There's nothing to review yet — no strategy brief, no action plan, no execution outputs. Want me to help build one of those first?"

### Step 2 — Assess status per initiative

For each initiative in the Action Plan:

| Status | Criteria |
|---|---|
| **Complete** | Success metric met, output reviewed and approved |
| **In progress** | Work started, no blockers, metric not yet measurable |
| **At risk** | Work started but a risk or dependency is threatening the timeline |
| **Blocked** | Cannot proceed without a decision, resource, or dependency resolution |
| **Not started** | In a future wave, not yet kicked off |

### Step 3 — Flag issues by severity

Scan across all initiatives and the broader context for issues:

| Severity | Meaning | Examples | Action |
|---|---|---|---|
| **Critical** | Blocks the entire plan or a key initiative; unresolvable without a human decision | Budget exceeded, key person left, fundamental assumption invalidated, constraint violated | **Block forward progress** until resolved |
| **Major** | Threatens timeline or quality of a specific initiative; workaround may exist | Initiative behind schedule, data unavailable, dependency delayed, output failed quality review | **Warn** — present to user with options |
| **Minor** | Cosmetic or low-impact; can be fixed without blocking | Formatting issues, small scope adjustments, unclear ownership on a non-critical task | **Note** — fix or mention in passing |

### Step 4 — Identify decision points

List any decisions that require human judgment:

> **Decisions needed:**
>
> 1. **[Critical]** The portfolio audit revealed Willow Bridge has 200+ total properties but only 40 are in our target unit-count range. Do we adjust the expansion target, or expand the service offering to smaller units?
>    - **Recommended decider:** Mike (CEO)
>    - **Impact if delayed:** Wave 2 initiatives can't proceed
>
> 2. **[Major]** The referral incentive structure needs budget approval — $500/referral is within norms but adds up to ~$50K/year at target volume.
>    - **Recommended decider:** Mike (CEO)
>    - **Impact if delayed:** Referral program launch slips by 2+ weeks

### Step 5 — Present the review

**Single message** (unlike progressive-disclosure skills, a review should be scannable in one pass — the user needs the full picture to make decisions):

> ## Stakeholder Review: [plan name]
> **Date:** [today]
> **Plan status:** [N/M initiatives complete, X at risk, Y blocked]
>
> ### Status summary
> [table or bullet list of per-initiative status]
>
> ### Issues
> [Critical issues first, then Major, then Minor — each with severity tag, description, and impact]
>
> ### Decisions needed
> [numbered list with recommended decider and delay impact]
>
> ### Recommendation
> [one of: "Proceed to next wave", "Resolve critical issues first", "Revise the action plan", "Ready for final packaging"]

---

## Blocking behavior

**Critical issues block.** If the review surfaces a Critical issue, the skill must explicitly block forward progress:

> "There's a critical issue that needs resolution before we continue: [issue]. I won't proceed to the next wave until this is addressed. Want to discuss it now, or take it offline and come back?"

Do not offer to "work around" critical issues. They exist precisely because the plan can't proceed without a human call.

**Major issues warn but don't block.** Present the issue, offer options, let the user decide whether to proceed or pause:

> "Major issue: [issue]. Options: (a) proceed with the risk accepted, (b) pause and address it, (c) adjust the plan."

---

## Save behavior

- Save as `review-[date]-[plan-slug].md` in the working folder if the review has 3+ issues or decisions
- Keep inline for quick status checks with no blockers

---

## Quality checks

- [ ] Every initiative in the Action Plan has a status (complete / in-progress / at-risk / blocked / not-started)
- [ ] Issues are flagged with explicit severity levels — no "there might be some risk"
- [ ] Critical issues explicitly block forward progress
- [ ] Decision points name the recommended decider (by name or role)
- [ ] Decision points state the impact of delay
- [ ] The review is scannable — one message, not progressive disclosure (this skill is different from the others)
- [ ] The recommendation maps to a concrete next action
