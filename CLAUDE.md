# Business Superpowers

A workflow orchestration plugin for Claude. This is the meta-instructions file — the plugin-level playbook that governs how the skills in this pack cooperate.

> **Inspiration:** [obra/superpowers](https://github.com/obra/superpowers) and [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done) for coding. This is that shape, rewritten for strategy, operations, and planning work.

---

## What this plugin does

Ships a disciplined pipeline for business work with composable, automatically-triggered skills:

```
strategic-discovery
      │
      ▼
situation-analysis
      │
      ▼
action-planning ───────────────────► rocks/OKRs/V2MOM/4DX (if methodology bundle installed)
      │
      ▼
subagent-driven-execution
      │
      ▼
metric-driven-validation
      │
      ▼
stakeholder-review
      │
      ▼
deliverable-finalization

Out-of-band diagnostic: root-cause-analysis
```

Every phase ends at a human approval gate. Progressive disclosure is mandatory — no multi-page walls of text for the user to sign off on.

---

## How skills activate

**Context-triggered, not name-triggered.** Users never have to remember skill names. Each skill's frontmatter declares the natural phrases and situations that should cause it to fire. Representative triggers:

| User intent | Skill that should fire |
|---|---|
| "Help me think through…", "I'm trying to decide…", "Let's plan…" | `strategic-discovery` |
| "Run a SWOT", "Check the competitive landscape", "Where does this stand?" | `situation-analysis` |
| "Break this into initiatives", "Turn this into a plan", "Who owns what?" | `action-planning` |
| "Go ahead", "Execute the plan", "Kick this off" | `subagent-driven-execution` |
| "Are we on track?", "Check the KPI", "How's the leading indicator?" | `metric-driven-validation` |
| "Ready to ship this?", "Any blockers before we move on?" | `stakeholder-review` |
| "Write this up for the board", "Package this for distribution" | `deliverable-finalization` |
| "Why did we miss the target?", "Let's pre-mortem this", "What went wrong?" | `root-cause-analysis` |

If multiple skills could apply, fire the one earliest in the pipeline (discovery over planning, planning over execution). Never fire a downstream skill without evidence that its upstream approval gate was passed.

---

## The mandatory opening step: load org context

**Every skill in this pack invokes `load-org-context` in its opening step. This is not optional.**

`load-org-context` runs a three-step cascade:

1. **Check Claude memory** for `business-superpowers` entries (company, methodology, tools, team, goals, constraints). Hit? Use it.
2. **Check for `BUSINESS_CONTEXT.md`** in the working folder, then parent folders. Hit? Load it, backfill memory if memory was missing fields.
3. **Ask the user** only for the fields the current skill actually needs and that neither source provided. On answer, write back to both memory and `BUSINESS_CONTEXT.md`.

Skills declare their required context fields in their frontmatter. Only ask for what's needed, never flood the user with a mega-form.

---

## Methodology agnosticism (load-bearing)

Core workflow skills **never hardcode a planning methodology.** They use generic vocabulary: *initiative, milestone, owner, KPI, review*. When a methodology bundle is installed (`business-superpowers-eos`, `business-superpowers-okrs`, etc.), core skills detect it via plugin presence and adapt output format.

If `BUSINESS_CONTEXT.md` declares `planning_methodology: eos` but the `eos` bundle is not installed, core skills still work — they just won't produce Rocks/IDS-shaped artifacts. Offer to install the bundle; don't block.

---

## Tool agnosticism

Same principle for integrations. Don't assume Monday.com, Linear, Jira, Slack, Drive, or Notion. Detect what MCPs are connected, use what's available, and gracefully explain what you couldn't do if a key integration is missing. See [reference/mcp-detection.md](reference/mcp-detection.md).

---

## Progressive disclosure rule

When presenting plans, situation assessments, or drafts for approval, break output into **digestible sections with explicit sign-off between them.** Never dump a 10-page plan in one message and ask "looks good?" Ask for approval on framing first, then structure, then content — one layer at a time.

---

## Subagent dispatch rule

When a skill dispatches subagents (via the `subagent-driven-execution` pattern), every subagent output goes through a **two-stage review**:

1. **Spec compliance** — does this output match what the Action Plan said to produce?
2. **Quality check** — is it accurate, well-structured, and stakeholder-ready?

Flag gaps by severity: **Critical / Major / Minor.** Critical gaps block forward progress until fixed.

---

## Cross-platform notes

This plugin is designed to work identically on:

- **Claude Code** — installed via `/plugin marketplace add` or placed in `~/.claude/plugins/`
- **Claude Desktop** — same skill triggers, same BUSINESS_CONTEXT.md discovery
- **Cowork** — install via the Cowork plugin system; skills auto-trigger without slash-command invocation

The one platform-specific behavior is memory: Claude Code and Cowork both support persistent memory across sessions; Claude Desktop's memory model is more limited. `load-org-context` treats memory as a best-effort cache — `BUSINESS_CONTEXT.md` is always the source of truth.

---

## What this plugin is NOT

- **Not a coding pack.** See [obra/superpowers](https://github.com/obra/superpowers) if you want that.
- **Not a role-persona pack.** No CEO/COO/CFO personas. See [garrytan/gstack](https://github.com/garrytan/gstack) if you want that.
- **Not a marketing kitchen sink.** We cite `coreyhaines31/marketingskills` and `alirezarezvani/claude-skills` in the README for marketing-specific work.
- **Not an atomic frameworks library.** Frameworks appear inline as reference material inside `situation-analysis`, not as standalone skills.
- **Not a document producer by default.** PPTX/DOCX output happens only on explicit user request.

---

## For contributors

Each skill lives in `skills/<skill-name>/SKILL.md` and follows the six-section anatomy:

1. **Frontmatter** — `name`, `description` (with strong trigger phrases), `version`, `required_context_fields`
2. **When to use this skill** — concrete trigger examples
3. **Load org context first** — mandatory invocation of `load-org-context`
4. **Guided questions / procedure** — the Socratic or procedural steps
5. **Output format** — inline markdown, saved artifact, or escalation rules
6. **Quality checks** — what makes output good vs. generic

See any existing skill for a template.
