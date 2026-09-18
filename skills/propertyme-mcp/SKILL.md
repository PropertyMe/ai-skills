---
name: propertyme-mcp
description: >-
  PropertyMe MCP server — how to use its tools effectively as a property management assistant.
  Use this skill whenever the user mentions PropertyMe, property management, portfolios,
  properties, contacts, jobs, tasks, inspections, lease renewals, or team members in the context
  of the PropertyMe MCP server. Covers domain terminology, tool workflows, presentation rules,
  and entity relationships. Trigger even if the user doesn't explicitly say "MCP" — most
  questions about their PropertyMe data need this guidance.
---

# PropertyMe MCP

PropertyMe's MCP server exposes property management data (properties, contacts, jobs, tasks,
inspections, lease renewals, portfolios, team members) as tools that can be called. This skill
covers **using the tools well**.

## Creating a session

Every session must start with `create_session`. It returns a `sessionId` that must be carried on
every subsequent tool call as the `Mcp-Param-SessionId` header. The session is created against
the user's default portfolio; call `set_portfolio` to switch to a different one.

## Switching portfolios

If you belong to multiple property management businesses (portfolios), use `set_portfolio` to
switch the active context. All subsequent tool calls operate against the selected portfolio's
data. Check which is active with `list_portfolios` — the one with `IsCurrent = true` is active.

## Domain glossary

Speak the user's language. Never use internal field names.

| Term | What it means | Do not say |
|---|---|---|
| **Portfolio** | A property management agency or business entity | "customer" |
| **Property** | A managed property (residential or commercial) | "lot" |
| **Contact** | A person or organisation — owner, tenant, supplier, or agency | — |
| **Job** | A maintenance or repair work order against a property | — |
| **Task** | A to-do item (not a job — tasks are lighter, no supplier/access workflow) | — |
| **Inspection** | A property inspection (Routine, Entry, Exit, or Open) | — |
| **Lease renewal** | The workflow of renewing a tenancy agreement (owner + tenant sign-off) | — |
| **Team member** | A staff user of the property management business (Admin, Standard, Limited, Read-only) | "member" alone |

### Key workflows

**1. Always verify the active portfolio first.**

Before querying data, confirm which portfolio the session is operating in. Call `list_portfolios`
and check which one has `IsCurrent = true`. If the user names a specific portfolio, verify it's
the active one — if not, offer to switch with `set_portfolio`.

**2. "My" queries — resolve the current user first.**

When the user says "my tasks," "jobs assigned to me," or "my inspections," call `get_current_user`
first to get their `MemberId`, then match it against the `ManagerMemberId` field on the returned
records.

```
User: "What jobs are assigned to me?"
  1. get_current_user  → MemberId = <guid>
  2. list_jobs         → filter results where ManagerMemberId == MemberId
  3. Present matching jobs (address, summary, priority, due date, status)
```

**3. Search before get.**

Most list/search tools return summary records (id + key fields). Use search to find what the user
is looking for, then `get_*` with the id for full details. Search returns max 2000 results — if
there are more, ask the user to refine their search. When the user wants a report (e.g. "all
properties", "all jobs"), use `list_*` and page through results instead of `search_*` — search is
for finding a specific record, not enumerating datasets.

```
User: "Show me the details for the Smith property"
  1. search_properties("Smith") → get the property id
  2. get_property(id)           → full details (address, bedrooms, bathrooms, etc.)
```

**4. Pagination.**

All list tools use `offset` (default 0) and `limit` (default 2000, maximum 2000). Page through by
incrementing offset by 2000. If search returns exactly 2000 results, there may be more — suggest the
user narrow their search.

**5. Offer to switch to a portfolio that supports MCP.**

If `list_portfolios` shows the user's active portfolio isn't MCP-enabled (MCP isn't available for
it), or they belong to another MCP-enabled portfolio, offer to switch with `set_portfolio`. When
running a report across multiple portfolios, do not switch to a portfolio that does not support
MCP — only switch between MCP-enabled portfolios.

### Key field connections

The following field connections are used internally (never displayed to the user):
- `ManagerMemberId` on jobs and tasks → matches `MemberId` from `get_current_user` and `Id` from
  `list_team_members` (the assigned team member)
- `LotId` on jobs, tasks, and inspections → the property id (from `list_properties` / `get_property`)
- `ContactId` → the contact id (from `list_contacts` / `search_contacts`)
- `LotReference` on lease renewals → the property reference

### Presentation rules

These are non-negotiable — the user should never see raw field names or internal identifiers.

**Never show GUIDs.** Use ids internally to call follow-up tools, but present entities by name,
address, or reference to the user. If you must reference something, use its human-readable label.

**Date format:** `dd/MM/yyyy` for dates (e.g. 11/12/2024), `dd/MM/yyyy HH:mm` for date-times.

**Priority:** Show the label, never the number. Jobs use: Low, Medium, High, Urgent. Tasks use:
Low, Medium, High (Urgent is jobs only — tasks only go to High).

**Status translations:**
| Internal | Show as |
|---|---|
| `inprogress` | "in progress" |
| `ReScheduled` | "rescheduled" |
| `ReadOnly` | "read-only" |
| `todo` (lease renewals) | "not started" |
| `pending` (lease renewals) | "awaiting signatures" |

**Job lifecycle:** Reported → Quoted → Approved → Assigned → Finished → Closed (or Rejected/Cancelled).

**Lease renewal workflow status:** todo (not started) → inprogress (being worked on) → pending
(awaiting owner/tenant signatures) → closed (finished). `active` means everything not closed.

**Lease renewal milestones:** The `States` list in a lease renewal tracks per-party progress
(intent → agreed → signed) with dates. Use it to tell the user where each party is in the process
— the top-level owner/tenant flags don't carry dates.

### Common user requests and how to handle them

| User says | Tools to call |
|---|---|
| "What properties do we manage?" | `list_properties` |
| "Show me the Smith property" | `search_properties("Smith")` → `get_property(id)` |
| "What jobs are assigned to me?" | `get_current_user` → `list_jobs`, filter by ManagerMemberId |
| "What inspections are coming up?" | `list_inspections` (filter by status if needed) |
| "Show me pending lease renewals" | `list_lease_renewals(status="pending")` |
| "Who's on the team?" | `list_team_members` |
| "Switch to my other portfolio" | `list_portfolios` → `set_portfolio(id)` |
| "What's the status of the lease renewal for 123 Main St?" | `search_properties("123 Main")` → `list_lease_renewals` → `get_lease_renewal(id)` |

### Rate limiting

The server enforces rate limits. If you hit a rate limit error, 
tell the user and wait — don't retry immediately. Avoid unnecessary calls: prefer
`search` over repeated `list` + manual filtering. When retrieving large amounts of data, batch
calls in groups of 5, then wait 5 seconds before issuing the next batch.

### When MCP is not enabled for a portfolio

If `list_portfolios` shows no portfolios or the user's active portfolio isn't listed, MCP access
hasn't been enabled for their account. Tell them to contact their PropertyMe administrator to
enable MCP for their portfolio.
