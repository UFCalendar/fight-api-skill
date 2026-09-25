# Conventions

Everything here applies to every endpoint. Copied from the API's own
`llms.txt`/`llms-full.txt` — if the two ever disagree, the live file wins.

## Shape

- JSON, `snake_case`, always wrapped: `{"data": …, "meta": …}`.
- Errors: `{"error": {"code", "message", "request_id"}}` with the matching HTTP
  status. Every response also carries `x-request-id`. Quote it in support mail
  (api@ufcalendar.com) — it identifies the exact logged request.
- Timestamps are UTC ISO-8601. Events additionally carry the venue's IANA `tz`,
  so you can render local time without guessing.

## Pagination

List endpoints are cursor-paginated:

```
GET /v1/events?org=ufc&limit=100
→ meta.pagination = {"next_cursor": "eyJ…", "has_more": true}
GET /v1/events?org=ufc&limit=100&cursor=eyJ…
```

`limit` maxes out at 100. Stop when `next_cursor` is null — never guess a
cursor, and never page past what you need.

## Rate limits and quotas

- `X-RateLimit-Limit` / `-Remaining` / `-Reset` describe the **monthly** quota.
- The per-minute limit is separate and is the real anti-scraping throttle.
- A 429 carries `Retry-After`. Honour it; do not retry in a tight loop.
- Quotas are hard caps with no overage billing: past the cap, calls fail.

## Tier gates

- Webhook endpoints and odds
  history (`GET /v1/fights/{id}/odds/history`) need **Pro or above**.
- Bulk snapshots, including the `odds-closing.json.gz` closing-line archive,
  are **Enterprise only** (contract required — api@ufcalendar.com).
- Everything else, ICS feeds included, is open to any active plan.
- A key below the required tier gets `403 tier_required`.

## Thin lists, rich details

List endpoints return identity fields only; the full record lives on the detail
route. `/v1/events` omits the card, `/v1/fighters` omits the physical bio, and
there is **no `/v1/fights` list** — fetch a bout by id or read it off the event.
That split is deliberate, so plan one detail call per entity you actually need.

## Webhooks

- Deliveries do **not** count against your monthly quota; only the calls that
  register, list, rotate or delete endpoints do.
- Delivery is at-least-once. Verify `X-UFCalendar-Signature` — HMAC-SHA256 over
  `"<t>.<rawBody>"` with the endpoint secret — **and reject a `t` older than
  300 seconds**. Deduplicate on the body and answer 2xx fast.
- We retry about once a minute until you answer 2xx; 20 consecutive failures
  auto-disable the endpoint.
- Event types: `event.announced`, `fight.result`, `card.changed`,
  `event.completed`, `odds.moved` (the UFCalendar consensus line on an upcoming
  bout moved 5+ implied-probability points on corner a, or the favourite
  flipped — measured against the last line delivered, so a slow drift arrives
  once; information only, not betting advice).

## Hard rules about the data

- **Odds are the UFCalendar consensus line only**: one line per corner across
  the sportsbooks we track, `sources` = how many books backed each point, book
  identities never exposed. Current, opening and closing ship on every plan;
  the movement series is Pro and up. Information only, not betting advice —
  describe what the market priced, never what to bet.
- **Scorecards are the official commission record** — judge identity, per-round
  points, card totals, decision type, point deductions. Media-member scorecards
  and fan scoring are a third party's compilation and are not part of the API.
  Check `scores_known` on a card before charting totals: when it is false the
  commission published only the outcome, so totals are a placeholder and
  `rounds` is empty.
- **`source` / `source_slug` name official publishers only** — `ufc-stats`,
  `ufc-official`, `ufc-official-cards`, a promotion's `*-official`,
  `wikipedia`, `ufcalendar`. Every other publisher is `public-records` (career
  records, history rows) or `commission-record` (judges' scorecards). Quote
  those labels as-is; do not guess which website stands behind them.
- **Fighter images are Wikimedia Commons / Creative Commons only** and carry
  `license`, `license_url` and `artist`. Displaying that credit is a licence
  requirement.
- Not affiliated with UFC, Zuffa, TKO or any promotion.
