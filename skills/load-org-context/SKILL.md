---
name: load-org-context
description: Progressive org-context discovery. Invoked by every other skill in the pack as its opening step. Runs a three-step cascade — Claude memory → BUSINESS_CONTEXT.md → AskUserQuestion — and only prompts the user for missing fields that the calling skill actually needs. Writes answers back to both memory and file so the user never re-explains their org. This skill is not directly triggered by the user; it is invoked by other skills.
version: 0.1.0
invoked_by: all workflow skills (mandatory opening step)
writes_to: [claude_memory, BUSINESS_CONTEXT.md]
required_context_fields: []
---

# load-org-context

> **This skill is invoked by other skills, not by the user directly.** Every core workflow skill in this pack calls `load-org-context` in its opening step. If you are the user and you are reading this, you don't need to do anything — just start the conversation naturally.

## When this skill activates

At the start of any other `business-superpowers` skill. The calling skill passes two lists:

- `required_fields` — the calling skill will not proceed without these
- `recommended_fields` — nice to have; the calling skill produces better output with them

This skill resolves those fields from the three-step cascade and returns a populated context object to the caller.

## The three-step cascade

### Step 1 — Claude memory check

Look for memory entries under the `business-superpowers/` prefix. Match against the caller's `required_fields + recommended_fields` list. Keep a running `resolved` map and a running `unresolved` list.

Memory keys:
- `business-superpowers/company_name`
- `business-superpowers/one_liner`
- `business-superpowers/stage`
- `business-superpowers/revenue_model`
- `business-superpowers/primary_customer_segment`
- `business-superpowers/ideal_customer_profile`
- `business-superpowers/top_3_competitors`
- `business-superpowers/current_strategic_goals`
- `business-superpowers/known_constraints`
- `business-superpowers/planning_methodology`
- `business-superpowers/leadership_team`
- `business-superpowers/connected_tools`
- `business-superpowers/recent_strategic_bets_status`

See [reference/context-schema.md](../../reference/context-schema.md) for the full field reference.

### Step 2 — BUSINESS_CONTEXT.md check

If `unresolved` is non-empty after Step 1, look for `BUSINESS_CONTEXT.md`:

1. In the current working directory
2. Walking up to the nearest parent directory that contains one (stop at `~` or the nearest `.git` root)
3. Fall back to `BUSINESS_CONTEXT.template.md` only for schema reference, never as a data source

Parse the file leniently — look for markdown headings that match field names (case-insensitive, whitespace-tolerant) and take the free-form content underneath until the next heading.

**Conflict policy:** if a field is filled in `BUSINESS_CONTEXT.md` and also in memory, the file wins. Overwrite memory with the file's value. The file is the source of truth.

After Step 2, update `resolved` with file-sourced values and re-compute `unresolved`.

### Step 3 — Asking the user

If `unresolved` still contains any of the caller's `required_fields`, ask the user. **Use the right tool for each field type** — this is the single most important rule in the skill because violating it produces terrible UX:

#### Field-type routing

| Field type | Examples | Ask mechanism |
|---|---|---|
| **Enum** — 2–4 canonical archetypes cover the space | `stage`, `revenue_model`, `planning_methodology` | **AskUserQuestion** with `options` + auto-Other |
| **Open-ended text** — anything not reducible to 2–4 archetypes | `company_name`, `one_liner`, `primary_customer_segment`, `ideal_customer_profile`, `top_3_competitors`, `current_strategic_goals`, `known_constraints`, `leadership_team`, `recent_strategic_bets_status` | **Plain conversational message.** Ask in chat; user types a reply. DO NOT use AskUserQuestion. |

**Why this matters.** AskUserQuestion is Claude's multiple-choice UI; it requires 2–4 `options` per question. When used for open-ended fields, Claude has to manufacture fake options and force the user to navigate to "Other" just to type. Observed UX failure: user had to arrow-key through 2–3 fake options to reach "Other," then wasn't sure whether to select an option AND type, or just type. Never put an open-ended field in an AskUserQuestion call.

#### Asking procedure

1. **Partition the unresolved list** into `enum_fields` and `open_fields`.
2. **Ask enum fields first** in a single AskUserQuestion call (up to 4 questions, each with ≤3 real options + auto-Other).
3. **Ask open fields next** in a single plain-text message. Format:
   > "A few more quick things — just reply in one message:
   >  (a) _<question about field 1>_
   >  (b) _<question about field 2>_
   >  (c) _<question about field 3>_"
   Keep it under 4 open questions per batch; beyond that, users tend to skip or give one-word answers that trigger shape flags.
