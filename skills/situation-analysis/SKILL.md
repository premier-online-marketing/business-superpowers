---
name: situation-analysis
description: >
  Pulls context from connected MCPs (project trackers, CRMs, messaging, docs,
  data sources) and synthesizes into a structured assessment using an appropriate
  framework — SWOT, Porter's Five Forces, JTBD, or a custom lens. Fires on
  "run a SWOT", "check the competitive landscape", "where does this stand?",
  "what's the current state of X?", "pull the data on this". Uses inline
  framework reference docs rather than separate framework skills.
version: 0.1.0
required_context_fields: [company_name, one_liner, stage, revenue_model]
recommended_context_fields: [primary_customer_segment, top_3_competitors, current_strategic_goals, connected_tools]
input: A Strategy Brief (from strategic-discovery) or a direct user question about competitive/market/internal positioning
output: Situation Assessment (inline markdown or saved .md artifact)
next_skills: [action-planning, strategic-discovery]
---

# situation-analysis

## When to use this skill

**Triggers:**

- "Run a SWOT on this", "Do a competitive analysis", "What's the landscape look like?"
- "Where does this stand?", "What's the current state of X?"
- "Pull the data on this", "What does Monday / our pipeline / the tracker say?"
- "Check what's happening in Slack about X", "Search Drive for our strategy doc"
- Auto-triggered by `strategic-discovery` when the Strategy Brief references a market, competitor, or client that needs grounding in data.

**Do NOT trigger:**

| User's ask looks like… | Hand off to |
|---|---|
| "Help me think through…" (still fuzzy) | `strategic-discovery` |
| "Break this into tasks" | `action-planning` |
| "Why did X fail?" | `root-cause-analysis` |

---

## Load org context first

Invoke `load-org-context` with:

```
required_fields:    [company_name, one_liner, stage, revenue_model]
recommended_fields: [primary_customer_segment, top_3_competitors,
                     current_strategic_goals, connected_tools]
```

`top_3_competitors` and `connected_tools` are especially important here. If competitors are flagged `underspecified` (names only, no differentiation), this is the skill that should offer to enrich them.

---

## Guided procedure

### Step 1 — Choose the framework

Based on what the user asked and what context is available, pick the most useful lens. Don't default to SWOT every time — pick the framework that answers the actual question.

| User's question shape | Best framework | Reference doc |
|---|---|---|
| "What are our strengths / weaknesses / opportunities / threats?" | SWOT | [reference/frameworks/swot.md](../../reference/frameworks/swot.md) |
| "How attractive is this market?", "Who has power?" | Porter's Five Forces | [reference/frameworks/porters-five-forces.md](../../reference/frameworks/porters-five-forces.md) |
| "What job are our customers hiring us for?", "Why do they buy?" | JTBD | [reference/frameworks/jtbd.md](../../reference/frameworks/jtbd.md) |
| "What could go wrong if we launch this?" | Pre-mortem | [reference/frameworks/pre-mortem.md](../../reference/frameworks/pre-mortem.md) |
| "What does the data say?" (no specific framework) | Custom assessment | No reference doc — structure based on the data |

Tell the user which framework you're using and why:

> "I'll run a SWOT here because you're evaluating a strategic position with both internal and external factors. If you'd prefer a different lens — competitive dynamics (Porter's), customer motivation (JTBD), or risk analysis (pre-mortem) — just say so."

### Step 2 — Pull data from connected tools

Detect available MCPs per [reference/mcp-detection.md](../../reference/mcp-detection.md) and pull relevant data. This step is what separates this skill from generic "run a SWOT" prompts — it grounds the analysis in real data, not hypotheticals.

**What to pull by category:**

| Category | What to look for | Why it matters |
|---|---|---|
| Project tracking | Current initiatives, their status, blockers | Internal capabilities / weaknesses |
| CRM | Pipeline data, win/loss rates, deal sizes | Competitive position, customer behavior |
| Messaging (Slack) | Recent threads about the topic, recurring complaints, wins | Ground-truth signals that reports miss |
| Long-form docs | Strategy docs, QBR decks, past assessments | Historical context, prior decisions |
| Data / analytics | KPIs, trends, dashboards | Quantitative evidence |

**If no relevant tools are connected:** tell the user what data would make the analysis stronger, ask them to paste or describe it, and proceed with what's available. Never fabricate data to fill the framework.

### Step 3 — Build the assessment

Apply the chosen framework to the gathered data + org context. Follow the specific framework reference doc for structure and quality rules.

**Universal rules across all frameworks:**

- **Every claim needs evidence.** "Weakness: limited sales capacity" is incomplete. "Weakness: limited sales capacity — 3 strategists double as AMs, book sizes approaching ceiling, no dedicated sales role" is grounded.
- **Be specific to the company.** "Opportunity: growing market" is generic filler. "Opportunity: multifamily apartment construction up 12% YoY in Austin metro; PMCs adding properties need marketing vendors" is useful.
- **Distinguish fact from inference.** Clearly label what came from data ("Q2 close rate was 23% per Monday pipeline") vs. what you're interpreting ("this suggests lead quality may have shifted").
- **Challenge the user's assumptions.** If the SWOT reveals the user's planned initiative is running straight into a threat they haven't considered, say so.

### Step 4 — Present with progressive disclosure

**Message 1 — Framework and data sources:**

> ## Situation Assessment: [topic]
> **Framework:** SWOT
> **Data sources:** BUSINESS_CONTEXT.md, [list of MCPs queried and what was found]
> **Data gaps:** [anything you looked for but couldn't find]
>
> _Proceeding with the analysis — I'll present each quadrant separately._

**Messages 2–N — One section per message** (for SWOT: Strengths, then Weaknesses, then Opportunities, then Threats — each as its own message with a brief confirmation pause).

**Final message — Synthesis:**

> ### So what?
> [2–3 sentences connecting the analysis to the user's actual decision or initiative. "The SWOT suggests your expansion strategy is well-positioned on strengths and opportunities, but the AM capacity weakness is the binding constraint — if that breaks, the whole plan stalls."]
>
> ### Recommended next step
> [Hand off to action-planning, or back to strategic-discovery if the analysis revealed the strategy needs rethinking]

---

## Save behavior

- **Short assessments** (≤40 lines): keep inline
- **Longer assessments**: save as `situation-assessment-[topic-slug].md` in the working folder
- **Always save** if the assessment will be referenced by downstream skills

---

## Handling shape-flagged context

If `load-org-context` returned shape flags (e.g. `top_3_competitors: underspecified`), this is the natural place to address them:

> "I have 5 competitor names but no differentiation. Before I run the competitive analysis, want me to research each one and draft a one-liner? That'll make the SWOT much sharper."

If the user says yes, do the enrichment and write the improved data back to `BUSINESS_CONTEXT.md` via `load-org-context`'s write-back mechanism.

---

## Quality checks

- [ ] Framework was chosen deliberately and the choice was explained to the user
- [ ] Every claim in the assessment cites evidence (data source, org context field, or explicit inference label)
- [ ] Analysis is specific to the company — no generic MBA filler
- [ ] Data gaps are flagged, not glossed over
- [ ] Fact vs. inference is clearly distinguished
- [ ] The "So what?" synthesis connects back to the user's actual decision
- [ ] Assessment was presented progressively (section by section), not as a wall
- [ ] If competitors were underspecified, enrichment was offered
