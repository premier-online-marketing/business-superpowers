# MCP detection and tool-agnostic dispatch

`business-superpowers` is deliberately tool-agnostic. Skills never assume Monday.com, Linear, Slack, or any specific MCP is connected. This file documents how skills detect what's available at runtime and dispatch work to whichever tools are present.

---

## Why this matters

The POM-specific blueprint hardcoded Monday.com as "the data home." That works for one org. For a public plugin, it would:

- Fail silently for orgs on Linear / Jira / Asana
- Reject installs from orgs that don't use Monday at all
- Force a tool-adoption decision that belongs to the user, not us

Instead: detect what's connected, use what's there, explain gaps.

---

## The detection cascade

At the start of any skill that needs to pull or push to external tools:

### 1. Read declared intent from `BUSINESS_CONTEXT.md`
The optional `connected_tools` field lists what the org says they use:
`monday`, `linear`, `jira`, `asana`, `trello`, `slack`, `google-drive`, `notion`, `airtable`, `salesforce`, `hubspot`, etc.

Treat this as **declared intent, not live truth.** A user may have listed `monday` in the file but not actually connected the Monday MCP to this Claude session.

### 2. Enumerate actually-connected tools

Claude exposes external services through three related but not-identical surfaces:

1. **Claude Desktop connectors** — managed OAuth connectors surfaced through the official connectors directory (Gmail, Slack, Google Calendar, Notion, Linear, Asana, Monday, Ahrefs, Apollo.io, Miro, PandaDoc, Granola, Otter, QuickBooks, Zapier, Excalidraw, BigQuery, Supabase, and more). Tools arrive prefixed **`mcp__claude_ai_<Service>__<tool>`**. The service segment can be PascalCase (`Slack`, `Gmail`), snake_case (`monday_com`, `Apollo_io`), or compound (`Google_Calendar`, `Google_Cloud_BigQuery`, `Intuit_QuickBooks`).
2. **Claude Code MCP servers** — user-installed MCP servers (local stdio, HTTP, remote). Prefix is `mcp__<server-id>__<tool>` where `<server-id>` is whatever the user named the server in their config. No consistent casing.
3. **Cowork integrations** — same MCP-over-the-wire model as Claude Code, same flexible prefixes.

**All three share the `mcp__*__<tool>` shape,** so the wildcard-middle matcher works uniformly. The gotcha is the service-name segment:

**Normalization rule.** Before matching, lowercase the service segment and strip `_com`, `_io`, `_cloud` suffixes. So `Slack` → `slack`, `monday_com` → `monday`, `Apollo_io` → `apollo`, `Google_Cloud_BigQuery` → `google_bigquery` (preserve the prefix to disambiguate from Google Calendar / Drive). Compare against a normalized catalog.

**Starter catalog.** The table below is a *boilerplate starting point*, not an exhaustive list. It covers the mainstream tool in each category that a typical business will have connected. Users are expected to extend this catalog — that's how we stay open-source-friendly instead of reflecting any one maintainer's stack.

| Category | Canonical name | Example matches (normalized service segment) |
|---|---|---|
| Project tracking | `monday` | `monday`, `monday_com` |
| Project tracking | `linear` | `linear` |
| Project tracking | `jira` | `jira`, `atlassian` |
| Project tracking | `asana` | `asana` |
| Project tracking | `notion` | `notion` (when used as DB) |
| Messaging | `slack` | `slack` |
| Messaging | `teams` | `teams`, `microsoft_teams` |
| Email | `gmail` | `gmail` |
| Email | `outlook` | `outlook`, `microsoft_outlook` |
| Calendar | `google_calendar` | `google_calendar`, `gcal` |
| Calendar | `outlook_calendar` | `outlook_calendar` |
| Long-form docs | `google_drive` | `google_drive`, `gdrive`, `google_workspace` |
| Long-form docs | `notion` | `notion` (when used as doc) |
| Data / analytics | `sheets` | `google_sheets`, `sheets` |
| Data / analytics | `airtable` | `airtable` |
| Data / analytics | `bigquery` | `bigquery`, `google_cloud_bigquery` |
| CRM | `salesforce` | `salesforce` |
| CRM | `hubspot` | `hubspot` |

