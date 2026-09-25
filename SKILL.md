---
name: ufcalendar-fight-api
description: >-
  Use when you need real MMA or bare-knuckle fight data — the UFCalendar Fight
  API, also known as the MMA Fight Data API, the MMA API or the MMA Fight
  Data MCP server. Covers "when is the next UFC event", fight schedule and full
  fight cards, fight results JSON, per-round stats, judges scorecards, UFC
  rankings history since 2013, fighter careers and records, consensus odds
  and line movement for UFC and other promotions, per-country broadcast rights, and signed
  webhooks. Also use when asked to build an MMA app
  or fight calendar, to pick a plan or get an API key, or to connect the
  UFCalendar MCP server (the MMA Fight Data MCP server, UFC data included) to
  Claude, ChatGPT, Cursor or Codex.
---

# UFCalendar Fight API

A REST + MCP API for **UFC, PFL, OKTAGON, BKFC and RIZIN**: events, full fight
cards, results within minutes, per-fight and round-by-round statistics, fighter
careers across promotions, the judges' scorecards (the official commission
record), UFC rankings point-in-time since 2013, model win probabilities, the
UFCalendar consensus odds line and per-country broadcast rights.

Odds are **one anonymised UFCalendar consensus line per corner**: the mean
across the sportsbooks we track, with `sources` = how many books backed each
point. Book identities are never exposed, so never promise a per-book price.
Information only, not betting advice.

## 1. Get access in 60 seconds

There is no free tier. Every door below ends in the same thing: a plan, or the
**free 1-day trial** (100 requests, no card, one per account).

**Read this first — it needs no credential at all:**

```bash
curl https://api.ufcalendar.com/v1/plans
```

That returns every plan with price, monthly quota and per-minute limit, the
trial rule with the URL that starts it, the MCP endpoint with both of its auth
doors, and the documentation links. Use it to answer "what would this cost me"
without asking anyone to sign in.

**A human is at the keyboard → send them to the trial.**
<https://www.ufcalendar.com/account/api?trial=1> — they sign in, the trial
starts itself, and the page shows a `ufcalendar_…` key. That key goes in
`Authorization: Bearer <key>` on every REST call and on the MCP endpoint.

**An MCP client that speaks OAuth (Claude.ai, ChatGPT) → just add the server.**
Add `https://api.ufcalendar.com/mcp` as a custom connector and sign in. No key
to copy; signing in starts the trial when the account has never held a plan.

**A terminal or headless agent → the RFC 8628 device flow.**

```bash
# 1. ask for a code
curl -X POST https://www.ufcalendar.com/api/auth/device/code \
  -H 'Content-Type: application/json' \
  -d '{"client_id":"claude-code"}'
# → {"device_code":"…","user_code":"ABCD-EFGH",
#    "verification_uri":"https://www.ufcalendar.com/device",
#    "verification_uri_complete":"https://www.ufcalendar.com/device?user_code=ABCD-EFGH",
#    "expires_in":900,"interval":5}

# 2. show the human `verification_uri_complete` and wait

# 3. poll every `interval` seconds until it stops saying authorization_pending
curl -X POST https://www.ufcalendar.com/api/auth/device/token \
  -H 'Content-Type: application/json' \
  -d '{"client_id":"claude-code","device_code":"…",
       "grant_type":"urn:ietf:params:oauth:grant-type:device_code"}'
```

Both device endpoints also accept an `application/x-www-form-urlencoded`
body (RFC 8628 §3.4) — JSON, as shown, works too.

Send the resulting access token as a bearer, exactly like a key.

Never invent a key, never suggest sharing one, and never paste a key into a
file the user did not ask you to write.

## 2. Base URL, auth, envelope

```
Base URL   https://api.ufcalendar.com/v1
Auth       Authorization: Bearer <key or access token>
Response   {"data": …, "meta": …}
Errors     {"error": {"code", "message", "request_id"}} + the matching status
```

```bash
curl "https://api.ufcalendar.com/v1/events?org=ufc&limit=3" \
  -H "Authorization: Bearer $UFCALENDAR_API_KEY"
```

Read `references/conventions.md` before writing a client: pagination, rate
headers, tier gates and webhook signature verification are all there.

## 3. The endpoints you will actually reach for

