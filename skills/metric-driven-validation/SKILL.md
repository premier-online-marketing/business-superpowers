---
name: metric-driven-validation
description: >
  KPI and leading-indicator checkpoints during and after execution. Business
  validation replaces TDD — define success metrics BEFORE execution, check
  leading indicators at each milestone, flag when metrics diverge from targets.
  Uses a RED-GREEN-REFACTOR metaphor: RED = define target + baseline, GREEN =
  execute until metric hits target, REFACTOR = optimize based on what worked.
  Fires on "are we on track?", "check the KPI", "how's the leading indicator?",
  "did we hit the target?", "what does the data say about our progress?"
version: 0.1.0
required_context_fields: [company_name]
recommended_context_fields: [current_strategic_goals, connected_tools]
input: Metrics from an Action Plan, or a direct user question about progress
output: Validation report (inline or saved .md)
next_skills: [stakeholder-review, root-cause-analysis, action-planning]
---

# metric-driven-validation

## When to use this skill

**Triggers:**

- "Are we on track?", "How are we doing against the plan?"
- "Check the KPI", "What does the data say?"
- "Did we hit the target?", "How's the leading indicator?"
- "It's been 2 weeks — where do we stand?"
- Auto-triggered between execution waves when `subagent-driven-execution` completes a wave

**Do NOT trigger:**

| User's ask looks like… | Hand off to |
|---|---|
| "Why did we miss?" (root cause, not status) | `root-cause-analysis` |
| "Any blockers?" (decision gate, not metric check) | `stakeholder-review` |

---

## Load org context first

Invoke `load-org-context` with:

```
required_fields:    [company_name]
recommended_fields: [current_strategic_goals, connected_tools]
```

---

## The RED-GREEN-REFACTOR metaphor

This is how the skill talks about metric validation in business terms:

**RED — Define the target.** Before execution starts, every initiative should have a measurable success metric with a baseline and a target. "RED" means the target is defined but not yet met. If you're running validation and the metrics were never defined, start here.

**GREEN — Check against the target.** During and after execution, pull actual data and compare to the target. "GREEN" means the metric hit the target. If it hasn't, the initiative stays RED — that's a signal, not a failure.

**REFACTOR — Optimize.** Once an initiative is GREEN, look at what worked and what didn't. Can the approach be replicated? Can the metric be pushed further? Should the target be revised upward?

---

## Guided procedure

### Step 1 — Identify what to validate

Pull the metrics from the Action Plan (if one exists in context) or ask the user:

> "What metric are you checking on? If we have an action plan with defined success metrics, I'll pull from there. Otherwise, tell me the KPI and I'll check it."

### Step 2 — Establish baseline and target

For each metric being validated:

| Field | Example |
|---|---|
| **Metric name** | Net-new clients per month |
| **Baseline** (before execution) | 15/month |
| **Target** | 30/month |
| **Leading indicator** | Referral program invitations sent per week |
| **Data source** | Monday pipeline board / CRM / manual tracking |
| **Check frequency** | Bi-weekly |

If the baseline or target is missing, flag it: "This metric doesn't have a defined baseline — I can't tell if we're improving without one. Can you give me the starting number?"

### Step 3 — Pull current data

If `connected_tools` includes a relevant data source, pull current metric values. If not, ask the user for current numbers.

### Step 4 — Report status

For each metric, report using the RED-GREEN-REFACTOR frame:

> ### Net-new clients per month
> **Status:** 🔴 RED — below target
> **Baseline:** 15/month · **Current:** 19/month · **Target:** 30/month
> **Progress:** 27% of the way (4 of 15 incremental units)
> **Leading indicator:** Referral invitations at 8/week (target was 20/week) — this is likely why the lagging metric is behind
>
> **Reading:** The referral program is underperforming on activation, which is dragging the lagging metric. The organic baseline hasn't degraded (good), but the new channel isn't contributing enough yet.

### Step 5 — Recommend action

Based on the validation results:

| Status | Recommended action |
|---|---|
| All GREEN | "Metrics are on track. Proceed to next wave or run REFACTOR to optimize." → `stakeholder-review` or `action-planning` (for REFACTOR) |
| Some RED, leading indicators explain it | "Leading indicators point to a specific bottleneck. Want me to investigate?" → `root-cause-analysis` |
| Some RED, no clear explanation | "Lagging metric is off-target but the leading indicators look fine — something else may be happening. Want me to dig in?" → `root-cause-analysis` |
| No metrics defined yet | "There are no metrics defined for this initiative. Want me to help define them?" → return to `action-planning` |

---

## Save behavior

- **Quick check** (1–2 metrics, inline): keep in conversation
- **Full validation report** (3+ metrics or cross-initiative): save as `validation-report-[date]-[topic-slug].md`
- **Trend data**: if this is a repeat check, reference prior validation reports to show trajectory

---

## Quality checks

- [ ] Every validated metric has a baseline, current value, and target — no "we're doing well" without numbers
- [ ] Leading indicators are connected to lagging metrics — the report explains the causal chain
- [ ] Data sources are cited (which MCP, which board, or "user-reported")
- [ ] Status labels (RED/GREEN) are earned from data, not vibes
- [ ] The recommendation maps to a real next skill
- [ ] If metrics were never defined, the skill flagged it and offered to define them
