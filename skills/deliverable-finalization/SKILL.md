---
name: deliverable-finalization
description: >
  Packages pipeline outputs into stakeholder-ready formats and distributes
  through connected tools. Generates a retrospective summary. The last skill in
  the standard pipeline. Fires on "write this up for the board", "package this
  for distribution", "send this to the team", "create a summary for [person]",
  "finalize this". Also auto-activated after all execution waves are complete
  and stakeholder-review approves.
version: 0.1.0
required_context_fields: [company_name]
recommended_context_fields: [leadership_team, connected_tools]
input: Completed pipeline artifacts (Strategy Brief, Action Plan, execution outputs, validation reports, review summaries)
output: Packaged deliverables + distribution + retrospective
next_skills: [] (terminal skill in the pipeline)
---

# deliverable-finalization

## When to use this skill

**Triggers:**

- "Write this up for the board", "Make this presentation-ready"
- "Package this for distribution", "Send this to the team"
- "Create a summary for [Mike / Claire / the leadership team]"
- "Finalize this", "We're done — wrap it up"
- Auto-triggered after `stakeholder-review` gives a "ready for final packaging" recommendation

**Do NOT trigger:**

| User's ask looks like… | Hand off to |
|---|---|
| "Are we on track?" | `metric-driven-validation` |
| "Any blockers?" | `stakeholder-review` |
| "Execute the plan" | `subagent-driven-execution` |

---

## Load org context first

Invoke `load-org-context` with:

```
required_fields:    [company_name]
recommended_fields: [leadership_team, connected_tools]
```

`connected_tools` is essential here — it determines where deliverables get distributed (Slack, Drive, Monday, email, etc.).

---

## Guided procedure

### Step 1 — Inventory what's been produced

Scan the working folder and conversation history for pipeline artifacts:

- Strategy Briefs (`strategy-brief-*.md`)
- Situation Assessments (`situation-assessment-*.md`)
- Action Plans (`action-plan-*.md`)
- Initiative outputs (`initiative-*.md`)
- Execution summaries (`execution-summary-*.md`)
- Validation reports (`validation-report-*.md`)
- Review summaries (`review-*.md`)

Present the inventory:

> I've found **N artifacts** from this pipeline run:
> [bullet list of files with one-line descriptions]
>
> Who's the audience for the final package, and what format do they need?

### Step 2 — Determine audience and format

Ask the user (or infer from context) who needs to see the output and in what shape.

**Common audience → format patterns:**

| Audience | Likely format | Structure |
|---|---|---|
| CEO / board / investors | Executive summary (1 page) | Objective → key findings → decision → next steps |
| Leadership team | Full brief + action plan | All pipeline artifacts, organized and cross-referenced |
| Department leads | Their slice of the action plan | Filtered to initiatives they own, with relevant context |
| External stakeholders (clients, partners) | Polished summary or deck | Clean, branded, no internal jargon |
| The team (broad) | Slack summary + link to full docs | Short message with key outcomes and links |

**Format options:**

- **Markdown** (default) — saved as `.md` in the working folder. Always available, no dependencies.
- **Slack message** — if Slack is connected, draft and post (with user confirmation before sending).
- **Google Doc / Notion page** — if connected, create and share.
- **Email draft** — if Gmail/Outlook connected, draft (never auto-send).
- **PPTX / DOCX** — only on explicit user request ("make a deck", "write a memo"). These are heavier-weight and should not be the default.

### Step 3 — Package the deliverable

Based on audience and format, create the package. Universal rules:

- **Lead with the answer.** The first paragraph should tell the reader what happened and what they need to know. Details follow.
- **No internal process language.** Don't say "the strategic-discovery skill produced a Strategy Brief." Say "We analyzed the growth opportunity and identified two primary levers."
- **Attribute insights to data, not to Claude.** "Pipeline data shows close rates dropped 15%" not "I found that close rates dropped."
- **Include the "so what."** Every section should connect to what the reader should do with the information.
- **Keep it scannable.** Headers, bullets, tables. No dense paragraphs. Stakeholders skim.

### Step 4 — Distribution

Based on connected tools, offer distribution options:

