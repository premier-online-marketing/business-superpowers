---
name: action-planning
description: >
  Breaks an approved strategy into numbered initiatives with owner, timeline,
  success metric, dependencies, and risk factors. Every initiative is sized to
  be completable (not multi-month epics) and has a measurable "done" definition.
  Fires on "break this into tasks", "turn this into a plan", "who owns what?",
  "give me initiatives with deadlines". The output is an Action Plan that
  subagent-driven-execution can execute against.
version: 0.1.0
required_context_fields: [company_name, one_liner]
recommended_context_fields: [leadership_team, known_constraints, current_strategic_goals, connected_tools]
input: Strategy Brief (from strategic-discovery) or direct user instruction
output: Action Plan (numbered initiatives with owners, metrics, timelines, dependencies)
next_skills: [subagent-driven-execution, stakeholder-review]
---

# action-planning

## When to use this skill

**Triggers:**

- "Break this into tasks", "Turn this into a plan", "Who owns what?"
- "Give me initiatives with deadlines", "What are the workstreams?"
- "Make this actionable", "I need a project plan"
- Auto-triggered by `strategic-discovery` when the Strategy Brief's recommended next step is "Break this into an action plan"

**Do NOT trigger:**

| User's ask looks like… | Hand off to |
|---|---|
| "Help me think through…" (still fuzzy) | `strategic-discovery` |
| "Go ahead / execute / kick this off" | `subagent-driven-execution` |
| "Why did this fail?" | `root-cause-analysis` |

---

## Load org context first

Invoke `load-org-context` with:

```
required_fields:    [company_name, one_liner]
recommended_fields: [leadership_team, known_constraints,
                     current_strategic_goals, connected_tools]
```

`leadership_team` is important here — you need names to assign owners. If missing, the plan uses role placeholders ("CEO", "COO", "Head of Paid Media") and flags that owners need to be confirmed.

---

## Guided procedure

### Step 1 — Confirm the source

If a Strategy Brief exists in context (from `strategic-discovery` earlier in the session, or saved as `strategy-brief-*.md`), confirm it:

> "I'll build the action plan against the Strategy Brief: *[one-line objective]*. Still the right target?"

If no Strategy Brief exists, ask the user to describe the goal in 1–2 sentences. Don't require them to run `strategic-discovery` first — some users come in knowing exactly what they want to plan.

### Step 2 — Decompose into initiatives

Break the strategy into 3–7 initiatives. Each initiative is a **discrete, completable piece of work** — not a multi-month epic and not a single task.

**Sizing guide:**

| Too big (epic) | Right size (initiative) | Too small (task) |
|---|---|---|
| "Build a referral program" | "Design referral incentive structure and draft program one-pager" | "Write the email template for referral asks" |
| "Improve account management" | "Audit current AM capacity and recommend a staffing model" | "Update Claire's calendar" |
| "Grow revenue" | "Run portfolio coverage audit across top 6 logos" | "Count Willow Bridge properties" |

### Step 3 — Structure each initiative

Every initiative gets these fields:

```
## Initiative N: [name]

**Owner:** [name from leadership_team, or role placeholder]
**Timeline:** [specific: "2 weeks", "by May 15", "Sprint 1"]
**Success metric:** [measurable "done" definition]
**Mode:** [research / draft / coordinate / analyze — from subagent-driven-execution]
**Dependencies:** [list initiative numbers this depends on, or "none"]
**Risk:** [one-line risk + mitigation, or "low"]
```

### Step 4 — Sequence into waves

Group initiatives by dependency relationships:

- **Wave 1:** initiatives with no dependencies (can run in parallel)
- **Wave 2:** initiatives that depend on Wave 1 outputs
- **Wave 3:** initiatives that depend on Wave 2 outputs

Present the wave structure visually:

```
Wave 1 (parallel):  Initiative 1, Initiative 2
         ↓
Wave 2 (parallel):  Initiative 3, Initiative 4
         ↓
Wave 3 (sequential): Initiative 5
```

### Step 5 — Present for approval

Present the full Action Plan with progressive disclosure:

**Message 1 — Overview and wave structure:**

> ## Action Plan: [initiative name]
> **Objective:** [one-line from Strategy Brief]
> **Total initiatives:** N across M waves
> **Timeline:** [first wave start → last wave end]
> **Owners:** [names involved]
>
> [wave structure diagram]
>
> _Does this high-level structure make sense? I'll show each initiative in detail next._

**Message 2 — Initiative details (all of them):**

Present each initiative with the full structure from Step 3. Group by wave.

> _Review the full plan. Anything to adjust — owners, timelines, scope?_

**Message 3 — What happens next (mandatory):**

After the user confirms the plan, explicitly present the next-step options. Do NOT say "want me to execute?" without explaining what that means. Users don't know what the pipeline does next unless you tell them.

> ### What happens next
>
> The action plan is saved. From here, three paths:
>
> 1. **Take this to stakeholders offline first.** Share the action plan with [names from leadership_team] for feedback before anything executes. Come back when you're ready to continue — I'll pick up where we left off.
>
> 2. **Run a stakeholder review now.** I'll scan the plan for critical decisions, blockers, and risks that need human sign-off before execution starts — things like budget approvals, capacity conflicts, or dependency risks. This is the recommended path if multiple people need to greenlight parts of the plan.
>
> 3. **Start execution directly.** I'll dispatch subagents for each initiative, wave by wave. **Before I start, I'll show you exactly which tools and connectors each subagent will use** (Ahrefs, Monday, Slack, etc.) and what kind of operations it'll run, so there are no surprises on cost or access.
>
> _Which path? (or something else)_

This message is **non-negotiable.** Every action plan delivery must end with this choice. The most common failure mode is Claude auto-proceeding to execution when the user expected a pause for offline stakeholder alignment.

### Step 6 — Save the plan

Save as `action-plan-[slugified-name].md` in the working folder. This file is what `subagent-driven-execution` reads from.

---

## Constraints the plan must respect

- **`known_constraints`** from org context — if there's a headcount freeze, don't propose hiring. If there's a budget ceiling, note it.
- **Timeline** from the Strategy Brief — initiatives must fit within the stated timeframe.
- **Owner capacity** — if the leadership team is 2 people, don't create a 7-initiative plan where both are double-booked in Wave 1. Sequence to respect capacity.
- **Methodology bundle** — if `planning_methodology` is set and a matching bundle is installed, adapt output format (e.g. for EOS: each initiative maps to a Rock or a To-Do under a Rock).

---

## Quality checks

- [ ] Every initiative has all 6 fields (owner, timeline, metric, mode, dependencies, risk)
- [ ] Initiatives are right-sized — not epics, not tasks
- [ ] Wave structure respects dependency ordering
- [ ] No initiative violates a stated constraint
- [ ] Owner assignments are realistic given team size and capacity
- [ ] Success metrics are measurable, not vibes ("increase revenue" is bad; "sign 3 enterprise PMCs by June 30" is good)
- [ ] The plan was presented progressively (overview first, details second)
- [ ] Plan was saved as a .md file in the working folder
- [ ] If leadership_team was missing, role placeholders were used and a warning was flagged
