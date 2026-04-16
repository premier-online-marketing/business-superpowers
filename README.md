# Business Superpowers

**A workflow orchestration plugin for Claude.** Strategy, operations, and planning work gets the same disciplined pipeline — discover → analyze → plan → execute → validate → review → finalize — that [obra/superpowers](https://github.com/obra/superpowers) brings to coding.

Org-agnostic by design. Methodology-agnostic by default. Progressive org-context discovery via memory, `BUSINESS_CONTEXT.md`, and targeted questions. Works on Claude Code, Claude Desktop, and Cowork.

---

## How to use this

**There is no `/business-superpowers` command. You don't type slash commands at all.**

Just describe your situation in plain English. The right skill fires automatically based on what you say.

- *"Help me think through Q3 priorities."* → fires `strategic-discovery`
- *"Why did acquisition drop 30%?"* → fires `root-cause-analysis`
- *"Turn this into a plan with owners."* → fires `action-planning`

Full trigger-phrase map in [CLAUDE.md](CLAUDE.md). The pipeline starts with `strategic-discovery` — any open-ended strategic question gets you in.

---

## Why this exists

Most business-oriented Claude skill packs are one of two shapes:

1. **Role personas** — "act as a CEO", "act as a CFO" (e.g. `gstack`, `alirezarezvani/c-level-skills`). Useful for one-off advice. Not a workflow.
2. **Framework libraries** — atomic SWOT / Porter's / OKR skills. Useful for structured thinking on a single question. Not a pipeline.

Neither gives you the thing Superpowers gives coding teams: **a disciplined operating rhythm** — clarify the goal, analyze the situation, plan, execute, validate, review, finalize, with quality gates between every phase.

Business work needs that rhythm more than coding does. A bad code change fails CI. A bad strategic decision compounds across quarters, clients, and headcount.

This pack ships that rhythm as code.

---

## What's in the box

**9 skills, all context-triggered (you don't type slash commands):**

- **`load-org-context`** — progressive org-context discovery; invoked by every other skill
- **`strategic-discovery`** — Socratic refinement of any new initiative, problem, or opportunity
- **`situation-analysis`** — pulls context from connected MCPs; applies SWOT / Porter's / JTBD / pre-mortem inline as fits
- **`action-planning`** — breaks strategy into numbered initiatives with owner, timeline, metrics, dependencies
- **`subagent-driven-execution`** — dispatches fresh subagents per initiative; two-stage review (spec + quality)
- **`metric-driven-validation`** — KPI / leading-indicator checkpoints; RED-GREEN-REFACTOR as metaphor
- **`stakeholder-review`** — decision gates between phases; flags blockers by severity
- **`deliverable-finalization`** — packages outputs and distributes through connected tools
- **`root-cause-analysis`** — 4-phase diagnostic (evidence → hypotheses → test → confirm + solve)

**Reference material:** inline framework guides (SWOT, Porter's Five Forces, JTBD, pre-mortem), the `BUSINESS_CONTEXT.md` field schema, MCP-detection logic, and two end-to-end worked examples.

---

## Install

### Claude Code

```bash
/plugin marketplace add premier-online-marketing/business-superpowers
/plugin install business-superpowers
```

### Claude Desktop

See the [Claude Desktop plugin docs](https://docs.claude.com/) for the current install path. Drop this folder under your plugins directory or clone directly.

### Cowork

Install via the Cowork plugin picker, or:

```bash
git clone https://github.com/premier-online-marketing/business-superpowers.git ~/.cowork/plugins/business-superpowers
```

### Manual install (any platform)

```bash
git clone https://github.com/premier-online-marketing/business-superpowers.git ~/.claude/plugins/business-superpowers
```

---

## Quickstart

1. **Copy the context template** into your workspace:
   ```bash
   cp ~/.claude/plugins/business-superpowers/BUSINESS_CONTEXT.template.md ./BUSINESS_CONTEXT.md
   ```
   Fill in the required fields at minimum (company, one-liner, stage, revenue model). The recommended and optional fields fill in over time as skills ask for them.

2. **Ask Claude a strategic question in natural language — no slash command needed.** Examples:
   - "Help me think through Q3 priorities for the sales team."
   - "Why did new-customer acquisition drop 30% this quarter?"
   - "Let's pre-mortem the pricing change we're considering."
   - "Turn this into a plan with owners and deadlines."

3. **Claude auto-triggers the right skill.** You'll be walked through clarifying questions, shown a draft in digestible sections for sign-off, and never slapped with a 10-page wall of text.

---

## The pipeline, concretely

```
You: "Help me plan Q3 for the paid media team."

→ load-org-context          (memory check → BUSINESS_CONTEXT.md → ask for gaps)
→ strategic-discovery       (Socratic Q&A to refine the goal)
→ situation-analysis        (pulls Q2 data from connected MCPs, synthesizes)
→ action-planning           (numbered initiatives with owner / KPI / timeline)
→ stakeholder-review        (decision gate before execution)
→ subagent-driven-execution (per-initiative subagents with two-stage review)
→ metric-driven-validation  (checkpoint against KPIs)
→ deliverable-finalization  (write up the plan, distribute)
```

Every arrow is a human approval gate.

---

## Methodology support

Core skills are methodology-agnostic. Install optional bundles for methodology-specific artifacts:

- **`business-superpowers-eos`** — EOS Rocks, IDS-shaped root-cause output, L10 scorecard review _(coming in v1.1)_
- **`business-superpowers-okrs`** — OKR drafting, scoring, quarterly cadence _(coming in v1.1)_
- **`business-superpowers-v2mom`** — Salesforce-style V2MOM _(coming in v1.1)_
- **`business-superpowers-4dx`** — 4 Disciplines of Execution, WIGs, lead measures _(coming in v1.1)_

---

## Related packs (we cite, not compete)

- [**obra/superpowers**](https://github.com/obra/superpowers) — the inspiration; for coding workflows
- [**gsd-build/get-shit-done**](https://github.com/gsd-build/get-shit-done) — spec-driven coding orchestration
- [**coreyhaines31/marketingskills**](https://github.com/coreyhaines31/marketingskills) — deep marketing execution skills
- [**alirezarezvani/claude-skills**](https://github.com/alirezarezvani/claude-skills) — the largest multi-domain business library (232+ skills)
- [**slavingia/skills**](https://github.com/slavingia/skills) — Sahil Lavingia's Minimalist Entrepreneur skill pack
- [**garrytan/gstack**](https://github.com/garrytan/gstack) — role-persona stack (CEO / designer / QA)

This pack is the *operating rhythm* layer. Those packs are the *execution muscle*. They compose.

---

## Non-goals

- Replacing L10 / standup meetings — we *feed them better inputs*
- Making decisions for you — every phase ends at a human approval gate
- One-shot framework generation (SWOT on demand with no org context) — we force structured thinking because frameworks without context are generic filler

---

## Contributing

PRs welcome for v1.1 methodology bundles and for trigger-phrase additions. See [CLAUDE.md](CLAUDE.md) for the six-section skill anatomy that every new skill must follow.

License: [MIT](LICENSE).

---

## Status

**v0.1.0 — pre-release scaffold.** Skills are being built phase-by-phase with dogfood checkpoints between each. See the planning doc in the parent folder for the build order. Don't install this on a production workspace yet.
