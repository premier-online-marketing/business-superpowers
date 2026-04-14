# Worked Example: Quarterly Planning

Shows the full pipeline from discovery to finalization for a realistic quarterly planning session. Company details are fictional.

---

## Setup

- **Company:** Apex Digital (growth-stage agency, 50 employees, SEO + PPC for B2B SaaS companies)
- **User:** Jordan (COO)
- **Connected tools:** Slack, Monday.com, Google Drive
- **BUSINESS_CONTEXT.md:** fully populated from a prior session

---

## The conversation

```
Jordan: "Let's plan Q3 for the growth team."
```

### → strategic-discovery activates

Claude does NOT jump to planning. It runs the Socratic process:

**Phase 1 — Understand the ask:**

> "Q3 planning for the growth team — a few things to sharpen:
>
> 1. What's the growth team's mandate? Are they focused on net-new client acquisition, expansion within existing accounts, or both?
> 2. What happened in Q2 — any Rocks or goals that carried over, or are we starting fresh?
> 3. Is there a company-level Q3 theme from the CEO that the growth team needs to align to?"

Jordan answers: net-new acquisition primarily, Q2 had a missed target on outbound pipeline, CEO wants 20% revenue growth in Q3.

**Phase 1 — Objective restatement:**

> "So the objective is: *Design a Q3 plan for the growth team that delivers 20% revenue growth, focused on net-new client acquisition, with lessons learned from Q2's missed outbound pipeline target.* Right?"

Jordan confirms.

**Phase 2 — Landscape:**

> "Two things to map:
> 1. What's the current pipeline look like — how many qualified leads are in play right now?
> 2. The outbound miss in Q2 — do you already know why it missed, or should we investigate before planning around it?"

Jordan: pipeline has 15 qualified leads, the Q2 miss was because the SDR team was understaffed.

**Phase 3 — Alternatives:**

> "You could attack the 20% growth target via outbound (fix the SDR capacity issue), inbound (lean into content/SEO), expansion (upsell existing accounts), or a mix. Which are you leaning toward?"

Jordan: primarily outbound with inbound as secondary.

**Strategy Brief delivered** in 3 messages. Jordan confirms.

### → situation-analysis activates (auto-triggered)

Claude detects that the strategy references Q2 performance data and competitive positioning.

> "I'll run a quick situation assessment before we plan. Pulling from Monday.com and Slack..."

Subagents pull:
- Q2 pipeline data from Monday (close rates, deal sizes, cycle times)
- Slack #growth-team for recurring themes from Q2
- Drive for the Q2 QBR deck

Synthesizes into a focused SWOT with data-backed evidence (not generic filler).

### → action-planning activates

Breaks the strategy into 5 initiatives across 3 waves:

**Wave 1 (parallel):**
- Initiative 1: Hire 2 SDRs — Owner: Jordan, Timeline: 3 weeks, Metric: 2 SDRs onboarded and ramping
- Initiative 2: Audit inbound funnel conversion rates — Owner: Marketing lead, Timeline: 1 week, Metric: conversion map published

**Wave 2 (depends on Wave 1):**
- Initiative 3: Design SDR outbound playbook — Owner: Sales lead, Timeline: 2 weeks, Metric: playbook tested on 50 prospects
- Initiative 4: Launch content series targeting top 3 ICP pain points — Owner: Marketing lead, Timeline: 4 weeks, Metric: 3 articles published, 500 organic visits

**Wave 3:**
- Initiative 5: Q3 pipeline review + forecast — Owner: Jordan, Timeline: end of month 2, Metric: pipeline covers 2x quarterly target

Jordan confirms. Action Plan saved as `action-plan-q3-growth.md`.

### → stakeholder-review activates

> "Before we execute — one critical decision and one major flag:
>
> **[Critical]** SDR hiring requires budget approval from the CEO. The Q2 miss was caused by understaffing; if this hire gets delayed, the plan stalls.
> - Recommended decider: CEO
> - Impact if delayed: Wave 2 can't start, Q3 target at risk
>
> **[Major]** The inbound initiatives (Initiative 4) depend on the marketing lead's capacity. She's also running a product launch in Q3.
> - Options: (a) deprioritize the content series, (b) bring in a freelance writer, (c) accept the risk
>
> Recommendation: Get CEO approval on hiring before starting Wave 1."

Jordan takes the hiring approval to the CEO. Comes back approved.

### → subagent-driven-execution activates

Dispatches Wave 1 subagents in parallel:
- Subagent 1 (draft mode): writes SDR job descriptions, posts to Monday as tasks
- Subagent 2 (analyze mode): pulls conversion data from CRM, builds funnel visualization

Both pass two-stage review. Jordan approves Wave 1 outputs.

Waves 2 and 3 proceed similarly.

### → metric-driven-validation activates (mid-Q3 check)

> "Week 6 check-in:
> - SDR pipeline: 🟢 GREEN — 40 outbound meetings booked (target: 35)
> - Inbound traffic: 🔴 RED — 280 organic visits (target: 500). Leading indicator: only 2 of 3 articles published (content series behind schedule).
> - Overall pipeline coverage: 🟡 1.4x (target: 2x) — SDR performance is strong but inbound gap is dragging total."

### → deliverable-finalization activates (end of Q3)

Packages everything into:
- Executive summary for the CEO (1-page markdown)
- Full Q3 growth report (with initiative-level results)
- Retrospective (what worked: SDR hire was the right call; what to improve: content capacity planning)

Posts summary to Slack #leadership. Saves all files locally.

---

## What this example demonstrates

1. **Skills auto-trigger based on context** — Jordan never typed a skill name
2. **Progressive disclosure** — every skill presented output in sections with confirmation gates
3. **Data-grounded analysis** — situation-analysis pulled from Monday and Slack, not generic filler
4. **Human decisions at every gate** — CEO budget approval, marketing capacity tradeoff
5. **End-to-end pipeline** — discovery → analysis → planning → review → execution → validation → finalization
6. **Org-agnostic** — no hardcoded methodology, tools, or team structure
