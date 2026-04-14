# BUSINESS_CONTEXT.md — full field reference

This is the authoritative schema for the `BUSINESS_CONTEXT.md` file and for memory entries keyed under `business-superpowers/*`.

Every field has a tier (**required** / **recommended** / **optional**), a short description, and a list of which skills declare that field as needed in their frontmatter. If you're building a new skill, add the field names it needs to the skill's frontmatter; `load-org-context` reads that list to decide what to check and ask for.

---

## Tiers

- **Required** — `load-org-context` will ask for these when missing and a skill that needs them is running. Skills can choose to block until these are filled in.
- **Recommended** — skills will *use* these when present but won't block. If a skill would be materially better with one of these and it's missing, the skill may offer to capture it ("Want me to fill in your top 3 competitors while we're here?") but must not force it.
- **Optional** — never asked for unless a skill explicitly requests them. Useful for advanced workflows and methodology bundles.

---

## Required fields

### `company_name`
The legal or trading name of the company.

**Type:** string
**Used by:** every skill (for naming and attribution in deliverables)

### `one_liner`
One sentence describing what the company does. Should be legible to someone outside the industry.

**Type:** string (1–2 sentences)
**Used by:** `strategic-discovery`, `situation-analysis`, `deliverable-finalization`

### `stage`
The company's maturity stage. Stored free-form. When asked via AskUserQuestion, the three archetype options are:

- `early-stage` — idea, pre-seed, seed, or pre-PMF
- `funded-growth` — Series A/B/C, scaling, pre-profitability
- `established` — profitable, public, mature operations, or non-profit

"Other" is auto-added and captures anything else (bootstrapped, late-stage-private, stealth, acquired). The file may store the user's verbatim answer; skills read leniently.

**Type:** string
**Used by:** `strategic-discovery` (to calibrate risk tolerance), `action-planning` (to calibrate horizon), `situation-analysis`

### `revenue_model`
How the company makes money. Stored free-form. When asked via AskUserQuestion, the three archetype options are:

- `services-retainer` — agency, consulting, professional services, done-for-you
- `saas-subscription` — recurring software or membership revenue
- `marketplace-transaction` — per-transaction fees, marketplace take-rate, commission

"Other" is auto-added and captures anything else (ad-supported, licensing, usage-based, hybrid, not-yet-revenue, hardware, etc.).

**Type:** string
**Used by:** `situation-analysis`, `metric-driven-validation` (to pick the right unit economics lens), `root-cause-analysis`

---

## Recommended fields

### `primary_customer_segment`
Who buys or uses the product. One-sentence description.

**Type:** string
**Used by:** `strategic-discovery`, `situation-analysis`, `action-planning`

### `ideal_customer_profile`
The narrow wedge: industry + size + role + trigger event. More specific than `primary_customer_segment`.

**Type:** string (structured prose OK)
**Used by:** `situation-analysis`, `action-planning` (for targeting)

### `top_3_competitors`
Named competitors with a one-line differentiation each.

**Type:** list of `{name, one_liner}`
**Used by:** `situation-analysis` (especially for SWOT and Porter's), `strategic-discovery`

### `current_strategic_goals`
3–5 outcome-shaped goals for the next 6–12 months.

**Type:** ordered list of strings
**Used by:** `strategic-discovery`, `action-planning`, `stakeholder-review`, `deliverable-finalization`

### `known_constraints`
Free-form description of what's blocking or boxing the company: budget caps, hiring freezes, regulatory limits, key-person dependencies, platform dependencies.

**Type:** string
**Used by:** `action-planning`, `stakeholder-review`, `root-cause-analysis`

---

## Optional fields

### `planning_methodology`
The org's formal planning system. Common values: `eos`, `okrs`, `v2mom`, `4dx`, `ad-hoc`, `none`.

**Type:** enum-like string
**Used by:** Core workflow skills to detect whether a methodology bundle should handle output formatting. If the matching bundle is installed, its output format is used; if not, generic output is produced and the user is optionally offered the bundle install.

### `leadership_team`
Named leadership with role and (optionally) their functional seat. Used by skills to propose owners for initiatives and to pick the right approver for reviews.

**Type:** list of `{name, role, seat?}`
**Used by:** `action-planning` (owner assignment), `stakeholder-review` (approver routing), `deliverable-finalization` (attribution)

### `connected_tools`
Which MCPs this org has connected — `monday`, `linear`, `jira`, `asana`, `trello`, `slack`, `google-drive`, `notion`, `airtable`, `salesforce`, `hubspot`, etc.

**Type:** list of strings
**Used by:** `situation-analysis` (for data-pull routing), `subagent-driven-execution` (for tool dispatch), `deliverable-finalization` (for distribution routing)

**Note:** Skills should still *detect at runtime* which MCPs are actually connected in the current session via the MCP detection logic in [mcp-detection.md](./mcp-detection.md). This field is a declared intent, not a live truth.

### `recent_strategic_bets_status`
Last 2–4 significant decisions and their current status. Gives skills real precedent to reason from instead of hypotheticals.

**Type:** list of `{bet, status, date}` where status ∈ `working`, `too-early-to-tell`, `abandoned`, `reversed`
**Used by:** `strategic-discovery` (to reference past patterns), `root-cause-analysis` (to cross-reference), `situation-analysis`

---

## How fields are stored

### In `BUSINESS_CONTEXT.md`
Markdown headings matching the field names, free-form prose under each. Skills parse leniently — they look for a heading match and read the following content block.

### In Claude memory
Memory keys are prefixed `business-superpowers/`. Each field becomes one memory entry:

- `business-superpowers/company_name` — string
- `business-superpowers/one_liner` — string
- `business-superpowers/stage` — string
- `business-superpowers/planning_methodology` — string (if filled)
- etc.

Memory is treated as a **best-effort cache.** The `BUSINESS_CONTEXT.md` file is the source of truth. When the file and memory disagree, the file wins and memory is refreshed.

---

## Writing fields back

When a skill asks the user for a missing field, on receiving the answer it MUST:

1. Write the field to Claude memory with the `business-superpowers/` prefix.
2. Append or update the matching heading in `BUSINESS_CONTEXT.md` in the working folder. If no `BUSINESS_CONTEXT.md` exists yet, create one from the template at `BUSINESS_CONTEXT.template.md`.
3. Update the `Metadata` section of the file with `Last updated`, `Updated by` (the skill name), and the field-count summary.

Never ask for the same field twice in a session once it's been captured.

---

## Adding a new field

To add a new field to the schema:

1. Add the field here with its tier, type, and using-skills.
2. Add a matching heading to `BUSINESS_CONTEXT.template.md`.
3. Update every skill's frontmatter `required_context_fields` / `recommended_context_fields` as appropriate.
4. `load-org-context` picks up new fields automatically from skill frontmatter — no changes needed there.
