# UFCalendar Fight API — agent skill

An installable skill that teaches a coding agent how to reach the
[UFCalendar Fight API](https://www.ufcalendar.com/developers) — the MMA API and
UFC API behind our [UFC MCP server](https://www.ufcalendar.com/developers/ufc-mcp-server)
and [MMA MCP server](https://www.ufcalendar.com/developers/mma-mcp-server): UFC, PFL,
OKTAGON, BKFC and RIZIN events, full fight cards, results, per-round stats,
fighter careers, judges' scorecards, UFC rankings history since 2013 and
per-country broadcast rights — over REST, the two SDKs, or the MCP server at
`https://api.ufcalendar.com/mcp`.

## Install

```bash
npx skills add UFCalendar/fight-api-skill
```

Or copy it in by hand:

| Agent | Where |
|---|---|
| Claude Code | `.claude/skills/ufcalendar-fight-api/` (project) or `~/.claude/skills/ufcalendar-fight-api/` (global) |
| Cursor | paste `SKILL.md` into a rule under `.cursor/rules/`, keep `references/` beside it |
| Codex | reference `SKILL.md` from `AGENTS.md`, keep `references/` beside it |

The skill is plain Markdown — `SKILL.md` plus four reference files. Nothing
executes.

## Contents

- `SKILL.md` — access in 60 seconds, base URL and envelope, the endpoints that
  answer the common questions, the don'ts, and when to use MCP vs REST vs SDK.
- `references/endpoints.md` — every endpoint and parameter. **Generated** from
  the OpenAPI document; do not edit by hand.
- `references/conventions.md` — envelope, pagination, errors, rate headers,
  tier gates, webhook signature verification.
- `references/mcp-clients.md` — copy-paste config for Claude Code, Cursor,
  Codex, Claude.ai and ChatGPT.
- `references/sdk.md` — Python and TypeScript quickstarts.

## Access

No free tier. A **free 1-day trial** (100 requests, no card, one per account)
self-serves at <https://www.ufcalendar.com/account/api?trial=1>; paid plans
start at $19/month. `GET https://api.ufcalendar.com/v1/plans` answers with no
credential at all, so an agent can evaluate the API before anyone signs in.

Odds are the UFCalendar consensus line only (no book identities; information
only, not betting advice). Fighter images are Creative Commons and
must be shown with their credit. Not affiliated with UFC, Zuffa, TKO or any
promotion.

MIT licensed.