4. **Parse the reply** for each labeled sub-question. If a label's answer is missing or ambiguous, ask once more in plain text ("I didn't catch your answer for (b) — want to fill that in, or skip it?"). Never silently drop.
5. **Required fields first, recommended second, stop.** Never ask for fields the caller didn't request.
6. **Never invent placeholder values.** If the user skips a required field, return a clear error to the caller so the caller can decide how to degrade gracefully.

#### Canonical enum choices

Use these exact option sets unless the caller overrides. Each is 3 options + auto-"Other". `description` field on each option is the disambiguation hint.

**stage**
1. "Early-stage" — idea, pre-seed, seed, or pre-product-market-fit
2. "Funded growth" — Series A/B/C, scaling, pre-profitability
3. "Established" — profitable, public, mature operations, or non-profit

*Other* handles: bootstrapped, late-stage-private, stealth, acquired, etc.

**revenue_model**
1. "Services / retainer" — agency, consulting, professional services, done-for-you
2. "SaaS / subscription" — recurring software or membership revenue
3. "Marketplace / transaction" — per-transaction fees, marketplace take-rate, commission

*Other* handles: ad-supported, licensing, usage-based, hybrid, not-yet-revenue, hardware, etc.

**planning_methodology** *(optional field — only asked when a methodology bundle is being configured)*
1. "EOS / Rocks"
2. "OKRs"
3. "None / ad-hoc"

*Other* handles: V2MOM, 4DX, Hoshin, SaFE, custom, etc.

#### Worked example

`required_fields = [company_name, one_liner, stage, revenue_model]` and memory + file had none of them.

**Pass 1 — enums via AskUserQuestion** (one call, two questions):

| Question | Options |
|---|---|
| What stage is the company at? | Early-stage · Funded growth · Established (+ Other) |
| How does the company make money? | Services / retainer · SaaS / subscription · Marketplace / transaction (+ Other) |

User picks from the structured options; arrow-key navigation makes sense here.

**Pass 2 — open fields via plain-text message:**

> Two more quick things — just reply in one message:
> (a) What's the company name?
> (b) One-sentence description of what it does?

User types a natural reply. No options, no arrow keys.

**After both passes:** all 4 required fields resolved. If recommended fields were also requested, do a third pass (partition into enum and open fields, same two-pass pattern).

**Do not list more than 3 options per enum question.** Violating this causes the UI to arbitrarily compress options into buckets (observed bug: `growth / public / non-profit` got collapsed into one option; `agency-retainer` was silently lost from `revenue_model`).

