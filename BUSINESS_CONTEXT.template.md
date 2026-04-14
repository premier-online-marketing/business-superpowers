# BUSINESS_CONTEXT.md

> Copy this file into your workspace (same folder you'll be running Claude sessions from, or a parent folder) and fill in what you can. The `business-superpowers` plugin reads this file so every skill can skip the "remind me what your company does" preamble.
>
> **You don't have to fill in everything.** Skills will ask for missing fields only when they actually need them, and they'll write your answers back to this file automatically.
>
> Format is free-form markdown under each heading. One or two sentences each is plenty.

---

## Required
*(most skills will block until these are filled in)*

### Company name


### One-liner
*(What does the company do, in one sentence, to someone who's never heard of it?)*


### Stage
*(One of: `early-stage` (idea / pre-seed / seed / pre-PMF), `funded-growth` (Series A/B/C / scaling / pre-profit), `established` (profitable / public / mature / non-profit). Free-form OK if none of these fit — e.g. `bootstrapped`, `stealth`, `acquired`.)*


### Revenue model
*(One of: `services-retainer` (agency / consulting / done-for-you), `saas-subscription` (recurring software or membership), `marketplace-transaction` (per-transaction fees / take-rate). Free-form OK — e.g. `ad-supported`, `licensing`, `usage-based`, `hybrid`, `not-yet-revenue`, `hardware`.)*


---

## Recommended
*(fills in over time; most skills work without these but produce better output with them)*

### Primary customer segment
*(Who buys / uses this? One sentence.)*


### Ideal customer profile (ICP)
*(The narrow wedge: industry, size, role, trigger event. Be specific.)*


### Top 3 competitors
*(Name + one-line on how they differ from you. Include indirect competitors and "the status quo" if relevant.)*

1.
2.
3.

### Current strategic goals (next 6–12 months)
*(Numbered list, 3–5 items. Outcome-shaped, not activity-shaped.)*

1.
2.
3.

### Known constraints
*(Budget caps, hiring freezes, regulatory limits, key-person dependencies, platform dependencies, etc.)*


---

## Optional
*(only fill in what's genuinely load-bearing for your work)*

### Planning methodology
*(eos / okrs / v2mom / 4dx / ad-hoc / none. Skills detect this to decide whether to produce Rocks, OKRs, WIGs, etc. If a methodology bundle isn't installed, the core skills produce generic output.)*


### Leadership team
*(Name — role — seat. Used so skills can propose owners and pick the right approver for reviews.)*

- Name —
- Name —
- Name —

### Connected tools
*(Which external tools does this org actually use day-to-day? Skills read this to know where to pull data and push deliverables. Use the shortest common name of each tool.)*

Examples by category (list whichever apply — don't feel obligated to fill every category):

- Project tracker: e.g. `monday`, `linear`, `jira`, `asana`, `notion`
- Messaging: e.g. `slack`, `teams`
- Email: e.g. `gmail`, `outlook`
- Calendar: e.g. `google_calendar`, `outlook_calendar`
- Docs / wiki: e.g. `google_drive`, `notion`, `confluence`, `onedrive`
- Data / analytics: e.g. `sheets`, `airtable`, `bigquery`, `snowflake`
- CRM: e.g. `salesforce`, `hubspot`, `pipedrive`, `attio`
- Anything else you rely on: just name it (e.g. `ahrefs`, `stripe`, `quickbooks`, `docusign`, `figma`, `zapier`, etc.)


### Recent strategic bets and their status
*(Last 2–4 big decisions and where they've landed: working / too-early-to-tell / abandoned / reversed. Gives skills real precedent to reason from instead of hypotheticals.)*


---

## Metadata

*This section is maintained by skills automatically — you don't need to edit it.*

- Last updated: `YYYY-MM-DD`
- Updated by: `<skill-name>`
- Fields filled: `required: X / Y`, `recommended: X / Y`, `optional: X / Y`
