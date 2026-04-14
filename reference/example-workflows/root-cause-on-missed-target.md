# Worked Example: Root-Cause Analysis on a Missed Target

Shows the root-cause-analysis skill's 4-phase discipline applied to a real business problem. Company details are fictional.

---

## Setup

- **Company:** Riverstone Partners (growth-stage B2B SaaS, 30 employees, project management tool for construction firms)
- **User:** Alex (CEO)
- **Connected tools:** Salesforce (CRM), Slack, Google Sheets
- **BUSINESS_CONTEXT.md:** populated; current_strategic_goals includes "Reach $3M ARR by year-end"

---

## The conversation

```
Alex: "New customer acquisition is down 30% this quarter. Why?"
```

### → root-cause-analysis activates (not strategic-discovery)

Claude recognizes "why is X down?" as a retrospective investigation, not a forward-looking planning question.

**Phase 1 — Gather evidence (no hypotheses yet):**

> "Before I speculate, let me collect evidence. Pulling from Salesforce and Slack..."

Subagents pull:
- 12 months of pipeline data from Salesforce (deals by stage, close rates by source, deal sizes, cycle times)
- Win/loss notes from Salesforce on last 20 closed-lost deals
- Slack #sales for mentions of lost deals, competitor sightings, pricing objections

Evidence presented as a clean list:

> **Quantitative:**
> - New deals closed: 18 this quarter vs. 26 last quarter (31% drop)
> - Pipeline volume: actually UP 15% — more leads entering top of funnel
> - Close rate: dropped from 34% to 22%
> - Average deal size: unchanged ($18K ARR)
> - Cycle time: increased from 45 days to 62 days
> - By source: inbound close rate 28% → 18%; outbound 40% → 25%
>
> **Qualitative (from Slack + CRM notes):**
> - 6 of 20 closed-lost deals mention "went with BuildFlow" (competitor launched 3 months ago)
> - 4 mention "budget frozen" or "pushed to next year"
> - Sales team mentioned "demo-to-proposal conversion seems off" in 3 threads
>
> **Timeline:** The drop started in month 2 of this quarter. Month 1 was normal.
>
> "Anything I'm missing before I form hypotheses?"

Alex adds: "We also raised prices 15% at the start of the quarter."

### Phase 2 — Form hypotheses:

> Based on the evidence, 4 hypotheses:
>
> 1. **New competitor impact.** BuildFlow launched 3 months ago and is winning deals on price. 6 of 20 losses cited them directly.
> 2. **Price increase backfire.** The 15% price increase pushed borderline deals out of budget. Close rate dropped most sharply on inbound (price-sensitive) leads.
> 3. **Sales capacity constraint.** Longer cycle times (45 → 62 days) could mean the sales team is stretched, leading to slower follow-ups and lost momentum.
> 4. **Macro / seasonal.** Budget freezes in construction are common in Q4. 4 of 20 losses cited budget as the reason.
>
> _Any I'm missing? Which ring true?_

Alex: "I think it's the pricing. But the BuildFlow thing worries me too."

### Phase 3 — Test against evidence:

> | Hypothesis | For | Against | Verdict |
> |---|---|---|---|
> | H1: BuildFlow competition | 6/20 losses cite them; they're priced 20% below us | Close rate dropped on ALL sources, not just competitive deals; only 30% of losses mention them | **Partially supported** — contributing factor but not primary |
> | H2: Price increase | Close rate dropped from 34% → 22% starting the same month as the increase; inbound (price-sensitive) hit hardest; 4 losses cite budget | Pipeline volume is UP (leads aren't being scared off at top of funnel, they're dying mid-funnel); outbound close rate also dropped despite being less price-sensitive | **Supported** — primary cause, with strongest evidence chain |
> | H3: Sales capacity | Cycle time increased 45 → 62 days | Team size unchanged, pipeline volume only up 15%, and longer cycles started AFTER close rate dropped (consequence, not cause) | **Eliminated** — longer cycles are a symptom of harder sells, not the root cause |
> | H4: Seasonal/macro | 4 losses cite budget freezes; Q4 in construction is historically slower | Prior Q4s didn't show this magnitude of drop; the budget-freeze losses are across industries (not construction-specific) | **Eliminated** — explains ~4 of 20 losses but not the systemic pattern |
>
> _Does this match your read?_

Alex: "Yeah, the pricing is clearly the driver. But I'm surprised the BuildFlow thing isn't bigger."

### Phase 4 — Confirm root cause + design fix:

> **Root cause:** The 15% price increase is the primary driver. It didn't reduce top-of-funnel demand (pipeline is up), but it's killing mid-funnel conversion — prospects who would have closed at the old price are now stalling, shopping alternatives (including BuildFlow), or citing budget.
>
> **Contributing factor:** BuildFlow's entry gave price-sensitive prospects a credible alternative they didn't have before. The price increase and the new competitor are compounding — neither alone explains the full 31% drop, but together they account for it.
>
> **Eliminated:** Sales capacity (symptom, not cause) and seasonality (doesn't match the magnitude or timing).
>
> **Recommended fix:**
> 1. **Introduce a lower entry tier** that preserves the price increase for the full product but gives mid-market prospects a path in at closer to the old price point. — Owner: Alex + Product lead, Timeline: 2 weeks to define, 4 weeks to launch
> 2. **Build a competitive battle card for BuildFlow** so the sales team can handle the comparison confidently rather than losing deals silently. — Owner: Sales lead, Timeline: 1 week
> 3. **Track close rate by cohort** (pre-increase vs. post-increase prospects) for the next 60 days to confirm the fix is working. — Owner: RevOps, Timeline: ongoing
>
> **How we'll know it worked:** Close rate recovers to 28%+ within 60 days of launching the entry tier (vs. current 22%).
>
> **If the fix doesn't work:** Revisit H1 more deeply — BuildFlow's impact may be larger than the current data shows, especially if they're winning deals that never enter our pipeline at all (invisible losses).

---

## What this example demonstrates

1. **Phase discipline held** — Claude did not accept Alex's initial theory ("I think it's the pricing") at face value. It collected evidence first, formed multiple hypotheses, and tested all of them before confirming pricing was indeed the primary cause.
2. **A hypothesis was eliminated with evidence** — sales capacity looked plausible but the evidence showed it was a symptom, not a cause. That's the kind of finding that changes what the org invests in fixing.
3. **Data from connected tools grounded every claim** — Salesforce pipeline data, CRM win/loss notes, Slack threads. Not vibes.
4. **The fix is specific and measurable** — not "look into pricing" but "launch an entry tier in 4 weeks, track close rate by cohort for 60 days, target 28%+"
5. **The "if the fix doesn't work" contingency** prevents the org from going all-in on one theory — it acknowledges that the competitive pressure could be larger than current data shows.
6. **Org-agnostic** — no methodology vocabulary was imposed. An EOS org could wrap Phase 4 in IDS format; an OKR org could link the fix to a Key Result. The generic output works for any org.
