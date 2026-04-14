---
name: subagent-driven-execution
description: >
  The execution engine for the pipeline. Takes an approved Action Plan (numbered
  initiatives with owners, timelines, and success metrics) and dispatches a fresh
  subagent for each initiative. Each subagent gets the initiative spec, relevant
  org context, and access to whatever MCPs are connected. Every subagent output
  goes through a two-stage review: spec compliance (does it match the Action Plan?)
  and quality check (is it accurate, well-structured, stakeholder-ready?). Fires
  on "go ahead", "execute the plan", "kick this off", "start working on this".
version: 0.1.0
required_context_fields: [company_name, one_liner]
recommended_context_fields: [connected_tools, leadership_team, known_constraints]
input: Action Plan (from action-planning skill or user-provided)
output: Per-initiative deliverables, reviewed and consolidated
next_skills: [metric-driven-validation, stakeholder-review, deliverable-finalization]
---

# subagent-driven-execution

## When to use this skill

**Triggers** — activate when the user confirms an Action Plan and signals execution:

- "Go ahead", "Execute the plan", "Kick this off", "Let's do it"
- "Start working on initiative 1", "Run the first three"
- "Here's my plan — execute it" (user provides their own Action Plan)

**Prerequisites** — this skill expects an Action Plan to already exist, either:

- Produced by `action-planning` in this session (preferred path)
- Provided directly by the user as a numbered list with enough detail per initiative
- Referenced from a saved `action-plan-*.md` file in the working folder

If the user says "go ahead" but there's no Action Plan in context, don't start executing. Ask: "I don't have an action plan to execute against yet. Want me to help break this into initiatives first?" and hand off to `action-planning`.

**Do NOT trigger** when:

| User's ask looks like… | Hand off to |
|---|---|
| "Break this into tasks / initiatives" | `action-planning` |
| "Are we on track?" | `metric-driven-validation` |
| "Any blockers before we continue?" | `stakeholder-review` |
| "Write this up for the board" | `deliverable-finalization` |

---

## Load org context first

Invoke `load-org-context` with:

```
required_fields:    [company_name, one_liner]
recommended_fields: [connected_tools, leadership_team, known_constraints]
```

`connected_tools` is especially important here — it determines what the subagents can actually do (pull data from Monday, post to Slack, update Drive, etc.). If missing, subagents fall back to producing markdown artifacts in the working folder. See [reference/mcp-detection.md](../../reference/mcp-detection.md).

---

## The execution model

### How subagents work in this skill

Each initiative in the Action Plan gets its own **fresh subagent** — a parallel Claude agent spawned to handle that one piece of work. Subagents are valuable here because:

1. **Context isolation.** Each subagent starts clean with only the initiative spec and relevant org context — no cross-contamination from other initiatives.
2. **Parallelism.** Independent initiatives can run simultaneously.
3. **Failure containment.** If one subagent fails or produces bad output, it doesn't block the others.

### What a subagent receives

Every subagent is briefed with:

```
1. The initiative spec (from the Action Plan):
   - Initiative number and name
   - Owner
   - Success metric / "done" definition
   - Timeline and milestones
   - Dependencies on other initiatives

2. Org context (from load-org-context):
   - Company name, one-liner, stage
   - Connected tools available for this session
   - Constraints relevant to this initiative
   - Leadership team (for stakeholder references)

3. Source material (from earlier in the pipeline):
   - Strategy Brief (if one exists from strategic-discovery)
   - Situation Assessment (if one exists from situation-analysis)

4. Tool access:
   - Whatever MCPs are connected this session
   - Web search (if available)
   - File read/write in the working folder
```

### What a subagent can do

Subagents are dispatched into one of four modes depending on what the initiative requires:

| Mode | When to use | Example |
|---|---|---|
| **Research** | Initiative requires gathering information before acting | "Audit portfolio coverage across top 6 logos", "Research referral program benchmarks in B2B agencies" |
| **Draft** | Initiative requires producing a document or artifact | "Draft a referral program one-pager", "Write a QBR agenda for Willow Bridge", "Create a job description for a growth AM" |
| **Coordinate** | Initiative requires posting/updating across tools | "Create initiative cards on Monday board", "Post a kickoff summary to Slack", "Add milestones to the project tracker" |
| **Analyze** | Initiative requires crunching numbers or evaluating data | "Run a unit economics model at 2000 clients", "Score account health across top 10 logos", "Model referral program ROI at different incentive tiers" |

A single initiative may require multiple modes in sequence (research → analyze → draft). The subagent handles the full sequence; don't split one initiative across multiple subagents.

---

## Execution procedure

### Step 1 — Parse the Action Plan

