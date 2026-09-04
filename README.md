[![skills.sh](https://skills.sh/b/PropertyMe/ai-skills)](https://skills.sh/PropertyMe/ai-skills)

# PropertyMe AI Skills

Agent skills for working with the [PropertyMe](https://propertyme.com) MCP server. PropertyMe is a property management platform; its MCP server exposes properties, contacts, jobs, tasks, inspections, lease renewals, portfolios, and team members as tools that an AI agent can call.

This repository currently contains one skill:

- **[propertyme-mcp](./skills/propertyme-mcp/SKILL.md)** — Teaches an agent how to use the PropertyMe MCP server's tools effectively: session and portfolio management, domain terminology, search-vs-get workflows, pagination, field connections, and presentation rules.

## Install

Install the skill into your agent with the [skills](https://skills.sh) CLI:

```
npx skills add PropertyMe/ai-skills
```

## Prerequisites

The skill assumes a PropertyMe MCP server is configured and available to your agent.

If `list_portfolios` shows no portfolios, or the active portfolio isn't listed, MCP access hasn't been enabled for the account.

## What the skill covers

- **Sessions & portfolios** — `create_session`, `list_portfolios`, `set_portfolio`
- **Domain glossary** — Portfolio, Property, Contact, Job, Task, Inspection, Lease renewal, Team member (and the internal terms never to show the user)
- **All 22 tools** — grouped by domain with what each does
- **Key workflows** — verify active portfolio first, resolve "my" queries via `get_current_user`, search-before-get, pagination, and offering to switch to MCP-enabled portfolios
- **Common requests** — a table mapping user phrasings to the tools to call
- **Rate limiting** — cost-weighted limits, batching guidance, and avoiding unnecessary calls

## License

[MIT](./LICENSE) © PropertyMe