Common extensions users add per category:

- **Project tracking**: `trello`, `clickup`, `smartsheet`, `basecamp`, `height`
- **Messaging**: `discord`, `mattermost`, `chanty`
- **Docs / wiki**: `onedrive`, `dropbox`, `confluence`, `coda`, `quip`
- **Data / analytics**: `snowflake`, `redshift`, `supabase`, `postgres`, `databricks`, `metabase`, `hex`
- **CRM**: `pipedrive`, `attio`, `close`, `copper`, `zoho_crm`
- **Sales intel / enrichment**: `apollo`, `clay`, `zoominfo`, `lusha`, `cognism`
- **SEO / marketing intel**: `ahrefs`, `semrush`, `moz`, `similarweb`, `search_console`
- **Meeting notes**: `granola`, `otter`, `fireflies`, `fathom`, `grain`
- **Whiteboard / visual**: `figma`, `figjam`, `miro`, `mural`, `excalidraw`
- **Contracts / signing**: `docusign`, `pandadoc`, `hellosign`, `adobe_sign`
- **Accounting / finance**: `quickbooks`, `xero`, `freshbooks`, `netsuite`, `wave`
- **Payments**: `stripe`, `paypal`, `braintree`, `adyen`
- **Automation bridge**: `zapier`, `make`, `n8n`, `workato`, `tray`
- **Dev / repo**: `github`, `gitlab`, `bitbucket`
- **Support / ticketing**: `zendesk`, `intercom`, `freshdesk`, `helpscout`

**Treat unknown services as available, not invisible.** If a skill encounters an `mcp__claude_ai_<Service>__*` or `mcp__<server-id>__*` that isn't in the catalog above and the service name doesn't normalize to any extension, surface it to the user ("I see a `Foo` connector is available — what do you use it for?") so it can be categorized and used. Never hide capability just because it isn't enumerated here.

### Extending the catalog

To add a new canonical service for the whole pack:

1. Add a row to the starter table or to the relevant extension bullet.
2. If the new service fits an existing *category* (project tracker / CRM / etc.), no skill changes are needed — category-level fallbacks handle dispatch.
3. If the new service is a whole new category, update the "Tool categories and fallbacks" table later in this doc.

For one-off customization (an org-specific internal tool), a user can add an `unmapped_tools` block to `BUSINESS_CONTEXT.md` mapping their custom server-id to a canonical name — skills will honor it.

### 3. Compute the intersection
```
available_tools = declared_intent ∩ connected_mcp_servers
missing_tools   = declared_intent - connected_mcp_servers
unexpected_tools = connected_mcp_servers - declared_intent
```

- **`available_tools`** — safe to use. Dispatch real work here.
- **`missing_tools`** — declared but not connected this session. Surface as a visible gap: "You've listed Monday in BUSINESS_CONTEXT.md but the Monday MCP isn't connected in this session. I'll skip the pipeline data pull; attach it manually or reconnect the MCP and I'll re-run."
- **`unexpected_tools`** — connected but not declared. Quietly use if the skill has reason to (e.g. a new Slack connection), and optionally suggest adding it to `BUSINESS_CONTEXT.md`.

### 4. Degrade gracefully when critical tools are missing
If a skill really needs a category of tool that has no available instance (e.g. *any* project tracker for `action-planning`, *any* messaging tool for `deliverable-finalization`), the skill must:

1. Tell the user clearly what would have happened with the tool connected.
2. Offer an inline-only fallback (produce a markdown artifact in the working folder).
3. Never silently skip the step or fabricate results.

---

## Tool categories and fallbacks

