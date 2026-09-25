# UFCalendar Fight API — every endpoint

GENERATED from the OpenAPI document by `sdk/skill/scripts/gen-endpoints.ts`.
Do not edit by hand.

Base URL: https://api.ufcalendar.com/v1 · auth: `Authorization: Bearer <key>`
Machine contract: https://api.ufcalendar.com/openapi.json · long-form reference: https://api.ufcalendar.com/llms-full.txt · human docs: https://api.ufcalendar.com/docs

## Discovery

### GET /v1/plans (no key required)
Plans, quotas and how to connect (no key required)
The only endpoint that answers without a credential.

## Orgs

### GET /v1/orgs
List launch orgs with capability flags
Each org carries flags (stats, rounds, rankings, broadcasts, predictions, scorecards, odds) so clients discover coverage programmatically — plus sport (mma, bare-knuckle-boxing) and country_code (ISO-2).

### GET /v1/orgs/{slug}
One org
Single org by slug.

Parameters:
  - `slug` (path, required)

### GET /v1/orgs/{slug}/divisions/{division}
One division in one promotion
One weight class in one promotion, in one call: division (slug, canonical name, weight_limit_lbs — null where no standard limit applies), rankings (the promotion's current official board for this division — same object as /v1/rankings/{org}?division= — or null when it publishes none), upcoming (up to 50 bouts booked at this weight, soonest first), recent (the 15 latest completed or no-contest bouts, newest first) and roster (up to 30 fighters who have fought at this weight, most recent first, each with last_fought_at).

Parameters:
  - `slug` (path, required) — one of `ufc`, `pfl`, `oktagon`, `bkfc`, `rizin`
  - `division` (path, required) — one of `strawweight`, `flyweight`, `bantamweight`, `featherweight`, `lightweight`, `welterweight`, `super-welterweight`, `middleweight`, `light-heavyweight`, `heavyweight`, `super-heavyweight`, `womens-strawweight`, `womens-flyweight`, `womens-bantamweight`, `womens-featherweight`, `atomweight`, `ironweight`

## Events

### GET /v1/events
List events (schedule + results)
A bare listing is the upcoming calendar: events from the last ~6h onward, soonest first.

Parameters:
  - `org` (query) — one of `ufc`, `pfl`, `oktagon`, `bkfc`, `rizin`
  - `status` (query) — Filter on lifecycle status; `postponed` is accepted (2026-09-24). `upcoming` = everything not yet over (announced, scheduled or live), soonest first.
  - `from` (query) — Inclusive `YYYY-MM-DD`; must be a real calendar day.
  - `to` (query) — Inclusive `YYYY-MM-DD`; must be a real calendar day.
  - `order` (query) — one of `asc`, `desc`
  - `is_title_card` (query) — `true` = only cards with a title bout, `false` = only cards without one.
  - `is_ppv` (query) — `true` = only pay-per-view cards, `false` = only non-PPV cards.
  - `include` (query) — one of `headline`
  - `cursor` (query) — Opaque cursor from `meta.pagination.next_cursor`.
  - `limit` (query)

### GET /v1/events/{idOrSlug}
Event with full card
Full fight card (both corners, results when completed), venue and event-scoped broadcast rows.

Parameters:
  - `idOrSlug` (path, required) — Numeric id or slug of the event. Stale slugs 308-redirect to the canonical URL.
  - `include` (query) — one of `eta`, `odds`

### GET /v1/events/{idOrSlug}/changes
Card-change log
The diff log behind "card updated": fight added/removed, opponent swapped, card order or date moved, or a fighter profile merged (fighter-merged: two of our fighter ids turned out to be one athlete, so a live bout now carries the surviving id — same humans, new ids).

Parameters:
  - `idOrSlug` (path, required) — Numeric id or slug of the event. Stale slugs 308-redirect to the canonical URL.

### GET /v1/events/{idOrSlug}/storylines
Storylines of one card
The talking points of one card, computed from the record — the pre-fight read, as of the card's start (a completed card reads as it did walking in).

Parameters:
  - `idOrSlug` (path, required) — Numeric id or slug of the event. Stale slugs 308-redirect to the canonical URL.

### GET /v1/events/{idOrSlug}/pickem
Pick'em splits for one card
How the UFCalendar community is picking each bout on one card, from the site's own pick'em game: fights[] in card order (non-cancelled bouts), each with fight_id, both corners (fighter_a, fighter_b: id, slug, name), picks_a, picks_b, total and pct_a (the percentage of picks on corner a, 0–100 with one decimal; null when nobody has picked the bout).

Parameters:
  - `idOrSlug` (path, required) — Numeric id or slug of the event. Stale slugs 308-redirect to the canonical URL.

### GET /v1/changes
Card-change feed across events
Every card change across the launch promotions (or one of them), newest first — the same audit trail the card.changed webhook delivers, for callers that poll instead.

Parameters:
  - `org` (query) — one of `ufc`, `pfl`, `oktagon`, `bkfc`, `rizin`
  - `since` (query) — Only changes observed at or after this point: `YYYY-MM-DD` (UTC midnight) or an ISO-8601 datetime. Default: 90 days ago.
  - `kind` (query) — one of `fight-added`, `fight-cancelled`, `fight-reinstated`, `opponent-changed`, `time-changed`, `venue-changed`, `fighter-merged`
  - `cursor` (query) — Opaque cursor from `meta.pagination.next_cursor`.
  - `limit` (query)

### GET /v1/venues
Search venues
Arenas and venues that have hosted (or have booked) a launch-org event, alphabetical.

Parameters:
  - `q` (query) — Venue name or city fragment.
  - `country` (query) — ISO-2 code or English country name.
  - `cursor` (query) — Opaque cursor from `meta.pagination.next_cursor`.
  - `limit` (query)

### GET /v1/venues/{id}/events
Events at a venue
Every launch-org event held (or booked) at one venue, newest first — upcoming cards on top, then the history.

Parameters:
  - `id` (path, required)
  - `status` (query) — Same values as `/v1/events`; `upcoming` = the venue's cards not yet over.
  - `from` (query)
  - `to` (query)
  - `order` (query) — one of `asc`, `desc`
  - `cursor` (query) — Opaque cursor from `meta.pagination.next_cursor`.
  - `limit` (query)

### GET /v1/venues/{id}
Venue
Venue with city, region, country (+ country_code), coordinates, capacity and IANA tz.

Parameters:
  - `id` (path, required)

### GET /v1/calendar/{org}.ics
ICS calendar feed
Subscribe your calendar app to an org's schedule.

Parameters:
  - `org` (path, required) — `ufc`, `pfl`, `oktagon`, `bkfc` or `rizin`; append `-sections` or `-fights` for the per-section or per-bout shape (e.g. `ufc-fights.ics`)
  - `key` (query, required)

## Broadcast

### GET /v1/events/{idOrSlug}/watch
How to watch one event
Who airs one event, country by country: the promotion's standing rights deals merged with the event's own confirmed broadcast listings.

Parameters:
  - `idOrSlug` (path, required) — Numeric id or slug of the event. Stale slugs 308-redirect to the canonical URL.
  - `country` (query) — ISO-3166 alpha-2 code; adds the worldwide row.

### GET /v1/broadcast-rights/{org}
Who airs it, per country
Org-level standing media rights per country (PFL harvested weekly across 212 countries; UFC ~110 curated).

Parameters:
  - `org` (path, required)
  - `country` (query) — ISO-2 country code.
  - `series` (query) — one of `dwcs`, `rtufc`

## Odds

### GET /v1/events/{idOrSlug}/odds
Consensus odds for one card
The UFCalendar consensus line for every non-cancelled bout on one card, in card order: fights[], each with fight_id, status, both corners (fighter_a, fighter_b: id, slug, name), consensus (the latest point; the closing line once the bout is settled), opening (the first point we recorded), closing (settled bouts only: the last point at or before the event start), movement (opening → consensus in implied-probability points on corner a, with the direction the market moved), points and updated_at.

Parameters:
  - `idOrSlug` (path, required) — Numeric id or slug of the event. Stale slugs 308-redirect to the canonical URL.

### GET /v1/fights/{id}/odds
Consensus odds for one bout
The UFCalendar consensus line for one bout: consensus (the latest point; the closing line once the bout is settled), opening (the first point we recorded), closing (settled bouts only, completed or no_contest: the last point at or before the event start; null until then), movement (opening → consensus: delta_points_a in implied-probability points on corner a, direction = the corner the market moved toward, since), points (stored points) and updated_at (when the line last moved).

Parameters:
  - `id` (path, required)

### GET /v1/fights/{id}/odds/history
Consensus line movement (Pro+)
Every stored consensus point for one bout, oldest first: the line-movement series.

Parameters:
  - `id` (path, required)
  - `from` (query) — Earliest recorded day, inclusive `YYYY-MM-DD`.
  - `to` (query) — Latest recorded day, inclusive `YYYY-MM-DD`; must not precede `from`.
  - `cursor` (query) — Opaque cursor from `meta.pagination.next_cursor`.
  - `limit` (query)

## Fights

### GET /v1/fights/search
Search completed bouts (filtered)
Completed bouts with a winner across the launch promotions, filtered — the one-call answer to "the last title fight to end by knockout" or "every submission win for this fighter".

Parameters:
  - `org` (query) — one of `ufc`, `pfl`, `oktagon`, `bkfc`, `rizin`
  - `title_only` (query) — `true` = title bouts only.
  - `method` (query) — one of `ko`, `sub`, `dec`, `finish`, `any`
  - `division` (query) — Division slug (`lightweight`, `womens-strawweight`, …) for an exact match, or a name fragment (≤40 characters) matched anywhere in the weight class.
  - `fighter` (query) — Fighter slug or id — bouts this fighter was in, either corner.
  - `winner` (query) — Fighter slug or id — bouts this fighter won.
  - `from` (query) — Earliest event date, YYYY-MM-DD.
  - `to` (query) — Latest event date, YYYY-MM-DD.
  - `main_events_only` (query) — `true` = main events only.
  - `order` (query) — one of `newest`, `oldest`
  - `cursor` (query) — Opaque cursor from `meta.pagination.next_cursor`; an unreadable cursor restarts at the first page.
  - `limit` (query)

### GET /v1/fights/{id}
One bout
Bout with corners (FighterRef with country_code), sport (bout override, else the org sport), weight_class + division_slug, result (raw method plus method_normalized / finish_detail / round / time / time_seconds / referee) and bonus flags, plus a compact event.

Parameters:
  - `id` (path, required)
  - `include` (query) — one of `odds`

### GET /v1/fights/{id}/stats
Per-fight totals
Both corners: knockdowns, significant/total strikes, takedowns, submission attempts, reversals, control time, and target/position splits (head/body/leg, distance/clinch/ground) where the source provides them.

Parameters:
  - `id` (path, required)

### GET /v1/fights/{id}/rounds
Round-by-round stats
Per-round stat lines for both corners — each with round (and the stored round_number twin) and corner — including target/position splits (head/body/leg, distance/clinch/ground) where they are logged per round (BKFC for head/body/distance/clinch; other orgs may additionally carry leg/ground).

Parameters:
  - `id` (path, required)

## Scorecards

### GET /v1/fights/{id}/scorecards
Judges' scorecards for a bout
The official athletic-commission record: every judge, their score for each round, their card total, the decision type and any point deductions.

Parameters:
  - `id` (path, required)

### GET /v1/scorecards/splits
Split and majority decisions
Completed bouts the judges decided on a split or majority (including majority and split draws), newest first, across the launch promotions or one of them, optionally in a date range.

Parameters:
  - `org` (query) — one of `ufc`, `pfl`, `oktagon`, `bkfc`, `rizin`
  - `from` (query) — Earliest event date, YYYY-MM-DD.
  - `to` (query) — Latest event date, YYYY-MM-DD.
  - `cursor` (query) — Opaque cursor from `meta.pagination.next_cursor`.
  - `limit` (query)

### GET /v1/judges
Judge directory
Every official who has scored a launch-org bout (~550), busiest first, with their career shape: fights, rounds scored, rounds scored 10-8 or wider, three-judge cards that came back split, and lone_dissents — cards where this judge alone picked the other corner.

Parameters:
  - `q` (query) — Name substring, case-insensitive.
  - `org` (query) — one of `ufc`, `pfl`, `oktagon`, `bkfc`, `rizin`
  - `min_fights` (query) — Only judges with at least this many scored fights.
  - `cursor` (query) — Opaque cursor from `meta.pagination.next_cursor`.
  - `limit` (query)

### GET /v1/judges/{id}
One judge
A single official's career aggregates.

Parameters:
  - `id` (path, required)

### GET /v1/judges/{id}/scorecards
Everything a judge has scored
This judge's card for every launch-org bout they worked, newest first — the bout, its event, the decision type, and their own card, with where it sat on the panel: lone_dissent (three known verdicts and this judge alone differs from both others), split (three known verdicts that do not all agree) — both null when the panel is not three judges or a verdict is unknown — and colleagues, the other judges' totals and verdicts (the full round-by-round panel is on /v1/fights/{id}/scorecards).

Parameters:
  - `id` (path, required)
  - `cursor` (query) — Opaque cursor from `meta.pagination.next_cursor`.
  - `limit` (query)

## Fighters

### GET /v1/fighters
Roster search
Fighters with at least one launch-org fight (~4.5k).

Parameters:
  - `q` (query)
  - `org` (query) — one of `ufc`, `pfl`, `oktagon`, `bkfc`, `rizin`
  - `country` (query) — ISO-2 code (`US`, `GE`) or country name; matches either nationality.
  - `cursor` (query) — Opaque cursor from `meta.pagination.next_cursor`.
  - `limit` (query)

### GET /v1/fighters/{idOrSlug}
Fighter profile
Bio, records, stats, UFCalendar Power Index summary, and CC-licensed images with attribution metadata (displaying the credit is a license requirement).

Parameters:
  - `idOrSlug` (path, required) — Numeric id or slug of the fighter. Stale slugs 308-redirect to the canonical URL.
  - `include` (query) — one of `bonuses`, `credentials`

### GET /v1/fighters/{idOrSlug}/history
Complete career timeline
The fighter's FULL multi-promotion career: native tracked bouts (richest data, source: "native") merged with the wider multi-promotion career record (source: "history").

Parameters:
  - `idOrSlug` (path, required) — Numeric id or slug of the fighter. Stale slugs 308-redirect to the canonical URL.

### GET /v1/fighters/{idOrSlug}/stats
Career statistics
Raw per-source career rows, plus the readable split in meta.records / meta.stats (the same objects the fighter profile carries — read those unless you need the per-source rows).

Parameters:
  - `idOrSlug` (path, required) — Numeric id or slug of the fighter. Stale slugs 308-redirect to the canonical URL.

### GET /v1/fighters/{idOrSlug}/rankings
Ranking history
Official-board rows over time (org, board in the public vocabulary — official / meta — division slug, snapshot_date, rank, is_champion), newest snapshot first.

Parameters:
  - `idOrSlug` (path, required) — Numeric id or slug of the fighter. Stale slugs 308-redirect to the canonical URL.
  - `cursor` (query) — Opaque cursor from `meta.pagination.next_cursor`.
  - `limit` (query)

### GET /v1/fighters/{idOrSlug}/power-index
Power Index trajectory
Per-bout rating history from the UFCalendar Power Index engine (rating before/after, model win probability, outcome 1 / 0 / 0.5 and its word twin result).

Parameters:
  - `idOrSlug` (path, required) — Numeric id or slug of the fighter. Stale slugs 308-redirect to the canonical URL.

### GET /v1/compare
Compare two fighters
The tale of the tape in one call.

Parameters:
  - `a` (query, required) — First fighter: slug (preferred) or numeric id.
  - `b` (query, required) — Second fighter: slug (preferred) or numeric id.

## Rankings

### GET /v1/rankings/{org}
Official board (point-in-time)
**The time machine**: pass ?date=YYYY-MM-DD for the board as it stood on any date (UFC official back to 2013, 543 snapshots).

Parameters:
  - `org` (path, required) — one of `ufc`, `pfl`, `oktagon`, `bkfc`
  - `date` (query) — A real calendar day; the snapshot in force on that date is returned.
  - `board` (query) — Default `official` (for UFC that is the media-panel vote ufc.com publishes). UFC additionally has `meta`.

### GET /v1/rankings/{org}/{division}
One division
Same as the board endpoint, filtered to a division (lightweight, womens-strawweight, pound-for-pound, …) — entries carry the same movement / is_new / nationality / country_code, and meta the neighbouring snapshot dates.

Parameters:
  - `org` (path, required)
  - `division` (path, required)
  - `date` (query) — A real calendar day.
  - `board` (query) — Default `official`; UFC additionally has `meta`.

### GET /v1/champions
Current champions
Rank-0 rows of each launch org's latest official board, with the champion's nationality and country_code (ISO-3166 alpha-2).

Parameters:
  - `org` (query) — one of `ufc`, `pfl`, `oktagon`, `bkfc`, `rizin`
  - `division` (query) — Division slug as the boards carry it (`lightweight`, `womens-strawweight`, BKFC `cruiserweight`, …) or its display form. A division with a vacant title returns an empty list; one no board in scope carries 400s `invalid_division` listing the available ones.

## Power Index

### GET /v1/power-index/{org}
Power Index board
The UFCalendar Power Index (our own rating engine, hourly refresh) for one org, in three views: - current (default): top-rated fighters whose latest bout was in this org.

Parameters:
  - `org` (path, required) — one of `ufc`, `pfl`, `oktagon`, `bkfc`, `rizin`
  - `view` (query) — one of `current`, `movers`, `peaks`
  - `division` (query) — Division slug, e.g. `lightweight`, `light-heavyweight`, `womens-strawweight`.
  - `days` (query) — Window for `view=movers`, default 365; days is snapped to 30/90/180/365/730/1825/3650 (the nearest one), and `meta.days` reports the window actually used.
  - `limit` (query)

## Matchmaker

### GET /v1/matchmaker/{org}
Matchmaker board
UFCalendar's matchmaker for one MMA promotion (ufc, pfl, oktagon, rizin; bkfc 400s unsupported_org — the engine runs on the Power Index, which rates MMA only).

Parameters:
  - `org` (path, required) — one of `ufc`, `pfl`, `oktagon`, `rizin`
  - `division` (query) — One division slug from `meta.divisions`, e.g. `lightweight`, `womens-strawweight`. Omit for the cross-division board.
  - `limit` (query)

### GET /v1/matchmaker/next/{fighter}
Who should this fighter fight next
One fighter's most sensible next opponents by UFCalendar's matchmaker: the same engine and candidate pool as the board (their division's ladder plus the promotion's active Power Index fighters), every pairing scored, best first.

Parameters:
  - `fighter` (path, required) — Fighter slug (preferred) or numeric id.
  - `limit` (query)

## Stats

### GET /v1/stats/leaders
One stat leaderboard
One board from UFCalendar's Record Book rollup (rebuilt daily) for ONE promotion — org is required; there is no cross-promotion board on the API.

Parameters:
  - `org` (query, required) — one of `ufc`, `pfl`, `oktagon`, `bkfc`, `rizin`
  - `metric` (query, required) — one of `total_fights`, `wins`, `decision_wins`, `title_fight_wins`, `bonuses`, `finishes`, `win_streak`, `total_fight_seconds`, `avg_fight_seconds_short`, `avg_fight_seconds_long`, `control_time_sec`, `control_time_pct`, `sig_strikes_landed`, `total_strikes_landed`, `slpm`, `sapm`, `striking_differential`, `sig_strike_accuracy`, `sig_strike_defense`, `knockdowns`, `knockdowns_per_15`, `knockouts`, `takedowns_landed`, `td_avg`, `takedown_accuracy`, `takedown_defense`, `submission_attempts`, `sub_avg_per_15`, `submissions`, `fastest_finish`, `fastest_knockout`, `fastest_submission`, `longest_fight_seconds`, `most_sig_strikes_fight`, `most_takedowns_fight`, `most_knockdowns_fight`, `longest_control_fight`, `most_sig_strikes_round`, `most_takedowns_round`, `most_knockdowns_round`, `most_finishes_event`, `most_knockouts_event`, `shortest_card_seconds`, `finish_rate`, `ko_rate`, `sub_rate`, `decision_rate`, `avg_fight_seconds`
  - `division` (query) — one of `strawweight`, `flyweight`, `bantamweight`, `featherweight`, `lightweight`, `welterweight`, `super-welterweight`, `middleweight`, `light-heavyweight`, `heavyweight`, `super-heavyweight`, `womens-strawweight`, `womens-flyweight`, `womens-bantamweight`, `womens-featherweight`, `atomweight`, `ironweight`
  - `population` (query) — one of `all`, `active`
  - `country` (query) — ISO-3166 alpha-2 nationality; population `all` only.
  - `limit` (query)

### GET /v1/stats/record-book
The Record Book
Every leaderboard's top rows for ONE promotion in one call — org is required — grouped by category (fights, time, striking, grappling, records): data is [{ category, boards: [{ metric, rows }] }], rows exactly as on /v1/stats/leaders.

Parameters:
  - `org` (query, required) — one of `ufc`, `pfl`, `oktagon`, `bkfc`, `rizin`
  - `division` (query) — one of `strawweight`, `flyweight`, `bantamweight`, `featherweight`, `lightweight`, `welterweight`, `super-welterweight`, `middleweight`, `light-heavyweight`, `heavyweight`, `super-heavyweight`, `womens-strawweight`, `womens-flyweight`, `womens-bantamweight`, `womens-featherweight`, `atomweight`, `ironweight`
  - `population` (query) — one of `all`, `active`
  - `country` (query) — ISO-3166 alpha-2 nationality; population `all` only.
  - `scope` (query) — one of `career`, `single_fight`, `round`, `division`, `event`
  - `top` (query) — Rows per board.

### GET /v1/stats/years/{year}
A year in review
One calendar year (UTC) of one promotion — or, without org, of every covered promotion together (never a promotion the API does not cover) — in numbers, over completed bouts of non-cancelled events: events_completed, title_fights, methods (ko, sub, dec, other, total), divisions (weight classes with 10+ bouts, highest finish rate first, up to 14), fastest_finishes (up to 8 round-1 KO/TKO/submissions, seconds and time, winner and loser), upsets (up to 8 wins with the lowest pre-fight UFCalendar Power Index probability, win_probability), climbers (up to 8 fighters with the biggest Power Index gain over the year, gain in rating points), busiest (up to 8 fighters by bouts, with wins), orgs (events and completed bouts per promotion), countries (host countries by events, up to 12) and judges (the 5 judges on the most commission-scored bouts).

Parameters:
  - `year` (path, required) — Calendar year, 1993 to the current year.
  - `org` (query) — one of `ufc`, `pfl`, `oktagon`, `bkfc`, `rizin`

## Predictions

### GET /v1/predictions/upcoming
Model win probabilities (UFC)
UFCalendar model win probabilities for upcoming UFC bouts, published about three weeks ahead of each card and repriced at least weekly.

Parameters:
  - `event` (query) — Event slug or numeric id. Stale slugs are followed.

## Articles

### GET /v1/articles
Search UFCalendar articles
UFCalendar's own editorial archive — previews, recaps, investigations, fighter-pay and technique pieces — newest first.

Parameters:
  - `q` (query) — Keyword: matches title, summary or a tag.
  - `tag` (query) — Exact tag.
  - `locale` (query) — one of `en`, `es`, `pt`, `de`, `fr`, `tr`, `ru`, `ka`, `ja`, `ko`, `pl`, `sr`, `zh`
  - `cursor` (query) — Opaque cursor from `meta.pagination.next_cursor`.
  - `limit` (query)

### GET /v1/articles/{slug}
Get an article
One published UFCalendar article by slug: title, description, author_name, author_slug, tags, published_at, url and the full body_md (Markdown; the site's client-only widgets — fighter cards, embeds and offer blocks — are removed).

Parameters:
  - `slug` (path, required) — Article slug, from `/v1/articles`.
  - `locale` (query) — one of `en`, `es`, `pt`, `de`, `fr`, `tr`, `ru`, `ka`, `ja`, `ko`, `pl`, `sr`, `zh`

## Search

### GET /v1/search
Typeahead search
Accent-insensitive search across roster fighters and events (up to 8 of each).

Parameters:
  - `q` (query, required)

## Account

### GET /v1/webhook-endpoints
List your webhook endpoints
Pro and above.

### POST /v1/webhook-endpoints
Register a webhook endpoint
Pro+.

### DELETE /v1/webhook-endpoints/{id}
Delete a webhook endpoint
Pro and above.

Parameters:
  - `id` (path, required)

### POST /v1/webhook-endpoints/{id}/rotate-secret
Rotate an endpoint signing secret
Pro and above.

Parameters:
  - `id` (path, required)

### GET /v1/bulk/{kind}
Bulk snapshot (Enterprise only)
Enterprise only — the full dataset is licensed, not self-serve; contact api@ufcalendar.com.

Parameters:
  - `kind` (path, required) — one of `events.json.gz`, `fights.json.gz`, `fighters.json.gz`, `odds-closing.json.gz`

### GET /v1/usage
Your quota usage
Current-month usage for the calling key: tier, requests used/limit, rpm limit, and the reset as both a Unix timestamp (reset) and ISO-8601 (reset_at); month (YYYYMM) and period (YYYY-MM) are the same month in two spellings.