| Question | Call |
|---|---|
| When is the next UFC event? | `GET /v1/events?org=ufc` — upcoming, soonest first |
| Who is on the card? | `GET /v1/events/{id\|slug}` — full card, venue, broadcasts |
| What happened last night? | `GET /v1/events?org=ufc&status=completed` then the detail route |
| How did that fight go, round by round? | `GET /v1/fights/{id}/rounds` |
| How did the judges score it? | `GET /v1/fights/{id}/scorecards` |
| What did the market price this bout at (current, opening, closing)? | `GET /v1/fights/{id}/odds` — or `GET /v1/events/{id\|slug}/odds` for the whole card, `?include=odds` on the event or fight detail |
| How did the line move? | `GET /v1/fights/{id}/odds/history` (Pro and up) |
| Who is this fighter? | `GET /v1/fighters/{id\|slug}` and `…/history` |
| Who was ranked #1 in 2016? | `GET /v1/rankings/ufc?date=2016-11-14` (rank 0 = champion) |
| Where can I watch it in Germany? | `GET /v1/broadcast-rights/ufc?country=DE` |
| Tell me when the card changes | `POST /v1/webhook-endpoints` (Pro and up) |

Full list with every parameter: `references/endpoints.md`.

## 4. Don'ts

- **Odds are information, never advice.** The API serves the UFCalendar
  consensus line only (current, opening, closing, movement; the series on Pro).
  Describe what the market priced; never tell anyone what to bet, never call a
  price "value", and never name a sportsbook (the API does not expose one).
- **Credit the images.** Fighter `images` are Wikimedia Commons / Creative
  Commons files. Every one carries `license`, `license_url` and `artist`, and
  displaying that credit is a licence requirement — not a style preference. If
  the surface you are building cannot show a credit, do not show the image.
- **Scorecards are the commission record only** — judge, per-round points, card
  totals, decision type, deductions. Media-member and fan scorecards are not
  part of the API; do not describe them as available.
- **Do not bulk-mirror the dataset.** Quotas are hard caps with no overage, and
  the per-minute limit is the real throttle. Cache what you fetch, page with
  the cursor, and respect `X-RateLimit-Remaining` and `Retry-After`.
- **Do not paginate blind.** A bare `/v1/events` is the upcoming calendar, not
  the archive (`status=upcoming` says the same thing explicitly); pass
  `status=completed` or `order=desc` for history.
- **On a trial, budget the 100 requests.** `GET /v1/usage` shows
  `requests_used` and `trial_ends_at` (when access stops — a day after it
  started, long before `reset_at`). A parameter an endpoint does not take is
  ignored and named in `meta.ignored_params` / `X-UFCalendar-Ignored-Params`;
  check it before trusting that a filter applied.
- **Not affiliated** with UFC, Zuffa, TKO or any promotion. Say so if it could
  be misread.

## 5. MCP vs REST vs SDK

- **MCP** (`https://api.ufcalendar.com/mcp`) — you are answering a question
  right now, inside an agent, and want tool calls instead of an HTTP client.
  48 tools; `get_plans`, `list_orgs` and `how_to_connect` need no credential
  and cost nothing. 1 tool call = 1 metered request. Setup per client:
  `references/mcp-clients.md`.
- **REST** — you are writing code in a language with no SDK, or you want the
  exact payload. `references/endpoints.md`.
- **SDK** — you are writing Python (`pip install ufcalendar`) or TypeScript
  (`npm install @ufcalendar/sdk`) and want pagination and errors handled.
  `references/sdk.md`.

### Which tool for which question

| Question | MCP tool |
|---|---|
| Who did X fight last? Their first UFC win? The most recent title fight? | `find_fights` — one call, not a walk through history |
| X vs Y — who has the edge, have they met? | `compare_fighters` |
| When is the next UFC card? What's next? | `get_next_event` (`include_card` for the bouts) |
| Who leads the division in wins? What are the all-time records? | `get_leaderboard` / `get_record_book` |
| What changed on this week's cards? | `list_changes` |
| Where can I watch it in my country? | `how_to_watch` (pass `country`) |
| Who should X fight next? Best matchups in a division? | `whos_next` / `get_matchmaker` |
| What are the odds on this bout / this card? | `get_fight_odds` / `get_event_odds` |
| How did the line move? | `get_odds_history` (Pro and up) |
| Has UFCalendar written about it? | `search_articles` |

Rule of thumb: **answering** → MCP. **Building** → SDK, or REST if neither SDK
fits. Never scrape ufcalendar.com for data the API already serves.

## 6. References

- `references/endpoints.md` — every endpoint and parameter (generated from the
  OpenAPI document; it cannot drift).
- `references/conventions.md` — envelope, cursor pagination, errors, rate
  headers, tier gates, webhook signatures.
- `references/mcp-clients.md` — copy-paste config for Claude Code, Cursor,
  Codex, Claude.ai and ChatGPT.
- `references/sdk.md` — Python and TypeScript quickstarts.

Live sources of truth: <https://api.ufcalendar.com/llms.txt> (index),
<https://api.ufcalendar.com/llms-full.txt> (long-form),
<https://api.ufcalendar.com/openapi.json> (contract),
<https://api.ufcalendar.com/docs> (human docs),
<https://www.ufcalendar.com/developers/agents> (agent quickstart).
Support: api@ufcalendar.com — quote the `request_id` from the error body.