**Do not put open-ended fields in AskUserQuestion.** Violating this forces users to navigate through manufactured fake options to reach "Other" just to type (observed bug: user had to arrow-key past 2 fake options to reach free-text input, couldn't tell whether to select an option AND type).

## Writing back

For every field you resolved in Steps 2 or 3 (i.e. anything that wasn't already in memory), **write it back**:

1. Write to Claude memory under `business-superpowers/<field_name>`.
2. Append or update the matching heading in `BUSINESS_CONTEXT.md`:
   - If `BUSINESS_CONTEXT.md` doesn't exist in the working directory, copy it from the template at the plugin root (`BUSINESS_CONTEXT.template.md`) first, then fill in.
   - If it exists, find the matching heading and replace the content block underneath. Don't duplicate headings.
3. Update the `Metadata` section at the bottom of the file:
   - `Last updated: <YYYY-MM-DD>` (today)
   - `Updated by: <calling-skill-name> (via load-org-context)` — attribute to the caller, not to this skill, because the user experiences the caller
   - `Fields filled: required: X / 4, recommended: X / 5, optional: X / 4`

### File output conventions

To keep the file readable and to let downstream skills parse reliably, follow these conventions every time the file is touched:

- **Unfilled fields:** render the heading followed by a single italic placeholder on its own line: `*(not captured yet)*`. Never leave a heading with no content underneath — that breaks lenient parsing.
- **Shape-flagged fields:** render the heading, then a single italic note explaining the flag on its own line, then the captured content. Example for an `underspecified` + `overspecified` competitor list:

  ```
  ### Top 3 competitors

  *(User provided 5 names without per-line differentiation; captured verbatim for now — refine on next pass.)*

  1. Digible
  2. ApartmentSEO
  ...
  ```

- **List-shaped fields:** render as a numbered markdown list, one item per line. Keep the order the user provided them in.
- **Free-form fields:** one paragraph, no leading indent, no bullet.
- **Never inject editorial commentary** beyond the shape-flag note. The file is the user's source of truth, not a place for skill opinions.

## Shape validation and mismatch flags

Captured answers don't always match the declared field shape. For example:

- `top_3_competitors` expects `[{name, one_liner}, ...]` with at least 3 entries, but a user may answer with 5 names and no differentiation
- `current_strategic_goals` expects 3–5 outcome-shaped goals, but a user may give 1 activity-shaped goal
- `leadership_team` expects `[{name, role, seat?}, ...]` but a user may give names only

**Do not re-prompt and do not reject.** Capture verbatim (raw user string, or best-effort parse), write to memory and file, and return with a `shape_flags` entry so downstream skills can decide what to do.

Shape-flag values:
- `"underspecified"` — answer has the right shape but insufficient detail or below minimum count (e.g. 1 competitor when 3+ expected, names without one-liners when structure expected)
- `"overspecified"` — answer exceeds expected count (e.g. 5 competitors when 3 expected); not an error, just logged
- `"wrong_shape"` — answer can't be parsed into the expected type at all (e.g. sentence when a list was expected); caller should decide whether to re-ask with a more specific prompt
- `"ok"` — matches the schema

Example: user gives "Digible, ApartmentSEO, RentCafe, RealPage, Dyverse" for `top_3_competitors`. Store as-is, flag `underspecified` (no one-liners), flag `overspecified` (5 when 3 expected). A downstream `situation-analysis` invocation will see these flags and can offer: "I have 5 competitor names but no differentiation — want me to enrich each with a one-liner before running SWOT, or skip?"

## Return value

Return a structured context object to the caller:

```
{
  resolved: {
    company_name: "...",
    one_liner: "...",
    top_3_competitors: ["Digible", "ApartmentSEO", ...],
    ...
  },
  unresolved_required: [],   // empty on success
  unresolved_recommended: ["known_constraints"],
  sources: {
    company_name: "memory",
    one_liner: "file",
    stage: "asked",
    top_3_competitors: "asked"
  },
  shape_flags: {
    top_3_competitors: ["underspecified", "overspecified"],
    current_strategic_goals: ["underspecified"]
  }
}
```

The `sources` map lets the caller log where each field came from if useful for debugging trigger misfires or context drift. The `shape_flags` map lets the caller decide whether to enrich or re-prompt on a per-field basis, without `load-org-context` having to block the whole cascade.

## Quality checks

Before returning to the caller:

- [ ] Every field in `required_fields` is either resolved or present in `unresolved_required` — no silent drops.
- [ ] Every resolved field that came from `asked` is now persisted to both memory and file.
- [ ] No field was asked for that the caller didn't request.
- [ ] No field was asked for that was already in memory or file.
- [ ] `BUSINESS_CONTEXT.md` exists after the first run of any workflow skill (create from template on first write).
- [ ] The `Metadata` section is updated whenever the file is touched.
- [ ] No enum question was issued with more than 3 real options — violating this causes meaningful options to be silently dropped.
- [ ] No open-ended field was routed through AskUserQuestion — violating this forces the user to navigate past fake options to reach free-text entry.
- [ ] Every resolved field has a `shape_flags` entry (`ok` when no issues); no silent underspecified/overspecified captures.

## Failure modes and handling

**No memory API available** (some Desktop contexts): skip Step 1 silently; proceed to Step 2.

**`BUSINESS_CONTEXT.md` exists but is unparseable**: warn the user once, offer to re-initialize from the template, and fall through to Step 3 for required fields. Do not corrupt the existing file without explicit user consent.

**User declines to answer a required field**: return `unresolved_required: [field_name]` to the caller and let the caller decide (usually: offer to proceed with degraded output vs. abort).

**User provides conflicting info during the same session** (e.g. names the company differently mid-session): trust the latest answer, update both stores, don't re-ask.

## Progressive disclosure

Never dump the full schema on a first-time user. The first run through this skill should only expose the fields the current workflow needs. Over multiple sessions, the file fills out naturally as different skills request different fields.