Read the Action Plan and extract each initiative as a structured object:

```
{
  number: 1,
  name: "Portfolio coverage audit",
  owner: "Claire",
  metric: "Complete coverage map for top 6 logos",
  timeline: "2 weeks",
  dependencies: [],
  mode: "research"    // inferred from the initiative description
}
```

Present the parsed list to the user:

> I've parsed **N initiatives** from the Action Plan. Here's how I'd sequence them:
>
> **Wave 1 (parallel — no dependencies):**
> - Initiative 1: Portfolio coverage audit [research] → Claire
> - Initiative 2: Referral program benchmarks [research] → Mike
>
> **Wave 2 (depends on Wave 1):**
> - Initiative 3: Draft referral program structure [draft] → Mike
> - Initiative 4: Expansion playbook for strategists [draft] → Claire
>
> **Wave 3 (depends on Wave 2):**
> - Initiative 5: Launch referral program [coordinate] → Mike
>
> _Does this sequencing look right? Want to adjust anything before I show the resource plan?_

Wait for confirmation before proceeding to the resource plan.

### Step 1.5 — Resource plan (mandatory — no surprises)

**Before dispatching ANY subagent, show the user exactly what tools each subagent will use and what it will do.** This step exists because dogfood testing revealed that subagents consuming external tool credits (Ahrefs queries, CRM API calls, Slack posts) without warning felt blindsiding. The user must consent to resource usage before execution starts.

For each wave, present a resource plan:

> **Wave 1 resource plan:**
>
> | Initiative | Tools used | Operations | Estimated scope |
> |---|---|---|---|
> | 1: Portfolio coverage audit | **Ahrefs** (SEO data), **Monday** (client list) | Read-only queries | ~10–15 Ahrefs queries, ~5 Monday board reads |
> | 2: Referral program benchmarks | **Web search**, **Google Drive** (past docs) | Read-only queries | ~8 web searches, ~3 Drive doc reads |
>
> **What this will NOT do:**
> - No Slack posts or messages (coordinate mode is Wave 3)
> - No writes to any external tool (research mode is read-only)
> - No emails sent
>
> **Cost/credit note:** Initiative 1 will consume Ahrefs API credits. If you'd prefer I skip Ahrefs and work from data you provide, say so.
>
> _Approve Wave 1 resource plan? I won't start until you confirm._

**Rules for the resource plan:**

- **List every MCP the subagent will touch** — by canonical name from the tool catalog, not by internal tool ID
- **Distinguish read vs. write operations** — "read-only queries" vs. "will post to Slack" vs. "will create cards on Monday"
- **Estimate scope** — "~10 Ahrefs queries" not "some queries." Precision isn't required but order-of-magnitude matters.
- **Flag credit/cost implications** — if a tool has usage-based pricing (Ahrefs, Apollo, paid API tiers), call it out explicitly
- **List what the wave will NOT do** — this is as important as what it will do, because users fear unintended side effects
- **Offer opt-outs per tool** — "If you'd prefer I skip [tool] and work from data you provide, say so"
- **One resource plan per wave** — don't dump all waves at once. Show Wave 1's plan, get approval, execute, then show Wave 2's plan.

Wait for explicit approval before dispatching subagents.

### Step 2 — Dispatch subagents per wave

For each wave, launch subagents for all initiatives in that wave. Independent initiatives within a wave run in parallel.

Each subagent prompt follows this template:

```
You are executing one initiative from a business action plan for [company_name].

## Initiative
[paste the initiative spec]

## Context
[paste relevant org context, strategy brief excerpt, constraints]

## Available tools
[list the MCPs connected this session, per mcp-detection]

## Your task
[mode-specific instructions: research/draft/coordinate/analyze]

## Output requirements
- Be specific to [company_name] and this initiative — no generic advice
- Include sources and evidence for any claims
- Format for stakeholder readability
- Flag any blockers, missing data, or assumptions you had to make
```

### Step 3 — Two-stage review (per subagent)

Every subagent output goes through two reviews before being presented to the user. This is the quality gate that separates disciplined execution from "Claude just made something up."

#### Stage 1: Spec compliance

Check the output against the initiative spec from the Action Plan:

- [ ] Does the output address what the initiative asked for?
- [ ] Does it meet the success metric / "done" definition?
- [ ] Does it respect the stated timeline and constraints?
- [ ] Does it account for dependencies on other initiatives?
- [ ] Is it scoped correctly — not too narrow (missed parts) or too broad (scope creep)?

If spec compliance fails, note the gap and fix it before proceeding to Stage 2. Do not present non-compliant work to the user.

#### Stage 2: Quality check

Check the output for accuracy and stakeholder-readiness:

- [ ] Is the content accurate? (No hallucinated numbers, no fabricated sources)
- [ ] Is it specific to the company and initiative? (No generic filler)
- [ ] Is it well-structured and scannable? (Headings, bullets, tables where appropriate)
- [ ] Could a stakeholder (Mike, Claire, or whoever owns this initiative) read it and act on it without heavy editing?
- [ ] Are assumptions and gaps flagged explicitly rather than glossed over?

#### Severity flags

If either review surfaces issues, flag them:

| Severity | Meaning | Action |
|---|---|---|
| **Critical** | Output is wrong, misleading, or violates a constraint | Block. Fix before presenting to user. |
| **Major** | Output is incomplete or unclear on a key point | Warn. Present to user with the gap flagged. |
| **Minor** | Formatting, tone, or minor structural issues | Note. Fix silently or mention in passing. |

### Step 4 — Present results to user

After each wave completes, present the reviewed outputs:

> **Wave 1 complete.** Here are the results:
>
> ### Initiative 1: Portfolio coverage audit
> **Review:** Spec ✓ · Quality ✓
> [output summary or full artifact]
>
> ### Initiative 2: Referral program benchmarks
> **Review:** Spec ✓ · Quality — Major gap flagged
> [output summary with gap noted]
>
> _Review these before I proceed to Wave 2. Any revisions needed?_

Wait for user sign-off before proceeding to the next wave.

### Step 5 — Save artifacts

For each initiative output:

- **Short outputs** (≤20 lines): keep inline in the conversation
- **Longer outputs**: save as `initiative-N-[slugified-name].md` in the working folder
- **Coordinate-mode outputs** that posted to external tools: confirm what was posted and where

After all waves complete, save a consolidated execution summary:

```
execution-summary-[slugified-plan-name].md
```

This file lists each initiative, its status (complete / complete-with-gaps / blocked), deliverables produced, and any open items.

---

## Handling failures and blocks

**Subagent produces unusable output.** Don't silently retry. Tell the user: "Initiative 3 didn't land — the referral program draft was too generic and didn't account for your retainer model. Want me to re-run it with more specific guidance, or would you rather draft this one yourself?"

**Initiative is blocked by a dependency.** If Wave 2 depends on Wave 1 output that failed review, don't proceed with the dependent initiative. Flag the block: "Initiative 4 depends on the portfolio audit from Initiative 1, which had a major gap. Let me fix Initiative 1 first."

**MCP is unavailable for a coordinate-mode initiative.** Fall back per [mcp-detection.md](../../reference/mcp-detection.md). Produce the artifact locally and tell the user where to push it manually.

**User wants to skip an initiative.** Fine. Mark it `skipped` in the execution summary and proceed with the remaining wave.

**User wants to change an initiative mid-execution.** Pause the current wave, incorporate the change, re-present the updated initiative for confirmation, then resume.

---

## What NOT to do

**Do not execute without a confirmed Action Plan.** If the user says "just go" with only a vague strategy brief, push back: "I need a specific list of initiatives to execute against. Want me to break this strategy into an action plan first?"

**Do not skip the two-stage review.** Every subagent output goes through both stages. The review is what prevents "Claude just generated a 500-word thing and nobody checked it" — the most common failure mode in multi-agent systems.

**Do not present Wave 2 before Wave 1 is approved.** Waves are sequential gates. The user must sign off on each wave before the next one starts.

**Do not let subagents hallucinate data.** If a research-mode subagent can't find real data, it must say so explicitly — "I couldn't find Q2 close rates in any connected tool; here's what I'd need to complete this" — not fabricate numbers.

**Do not scope-creep inside a subagent.** Each subagent handles exactly one initiative. If it discovers adjacent work that should be done, it flags it as a recommendation in its output — it doesn't do it.

---

## Quality checks

Before marking execution complete:

- [ ] A resource plan was shown for every wave BEFORE dispatch — listing tools, operation types, estimated scope, and cost/credit implications
- [ ] User explicitly approved each wave's resource plan before subagents ran
- [ ] Every initiative in the Action Plan has a status (complete / complete-with-gaps / blocked / skipped)
- [ ] Every completed initiative passed both review stages (spec compliance + quality)
- [ ] All critical issues were resolved before output was presented to the user
- [ ] All major issues were flagged to the user (not hidden)
- [ ] Artifacts were saved for outputs >20 lines
- [ ] Execution summary file exists in the working folder
- [ ] No subagent hallucinated data — all claims are sourced or flagged as assumptions
- [ ] No initiative scope-crept beyond its spec
- [ ] Waves were presented sequentially with user sign-off between each
- [ ] No external tool was used that wasn't listed in the resource plan