Most business workflows need one tool per category. Skills should dispatch by **category**, not by specific product — so if the user has Linear but not Monday, `action-planning` still works.

| Category | Fallback if no tool in this category is connected |
|---|---|
| Project tracking | Write a markdown initiative list to the working folder; suggest a Zap/Make recipe if an automation bridge is connected |
| Messaging | Write a "suggested message" block inline for the user to copy |
| Email | Draft the email inline as markdown |
| Calendar | Suggest meeting times inline; don't book |
| Long-form docs | Save a `.md` file in the working folder |
| Data / analytics | Ask the user to paste the data or share a link |
| CRM | Inline notes; flag that CRM updates must be manual |
| Sales intel / enrichment | Skip enrichment; note the gap explicitly |
| SEO / marketing intel | Rely on generic reasoning; flag that live SERP / traffic data is missing |
| Meeting notes | Ask user to paste transcript or meeting summary |
| Whiteboard / visual | Produce ASCII diagrams or save a Mermaid `.md` |
| Contracts / signing | Draft the document inline as markdown |
| Accounting / finance | Ask for CSV exports or summary numbers |
| Payments | Summarize what the user would see in the dashboard; don't synthesize transaction data |
| Automation bridge | Offer to draft the workflow inline for the user to wire up |

See the starter catalog + extensions above for which tools populate each category. Skills MUST NOT hardcode a specific product (e.g. "post to Slack") — they dispatch to whatever is available in the `messaging` category, falling back to inline if nothing is.

---

## Skill-level usage pattern

Every skill that touches external tools should follow this pattern at the top of its procedure:

```
1. Load org context (via load-org-context)
2. Detect available tools via the cascade above
3. Plan the work, mapping each step to a tool category
4. For each step:
     a. If the category has an available_tool → dispatch there
     b. Else → use the fallback for that category
     c. Never substitute across categories silently (don't post to Slack when the user asked for a Drive doc)
5. After execution, report per-step:
     - what was done
     - where it landed
     - what was skipped / degraded and why
```

---

## Never-assume list

These are the assumptions the POM-specific blueprint made that this pack must never make:

- ❌ Monday.com is always available
- ❌ There is a "Rocks board"
- ❌ Slack has a specific channel pattern (e.g. `#paid-media`)
- ❌ Google Drive contains a "strategy doc" at a known path
- ❌ Email is through Gmail
- ❌ EOS is the planning methodology
- ❌ There is exactly one CEO / COO / Integrator

Everything must be discovered from context (`BUSINESS_CONTEXT.md` + live MCP detection + AskUserQuestion).

---

## Cross-platform notes

- **Claude Code** — richest MCP ecosystem because users can wire any local stdio / HTTP MCP server. Expect `mcp__<arbitrary-id>__<tool>` naming. `BUSINESS_CONTEXT.md` `connected_tools` is a useful hint because server-ids aren't self-describing.
- **Claude Desktop** — managed connector catalog via the `mcp__claude_ai_<Service>__<tool>` prefix plus whatever custom remote MCP servers the user has added. The official connector catalog is growing; skills should not hardcode "Desktop has fewer tools" as an assumption. Use live detection.
- **Cowork** — MCP availability depends on the workspace's connected services. Detection works the same way; the `available_tools` set is just different per workspace.

Skills must behave identically across platforms at the **intent** layer. The only thing that changes is *which* specific connector or MCP delivers the output, and that's resolved at runtime by the normalized catalog lookup above.

### Verifying at session start

The fastest way for a skill to sanity-check tool availability is to inspect the tool registry at the moment the skill runs. Concretely:

1. Scan the available tool names for the `mcp__` prefix.
2. Split on `__`, take the middle segment, apply the normalization rule.
3. Build a `Set<canonical_name>` once; reuse for every category check during the session.

Do NOT cache this across sessions — connectors get added and removed between sessions, and a stale cache will produce silent failures.
