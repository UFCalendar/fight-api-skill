# Connect an MCP client

Endpoint: `https://api.ufcalendar.com/mcp`
Transport: stateless Streamable HTTP. POST JSON-RPC only; GET answers 405.
Send `Accept: application/json, text/event-stream` — the transport answers 406
without it.

Auth, either door:

- an API key as `Authorization: Bearer <key>`, or
- an OAuth 2.1 sign-in with a UFCalendar account (what the hosted connectors
  below use). Calling a gated tool unauthenticated answers 401 with
  `WWW-Authenticate: Bearer resource_metadata="https://api.ufcalendar.com/.well-known/oauth-protected-resource"`,
  which is the whole discovery chain.

Metering: 1 tool call = 1 metered request against the same plan quota as REST.
`get_plans`, `list_orgs` and `how_to_connect` need no credential and cost
nothing.

## Claude Code

```bash
claude mcp add --transport http ufcalendar https://api.ufcalendar.com/mcp --header "Authorization: Bearer $UFCALENDAR_API_KEY"
```

## Cursor (`~/.cursor/mcp.json`)

```json
{"mcpServers":{"ufcalendar":{"url":"https://api.ufcalendar.com/mcp","headers":{"Authorization":"Bearer $UFCALENDAR_API_KEY"}}}}
```

## Codex (`~/.codex/config.toml`)

```toml
[mcp_servers.ufcalendar]
url = "https://api.ufcalendar.com/mcp"
http_headers = { Authorization = "Bearer $UFCALENDAR_API_KEY" }
```

## Claude.ai (custom connector)

Settings → Connectors → Add custom connector, paste
`https://api.ufcalendar.com/mcp`, then sign in with your UFCalendar account.
No key to copy — first sign-in starts the free 1-day trial when the account has
never held a plan.

## ChatGPT (developer mode)

Enable developer mode, add `https://api.ufcalendar.com/mcp` as a connector and
sign in. Every read tool is annotated read-only and closed-world.

## Tools

48 tools. Three are free and need no credential (`get_plans`, `list_orgs`,
`how_to_connect`); the rest map onto the REST API (see `endpoints.md` for
what each returns). The four webhook tools and
`get_odds_history` need Pro or above. Three of them write (`create_webhook_endpoint`, `rotate_webhook_secret`,
`delete_webhook_endpoint`) and are annotated accordingly — rotate and delete as
destructive; ask before calling them. `list_webhook_endpoints` is read-only.