> The package is ready. Distribution options:
>
> 1. **Save locally** — already done: `deliverable-[name].md` in this folder
> 2. **Post to Slack** — draft a summary for #[channel] (I'll show you before posting)
> 3. **Google Drive** — create a shared doc
> 4. **Email** — draft to [stakeholders] (I'll show you before sending)
>
> Which do you want? (Multiple OK)

**Never auto-send messages or emails.** Always show the user the draft and get explicit confirmation before posting to Slack, sending email, or creating shared docs. This is a hard rule — one bad auto-send erodes trust faster than anything.

### Step 5 — Generate retrospective

After distribution, generate a brief retrospective:

> ## Retrospective: [plan name]
> **Date:** [today]
>
> ### What we produced
> [bullet list of deliverables and where they landed]
>
> ### What went well
> [2–3 bullets from the pipeline: e.g., "the portfolio audit surfaced concentration risk early", "referral benchmarks grounded the incentive structure"]
>
> ### What to improve next time
> [2–3 bullets: e.g., "AM capacity data was missing — have it ready before planning", "competitor differentiation should be enriched before running SWOT"]
>
> ### Open items
> [anything unresolved from the pipeline — blocked initiatives, decisions still pending, metrics that need a future check]

Save as `retrospective-[date]-[plan-slug].md`.

### Step 6 — Your next steps (mandatory human handoff)

**The pipeline produces artifacts. Humans close the loop.** This step exists because dogfood testing revealed that users finish the pipeline with a folder full of .md files and no idea what to do next. The final message of the entire pipeline must be a concrete, human-action checklist.

> ### Your next steps
>
> The pipeline is complete. Here's what needs to happen next — these are human actions, not things I can do:
>
> 1. **Share the action plan with [stakeholder names].** [One of: "I've posted a summary to Slack #[channel]" / "Here's a suggested message to send them" / "The file is at `action-plan-*.md` — share it however works for your team."]
>
> 2. **Get approval on [critical decisions from stakeholder review].** [List each decision with the recommended decider. E.g., "Mike needs to approve the referral program budget ($50K/year)."]
>
> 3. **Assign initiatives in your project tracker.** [If a tracker is connected: "I've created cards on Monday for Wave 1." If not: "The action plan has owners and timelines — create tickets/cards in whatever tracker your team uses."]
>
> 4. **Schedule a check-in for [date].** Come back and say "are we on track?" and I'll run metric-driven-validation against the KPIs defined in the action plan. Suggested cadence: [weekly for fast-moving plans, bi-weekly for quarterly plans, monthly for annual plans].
>
> 5. **[Any initiative-specific handoffs].** E.g., "Initiative 3 produced a referral program one-pager — Claire needs to review it before it goes to clients."

**Rules for the handoff checklist:**

- **Name real people** (from `leadership_team`) not abstractions ("relevant stakeholders")
- **Reference real artifacts** with file names or links, not "the deliverable"
- **Include a follow-up cadence** — when should the user come back for a metric check?
- **Flag anything the pipeline couldn't do** — "I drafted the Slack summary but couldn't post it because Slack isn't connected. Copy-paste from `slack-draft-*.md`."
- **End with a clear signal that the pipeline is done** — "That's the end of this pipeline run. Start a new conversation anytime to check progress, investigate a problem, or plan the next initiative."

---

## Save behavior

- Deliverable: always saved as a file (this is a finalization skill — the whole point is producing artifacts)
- Retrospective: always saved
- Slack/email/doc drafts: saved locally as `.md` copies even when posted to external tools (belt and suspenders)

---

## Quality checks

- [ ] Deliverable is tailored to the stated audience — no "one size fits all" summary
- [ ] Deliverable leads with the answer, not the process
- [ ] No Claude self-references ("I analyzed…") — attribute to data and team
- [ ] No messages or emails were auto-sent — user confirmed before every external action
- [ ] Retrospective exists and includes both "what went well" and "what to improve"
- [ ] All artifacts are saved with consistent naming in the working folder
- [ ] Open items from the pipeline are captured — nothing silently dropped
- [ ] Human handoff checklist was presented as the final message — with real names, real file paths, and a follow-up cadence
- [ ] The pipeline end was clearly signaled — user knows there's nothing left for Claude to do
